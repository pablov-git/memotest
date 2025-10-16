<script setup></script>

<template>
  <div class="modal-name">
    <input type="text" placeholder="Introduce tu nombre" />
    <button>Comenzar</button>
  </div>
</template>

<style scoped>
.modal-name {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -120%);
  z-index: 6;
  display: flex;
  flex-direction: column;
  gap: 10px;
  justify-content: center;
  align-items: center;
  width: 600px;
  height: 400px;
  background-color: #f5f5f5;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
}
</style>
