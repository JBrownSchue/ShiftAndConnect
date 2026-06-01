<template>
  <v-container class="text-center fill-height bg-deep-dark" fluid>
    <v-row justify="center" align="center" class="w-100">

      <v-col cols="12" class="mb-4 d-flex justify-center align-center">
        <v-btn icon="mdi-arrow-left" variant="text" color="grey" @click="router.push('/')" class="mr-4"></v-btn>

        <v-chip :color="currentPlayer === 1 ? '#00e5ff' : '#ff9800'" variant="outlined"
          class="text-subtitle-1 font-weight-bold px-6 py-5 player-chip">

          <span v-if="playerCount < 2 && gameMode !== 'solo'" class="d-flex align-center">
            <v-progress-circular indeterminate size="20" width="2" class="mr-3"></v-progress-circular>
            Warte auf Gegner ({{ playerCount }}/2)
          </span>

          <span v-else>
            Spieler {{ currentPlayer }} ist am Zug
          </span>

        </v-chip>
      </v-col>

      <v-col cols="12" class="d-flex justify-center">
        <div class="game-board-wrapper" :class="{ 'disabled-board': !isMyTurn && !winner, 'game-over-board': winner }">
          <div class="game-board">

            <div class="empty-corner"></div>
            <div v-for="col in 7" :key="'col-down-' + col" class="shift-btn">
              <v-btn icon="mdi-chevron-double-down" variant="outlined" class="cyber-btn" size="small"
                @click="shiftCol(col - 1, 'down')"></v-btn>
            </div>
            <div class="empty-corner"></div>

            <template v-for="(row, rowIndex) in board" :key="'row-' + rowIndex">

              <div class="shift-btn">
                <v-btn icon="mdi-chevron-double-right" variant="outlined" class="cyber-btn" size="small"
                  @click="shiftRow(rowIndex, 'right')"></v-btn>
              </div>

              <div v-for="(cell, colIndex) in row" :key="'cell-' + rowIndex + '-' + colIndex" class="cell" :class="{
                'anim-shift-right': shiftAnim.row === rowIndex && shiftAnim.direction === 'right',
                'anim-shift-left': shiftAnim.row === rowIndex && shiftAnim.direction === 'left',
                'anim-shift-down': shiftAnim.col === colIndex && shiftAnim.direction === 'down',
                'anim-shift-up': shiftAnim.col === colIndex && shiftAnim.direction === 'up',
              }" @click="placePiece(rowIndex, colIndex)">
                <div v-if="cell !== 0" class="piece" :class="{ 'player1': cell === 1, 'player2': cell === 2 }"></div>
              </div>

              <div class="shift-btn">
                <v-btn icon="mdi-chevron-double-left" variant="outlined" class="cyber-btn" size="small"
                  @click="shiftRow(rowIndex, 'left')"></v-btn>
              </div>
            </template>

            <div class="empty-corner"></div>
            <div v-for="col in 7" :key="'col-up-' + col" class="shift-btn">
              <v-btn icon="mdi-chevron-double-up" variant="outlined" class="cyber-btn" size="small"
                @click="shiftCol(col - 1, 'up')"></v-btn>
            </div>
            <div class="empty-corner"></div>

          </div>
        </div>
      </v-col>
    </v-row>
    <v-fade-transition>
      <div v-if="showWinDialog" class="win-banner bg-deep-dark cyber-dialog text-center pa-6">
        <v-card-title class="text-h4 font-weight-bold pt-2 pb-2 title-glow"
          :class="winner === 1 ? 'text-cyan' : 'text-orange'">
          SYSTEM OVERRIDE
        </v-card-title>
        <div class="d-flex justify-center align-center mb-4">
          <v-icon size="60" :color="winner === 1 ? '#00e5ff' : '#ff9800'" class="mr-4 trophy-glow">mdi-trophy</v-icon>
          <h2 class="text-h4 text-white">SPIELER {{ winner }} GEWINNT!</h2>
        </div>
        <v-btn size="x-large" variant="outlined" :color="winner === 1 ? '#00e5ff' : '#ff9800'" @click="returnToLobby">
          Feld verlassen
        </v-btn>
      </div>
    </v-fade-transition>
  </v-container>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from 'vue';
import { useRoute, useRouter } from 'vue-router';

const route = useRoute();
const router = useRouter();
const roomId = String(route.params.id).toUpperCase();
const gameMode = ref<string>('');

const storageKey = `shift_role_${roomId}`;

let storedRole = parseInt(localStorage.getItem(storageKey) || '0');

if (storedRole !== 1) {
  storedRole = 2;
  localStorage.setItem(storageKey, '2');
}

const myRole = ref<number>(storedRole);

const board = ref<number[][]>(Array.from({ length: 7 }, () => Array(7).fill(0)));
const currentPlayer = ref<number>(1);
const playerCount = ref<number>(0);
const winner = ref<number | null>(null);
const showWinDialog = ref<boolean>(false);

const isMyTurn = computed(() => {
  if (gameMode.value === 'solo') {
    return currentPlayer.value === 1;
  }

  return myRole.value === currentPlayer.value && playerCount.value >= 2;
});

const winningCells = ref<{ row: number, col: number }[]>([]);

const isWinningCell = (row: number, col: number) => {
  return winningCells.value.some(c => c.row === row && c.col === col);
};

const getWinningCells = (boardState: number[][], player: number) => {
  const dirs = [[0, 1], [1, 0], [1, 1], [1, -1]];
  for (let r = 0; r < 7; r++) {
    for (let c = 0; c < 7; c++) {
      if (boardState[r][c] !== player) continue;
      for (const [dr, dc] of dirs) {
        let count = 1;
        const cells = [{ row: r, col: c }];
        for (let i = 1; i < 5; i++) {
          const nr = r + dr * i;
          const nc = c + dc * i;
          if (nr >= 0 && nr < 7 && nc >= 0 && nc < 7 && boardState[nr][nc] === player) {
            count++;
            cells.push({ row: nr, col: nc });
          } else { break; }
        }
        if (count === 5) return cells; // 5 in einer Reihe gefunden!
      }
    }
  }
  return [];
};

interface ShiftAnimation {
  row: number;
  col: number;
  direction: string;
}
const shiftAnim = ref<ShiftAnimation>({ row: -1, col: -1, direction: '' });

let ws: WebSocket | null = null;

onMounted(() => {
  const protocol = window.location.protocol === 'https:' ? 'wss:' : 'ws:';
  ws = new WebSocket(`${protocol}//${window.location.host}/ws/games/${roomId}`);

  ws.onopen = () => {
    // console.log(`Erfolgreich mit Raum ${roomId} verbunden!`);
  };

  ws.onmessage = (event) => {
    const update = JSON.parse(event.data);

    board.value = update.board;
    currentPlayer.value = update.current_player;
    playerCount.value = update.player_count;

    if (update.game_mode) {
      gameMode.value = update.game_mode;
    }

    if (update.winner) {
      winner.value = update.winner;
      winningCells.value = getWinningCells(update.board, update.winner);

      setTimeout(() => {
        showWinDialog.value = true;
      }, 500);
    }
  };

  ws.onerror = (error) => {
    console.error("WebSocket Fehler:", error);
  };
});

onUnmounted(() => {
  if (ws) ws.close();
});

const placePiece = (row: number, col: number) => {
  if (!isMyTurn.value) return;
  if (board.value[row][col] === 0 && ws && ws.readyState === WebSocket.OPEN) {
    const payload = { action: 'place', row, col, direction: null, player: currentPlayer.value };
    ws.send(JSON.stringify(payload));
  }
};

const shiftRow = (rowIndex: number, direction: 'right' | 'left') => {
  if (!isMyTurn.value) return;
  if (ws && ws.readyState === WebSocket.OPEN) {
    triggerAnimation('row', rowIndex, direction);
    const payload = { action: 'shift_row', row: rowIndex, direction: direction, player: currentPlayer.value };
    ws.send(JSON.stringify(payload));
  }
};

const shiftCol = (colIndex: number, direction: 'up' | 'down') => {
  if (!isMyTurn.value) return;
  if (ws && ws.readyState === WebSocket.OPEN) {
    triggerAnimation('col', colIndex, direction);
    const payload = { action: 'shift_col', col: colIndex, direction: direction, player: currentPlayer.value };
    ws.send(JSON.stringify(payload));
  }
};

const triggerAnimation = (type: 'row' | 'col', index: number, direction: string) => {
  if (type === 'row') shiftAnim.value = { row: index, col: -1, direction };
  if (type === 'col') shiftAnim.value = { row: -1, col: index, direction };
  setTimeout(() => { shiftAnim.value = { row: -1, col: -1, direction: '' }; }, 300);
};

const returnToLobby = () => {
  showWinDialog.value = false;
  router.push('/');
};
</script>

<style scoped>
.bg-deep-dark {
  background-color: #121418;
  background-image: radial-gradient(circle at 50% 0%, #1f2532 0%, #121418 70%);
}

.game-board-wrapper {
  padding: 30px;
  background: linear-gradient(145deg, #252a35, #181a22);
  border-radius: 16px;
  box-shadow: 0 30px 60px rgba(0, 0, 0, 0.6), inset 0 1px 2px rgba(255, 255, 255, 0.1), inset 0 -1px 2px rgba(0, 0, 0, 0.5);
  border: 1px solid rgba(255, 255, 255, 0.05);
}

.game-board {
  display: grid;
  grid-template-columns: 45px repeat(7, 65px) 45px;
  grid-template-rows: 45px repeat(7, 65px) 45px;
  gap: 8px;
  align-items: center;
  justify-items: center;
}

.cell {
  width: 100%;
  height: 100%;
  background-color: #1e222b;
  border-radius: 8px;
  box-shadow: inset 0 3px 6px rgba(0, 0, 0, 0.4), 0 1px 0 rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(0, 229, 255, 0.05);
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  transition: background-color 0.2s ease, border-color 0.2s ease;
}

.cell:hover {
  background-color: #272c38;
  border-color: rgba(0, 229, 255, 0.3);
}

/* --- ANIMATIONEN --- */
.anim-shift-right {
  animation: slideGlowRight 0.3s ease-out;
}

.anim-shift-left {
  animation: slideGlowLeft 0.3s ease-out;
}

.anim-shift-down {
  animation: slideGlowDown 0.3s ease-out;
}

.anim-shift-up {
  animation: slideGlowUp 0.3s ease-out;
}

@keyframes slideGlowRight {
  0% {
    transform: translateX(-8px);
    box-shadow: inset 15px 0 15px -10px rgba(0, 229, 255, 0.8);
  }

  100% {
    transform: translateX(0);
    box-shadow: inset 0 0 0 0 rgba(0, 229, 255, 0);
  }
}

@keyframes slideGlowLeft {
  0% {
    transform: translateX(8px);
    box-shadow: inset -15px 0 15px -10px rgba(0, 229, 255, 0.8);
  }

  100% {
    transform: translateX(0);
    box-shadow: inset 0 0 0 0 rgba(0, 229, 255, 0);
  }
}

@keyframes slideGlowDown {
  0% {
    transform: translateY(-8px);
    box-shadow: inset 0 15px 15px -10px rgba(0, 229, 255, 0.8);
  }

  100% {
    transform: translateY(0);
    box-shadow: inset 0 0 0 0 rgba(0, 229, 255, 0);
  }
}

@keyframes slideGlowUp {
  0% {
    transform: translateY(8px);
    box-shadow: inset 0 -15px 15px -10px rgba(0, 229, 255, 0.8);
  }

  100% {
    transform: translateY(0);
    box-shadow: inset 0 0 0 0 rgba(0, 229, 255, 0);
  }
}

.piece {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  box-shadow: inset 0 -3px 6px rgba(0, 0, 0, 0.4);
  animation: popIn 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

@keyframes popIn {
  0% {
    transform: scale(0);
    opacity: 0;
  }

  100% {
    transform: scale(1);
    opacity: 1;
  }
}

.player1 {
  background: radial-gradient(circle at 30% 30%, #4dd0e1, #00acc1);
  box-shadow: 0 0 15px rgba(0, 229, 255, 0.6), inset 0 -3px 6px rgba(0, 0, 0, 0.4);
}

.player2 {
  background: radial-gradient(circle at 30% 30%, #ffb74d, #f57c00);
  box-shadow: 0 0 15px rgba(255, 152, 0, 0.6), inset 0 -3px 6px rgba(0, 0, 0, 0.4);
}

.cyber-btn {
  color: #00e5ff !important;
  border-color: rgba(0, 229, 255, 0.2) !important;
  background-color: rgba(0, 0, 0, 0.2);
  transition: all 0.3s ease;
  border-radius: 8px;
}

.cyber-btn:hover {
  background-color: rgba(0, 229, 255, 0.1);
  border-color: rgba(0, 229, 255, 0.8) !important;
  box-shadow: 0 0 12px rgba(0, 229, 255, 0.4);
  transform: scale(1.05);
}

.player-chip {
  background: rgba(0, 0, 0, 0.4);
  border: 1px solid rgba(255, 255, 255, 0.1);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.4);
  backdrop-filter: blur(5px);
}

.glow-dot {
  display: inline-block;
  width: 12px;
  height: 12px;
  border-radius: 50%;
  box-shadow: 0 0 10px currentColor;
  animation: pulse 1.5s infinite alternate;
}

@keyframes pulse {
  0% {
    opacity: 0.6;
    transform: scale(0.9);
  }

  100% {
    opacity: 1;
    transform: scale(1.1);
  }
}

.empty-corner {
  width: 100%;
  height: 100%;
}
</style>