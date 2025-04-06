<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'

// Message interface
interface Message {
  id: number
  text: string
  sender: 'user' | 'system'
  timestamp: number
  sessionId: string // Add sessionId property
}

// Session management
const currentSessionId = ref('')
const savedSessions = ref<string[]>([])

// Generate new session ID
const generateSessionId = () => {
  return Date.now().toString() + '-' + Math.random().toString(36).substring(2, 9)
}

// State for messages
const messages = ref<Message[]>([])
const newMessage = ref('')

// Format timestamp to readable format
const formatTimestamp = (timestamp: number) => {
  const date = new Date(timestamp)
  const now = new Date()
  const options: Intl.DateTimeFormatOptions = { 
    month: 'short', 
    day: 'numeric',
    hour: '2-digit', 
    minute: '2-digit' 
  }
  
  // Add year if the message is from a different year
  if (date.getFullYear() !== now.getFullYear()) {
    options.year = 'numeric'
  }
  
  return date.toLocaleString([], options)
}

// Get last message timestamp for a session
const getLastMessageTimestamp = (sessionId: string): number | null => {
  const savedMessages = localStorage.getItem('chatMessages')
  if (savedMessages) {
    const allMessages: Message[] = JSON.parse(savedMessages)
    const sessionMessages = allMessages.filter(message => message.sessionId === sessionId)
    
    if (sessionMessages.length > 0) {
      // Find the message with the latest timestamp
      const lastMessage = sessionMessages.reduce((latest, current) => 
        current.timestamp > latest.timestamp ? current : latest
      , sessionMessages[0])
      
      return lastMessage.timestamp
    }
  }
  return null
}

// Format session name for display with last message timestamp
const formatSessionName = (sessionId: string) => {
  const date = new Date(parseInt(sessionId.split('-')[0]))
  const sessionDate = date.toLocaleString([], { 
    month: 'short',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  })
  
  const lastTimestamp = getLastMessageTimestamp(sessionId)
  if (lastTimestamp) {
    return `${sessionDate} (Last msg: ${formatTimestamp(lastTimestamp)})`
  }
  
  return sessionDate
}

// Save messages to localStorage
const saveMessages = () => {
  const allMessages = JSON.parse(localStorage.getItem('chatMessages') || '[]')
  
  // Filter out messages from current session and add new ones
  const otherSessionMessages = allMessages.filter((msg: Message) => 
    msg.sessionId !== currentSessionId.value
  )
  
  const updatedMessages = [...otherSessionMessages, ...messages.value]
  localStorage.setItem('chatMessages', JSON.stringify(updatedMessages))
}

// Load messages from localStorage
const loadMessages = () => {
  const savedMessages = localStorage.getItem('chatMessages')
  if (savedMessages) {
    const allMessages: Message[] = JSON.parse(savedMessages)
    // Only load messages from the current session
    messages.value = allMessages.filter(message => message.sessionId === currentSessionId.value)
  }
}

// Save sessions to localStorage
const saveSessions = () => {
  localStorage.setItem('chatSessions', JSON.stringify(savedSessions.value))
}

// Load sessions from localStorage
const loadSessions = () => {
  const sessions = localStorage.getItem('chatSessions')
  if (sessions) {
    savedSessions.value = JSON.parse(sessions)
  }
}

// Start a new chat session
const startNewSession = () => {
  currentSessionId.value = generateSessionId()
  savedSessions.value.push(currentSessionId.value)
  saveSessions()
  
  // Initialize with welcome message
  messages.value = [{ 
    id: 1, 
    text: 'Welcome to the chat!', 
    sender: 'system',
    timestamp: Date.now(),
    sessionId: currentSessionId.value
  }]
  saveMessages()
}

// Initialize component
onMounted(() => {
  loadSessions()
  
  // Create new session if no sessions exist
  if (savedSessions.value.length === 0) {
    startNewSession()
  } else {
    currentSessionId.value = savedSessions.value[savedSessions.value.length - 1]
    loadMessages()
    
    // Add welcome message if no messages exist for this session
    if (messages.value.length === 0) {
      messages.value = [{ 
        id: 1, 
        text: 'Welcome to the chat!', 
        sender: 'system',
        timestamp: Date.now(),
        sessionId: currentSessionId.value
      }]
      saveMessages()
    }
  }
})

// Watch for session changes
watch(currentSessionId, () => {
  loadMessages()
})

// Send message function
const sendMessage = () => {
  if (newMessage.value.trim()) {
    messages.value.push({
      id: Date.now(),
      text: newMessage.value,
      sender: 'user',
      timestamp: Date.now(),
      sessionId: currentSessionId.value
    })
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
    <div class="session-selector">
      <select v-model="currentSessionId" class="session-select">
        <option v-for="session in savedSessions" :key="session" :value="session">
          {{ formatSessionName(session) }}
        </option>
      </select>
      <button @click="startNewSession" class="new-session-button">New Chat</button>
    </div>
    
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

.session-selector {
  display: flex;
  margin-bottom: 1rem;
  gap: 0.5rem;
}

.session-select {
  flex: 1;
  padding: 0.5rem;
  border-radius: 0.5rem;
  background-color: rgba(0, 0, 0, 0.1);
  border: none;
  color: inherit;
  outline: none;
  font-size: 0.9rem; /* Make font a bit smaller to fit longer option text */
  text-overflow: ellipsis;
}

.new-session-button {
  background-color: #646cff;
  color: white;
  border: none;
  border-radius: 0.5rem;
  padding: 0.5rem 1rem;
  cursor: pointer;
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
