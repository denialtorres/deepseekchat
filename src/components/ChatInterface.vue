<template>
  <div class="chat-container">
    <div class="chat-messages" ref="messagesContainer">
      <div v-for="(message, index) in messages" :key="index" :class="['message', message.role]">
        <template v-if="message.role === 'assistant-think'">
          <div class="think-header">
            <button class="toggle-think" @click="toggleThought(index)">
              {{ messageVisibility[index] ? 'Hide' : 'Show' }} thought process
            </button>
          </div>
          <p v-show="messageVisibility[index]"><i>Thinking: </i>{{ message.content }}</p>
        </template>
        <p v-else>{{ message.content }}</p>
      </div>
      <div v-if="isThinking" class="message thinking">
        <p>🤔🤔🤔..</p>
      </div>
    </div>

    <div class="chat-input">
      <textarea
        v-model="userInput"
        @keyup.enter.exact="sendMessage"
        placeholder="Type your message..."
      ></textarea>
      <button @click="sendMessage" :disabled="isThinking">Send</button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick } from 'vue'

const messages = ref([])
const userInput = ref('')
const messagesContainer = ref(null)
const isThinking = ref(false)
const messageVisibility = ref({})

const toggleThought = (index) => {
  messageVisibility.value[index] = !messageVisibility.value[index]
}

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
  isThinking.value = true

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
    // messages.value.push({
    //   role: 'assistant',
    //   content: data.response,
    // })

    const thinkMatch = data.response.match(/<think>(.*?)<\/think>/s)

    if (thinkMatch) {
      // Add thinking process as a separate message
      console.log(thinkMatch)
      messages.value.push({
        role: 'assistant-think',
        content: thinkMatch[1].trim(),
      })

      // Add the remaining content as the main response
      const mainContent = data.response.replace(/<think>.*?<\/think>/s, '').trim()
      if (mainContent) {
        messages.value.push({
          role: 'assistant',
          content: mainContent,
        })
      }
    } else {
      // if no think tags, add the response as is
      messages.value.push({
        role: 'assistant',
        content: data.response,
      })
    }

    // Scroll to bottom
    await nextTick()
    messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight
  } catch (error) {
    console.error('Error:', error)
    messages.value.push({
      role: 'system',
      content: 'Error: Unable to get response from the model',
    })
  } finally {
    isThinking.value = false
  }
}
</script>
