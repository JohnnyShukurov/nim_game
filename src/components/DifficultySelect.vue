<template>
  <div :class="['difficulty-container', theme]">
    <div class="difficulty-content fade-in">
      <h1 class="title">Выберите сложность</h1>
      
      <div class="difficulty-options">
        <div v-for="diff in difficulties" :key="diff.value" class="diff-option">
          <button class="menu-button" @click="$emit('select', diff.value)">
            {{ diff.emoji }} {{ diff.name }}
          </button>
          <p class="diff-desc">{{ diff.description }}</p>
        </div>
      </div>
      
      <button class="back-button" @click="$emit('back')">
        ← Назад
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'DifficultySelect',
  props: {
    theme: String
  },
  emits: ['select', 'back'],
  setup() {
    const difficulties = [
      {
        value: 'easy',
        emoji: '🌱',
        name: 'Легко',
        description: 'Компьютер играет случайно'
      },
      {
        value: 'medium',
        emoji: '⚡',
        name: 'Средне',
        description: 'Компьютер играет умно'
      },
      {
        value: 'hard',
        emoji: '🔥',
        name: 'Сложно',
        description: 'Компьютер использует оптимальную стратегию'
      }
    ]
    
    return {
      difficulties
    }
  }
}
</script>

<style scoped>
.difficulty-container {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.difficulty-container.dark {
  background-color: #1a1a2e;
}

.difficulty-container.light {
  background-color: #f0f4f8;
}

.difficulty-content {
  background: rgba(15, 52, 96, 0.9);
  padding: 50px 60px;
  border-radius: 20px;
  text-align: center;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

.light .difficulty-content {
  background: rgba(255, 255, 255, 0.95);
}

.title {
  font-size: 32px;
  font-weight: bold;
  color: #e94560;
  margin-bottom: 30px;
}

.difficulty-options {
  display: flex;
  flex-direction: column;
  gap: 20px;
  margin-bottom: 30px;
}

.diff-option {
  display: flex;
  flex-direction: column;
  gap: 5px;
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
}

.diff-desc {
  font-size: 10px;
  color: #8b95a8;
  margin: 0;
}

.light .diff-desc {
  color: #5a6c7d;
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