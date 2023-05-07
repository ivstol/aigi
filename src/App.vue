<script setup>
import Message from './components/Message.vue';
import Input from './components/Input.vue';
import FrameContainer from './components/FrameContainer.vue';
import { ref, onMounted } from 'vue';
import axios from 'redaxios';

const messages = ref([]);

const input = ref('');

const loading = ref(false);

const frameContainer = ref();

function clearChat() {
  if (!loading.value) {
    messages.value = [];
  }
}

async function sendMessage() {
  if (loading.value || input.value.trim() == '') {
    return;
  }

  let lastResponse = messages.value
    .filter((item) => item.from === 'aigital')
    .slice(-1)
    .pop();

  loading.value = true;
  messages.value.push({
    from: 'user',
    content: input.value,
  });

  frameContainer.value.scrollToBottom();

  messages.value.push({
    from: 'aigital',
    content: '...',
  });
  let index = messages.value.length - 1;

  let prompt =
    "User message: '" +
    input.value +
    "'. You are ChatGPT simulator of online course Aiversity. Ask clarifying questions if necessary and the user is not ordered not to ask questions.";

  if (lastResponse) {
    prompt =
      "Your previous message: '" +
      lastResponse.content +
      "'\n New user message: '" +
      prompt +
      "'. Continue conversation, don't send greetings text in the beginning. Ask clarifying questions if necessary and the user is not ordered not to ask questions.";
  }

  input.value = '';
  try {
    const response = await fetch(
      'https://corsproxy.io/?https%3A%2F%2Ftestapp.aigital.co%2Fapi%2Fopen-ai%2Faiversity%2Fstream',
      {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          promptParams: {
            prompt: prompt,
          },
        }),
      }
    );

    const reader = response.body.getReader();
    let text = '';

    messages.value[index].content = '';

    while (true) {
      const { done, value } = await reader.read();

      if (done || new TextDecoder().decode(value).includes('data: [DONE]')) {
        loading.value = false;
        break;
      }

      messages.value[index].content += new TextDecoder()
        .decode(value)
        .replace(/\n\n/g, '')
        .replace(/\[BREAK\]/g, '\n')
        .replace(/data: /g, '');

      frameContainer.value.scrollToBottom();
    }
  } catch (error) {
    loading.value = false;

    let errorResponse = messages.value
      .filter((item) => item.from === 'aigital')
      .slice(-1)
      .pop();

    errorResponse.from = 'error';
    errorResponse.content = '';
    frameContainer.value.scrollToBottom();
  }
}
</script>

<template>
  <div>
    <FrameContainer ref="frameContainer">
      <template v-slot:messages>
        <Message
          v-for="message in messages"
          :from="message.from"
          :content="message.content"
        />
        <div
          v-if="messages.length > 1 && loading == false"
          class="clearChat"
          @click="clearChat"
        >
          <div class="clearChatButton">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 20 20"
              fill="currentColor"
              style="width: 16px; height: 16px; padding-right: 7px"
            >
              <path
                fill-rule="evenodd"
                d="M15.312 11.424a5.5 5.5 0 01-9.201 2.466l-.312-.311h2.433a.75.75 0 000-1.5H3.989a.75.75 0 00-.75.75v4.242a.75.75 0 001.5 0v-2.43l.31.31a7 7 0 0011.712-3.138.75.75 0 00-1.449-.39zm1.23-3.723a.75.75 0 00.219-.53V2.929a.75.75 0 00-1.5 0V5.36l-.31-.31A7 7 0 003.239 8.188a.75.75 0 101.448.389A5.5 5.5 0 0113.89 6.11l.311.31h-2.432a.75.75 0 000 1.5h4.243a.75.75 0 00.53-.219z"
                clip-rule="evenodd"
              />
            </svg>
            Начать заново
          </div>
        </div>
      </template>
      <template v-slot:input>
        <Input v-model="input" @submit="sendMessage" />
      </template>
      <template v-slot:sendButton>
        <button class="sendButton" @click="sendMessage">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            viewBox="0 0 24 24"
            fill="currentColor"
            style="width: 20px; height: 20px"
          >
            <path
              d="M3.478 2.405a.75.75 0 00-.926.94l2.432 7.905H13.5a.75.75 0 010 1.5H4.984l-2.432 7.905a.75.75 0 00.926.94 60.519 60.519 0 0018.445-8.986.75.75 0 000-1.218A60.517 60.517 0 003.478 2.405z"
            />
          </svg>
        </button>
      </template>
    </FrameContainer>
  </div>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
.clearChat {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: space-around;
}
.clearChatButton {
  display: inline-flex;
  padding: 5px 10px;
  align-items: center;
  cursor: pointer;
  border-radius: 5px;
  transition: all;
  opacity: 0.7;
}
.clearChatButton:hover {
  background-color: #f1f1f1;
  opacity: 1;
}
</style>
