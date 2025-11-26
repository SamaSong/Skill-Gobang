<script setup>
import { computed, ref } from 'vue'

defineOptions({
  name: 'SkillChessBoard',
})

// 棋盘固定为 15x15
const BOARD_SIZE = 15
const players = {
  black: { id: 'black', label: 'Black', stone: '⚫' },
  white: { id: 'white', label: 'White', stone: '⚪' },
}

// 存储棋盘、玩家和历史状态
const board = ref(createBoard())
const currentPlayer = ref(players.black.id)
const moveHistory = ref([])
const winner = ref(null)
const winningLine = ref([])

const totalCells = BOARD_SIZE * BOARD_SIZE
const columnLabels = Array.from({ length: BOARD_SIZE }, (_, idx) =>
  String.fromCharCode(65 + idx)
)

const rowLabels = Array.from({ length: BOARD_SIZE }, (_, idx) => idx + 1)

// 最近一步用于显示高亮
const lastMove = computed(() => {
  if (!moveHistory.value.length) return null
  return moveHistory.value[moveHistory.value.length - 1]
})

// 盘满且无赢家则判平局
const isDraw = computed(
  () => !winner.value && moveHistory.value.length === totalCells,
)

// 顶部提示语：胜者 > 平局 > 当前玩家
const statusText = computed(() => {
  if (winner.value) return `${players[winner.value].label} wins!`
  if (isDraw.value) return 'Match drawn — board is full.'
  return `${players[currentPlayer.value].label} to move`
})

// 创建新的空棋盘
function createBoard() {
  return Array.from({ length: BOARD_SIZE }, () =>
    Array(BOARD_SIZE).fill(null)
  )
}

// 落子逻辑：只有空格且未结束时才能落
function handleCellClick(rowIndex, colIndex) {
  if (winner.value || isDraw.value) return
  if (board.value[rowIndex][colIndex]) return

  const player = currentPlayer.value
  board.value[rowIndex][colIndex] = player
  moveHistory.value.push({ row: rowIndex, col: colIndex, player })

  const winningCoords = evaluateWin(rowIndex, colIndex, player)
  if (winningCoords) {
    winner.value = player
    winningLine.value = winningCoords
    return
  }

  if (moveHistory.value.length === totalCells) return
  togglePlayer()
}

// 黑白轮流
function togglePlayer() {
  currentPlayer.value =
    currentPlayer.value === players.black.id
      ? players.white.id
      : players.black.id
}

// 最近一步向四个方向扫描五连
function evaluateWin(row, col, player) {
  const directions = [
    [1, 0],
    [0, 1],
    [1, 1],
    [1, -1],
  ]

  for (const [dr, dc] of directions) {
    const coords = gatherInDirection(row, col, dr, dc, player)
    if (coords.length >= 5) return coords
  }

  return null
}

// 按方向扩展，收集连续的棋子坐标
function gatherInDirection(row, col, dr, dc, player) {
  const coords = [{ row, col }]

  let r = row + dr
  let c = col + dc

  while (isValidCell(r, c) && board.value[r][c] === player) {
    coords.push({ row: r, col: c })
    r += dr
    c += dc
  }

  r = row - dr
  c = col - dc

  while (isValidCell(r, c) && board.value[r][c] === player) {
    coords.unshift({ row: r, col: c })
    r -= dr
    c -= dc
  }

  return coords
}

// 坐标合法性校验
function isValidCell(row, col) {
  return row >= 0 && row < BOARD_SIZE && col >= 0 && col < BOARD_SIZE
}

// 恢复初始状态
function restartGame() {
  board.value = createBoard()
  moveHistory.value = []
  winner.value = null
  winningLine.value = []
  currentPlayer.value = players.black.id
}

// 撤销一步：弹出历史并回滚棋盘
function undoMove() {
  const last = moveHistory.value.pop()
  if (!last) return

  board.value[last.row][last.col] = null
  winner.value = null
  winningLine.value = []
  currentPlayer.value = last.player
}

function isWinningCell(row, col) {
  return winningLine.value.some(
    (coord) => coord.row === row && coord.col === col,
  )
}

// 判断是否为最新一步
function isLastMove(row, col) {
  if (!lastMove.value) return false
  return lastMove.value.row === row && lastMove.value.col === col
}

// 提供读屏友好的格子描述
function describeCell(rowIndex, colIndex) {
  const cell = board.value[rowIndex][colIndex]
  const stone = cell ? players[cell].stone : '·'
  const column = columnLabels[colIndex]
  const row = rowLabels[rowIndex]
  return `${stone} (${column}${row})`
}
</script>

<template>
  <section class="gobang">
    <header class="panel">
      <div>
        <p class="eyebrow">Skill Gobang</p>
        <h1>{{ statusText }}</h1>
      </div>
      <div class="panel__actions">
        <button class="ghost" type="button" @click="undoMove" :disabled="!moveHistory.length">
          Undo
        </button>
        <button class="primary" type="button" @click="restartGame" :disabled="!moveHistory.length && !winner">
          Restart
        </button>
      </div>
    </header>

    <div class="board-wrapper">
      <div class="board-labels board-labels--cols">
        <span v-for="label in columnLabels" :key="label">
          {{ label }}
        </span>
      </div>

      <div class="board-with-rows">
        <div class="board-labels board-labels--rows">
          <span v-for="label in rowLabels" :key="label">
            {{ label }}
          </span>
        </div>

        <div class="board" role="grid" aria-label="Gobang board">
          <div v-for="(row, rowIndex) in board" :key="`row-${rowIndex}`" class="board-row" role="row">
            <button
              v-for="(cell, colIndex) in row"
              :key="`cell-${rowIndex}-${colIndex}`"
              class="cell"
              :class="{
                'cell--black': cell === players.black.id,
                'cell--white': cell === players.white.id,
                'cell--last': isLastMove(rowIndex, colIndex),
                'cell--win': isWinningCell(rowIndex, colIndex),
              }"
              type="button"
              role="gridcell"
              :aria-label="describeCell(rowIndex, colIndex)"
              @click="handleCellClick(rowIndex, colIndex)"
            >
              <span v-if="cell === players.black.id">⚫</span>
              <span v-else-if="cell === players.white.id">⚪</span>
            </button>
          </div>
        </div>
      </div>
    </div>

    <section class="panel history">
      <header>
        <h2>Move log</h2>
        <p>{{ moveHistory.length }} / {{ totalCells }} moves</p>
      </header>
      <ol v-if="moveHistory.length">
        <li v-for="(move, index) in moveHistory" :key="`${move.row}-${move.col}-${index}`">
          <span class="move-index">#{{ index + 1 }}</span>
          <span>
            {{ players[move.player].label }}
            →
            {{ columnLabels[move.col] }}{{ rowLabels[move.row] }}
          </span>
        </li>
      </ol>
      <p v-else class="empty">No moves yet — click the board to begin.</p>
    </section>
  </section>
</template>

<style scoped lang="scss">
.gobang {
  display: grid;
  gap: 1.5rem;
  padding: 1.5rem;
  max-width: 960px;
  margin: 0 auto;
}

.panel {
  background: #111927;
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 1rem;
  padding: 1.25rem 1.5rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
  color: #f5f6fb;
}

.panel header {
  display: flex;
  justify-content: space-between;
  width: 100%;
}

.panel h1,
.panel h2 {
  margin: 0;
}

.panel .eyebrow {
  text-transform: uppercase;
  letter-spacing: 0.08em;
  font-size: 0.75rem;
  color: #8ea1c0;
  margin-bottom: 0.2rem;
}

.panel__actions {
  display: flex;
  gap: 0.75rem;
}

.panel button {
  cursor: pointer;
  border-radius: 999px;
  padding: 0.4rem 1.1rem;
  font-weight: 600;
  border: none;
  transition: opacity 0.2s ease, transform 0.2s ease;
}

.panel button:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.panel button:not(:disabled):active {
  transform: scale(0.96);
}

.panel button.primary {
  background: #ffb347;
  color: #111;
}

.panel button.ghost {
  background: rgba(255, 255, 255, 0.12);
  color: #f5f6fb;
}

.board-wrapper {
  background: radial-gradient(circle at top, #f3d092, #e0b873);
  border-radius: 1rem;
  padding: 1rem;
  box-shadow: inset 0 0 30px rgba(0, 0, 0, 0.35);
}

.board-with-rows {
  display: flex;
}

.board {
  display: grid;
  grid-template-rows: repeat(15, 36px);
  grid-template-columns: repeat(15, 36px);
  gap: 0;
  margin-left: 0.5rem;
  border: 2px solid rgba(0, 0, 0, 0.4);
  border-radius: 0.5rem;
  background-image: linear-gradient(#b8894c 1px, transparent 1px),
    linear-gradient(90deg, #b8894c 1px, transparent 1px);
  background-size: 36px 36px;
  position: relative;
}

.board-row {
  display: contents;
}

.cell {
  width: 36px;
  height: 36px;
  background: transparent;
  border: none;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 22px;
  color: #000;
  cursor: pointer;
}

.cell::after {
  content: '';
  position: absolute;
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: transparent;
  transition: background 0.2s ease, transform 0.2s ease;
}

.cell--black span,
.cell--white span {
  pointer-events: none;
}

.cell--black span {
  filter: drop-shadow(0 2px 2px rgba(0, 0, 0, 0.5));
}

.cell--white span {
  filter: drop-shadow(0 2px 2px rgba(0, 0, 0, 0.35));
}

.cell--last::after {
  background: rgba(255, 255, 255, 0.9);
  transform: scale(0.7);
}

.cell--win {
  box-shadow: inset 0 0 0 2px rgba(255, 203, 71, 0.9);
  border-radius: 50%;
}

.board-labels {
  display: grid;
  gap: 0.1rem;
  color: #5e4623;
  font-weight: 600;
}

.board-labels--cols {
  grid-template-columns: repeat(15, 36px);
  justify-content: center;
  margin-bottom: 0.4rem;
}

.board-labels--rows {
  grid-template-rows: repeat(15, 36px);
  align-content: center;
  justify-items: end;
  padding-right: 0.4rem;
}

.history {
  flex-direction: column;
  align-items: flex-start;
  gap: 0.75rem;
}

.history header {
  align-items: flex-end;
  color: #f5f6fb;
}

.history ol {
  width: 100%;
  margin: 0;
  padding-left: 1.2rem;
  display: grid;
  gap: 0.35rem;
  max-height: 200px;
  overflow: auto;
}

.history .move-index {
  font-variant-numeric: tabular-nums;
  font-weight: 600;
  color: #8ea1c0;
  margin-right: 0.4rem;
}

.history .empty {
  color: #8ea1c0;
}

@media (max-width: 768px) {
  .gobang {
    padding: 1rem;
  }

  .panel,
  .history {
    flex-direction: column;
    align-items: flex-start;
  }

  .board-wrapper {
    overflow-x: auto;
  }
}
</style>
