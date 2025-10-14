<script setup>
import reverso from '../assets/iconos/reverso.png'

defineProps({
  card: Object
})

const emit = defineEmits(['flip-card'])

</script>

<template>
  <!-- Componente de una carta individual -->
  <div
    class="game-card"
    :class="{ 'is-flipped': card.flipped, 'is-matched': card.matched, 'is-fading': card.fading }"
  >
    <div class="card-inner">
      <!-- Cara frontal de la carta -->
      <div class="card-front">
        <img :src="card.icon" alt="icono de carta" />
      </div>
      <!-- Reverso de la carta -->
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
  transition: opacity 0.5s ease; /* transición para desaparecer */
}

/* Contenedor 3D */
.card-inner {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.6s;
}

/* Cuando la carta está volteada, gira el contenedor interno para mostrar la cara frontal */
.game-card.is-flipped .card-inner {
  transform: rotateY(180deg);
}

/* Cartas emparejadas desaparecen lentamente */
.game-card.is-fading {
  opacity: 0;
  pointer-events: none;
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

/* Frontal de la carta: oculta al inicio y visible cuando la carta se voltea */
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
