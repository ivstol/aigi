<template>
  <div class="container">
    <div ref="content" class="content">
      <div class="messages">
        <slot name="messages"></slot>
      </div>
    </div>
    <div class="footer">
      <div class="input">
        <slot name="input"></slot>
      </div>
      <slot name="sendButton"></slot>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';
const content = ref();

function scrollToBottom() {
  content.value.scrollTop = content.value.scrollHeight;
}

defineExpose({
  scrollToBottom,
});
</script>

<style>
.container {
  position: relative;
  height: 500px;
  width: 100%;
  max-width: 750px;
}

.content {
  height: 500px;
  width: 100%;
  max-width: 750px;
  overflow-y: auto;
  overflow-x: hidden;
}

.footer {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  max-width: 750px;
  display: flex;
  overflow-wrap: anywhere;
  align-items: end;
  background: rgb(255, 255, 255);
  background: linear-gradient(
    180deg,
    rgba(255, 255, 255, 0) 0%,
    rgba(255, 255, 255, 1) 50%
  );
}

.messages {
  padding: 20px;
  display: flex;
  flex-direction: column;
  padding-bottom: 70px;
}

.input {
  display: flex;
  width: 100%;
  margin: 15px;
  box-shadow: rgba(99, 99, 99, 0.1) 0px 2px 8px 0px;
  border: solid 1px rgba(99, 99, 99, 0.1);
  border-radius: 10px;
  overflow: hidden;
}

.content::-webkit-scrollbar {
  width: 4px;
}

.content::-webkit-scrollbar-button {
  display: none;
}

.content::-webkit-scrollbar-track {
  background-color: transparent;
}

.content::-webkit-scrollbar-thumb {
  background: #d2d2d2;
}
</style>
