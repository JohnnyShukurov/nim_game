<template>
  <div :class="['gameover-container', theme]">
    <canvas ref="confettiCanvas" class="confetti-canvas"></canvas>
    
    <div class="gameover-content fade-in pulse">
      <h1 class="trophy">{{ winner === 'player1' ? '🏆' : '🤖' }} ПОБЕДА! {{ winner === 'player1' ? '🏆' : '🤖' }}</h1>
      
      <div class="winner-name">{{ winnerName }}</div>
      
      <p class="winner-text">выигрывает игру!</p>
      
      <div class="stats-box">
        <div class="stats-title">Итоговая позиция:</div>
        <div class="piles-text">{{ pilesText }}</div>
        <div class="move-count">Всего ходов: {{ moveCount }}</div>
      </div>
      
      <div class="button-group">
        <button class="menu-button" @click="$emit('replay')">
          🔄 Играть снова
        </button>
        <button class="menu-button" @click="$emit('menu')">
          🏠 Главное меню
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'

export default {
  name: 'GameOver',
  props: {
    theme: String,
    winner: String,
    winnerName: String,
    piles: Array,
    moveCount: Number
  },
  emits: ['replay', 'menu'],
  setup(props) {
    const confettiCanvas = ref(null)
    
    const pilesText = computed(() => {
      return [...props.piles].sort((a, b) => b - a).join(' | ')
    })
    
    onMounted(() => {
      if (confettiCanvas.value) {
        const canvas = confettiCanvas.value
        canvas.width = window.innerWidth
        canvas.height = window.innerHeight
        const ctx = canvas.getContext('2d')
        
        const colors = ['#e94560', '#4ecca3', '#5e72e4', '#ffd700']
        const confetti = []
        
        for (let i = 0; i < 100; i++) {
          confetti.push({
            x: Math.random() * canvas.width,
            y: Math.random() * canvas.height - canvas.height,
            size: Math.random() * 10 + 5,
            color: colors[Math.floor(Math.random() * colors.length)],
            speedY: Math.random() * 3 + 2,
            speedX: Math.random() * 2 - 1,
            rotation: Math.random() * 360
          })
        }
        
        function animate() {
          ctx.clearRect(0, 0, canvas.width, canvas.height)
          
          confetti.forEach(c => {
            ctx.save()
            ctx.translate(c.x, c.y)
            ctx.rotate(c.rotation * Math.PI / 180)
            ctx.fillStyle = c.color
            ctx.fillRect(-c.size / 2, -c.size / 2, c.size, c.size)
            ctx.restore()
            
            c.y += c.speedY
            c.x += c.speedX
            c.rotation += 5
            
            if (c.y > canvas.height) {
              c.y = -c.size
              c.x = Math.random() * canvas.width
            }
          })
          
          requestAnimationFrame(animate)
        }
        
        animate()
      }
    })
    
    return {
      confettiCanvas,
      pilesText
    }
  }
}
</script>

<style scoped>
.gameover-container {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

.gameover-container.dark {
  background-color: #1a1a2e;
}

.gameover-container.light {
  background-color: #f0f4f8;
}

.confetti-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.gameover-content {
  background: rgba(15, 52, 96, 0.95);
  padding: 50px 80px;
  border-radius: 20px;
  text-align: center;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
  z-index: 1;
}

.light .gameover-content {
  background: rgba(255, 255, 255, 0.95);
}

.trophy {
  font-size: 2.33rem;
  font-weight: bold;
  color: #ffd700;
  margin-bottom: 20px;
}

.winner-name {
  font-size: 2rem;
  font-weight: bold;
  color: #4ecca3;
  margin-bottom: 10px;
}

.light .winner-name {
  color: #10b981;
}

.winner-text {
  font-size: 1rem;
  color: #8b95a8;
  margin-bottom: 30px;
}

.light .winner-text {
  color: #5a6c7d;
}

.stats-box {
  background: #16213e;
  padding: 20px;
  border-radius: 10px;
  margin-bottom: 30px;
}

.light .stats-box {
  background: #e1e8ed;
}

.stats-title {
  font-size: 0.78rem;
  font-weight: bold;
  color: white;
  margin-bottom: 10px;
}

.light .stats-title {
  color: #1a1a2e;
}

.piles-text {
  font-size: 0.89rem;
  color: #4ecca3;
  margin-bottom: 10px;
}

.light .piles-text {
  color: #10b981;
}

.move-count {
  font-size: 0.67rem;
  color: #8b95a8;
}

.light .move-count {
  color: #5a6c7d;
}

.button-group {
  display: flex;
  gap: 10px;
}

.menu-button {
  flex: 1;
  font-size: 0.89rem;
  font-weight: bold;
  background: #e94560;
  color: white;
  border: none;
  padding: 15px 30px;
  border-radius: 10px;
  cursor: pointer;
  transition: all 0.3s;
}

.menu-button:hover {
  background: #ff5370;
  transform: translateY(-2px);
  box-shadow: 0 5px 20px rgba(233, 69, 96, 0.4);
}

@media (max-width: 768px) {
  .gameover-content {
    width: 90%;
    padding: 30px 25px;
  }

  .trophy {
    font-size: 1.8rem;
  }

  .winner-name {
    font-size: 1.5rem;
  }

  .button-group {
    flex-direction: column;
    gap: 15px;
  }
}
</style>