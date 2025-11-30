<template>
  <div :class="['game-container', theme]">
    <div :class="['game-layout', 'fade-in']">
      <!-- Боковая панель -->
      <div :class="['sidebar', theme]">
        <div class="sidebar-header">📜 История ходов</div>
        <div class="history-list" ref="historyList">
          <div class="history-message system">Игра началась!</div>
          <div 
            v-for="(move, index) in gameState.moveHistory" 
            :key="index"
            class="history-message"
          >
            {{ move.player }}: {{ move.original }} → {{ move.part1 }} и {{ move.part2 }}
          </div>
        </div>
      </div>

      <!-- Игровая область -->
      <div class="game-area">
        <!-- Верхняя панель -->
        <div :class="['top-panel', theme]">
          <button class="back-btn" @click="$emit('back')">⬅</button>
          
          <div class="players-info">
            <div :class="['player-name', { active: gameState.currentPlayer === 'player1' }]">
              {{ gameState.player1Name }}
            </div>
            <div class="vs">VS</div>
            <div :class="['player-name', { active: gameState.currentPlayer === 'player2', 'computer-turn': isComputerTurn }]">
              {{ gameState.player2Name }}
            </div>
          </div>
        </div>
        
        <!-- Статус -->
        <div :class="['status-label', theme, { 'computer-turn': isComputerTurn }]">
          Ход: {{ gameState.currentPlayer === 'player1' ? gameState.player1Name : gameState.player2Name }}
        </div>
        
        <!-- Игровое поле -->
        <div ref="gameCanvas" class="game-canvas">
          <div 
            v-for="(pile, index) in gameState.piles" 
            :key="`pile-${index}`"
            :class="['pile-container', { 
              'pile-invalid': pile < 3,
              'pile-selected': selectedPileIndex === index
            }]"
            @click="selectPile(index)"
          >
            <div class="chips-display">
              <div 
                v-for="(chip, chipIndex) in Math.min(pile, 50)" 
                :key="`chip-${index}-${chipIndex}`"
                :class="['chip', getChipClass(pile)]"
                :style="getChipStyle(chipIndex, Math.min(pile, 50))"
              >
              </div>
            </div>
            <div class="pile-platform"></div>
            <div class="pile-count">{{ pile }}</div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Диалог разделения -->
    <div v-if="showSplitDialog" class="modal-overlay" @click="closeSplitDialog">
      <div :class="['split-dialog', theme]" @click.stop>
        <h2 class="dialog-title">Разделите кучку</h2>
        
        <div class="info-box">
          <div class="info-text">Всего фишек: {{ currentPileSize }}</div>
        </div>
        
        <div class="split-label">Размер первой части:</div>
        
        <div class="split-preview">
          <div class="split-part">{{ part1 }}</div>
          <div class="split-and">и</div>
          <div class="split-part">{{ part2 }}</div>
        </div>
        
        <div :class="['split-error', { valid: part1 !== part2 }]">
          {{ part1 === part2 ? '⚠ Части должны быть неравными!' : '✓ Допустимое деление' }}
        </div>
        
        <input 
          v-model.number="part1" 
          type="range" 
          :min="1" 
          :max="currentPileSize - 1" 
          class="split-slider"
        />
        
        <div class="dialog-buttons">
          <button class="dialog-btn confirm" @click="confirmSplit" :disabled="part1 === part2">
            ✓ Разделить
          </button>
          <button class="dialog-btn cancel" @click="closeSplitDialog">
            ✗ Отмена
          </button>
        </div>
      </div>
    </div>

    <!-- History Button for Mobile -->
    <button class="history-btn-mobile" @click="toggleHistoryOverlay">
      История ходов
    </button>

    <!-- History Overlay -->
    <div v-if="showHistoryOverlay" class="history-overlay" @click="toggleHistoryOverlay">
      <div :class="['history-overlay-content', theme]" @click.stop>
        <div class="sidebar-header">📜 История ходов</div>
        <div class="history-list">
          <div class="history-message system">Игра началась!</div>
          <div
            v-for="(move, index) in gameState.moveHistory"
            :key="index"
            class="history-message"
          >
            {{ move.player }}: {{ move.original }} → {{ move.part1 }} и {{ move.part2 }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch, nextTick } from 'vue'

export default {
  name: 'GameScreen',
  props: {
    theme: String,
    gameState: Object
  },
  emits: ['split-pile', 'back'],
  setup(props, { emit }) {
    const selectedPileIndex = ref(null)
    const showSplitDialog = ref(false)
    const part1 = ref(1)
    const historyList = ref(null)
    const showHistoryOverlay = ref(false)

    const isComputerTurn = computed(() => {
        return props.gameState.currentPlayer === 'player2' && props.gameState.player2Name === 'Компьютер';
    });

    const toggleHistoryOverlay = () => {
      showHistoryOverlay.value = !showHistoryOverlay.value
    }
    
    const currentPileSize = computed(() => {
      if (selectedPileIndex.value !== null) {
        return props.gameState.piles[selectedPileIndex.value]
      }
      return 0
    })
    
    const part2 = computed(() => currentPileSize.value - part1.value)
    
    watch(() => props.gameState.moveHistory.length, async () => {
      await nextTick()
      if (historyList.value) {
        historyList.value.scrollTop = historyList.value.scrollHeight
      }
    })
    
    const getChipClass = (pileSize) => {
      if (pileSize < 3) return 'chip-invalid'
      return 'chip-normal'
    }
    
    const getChipStyle = (index, total) => {
      const row = Math.floor(index / 8)
      const col = index % 8
      const offsetX = col * 18 - (Math.min(total, 8) * 18) / 2
      const offsetY = -row * 18
      
      return {
        transform: `translate(${offsetX}px, ${offsetY}px)`
      }
    }
    
    const selectPile = (index) => {
      if (props.gameState.animating) return
      if (props.gameState.currentPlayer === 'player2' && props.gameState.player2Name === 'Компьютер') {
  return
}
      
      const pile = props.gameState.piles[index]
      if (pile < 3) {
        alert('Эту кучку нельзя разделить на две неравные части!\nМинимум должно быть 3 фишки.')
        return
      }
      
      selectedPileIndex.value = index
      part1.value = Math.floor(pile / 2)
      showSplitDialog.value = true
    }
    
    const closeSplitDialog = () => {
      document.activeElement.blur()
      showSplitDialog.value = false
      selectedPileIndex.value = null
    }
    
    const confirmSplit = () => {
      document.activeElement.blur()
      if (part1.value === part2.value) {
        alert('Части должны быть неравными!')
        return
      }
      
      emit('split-pile', {
        pileIndex: selectedPileIndex.value,
        part1: part1.value,
        part2: part2.value
      })
      
      closeSplitDialog()
    }
    
    return {
      selectedPileIndex,
      showSplitDialog,
      part1,
      part2,
      currentPileSize,
      historyList,
      showHistoryOverlay,
      toggleHistoryOverlay,
      getChipClass,
      getChipStyle,
      selectPile,
      closeSplitDialog,
      confirmSplit,
      isComputerTurn,
    }
  }
}
</script>

<style scoped>
.game-container {
  width: 100%;
  height: 100vh;
  overflow: hidden;
}

.game-container.dark {
  background-color: #1a1a2e;
}

.game-container.light {
  background-color: #f0f4f8;
}

.game-layout {
  display: flex;
  height: 100%;
}

.game-area {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-width: 0; /* Fix for flexbox overflow */
}

.top-panel {
  height: 80px;
  display: flex;
  align-items: center;
  padding: 0 20px;
  gap: 40px;
  flex-shrink: 0;
}

.top-panel.dark {
  background-color: #0f3460;
}

.top-panel.light {
  background-color: white;
}

.back-btn {
  font-size: 20px;
  background: #16213e;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s;
}

.light .back-btn {
  background: #e1e8ed;
  color: #1a1a2e;
}

.back-btn:hover {
  transform: scale(1.1);
}

.players-info {
  display: flex;
  align-items: center;
  gap: 20px;
  flex: 1;
  justify-content: center;
}

.player-name {
  font-size: 16px;
  font-weight: bold;
  color: #8b95a8;
  min-width: 150px;
  text-align: center;
  transition: all 0.3s;
}

.player-name.active {
  color: #4ecca3;
  transform: scale(1.1);
  animation: pulse 2s infinite;
}

.light .player-name {
  color: #5a6c7d;
}

.light .player-name.active {
  color: #10b981;
}

.player-name.active.computer-turn {
  color: #e94560;
}

.light .player-name.active.computer-turn {
  color: #e94560;
}

.vs {
  font-size: 20px;
  font-weight: bold;
  color: #8b95a8;
}

.status-label {
  font-size: 16px;
  font-weight: bold;
  text-align: center;
  padding: 15px;
  flex-shrink: 0;
}

.status-label.dark {
  color: #4ecca3;
}

.status-label.light {
  color: #10b981;
}

.status-label.computer-turn {
  color: #e94560;
}

.light .status-label.computer-turn {
  color: #e94560;
}

.game-canvas {
  flex: 1;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 30px;
  padding: 30px;
  overflow-y: auto;
}

.pile-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s;
  position: relative;
  padding: 20px;
}

.pile-container:hover {
  transform: scale(1.05);
}

.pile-container.pile-invalid {
  opacity: 0.5;
  cursor: not-allowed;
}

.pile-container.pile-selected {
  background: rgba(78, 204, 163, 0.1);
  border-radius: 15px;
}

.chips-display {
  position: relative;
  width: 150px;
  height: 120px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 10px;
}

.chip {
  position: absolute;
  width: 14px;
  height: 14px;
  border-radius: 50%;
  transition: all 0.3s;
}

.chip-normal {
  background: linear-gradient(135deg, #5e72e4, #4c63d2);
  border: 2px solid #4c63d2;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.light .chip-normal {
  background: linear-gradient(135deg, #3b82f6, #2563eb);
  border: 2px solid #2563eb;
}

.chip-invalid {
  background: linear-gradient(135deg, #ff6b6b, #ee5a52);
  border: 2px solid #ee5a52;
}

.pile-platform {
  width: 140px;
  height: 12px;
  background: #0f3460;
  border-radius: 6px;
  margin-bottom: 10px;
}

.light .pile-platform {
  background: #e1e8ed;
}

.pile-count {
  font-size: 24px;
  font-weight: bold;
  color: white;
}

.light .pile-count {
  color: #1a1a2e;
}

.sidebar {
  width: 300px;
  display: flex;
  flex-direction: column;
  flex-shrink: 0;
}

.sidebar.dark {
  background-color: #0f3460;
  border-left: 2px solid #1a1a2e;
}

.sidebar.light {
  background-color: white;
  border-left: 2px solid #e1e8ed;
}

.sidebar-header {
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #e94560;
  color: white;
  font-size: 14px;
  font-weight: bold;
  flex-shrink: 0;
}

.history-list {
  flex: 1;
  overflow-y: auto;
  padding: 10px;
}

.history-message {
  background: #16213e;
  color: white;
  padding: 10px;
  margin-bottom: 8px;
  border-radius: 8px;
  font-size: 10px;
  line-height: 1.4;
}

.light .history-message {
  background: #e1e8ed;
  color: #1a1a2e;
}

.history-message.system {
  background: transparent;
  color: #8b95a8;
  font-style: italic;
  padding: 5px 10px;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.split-dialog {
  width: 450px;
  padding: 30px;
  border-radius: 20px;
  animation: fadeIn 0.3s;
}

.split-dialog.dark {
  background: #0f3460;
}

.split-dialog.light {
  background: white;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.2);
}

.dialog-title {
  font-size: 20px;
  font-weight: bold;
  color: #e94560;
  text-align: center;
  margin-bottom: 20px;
}

.info-box {
  background: #16213e;
  padding: 10px;
  border-radius: 8px;
  margin-bottom: 20px;
}

.light .info-box {
  background: #e1e8ed;
}

.info-text {
  font-size: 14px;
  font-weight: bold;
  text-align: center;
  color: white;
}

.light .info-text {
  color: #1a1a2e;
}

.split-label {
  font-size: 12px;
  color: #8b95a8;
  text-align: center;
  margin-bottom: 10px;
}

.light .split-label {
  color: #5a6c7d;
}

.split-preview {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  margin-bottom: 10px;
}

.split-part {
  font-size: 20px;
  font-weight: bold;
  color: #4ecca3;
  min-width: 40px;
  text-align: center;
}

.light .split-part {
  color: #10b981;
}

.split-and {
  font-size: 16px;
  color: #8b95a8;
}

.split-error {
  font-size: 10px;
  color: #ff6b6b;
  text-align: center;
  margin-bottom: 15px;
  min-height: 20px;
}

.split-error.valid {
  color: #4ecca3;
}

.light .split-error.valid {
  color: #10b981;
}

.split-slider {
  width: 100%;
  height: 15px;
  margin-bottom: 20px;
  -webkit-appearance: none;
  background: #1a1a2e;
  border-radius: 8px;
  outline: none;
}

.light .split-slider {
  background: #e1e8ed;
}

.split-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 25px;
  height: 25px;
  background: #e94560;
  border-radius: 50%;
  cursor: pointer;
}

.split-slider::-moz-range-thumb {
  width: 25px;
  height: 25px;
  background: #e94560;
  border-radius: 50%;
  cursor: pointer;
  border: none;
}

.dialog-buttons {
  display: flex;
  gap: 10px;
}

.dialog-btn {
  flex: 1;
  font-size: 14px;
  font-weight: bold;
  padding: 12px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s;
}

.dialog-btn.confirm {
  background: #4ecca3;
  color: white;
}

.light .dialog-btn.confirm {
  background: #10b981;
}

.dialog-btn.confirm:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(78, 204, 163, 0.3);
}

.dialog-btn.confirm:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.dialog-btn.cancel {
  background: #ff6b6b;
  color: white;
}

.dialog-btn.cancel:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(255, 107, 107, 0.3);
}

.history-btn-mobile {
  display: none;
}

.history-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.8);
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
}

.history-overlay-content {
  width: 90%;
  height: 80%;
  border-radius: 10px;
  display: flex;
  flex-direction: column;
}

.history-overlay-content.dark {
  background-color: #0f3460;
}

.history-overlay-content.light {
  background-color: white;
}

.history-overlay-content .history-list {
  flex: 1;
  overflow-y: auto;
  padding: 10px;
}

@media (max-width: 768px) {
  .sidebar {
    display: none;
  }

  .history-btn-mobile {
    display: block;
    position: fixed;
    bottom: 20px;
    left: 50%;
    transform: translateX(-50%);
    padding: 10px 20px;
    background-color: #e94560;
    color: white;
    border: none;
    border-radius: 8px;
    font-size: 16px;
    font-weight: bold;
    cursor: pointer;
    z-index: 1001;
  }

  /* Hide back button */
  .back-btn {
    display: none;
  }

  /* Make top-panel sticky */
  .top-panel {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    z-index: 999; /* Ensure it's above game content */
    padding: 0 10px; /* Adjust padding for mobile */
    gap: 15px; /* Adjust gap for mobile */
    height: 60px; /* Adjust height for mobile */
  }

  /* Make status-label sticky */
  .status-label {
    position: fixed;
    top: 60px; /* Below the top-panel */
    left: 0;
    width: 100%;
    z-index: 999;
    padding: 10px; /* Adjust padding for mobile */
    height: 45px; /* Adjust height for mobile */
  }

  /* Adjust game-area padding to prevent content being hidden under fixed headers */
  .game-area {
    padding-top: 105px; /* top-panel height (60px) + status-label height (45px) */
  }
}
</style>