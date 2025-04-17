<script setup>
import { ref, computed, onMounted, nextTick } from 'vue'

const GAME_STATES = {
  start: 'start',
  aiming: 'aiming',
  choiceImpactForce: 'choiceImpactForce',
  ballFlying: 'ballFlying',
  goal: 'goal',
  catch: 'catch',
  miss: 'miss',
  pause: 'pause',
}

const props = defineProps({
  playerName: String,
  timeLimit: Number,
  isTestMode: Boolean,
})

const emit = defineEmits(['endGame'])

const score = ref(0)
const timeLeft = ref(props.timeLimit)
const timerInterval = ref(null)

const ballRef = ref(null)
const goalkeeperRef = ref(null)
const goalRef = ref(null)

const gameState = ref(GAME_STATES.start)
const mousePos = ref({ x: 0, y: 0 })
const ballCenter = ref({ x: 0, y: 0 })

const selectedTargetPos = ref({ x: 0, y: 0 })
const transformBall = ref({ x: 0, y: 0 })

const isStart = computed(() => {
  return gameState.value === GAME_STATES.start
})
const isAiming = computed(() => {
  return gameState.value === GAME_STATES.aiming
})
const isChoiceImpactForce = computed(() => {
  return gameState.value === GAME_STATES.choiceImpactForce
})
const isBallFlying = computed(() => {
  return gameState.value === GAME_STATES.ballFlying
})
const isGoal = computed(() => {
  return gameState.value === GAME_STATES.goal
})
const isCatch = computed(() => {
  return gameState.value === GAME_STATES.catch
})
const isMiss = computed(() => {
  return gameState.value === GAME_STATES.miss
})
const isPause = computed(() => {
  return gameState.value === GAME_STATES.pause
})

function handleClickBall() {
  if (isStart.value) {
    startAiming()
  } else if (isChoiceImpactForce.value) {
    stopPowerSelection()
  }
}

function startAiming() {
  gameState.value = GAME_STATES.aiming
  updateBallCenter()
}

function selectTarget() {
  if (isAiming.value) {
    gameState.value = GAME_STATES.ballFlying
    selectedTargetPos.value = mousePos.value
    startPowerSelection()
  }
}

function flyBall() {
  const currentBallPos = ballRef.value.getBoundingClientRect()
  transformBall.value = {
    x: selectedTargetPos.value.x - currentBallPos.x - 28,
    y: selectedTargetPos.value.y - currentBallPos.y + 28,
  }

  gameState.value = GAME_STATES.ballFlying

  setTimeout(() => {
    checkGoal()
  }, flightTimeMS.value)
}
let goalStrike = ref(0)
function checkGoal() {
  const goalkeeperBox = goalkeeperRef.value.getBoundingClientRect()
  const goalBox = goalRef.value.getBoundingClientRect()

  if (
    goalBox.left > selectedTargetPos.value.x ||
    goalBox.right < selectedTargetPos.value.x ||
    goalBox.top > selectedTargetPos.value.y + 66 ||
    goalBox.bottom < selectedTargetPos.value.y + 66
  ) {
    gameState.value = GAME_STATES.miss
    goalStrike.value = 0
  } else if (
    goalkeeperBox.left - goalkeeperBox.width / 2 <= selectedTargetPos.value.x &&
    goalkeeperBox.right + goalkeeperBox.width / 2 >= selectedTargetPos.value.x &&
    goalkeeperBox.top - goalkeeperBox.height / 2 <= selectedTargetPos.value.y &&
    goalkeeperBox.bottom + goalkeeperBox.height / 2 >= selectedTargetPos.value.y
  ) {
    gameState.value = GAME_STATES.catch
    goalStrike.value = 0
  } else {
    gameState.value = GAME_STATES.goal
    score.value++
    goalStrike.value++
    if (goalStrike.value >= 3) {
      triggerBonus()
    }
  }
  stopGoalkeeperAnimation()

  setTimeout(() => {
    gameState.value = GAME_STATES.start
    selectedTargetPos.value = { x: 0, y: 0 }
    transformBall.value = { x: 0, y: 0 }
    animationFrameId = requestAnimationFrame(animateGoalkeeper)
  }, 1000)
}

function updateAim(event) {
  if (!isAiming.value) return
  const fieldRect = event.currentTarget.getBoundingClientRect()
  mousePos.value = {
    x: event.clientX - fieldRect.left,
    y: event.clientY - fieldRect.top,
  }
}

function updateBallCenter() {
  nextTick(() => {
    const rect = ballRef.value.getBoundingClientRect()
    const fieldRect = ballRef.value.offsetParent.getBoundingClientRect()
    ballCenter.value = {
      x: rect.left - fieldRect.left + rect.width / 2,
      y: rect.top - fieldRect.top + rect.height / 2,
    }
  })
}

const displayName = computed(() => {
  return props.playerName
})

const formattedTime = computed(() => {
  const minutes = Math.floor(timeLeft.value / 60)
    .toString()
    .padStart(2, '0')
  const seconds = (timeLeft.value % 60).toString().padStart(2, '0')
  return `${minutes}:${seconds}`
})

function handleEndGame() {
  emit('endGame', score.value)
}

const goalkeeperX = ref(0)
const direction = ref(1)

let goalWidth = 0
let keeperWidth = 0
const DEFAULT_DURATION = 1000
let lastTimestamp = null
let animationFrameId = null

const currentDuration = computed(() => {
  return DEFAULT_DURATION - 10 * score.value
})

onMounted(() => {
  goalWidth = goalRef.value.clientWidth
  keeperWidth = goalkeeperRef.value.clientWidth
  animationFrameId = requestAnimationFrame(animateGoalkeeper)

  startTimer()
})

function animateGoalkeeper(timestamp) {
  if (!lastTimestamp) lastTimestamp = timestamp
  const delta = timestamp - lastTimestamp
  lastTimestamp = timestamp

  const distance = goalWidth - keeperWidth
  const speed = distance / currentDuration.value
  goalkeeperX.value += direction.value * speed * delta

  if (goalkeeperX.value >= distance || goalkeeperX.value <= 0) {
    direction.value *= -1
    goalkeeperX.value = Math.max(0, Math.min(goalkeeperX.value, distance))
  }

  animationFrameId = requestAnimationFrame(animateGoalkeeper)
}

function stopGoalkeeperAnimation() {
  cancelAnimationFrame(animationFrameId)
  animationFrameId = null
}

const powerValue = ref(0)
const flightTimeMS = computed(() => {
  const BASE_TIME = 500
  const PART_POWER = 2000
  const BONUS_MULTIPLIER = goalStrike.value >= 3 ? 2 : 1
  return (
    (BASE_TIME + +((PART_POWER * (100 - powerValue.value)) / 100)) /
    BONUS_MULTIPLIER
  ).toFixed(2)
})

const flightTime = computed(() => {
  return `${flightTimeMS.value / 1000}s`
})
let powerIncreasing = true
let powerTimer = null

function startPowerSelection() {
  gameState.value = GAME_STATES.choiceImpactForce
  powerValue.value = 0
  powerIncreasing = true

  powerTimer = setInterval(() => {
    if (powerIncreasing) {
      powerValue.value += 5
      if (powerValue.value >= 100) {
        powerValue.value = 100
        powerIncreasing = false
      }
    } else {
      powerValue.value -= 5
      if (powerValue.value <= 0) {
        powerValue.value = 0
        powerIncreasing = true
      }
    }
  }, 15)
}

function stopPowerSelection() {
  clearInterval(powerTimer)
  powerTimer = null

  flyBall()
}

function startTimer() {
  if (!props.isTestMode) {
    timerInterval.value = setInterval(() => {
      if (timeLeft.value <= 0) {
        handleEndGame()
      }
      timeLeft.value = timeLeft.value - 1
    }, 1000)
  }
}

function handlePauseClick() {
  if (isPause.value) {
    continueGame()
  } else if (isStart.value) {
    pauseGame()
  }
}
function pauseGame() {
  gameState.value = GAME_STATES.pause
  stopGoalkeeperAnimation()
  clearInterval(timerInterval.value)
}
function continueGame() {
  gameState.value = GAME_STATES.start
  animationFrameId = requestAnimationFrame(animateGoalkeeper)
  startTimer()
}

const showBonus = ref(false)
function triggerBonus() {
  showBonus.value = true
  setTimeout(() => {
    showBonus.value = false
  }, 2000)
}
</script>

<template>
  <div class="game-screen">
    <header class="game-header">
      <div class="timer">{{ formattedTime }}</div>
      <div class="score">Очки: {{ score }}</div>
      <div class="player-name">Игрок: {{ displayName }}</div>
      <button v-if="isTestMode" class="continue-btn" @click="handleEndGame">
        Закончить тестовую игру
      </button>
      <button class="pause-btn" @click="handlePauseClick" :disabled="!isStart && !isPause">
        <template v-if="!isPause"> Пауза </template>
        <template v-else> Продолжить </template>
      </button>
    </header>

    <main class="game-field" @mousemove="updateAim" @click="selectTarget">
      <div class="goal" ref="goalRef" :style="{}">
        <div class="goalkeeper" :style="{ left: goalkeeperX + 'px' }" ref="goalkeeperRef">
          <img v-if="isCatch" src="../assets/goalkeeperWithBall.png" alt="goalkeeperWithBall" />
          <img v-else src="../assets/goalkeeper.png" alt="goalkeeper" />
        </div>
      </div>
      <transition name="notification-fade">
        <div v-if="isGoal || isCatch || isMiss" class="notification">
          <div v-if="isGoal" class="notification-goal">Гол!</div>
          <div v-if="isCatch" class="notification-catch">Поймал!</div>
          <div v-if="isMiss" class="notification-miss">Промазал!</div>
        </div>
      </transition>
      <transition name="bonus-fade">
        <div v-if="showBonus" class="bonus-notification">Бонус: Ускорение мяча!</div>
      </transition>

      <div v-if="isChoiceImpactForce || isBallFlying" class="power-indicator">
        <div class="bar">
          <div class="fill" :style="{ width: powerValue + '%' }"></div>
        </div>
      </div>
      <div
        class="ball"
        @click.stop="handleClickBall"
        ref="ballRef"
        :style="{
          transitionDuration: flightTime,
          transform: isBallFlying ? `translate(${transformBall.x}px, ${transformBall.y}px)` : '',
        }"
      >
        <img src="../assets/ball.png" alt="ball" />
      </div>

      <svg v-if="isAiming || isChoiceImpactForce" class="trajectory">
        <defs>
          <linearGradient id="dash-gradient" x1="0" y1="0" x2="0" y2="1">
            <stop offset="0%" stop-color="#6aaa64" />
            <stop offset="100%" stop-color="#3a3a3a" />
          </linearGradient>
        </defs>
        <line
          :x1="ballCenter.x"
          :y1="ballCenter.y"
          :x2="mousePos.x"
          :y2="mousePos.y"
          stroke="url(#dash-gradient)"
          stroke-width="3"
          stroke-dasharray="6, 6"
          class="dash-line"
        />
        <circle :cx="mousePos.x" :cy="mousePos.y" r="6" fill="#6aaa64" class="end-circle" />
        <circle
          :cx="ballCenter.x"
          :cy="ballCenter.y"
          r="38"
          stroke="#6aaa64"
          stroke-width="3"
          stroke-dasharray="6, 6"
          class="dash-line"
        />
      </svg>

      <svg v-if="isBallFlying" class="target">
        <defs>
          <linearGradient id="dash-gradient" x1="0" y1="0" x2="0" y2="1">
            <stop offset="0%" stop-color="#6aaa64" />
            <stop offset="100%" stop-color="#3a3a3a" />
          </linearGradient>
        </defs>
        <circle
          :cx="selectedTargetPos.x"
          :cy="selectedTargetPos.y"
          r="6"
          fill="#6aaa64"
          class="end-circle"
        />
      </svg>
    </main>
  </div>
</template>

<style scoped>
.game-screen {
  user-select: none;
  background-color: #2f2f2f;
  color: white;
  min-height: 100vh;
  font-family: 'Arial', sans-serif;
  display: flex;
  flex-direction: column;
}

.game-header {
  display: flex;
  justify-content: space-between;
  padding: 1rem 2rem;
  background-color: #3a3a3a;
  font-size: 1.2rem;
  align-items: center;
}

.continue-btn {
  padding: 0.5rem 1rem;
  background-color: #6aaa64;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-weight: bold;
}

.pause-btn {
  padding: 0.5rem 1rem;
  background-color: #6aaa64;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-weight: bold;
}

.pause-btn:disabled {
  background-color: #888;
  cursor: not-allowed;
}

.game-field {
  flex-grow: 1;
  position: relative;
  display: flex;
  justify-content: center;
  padding: 2rem;
}

.goal {
  position: relative;
  width: 80%;
  height: 300px;
  background: #444;
  border: 4px solid white;
  margin-bottom: 2rem;
}

.goalkeeper {
  position: absolute;
  bottom: 0;
  width: 15%;
  height: 100%;
  border-radius: 8px;
}

.goalkeeper img {
  height: 100%;
}

.ball {
  width: 75px;
  height: 75px;
  border-radius: 50%;
  position: absolute;
  bottom: 20px;
  cursor: pointer;
  z-index: 10;
  transition: 1s linear;
}
.ball img {
  height: 100%;
}

.target {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.trajectory {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 3;
}

.dash-line {
  stroke-dasharray: 6, 6;
  animation: dash 1s linear infinite;
}

@keyframes dash {
  to {
    stroke-dashoffset: -12;
  }
}

.notification {
  position: absolute;
  top: 70px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 2rem;
  color: #6aaa64;
  font-weight: bold;
  z-index: 5;
}
.notification-goal {
  color: #6aaa64;
}
.notification-catch {
  color: #ff0a16;
}
.notification-miss {
  color: #e2e2e2;
}

.notification-fade-enter-active,
.notification-fade-leave-active {
  transition:
    opacity 0.5s ease,
    transform 0.5s ease;
}
.notification-fade-enter-from,
.notification-fade-leave-to {
  opacity: 0;
  transform: translate(-50%, -10px);
}
.notification-fade-enter-to,
.notification-fade-leave-from {
  opacity: 1;
  transform: translate(-50%, 0);
}

.power-indicator {
  position: absolute;
  bottom: 120px;
  left: 50%;
  transform: translateX(-50%);
  width: 200px;
  height: 20px;
  background: #333;
  border: 2px solid #6aaa64;
  border-radius: 10px;
  z-index: 4;
  overflow: hidden;
}

.power-indicator .bar {
  width: 100%;
  height: 100%;
  background: transparent;
}

.power-indicator .fill {
  height: 100%;
  background: #6aaa64;
}

.bonus-notification {
  position: absolute;
  top: 120px;
  left: 50%;
  transform: translateX(-50%);
  padding: 12px 24px;
  background-color: #ffd700;
  color: #1f1f1f;
  font-weight: bold;
  font-size: 1.5rem;
  border-radius: 12px;
  box-shadow: 0 0 20px rgba(255, 215, 0, 0.6);
  z-index: 10;
  animation: pulse 1s ease-in-out infinite;
}

@keyframes pulse {
  0%,
  100% {
    transform: translateX(-50%) scale(1);
  }
  50% {
    transform: translateX(-50%) scale(1.1);
  }
}

.bonus-fade-enter-active,
.bonus-fade-leave-active {
  transition:
    opacity 0.5s ease,
    transform 0.5s ease;
}
.bonus-fade-enter-from,
.bonus-fade-leave-to {
  opacity: 0;
  transform: translate(-50%, -10px);
}
.bonus-fade-enter-to,
.bonus-fade-leave-from {
  opacity: 1;
  transform: translate(-50%, 0);
}
</style>
