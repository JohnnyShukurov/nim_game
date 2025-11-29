<template>
  <div :class="['chips-container', theme]">
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
import { ref } from 'vue'

export default {
  name: 'ChipsInput',
  props: {
    theme: String
  },
  emits: ['start', 'back'],
  setup(props, { emit }) {
    const chips = ref(20)
    
    const startGame = () => {
      document.activeElement.blur()
      emit('start', chips.value)
    }
    
    return {
      chips,
      startGame
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