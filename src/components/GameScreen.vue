<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  playerName: String,
  timeLimit: Number,
})

const emit = defineEmits(['endGame'])

const score = ref(0)
const timeLeft = ref(props.timeLimit)

const displayName = computed(() => {
  return props.playerName
})

const formattedTime = computed(() => {
  return timeLeft.value
})

function handleEndGame() {
  emit('endGame', score.value)
}
</script>

<template>
  <div class="game-screen">
    <header class="game-header">
      <div class="timer">{{ formattedTime }}</div>
      <div class="score">Очки: {{ score }}</div>
      <div class="player-name">Игрок: {{ displayName }}</div>
      <button class="continue-btn" @click="handleEndGame">
        Далее
      </button>
    </header>

    <main class="game-field">
      <div class="goal">
        <div class="goalkeeper-wrap">
          <div class="goalkeeper"></div>
        </div>
      </div>

      <div class="ball"></div>
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
  height: 245px;
  background: #444;
  border: 4px solid white;
  margin-bottom: 2rem;
}

.goalkeeper {
  position: absolute;
  bottom: 0;
  width: 15%;
  height: 100%;
  background-color: #6aaa64;
  transition: left 0.3s;
  border-radius: 8px;
}

.ball {
  width: 40px;
  height: 40px;
  background-color: white;
  border-radius: 50%;
  position: absolute;
  bottom: 20px;
  cursor: pointer;
}
</style>
