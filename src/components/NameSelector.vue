<script setup>
import { ref } from 'vue'

const name = ref('')
const emit = defineEmits(['start-game', 'close-modal'])

function submit() {
  const trimmed = name.value.trim()
  if (!trimmed) {
    return
  }
  emit('start-game', trimmed)
}

function closeModal() {
  emit('close-modal')
}

function handleBackdropClick(event) {
  if (event.target.classList.contains('modal-backdrop')) {
    closeModal()
  }
}
</script>

<template>
  <div class="modal-backdrop" @click="handleBackdropClick">
    <div class="modal-name">
      <button class="close-btn" @click="closeModal">✕</button>
      <input
        class="name-input"
        v-model="name"
        type="text"
        placeholder="Introduce tu nombre"
        @keyup.enter="submit"
      />
      <button @click="submit">Comenzar</button>
    </div>
  </div>
</template>

<style scoped>
.modal-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 99;
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-name {
  position: relative;
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

.close-btn {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: transparent;
  border: none;
  font-size: 1.5rem;
  color: #999;
  cursor: pointer;
  padding: 0.2rem 0.5rem;
  line-height: 1;
  transition: color 0.2s ease;
}

.close-btn:hover {
  color: #333;
}

.name-input {
  margin-top: 1.4rem;
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

.modal-name button:not(.close-btn) {
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

.modal-name button:not(.close-btn):hover {
  background-color: var(--deep-sea-blue);
  transform: scale(1.05);
}

.modal-name button:not(.close-btn):active {
  transform: scale(0.97);
}

@media (max-width: 767px) {
  .modal-name {
    width: 85%;
    max-width: 300px;
    padding: 2rem 1.5rem;
    gap: 1rem;
  }

  .close-btn {
    top: 0.8rem;
    right: 0.8rem;
    font-size: 1.3rem;
  }

  .modal-name input {
    font-size: 1rem;
    padding: 0.7rem 0.8rem;
  }

  .modal-name button:not(.close-btn) {
    padding: 0.8rem 1.5rem;
    font-size: 1.1rem;
  }
}
</style>
