<template>
  <textarea
    id="textarea"
    rows="1"
    ref="textarea"
    placeholder="Введите ваш запрос..."
    :value="props.modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
  ></textarea>
</template>
<script setup>
import autosize from 'autosize';

import { ref, onMounted, watch, nextTick } from 'vue';
const props = defineProps(['modelValue']);
const textarea = ref();

const emit = defineEmits(['submit', 'update:modelValue']);

watch(
  () => props.modelValue,
  (value) => {
    nextTick(() => {
      autosize.update(textarea.value);
    });
  }
);

onMounted(() => {
  autosize(textarea.value);

  textarea.value.addEventListener('keydown', function (event) {
    if (event.keyCode === 13 && !event.shiftKey) {
      // Check for enter key without shift key
      event.preventDefault(); // Prevent the newline character from being added to the textarea
      emit('submit');
    }
  });
});
</script>

<style>
#textarea {
  width: 100%;
  max-height: 200px;
  resize: none;
  border: none;
  padding: 15px 15px;
  font-size: 16px;
}

#textarea:focus {
  outline: none;
}

#textarea::-webkit-scrollbar {
  width: 4px;
}

#textarea::-webkit-scrollbar-button {
  display: none;
}

#textarea::-webkit-scrollbar-track {
  background-color: transparent;
}

#textarea::-webkit-scrollbar-thumb {
  background: #d2d2d2;
}
</style>
