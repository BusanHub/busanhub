<script setup>
defineProps({
  messages: {
    type: Array,
    required: true
  },
  draft: {
    type: String,
    default: ''
  },
  isOpen: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['toggle-chat', 'update-draft', 'send-message'])
</script>

<template>
  <div>
    <button class="chat-toggle" @click="emit('toggle-chat')">💬</button>

    <div v-if="isOpen" class="chat-box">
      <div class="chat-header">
        <strong>BusanHub 챗봇</strong>
        <button @click="emit('toggle-chat')">닫기</button>
      </div>

      <div class="chat-messages">
        <div
          v-for="(msg, index) in messages"
          :key="index"
          :class="['bubble', msg.role === 'user' ? 'user' : 'bot']"
        >
          {{ msg.content }}
        </div>
      </div>

      <div class="chat-input">
        <input
          :value="draft"
          placeholder="질문을 입력하세요"
          @input="emit('update-draft', $event.target.value)"
          @keyup.enter.prevent="emit('send-message')"
        />
        <button @click="emit('send-message')">전송</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.chat-toggle {
  position: fixed;
  right: 24px;
  bottom: 24px;
  width: 56px;
  height: 56px;
  border: none;
  border-radius: 50%;
  background: #2b7fff;
  color: white;
  font-size: 1.4rem;
  cursor: pointer;
}

.chat-box {
  position: fixed;
  right: 24px;
  bottom: 92px;
  width: min(360px, calc(100vw - 24px));
  background: white;
  border-radius: 16px;
  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.16);
  overflow: hidden;
}

.chat-header,
.chat-input {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px;
  background: #f5f9ff;
}

.chat-messages {
  display: grid;
  gap: 8px;
  padding: 12px;
  max-height: 320px;
  overflow-y: auto;
  background: #fcfdff;
}

.bubble {
  max-width: 85%;
  padding: 10px 12px;
  border-radius: 12px;
  font-size: 0.95rem;
  line-height: 1.4;
}

.bubble.user {
  justify-self: end;
  background: #2b7fff;
  color: white;
}

.bubble.bot {
  justify-self: start;
  background: #eef5ff;
  color: #2c3f53;
}

.chat-input input {
  flex: 1;
  margin-right: 8px;
  border: 1px solid #dce8f5;
  border-radius: 10px;
  padding: 10px 12px;
}

.chat-input button {
  border: none;
  border-radius: 10px;
  padding: 10px 14px;
  background: #2b7fff;
  color: white;
  cursor: pointer;
}
</style>