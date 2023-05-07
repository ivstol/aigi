<template>
  <div class="message">
    <div class="messageInner aigitalMessage" v-if="props.from == 'aigital'">
      {{ content }}
    </div>
    <div class="messageInner userMessage" v-else>
      {{ content }}
    </div>
  </div>
</template>

<script setup>
const props = defineProps(['from', 'content']);
</script>

<style>
.message {
  display: flex;
  margin-bottom: 20px;
}

.messageInner {
  border-radius: 20px;
  overflow: hidden;
  padding: 10px 15px;
  display: inline-flex;
  max-width: 85%;
  white-space: pre-wrap;
  overflow-wrap: anywhere;
}

.aigitalMessage {
  background-color: #f1f1f1;
}

.userMessage {
  margin-left: auto;
  background-color: #f1f1f1;
  background: rgb(9, 80, 255);
  background: linear-gradient(
    33deg,
    rgba(9, 80, 255, 1) 0%,
    rgba(0, 142, 255, 1) 100%
  );
  color: white;
}
</style>
