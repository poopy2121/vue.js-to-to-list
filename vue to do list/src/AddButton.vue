<script setup>
import { computed } from "vue";

const props = defineProps({
  inputContent: String,
  inputCategory: [String, null],
});

const emit = defineEmits(["add-todo"]);

const isDisabled = computed(() => !props.inputContent.trim() || !props.inputCategory);

const handleClick = () => {
  if (!isDisabled.value) {
    emit("add-todo");
  }
};
</script>

<template>
  <input
    type="submit"
    value="Add todo"
    :disabled="isDisabled"
    @click="handleClick"
  />
</template>
