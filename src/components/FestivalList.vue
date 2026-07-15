<script setup>
defineProps({
  festivals: {
    type: Array,
    required: true
  }
})
</script>

<template>
  <div class="section-head">
    <h2>부산 축제 일정</h2>
    <p>다가오는 축제와 행사를 미리 확인해보세요.</p>
  </div>

  <div class="card-grid">
    <article v-for="festival in festivals" :key="festival.title" class="card">
      <img :src="festival.image" :alt="festival.title" />
      <div class="card-body">
        <h3>{{ festival.title }}</h3>
        <p>{{ festival.address }}</p>
      </div>
    </article>
  </div>
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

.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 16px;
}

.card {
  overflow: hidden;
  border-radius: 14px;
  background: #f7fbff;
  border: 1px solid #e6eef7;
}

.card img {
  width: 100%;
  height: 160px;
  object-fit: cover;
}

.card-body {
  padding: 14px;
}

.card-body h3 {
  margin: 0 0 6px;
  font-size: 1rem;
}

.card-body p {
  margin: 0;
  color: #5f6d7a;
  font-size: 0.95rem;
}
</style>