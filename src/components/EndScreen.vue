<script setup>
import { defineProps, defineEmits } from 'vue'

defineProps({
  playerName: String,
  score: Number,
})

const emit = defineEmits(['restartGame'])

function restart() {
  emit('restartGame')
}
</script>

<template>
  <div class="end-screen">
    <div class="message-box">
      <h2>Поздравляем, {{ playerName }}!</h2>
      <p class="result-text">
        Вы набрали <strong>{{ score }}</strong> очков
      </p>
      <button @click="restart">Сыграть ещё раз</button>
    </div>
  </div>
</template>

<style scoped>
.end-screen {
  background-color: #2f2f2f;
  color: #fff;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  font-family: 'Arial', sans-serif;
}

.message-box {
  background-color: #3a3a3a;
  padding: 2rem 3rem;
  border-radius: 12px;
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.5);
  text-align: center;
}

h2 {
  font-size: 1.8rem;
  margin-bottom: 1rem;
  color: #ffffff;
}

.result-text {
  font-size: 1.2rem;
  margin-bottom: 2rem;
}

button {
  padding: 0.8rem 1.5rem;
  font-size: 1rem;
  background-color: #6aaa64;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s;
  font-weight: bold;
}

button:hover {
  background-color: #5b9857;
}
</style>
