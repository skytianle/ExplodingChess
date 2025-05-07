<template>
  <div class="game-container">
    <h1>爆炸棋</h1>
    <div v-if="!gameStarted" class="setup-screen">
      <div class="size-selector">
        <label>棋盘尺寸：</label>
        <select v-model="selectedSize">
          <option value="5">5x5</option>
          <option value="7">7x7</option>
          <option value="9">9x9</option>
        </select>
      </div>
      <button @click="startGame" class="start-btn">开始游戏</button>
    </div>
    <ExplosionChess v-else :board-size="selectedSize" />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import ExplosionChess from './components/ExplosionChess.vue'

const gameStarted = ref(false)
const selectedSize = ref(5)

const startGame = () => {
  gameStarted.value = true
}
</script>

<style>
.game-container {
  max-width: 800px;
  margin: 0 auto;
  text-align: center;
  padding: 20px;
}

.setup-screen {
  background: #f8f9fa;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin: 2rem auto;
  max-width: 400px;
}

.size-selector {
  margin: 1.5rem 0;
}

.size-selector select {
  padding: 8px 12px;
  border-radius: 6px;
  border: 1px solid #ced4da;
  font-size: 16px;
}

.start-btn {
  background: #4a90e2;
  color: white;
  padding: 12px 24px;
  border-radius: 6px;
  border: none;
  font-size: 16px;
  cursor: pointer;
  transition: background 0.3s ease;
}

.start-btn:hover {
  background: #357abd;
}
</style>