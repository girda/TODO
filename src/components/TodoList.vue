<template>
  <div>
    <input type="text" class="todo-input" placeholder="What needs to be done?" v-model="newTodo" @keyup.enter="addTodo">
    
    <div v-for="(todo, index) in todos" :key="todo.id" class="todo-item">
      <div>
        {{todo.title}}
      </div>

      <div class="remove-item" @click="removeTodo(index)">&times;</div>

    </div>

  </div>
</template>

<script>
export default {
  name: 'todo-list',
  data () {
    return {
      newTodo: '',
      idForTodo: 1,
      todos: []
    }
  },
  methods: {
    addTodo() {
      if (this.newTodo.trim().length == 0) return

      this.todos.push({
        id: this.idForTodo,
        title: this.newTodo,
        completed: false
      })

      this.newTodo = ''
      this.idForTodo++
    },
    removeTodo(index) {
      this.todos.splice(index, 1)
    }
  }
}
</script>

<style lang="sass">

.todo-input 
  width: 100%
  padding: 20px 30px
  font-size: 24px
  margin-bottom: 15px
  background-color: #fffff9
  border: none
  box-shadow:  1px 4px 20px rgba(0,0,0,0.3)
  &:focus 
    outline: 0

.todo-item
  display: flex
  align-items: center
  justify-content: space-between
  padding: 20px 10px
  margin-bottom: 10px
  background-color: #fffff9
  box-shadow:  1px 2px 10px rgba(0,0,0,0.3)

.remove-item
  cursor: pointer
  margin-left: 14px
  &:hover
    color: #000000

</style>