<script setup>
import { ref } from 'vue'
import GameBoard from './components/GameBoard.vue'
import DifficultySelector from './components/DifficultySelector.vue'

const difficulty = ref('')

// Cambiar dificultad desde selector
function changeDifficulty(level) {
  difficulty.value = level
}
</script>

<template>
  <h2>Memotest</h2>
  <DifficultySelector v-if="difficulty==''" :difficulty="difficulty" @changeDifficulty="changeDifficulty" />
  <GameBoard @changeDifficulty="changeDifficulty" v-else :difficulty="difficulty" />
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
</style>
