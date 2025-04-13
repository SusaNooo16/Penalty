<script setup>
import { ref, computed } from 'vue'

const emit = defineEmits(['startGame'])

const name = ref('')
const time = ref(0)

const isValid = computed(() => {
  return name.value.trim() !== '' && time.value >= 10
})

function handleStart() {
  if (isValid.value) {
    emit('startGame', name.value.trim(), time.value)
  }
}
</script>

<template>
  <div class="start-screen">
    <h1>Игра "Пенальти"</h1>
    <form @submit.prevent="handleStart">
      <div class="input-group">
        <label for="name">Имя игрока:</label>
        <input id="name" v-model="name" type="text" required />
      </div>
      <div class="input-group">
        <label for="time">Время (сек):</label>
        <input id="time" v-model.number="time" type="number" min="10" required />
      </div>
      <div class="btn-group">
        <button class="btn-start" :disabled="!isValid">Начать игру</button>
      </div>
    </form>
  </div>
</template>

<style scoped>
.start-screen {
  background-color: #2f2f2f;
  color: #fff;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  font-family: 'Arial', sans-serif;
}

h1 {
  font-size: 2rem;
  margin-bottom: 2rem;
  color: #ffffff;
}

form {
  background-color: #3a3a3a;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
  display: flex;
  flex-direction: column;
  gap: 1rem;
  min-width: 280px;
}

.input-group {
  display: flex;
  flex-direction: column;
}

label {
  margin-bottom: 0.5rem;
  color: #ccc;
  font-weight: bold;
}

input {
  padding: 0.6rem;
  font-size: 1rem;
  border: none;
  border-radius: 6px;
  background-color: #555;
  color: white;
}

button {
  padding: 0.8rem;
  font-size: 1rem;
  background-color: #6aaa64;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s;
  font-weight: bold;
}
.btn-group {
  display: flex;
  gap: 10px;
}
.btn-start {
  flex: 1;
}
button:hover:not(:disabled) {
  background-color: #5b9857;
}

button:disabled {
  background-color: #888;
  cursor: not-allowed;
}
</style>
