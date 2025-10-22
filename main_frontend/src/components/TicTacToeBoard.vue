<script setup lang="ts">
import { computed, reactive, ref, watch } from 'vue'

type Player = 'X' | 'O' | ''
type Winner = Player | 'draw' | ''

// Game state (board, players, scores)
const state = reactive({
  board: Array<Player>(9).fill(''),
  currentPlayer: 'X' as Player,
  winner: '' as Winner,
  isDraw: false,
  scores: {
    X: 0,
    O: 0
  }
})

// Winning combinations
const winLines: number[][] = [
  [0,1,2],[3,4,5],[6,7,8],
  [0,3,6],[1,4,7],[2,5,8],
  [0,4,8],[2,4,6]
]

// PUBLIC_INTERFACE
function resetBoard(): void {
  /** Reset only the board, keep scores. */
  state.board = Array<Player>(9).fill('')
  state.currentPlayer = 'X'
  state.winner = ''
  state.isDraw = false
}

// PUBLIC_INTERFACE
function newGame(): void {
  /** Reset board and scores for a full new game session. */
  resetBoard()
  state.scores.X = 0
  state.scores.O = 0
}

const gameOver = computed(() => !!state.winner || state.isDraw)

function checkWinner(): Winner {
  for (const [a, b, c] of winLines) {
    const v = state.board[a]
    if (v && v === state.board[b] && v === state.board[c]) {
      return v
    }
  }
  if (state.board.every(cell => cell !== '')) {
    return 'draw'
  }
  return ''
}

// PUBLIC_INTERFACE
function makeMove(index: number): void {
  /**
   * Handle a cell click. Places the current player's mark if valid
   * and advances the game state, detecting wins/draws.
   */
  if (state.board[index] !== '' || gameOver.value) return
  state.board[index] = state.currentPlayer

  const result = checkWinner()
  if (result === 'X' || result === 'O') {
    state.winner = result
    state.scores[result] += 1
  } else if (result === 'draw') {
    state.isDraw = true
  } else {
    state.currentPlayer = state.currentPlayer === 'X' ? 'O' : 'X'
  }
}

// Accessible status string
const statusText = computed(() => {
  if (state.winner === 'X' || state.winner === 'O') {
    return `Player ${state.winner} wins!`
  }
  if (state.isDraw) {
    return 'It’s a draw.'
  }
  return `Player ${state.currentPlayer}'s turn`
})

// subtle announce on changes for assistive tech
const liveMessage = ref(statusText.value)
watch(statusText, (v) => { liveMessage.value = v })

// Cell aria-labels
function cellLabel(index: number): string {
  const value = state.board[index]
  if (value) return `Cell ${index + 1}: ${value}`
  return `Cell ${index + 1}: empty`
}

// derived for highlighting win cells
const winningLine = computed<number[] | null>(() => {
  for (const line of winLines) {
    const [a, b, c] = line
    const v = state.board[a]
    if (v && v === state.board[b] && v === state.board[c]) return line
  }
  return null
})
</script>

<template>
  <div class="game-wrapper stack">
    <div class="stack center">
      <img class="dino-top" src="@/assets/dino.svg" alt="" aria-hidden="true" />
      <div class="surface-card dino-wrap">
        <div class="dino-ring" aria-hidden="true"></div>
        <div class="stack header">
          <h1 class="title">Dino Tic Tac Toe</h1>
          <p class="subtitle">A playful duel in the land of Xs and Os</p>

          <div class="row info">
            <span class="badge primary" :aria-label="statusText">{{ statusText }}</span>
            <span class="badge success">X Wins: {{ state.scores.X }}</span>
            <span class="badge success">O Wins: {{ state.scores.O }}</span>
          </div>
        </div>

        <div class="board" role="grid" aria-label="Tic Tac Toe board">
          <button
            v-for="i in 9" :key="i"
            class="cell"
            :class="{
              x: state.board[i-1] === 'X',
              o: state.board[i-1] === 'O',
              win: winningLine && winningLine.includes(i-1),
              disabled: gameOver || state.board[i-1] !== '',
            }"
            role="gridcell"
            :aria-label="cellLabel(i-1)"
            :aria-disabled="gameOver || state.board[i-1] !== ''"
            @click="makeMove(i-1)"
          >
            <span class="mark" aria-hidden="true">{{ state.board[i-1] }}</span>
          </button>
        </div>

        <div class="row actions">
          <button class="button" @click="resetBoard" :disabled="!gameOver">Restart Round</button>
          <button class="button secondary" @click="newGame">New Game</button>
        </div>

        <div class="sr-live" aria-live="polite" aria-atomic="true">{{ liveMessage }}</div>
      </div>
      <img class="dino-bottom" src="@/assets/dino.svg" alt="" aria-hidden="true" />
    </div>
  </div>
</template>

<style scoped>
.game-wrapper {
  width: 100%;
}

/* Dino images positioning */
.dino-top, .dino-bottom {
  width: 96px;
  height: auto;
  opacity: 0.9;
  filter: drop-shadow(0 6px 12px rgba(17, 24, 39, 0.12));
  transition: transform 220ms ease;
}

.dino-top:hover, .dino-bottom:hover {
  transform: translateY(-2px) scale(1.02);
}

/* Header and info */
.header {
  gap: 8px;
  margin-bottom: 18px;
}

.info {
  gap: 10px;
}

/* Board styles */
.board {
  position: relative;
  display: grid;
  grid-template-columns: repeat(3, 96px);
  grid-template-rows: repeat(3, 96px);
  gap: 10px;
  padding: 8px;
  background: linear-gradient(180deg, rgba(255,255,255,0.9), rgba(255,255,255,1));
  border-radius: 18px;
  box-shadow: inset 0 1px 0 rgba(255,255,255,0.6), var(--shadow-md);
  z-index: 1;
}

@media (min-width: 480px) {
  .board {
    grid-template-columns: repeat(3, 110px);
    grid-template-rows: repeat(3, 110px);
  }
}

@media (min-width: 768px) {
  .board {
    grid-template-columns: repeat(3, 130px);
    grid-template-rows: repeat(3, 130px);
  }
}

.cell {
  appearance: none;
  border: 1px solid rgba(17, 24, 39, 0.08);
  border-radius: 14px;
  background: var(--color-surface);
  box-shadow: var(--shadow-sm);
  outline: none;
  cursor: pointer;
  display: grid;
  place-items: center;
  transition:
    transform 120ms ease,
    box-shadow 180ms ease,
    background 180ms ease,
    border-color 180ms ease;
}

.cell:hover {
  transform: translateY(-1px);
  box-shadow: var(--shadow-md);
  border-color: rgba(37, 99, 235, 0.25);
}

.cell:active {
  transform: translateY(0);
  box-shadow: var(--shadow-sm);
}

.cell:focus-visible {
  box-shadow: var(--shadow-md), var(--focus-ring);
}

/* Marks */
.mark {
  font-size: 42px;
  font-weight: 800;
  letter-spacing: -0.02em;
  transition: transform 150ms ease, color 180ms ease;
  color: var(--color-text);
}

.cell.x .mark { color: var(--color-primary); }
.cell.o .mark { color: var(--color-secondary); }

/* Win highlight */
.cell.win {
  background: linear-gradient(180deg, rgba(37,99,235,0.08), rgba(245,158,11,0.08));
  border-color: rgba(37, 99, 235, 0.35);
  animation: pulse 1000ms ease-in-out infinite alternate;
}

@keyframes pulse {
  from { transform: translateY(-1px); }
  to { transform: translateY(0px); }
}

.cell.disabled {
  cursor: default;
  opacity: 0.9;
}

/* Actions */
.actions {
  margin-top: 16px;
}

/* Screen reader live region */
.sr-live {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
</style>
