<template>
  <div class="chat-container">
    <div class="chat-messages" ref="messagesContainer">
      <div v-for="(message, index) in messages" :key="index" class="['message', message.role]">
        <p>{{ message.content }}</p>
      </div>
    </div>

    <div class="chat-input">
      <textarea
        v-model="userInput"
        @keyup.enter.exact="sendMessage"
        placeholder="Type your message..."
      ></textarea>
      <button @click="sendMessage">Send</button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick } from 'vue'

const messages = ref([])
const userInput = ref('')
const messagesContainer = ref(null)

const sendMessage = async () => {
  if (!userInput.value.trim()) return

  // Add user message
  messages.value.push({
    role: 'user',
    content: userInput.value,
  })

  // Clear Input
  const userMessage = userInput.value
  userInput.value = ''

  // curl http://localhost:11434/api/generate -d '{
  //   "model": "deepseek-r1:1.5b",
  //   "prompt": "Why is the sky blue?"
  // }'

  try {
    const response = await fetch('http://localhost:11434/api/generate', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        model: 'deepseek-r1:1.5b',
        prompt: userMessage,
        stream: false,
      }),
    })
    const data = await response.json()

    // Add AI response
    messages.value.push({
      role: 'assistant',
      content: data.response,
    })

    // Scroll to bottom
    await nextTick()
    messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight
  } catch (error) {
    console.error('Error:', error)
    messages.value.push({
      role: 'system',
      content: 'Error: Unable to get response from the model',
    })
  }
}
</script>
