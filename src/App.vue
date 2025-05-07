<template>
  <div class="game-container">
    <h1>爆炸棋 天乐出品</h1>
    <div v-if="!gameStarted" class="setup-screen">
      <div class="player-selection">
        <h3>选择玩家颜色（至少2个）</h3>
        <div class="color-grid">
          <label 
            v-for="color in colorOptions" 
            :key="color.id"
            :class="['color-option', color.class, { selected: selectedColors.includes(color.id) }]"
          >
            <input 
              type="checkbox"
              :value="color.id"
              v-model="selectedColors"
              @change="validateSelection"
            />
            {{ color.name }}
          </label>
        </div>
      </div>
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
    <ExplosionChess v-else :board-size="selectedSize" :player-colors="selectedColors" @reset-board-size="handleResetBoard" />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import ExplosionChess from './components/ExplosionChess.vue'

const gameStarted = ref(false)
const selectedSize = ref(5);
const selectedColors = ref([1, 2]); // 默认选择红橙两色

const colorOptions = [
  { id: 1, name: '红色', class: 'red' },
  { id: 2, name: '橙色', class: 'orange' },
  { id: 3, name: '黄色', class: 'yellow' },
  { id: 4, name: '绿色', class: 'green' },
  { id: 5, name: '蓝色', class: 'blue' },
  { id: 6, name: '紫色', class: 'purple' },
  { id: 7, name: '黑色', class: 'black' },
  { id: 8, name: '白色', class: 'white' }
];

const handleResetBoard = () => {
  gameStarted.value = false;
}

const validateSelection = () => {
  if (selectedColors.value.length < 2) {
    alert('请至少选择两个玩家颜色！');
    selectedColors.value = [1, 2];
  }
};

const startGame = () => {
  // 随机打乱玩家顺序
  selectedColors.value = selectedColors.value
    .map(value => ({ value, sort: Math.random() }))
    .sort((a, b) => a.sort - b.sort)
    .map(({ value }) => value);
    gameStarted.value = true
}
</script>

<style>
.game-container {
  max-width: 1000px;
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

.player-selection {
  margin: 1.5rem 0;
}

.color-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 1rem;
  max-width: 500px;
  margin: 0 auto;
}

.color-option {
  padding: 8px 12px;
  border: 2px solid #ddd;
  border-radius: 6px;
  cursor: pointer;
  text-align: center;
  transition: all 0.2s;

  &.selected {
    border-color: #666;
    box-shadow: 0 2px 6px rgba(0,0,0,0.2);
  }
  &.red { background: #ff5252; }
  &.orange { background: #ffa726; }
  &.yellow { background: #ffee58; }
  &.green { background: #66bb6a; }
  &.blue { background: #42a5f5; }
  &.purple { background: #ab47bc; }
  &.black { background: #424242; color: white; }
  &.white { background: #f5f5f5; }
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