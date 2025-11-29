<template>
  <div :class="['chips-container', theme]">
    <canvas ref="bgCanvas" class="bg-canvas"></canvas>
    <div class="chips-content fade-in">
      <h1 class="title">Начальное количество</h1>
      
      <p class="description">Введите количество фишек (5-100):</p>
      
      <div class="value-display">{{ chips }}</div>
      
      <input 
        v-model.number="chips" 
        type="range" 
        min="5" 
        max="100" 
        class="slider"
      />
      
      <button class="menu-button" @click="startGame">
        Начать игру! 🎮
      </button>
      
      <button class="back-button" @click="$emit('back')">
        ← Назад
      </button>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'

export default {
  name: 'ChipsInput',
  props: {
    theme: String
  },
  emits: ['start', 'back'],
  setup(props, { emit }) {
    const chips = ref(20)
    const bgCanvas = ref(null)
    
    const startGame = () => {
      document.activeElement.blur()
      emit('start', chips.value)
    }

    onMounted(() => {
      if (bgCanvas.value) {
        const canvas = bgCanvas.value
        canvas.width = window.innerWidth
        canvas.height = window.innerHeight
        const ctx = canvas.getContext('2d')
        
        const stars = [];
        for (let i = 0; i < 100; i++) {
          stars.push({
            x: Math.random() * canvas.width,
            y: Math.random() * canvas.height,
            size: Math.random() * 2 + 1,
            speed: Math.random() * 0.5 + 0.1
          });
        }

        function animate() {
          ctx.clearRect(0, 0, canvas.width, canvas.height);
          ctx.fillStyle = 'rgba(255, 255, 255, 0.1)';
          
          stars.forEach(star => {
            star.y -= star.speed;
            if (star.y < 0) {
              star.y = canvas.height;
              star.x = Math.random() * canvas.width;
            }
            ctx.beginPath();
            ctx.arc(star.x, star.y, star.size, 0, Math.PI * 2);
            ctx.fill();
          });

          requestAnimationFrame(animate);
        }

        animate();
      }
    })
    
    return {
      chips,
      startGame,
      bgCanvas
    }
  }
}
</script>

<style scoped>
.chips-container {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

.bg-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 0;
}

.chips-content {
  position: relative;
  z-index: 1;
}

.chips-container.dark {
  background-color: #1a1a2e;
}

.chips-container.light {
  background-color: #f0f4f8;
}

.chips-content {
  background: rgba(15, 52, 96, 0.9);
  padding: 50px 60px;
  border-radius: 20px;
  text-align: center;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  min-width: 400px;
}

.light .chips-content {
  background: rgba(255, 255, 255, 0.95);
}

.title {
  font-size: 28px;
  font-weight: bold;
  color: #e94560;
  margin-bottom: 20px;
}

.description {
  font-size: 14px;
  color: #8b95a8;
  margin-bottom: 30px;
}

.light .description {
  color: #5a6c7d;
}

.value-display {
  font-size: 36px;
  font-weight: bold;
  color: #e94560;
  margin-bottom: 20px;
}

.slider {
  width: 100%;
  height: 20px;
  -webkit-appearance: none;
  appearance: none;
  background: #1a1a2e;
  border-radius: 10px;
  outline: none;
  margin-bottom: 30px;
}

.light .slider {
  background: #e1e8ed;
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 30px;
  height: 30px;
  background: #e94560;
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.3s;
}

.slider::-webkit-slider-thumb:hover {
  transform: scale(1.2);
  box-shadow: 0 0 20px rgba(233, 69, 96, 0.5);
}

.slider::-moz-range-thumb {
  width: 30px;
  height: 30px;
  background: #e94560;
  border-radius: 50%;
  cursor: pointer;
  border: none;
  transition: all 0.3s;
}

.slider::-moz-range-thumb:hover {
  transform: scale(1.2);
  box-shadow: 0 0 20px rgba(233, 69, 96, 0.5);
}

.menu-button {
  font-size: 16px;
  font-weight: bold;
  background: #e94560;
  color: white;
  border: none;
  padding: 15px 40px;
  border-radius: 10px;
  cursor: pointer;
  transition: all 0.3s;
  width: 100%;
  margin-bottom: 10px;
}

.menu-button:hover {
  background: #ff5370;
  transform: translateY(-2px);
}

.back-button {
  font-size: 12px;
  background: #16213e;
  color: white;
  border: none;
  padding: 8px 20px;
  border-radius: 5px;
  cursor: pointer;
  transition: all 0.3s;
}

.light .back-button {
  background: #e1e8ed;
  color: #1a1a2e;
}

.back-button:hover {
  transform: scale(1.05);
}
</style>