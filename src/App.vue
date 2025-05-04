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