<template>
  <div>
    <section class="todoapp">
      <header class="header">
        <h1>todos</h1>
        <input class="new-todo"
          autofocus autocomplete="off"
          placeholder="What needs to be done?"
          v-model="newTodo"
          :style="{border: '2px solid' + color}"
          @keyup.enter="addTodo"
          @keydown="randomize">
      </header>
      <section class="main" v-show="todos.length" v-cloak>
        <input class="toggle-all" type="checkbox" v-model="allDone">
        <ul class="todo-list">
          <li v-for="todo in filteredTodos"
            class="todo"
            :key="todo.id"
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
      <footer class="footer" v-show="todos.length" v-cloak>
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

  // localStorage persistence
  var STORAGE_KEY = 'todos-vuejs-2.0'
  var todoStorage = {
    save: function (todos) {

      firebase.database().ref('/todos').set(JSON.stringify(todos));

      // localStorage.setItem(STORAGE_KEY, JSON.stringify(todos))
    }
  }

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

  export default {
    name: 'HelloWorld',
    data: () => ({
      todos: [],
      newTodo: '',
      editedTodo: null,
      visibility: 'all',
      color: palette[0]
    }),
    // watch todos change for localStorage persistence
    watch: {
      todos: {
        handler: function (todos) {
          todoStorage.save(todos)
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

      randomize: function(e) {
        // console.log(e.target.value);((
        if(e.target.value) {
          this.color = palette[Math.floor((Math.random() * palette.length) + 1)];
        } else {
          this.color = palette[0];
        }
      },
      addTodo: function () {
        var value = this.newTodo && this.newTodo.trim()
        if (!value) {
          return
        }
        this.todos.push({
          id: todoStorage.uid++,
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
        todoStorage.uid = todos.length;

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
    'transparent',
    '#ffde00',
    '#0936ff',
    '#ff0466',
    '#ffc003',
    '#7f39fb',
    '#b00020',
    '#05dbc5',
    '#bb86fc',
    '#ff7fab',
    '#b2ff59',
    '#ffac36'

  ]
</script>
