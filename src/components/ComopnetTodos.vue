<template>
  <v-app>
    <div><input type="text" name="" id=""></div>
    <div class="inpDiv">
      <input class="inp" type="text" v-model="userId" placeholder="User ID" />
      <input
        class="inp"
        type="text"
        v-model="quantity"
        placeholder="Quantity"
      />
      <v-btn @click="getData">Search</v-btn>
      <v-btn color="red darken-1" text @click="deleteUser">Delete</v-btn>
    </div>
    <v-dialog v-model="dialog" persistent max-width="500">
      <template v-slot:activator="{ on, attrs }">
        <v-btn color="primary" dark v-bind="attrs" v-on="on">
          Create To Do
        </v-btn>
      </template>
      <v-card>
        <v-card-title class="text-h5"> Create To Do </v-card-title>
        <v-row>
          <v-col>
            <v-card-text>User id:</v-card-text>
          </v-col>
          <v-col>
            <v-text-field
              v-model="userId"
              type="number"
              style="width: 200px"
            ></v-text-field>
          </v-col>
        </v-row>
        <v-row>
          <v-col>
            <v-card-text>id</v-card-text>
          </v-col>
          <v-col>
            <v-text-field
              v-model="id"
              type="number"
              style="width: 200px"
            ></v-text-field>
          </v-col>
        </v-row>
        <v-row>
          <v-col>
            <v-card-text>User Name:</v-card-text>
          </v-col>
          <v-col>
            <v-text-field
              v-model="username"
              type="string"
              style="width: 200px"
            ></v-text-field>
          </v-col>
        </v-row>
        <v-row>
          <v-col>
            <v-card-text>Email:</v-card-text>
          </v-col>
          <v-col>
            <v-text-field
              v-model="email"
              type="string"
              style="width: 200px"
            ></v-text-field>
          </v-col>
        </v-row>
        <v-row>
          <v-col>
            <v-card-text>Title:</v-card-text>
          </v-col>
          <v-col>
            <v-text-field
              v-model="title"
              type="string"
              style="width: 200px"
            ></v-text-field>
          </v-col>
        </v-row>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="green darken-1" text @click="createTodo">
            Create
          </v-btn>
          <v-btn color="green darken-1" text @click="updateTodo">
            Update
          </v-btn>
          <v-btn color="green darken-1" text @click="close"> Close </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-data-table
      :headers="headers"
      :items="todos"
      :items-per-page="-1"
      @click:row="edit"
      class="elevation-1"
    >
      <template v-slot:top>
        <v-dialog v-model="editDialog">
          <v-card>
            <v-card-text> {{ editableTodo }} </v-card-text>
          </v-card>
        </v-dialog>
      </template>
    </v-data-table>
  </v-app>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      editableTodo: null,
      dialog: false,
      editDialog: false,
      userId: null,
      id: null,
      username: null,
      email: null,
      title: null,
      todos: [],
      headers: [
        { text: "My progect", align: 'left', sortable: true, value: 'name' },
        { text: 'User ID', align: 'left', sortable: true, value: 'userId' },
        { text: 'ID', align: 'left', sortable: true, value: 'id' },
        { text: 'User Name', align: 'left', sortable: true, value: 'username' },
        { text: 'Email', align: 'left', sortable: true, value: 'email' },
        { text: 'Title', align: 'left', sortable: true, value: 'title' },
      ],
      quantity: null,
    }
  },
  mounted() {
    this.getData();
  },
  methods: {
    async getData() {
      try {
        const todosResponse = await axios.get('https://jsonplaceholder.typicode.com/todos');
        const usersResponse = await axios.get('https://jsonplaceholder.typicode.com/users');

        const todos = todosResponse.data;
        const users = usersResponse.data;

        let filteredTodos = todos;

        if (this.userId !== null) {
          filteredTodos = filteredTodos.filter(todo => todo.userId === parseInt(this.userId));
        }

        if (this.quantity !== null) {
          const quantity = parseInt(this.quantity);
          filteredTodos = filteredTodos.slice(0, quantity);
        }
        if (filteredTodos.length < this.quantity){
          alert(filteredTodos.length)
          filteredTodos = todos
          this.quantity = ""
          this.userId = ""
        }
        console.log(this.todos,"++");
        console.log(todos,"--");
        const mappedData = filteredTodos.map(todo => {
          const user = users.find(user => user.id === todo.userId);
          return {
            userId: todo.userId,
            title: todo.title,
            username: user.username,
            id: todo.id,
            email: user.email
          };
        });
        this.todos = mappedData;
      } catch (error) {
        console.log(error);
      }
    },
    updateTodo() {
      const index = this.todos.indexOf(this.editableTodo);
      this.editableTodo.title = this.title;
      this.editableTodo.userId = this.userId;
      this.editableTodo.id = this.id;
      this.editableTodo.username = this.username;
      this.editableTodo.email = this.email;
      this.todos.splice(index, 1, this.editableTodo);
      this.editDialog = false;
      this.clear();
    },
    createTodo() {
      const newTodo = {
        id: this.id,
        title: this.title,
        userId: this.userId,
        username: this.username,
        email: this.email
      };
      this.todos.push(newTodo);
      this.close();
      this.clear();
    },
    deleteUser() {
      if (this.editableTodo) {
        const index = this.todos.indexOf(this.editableTodo);
        this.todos.splice(index, 1);
        this.editDialog = false;
        this.clear();
      }
    },
    close() {
      this.dialog = false;
    },
    clear() {
      this.title = null;
      this.userId = null;
      this.id = null;
      this.username = null;
      this.email = null;
    },
    edit(todo) {
      this.editableTodo = todo;
      this.editDialog = !this.editDialog;
    },
  }
}
</script>

<style>
.inp {
  width: 200px;
  height: 35px;
  border: 0.5px solid black;
  text-align: center;
  font-size: 16px;
  border-radius: 5px;
  font-family: Arial, Helvetica, sans-serif;
  margin: 20px;
  outline: none;
}
</style>
