<script setup>
import { computed } from 'vue'
import GameCard from './GameCard.vue'

const props = defineProps({
  difficulty: { type: String, default: 'easy' }
})

// Carga todas las imágenes del folder assets/iconos
const images = import.meta.glob('../assets/iconos/*.png', { eager: true, import: 'default' })

const imageNames = [
  'almeja.png',
  'ancla.png',
  'atardecer.png',
  'bikini.png',
  'boya-salvavidas.png',
  'brocheta.png',
  'camara-fotografica.png',
  'cangrejo.png',
  'caracol-de-mar.png',
  'castillo-de-arena.png',
  'chalecos-salvavidas.png',
  'chancletas.png',
  'chiringuito.png',
  'clima-caliente.png',
  'coco.png',
  'coctel-rojo.png',
  'coctel-verde.png',
  'cubo-de-arena.png',
  'cucurucho-de-helado.png',
  'estrella-de-mar.png',
  'faro.png',
  'furgoneta-de-surf.png',
  'gafas-de-sol.png',
  'hamaca.png',
  'hoguera.png',
  'huella.png',
  'lata-de-refresco.png',
  'mascara-de-buceo.png',
  'onda.png',
  'paleta-de-hielo.png',
  'pantalones-cortos.png',
  'pelota-de-playa.png',
  'pescado.png',
  'pina.png',
  'playa-hamaca.png',
  'playa-palmera.png',
  'playa.png',
  'protector-solar.png',
  'radio.png',
  'sandia.png',
  'sol.png',
  'sombrero-para-el-sol.png',
  'sombrilla.png',
  'tabla-de-surf.png',
  'toalla-de-playa.png',
  'tortuga.png',
  'velero.png',
  'verderon.png',
  'voleibol.png',
  'windsurf.png',
]

// Recorre imageNames y genera un objeto carta por cada imagen, con id, icono y estados iniciales (flipped y matched)
const cards = imageNames.map((name, index) => ({
  id: index + 1,
  icon: images[`../assets/iconos/${name}`],
  flipped: false,
  matched: false,
}))

// Número de cartas
const difficultyMap = { easy: 8, normal: 15, hard: 24 }

// Filtra cartas según dificultad
const filteredCards = computed (() => cards.slice(0, difficultyMap[props.difficulty]))

// Duplica cartas y las coloca de manera aleatoria con una llamada a shuffle
const filteredCardsAll = computed(() => {
  const doubled = [...filteredCards.value, ...filteredCards.value]
  return shuffle([...doubled])
})

// Coloca las cartas de manera aleatoria
function shuffle(array) {
  let currentIndex = array.length
  let shuffled = [...array]

  while (currentIndex != 0) {
    let randomIndex = Math.floor(Math.random() * currentIndex)
    currentIndex--
    ;[shuffled[currentIndex], shuffled[randomIndex]] = [
      shuffled[randomIndex], shuffled[currentIndex]
    ]
  }

  return shuffled
}

// Calcula cuántas columnas necesita el grid para que sea cuadrado (hace la raíz cuadrada y redondea hacia arriba)
const gridCols = computed(() => Math.ceil(Math.sqrt(filteredCardsAll.value.length)))



function startGame() {
  console.log("Juego iniciado");
}
</script>

<template>
  <section class="game-board">
    <div
      class="board-grid"
      role="grid"
      :style="{ gridTemplateColumns: `repeat(${gridCols}, 80px)` }"
    >
      <GameCard v-for="(card, idx) in filteredCardsAll" :key="card.id + Math.random()" :card="card" :index="idx" />
      <button class="play-btn" @click="startGame">▶</button>
    </div>
  </section>
</template>

<style scoped>
.game-board {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.board-grid {
  display: grid;
  column-gap: 1.6rem;
  row-gap: 1.6rem;
  justify-content: center;
}

.play-btn {
  padding: 12px 24px;
  font-size: 1.2rem;
  font-weight: bold;
  color: white;
  background-color: var(--sea-blue);
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.2s, transform 0.1s;
}

.play-btn:hover {
  background-color: var(--deep-sea-blue);
}

.play-btn:active {
  transform: scale(0.95);
}
</style>
