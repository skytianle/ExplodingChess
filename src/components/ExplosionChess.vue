<template>
    <div class="game-info">
      <p>当前玩家：{{ currentPlayer }}</p>
      <p>当前状态：{{ getCurrentStatus(currentStatus) }}</p>
      <button @click="resetGame">重置游戏</button>
      <button @click="$emit('reset-board-size')">退出并重选棋盘大小</button>
    </div>
  <div class="chess-board">
    <div 
      v-for="(row, rowIndex) in board" 
      :key="rowIndex" 
      class="board-row"
    >
      <div 
        v-for="(cell, cellIndex) in row" 
        :key="cellIndex" 
        class="board-cell"
        :class="{
          'corner': isCorner(rowIndex, cellIndex),
          'edge': isEdge(rowIndex, cellIndex),
          'center': isCenter(rowIndex, cellIndex)
        }"
      >
        <div 
          v-for="(subCell, subIndex) in cell.subCells" 
          :key="subIndex" 
          class="sub-cell"
          :class="subCell.owner"
          @click="handleCellClick(rowIndex, cellIndex, subIndex)"
        ></div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

const props = defineProps({
  boardSize: {
    type: Number,
    required: true
  }
});

const boardSize = ref(props.boardSize);
const players = ['player1', 'player2'];
const currentPlayer = ref(players[0]);
const currentStatus = ref(0); // 0: 正常游戏状态, 1: 扩散中, 2: 游戏结束

// 初始化棋盘
const board = ref(initializeBoard());

function initializeBoard() {
  const board = [];
  for (let i = 0; i < boardSize.value; i++) {
    const row = [];
    for (let j = 0; j < boardSize.value; j++) {
      row.push({
        subCells: createSubCells(i, j),
        owner: null
      });
    }
    row.push();
    board.push(row);
  }
  return board;
}

function createSubCells(row, col) {
  if (isCorner(row, col)) {
    return Array(2).fill().map(() => ({ owner: null }));
  } else if (isEdge(row, col)) {
    return Array(3).fill().map(() => ({ owner: null }));
  } else {
    return Array(4).fill().map(() => ({ owner: null }));
  }
}

function isCorner(row, col) {
  return (
    (row === 0 && col === 0) ||
    (row === 0 && col === boardSize.value - 1) ||
    (row === boardSize.value - 1 && col === 0) ||
    (row === boardSize.value - 1 && col === boardSize.value - 1)
  );
}

function isEdge(row, col) {
  return (
    (row === 0 || row === boardSize.value - 1 || col === 0 || col === boardSize.value - 1) &&
    !isCorner(row, col)
  );
}

function isCenter(row, col) {
  return !isCorner(row, col) && !isEdge(row, col);
}

async function handleCellClick(row, col , sub) {
  if (currentStatus.value !== 0) {
    alert('游戏结束或者正在扩散或者正在结算，不允许行动');
    return;
  }
  const cell = board.value[row][col];
  const subcell = board.value[row][col].subCells[sub];

  if (subcell.owner) {
    return;
  } else if (cell.owner) {
    if (cell.owner === currentPlayer.value) {
    } else {
     alert('不能在对方格子上放置棋子');
      return; // 阻止放置棋子，返回不执行后面的代码 
    }
  }

  // 占领子格子
  subcell.owner = currentPlayer.value;
  cell.owner = currentPlayer.value;
  
  // 检查是否填满所有子格子
  if (cell.subCells.every(subCell => subCell.owner)) {
    explodeCell(row, col);
  }
  
  // 切换玩家
  currentPlayer.value = currentPlayer.value === players[0] ? players[1] : players[0];
}

async function explodeCell(row, col) {
  currentStatus.value = 1;
  await sleep(1000);
  const owner = board.value[row][col].owner;
  
  // 清空当前格子的所有子格子
  board.value[row][col].subCells = board.value[row][col].subCells.map(() => ({ owner: null }));
  board.value[row][col].owner = null;
  
  // 收集所有扩散任务
  const explosions = [];
  const directions = [
    [-1, 0], [1, 0], [0, -1], [0, 1]
  ];

  // 处理单个方向扩散的异步函数
  const processDirection = async (dx, dy) => {
    const newRow = row + dx;
    const newCol = col + dy;
    
    if (newRow >= 0 && newRow < boardSize.value && newCol >= 0 && newCol < boardSize.value) {
      const targetCell = board.value[newRow][newCol];
      
      if (!targetCell.owner) {
        const emptySubCellIndex = targetCell.subCells.findIndex(subCell => !subCell.owner);
        if (emptySubCellIndex !== -1) {
          targetCell.owner = owner;
          targetCell.subCells[emptySubCellIndex].owner = owner;
          targetCell.subCells[emptySubCellIndex].animate = true;
          
          if (targetCell.subCells.every(subCell => subCell.owner)) {
            explosions.push(await explodeCell(newRow, newCol));
          }
        }
      } else if (targetCell.owner === owner) {
        const emptySubCellIndex = targetCell.subCells.findIndex(subCell => !subCell.owner);
        if (emptySubCellIndex !== -1) {
          targetCell.owner = owner;
          targetCell.subCells[emptySubCellIndex].owner = owner;
          targetCell.subCells[emptySubCellIndex].animate = true;
          
          if (targetCell.subCells.every(subCell => subCell.owner)) {
            explosions.push(await explodeCell(newRow, newCol));
          }
        }
      } else {
        targetCell.owner = owner;
        targetCell.subCells.forEach(subCell => {
          if(subCell.owner) {
            subCell.owner = owner;
            subCell.animate = true;
          }
        });
        const emptySubCellIndex = targetCell.subCells.findIndex(subCell => !subCell.owner);
        if (emptySubCellIndex !== -1) {
          targetCell.owner = owner;
          targetCell.subCells[emptySubCellIndex].owner = owner;
          targetCell.subCells[emptySubCellIndex].animate = true;
          
          if (targetCell.subCells.every(subCell => subCell.owner)) {
            explosions.push(await explodeCell(newRow, newCol));
          }
        }
      }
    }
  };

  // 并行处理所有方向的扩散
  await Promise.all(directions.map(([dx, dy]) => processDirection(dx, dy)));
  
  // 等待所有递归爆炸完成
  await Promise.all(explosions);
  
  console.log(getCurrentStatus(currentStatus.value))
  // 最后执行游戏结束检查
  checkGameEnd();
}

function checkGameEnd() {
  const owners = new Set();
  
  // 收集所有被占领的格子
  for (let i = 0; i < boardSize.value; i++) {
    for (let j = 0; j < boardSize.value; j++) {
      if (board.value[i][j].owner) {
        owners.add(board.value[i][j].owner);
      }
    }
  }
  
  // 如果只剩一个玩家，游戏结束
  if (owners.size === 1) {
    const winner = [...owners][0];
    currentStatus.value = 3;
    setTimeout(() => {
      currentStatus.value = 2;
    }, 500);
  } else {
    currentStatus.value = 0; 
  }
}
function resetGame() {
  // 重置棋盘
  boardSize.value = props.boardSize;
  board.value = initializeBoard();
  // 重置玩家
  currentPlayer.value = players[0];
  currentStatus.value = 0;
  owners.clear(); 
}

function getCurrentStatus(N) {
  switch (N) {
    case 0:
      return "正在进行";
    case 1:
      return `${currentPlayer.value} 正在爆炸`;
    case 2:
      return "已结束 "+`${currentPlayer.value}获胜`;
    case 3:
      return "正在结算";
    default:
      return "正在进行";
  }
}
</script>

<style scoped>
.chess-board {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 20px;
}

.board-row {
  display: flex;
}

.board-cell {
  width: 60px;
  height: 60px;
  margin: 2px;
  background-color: #eee;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
  cursor: pointer;
}

.sub-cell {
  position: relative;
  transition: all 0.3s ease;
  width: 20px;
  height: 20px;
  margin: 2px;
  background-color: #ddd;
  transition: background-color 0.3s ease, transform 0.2s;
  position: relative;
}

.sub-cell.player1 {
  background-color: #ff5252;
}

.sub-cell[animate] {
  animation: expand 0.4s ease-out;
}

.sub-cell.player2 {
  background-color: #4caf50;
}

.corner {
  border-radius: 10px 0;
}

.edge {
  border-radius: 5px;
}

@keyframes explode {
  0% { transform: scale(1); opacity: 1; }
  50% { transform: scale(1.5); opacity: 0.5; }
  100% { transform: scale(1); opacity: 1; }
}

@keyframes particle {
  0% { transform: translate(0, 0); opacity: 1; }
  100% { transform: translate(var(--x), var(--y)); opacity: 0; }
}

.active {
  animation: explode 0.4s ease-out;
}

.sub-cell::after {
  content: '';
  position: absolute;
  width: 0;
  height: 0;
  background: currentColor;
  border-radius: 50%;
  animation: particle 0.6s ease-out;
}
.game-info {
  display: flex;
  gap: 2rem;
  align-items: center;
  background: #ffffff;
  padding: 1rem 2rem;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  margin: 1rem 0;
}

.game-info p {
  margin: 0;
  font-size: 1.1rem;
  color: #333;
}

.game-info button {
  background: #4a90e2;
  color: white;
  padding: 8px 16px;
  border-radius: 4px;
  border: none;
  cursor: pointer;
  transition: background 0.3s ease;
}

.game-info button:hover {
  background: #357abd;
}
</style>