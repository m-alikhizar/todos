<template>
  <div>
    <section class="todoapp">
      <header class="header">
        <h1>todos</h1>
        <input class="new-todo"
          autofocus autocomplete="off"
          placeholder="What needs to be done?"
          v-model="newTodo"
          :style="{border: '2px solid' + (newTodo ? palette[Math.floor((Math.random() * palette.length) + 1)] : palette[0])}"
          @keyup.enter="addTodo">
      </header>
      <section class="main" v-show="todos.length" v-cloak>
        <input class="toggle-all" type="checkbox" v-model="allDone">
        <ul class="todo-list">
          <li v-for="todo in filteredTodos"
            class="todo"
            :key="todo.id"
            :style="{border: '2px solid' + (newTodo ? palette[Math.floor((Math.random() * palette.length) + 1)] : palette[0])}"
            :class="{ completed: todo.completed, editing: todo == editedTodo }">
            <div class="view">
              <input class="toggle" type="checkbox" v-model="todo.completed">
              <label @dblclick="editTodo(todo)">{{ todo.title }}</label>
              <button class="destroy" @click="removeTodo(todo)"></button>
            </div>
            <input class="edit" type="text"
              v-model="todo.title"
              v-todo-focus="todo == editedTodo"
              @blur="doneEdit(todo)"
              @keyup.enter="doneEdit(todo)"
              @keyup.esc="cancelEdit(todo)">
          </li>
        </ul>
      </section>
      <footer class="footer" v-show="todos.length" v-cloak :style="{border: '2px solid' + (newTodo ? palette[Math.floor((Math.random() * palette.length) + 1)] : palette[0]) }">
        <span class="todo-count">
          <strong>{{ remaining }}</strong> {{ remaining | pluralize }} left
        </span>
        <ul class="filters">
          <li><a href="#/all" :class="{ selected: visibility == 'all' }">All</a></li>
          <li><a href="#/active" :class="{ selected: visibility == 'active' }">Active</a></li>
          <li><a href="#/completed" :class="{ selected: visibility == 'completed' }">Completed</a></li>
        </ul>
        <button class="clear-completed" @click="removeCompleted" v-show="todos.length > remaining">
          Clear completed
        </button>
      </footer>
    </section>
    <footer class="info">
      <p>Double-click to edit a todo</p>
    </footer>
  </div>
</template>

<script>

  // visibility filters
  var filters = {
    all: function (todos) {
      return todos
    },
    active: function (todos) {
      return todos.filter(function (todo) {
        return !todo.completed
      })
    },
    completed: function (todos) {
      return todos.filter(function (todo) {
        return todo.completed
      })
    }
  }

  var uid = '';

  export default {
    name: 'HelloWorld',
    data: () => ({
      todos: [],
      newTodo: '',
      editedTodo: null,
      visibility: 'all',
      palette: palette
    }),
    // watch todos change for localStorage persistence
    watch: {
      todos: {
        handler: function (todos) {
          firebase.database().ref('/todos').set(JSON.stringify(todos));
        },
        deep: true
      }
    },

    computed: {
      filteredTodos: function () {
        return filters[this.visibility](this.todos)
      },
      remaining: function () {
        return filters.active(this.todos).length
      },
      allDone: {
        get: function () {
          return this.remaining === 0
        },
        set: function (value) {
          this.todos.forEach(function (todo) {
            todo.completed = value
          })
        }
      }
    },

    filters: {
      pluralize: function (n) {
        return n === 1 ? 'item' : 'items'
      }
    },

    // methods that implement data logic.
    // note there's no DOM manipulation here at all.
    methods: {
      addTodo: function () {
        var value = this.newTodo && this.newTodo.trim()
        if (!value) {
          return
        }
        this.todos.push({
          id: uid++,
          title: value,
          completed: false
        })
        this.newTodo = ''
      },

      removeTodo: function (todo) {
        this.todos.splice(this.todos.indexOf(todo), 1)
      },

      editTodo: function (todo) {
        this.beforeEditCache = todo.title
        this.editedTodo = todo
      },

      doneEdit: function (todo) {
        if (!this.editedTodo) {
          return
        }
        this.editedTodo = null
        todo.title = todo.title.trim()
        if (!todo.title) {
          this.removeTodo(todo)
        }
      },

      cancelEdit: function (todo) {
        this.editedTodo = null
        todo.title = this.beforeEditCache
      },

      removeCompleted: function () {
        this.todos = filters.active(this.todos)
      },

      setVisibility: function(visibility) {
        this.visibility = visibility;
      },
    },

    // a custom directive to wait for the DOM to be updated
    // before focusing on the input field.
    // http://vuejs.org/guide/custom-directive.html
    directives: {
      'todo-focus': function (el, binding) {
        if (binding.value) {
          el.focus();
        }
      }
    },

    mounted() {

      const that = this;

      firebase.database().ref('/todos').once('value').then(function(snapshot) {
        var todos = JSON.parse(snapshot && snapshot.val());
        todos.forEach(function (todo, index) {
          todo.id = index
        })
        uid = todos.length;

        that.todos = todos;
      });

      // handle routing

      function onHashChange () {
        var visibility = window.location.hash.replace(/#\/?/, '');
        if (filters[visibility]) {
          that.setVisibility(visibility);
        } else {
          window.location.hash = '';
          that.setVisibility('all');
        }
      }

      window.addEventListener('hashchange', onHashChange);
      onHashChange();
    }
  };

  const palette = [
    '#ffffff',
    '#ffde00a6',
    '#05dbc57a',
    '#ff7fab8a',
    '#b2ff59d6',
    '#ffac36a6'

  ]
</script>

<style>
/* body {
  color: transparent;
  color: #ffde00;
  color: #05dbc5;
  color: #ff7faba6;
  color: #b2ff59d6;
  color: #ffac36a6;
} */
</style>
