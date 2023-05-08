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

const promptSuggestion = ref();

onMounted(() => {
  let urlParams = new URLSearchParams(window.location.search);
  if (urlParams.has('prompt')) {
    promptSuggestion.value = urlParams.get('prompt');
  }
});

function clearChat() {
  if (!loading.value) {
    messages.value = [];
  }
}

async function sendMessage(fromSuggestion = false) {
  let message = null;
  if (fromSuggestion) {
    message = promptSuggestion.value;
  } else {
    message = input.value;
  }

  if (loading.value || (message && message.trim() == '')) {
    return;
  }

  let lastResponse = messages.value
    .filter((item) => item.from === 'aigital')
    .slice(-1)
    .pop();

  loading.value = true;
  messages.value.push({
    from: 'user',
    content: message,
  });

  frameContainer.value.scrollToBottom();

  messages.value.push({
    from: 'aigital',
    content: '...',
  });
  let index = messages.value.length - 1;

  let prompt = message;

  if (lastResponse) {
    prompt =
      "Your previous message: '" +
      lastResponse.content +
      "'\n New user message: '" +
      prompt +
      "'. Continue conversation, don't send greetings text in the beginning. Try not to ask clarifying questions.";
  }

  input.value = null;
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
      <div
        v-if="promptSuggestion && messages.length < 1"
        style="
          display: flex;
          width: 100%;
          align-items: center;
          justify-content: space-around;
          padding: 20px 0;
        "
      >
        <div
          style="text-align: center; cursor: pointer"
          @click="sendMessage(true)"
        >
          <div style="padding-bottom: 15px">Пример запроса</div>
          <div
            style="
              padding: 10px;
              background-color: #f0f9ff;
              border: solid 1px #2a65df;
              color: #2a65df;
              width: 80%;
              margin: auto;
              border-radius: 10px;
            "
          >
            {{ promptSuggestion }}
          </div>
          <div style="text-size: 10px; opacity: 0.5; padding-top: 10px">
            Нажмите, чтобы отправить это сообщение
          </div>
        </div>
      </div>
      <template v-slot:messages v-if="messages.length > 0">
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
        <Input v-model="input" @submit="sendMessage(false)" />
      </template>
      <template v-slot:sendButton>
        <button class="sendButton" @click="sendMessage(false)">
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
