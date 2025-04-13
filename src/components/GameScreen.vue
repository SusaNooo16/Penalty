<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const props = defineProps({
  playerName: String,
  timeLimit: Number,
})

const emit = defineEmits(['endGame'])

const score = ref(0)
const timeLeft = ref(props.timeLimit)
const keeperPosition = ref(0)
const keeperInterval = ref(null)

const displayName = computed(() => {
  return props.playerName
})

const formattedTime = computed(() => {
  return timeLeft.value
})

function handleEndGame() {
  emit('endGame', score.value)
}

onMounted(() => {
  keeperInterval.value = setInterval(() => {
    if (keeperPosition.value === 0) {
      keeperPosition.value = 85
    } else {
      keeperPosition.value = 0
    }
  }, 1000)
})

onUnmounted(() => {
  clearInterval(keeperInterval.value)
})
</script>

<template>
  <div class="game-screen">
    <header class="game-header">
      <div class="timer">{{ formattedTime }}</div>
      <div class="score">Очки: {{ score }}</div>
      <div class="player-name">Игрок: {{ displayName }}</div>
      <button class="continue-btn" @click="handleEndGame">Далее</button>
    </header>

    <main class="game-field">
      <div class="goal">
        <div class="goalkeeper-wrap">
          <div class="goalkeeper" :style="{ left: keeperPosition + '%' }">
            <img v-if="true" src="../assets/goalkeeper.png" alt="goalkeeper" />
            <img v-else src="../assets//goalkeeperWithBall.png" alt="goalkeeperWithBall" />
          </div>
        </div>
      </div>

      <div class="ball">
        <img src="../assets/ball.png" alt="ball" />
      </div>
    </main>
  </div>
</template>

<style scoped>
.game-screen {
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
  transition: left 1s;
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
}
.ball img {
  height: 100%;
}
</style>
