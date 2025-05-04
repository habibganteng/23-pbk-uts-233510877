<script setup>
import { ref, computed, onMounted, watch } from 'vue'

const todos = ref([])
const newTask = ref('')
const selectedCategory = ref(null)
const filterGender = ref('all')

// Ambil dari localStorage saat mount
onMounted(() => {
  const saved = localStorage.getItem('todos')
  if (saved) {
    todos.value = JSON.parse(saved)
  }
})

// Simpan otomatis jika todos berubah
watch(todos, (newVal) => {
  localStorage.setItem('todos', JSON.stringify(newVal))
}, { deep: true })

// Filter berdasarkan kategori
const filteredTodos = computed(() => {
  if (filterGender.value === 'all') return todos.value
  return todos.value.filter(todo => todo.category === filterGender.value)
})

// Tambah peserta
const addTodo = () => {
  if (newTask.value.trim() === '' || !selectedCategory.value) return
  todos.value.unshift({
    id: Date.now(),
    task: newTask.value,
    category: selectedCategory.value,
    done: false
  })
  newTask.value = ''
  selectedCategory.value = null
}

// Tandai selesai
const toggleDone = (id) => {
  const todo = todos.value.find(t => t.id === id)
  if (todo) todo.done = !todo.done
}

// Hapus peserta
const removeTodo = (id) => {
  todos.value = todos.value.filter(todo => todo.id !== id)
}
</script>

<template>
  <div class="background">
    <div class="todo-app">
      <header class="app-header">
        <h1>Daftar Pinjol</h1>
        <div class="stats">
          <div class="stat">
            <span class="count">{{ todos.length }}</span>
            <span class="label"> Total Peserta</span>
          </div>
          <div class="stat">
            <span class="count">{{ todos.filter(t => t.done).length }}</span>
            <span class="label"> Sudah Lunas</span>
          </div>
        </div>
      </header>

      <div class="input-section">
        <input 
          type="text" 
          v-model="newTask"
          placeholder="Masukkan Nama Peserta"
          @keyup.enter="addTodo"
          class="task-input"
        />

        <div class="categories">
          <button 
            @click="selectedCategory = 'male'"
            :class="{ active: selectedCategory === 'male' }"
            class="male-btn"
          >
            ♂ Laki-laki
          </button>
          <button 
            @click="selectedCategory = 'female'"
            :class="{ active: selectedCategory === 'female' }"
            class="female-btn"
          >
            ♀ Perempuan
          </button>
        </div>

        <button @click="addTodo" class="add-btn">
          ➕ Tambah Peserta
        </button>
      </div>

      <div class="filter-section">
        <button 
          @click="filterGender = 'all'"
          :class="{ active: filterGender === 'all' }"
          class="filter-btn"
        >
          Semua
        </button>
        <button 
          @click="filterGender = 'male'"
          :class="{ active: filterGender === 'male' }"
          class="filter-btn"
        >
          Laki-laki
        </button>
        <button 
          @click="filterGender = 'female'"
          :class="{ active: filterGender === 'female' }"
          class="filter-btn"
        >
          Perempuan
        </button>
      </div>

      <div class="todo-list">
        <div 
          v-for="todo in filteredTodos" 
          :key="todo.id"
          class="todo-item"
          :class="{ done: todo.done, [todo.category]: true }"
        >
          <div class="checkbox" @click="toggleDone(todo.id)">
            <span v-if="todo.done">✅</span>
            <span v-else>⬜</span>
          </div>

          <div class="task-content">
            <span class="task">{{ todo.task }}</span>
            <span class="badge" :class="todo.category">
              {{ todo.category === 'male' ? '♂' : '♀' }}
            </span>
          </div>

          <button @click="removeTodo(todo.id)" class="delete-btn">
            ❌
          </button>
        </div>

        <div v-if="filteredTodos.length === 0" class="empty-state">
          <p>Tidak ada peserta</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
html, body {
  margin: 0;
  padding: 0;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  height: 100%;
  width: 100%;
  background: linear-gradient(135deg, 
    #ff9a9e 0%, 
    #fad0c4 50%, 
    #fbc2eb 100%);
  background-size: 400% 400%;
  animation: gradientBG 15s ease infinite;
  box-sizing: border-box;
}

@keyframes gradientBG {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}

.background {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: start;
  padding: 40px 20px;
  position: relative;
}

.background::before {
  content: "";
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: url('https://www.transparenttextures.com/patterns/soft-wallpaper.png');
  opacity: 0.3;
  pointer-events: none;
  z-index: -1;
}

.todo-app {
  background-color: rgba(255, 255, 255, 0.95);
  padding: 2rem;
  border-radius: 20px;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.2);
  width: 100%;
  max-width: 400px;
  position: relative;
  overflow: hidden;
  border-bottom: 15px solid #e91e63;
  margin-top: 20px;
}

.todo-app::after {
  content: "";
  position: absolute;
  bottom: -15px;
  left: 0;
  right: 0;
  height: 15px;
  background: linear-gradient(90deg, 
    #e91e63 0%, 
    #9c27b0 20%, 
    #3f51b5 40%, 
    #2196f3 60%, 
    #00bcd4 80%, 
    #4caf50 100%);
  border-radius: 0 0 20px 20px;
}

.app-header {
  text-align: center;
  margin-bottom: 1.5rem;
}

.app-header h1 {
  color: #d61c4e;
  font-size: 1.5rem;
  margin-bottom: 1rem;
  text-shadow: 1px 1px 2px rgba(0,0,0,0.1);
}

.stats {
  display: flex;
  justify-content: space-around;
  background: rgba(233, 30, 99, 0.1);
  padding: 10px;
  border-radius: 10px;
}

.stat {
  text-align: center;
  color: #555;
  font-weight: bold;
}

.count {
  display: block;
  font-size: 1.5rem;
  color: #e91e63;
}

.input-section {
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
  margin-bottom: 1.5rem;
}

.task-input {
  padding: 0.8rem 1rem;
  border: 2px solid #e0e0e0;
  border-radius: 12px;
  font-size: 1rem;
  width: 100%;
  transition: all 0.3s;
  background-color: rgba(255, 255, 255, 0.8);
}

.task-input:focus {
  outline: none;
  border-color: #e91e63;
  box-shadow: 0 0 0 3px rgba(233, 30, 99, 0.2);
}

.categories {
  display: flex;
  gap: 10px;
}

.male-btn,
.female-btn {
  flex: 1;
  padding: 0.8rem;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s;
}

.male-btn {
  background-color: rgba(0, 188, 212, 0.2);
  color: #00796b;
}

.female-btn {
  background-color: rgba(244, 143, 177, 0.2);
  color: #c2185b;
}

.male-btn:hover, .female-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}

.male-btn.active {
  background-color: #00bcd4;
  color: white;
}

.female-btn.active {
  background-color: #f48fb1;
  color: white;
}

.add-btn {
  background: linear-gradient(45deg, #e91e63, #ff5722);
  color: white;
  font-weight: bold;
  border: none;
  border-radius: 12px;
  padding: 0.8rem;
  cursor: pointer;
  transition: all 0.3s;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
}

.add-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0,0,0,0.15);
}

.filter-section {
  display: flex;
  gap: 0.5rem;
  margin: 1.5rem 0;
  flex-wrap: wrap;
  justify-content: center;
}

.filter-btn {
  padding: 0.5rem 1rem;
  border: 2px solid #e0e0e0;
  border-radius: 12px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s;
  background-color: white;
}

.filter-btn:hover {
  transform: translateY(-2px);
}

.filter-btn.active {
  background: linear-gradient(45deg, #e91e63, #9c27b0);
  color: white;
  border-color: transparent;
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}

.todo-list {
  margin-top: 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
}

.todo-item {
  background-color: white;
  padding: 0.8rem 1.2rem;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-left: 6px solid #ccc;
  box-shadow: 0 2px 4px rgba(0,0,0,0.05);
  transition: all 0.3s;
}

.todo-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0,0,0,0.1);
}

.todo-item.male {
  border-left-color: #00bcd4;
}

.todo-item.female {
  border-left-color: #f48fb1;
}

.todo-item.done {
  opacity: 0.6;
  text-decoration: line-through;
  background-color: #f5f5f5;
}

.checkbox {
  cursor: pointer;
  margin-right: 12px;
  font-size: 1.2rem;
}

.task-content {
  flex: 1;
  display: flex;
  align-items: center;
  gap: 0.8rem;
}

.task {
  font-weight: 500;
}

.badge {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  font-weight: bold;
}

.badge.male {
  background-color: #e0f7fa;
  color: #00796b;
}

.badge.female {
  background-color: #fce4ec;
  color: #c2185b;
}

.delete-btn {
  background: transparent;
  border: none;
  color: #c62828;
  font-size: 1.2rem;
  cursor: pointer;
  padding: 0.2rem;
  transition: all 0.2s;
}

.delete-btn:hover {
  transform: scale(1.2);
}

.empty-state {
  text-align: center;
  color: #888;
  font-style: italic;
  margin-top: 1.5rem;
  padding: 1rem;
  background-color: rgba(255,255,255,0.7);
  border-radius: 10px;
}
</style>