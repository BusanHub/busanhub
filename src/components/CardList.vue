<script setup>
defineProps({
  title: {
    type: String,
    required: true,
  },
  description: {
    type: String,
    default: "",
  },
  items: {
    type: Array,
    default: () => [],
  },
});
</script>


<template>
  <div>
    <div class="section-head">
      <h2>{{ title }}</h2>
      <p>{{ description }}</p>
    </div>

    <div class="card-grid">
      <article
        v-for="item in items"
        :key="item.title"
        class="card"
      >
        <img :src="item.image" :alt="item.title" />

        <div class="card-body">
          <h3>{{ item.title }}</h3>
          <p>{{ item.address }}</p>
        </div>
      </article>
    </div>
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