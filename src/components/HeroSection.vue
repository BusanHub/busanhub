<script setup>
defineProps({
  regionName: {
    type: String,
    required: true
  },
  stats: {
    type: Object,
    required: true
  }
})
</script>

<template>
  <header class="hero">
    <div class="hero-text">
      <h1>BusanHub</h1>
      <p>부산의 모든 정보와 익명 커뮤니티를 한 번에 만나보세요.</p>
    </div>

    <div class="hero-stats">
      <div>
        <strong>{{ stats.attractions }}</strong>
        <span>관광지</span>
      </div>
      <div>
        <strong>{{ stats.festivals }}</strong>
        <span>축제</span>
      </div>
      <div>
        <strong>{{ stats.posts }}</strong>
        <span>게시글</span>
      </div>
    </div>
  </header>
</template>

<style scoped>
.hero {
  background: linear-gradient(135deg, #0d4c92, #2b7fff);
  color: white;
  padding: 48px 24px;
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  align-items: center;
  gap: 24px;
}

.eyebrow {
  text-transform: uppercase;
  letter-spacing: 0.2em;
  opacity: 0.8;
  margin-bottom: 8px;
}

.hero h1 {
  margin: 0 0 8px;
  font-size: 2.2rem;
}

.hero-stats {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
}

.hero-stats > div {
  background: rgba(255,255,255,0.16);
  padding: 12px 16px;
  border-radius: 12px;
  min-width: 90px;
  text-align: center;
}

.hero-stats strong {
  display: block;
  font-size: 1.2rem;
}
</style>