<template>
  <div :class="['app-container', theme]">
    <transition name="screen-fade" mode="out-in">
      <MainMenu 
        v-if="currentScreen === 'menu'"
        :theme="theme"
        @toggle-theme="toggleTheme"
        @select-pvc="currentScreen = 'difficulty'"
        @select-pvp="currentScreen = 'names'"
      />
      
      <DifficultySelect
        v-else-if="currentScreen === 'difficulty'"
        :theme="theme"
        @select="selectDifficulty"
        @back="currentScreen = 'menu'"
      />
      
      <PlayerNames
        v-else-if="currentScreen === 'names'"
        :theme="theme"
        @continue="setupPvP"
        @back="currentScreen = 'menu'"
      />
      
      <ChipsInput
        v-else-if="currentScreen === 'chips'"
        :theme="theme"
        @start="startGame"
        @back="currentScreen = 'menu'"
      />
      
      <GameScreen
        v-else-if="currentScreen === 'game'"
        :theme="theme"
        :game-state="gameState"
        @split-pile="splitPile"
        @back="currentScreen = 'menu'"
      />
      
      <GameOver
        v-else-if="currentScreen === 'gameover'"
        :theme="theme"
        :winner="winner"
        :winnerName="winnerName"
        :piles="gameState.piles"
        :move-count="gameState.moveHistory.length"
        @replay="currentScreen = 'chips'"
        @menu="currentScreen = 'menu'"
      />
    </transition>
  </div>
</template>

<script>
import { reactive, ref, computed } from 'vue'
import MainMenu from './components/MainMenu.vue'
import DifficultySelect from './components/DifficultySelect.vue'
import PlayerNames from './components/PlayerNames.vue'
import ChipsInput from './components/ChipsInput.vue'
import GameScreen from './components/GameScreen.vue'
import GameOver from './components/GameOver.vue'

export default {
  name: 'App',
  components: {
    MainMenu,
    DifficultySelect,
    PlayerNames,
    ChipsInput,
    GameScreen,
    GameOver
  },
  setup() {
    const currentScreen = ref('menu')
    const theme = ref('dark')
    const gameMode = ref(null)
    const difficulty = ref(null)
    const winner = ref(null)
    
    const gameState = reactive({
      piles: [],
      currentPlayer: 'player1',
      player1Name: 'Игрок 1',
      player2Name: 'Игрок 2',
      moveHistory: [],
      selectedPile: null,
      animating: false
    })

    const winnerName = computed(() => {
      if (winner.value === 'player1') {
        return gameState.player1Name
      } else if (winner.value === 'player2') {
        return gameState.player2Name
      }
      return ''
    })

    const toggleTheme = () => {
      theme.value = theme.value === 'dark' ? 'light' : 'dark'
    }

    const selectDifficulty = (diff) => {
      difficulty.value = diff
      gameMode.value = 'pvc'
      gameState.player2Name = 'Компьютер'
      currentScreen.value = 'chips'
    }

    const setupPvP = ({ player1, player2 }) => {
      gameMode.value = 'pvp'
      gameState.player1Name = player1
      gameState.player2Name = player2
      currentScreen.value = 'chips'
    }

    const startGame = (chips) => {
      gameState.piles = [chips]
      gameState.currentPlayer = 'player1'
      gameState.moveHistory = []
      gameState.selectedPile = null
      currentScreen.value = 'game'
    }

    const splitPile = async ({ pileIndex, part1, part2 }) => {
      if (gameState.animating) return
      
      gameState.animating = true
      const pileSize = gameState.piles[pileIndex]
      
      // Добавляем в историю
      const playerName = gameState.currentPlayer === 'player1' 
        ? gameState.player1Name 
        : gameState.player2Name
      gameState.moveHistory.push({
        player: playerName,
        original: pileSize,
        part1,
        part2
      })
      
      // Выполняем разделение
      gameState.piles.splice(pileIndex, 1)
      gameState.piles.push(part1, part2)
      gameState.piles.sort((a, b) => b - a)
      
      await new Promise(resolve => setTimeout(resolve, 500))
      gameState.animating = false
      
      // Проверка окончания игры
      const hasValidMoves = gameState.piles.some(p => p >= 3)
      if (!hasValidMoves) {
        winner.value = gameState.currentPlayer
        setTimeout(() => {
          currentScreen.value = 'gameover'
        }, 500)
        return
      }
      
      // Переключение хода
      gameState.currentPlayer = gameState.currentPlayer === 'player1' ? 'player2' : 'player1'
      
      // Ход компьютера
      if (gameMode.value === 'pvc' && gameState.currentPlayer === 'player2') {
        setTimeout(() => {
          makeComputerMove()
        }, 1200)
      }
    }

    const makeComputerMove = () => {
      const validPiles = gameState.piles
        .map((size, index) => ({ index, size }))
        .filter(p => p.size >= 3)
      
      if (validPiles.length === 0) {
        winner.value = 'player1'
        currentScreen.value = 'gameover'
        return
      }

      let pileIndex, pileSize, part1, part2

      if (difficulty.value === 'easy') {
        // Легкий: случайный ход
        const pile = validPiles[Math.floor(Math.random() * validPiles.length)]
        pileIndex = pile.index
        pileSize = pile.size
        part1 = Math.floor(Math.random() * (pileSize - 1)) + 1
        while (part1 === pileSize - part1) {
          part1 = Math.floor(Math.random() * (pileSize - 1)) + 1
        }
        part2 = pileSize - part1
      } else if (difficulty.value === 'medium') {
        // Средний: избегаем дубликатов
        const pile = validPiles[Math.floor(Math.random() * validPiles.length)]
        pileIndex = pile.index
        pileSize = pile.size
        
        const possibleSplits = []
        for (let i = 1; i < pileSize; i++) {
          if (i !== pileSize - i) {
            possibleSplits.push([i, pileSize - i])
          }
        }
        
        const goodSplits = possibleSplits.filter(
          ([p1, p2]) => !gameState.piles.includes(p1) && !gameState.piles.includes(p2)
        )
        
        const split = goodSplits.length > 0
          ? goodSplits[Math.floor(Math.random() * goodSplits.length)]
          : possibleSplits[Math.floor(Math.random() * possibleSplits.length)]
        
        part1 = split[0]
        part2 = split[1]
      } else {
        // Сложный: оптимальная стратегия
        let nimSum = 0
        gameState.piles.forEach(size => {
          nimSum ^= size
        })
        
        let moveFound = false
        
        if (nimSum !== 0) {
          for (const pile of validPiles) {
            const target = pile.size ^ nimSum
            if (target < pile.size && target > 0) {
              const other = pile.size - target
              if (target !== other) {
                pileIndex = pile.index
                pileSize = pile.size
                part1 = target
                part2 = other
                moveFound = true
                break
              }
            }
          }
        }
        
        if (!moveFound) {
          const pile = validPiles.reduce((max, p) => p.size > max.size ? p : max)
          pileIndex = pile.index
          pileSize = pile.size
          part1 = Math.floor(pileSize / 3) || 1
          part2 = pileSize - part1
          
          while (part1 === part2) {
            part1++
            part2 = pileSize - part1
          }
        }
      }

      splitPile({ pileIndex, part1, part2 })
    }

    return {
      currentScreen,
      theme,
      gameState,
      winner,
      winnerName,
      toggleTheme,
      selectDifficulty,
      setupPvP,
      startGame,
      splitPile
    }
  }
}
</script>

<style scoped>
.app-container {
  width: 100%;
  height: 100%;
  transition: background-color 0.3s;
}

.app-container.dark {
  background-color: #1a1a2e;
  color: white;
}

.app-container.light {
  background-color: #f0f4f8;
  color: #1a1a2e;
}

.screen-fade-enter-active,
.screen-fade-leave-active {
  transition: opacity 0.3s ease, transform 0.3s ease;
}

.screen-fade-enter-from,
.screen-fade-leave-to {
  opacity: 0;
  transform: translateY(20px);
}
</style>