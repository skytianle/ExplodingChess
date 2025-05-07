<template>
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

const boardSize = 5; // 5x5棋盘
const players = ['player1', 'player2'];
const currentPlayer = ref(players[0]);

// 初始化棋盘
const board = ref(initializeBoard());

function initializeBoard() {
  const board = [];
  for (let i = 0; i < boardSize; i++) {
    const row = [];
    for (let j = 0; j < boardSize; j++) {
      row.push({
        subCells: createSubCells(i, j),
        owner: null
      });
    }
    row.push();
    board.push(row);
  }
  console.log(board);
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
  return [];
}

function isCorner(row, col) {
  return (
    (row === 0 && col === 0) ||
    (row === 0 && col === boardSize - 1) ||
    (row === boardSize - 1 && col === 0) ||
    (row === boardSize - 1 && col === boardSize - 1)
  );
}

function isEdge(row, col) {
  return (
    (row === 0 || row === boardSize - 1 || col === 0 || col === boardSize - 1) &&
    !isCorner(row, col)
  );
}

function isCenter(row, col) {
  return !isCorner(row, col) && !isEdge(row, col);
}

function handleCellClick(row, col , sub) {
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

function explodeCell(row, col) {
  const directions = [
    [-1, 0], [1, 0], [0, -1], [0, 1] // 上下左右四个方向
  ];
  const owner = board.value[row][col].owner;
  
  // 清空当前格子的所有子格子
  board.value[row][col].subCells = board.value[row][col].subCells.map(() => ({ owner: null }));
  board.value[row][col].owner = null;
  
  // 向四周扩散
  for (const [dx, dy] of directions) {
    const newRow = row + dx;
    const newCol = col + dy;
    
    // 检查是否在棋盘范围内
    if (newRow >= 0 && newRow < boardSize && newCol >= 0 && newCol < boardSize) {
      const targetCell = board.value[newRow][newCol];
      
      // 如果目标格子未被完全占领
      if (!targetCell.owner) {
        // 找到第一个未被占领的子格子
        const emptySubCellIndex = targetCell.subCells.findIndex(subCell => !subCell.owner);
        if (emptySubCellIndex !== -1) {
          // 占领子格子
          targetCell.subCells[emptySubCellIndex].owner = owner;
          
          // 检查是否填满所有子格子
          if (targetCell.subCells.every(subCell => subCell.owner)) {
            targetCell.owner = owner;
            explodeCell(newRow, newCol); // 递归爆炸
          }
        }
      }
    }
  }
  
  // 检查游戏是否结束
  checkGameEnd();
}

function checkGameEnd() {
  const owners = new Set();
  
  // 收集所有被占领的格子
  for (let i = 0; i < boardSize; i++) {
    for (let j = 0; j < boardSize; j++) {
      if (board.value[i][j].owner) {
        owners.add(board.value[i][j].owner);
      }
    }
  }
  
  // 如果只剩一个玩家，游戏结束
  if (owners.size === 1) {
    const winner = [...owners][0];
    alert(`游戏结束！${winner}获胜！`);
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
  width: 20px;
  height: 20px;
  margin: 2px;
  background-color: #ddd;
}

.sub-cell.player1 {
  background-color: #ff5252;
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

/* .center { */
/* } */
</style>