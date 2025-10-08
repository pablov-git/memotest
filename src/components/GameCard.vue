<script setup>
import reverso from '../assets/iconos/reverso.png'

defineProps({
  card: Object
})

const emit = defineEmits(['flip-card'])

</script>

<template>
  <div class="game-card" :class="{ 'is-flipped': card.flipped, 'is-matched' : card.matched}">
    <div class="card-inner">
      <div class="card-front">
        <img :src="card.icon" alt="icono de carta" />
      </div>
      <div class="card-back">
        <img @click="emit('flip-card')" :src="reverso" alt="reverso de carta" />
      </div>
    </div>
  </div>
</template>

<style scoped>
.game-card {
  width: 80px;
  height: 80px;
  perspective: 1000px;
  display: inline-block;
  cursor: pointer;
}

/* Contenedor 3D */
.card-inner {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.6s;
}

/* Al voltear, rota el inner para mostrar el frente */
.game-card.is-flipped .card-inner {
  transform: rotateY(180deg);
}

.game-card.is-matched .card-inner {
  display: none;
}

/* Caras */
.card-front,
.card-back {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  backface-visibility: hidden;
  -webkit-backface-visibility: hidden;
  border: 2px solid #444;
  border-radius: 8px;
  background: white;
}

/* Por defecto la cara frontal está girada 180 para que, cuando inner rote, se muestre */
.card-front {
  transform: rotateY(180deg);
  z-index: 2;
}

.card-back {
  z-index: 1;
}

.game-card img {
  max-width: 90%;
  max-height: 90%;
}
</style>
