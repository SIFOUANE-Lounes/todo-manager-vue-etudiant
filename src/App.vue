<template>
  <div class="app">
    <header class="app__header">
      <h1>🎓 To-Do Manager Étudiant</h1>
      <p class="subtitle">Projet Vue.js – Lounes Sifouane</p>
    </header>

    <main class="app__main">
      <TaskForm @add="addTask" />

      <section class="toolbar">
        <input
          id="search"
          v-model.trim="query"
          type="search"
          placeholder="🔍 Rechercher une tâche..."
          class="input"
        />

        <select v-model="filter" class="select">
          <option value="all">Toutes</option>
          <option value="to do">À faire</option>
          <option value="in progress">En cours</option>
          <option value="done">Terminée</option>
        </select>
      </section>

      <TaskList
        :tasks="filteredTasks"
        @remove="removeTask"
        @status="setStatus"
      />

      <p v-if="stats.total" class="progress">Progression : {{ stats.progress }}%</p>
      <div v-if="stats.total" class="bar">
        <div class="bar__fill" :style="{ width: stats.progress + '%' }"></div>
      </div>
    </main>

    <footer class="app__footer">
      <p>© 2025 – Projet étudiant réalisé avec ❤️ par Lounes Sifouane</p>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import TaskForm from './components/TaskForm.vue'
import TaskList from './components/TaskList.vue'

const tasks = ref(loadTasks())
const query = ref('')
const filter = ref('all')

watch(tasks, (val) => localStorage.setItem('tasks', JSON.stringify(val)), { deep: true })

function loadTasks() {
  try {
    return JSON.parse(localStorage.getItem('tasks')) || []
  } catch {
    return []
  }
}

function addTask(title) {
  tasks.value.push({
    id: Date.now(),
    title,
    status: 'to do',
    createdAt: new Date().toLocaleString()
  })
}

function removeTask(id) {
  tasks.value = tasks.value.filter(t => t.id !== id)
}

function setStatus(id, status) {
  const t = tasks.value.find(t => t.id === id)
  if (t) t.status = status
}

const filteredTasks = computed(() => {
  const q = query.value.toLowerCase()
  return tasks.value.filter(t => {
    const okStatus = filter.value === 'all' || t.status === filter.value
    const okQuery = t.title.toLowerCase().includes(q)
    return okStatus && okQuery
  })
})

const stats = computed(() => {
  const total = tasks.value.length
  const done = tasks.value.filter(t => t.status === 'done').length
  const progress = total ? Math.round((done / total) * 100) : 0
  return { total, done, progress }
})
</script>

<style scoped>
.app {
  max-width: 700px;
  margin: 0 auto;
  padding: 1rem;
  font-family: "Poppins", sans-serif;
  background: #fefefe;
  border-radius: 10px;
  box-shadow: 0 0 12px rgba(0,0,0,0.1);
}
.app__header {
  text-align: center;
  color: #222;
}
.subtitle {
  color: #777;
  font-size: 0.9rem;
  margin-top: -5px;
}
.app__main { display: grid; gap: 1rem; margin-top: 1rem; }
.toolbar { display: flex; gap: .5rem; }
.input, .select {
  flex: 1;
  padding: .6rem;
  border: 1px solid #ccc;
  border-radius: .5rem;
}
.progress { text-align: center; font-weight: 500; color: #333; }
.bar { height: 8px; background: #e0e0e0; border-radius: 5px; overflow: hidden; }
.bar__fill { height: 100%; background: linear-gradient(90deg,#00b09b,#96c93d); transition: width 0.3s; }
.app__footer {
  text-align: center;
  margin-top: 1.5rem;
  font-size: 0.85rem;
  color: #555;
}
</style>
