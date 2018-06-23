<template>
  <div class="todo-list">
    <input type="text" class="todo-list__input" placeholder="What needs to be done?" v-model="newTodo" @keyup.enter="addTodo">
    
    <transition-group name="fade-scale">
      <div v-for="(todo, index) in todosFiltered" :key="todo.id" class="todo-item">
        
        <input type="checkbox" v-model="todo.completed" @click="completedTodo(todo, index)">

        <div class="todo-item__left">
          <div v-if="!todo.editing" @dblclick="editTodo(todo)" class="todo-item__label" :class="{ completed : todo.completed }">{{todo.title}}</div>
          <input v-else class="todo-item__edit" type="text" v-model="todo.title" @blur="doneEdit(todo, index)" @keyup.enter="doneEdit(todo, index)" @keyup.esc="cancelEdit(todo)" v-focus >
        </div>

        <div class="todo-item__remove" @click="removeTodo(index)">&times;</div>

      </div>
    </transition-group>

    <div class="extra-container">
      <div class="extra-container__left">
        <input type="checkbox" id="check-all" :checked="!anyRemainig" @change="checkAllTodos()">
        <label for="check-all">Check all</label>
      </div>
      <div class="extra-container__right">{{ remainig }} items left</div>
    </div>

    <div class="extra-container">
      <div class="extra-container__left">
        <button class="extra-container__btn btn" :class="{ active : filter == 'all'}" @click="filter = 'all'">all</button>
        <button class="extra-container__btn btn" :class="{ active : filter == 'active'}" @click="filter = 'active'">active</button>
        <button class="extra-container__btn btn" :class="{ active : filter == 'completed'}" @click="filter = 'completed'">completed</button>
      </div>
      <div class="extra-container__rigth">
        <transition name="fade">
          <button class="btn" v-if="showClearCompletedButton" @click="clearCompleted">Clear completed</button>
        </transition>
      </div>
    </div>

  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'todo-list',
  async created() {
    const {data} = await axios.get('http://localhost:3000/todos')
    let currentId = await axios.get('http://localhost:3000/currentId')
    this.todos = data
    this.idForTodo = currentId.data.id

  },
  data () {
    return {
      idForTodo: Number,
      newTodo: '',
      beforeEditCache: '',
      filter: 'all',
      todos: []
    }
  },
  directives: {
    focus: {
      inserted: function (el) {
        el.focus()
      }
    }
  },
  computed: {
    remainig: function () {
      return this.todos.filter( todo => !todo.completed).length
    },
    anyRemainig() {
      return this.remainig != 0
    },
    todosFiltered() {
      if (this.filter == 'all') {
        return this.todos

      } else if (this.filter == 'active') {
        return this.todos.filter(todo => !todo.completed)

      } else if (this.filter == 'completed') {
        return this.todos.filter(todo => todo.completed)
      }

      return this.todos
    },
    showClearCompletedButton() {
      return this.todos.filter(todo => todo.completed).length > 0
    }
  },
  methods: {
    addTodo() {
      if (this.newTodo.trim().length == 0) return

      let todoItem = {
        id: this.idForTodo,
        title: this.newTodo,
        completed: false,
        editing: false
      }

      this.newTodo = ''
      this.idForTodo++

      let currentId = {id: this.idForTodo}

      this.todos.push(todoItem)
      axios.post('http://localhost:3000/todos', todoItem)
      axios.post('http://localhost:3000/currentId', currentId)
    },
    removeTodo(index) {
      axios.delete('http://localhost:3000/todos/' + this.todos[index].id )
      this.todos.splice(index, 1)
    },
    editTodo(todo) {
      this.beforeEditCache = todo.title
      todo.editing = true
    },
    doneEdit(todo, index) {
      if (todo.title.trim() == '') todo.title = this.beforeEditCache

      axios.put('http://localhost:3000/todos/' + (this.todos[index].id), { 'title'    : todo.title,
                                                                           'completed': todo.completed,
                                                                           'editing'  : false})
      todo.editing = false
    },
    cancelEdit(todo) {
      todo.title = this.beforeEditCache
      todo.editing = false
    },
    completedTodo(todo, index) {
      if ( todo.completed === false) {
        todo.completed = true
        axios.put('http://localhost:3000/todos/' + this.todos[index].id, { 'title'    : todo.title,
                                                                           'completed': todo.completed,
                                                                           'editing'  : false})
      } else {
        todo.completed = false
        axios.put('http://localhost:3000/todos/' + (this.todos[index].id), { 'title'    : todo.title,
                                                                             'completed': todo.completed,
                                                                             'editing'  : false})
      }
      
      
    },
    checkAllTodos() {
      this.todos.forEach( todo => todo.completed = event.target.checked )

      for ( let i = 0; i < this.todos.length; i++ ) {

        if ( this.todos[i].completed === true ) {
          axios.put('http://localhost:3000/todos/' + this.todos[i].id, { 'title'    : this.todos[i].title,
                                                                         'completed': this.todos[i].completed,
                                                                         'editing'  : false})
        } else {
          axios.put('http://localhost:3000/todos/' + this.todos[i].id, { 'title'    : this.todos[i].title,
                                                                         'completed': this.todos[i].completed,
                                                                         'editing'  : false})
        }

      }
    },
    clearCompleted() {

      for ( let i = 0; i < this.todos.length; i++ ) {

        if ( this.todos[i].completed === true ) {
          axios.delete('http://localhost:3000/todos/' + this.todos[i].id )
        }

      }

      this.todos = this.todos.filter(todo => !todo.completed)
      
    }
  }
}
</script>

<style lang="sass">

.todo-list
  background-color: #fffff9
  box-shadow:  1px 4px 20px rgba(0,0,0,0.3)
  &__input 
    width: 100%
    padding: 20px 30px
    font-size: 24px
    background-color: #fffff9
    border: none
    border-bottom: 1px solid #e7e7e7
    &:focus 
      outline: 0

.todo-item
  display: flex
  align-items: center
  color: #34495e
  padding: 5px 10px 5px 20px
  border-bottom: 1px solid #e7e7e7

  &__label
    padding: 15px

  &__remove
    cursor: pointer
    margin-left: auto
    &:hover
      color: #000000

  &__edit
    padding: 14px
    font-size: 24px
    color: #34495e
    border: 1px solid #eeeeee
    &:focus
      outline: none

.completed
  text-decoration: line-through
  color: #e7e7e7

.extra-container
  display: flex
  justify-content: space-between
  padding: 10px 20px
  font-size: 12px
  color: #888888
  // background-color: #fffff9
  border-bottom: 1px solid #e7e7e7

  &__left
    display: flex
    align-items: center
  &__btn
    margin-right: 10px
    &:last-child
      margin-right: 0

.btn
  display: inline-block
  padding: 2px 5px
  color: #fff
  background-color: #41b883
  border: 1px solid #e7e7e7
  cursor: pointer
  &:hover
    background-color: #35495e
  &:focus
    outline: none

.btn.active
  background-color: #35495e

.fade-enter-active, .fade-leave-active
  transition: ease opacity 0.3s

.fade-enter, .fade-leave-to
  opacity: 0

.fade-scale-enter-active, .fade-scale-leave-active
  transition: ease all 0.5s

.fade-scale-enter, .fade-scale-leave-to
  opacity: 0
  transform: scale(0)

</style>