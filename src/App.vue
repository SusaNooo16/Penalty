<script setup>
import { ref } from 'vue'
import StartScreen from './components/StartScreen.vue'
import GameScreen from './components/GameScreen.vue'
import EndScreen from './components/EndScreen.vue'

const gameState = ref('start')
const playerName = ref('')
const timeLimit = ref(0)
const score = ref(0)
const isTestMode = ref(false)

function startGame(name, time) {
  playerName.value = name === 'tester' ? 'Тестовый режим' : name
  isTestMode.value = name === 'tester'
  timeLimit.value = time
  score.value = 0
  gameState.value = 'playing'
}

function endGame(finalScore) {
  score.value = finalScore
  gameState.value = 'end'
}

function restartGame() {
  gameState.value = 'start'
}
</script>

<template>
  <div>
    <StartScreen v-if="gameState === 'start'" @startGame="startGame" />
    <GameScreen
      v-else-if="gameState === 'playing'"
      :playerName="playerName"
      :timeLimit="timeLimit"
      :isTestMode="isTestMode"
      @endGame="endGame"
    />
    <EndScreen
      v-else-if="gameState === 'end'"
      :score="score"
      :playerName="playerName"
      @restartGame="restartGame"
    />
  </div>
</template>

<style scoped>
body {
  margin: 0;
  font-family: sans-serif;
}
</style>
