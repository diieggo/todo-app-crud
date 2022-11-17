<template>
  <div>
    <div id="main-container">
      <h2>Todos</h2>
      <div id="add-container">
        <form @submit.prevent="addTodo">
            <input type="text" v-model="newTodoTitle" class="insert-todo" placeholder="Add new to-do" required>
            <input type="submit">
        </form>
      </div>
      <div v-bind:key="todo.id" v-for="todo in todos">
        <div class="todo">
          <div class="todo-body" v-bind:class="{completed: todo.completed}">
            <input type="checkbox" v-bind:checked="todo.completed" @change="todoCompleted(todo.id)">
            <p>{{todo.title}}</p>
          </div>
          <div class="todo-actions">
            <button @click="deleteTodo(todo.id)">Delete</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
// Imports
import { ref, onMounted } from 'vue';
import {
  collection, onSnapshot,
  addDoc, doc, deleteDoc, updateDoc,
  query, orderBy
} from "firebase/firestore";
import { db } from '@/firebase';

// Firebase Refs
const todosCollectionRef = collection(db, 'todos');
const todosCollectionQuery = query(todosCollectionRef, orderBy('date', 'desc'));

// Todos
const todos = ref([]);

onMounted(async () => {
  try {
    await onSnapshot(todosCollectionQuery, (querySnapshot) => {
      const fbTodos = [];
      querySnapshot.forEach((doc) => {
        const todo = {
          id: doc.id,
          title: doc.data().title,
          completed: doc.data().completed
        }
        fbTodos.push(todo);
      });
      todos.value = fbTodos;
    });
  } catch(err) {
    console.log(err)
  }
})

var newTodoTitle = ref('');
const addTodo = (async () => {
  try {
    await addDoc(todosCollectionRef, {
      title: newTodoTitle.value,
      completed: false,
      date: Date.now()
    });
    newTodoTitle.value = '';
  } catch(err) {
    console.log(err);
  }
})

const deleteTodo = (async(id) => {
  try {
    await deleteDoc(doc(todosCollectionRef, id));
  } catch(err) {
    console.log(err)
  }
})

const todoCompleted = (async (id) => {
  try {
    const todoIndex = todos.value.findIndex(todo => todo.id === id)

    await updateDoc(doc(todosCollectionRef, id), {
      completed: !todos.value[todoIndex].completed
  });
  } catch(err) {
    console.log(err)
  }
})
</script>