<script setup lang="ts">
import { ref, onMounted } from 'vue'

// Message interface
interface Message {
  id: number
  text: string
  sender: 'user' | 'system'
  timestamp: number // Add timestamp property
}

// State for messages
const messages = ref<Message[]>([
  { 
    id: 1, 
    text: 'Welcome to the chat!', 
    sender: 'system',
    timestamp: Date.now() 
  }
])
const newMessage = ref('')

// Format timestamp to readable format
const formatTimestamp = (timestamp: number) => {
  const date = new Date(timestamp)
  return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
}

// Save messages to localStorage
const saveMessages = () => {
  localStorage.setItem('chatMessages', JSON.stringify(messages.value))
}

// Load messages from localStorage
const loadMessages = () => {
  const savedMessages = localStorage.getItem('chatMessages')
  if (savedMessages) {
    messages.value = JSON.parse(savedMessages)
  }
}

// Load messages when component mounts
onMounted(() => {
  loadMessages()
})

// Send message function
const sendMessage = () => {
  if (newMessage.value.trim()) {
    messages.value.push({
      id: Date.now(),
      text: newMessage.value,
      sender: 'user',
      timestamp: Date.now()
    })
    // Save messages to localStorage after adding a new one
    saveMessages()
    newMessage.value = ''
  }
}

// Handle Enter key press
const handleKeyDown = (event: KeyboardEvent) => {
  if (event.key === 'Enter' && !event.shiftKey) {
    event.preventDefault()
    sendMessage()
  }
}
</script>

<template>
  <div class="chat-container">
    <div class="messages-container">
      <div v-for="message in messages" :key="message.id" class="message" :class="message.sender">
        <div class="message-content">
          {{ message.text }}
          <div class="message-timestamp">{{ formatTimestamp(message.timestamp) }}</div>
        </div>
      </div>
    </div>
    <div class="input-container">
      <input 
        v-model="newMessage" 
        @keydown="handleKeyDown" 
        placeholder="Type a message and press Enter..." 
        class="message-input"
      />
      <button @click="sendMessage" class="send-button">Send</button>
    </div>
  </div>
</template>

<style scoped>
.chat-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  max-width: 800px;
  margin: 0 auto;
  padding: 1rem;
  box-sizing: border-box;
}

.messages-container {
  flex: 1;
  overflow-y: auto;
  margin-bottom: 1rem;
  display: flex;
  flex-direction: column;
  padding: 1rem;
  border-radius: 0.5rem;
  background-color: rgba(0, 0, 0, 0.1);
}

.message {
  margin-bottom: 1rem;
  max-width: 80%;
  padding: 0.75rem 1rem;
  border-radius: 1rem;
  word-break: break-word;
}

.message.user {
  align-self: flex-end;
  background-color: #646cff;
  color: white;
  border-bottom-right-radius: 0.25rem;
}

.message.system {
  align-self: flex-start;
  background-color: #1a1a1a;
  color: white;
  border-bottom-left-radius: 0.25rem;
}

.input-container {
  display: flex;
  border-radius: 1rem;
  background-color: rgba(0, 0, 0, 0.1);
  padding: 0.5rem;
}

.message-input {
  flex: 1;
  border: none;
  outline: none;
  background-color: transparent;
  padding: 0.75rem;
  color: inherit;
  font-family: inherit;
  border-radius: 0.5rem;
}

.send-button {
  background-color: #646cff;
  color: white;
  border: none;
  border-radius: 0.5rem;
  padding: 0.5rem 1rem;
  margin-left: 0.5rem;
  cursor: pointer;
}

.message-timestamp {
  font-size: 0.75rem;
  opacity: 0.7;
  margin-top: 0.25rem;
  text-align: right;
}
</style>
