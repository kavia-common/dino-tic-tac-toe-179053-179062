<script setup lang="ts">
import './styles/theme.css'
import TicTacToeBoard from './components/TicTacToeBoard.vue'
</script>

<template>
  <div class="app-shell">
    <main class="center">
      <TicTacToeBoard />
    </main>
    <footer class="center" style="margin-top: 20px;">
      <small style="color:#6b7280">Built with Ocean Professional theme • 🦖</small>
    </footer>
  </div>
</template>

<style scoped>
/* No additional scoped styles; layout handled by theme.css and component */
</style>
