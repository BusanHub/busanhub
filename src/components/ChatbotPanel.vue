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
        <div class="header-left">
          <strong class="header-title">BusanHub 챗봇</strong>
        </div>
        <button class="close-btn" @click="emit('toggle-chat')">✕</button>
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
  background: linear-gradient(90deg,#00a8ff,#00d2fc);
  color: white;
  font-size: 1.6rem;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  line-height: 1;
  padding: 0;
  box-sizing: border-box;
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
  height: 560px; /* increased height */
  display: flex;
  flex-direction: column;
}

.chat-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  background: linear-gradient(90deg,#00a8ff,#00d2fc);
  color: white;
}

.chat-input {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px;
  background: #f5f9ff;
}

.chat-messages {
  display: block;
  gap: 8px;
  padding: 16px;
  flex: 1 1 auto;
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
  outline: none;
}

/* prevent focus from changing border color or showing outline */
.chat-input input:focus {
  border-color: #dce8f5 !important;
  box-shadow: none !important;
}

.chat-input button {
  border: none;
  border-radius: 10px;
  padding: 10px 14px;
  background: linear-gradient(90deg,#00a8ff,#00d2fc);
  color: white;
  cursor: pointer;
}

.close-btn{
  border: none;
  background: transparent;
  font-size: 1.1rem;
  cursor: pointer;
  color: #fff;
}

.header-left{ display:flex; align-items:center; gap:10px; color: #fff }
.robot-icon{ width:22px; height:22px; display:block; flex:0 0 auto }
.header-title{ color: #fff; line-height:1 }
</style>