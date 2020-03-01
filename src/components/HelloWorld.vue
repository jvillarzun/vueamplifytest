<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <div v-if="!signedIn">
      <amplify-authenticator></amplify-authenticator>
    </div>
    <div v-if="signedIn">
      <amplify-sign-out></amplify-sign-out>
      <button @click="createNewTodo">Add Todo</button>
      <ul>
        <li v-for="todo in todos" :key="todo.id">
          {{todo.name}} - {{todo.description}}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import { Auth } from 'aws-amplify'
import API, {  graphqlOperation } from '@aws-amplify/api';
import { AmplifyEventBus } from 'aws-amplify-vue'

import { createTodo } from "../graphql/mutations"
import { listTodos } from '../graphql/queries'
import { onCreateTodo } from '../graphql/subscriptions'

export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  created() {
    this.getUser()
    AmplifyEventBus.$on('authState', info => {
      console.log(info)
      if(info == 'signedIn')
        this.getUser()
      else
        this.signedIn = false
    })
    this.getData()
    this.subscribe()
  },
  data(){
    return {
      todos: [],
      signedIn: false
    }
  },
  methods :{
    async getUser() {
      try {
        const user = await Auth.currentAuthenticatedUser()
        this.signedIn = true
        console.log(user)
      } catch (error) {
        console.log(error)
        this.signedIn = false
      }
    },
    async createNewTodo(){
      const todo = { name: "Use AppSync" , description: "Realtime and Offline"}
      await API.graphql(graphqlOperation(createTodo, { input: todo }))
    },
    async getData(){
      const todoData = await API.graphql(graphqlOperation(listTodos))
      this.todos.push(...this.todos, ...todoData.data.listTodos.items);
    },
    subscribe(){
      API.graphql(graphqlOperation(onCreateTodo)).subscribe({
        next: (eventData) => {
          const todo = eventData.value.data.onCreateTodo;
          this.todos.push(todo);
        }
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
