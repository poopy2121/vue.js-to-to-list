<script setup>
import { computed } from "vue";

const props = defineProps({
  inputContent: String,
  inputCategory: [String, null],
});

const emit = defineEmits(["add-todo", "show-error"]);

const isDisabled = computed(() => !props.inputContent.trim() || !props.inputCategory);

const handleClick = () => {
  if (!isDisabled.value) {
    emit("add-todo");
  } else {
    emit("show-error");
  }
};
</script>

<template>
  <button
    :class="{ 'disabled-btn': isDisabled, 'add-todo-btn': true }"
    @click="handleClick"
  >
    Add todo
  </button>
</template>

<style scoped>
.add-todo-btn {
  width: 100%;
  font-size: 1.125rem;
  padding: 1rem 1.5rem;
  color: #FFF;
  background-color: var(--primary);
  border-radius: 0.5rem;
  box-shadow: var(--personal-glow);
  cursor: pointer;
  transition: opacity 0.2s ease-in-out;
}

.add-todo-btn:hover {
  opacity: 0.75;
}

.disabled-btn {
  background-color: #ccc;
  cursor: not-allowed;
  opacity: 0.7;
}
</style>
