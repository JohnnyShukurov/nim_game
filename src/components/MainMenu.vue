<template>
  <div :class="['menu-container', theme]">
    <canvas ref="bgCanvas" class="bg-canvas"></canvas>
    
    <div class="menu-content fade-in">
      <div class="title-container">
        <h1 class="title-shadow">NIM</h1>
        <h1 class="title">NIM</h1>
      </div>
      
      <p class="subtitle">Стратегическая игра на логику</p>
      
      <div class="button-group">
        <button class="menu-button" @click="$emit('select-pvc')">
          🤖 Игра с компьютером
        </button>
        <button class="menu-button" @click="$emit('select-pvp')">
          👥 Два игрока
        </button>
      </div>
      
      <div class="theme-toggle">
        <span class="theme-label">Тема:</span>
        <button class="theme-button" @click="$emit('toggle-theme')">
          {{ theme === 'dark' ? '🌙 Темная' : '☀️ Светлая' }}
        </button>
      </div>
      
      <p class="rules">
        Правила: Делите кучки на две неравные части.<br>
        Проигрывает тот, кто не может сделать ход!
      </p>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'

export default {
  name: 'MainMenu',
  props: {
    theme: {
      type: String,
      required: true
    }
  },
  emits: ['toggle-theme', 'select-pvc', 'select-pvp'],
  setup() {
    const bgCanvas = ref(null)
    
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
      bgCanvas
    }
  }
}
</script>

<style scoped>
.menu-container {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

.menu-container.dark {
  background-color: #1a1a2e;
}

.menu-container.light {
  background-color: #f0f4f8;
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

.menu-content {
  background: rgba(15, 52, 96, 0.9);
  padding: 70px 80px;
  border-radius: 20px;
  text-align: center;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  position: relative;
  z-index: 1;
}

.light .menu-content {
  background: rgba(255, 255, 255, 0.95);
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.1);
}

.title-container {
  position: relative;
  margin-bottom: 10px;
}

.title-shadow {
  position: absolute;
  top: 2px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 56px;
  font-weight: bold;
  color: rgba(0, 0, 0, 0.3);
  margin: 0;
  pointer-events: none;
}

.title {
  font-size: 56px;
  font-weight: bold;
  color: #e94560;
  margin: 0;
  position: relative;
}

.subtitle {
  font-size: 14px;
  font-style: italic;
  color: #8b95a8;
  margin-bottom: 30px;
}

.light .subtitle {
  color: #5a6c7d;
}

.button-group {
  display: flex;
  flex-direction: column;
  gap: 15px;
  margin: 20px 0;
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
}

.menu-button:hover {
  background: #ff5370;
  transform: translateY(-2px);
  box-shadow: 0 5px 20px rgba(233, 69, 96, 0.4);
}

.theme-toggle {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  margin: 20px 0;
}

.theme-label {
  font-size: 12px;
  color: #8b95a8;
}

.light .theme-label {
  color: #5a6c7d;
}

.theme-button {
  font-size: 12px;
  background: #16213e;
  color: white;
  border: none;
  padding: 8px 15px;
  border-radius: 5px;
  cursor: pointer;
  transition: all 0.3s;
}

.light .theme-button {
  background: #e1e8ed;
  color: #1a1a2e;
}

.theme-button:hover {
  transform: scale(1.05);
}

.rules {
  font-size: 11px;
  color: #8b95a8;
  margin-top: 30px;
  line-height: 1.6;
}

.light .rules {
  color: #5a6c7d;
}
</style>