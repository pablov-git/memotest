<script setup>
import { ref, computed, watch, onBeforeUnmount } from 'vue'
import GameCard from './GameCard.vue'
import NameSelector from './NameSelector.vue'
import { DateTime } from 'luxon'
import UserRanking from './UserRanking.vue'
import confetti from 'canvas-confetti'

const showRanking = ref(false)
const startTime = ref(null) // Momento en que empieza el juego
const elapsedTime = ref('00:00') // Tiempo transcurrido en formato mm:ss
let timerId = null // Para el setInterval

const flipSound = new Audio('/sounds/cardFlip.ogg')
const correctSound = new Audio('/sounds/correct.ogg')
const victorySound = new Audio('/sounds/victory.ogg')
const failureSound = new Audio('/sounds/failure.ogg')
const errorSound = new Audio('/sounds/error.ogg')
const confettiSound = new Audio('/sounds/confetti.wav')

const props = defineProps({ difficulty: { type: String, default: 'easy' } })

const emit = defineEmits(['changeDifficulty'])

const modal = ref(false)
const playerName = ref('')

// Importa automáticamente todas las imágenes
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

// Crea un array reactivo con todas las cartas base (una por icono)
const cards = ref(
  imageNames.map((n, i) => ({
    id: i + 1,
    icon: images[`../assets/iconos/${n}`],
    flipped: false,
    matched: false,
    fading: false,
  })),
)

console.log(cards.value)

const flippedCards = ref([])

// Define cuántas cartas usar según la dificultad elegida
const difficultyMap = { easy: 8, normal: 15, hard: 28 }

// Array con las cartas visibles en el tablero (duplicadas y mezcladas)
const boardCards = ref([])

// Función auxiliar para mezclar aleatoriamente un array
const shuffle = (arr) => arr.sort(() => Math.random() - 0.5)

function playFlipSound() {
  flipSound.currentTime = 0 // reinicia por si se reproduce rápido varias veces
  flipSound.play().catch(() => {}) // evita errores si el usuario no ha interactuado aún
}

// Funciones para la reproducción de sonidos //
function playCorrectSound() {
  // Reinicia el sonido por si se reproduce varias veces seguidas
  correctSound.currentTime = 0
  // Reproduce el sonido, evitando errores si el usuario aún no ha interactuado con la página
  correctSound.play().catch(() => {})
}

function playErrorSound() {
  errorSound.currentTime = 0
  errorSound.play().catch(() => {})
}

function playVictorySound() {
  victorySound.currentTime = 0
  victorySound.play().catch(() => {})
}

function playFailureSound() {
  failureSound.currentTime = 0
  failureSound.play().catch(() => {})
}

function playConfettiSound() {
  confettiSound.currentTime = 0
  confettiSound.play().catch(() => {})
}

// --- CONFETTI ---
function launchConfetti() {
  const duration = 2000 // duración total de la animación en milisegundos
  const end = Date.now() + duration // momento exacto en que debe terminar la animación

  // Colores que se usarán para las partículas de confeti
  const colors = ['#00c3ff', '#ffcb00', '#ff4081', '#4caf50']

  // Función autoinvocada que genera un "frame" de confeti
  ;(function frame() {
    // Estallido de confeti desde la izquierda
    confetti({
      particleCount: 5, // número de partículas
      angle: 60, // dirección de lanzamiento
      spread: 55, // dispersión de partículas
      origin: { x: 0 }, // punto de origen en el borde izquierdo
      colors, // colores de las partículas
    })

    // Estallido de confeti desde la derecha
    confetti({
      particleCount: 5,
      angle: 120,
      spread: 55,
      origin: { x: 1 }, // punto de origen en el borde derecho
      colors,
    })

    // Si aún no ha pasado el tiempo total, pedimos el siguiente frame
    if (Date.now() < end) {
      requestAnimationFrame(frame) // llama a frame de nuevo para animación continua
    }
  })() // la función se ejecuta inmediatamente para iniciar la animación
}

function openNameModal() {
  modal.value = true
}

function handleStartFromModal(name) {
  // opcional: guardar el nombre si más adelante quieres usarlo
  if (name) playerName.value = name
  modal.value = false
  startGame()
}

function changeDifficultyAndReset() {
  showRanking.value = false
  emit('changeDifficulty', '')
  clearTimer()
  elapsedTime.value = '00:00'
}

// Genera el tablero duplicando las cartas seleccionadas y mezclándolas
function buildBoard() {
  const count = difficultyMap[props.difficulty]
  // Selecciona solo las cartas necesarias según la dificultad
  const selected = cards.value.slice(0, count)

  // Crea dos copias independientes de cada carta y mezcla el resultado
  boardCards.value = shuffle(
    selected.flatMap((c, i) => [
      { ...c, instanceId: `${c.id}-a-${i}`, flipped: false, matched: false, fading: false },
      { ...c, instanceId: `${c.id}-b-${i}`, flipped: false, matched: false, fading: false },
    ]),
  )

  console.log(boardCards.value)

  // Resetea el estado de juego
  gamePhase.value = 'idle'
  countdown.value = 0
  isChecking = false
  flippedCards.value = []
  victory.value = false
}

let isChecking = false
const errorCount = ref(0)
const victory = ref(false)

// Función para voltear cartas
function flipCard(instanceId) {
  console.log('Se gira la carta ' + instanceId)
  const card = boardCards.value.find((c) => c.instanceId === instanceId)
  // Bloqueamos la acción si la carta no existe, ya está volteada, está emparejada,
  // si ya estamos comprobando otra carta o si el juego terminó.
  if (
    !card ||
    card.flipped ||
    card.matched ||
    isChecking ||
    victory.value ||
    gamePhase.value === 'revealing'
  ) {
    return
  }
  card.flipped = true
  flippedCards.value.push(card)
  playFlipSound()

  if (flippedCards.value.length === 2) {
    isChecking = true
    const [first, second] = flippedCards.value

    if (first.id === second.id) {
      // Se ha encontrado un par
      first.matched = true
      second.matched = true
      first.fading = true
      second.fading = true
      flippedCards.value = []
      isChecking = false
      playCorrectSound()

      // Comprobamos si se han emparejado todas las cartas (victoria)
      if (boardCards.value.every((c) => c.matched)) {
        victory.value = true
        gamePhase.value = 'victory'
        clearTimer()

        // Guardamos puntuación y obtenemos posición
        const position = saveScore()

        playVictorySound()

        // Mostrar victoria durante 5 segundos, luego el ranking
        setTimeout(() => {
          victory.value = false
          showRanking.value = true
          // Efecto cuando aparece el ranking
          if (position > 0 && position <= 3) {
            launchConfetti()
            playConfettiSound()
          } else {
            playFailureSound()
          }
        }, 5000)
      }
    } else {
      // No es un par
      errorCount.value++
      playErrorSound()
      setTimeout(() => {
        first.flipped = false
        second.flipped = false
        flippedCards.value = []
        isChecking = false
      }, 1000)
    }
  }
  console.log(flippedCards.value)
}

// Guarda la puntuación del jugador en localStorage
function saveScore() {
  const name = playerName.value

  const score = {
    name,
    time: elapsedTime.value,
    errors: errorCount.value,
  }

  // Recuperar ranking existente o crear uno vacío
  const stored = JSON.parse(localStorage.getItem('ranking' + props.difficulty) || '[]')

  // Añadir nueva puntuación
  stored.push(score)

  // Ordenar: primero por errores, luego por tiempo (mm:ss)
  stored.sort((a, b) => {
    if (a.errors !== b.errors) return a.errors - b.errors
    // Convertir tiempo "mm:ss" a segundos
    const toSeconds = (t) => {
      const [m, s] = t.split(':').map(Number)
      return m * 60 + s
    }
    return toSeconds(a.time) - toSeconds(b.time)
  })

  // Mantener solo los 10 mejores
  const top10 = stored.slice(0, 10)

  // Guardar en localStorage
  localStorage.setItem('ranking' + props.difficulty, JSON.stringify(top10))

  // Buscamos el índice de la puntuación actual dentro del top10
  const playerIndex = top10.findIndex((entry) => {
    const sameName = entry.name === name
    const sameErrors = entry.errors === score.errors
    const sameTime = entry.time === score.time
    return sameName && sameErrors && sameTime
  })

  // Convertimos el índice a posición
  const playerPosition = playerIndex + 1

  // Devolvemos la posición (0 si no se encontró)
  return playerPosition
}

const gridCols = computed(() => Math.ceil(Math.sqrt(boardCards.value.length)))
const gamePhase = ref('idle')
const countdown = ref(0)

// Inicia un cronómetro desde cero y actualiza cada segundo la variable elapsedTime
function startTimer() {
  // Guardamos el momento actual como hora de inicio del juego
  startTime.value = DateTime.now()
  // Inicializamos el tiempo transcurrido en 00:00
  elapsedTime.value = '00:00'

  // Creamos un intervalo que se ejecuta cada segundo
  timerId = setInterval(() => {
    // Obtenemos el momento actual
    const now = DateTime.now()
    // Calculamos la diferencia entre ahora y el inicio en minutos y segundos
    const diff = now.diff(startTime.value, ['minutes', 'seconds']).toObject()
    // Redondeamos hacia abajo los minutos y segundos y los convertimos en strings de 2 dígitos
    const minutes = String(Math.floor(diff.minutes)).padStart(2, '0')
    const seconds = String(Math.floor(diff.seconds)).padStart(2, '0')
    // Actualizamos la variable elapsedTime con el formato "mm:ss"
    elapsedTime.value = `${minutes}:${seconds}`
  }, 1000)
}

// Detiene el cronómetro y limpia el identificador del intervalo, evitando que siga actualizando la variable elapsedTime
const clearTimer = () => {
  if (timerId) clearInterval(timerId)
  timerId = null
}

// Inicia el juego
function startGame() {
  showRanking.value = false
  clearTimer()
  errorCount.value = 0
  victory.value = false
  flippedCards.value = []

  // Reinicia todas las cartas a su estado inicial (boca abajo y sin emparejar)
  boardCards.value.forEach((c) =>
    Object.assign(c, { flipped: false, matched: false, fading: false }),
  )

  // Fase 1: cuenta atrás de preparación (3 segundos)
  gamePhase.value = 'revealing'
  countdown.value = 3

  timerId = setInterval(() => {
    if (--countdown.value <= 0) {
      clearTimer()

      // Fase 2: muestra las cartas durante 5 segundos para memorizar
      boardCards.value.forEach((c) => (c.flipped = true))
      gamePhase.value = 'showing'
      countdown.value = 5

      timerId = setInterval(() => {
        if (--countdown.value <= 0) {
          clearTimer()

          // Fase 3: oculta las cartas y comienza el juego real
          boardCards.value.forEach((c) => (c.flipped = false))
          gamePhase.value = 'playing'
          startTimer()
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
            {{ gamePhase === 'revealing' ? 'Preparando...' : '¡Memoriza!' }}
          </div>
        </div>
      </div>

      <button class="play-btn" @click="openNameModal">
        {{ gamePhase === 'idle' ? '▶' : '↩️ Reset game' }}
      </button>
      <button class="change-difficulty" @click="changeDifficultyAndReset()">
        ⚙️ Change difficulty
      </button>
    </div>

    <!-- Mensaje de victoria -->
    <div v-if="victory" class="victory-overlay">
      <div class="victory-card">
        <h2> ¡Victoria! </h2>
        <p><strong>Jugador:</strong> {{ playerName }}</p>
        <p><strong>Tiempo:</strong> {{ elapsedTime }}</p>
        <p><strong>Errores:</strong> {{ errorCount }}</p>
      </div>
    </div>
    <UserRanking
      v-if="showRanking"
      :difficulty="difficulty"
      :current-player="{ name: playerName, errors: errorCount, time: elapsedTime }"
    />
    <div v-else>
      <div
        class="board-grid"
        role="grid"
        :style="{ gridTemplateColumns: `repeat(${gridCols}, 80px)` }"
      >
        <GameCard
          v-for="c in boardCards"
          :key="c.instanceId"
          :card="c"
          @flip-card="flipCard(c.instanceId)"
        />
      </div>
      <div class="counters-container">
        <div class="error-counter">Errores: {{ errorCount }}</div>
        <div class="time-counter">Tiempo: {{ elapsedTime }}</div>
      </div>
    </div>
  </section>
  <div v-if="modal">
    <!-- NameSelector emitirá ('start-game', nombre) -->
    <NameSelector @start-game="handleStartFromModal" />
  </div>
</template>

<style scoped>
.game-board {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  align-items: center;
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

.counters-container {
  display: flex;
  justify-content: space-between;
  width: 100%;
}

.error-counter {
  margin-top: 1.5rem;
  text-align: center;
  font-size: 1.5rem;
  font-weight: bold;
  color: var(--deep-sea-blue);
  margin-bottom: 7rem;
}

.time-counter {
  margin-top: 1.5rem;
  text-align: center;
  font-size: 1.5rem;
  font-weight: bold;
  color: var(--deep-sea-blue);
  margin-bottom: 7rem;
}

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

.victory-overlay {
  text-align: center;
  font-size: 3rem;
  font-weight: bold;
  color: white;
  padding: 1rem 2rem;
  border-radius: 12px;
  margin: 0 auto 1rem auto;
}

.victory-card {
  background: linear-gradient(135deg, var(--sea-blue), var(--deep-sea-blue));
  color: white;
  padding: 2rem 3rem;
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
  text-align: center;
  animation: popIn 0.5s ease-out forwards;
  min-width: 280px;
}

.victory-card h2 {
  font-size: 2.5rem;
  margin-bottom: 1rem;
  text-shadow: 2px 2px 6px rgba(0,0,0,0.4);
}

.victory-card p {
  font-size: 1.3rem;
  margin: 0.5rem 0;
}


.change-difficulty {
  color: black;
  background-color: var(--celeste);
  padding: 18px 42px;
  font-size: 2rem;
  font-weight: bold;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  transition:
    background-color 0.2s,
    transform 0.1s;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
  margin-left: 2rem;
}

/* Tablet */
@media (min-width: 768px) and (max-width: 1024px) {
  .board-grid {
    column-gap: 1rem;
    row-gap: 1rem;
    /* Ajusta el tamaño de las cartas para que quepan mejor */
    grid-template-columns: repeat(auto-fit, minmax(60px, 1fr));
  }

  .play-btn,
  .change-difficulty {
    padding: 14px 32px;
    font-size: 1.5rem;
  }

  .error-counter,
  .time-counter {
    font-size: 1.2rem;
  }

  .victory-overlay {
    font-size: 2rem;
    padding: 0.8rem 1.5rem;
  }
}
</style>
