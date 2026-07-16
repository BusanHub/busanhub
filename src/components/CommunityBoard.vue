<script setup>
import { computed, toRefs, ref } from 'vue'

const props = defineProps({
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
// expose props as refs to use reactively in script and template
const { posts, boardMessage, boardMode, selectedPost, form } = toRefs(props)

// Return sequential number (oldest post = 1) for a given post
// build a stable mapping key for posts
const keyOf = (p) => (p && p.id) ? `id:${p.id}` : `ct:${Number(p?.createdAt)||0}-t:${String(p?.title||'')}`

// computed map from key -> sequential number (oldest = 1)
const postNumberMap = computed(() => {
  try {
    const arr = (posts.value || []).slice().sort((a, b) => (Number(a?.createdAt) || 0) - (Number(b?.createdAt) || 0))
    const map = new Map()
    arr.forEach((p, idx) => map.set(keyOf(p), idx + 1))
    return map
  } catch {
    return new Map()
  }
})

// local search state
const search = ref('')
function onSearchInput(e) {
  search.value = e.target.value || ''
}

// posts sorted for display: filter by search (title/content), then createdAt descending (newest first)
const sortedPosts = computed(() => {
  try {
    const q = String(search.value || '').trim().toLowerCase()
    const base = (posts.value || []).filter((p) => {
      if (!q) return true
      return String(p.title || '').toLowerCase().includes(q) || String(p.content || '').toLowerCase().includes(q)
    })
    return base.slice().sort((a, b) => (Number(b?.createdAt) || 0) - (Number(a?.createdAt) || 0))
  } catch {
    return posts.value || []
  }
})

function getPostNumber(post) {
  return postNumberMap.value.get(keyOf(post)) ?? ''
}

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
    <div class="left">
      <div class="title-row">
        <svg class="bubble-icon" viewBox="0 0 24 24" width="32" height="32" aria-hidden="true">
          <defs>
            <linearGradient id="gradCB" x1="0%" y1="0%" x2="100%" y2="0%">
              <stop offset="0%" stop-color="#00a8ff" />
              <stop offset="100%" stop-color="#00d2fc" />
            </linearGradient>
          </defs>
          <path d="M3 5a3 3 0 013-3h12a3 3 0 013 3v7a3 3 0 01-3 3H9l-6 3V5z" fill="none" stroke="url(#gradCB)" stroke-width="1.6" stroke-linejoin="round"/>
        </svg>
        <h2>소통마당</h2>
      </div>
      <div class="search-wrap">
        <svg class="search-icon" viewBox="0 0 24 24" width="18" height="18" aria-hidden="true"><path d="M10.5 3a7.5 7.5 0 105.303 12.803l4.697 4.697 1.414-1.414-4.697-4.697A7.5 7.5 0 0010.5 3zM5 10.5a5.5 5.5 0 1111 0 5.5 5.5 0 01-11 0z"/></svg>
        <input class="search-input" placeholder="제목, 내용 검색..." :value="search" @input="onSearchInput" />
      </div>
    </div>
    <button class="primary-btn write-btn" @click="emit('open-create')">
      <svg viewBox="0 0 24 24" width="16" height="16" aria-hidden="true" class="btn-icon"><path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04a1 1 0 000-1.41l-2.34-2.34a1 1 0 00-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83z"/></svg>
      <span>글쓰기</span>
    </button>
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
    <p>{{ selectedPost.content }}</p>
    <p class="meta meta-below">{{ formatRelativeTime(selectedPost.createdAt) }} · 조회수 {{ selectedPost.views || 0 }}</p>
    <div class="actions">
        <button :class="['icon-btn', { liked: selectedPost.liked }]" @click.stop="emit('toggle-like', selectedPost.id)" aria-label="좋아요">
          <svg viewBox="0 0 24 24" width="18" height="18" aria-hidden="true" focusable="false"><path d="M12 21s-7.5-4.35-10-7.09C-0.1 10.6 2.5 6 6 6c2.1 0 3.2 1.1 4 2.1C11.8 7.1 12.9 6 15 6c3.5 0 6.1 4.6 4 7.91C19.5 16.65 12 21 12 21z"/></svg>
          <span class="count">{{ selectedPost.likes || 0 }}</span>
        </button>
      <button :class="['icon-btn', { bookmarked: selectedPost.bookmarked }]" @click.stop="emit('toggle-bookmark', selectedPost.id)" aria-label="북마크">
          <svg viewBox="0 0 24 24" width="14" height="18" aria-hidden="true" focusable="false"><path d="M6 2h12v18l-6-3-6 3V2z"/></svg>
          <span class="count">{{ selectedPost.bookmarks || 0 }}</span>
      </button>
      <button class="primary-btn" @click="emit('edit-post', selectedPost)">수정</button>
      <button class="ghost-btn" @click="emit('delete-post', selectedPost.id)">삭제</button>
      <button class="ghost-btn" @click="emit('back-list')">목록</button>
    </div>
  </div>

  <ul v-else class="post-list">
    <li v-for="(post, idx) in sortedPosts" :key="post.id || idx" class="post-item" @click="emit('open-post', post)">
      <div class="post-main">
        <div class="index-number">{{ getPostNumber(post) }}</div>
        <div class="post-body">
          <strong>{{ post.title }}</strong>
          <p>{{ post.content }}</p>
          <p class="meta meta-below">{{ formatRelativeTime(post.createdAt) }} · 조회수 {{ post.views || 0 }}</p>
        </div>
      </div>
      <div class="post-actions">
        <button :class="['icon-btn', { liked: post.liked }]" @click.stop="emit('toggle-like', post.id)" aria-label="좋아요">
          <svg viewBox="0 0 24 24" width="18" height="18" aria-hidden="true" focusable="false"><path d="M12 21s-7.5-4.35-10-7.09C-0.1 10.6 2.5 6 6 6c2.1 0 3.2 1.1 4 2.1C11.8 7.1 12.9 6 15 6c3.5 0 6.1 4.6 4 7.91C19.5 16.65 12 21 12 21z"/></svg>
          <span class="count">{{ post.likes || 0 }}</span>
        </button>
        <button :class="['icon-btn', { bookmarked: post.bookmarked }]" @click.stop="emit('toggle-bookmark', post.id)" aria-label="북마크">
          <svg viewBox="0 0 24 24" width="14" height="18" aria-hidden="true" focusable="false"><path d="M6 2h12v18l-6-3-6 3V2z"/></svg>
          <span class="count">{{ post.bookmarks || 0 }}</span>
        </button>
      </div>
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

.section-head .left {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.title-row { display:flex; align-items:center; gap:10px }
.bubble-icon { display: block }
.section-head h2 { margin:0; font-size:1.9rem; color: #0d3b66 }
.search-wrap{ display:flex; align-items:center; gap:10px; width:560px; max-width:100%; padding:6px 10px; border-radius:24px; border:1px solid #d9dde0; transition: border-color .12s ease, box-shadow .12s ease; background:#fff }
.search-icon{ color: #d9dde0; flex:0 0 auto }
.search-wrap:focus-within{ border-color: #00a8ff; box-shadow: 0 6px 18px rgba(0,168,255,0.12) }
.search-wrap:focus-within .search-icon{ color: #00a8ff }
.search-input{ border:none; outline:none; width:100%; padding:8px 6px; font:inherit }
.search-input::placeholder { color:#aab6bf }

.primary-btn,
.ghost-btn {
  border: none;
  border-radius: 10px;
  padding: 10px 14px;
  cursor: pointer;
}

.primary-btn {
  background: linear-gradient(90deg,#00a8ff,#00d2fc);
  color: white;
}

.primary-btn.write-btn{ display:inline-flex; align-items:center; gap:10px; padding:8px 18px; border-radius:999px; font-weight:600 }
.primary-btn.write-btn .btn-icon{ fill: #fff }

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

.post-main {
  flex: 1 1 auto;
  display: flex;
  align-items: center; /* vertical center the number and content */
}

.index-number {
  width: 48px;
  min-width: 48px;
  height: 48px;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  font-weight: 700;
  margin-right: 28px;
  font-size: 0.95rem;
  border-radius: 8px;
  /* gradient text instead of background */
  background-image: linear-gradient(90deg,#00a8ff,#00d2fc);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  box-shadow: none;
  /* shift slightly left to sit closer to the cell edge */
  transform: translateX(-6px);
}

.post-body {
  display: block;
}

.post-actions {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-left: 12px;
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
  color: #000; /* default black color for icon and number */
  margin-right: 0;
  font-size: 0.95rem;
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 2px 4px;
}

.icon-btn svg {
  display: block;
  /* outline by default: no fill, stroke uses currentColor */
  fill: none;
  stroke: currentColor;
  stroke-width: 1.2;
  stroke-linejoin: round;
  stroke-linecap: round;
}

.icon-btn .count {
  color: inherit;
}

.icon-btn.liked {
  color: #ff4d4f; /* red for liked */
}

.icon-btn.bookmarked {
  color: #f5c542; /* yellow for bookmarked */
}

/* when active, fill the icon and remove stroke */
.icon-btn.liked svg path,
.icon-btn.bookmarked svg path {
  fill: currentColor;
  stroke: none;
}

.icon-btn svg path {
  transition: fill 0.12s ease, stroke 0.12s ease, transform 0.12s ease;
}

@media (max-width: 700px) {
  .post-item {
    flex-direction: column;
    align-items: flex-start;
    gap: 6px;
  }
}
.meta-below {
  margin-top: 8px;
  color: #647382;
  font-size: 0.9rem;
}
</style>