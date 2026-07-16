<script setup>
import { computed, onMounted, ref, watch } from 'vue'
import tourismData from './data/부산/부산_관광지.json'
import festivalData from './data/부산/부산_축제공연행사.json'
import leisureData from './data/부산/부산_레포츠.json'
import cultureData from './data/부산/부산_문화시설.json'
import shoppingData from './data/부산/부산_쇼핑.json'
import lodgingData from './data/부산/부산_숙박.json'
import courseData from './data/부산/부산_여행코스.json'

import HeroSection from './components/HeroSection.vue'
import TouristList from './components/TouristList.vue'
import FestivalList from './components/FestivalList.vue'
import CommunityBoard from './components/CommunityBoard.vue'
import ChatbotPanel from './components/ChatbotPanel.vue'

const regionName = '부산'

const attractions = computed(() =>
  tourismData.items.slice(0, 8).map((item) => ({
    title: item.title,
    address: item.addr1 || '주소 정보 없음',
    image: item.firstimage || 'https://images.unsplash.com/photo-1507525428034-b723cf961d3e?auto=format&fit=crop&w=800&q=80'
  }))
)

const festivals = computed(() =>
  festivalData.items.slice(0, 6).map((item) => ({
    title: item.title,
    address: item.addr1 || '주소 정보 없음',
    image: item.firstimage || 'https://images.unsplash.com/photo-1499856871958-2b0f9b4e8a2f?auto=format&fit=crop&w=800&q=80'
  }))
)

const posts = ref([])
const form = ref({
  title: '',
  content: '',
  password: '',
  editingId: null,
  _isDelete: false
})

const boardMode = ref('list')
const selectedPostId = ref(null)
const boardMessage = ref('')
const chatOpen = ref(false)
const chatMessages = ref([
  {
    role: 'assistant',
    content: '안녕하세요! 부산 여행 도우미입니다. 관광지, 축제, 커뮤니티 게시글에 대해 질문해보세요.'
  }
])
const chatDraft = ref('')

const stats = computed(() => ({
  attractions: attractions.value.length,
  festivals: festivals.value.length,
  posts: posts.value.length
}))

const selectedPost = computed(() =>
  posts.value.find((post) => post.id === selectedPostId.value) || null
)

function loadPosts() {
  try {
    const saved = localStorage.getItem('localhub-posts')
    const raw = saved ? JSON.parse(saved) : []
    // ensure posts have views, likes and bookmark fields
    posts.value = raw.map((p) => {
      // helper to robustly parse stored timestamps (number, numeric-string, ISO string)
      const parseTimestamp = (val, fallback = null) => {
        if (val == null) return fallback
        if (typeof val === 'number') return val
        // numeric strings ("162...") -> Number
        const asNum = Number(val)
        if (!Number.isNaN(asNum) && Number.isFinite(asNum)) return asNum
        // date strings -> Date.parse
        const parsed = Date.parse(val)
        return Number.isNaN(parsed) ? fallback : parsed
      }

      const createdAt = parseTimestamp(p.createdAt, Date.now())
      const updatedAt = parseTimestamp(p.updatedAt, null)

      return {
        views: p.views || 0,
        likes: p.likes || 0,
        liked: p.liked || false,
        bookmarks: p.bookmarks || 0,
        bookmarked: p.bookmarked || false,
        ...p,
        // preserve existing timestamps where possible; fallback only when missing/invalid
        createdAt,
        updatedAt
      }
    })
  } catch {
    posts.value = []
  }
}

function savePosts() {
  localStorage.setItem('localhub-posts', JSON.stringify(posts.value))
}

function resetForm() {
  form.value = {
    title: '',
    content: '',
    password: '',
    editingId: null,
    _isDelete: false
  }
  boardMessage.value = ''
}

function openCreateForm() {
  resetForm()
  boardMode.value = 'form'
  selectedPostId.value = null
}

function cancelForm() {
  resetForm()
  boardMode.value = 'list'
}

function submitPost() {
  if (!form.value.title.trim() || !form.value.content.trim() || !form.value.password.trim()) {
    boardMessage.value = '제목, 내용, 비밀번호를 모두 입력해주세요.'
    return
  }
  // deletion flow
  if (form.value._isDelete && form.value.editingId) {
    const original = posts.value.find((p) => p.id === form.value.editingId)
    if (!original) {
      boardMessage.value = '원본 게시글을 찾을 수 없습니다.'
      return
    }

    if (form.value.password.trim() !== original.password) {
      boardMessage.value = '비밀번호가 일치하지 않습니다.'
      return
    }

    posts.value = posts.value.filter((item) => item.id !== form.value.editingId)
    if (selectedPostId.value === form.value.editingId) selectedPostId.value = null
    boardMessage.value = '게시글이 삭제되었습니다.'
    resetForm()
    boardMode.value = 'list'
    return
  }
  if (form.value.editingId) {
    const original = posts.value.find((p) => p.id === form.value.editingId)
    if (!original) {
      boardMessage.value = '원본 게시글을 찾을 수 없습니다.'
      return
    }

    if (form.value.password.trim() !== original.password) {
      boardMessage.value = '비밀번호가 일치하지 않습니다.'
      return
    }

    posts.value = posts.value.map((post) =>
      post.id === form.value.editingId
        ? {
            ...post,
            title: form.value.title.trim(),
            content: form.value.content.trim(),
            updatedAt: Date.now()
          }
        : post
    )
    boardMessage.value = '게시글이 수정되었습니다.'
  } else {
    posts.value.unshift({
      id: Date.now().toString(),
      title: form.value.title.trim(),
      content: form.value.content.trim(),
      password: form.value.password.trim(),
      views: 0,
      likes: 0,
      liked: false,
      bookmarks: 0,
      bookmarked: false,
      createdAt: Date.now(),
      updatedAt: null
    })
    boardMessage.value = '게시글이 등록되었습니다.'
  }

  resetForm()
  boardMode.value = 'list'
}

function openPost(post) {
  const idx = posts.value.findIndex((p) => p.id === post.id)
  if (idx !== -1) {
    posts.value[idx].views = (posts.value[idx].views || 0) + 1
  }
  selectedPostId.value = post.id
  boardMode.value = 'detail'
}

function backToList() {
  selectedPostId.value = null
  boardMode.value = 'list'
}

function editPost(post) {
  form.value = {
    title: post.title,
    content: post.content,
    password: '',
    editingId: post.id,
    _isDelete: false
  }
  boardMessage.value = ''
  boardMode.value = 'form'
}

function deletePost(postId) {
  const post = posts.value.find((item) => item.id === postId)
  if (!post) return
  // open the form in delete mode so user can enter password and confirm
  form.value = {
    title: post.title,
    content: post.content,
    password: '',
    editingId: postId,
    _isDelete: true
  }
  boardMessage.value = '삭제하려면 비밀번호를 입력한 뒤 삭제 버튼을 누르세요.'
  boardMode.value = 'form'
}

function updateFormField(field, value) {
  form.value[field] = value
}

function toggleLike(postId) {
  const post = posts.value.find((p) => p.id === postId)
  if (!post) return
  if (post.liked) {
    post.likes = Math.max(0, (post.likes || 0) - 1)
    post.liked = false
  } else {
    post.likes = (post.likes || 0) + 1
    post.liked = true
  }
}

function toggleBookmark(postId) {
  const post = posts.value.find((p) => p.id === postId)
  if (!post) return
  if (post.bookmarked) {
    post.bookmarks = Math.max(0, (post.bookmarks || 0) - 1)
    post.bookmarked = false
  } else {
    post.bookmarks = (post.bookmarks || 0) + 1
    post.bookmarked = true
  }
}

watch(posts, savePosts, { deep: true })

onMounted(() => {
  loadPosts()
})

function toggleChat() {
  chatOpen.value = !chatOpen.value
}

async function sendChatMessage() {
  const text = chatDraft.value.trim()
  if (!text) return

  chatMessages.value.push({ role: 'user', content: text })
  chatDraft.value = ''

  const reply = await getBotReply(text)
  chatMessages.value.push({ role: 'assistant', content: reply })
}

async function getBotReply(text) {
  const apiKey = import.meta.env.VITE_OPENAI_API_KEY

  if (apiKey) {
    try {
      const context = `
부산 지역 정보 요약:
- 관광지: ${attractions.value.slice(0, 5).map((item) => item.title).join(', ')}
- 축제: ${festivals.value.slice(0, 5).map((item) => item.title).join(', ')}
- 레포츠: ${leisureData.items.slice(0, 5).map((item) => item.title).join(', ')}
- 문화시설: ${cultureData.items.slice(0, 5).map((item) => item.title).join(', ')}
- 쇼핑: ${shoppingData.items.slice(0, 5).map((item) => item.title).join(', ')}
- 숙박: ${lodgingData.items.slice(0, 5).map((item) => item.title).join(', ')}
- 여행코스: ${courseData.items.slice(0, 5).map((item) => item.title).join(', ')}
- 커뮤니티 게시글: ${posts.value.slice(0, 5).map((post) => post.title).join(', ')}
`

      const response = await fetch('https://api.openai.com/v1/chat/completions', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          Authorization: `Bearer ${apiKey}`
        },
        body: JSON.stringify({
          model: 'gpt-5-mini',
          temperature: 1,
          messages: [
            {
              role: 'system',
              content: '당신은 부산 지역 정보와 익명 커뮤니티를 돕는 친절한 여행 도우미입니다. 제공된 지역 데이터를 바탕으로 짧고 정확하게 답하세요.'
            },
            {
              role: 'user',
              content: `${context}\n사용자 질문: ${text}`
            }
          ]
        })
      })

      const data = await response.json()
      const rawReply = data?.choices?.[0]?.message?.content?.trim() ?? ''
      const reply = formatBotReply(rawReply || fallbackReply(text))

      if (reply) return reply
    } catch {
      // fallback
    }
  }

  return fallbackReply(text)
}

function formatBotReply(reply) {
  return reply
    .replace(/\r/g, '')
    .replace(/\n{3,}/g, '\n\n')
    .trim()
}

function fallbackReply(text) {
  const lower = text.toLowerCase()

  if (lower.includes('축제')) {
    return `부산의 축제 추천으로는 ${festivals.value[0]?.title || '부산국제불교박람회'}를 먼저 살펴보시면 좋아요.`
  }

  if (lower.includes('관광') || lower.includes('추천')) {
    return `추천 관광지는 ${attractions.value[0]?.title || '해운대'}와 ${attractions.value[1]?.title || '광안리'}를 고려해보세요.`
  }

  if (lower.includes('게시글') || lower.includes('커뮤니티')) {
    return '커뮤니티 게시판에서는 제목, 내용, 비밀번호를 입력해 익명 글을 작성하고 수정·삭제할 수 있습니다.'
  }

  if (lower.includes('맛') || lower.includes('음식')) {
    return '부산 맛집 관련은 현재 샘플 데이터 기준으로 관광지와 축제 중심으로 안내하고 있습니다.'
  }

  return '부산 관광지, 축제, 커뮤니티 게시글에 대해 질문해 주세요.'
}
</script>

<template>
  <div class="page">
    <HeroSection :region-name="regionName" :stats="stats" />

    <main class="content">
      <section class="panel">
        <TouristList :attractions="attractions" />
      </section>

      <section class="panel">
        <FestivalList :festivals="festivals" />
      </section>

      <section class="panel">
        <CommunityBoard
          :posts="posts"
          :board-message="boardMessage"
          :board-mode="boardMode"
          :selected-post="selectedPost"
          :form="form"
          @open-create="openCreateForm"
          @submit-post="submitPost"
          @cancel-form="cancelForm"
          @open-post="openPost"
          @back-list="backToList"
          @edit-post="editPost"
          @delete-post="deletePost"
            @update-field="updateFormField"
            @toggle-like="toggleLike"
            @toggle-bookmark="toggleBookmark"
        />
      </section>
    </main>

    <ChatbotPanel
      :messages="chatMessages"
      :draft="chatDraft"
      :is-open="chatOpen"
      @toggle-chat="toggleChat"
      @update-draft="chatDraft = $event"
      @send-message="sendChatMessage"
    />
  </div>
</template>

<style scoped>
.page {
  min-height: 100vh;
  background: #f4f8fb;
  color: #243447;
}

.content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 24px;
  display: grid;
  gap: 24px;
}

.panel {
  background: white;
  border-radius: 18px;
  padding: 24px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.05);
}
</style>