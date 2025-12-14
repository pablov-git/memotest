<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import GameBoard from './components/GameBoard.vue'
import DifficultySelector from './components/DifficultySelector.vue'

const difficulty = ref('')
const isMobile = ref(false)

// Detectar si es móvil
function checkMobile() {
  isMobile.value = window.innerWidth < 768
  // En móvil, establecer directamente dificultad fácil
  if (isMobile.value && difficulty.value === '') {
    difficulty.value = 'easy'
  }
}

onMounted(() => {
  checkMobile()
  window.addEventListener('resize', checkMobile)
})

onUnmounted(() => {
  window.removeEventListener('resize', checkMobile)
})

// Cambiar dificultad desde selector
function changeDifficulty(level) {
  difficulty.value = level
}
</script>

<template>
  <h2>Memotest</h2>
  <DifficultySelector
    v-if="difficulty == '' && !isMobile"
    :difficulty="difficulty"
    @changeDifficulty="changeDifficulty"
  />
  <GameBoard
    v-if="difficulty !== ''"
    @changeDifficulty="changeDifficulty"
    :difficulty="difficulty"
    :is-mobile="isMobile"
  />
</template>

<style scoped>
h2 {
  font-family: 'DynaPuff', cursive;
  font-weight: bold;
  font-size: 6rem;
  text-align: center;
  margin-bottom: 1rem;
  color: var(--sea-blue);
}

@media (max-width: 767px) {
  h2 {
    font-size: 3rem;
    margin-bottom: 0.5rem;
  }
}
</style>
