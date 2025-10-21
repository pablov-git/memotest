<script setup>
import { ref } from 'vue'
const name = ref('')
const emit = defineEmits(['start-game'])

function submit() {
  const trimmed = name.value.trim()
  if (!trimmed){
    return
  }
  emit('start-game', trimmed)
}
</script>

<template>
  <div class="modal-name">
    <input v-model="name" type="text" placeholder="Introduce tu nombre" @keyup.enter="submit" />
    <button @click="submit">Comenzar</button>
  </div>
</template>

<style scoped>
.modal-name {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 100;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 1.2rem;
  width: 400px;
  padding: 2.5rem;
  background: #ffffff;
  border-radius: 16px;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.25);
  font-family: 'DynaPuff', cursive;
  animation: popIn 0.3s ease-out;
}

.modal-name input {
  width: 100%;
  padding: 0.8rem 1rem;
  font-size: 1.1rem;
  border: 2px solid #ddd;
  border-radius: 10px;
  outline: none;
  transition: all 0.2s ease;
  text-align: center;
}

.modal-name input:focus {
  border-color: var(--sea-blue);
  box-shadow: 0 0 6px rgba(0, 102, 255, 0.2);
}

.modal-name button {
  background-color: var(--sea-blue);
  color: #fff;
  border: none;
  border-radius: 10px;
  padding: 0.9rem 2rem;
  font-size: 1.2rem;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.2s ease;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
}

.modal-name button:hover {
  background-color: var(--deep-sea-blue);
  transform: scale(1.05);
}

.modal-name button:active {
  transform: scale(0.97);
}
</style>
