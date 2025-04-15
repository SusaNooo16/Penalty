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
function handleTest() {
  emit('startGame', 'tester', 10)
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
        <button class="btn-test" @click="handleTest">
          <svg
            fill="#000000"
            width="25px"
            height="25px"
            viewBox="0 0 32 32"
            version="1.1"
            xmlns="http://www.w3.org/2000/svg"
          >
            <path
              d="M19.332 19.041c0 0-1.664 2.125-3.79 0-2.062-2-3.562 0-3.562 0l-4.967 9.79c-0.144 0.533 0.173 1.081 0.706 1.224h16.497c0.533-0.143 0.85-0.69 0.707-1.224l-5.591-9.79zM26.939 28.33l-7.979-13.428v-0.025l-0.014-7.869h0.551c0.826 0 1.498-0.671 1.498-1.499 0-0.827-0.672-1.498-1.498-1.498h-7.995c-0.827 0-1.498 0.671-1.498 1.498 0 0.828 0.671 1.499 1.498 1.499h0.482l-0.016 7.871-6.908 13.451c-0.428 1.599 0.521 3.242 2.119 3.67h17.641c1.6-0.428 2.549-2.071 2.119-3.67zM24.553 30.998l-17.108-0.019c-1.065-0.286-1.697-1.382-1.412-2.446l6.947-13.616 0.021-8.908h-1.498c-0.275 0-0.499-0.224-0.499-0.5s0.224-0.499 0.499-0.499h7.995c0.275 0 0.498 0.224 0.498 0.499 0 0.276-0.223 0.5-0.498 0.5h-1.498l0.025 8.875 7.939 13.666c0.286 1.067-0.347 2.163-1.411 2.448zM16.48 2.512c0 0.552 0.448 1 1 1s1-0.448 1-1-0.447-1-1-1-1 0.447-1 1zM17.48 0.012c0.828 0 1.5-0.671 1.5-1.5s-0.672-1.5-1.5-1.5-1.5 0.671-1.5 1.5 0.672 1.5 1.5 1.5zM13.48 2.512c0.553 0 1-0.448 1-1s-0.447-1-1-1-1 0.448-1 1 0.447 1 1 1z"
            ></path>
          </svg>
        </button>
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
