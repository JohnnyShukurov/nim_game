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
        :game-mode="gameMode"
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

    // ============ НОВЫЙ АЛГОРИТМ ДЛЯ СЛОЖНОГО УРОВНЯ ============
    
    // Генерация всех возможных ходов для текущей позиции
    const generateAllMoves = (piles) => {
      const moves = []
      piles.forEach((pileSize, pileIndex) => {
        if (pileSize >= 3) {
          // Генерируем все возможные неравные разбиения
          for (let part1 = 1; part1 < pileSize; part1++) {
            const part2 = pileSize - part1
            if (part1 !== part2) {
              moves.push({ pileIndex, part1, part2 })
            }
          }
        }
      })
      return moves
    }

    // Применение хода и получение новой позиции
    const applyMove = (piles, move) => {
      const newPiles = [...piles]
      newPiles.splice(move.pileIndex, 1)
      newPiles.push(move.part1, move.part2)
      return newPiles.sort((a, b) => b - a)
    }

    // Эвристическая оценка позиции
    const evaluatePosition = (piles) => {
      const validPiles = piles.filter(p => p >= 3)
      
      // Если нет ходов - проигрышная позиция
      if (validPiles.length === 0) {
        return -10000
      }

      let score = 0
      
      // 1. Количество возможных ходов (чем больше - тем лучше)
      const moveCount = generateAllMoves(piles).length
      score += moveCount * 50
      
      // 2. Количество кучек (больше кучек = больше вариантов)
      score += piles.length * 30
      
      // 3. Средний размер кучек (меньше - лучше, больше контроля)
      const avgSize = piles.reduce((sum, p) => sum + p, 0) / piles.length
      score -= avgSize * 10
      
      // 4. Бонус за разнообразие размеров кучек
      const uniqueSizes = new Set(piles).size
      score += uniqueSizes * 20
      
      // 5. Штраф за большие кучки (сложнее контролировать)
      const maxPile = Math.max(...piles)
      if (maxPile > 10) {
        score -= (maxPile - 10) * 5
      }
      
      return score
    }

    // Минимакс с альфа-бета отсечением
    const minimax = (piles, depth, alpha, beta, isMaximizing) => {
      // Проверка окончания игры
      const validMoves = generateAllMoves(piles)
      if (validMoves.length === 0) {
        // Если нет ходов, текущий игрок проигрывает
        return isMaximizing ? -10000 : 10000
      }
      
      // Достигнута максимальная глубина - оцениваем позицию
      if (depth === 0) {
        return evaluatePosition(piles)
      }

      if (isMaximizing) {
        // Ход компьютера - максимизируем оценку
        let maxEval = -Infinity
        
        for (const move of validMoves) {
          const newPiles = applyMove(piles, move)
          const evalScore = minimax(newPiles, depth - 1, alpha, beta, false)
          maxEval = Math.max(maxEval, evalScore)
          alpha = Math.max(alpha, evalScore)
          
          // Альфа-бета отсечение
          if (beta <= alpha) {
            break
          }
        }
        
        return maxEval
      } else {
        // Ход противника - минимизируем оценку
        let minEval = Infinity
        
        for (const move of validMoves) {
          const newPiles = applyMove(piles, move)
          const evalScore = minimax(newPiles, depth - 1, alpha, beta, true)
          minEval = Math.min(minEval, evalScore)
          beta = Math.min(beta, evalScore)
          
          // Альфа-бета отсечение
          if (beta <= alpha) {
            break
          }
        }
        
        return minEval
      }
    }

    // Выбор лучшего хода с помощью минимакса
    const findBestMove = (piles) => {
      const moves = generateAllMoves(piles)
      if (moves.length === 0) return null

      // Определяем глубину поиска в зависимости от количества фишек
      const totalChips = piles.reduce((sum, p) => sum + p, 0)
      let maxDepth
      
      if (totalChips <= 15) {
        maxDepth = 5  // Малое количество - глубокий поиск
      } else if (totalChips <= 30) {
        maxDepth = 4  // Среднее количество
      } else if (totalChips <= 50) {
        maxDepth = 3  // Много фишек - ограничиваем глубину
      } else {
        maxDepth = 2  // Очень много - минимальная глубина
      }

      let bestMove = null
      let bestScore = -Infinity

      for (const move of moves) {
        const newPiles = applyMove(piles, move)
        const score = minimax(newPiles, maxDepth, -Infinity, Infinity, false)
        
        if (score > bestScore) {
          bestScore = score
          bestMove = move
        }
      }

      return bestMove
    }

    // ============ ОСНОВНАЯ ЛОГИКА ХОДА КОМПЬЮТЕРА ============
    
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
        // ЛЕГКИЙ: Полностью случайная игра
        // 1. Выбираем случайную кучку из всех доступных (≥3)
        const pile = validPiles[Math.floor(Math.random() * validPiles.length)]
        pileIndex = pile.index
        pileSize = pile.size
        
        // 2. Делим на две случайные неравные части
        // Генерируем все возможные разбиения
        const allSplits = []
        for (let i = 1; i < pileSize; i++) {
          if (i !== pileSize - i) {
            allSplits.push([i, pileSize - i])
          }
        }
        
        // Выбираем любое случайное разбиение
        const randomSplit = allSplits[Math.floor(Math.random() * allSplits.length)]
        part1 = randomSplit[0]
        part2 = randomSplit[1]
        
      } else if (difficulty.value === 'medium') {
        // СРЕДНИЙ: Стратегия с базовой тактикой
        
        // 1. Приоритет: делим самую большую кучку (захват контроля)
        const largestPile = validPiles.reduce((max, p) => p.size > max.size ? p : max)
        pileIndex = largestPile.index
        pileSize = largestPile.size
        
        // 2. Генерируем все возможные разбиения
        const allSplits = []
        for (let i = 1; i < pileSize; i++) {
          if (i !== pileSize - i) {
            allSplits.push([i, pileSize - i])
          }
        }
        
        // 3. Фильтруем "хорошие" ходы (избегаем дубликатов существующих кучек)
        const goodSplits = allSplits.filter(
          ([p1, p2]) => !gameState.piles.includes(p1) && !gameState.piles.includes(p2)
        )
        
        // 4. Если есть хорошие ходы - выбираем среди них
        // Иначе берём любой допустимый
        let chosenSplit
        if (goodSplits.length > 0) {
          // Среди хороших ходов выбираем тот, что создаёт больше разнообразия
          // Предпочитаем делить на более мелкие части
          chosenSplit = goodSplits.sort((a, b) => {
            const diffA = Math.abs(a[0] - a[1])
            const diffB = Math.abs(b[0] - b[1])
            return diffB - diffA // Больше разница = лучше
          })[0]
        } else {
          // Нет "хороших" - берём случайный
          chosenSplit = allSplits[Math.floor(Math.random() * allSplits.length)]
        }
        
        part1 = chosenSplit[0]
        part2 = chosenSplit[1]
        
      } else {
        // Сложный: минимакс с эвристикой
        const bestMove = findBestMove(gameState.piles)
        
        if (bestMove) {
          pileIndex = bestMove.pileIndex
          part1 = bestMove.part1
          part2 = bestMove.part2
        } else {
          // Запасной вариант (не должно произойти)
          const pile = validPiles[0]
          pileIndex = pile.index
          pileSize = pile.size
          part1 = 1
          part2 = pileSize - 1
        }
      }

      splitPile({ pileIndex, part1, part2 })
    }

    return {
      currentScreen,
      theme,
      gameMode,
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
  overflow: hidden;
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