<script setup>
import { ref, computed, watch, onBeforeUnmount } from 'vue'
import GameCard from './GameCard.vue'

// Props del componente: solo define la dificultad del juego
const props = defineProps({ difficulty: { type: String, default: 'easy' } })

// Importa automáticamente todas las imágenes PNG dentro de la carpeta "assets/iconos"
const images = import.meta.glob('../assets/iconos/*.png', { eager: true, import: 'default' })

// Nombres de los iconos disponibles (cada uno corresponde a un archivo en /assets/iconos)
const imageNames = [
  'almeja.png','ancla.png','atardecer.png','bikini.png','boya-salvavidas.png','brocheta.png','camara-fotografica.png',
  'cangrejo.png','caracol-de-mar.png','castillo-de-arena.png','chalecos-salvavidas.png','chancletas.png','chiringuito.png',
  'clima-caliente.png','coco.png','coctel-rojo.png','coctel-verde.png','cubo-de-arena.png','cucurucho-de-helado.png',
  'estrella-de-mar.png','faro.png','furgoneta-de-surf.png','gafas-de-sol.png','hamaca.png','hoguera.png','huella.png',
  'lata-de-refresco.png','mascara-de-buceo.png','onda.png','paleta-de-hielo.png','pantalones-cortos.png','pelota-de-playa.png',
  'pescado.png','pina.png','playa-hamaca.png','playa-palmera.png','playa.png','protector-solar.png','radio.png','sandia.png',
  'sol.png','sombrero-para-el-sol.png','sombrilla.png','tabla-de-surf.png','toalla-de-playa.png','tortuga.png','velero.png',
  'verderon.png','voleibol.png','windsurf.png',
]

// Crea un array reactivo con todas las cartas base (una por icono)
const cards = ref(imageNames.map((n, i) => ({
  id: i + 1,
  icon: images[`../assets/iconos/${n}`],
  flipped: false,
  matched: false,
})))

console.log(cards.value)

const flippedCards = ref([])

// Define cuántas cartas usar según la dificultad elegida
const difficultyMap = { easy: 8, normal: 15, hard: 28 }

// Array con las cartas visibles en el tablero (duplicadas y mezcladas)
const boardCards = ref([])

// Función auxiliar para mezclar aleatoriamente un array
const shuffle = arr => arr.sort(() => Math.random() - 0.5)

// Genera el tablero duplicando las cartas seleccionadas y mezclándolas
function buildBoard() {
  const count = difficultyMap[props.difficulty]
  // Selecciona solo las cartas necesarias según la dificultad
  const selected = cards.value.slice(0, count)

  // Crea dos copias independientes de cada carta y mezcla el resultado
  boardCards.value = shuffle(
    selected.flatMap((c, i) => [
      { ...c, instanceId: `${c.id}-a-${i}`, flipped: false, matched: false },
      { ...c, instanceId: `${c.id}-b-${i}`, flipped: false, matched: false },
    ])
  )

console.log(boardCards.value)

  // Resetea el estado de juego
  gamePhase.value = 'idle'
  countdown.value = 0
}

function flipCard(instanceId){
  console.log("Se gira la carta " + instanceId)
  const card = boardCards.value.find(c => c.instanceId === instanceId)
  card.flipped = !card.flipped
  console.log(card)
  flippedCards.value.push(card)
  if (flippedCards.value.length === 2) {
    if(flippedCards.value[0].id == flippedCards.value[1].id){
      boardCards.value.find(c => c.instanceId == flippedCards.value[0].instanceId).matched = true;
      boardCards.value.find(c => c.instanceId == flippedCards.value[1].instanceId).matched = true;
      flippedCards.value = [];
    }else{
      setTimeout(() => {
        boardCards.value.find(c => c.instanceId == flippedCards.value[0].instanceId).flipped = false;
        boardCards.value.find(c => c.instanceId == flippedCards.value[1].instanceId).flipped = false;
        flippedCards.value = [];
      }, 1000);
    }
  }
  console.log(flippedCards.value)
}

// --- Variables de control del juego ---
// Estas se colocan aquí, cerca de las funciones que las usan
const gridCols = computed(() => Math.ceil(Math.sqrt(boardCards.value.length)))
const errorCount = ref(0)
const gamePhase = ref('idle')
const countdown = ref(0)
let timerId = null

// Limpia el temporizador si hay uno activo
const clearTimer = () => (timerId && clearInterval(timerId), (timerId = null))

// Inicia el juego
function startGame() {
  clearTimer()
  errorCount.value = 0

  // Reinicia todas las cartas a su estado inicial (boca abajo y sin emparejar)
  boardCards.value.forEach(c => Object.assign(c, { flipped: false, matched: false }))

  // Fase 1: cuenta atrás de preparación (3 segundos)
  gamePhase.value = 'revealing'
  countdown.value = 3

  timerId = setInterval(() => {
    if (--countdown.value <= 0) {
      clearTimer()

      // Fase 2: muestra las cartas durante 5 segundos para memorizar
      boardCards.value.forEach(c => (c.flipped = true))
      gamePhase.value = 'showing'
      countdown.value = 5

      timerId = setInterval(() => {
        if (--countdown.value <= 0) {
          clearTimer()

          // Fase 3: oculta las cartas y comienza el juego real
          boardCards.value.forEach(c => (c.flipped = false))
          gamePhase.value = 'playing'
        }
      }, 1000)
    }
  }, 1000)
}

// Cuando cambia la dificultad, se vuelve a generar el tablero
watch(() => props.difficulty, buildBoard)

// Genera el tablero al montar el componente
buildBoard()

// Limpia los temporizadores al desmontar el componente
onBeforeUnmount(clearTimer)
</script>

<template>
  <section class="game-board">
    <div class="play-container">
      <div
        v-if="(gamePhase === 'revealing' || gamePhase === 'showing') && countdown > 0"
        class="countdown-overlay"
      >
        <div class="countdown-circle">
          <div class="countdown-number">{{ countdown }}</div>
          <div class="countdown-text">
            {{ gamePhase === 'revealing' ? 'Preparando...' : 'Memoriza!' }}
          </div>
        </div>
      </div>

      <button class="play-btn" @click="startGame">▶</button>
    </div>

    <div
      class="board-grid"
      role="grid"
      :style="{ gridTemplateColumns: `repeat(${gridCols}, 80px)` }"
    >
      <GameCard v-for="c in boardCards" @flip-card="flipCard(c.instanceId)" :key="c.instanceId" :card="c" />
    </div>

    <div class="error-counter">Errores: {{ errorCount }}</div>
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

.play-container {
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
  margin-top: 3rem;
  margin-bottom: 3rem;
}

.play-btn {
  padding: 18px 42px;
  font-size: 2rem;
  font-weight: bold;
  color: white;
  background-color: var(--sea-blue);
  border: none;
  border-radius: 12px;
  cursor: pointer;
  transition:
    background-color 0.2s,
    transform 0.1s;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
}

.play-btn:hover {
  background-color: var(--deep-sea-blue);
  transform: scale(1.05);
}

.play-btn:active {
  transform: scale(0.95);
}

.error-counter {
  margin-top: 1.5rem;
  text-align: center;
  font-size: 1.5rem;
  font-weight: bold;
  color: var(--deep-sea-blue);
  margin-bottom: 7rem;
}

/* Estilos del contador visual (cuenta regresiva) */
.countdown-overlay {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -120%);
  z-index: 5;
  display: flex;
  justify-content: center;
  align-items: center;
}

.countdown-circle {
  width: 110px;
  height: 110px;
  border-radius: 999px;
  background: rgba(0, 0, 0, 0.75);
  color: white;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.4);
}

.countdown-number {
  font-size: 3rem;
  line-height: 1;
}

.countdown-text {
  font-size: 1rem;
  opacity: 0.9;
}
</style>
