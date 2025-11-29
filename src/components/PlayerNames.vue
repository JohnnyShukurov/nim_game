<template>
  <div :class="['names-container', theme]">
    <div class="names-content fade-in">
      <h1 class="title">Имена игроков</h1>
      
      <div class="input-group">
        <label class="input-label">Игрок 1:</label>
        <input 
          v-model="player1" 
          type="text" 
          class="name-input"
          placeholder="Игрок 1"
          @keyup.enter="continueGame"
        />
      </div>
      
      <div class="input-group">
        <label class="input-label">Игрок 2:</label>
        <input 
          v-model="player2" 
          type="text" 
          class="name-input"
          placeholder="Игрок 2"
          @keyup.enter="continueGame"
        />
      </div>
      
      <button class="menu-button" @click="continueGame">
        Продолжить →
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
  name: 'PlayerNames',
  props: {
    theme: String
  },
  emits: ['continue', 'back'],
  setup(props, { emit }) {
    const player1 = ref('Игрок 1')
    const player2 = ref('Игрок 2')
    
    const continueGame = () => {
      document.activeElement.blur()
      emit('continue', {
        player1: player1.value.trim() || 'Игрок 1',
        player2: player2.value.trim() || 'Игрок 2'
      })
    }
    
    return {
      player1,
      player2,
      continueGame
    }
  }
}
</script>

<style scoped>
.names-container {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.names-container.dark {
  background-color: #1a1a2e;
}

.names-container.light {
  background-color: #f0f4f8;
}

.names-content {
  background: rgba(15, 52, 96, 0.9);
  padding: 50px 60px;
  border-radius: 20px;
  text-align: center;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

.light .names-content {
  background: rgba(255, 255, 255, 0.95);
}

.title {
  font-size: 28px;
  font-weight: bold;
  color: #e94560;
  margin-bottom: 30px;
}

.input-group {
  margin-bottom: 20px;
}

.input-label {
  display: block;
  font-size: 14px;
  color: #8b95a8;
  margin-bottom: 5px;
}

.light .input-label {
  color: #5a6c7d;
}

.name-input {
  width: 100%;
  font-size: 16px;
  padding: 12px;
  border: 2px solid transparent;
  border-radius: 8px;
  background: #16213e;
  color: white;
  text-align: center;
  transition: all 0.3s;
}

.light .name-input {
  background: #e1e8ed;
  color: #1a1a2e;
}

.name-input:focus {
  outline: none;
  border-color: #e94560;
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
  margin: 20px 0 10px;
  width: 100%;
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