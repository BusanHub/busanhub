<script setup>
import { ref } from 'vue'

const emit = defineEmits(['update-search'])
const search = ref('')

function onInput(e) {
  search.value = e.target.value
  emit('update-search', search.value)
}
</script>

<template>
  <div class="community-header">
    <div class="title">소통마당</div>
    <input class="search-input" :value="search" @input="onInput" placeholder="제목, 내용 검색..." />
  </div>
</template>

<style scoped>
.community-header{display:flex;flex-direction:column;gap:8px;margin:18px 0}
.title{font-size:1.6rem;color:#0d3b66;font-weight:700}
.search-input{width:100%;padding:10px 14px;border-radius:24px;border:1px solid #e6eef8;background:white}
@media (max-width:700px){.search-input{width:100%}}
</style>
