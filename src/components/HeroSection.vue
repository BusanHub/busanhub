<script setup>
defineProps({
  regionName: {
    type: String,
    required: true
  },
  stats: {
    type: Object,
    required: true
  },
  navItems: {
    type: Array,
    default: () => []
  }
})
</script>

<template>
  <header class="hero">
    <div class="hero-text">
      <h1>BusanHub</h1>
      <p>부산의 모든 정보와 익명 커뮤니티를 한 번에 만나보세요.</p>
    </div>

    <div v-if="navItems.length" class="hero-nav">
      <a
        v-for="item in navItems"
        :key="item.id"
        :href="`#${item.id}`"
        class="nav-chip"
      >
        {{ item.label }}
      </a>
    </div>
  </header>
</template>

<style scoped>
.hero {
  position: sticky;
  top: 0;
  z-index: 1000;
  background: linear-gradient(135deg, #0d4c92, #2b7fff);
  color: white;
  padding: 24px 24px 20px;
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  align-items: center;
  gap: 20px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.12);
}

.hero-text {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

.hero h1 {
  margin: 0 0 8px;
  font-size: 2.2rem;
}

.hero-nav {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-left: auto;
}

.nav-chip {
  text-decoration: none;
  color: white;
  background: rgba(255, 255, 255, 0.16);
  border: 1px solid rgba(255, 255, 255, 0.24);
  padding: 8px 12px;
  border-radius: 999px;
  font-size: 0.92rem;
}

.nav-chip:hover {
  background: rgba(255, 255, 255, 0.28);
}
</style>