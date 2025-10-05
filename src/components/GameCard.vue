<script setup>
defineProps({
  card: Object
})
</script>

<template>
  <div class="game-card">
    <img :src="card.icon" alt="icono de carta" />
  </div>
</template>

<style scoped>
.game-card {
  width: 80px;
  height: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 2px solid #444;
  border-radius: 8px;
  background: white;
  cursor: pointer;
}

.game-card img {
  max-width: 100%;
  max-height: 100%;
}
</style>
