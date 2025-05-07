<template>
  <div class="game-container">
    <h1>爆炸棋</h1>
    <ExplosionChess />
  </div>
</template>

<script setup>
import ExplosionChess from './components/ExplosionChess.vue'
</script>

<style>
.game-container {
  max-width: 800px;
  margin: 0 auto;
  text-align: center;
}
</style>