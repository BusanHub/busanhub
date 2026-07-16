<script setup>
defineProps({
  posts: {
    type: Array,
    required: true
  },
  boardMessage: {
    type: String,
    default: ''
  },
  boardMode: {
    type: String,
    default: 'list'
  },
  selectedPost: {
    type: Object,
    default: null
  },
  form: {
    type: Object,
    required: true
  }
})

const emit = defineEmits([
  'open-create',
  'submit-post',
  'cancel-form',
  'open-post',
  'back-list',
  'edit-post',
  'delete-post',
  'update-field',
  'toggle-like',
  'toggle-bookmark'
])

function formatRelativeTime(ts) {
  if (!ts) return ''
  const time = typeof ts === 'number' ? ts : Number(ts)
  if (!time || Number.isNaN(time)) return ''
  const now = Date.now()
  const diff = now - time
  const minute = 60 * 1000
  const hour = 60 * minute
  const day = 24 * hour

  if (diff < minute) return '방금전'
  if (diff < hour) return `${Math.floor(diff / minute)}분전`
  const daysDiff = Math.floor(diff / day)
  if (daysDiff >= 1) return daysDiff === 1 ? '어제' : `${daysDiff}일 전`
  return `${Math.floor(diff / hour)}시간 전`
}
</script>

<template>
  <div class="section-head">
    <div>
      <h2>익명 커뮤니티</h2>
      <p>비밀번호를 이용해 수정·삭제가 가능한 로컬 게시판입니다.</p>
    </div>
    <button class="primary-btn" @click="emit('open-create')">글쓰기</button>
  </div>

  <div v-if="boardMessage" class="message">{{ boardMessage }}</div>

  <div v-if="boardMode === 'form'" class="form-box">
    <h3>{{ form._isDelete ? '게시글 삭제' : form.editingId ? '게시글 수정' : '게시글 작성' }}</h3>
    <input :value="form.title" placeholder="제목" @input="emit('update-field', 'title', $event.target.value)" />
    <textarea :value="form.content" rows="5" placeholder="내용" @input="emit('update-field', 'content', $event.target.value)"></textarea>
    <input :value="form.password" type="password" placeholder="수정/삭제용 비밀번호" @input="emit('update-field', 'password', $event.target.value)" />
    <div class="actions">
      <button class="primary-btn" @click="emit('submit-post')">{{ form._isDelete ? '삭제' : '저장' }}</button>
      <button class="ghost-btn" @click="emit('cancel-form')">취소</button>
    </div>
  </div>

  <div v-else-if="boardMode === 'detail' && selectedPost" class="detail-box">
    <h3>{{ selectedPost.title }}</h3>
    <p class="meta">{{ formatRelativeTime(selectedPost.createdAt) }} · 조회수 {{ selectedPost.views || 0 }}</p>
    <p>{{ selectedPost.content }}</p>
    <div class="actions">
        <button class="icon-btn" @click.stop="emit('toggle-like', selectedPost.id)">❤️ {{ selectedPost.likes || 0 }}</button>
      <button class="icon-btn" @click.stop="emit('toggle-bookmark', selectedPost.id)">🔖 {{ selectedPost.bookmarks || 0 }}</button>
      <button class="primary-btn" @click="emit('edit-post', selectedPost)">수정</button>
      <button class="ghost-btn" @click="emit('delete-post', selectedPost.id)">삭제</button>
      <button class="ghost-btn" @click="emit('back-list')">목록</button>
    </div>
  </div>

  <ul v-else class="post-list">
    <li v-for="post in posts" :key="post.id" class="post-item" @click="emit('open-post', post)">
      <div>
        <strong>{{ post.title }}</strong>
        <p>{{ post.content }}</p>
      </div>
      <span>
          <button class="icon-btn" @click.stop="emit('toggle-like', post.id)">❤️ {{ post.likes || 0 }}</button>
          <button class="icon-btn" @click.stop="emit('toggle-bookmark', post.id)">🔖 {{ post.bookmarks || 0 }}</button>
        {{ formatRelativeTime(post.createdAt) }} · 조회수 {{ post.views || 0 }}
      </span>
    </li>
  </ul>
</template>

<style scoped>
.section-head {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  margin-bottom: 16px;
  flex-wrap: wrap;
}

.primary-btn,
.ghost-btn {
  border: none;
  border-radius: 10px;
  padding: 10px 14px;
  cursor: pointer;
}

.primary-btn {
  background: #2b7fff;
  color: white;
}

.ghost-btn {
  background: #eef5ff;
  color: #2b7fff;
}

.message {
  background: #ffe9e9;
  color: #b30000;
  padding: 10px 12px;
  border-radius: 10px;
  margin-bottom: 12px;
}

.form-box,
.detail-box {
  display: grid;
  gap: 10px;
}

input,
textarea {
  width: 100%;
  border: 1px solid #dce8f5;
  border-radius: 10px;
  padding: 10px 12px;
  font: inherit;
}

.actions {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.post-list {
  list-style: none;
  padding: 0;
  margin: 0;
  display: grid;
  gap: 10px;
}

.post-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border: 1px solid #e8eef6;
  border-radius: 12px;
  padding: 12px;
  cursor: pointer;
  background: #fbfdff;
}

.post-item p {
  margin: 4px 0 0;
  color: #647382;
  font-size: 0.95rem;
}

.icon-btn {
  border: none;
  background: transparent;
  cursor: pointer;
  color: #54707f;
  margin-right: 6px;
  font-size: 0.95rem;
}

@media (max-width: 700px) {
  .post-item {
    flex-direction: column;
    align-items: flex-start;
    gap: 6px;
  }
}
</style>