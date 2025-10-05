<script setup>
import { defineProps, defineEmits } from 'vue'

defineProps({
  difficulty: { type: String, default: 'easy' }
})

const emit = defineEmits(['changeDifficulty'])
</script>

<template>
  <div class="difficulty-selector">
    <div class="buttons">
      <button
        :class="['difficulty-btn', difficulty === 'easy' ? 'active' : '']"
        @click="emit('changeDifficulty', 'easy')"
      >
        Fácil
      </button>
      <button
        :class="['difficulty-btn', difficulty === 'normal' ? 'active' : '']"
        @click="emit('changeDifficulty', 'normal')"
      >
        Normal
      </button>
      <button
        :class="['difficulty-btn', difficulty === 'hard' ? 'active' : '']"
        @click="emit('changeDifficulty', 'hard')"
      >
        Difícil
      </button>
    </div>
  </div>
</template>

<style scoped>
.difficulty-selector {
  display: flex;
  justify-content: center;
  margin-bottom: 1.5rem;
}

.difficulty-selector .buttons {
  display: flex;
  gap: 16px;
}

.difficulty-btn {
  font-family: 'DynaPuff', cursive;
  font-weight: bold;
  font-size: 1.4rem;
  padding: 18px 30px;
  border-radius: 14px;
  border: 1px solid #444;
  background: white;
  cursor: pointer;
  transition: background-color 0.2s, color 0.2s, transform 0.2s;
}

.difficulty-btn:hover {
  background: #f0f0f0;
  transform: scale(1.05);
}

.difficulty-btn.active {
  background: var(--sea-blue);
  color: var(--mint-cream);
  border-color: var(--deep-sea-blue);
}
</style>
