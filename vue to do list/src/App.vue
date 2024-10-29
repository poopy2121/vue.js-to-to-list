<script setup>
import { ref, onMounted, computed, watch } from "vue";
import AddButton from "./AddButton.vue";

const todos = ref([]);
const name = ref(localStorage.getItem("name") || "");
const input_content = ref("");
const input_category = ref(null);
const input_priority = ref("1");
const errorMessage = ref("");

const buttonClickedWhileDisabled = ref(false);

watch(name, (newName) => {
  localStorage.setItem("name", newName);
});

const todos_asc = computed(() =>
  [...todos.value].sort((a, b) => b.priority - a.priority || b.createdAt - a.createdAt)
);

const addTodo = () => {
  if (input_content.value.trim() === "" || input_category.value === null) {
    if (buttonClickedWhileDisabled.value) {
      errorMessage.value = "Please enter content and select a category.";
    }
    return;
  }

  todos.value.push({
    content: input_content.value,
    category: input_category.value,
    priority: parseInt(input_priority.value, 10),
    done: false,
    createdAt: new Date().getTime(),
  });

  input_content.value = "";
  input_category.value = null;
  input_priority.value = "1";
  errorMessage.value = "";

  buttonClickedWhileDisabled.value = false;
};

const showError = () => {
  buttonClickedWhileDisabled.value = true;
};

const removeTodo = (todoToRemove) => {
  todos.value = todos.value.filter((todo) => todo !== todoToRemove);
};

watch(todos, (newVal) => localStorage.setItem("todos", JSON.stringify(newVal)), { deep: true });
onMounted(() => {
  todos.value = JSON.parse(localStorage.getItem("todos")) || [];
});
</script>

<template>
  <main class="app">
    <section class="greeting">
      <h2>What's up,
        <input id="username" type="text" placeholder="Your name..." v-model="name" />
      </h2>
    </section>

    <section class="create-todo">
      <h3>CREATE A TODO</h3>
      <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>
      <form @submit.prevent="addTodo">
        <h4>What's on your todo list?</h4>
        <input type="text" placeholder="e.g., make a video" v-model="input_content" />

        <h4>Pick a category</h4>
        <div class="options">
          <label>
            <input type="radio" name="category" value="business" v-model="input_category" />
            <span class="bubble business"></span>
            <div>Business</div> 
          </label>
          <label>
            <input type="radio" name="category" value="personal" v-model="input_category" />
            <span class="bubble personal"></span>
            <div>Personal</div> 
          </label>
        </div>

        <h4>Set Priority</h4>
        <select v-model="input_priority" class="priority-select">
          <option value="1">Low</option>
          <option value="2">Medium</option>
          <option value="3">High</option>
        </select>

        <AddButton
          :inputContent="input_content"
          :inputCategory="input_category"
          @add-todo="addTodo"
          @show-error="showError"
        />
      </form>
    </section>

    <section class="todo-list">
      <h3>TODO LIST</h3>
      <transition-group name="slide" tag="div" class="list" id="todo-list">
        <div v-for="todo in todos_asc" :key="todo.createdAt" :class="`todo-item ${todo.done && 'done'}`">
          <label>
            <input type="checkbox" v-model="todo.done" />
            <span :class="`bubble ${todo.category}`"></span>
          </label>

          <div class="todo-content">
            <input type="text" v-model="todo.content" :class="{ 'crossed-out': todo.done }" />
          </div>

          <div class="priority" :class="`priority-${todo.priority}`">
            Priority: {{ todo.priority === 3 ? "High" : todo.priority === 2 ? "Medium" : "Low" }}
          </div>

          <div class="actions">
            <button class="delete" @click="removeTodo(todo)">Delete</button>
          </div>
        </div>
      </transition-group>
    </section>
  </main>
</template>

<style scoped>
.error-message {
  background-color: #ffdddd;
  color: #a33;
  padding: 0.75rem 1rem;
  border-radius: 0.25rem;
  margin-bottom: 1rem;
  font-weight: bold;
}

.slide-enter-active, .slide-leave-active {
  transition: all 0.5s ease;
}
.slide-leave-active {
  opacity: 0;
  transform: translateX(100%);
}

#username {
	font-size: 1em;
	font-weight: 700;
}

.priority-3 { color: red; margin-right: 1.5rem; }
.priority-2 { color: orange; margin-right: 1.5rem; }
.priority-1 { color: green; margin-right: 1.5rem; }

.priority-select {
  width: 100%;
  font-size: 1.125rem;
  padding: 0.75rem 1rem;
  color: var(--dark);
  background-color: #fff;
  border: 2px solid var(--grey);
  border-radius: 0.5rem;
  box-shadow: var(--shadow);
  margin-bottom: 1.5rem;
  cursor: pointer;
  transition: border-color 0.2s ease, color 0.2s ease;
}

.priority-select option[value="1"] { color: green; }
.priority-select option[value="2"] { color: orange; }
.priority-select option[value="3"] { color: red; }

.priority-select:focus {
  outline: none;
  border-color: var(--primary);
}

.priority-select:hover {
  border-color: var(--primary);
}

.create-todo .options label div {
  font-size: 1.125rem;
  margin-top: 0.5rem; 
}
</style>

<style>
:root {
	--primary: #EA40A4;
	--business: #3A82EE;
	--personal: var(--primary);
	--light: #EEE;
	--grey: #888;
	--dark: #313154;
	--danger: #ff5b57;
	--shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
	--business-glow: 0px 0px 4px rgba(58, 130, 238, 0.75);
	--personal-glow: 0px 0px 4px rgba(234, 64, 164, 0.75);
}

* {
	margin: 0;
	padding: 0;
	box-sizing: border-box;
	font-family: 'montserrat', sans-serif;
}

.todo-item .actions .delete {
  background-color: var(--danger);
  padding: 0.5rem;
  border-radius: 0.25rem;
  color: #FFF;
  cursor: pointer;
  transition: 0.2s ease-in-out;
}

.todo-item .actions .delete:hover {
  opacity: 0.75;
}

input:not([type="radio"]):not([type="checkbox"]), button {
	appearance: none;
	border: none;
	outline: none;
	background: none;
	cursor: initial;
}

body {
	background: var(--light);
	color: var(--dark);
}

section {
	margin-top: 2rem;
	margin-bottom: 2rem;
	padding-left: 1.5rem;
	padding-right: 1.5em;
}

.crossed-out {
  text-decoration: line-through;
  color: var(--grey);
}

h3 { color: var(--dark); font-size: 1rem; font-weight: 400; margin-bottom: 0.5rem; }

h4 { color: var(--grey); font-size: 0.875rem; font-weight: 700; margin-bottom: 0.5rem; }

.greeting .title {
	display: flex;
}

.greeting .title input {
	margin-left: 0.5rem;
	flex: 1 1 0%;
	min-width: 0;
}

.greeting .title,
.greeting .title input {
	color: var(--dark);
	font-size: 1.5rem;
	font-weight: 700;
}

.create-todo input[type="text"] {
	display: block;
	width: 100%;
	font-size: 1.125rem;
	padding: 1rem 1.5rem;
	color: var(--dark);
	background-color: #FFF;
	border-radius: 0.5rem;
	box-shadow: var(--shadow);
	margin-bottom: 1.5rem;
}

.create-todo .options {
	display: grid;
	grid-template-columns: repeat(2, 1fr);
	grid-gap: 1rem;
	margin-bottom: 1.5rem;
}

.create-todo .options label {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	padding: 1.5rem;
	background-color: #FFF;
	border-radius: 0.5rem;
	box-shadow: var(--shadow);
	cursor: pointer;
}

input[type="radio"],
input[type="checkbox"] {
	display: none;
}

.bubble {
	display: flex;
	align-items: center;
	justify-content: center;
	width: 20px;
	height: 20px;
	border-radius: 50%;
	border: 2px solid var(--business);
	box-shadow: var(--business-glow);
}

.bubble.personal {
	border-color: var(--personal);
	box-shadow: var(--personal-glow);
}

.bubble::after {
	content: "";
	display: block;
	opacity: 0;
	width: 0px;
	height: 0px;
	background-color: var(--business);
	box-shadow: var(--business-glow);
	border-radius: 50%;
	transition: 0.2s ease-in-out;
}

.bubble.personal::after {
	background-color: var(--personal);
	box-shadow: var(--personal-glow);
}

input:checked ~ .bubble::after {
	width: 10px;
	height: 10px;
	opacity: 1;
}

.create-todo input[type="submit"] {
	display: block;
	width: 100%;
	font-size: 1.125rem;
	padding: 1rem 1.5rem;
	color: #FFF;
	background-color: var(--primary);
	border-radius: 0.5rem;
	box-shadow: var(--personal-glow);
	cursor: pointer;
	transition: 0.2s ease-in-out;
}

.create-todo input[type="submit"]:hover {
	opacity: 0.75;
}

.create-todo input[type="submit"]:disabled {
	background-color: #ccc;
	cursor: not-allowed;
	opacity: 0.7;
}

.todo-list .list {
	margin: 1rem 0;
} 

.todo-list .todo-item {
	display: flex;
	align-items: center;
	background-color: #FFF;
	padding: 1rem;
	border-radius: 0.5rem;
	box-shadow: var(--shadow);
	margin-bottom: 1rem;
}

.todo-item label {
	display: block;
	margin-right: 1rem;
	cursor: pointer;
}

.todo-item .todo-content {
	flex: 1 1 0%;
}

.todo-item .todo-content input {
	color: var(--dark);
	font-size: 1.125rem;
}

.todo-item .actions {
	display: flex;
	align-items: center;
}

.todo-item .actions button {
	display: block;
	padding: 0.5rem;
	border-radius: 0.25rem;
	color: #FFF;
	cursor: pointer;
	transition: 0.2s ease-in
}

</style>
