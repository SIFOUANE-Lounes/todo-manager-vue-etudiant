<template>
  <form @submit.prevent="submit" class="form">
    <label class="sr-only" for="task-title">Titre de la tâche</label>
    <input
      id="task-title"
      v-model.trim="title"
      type="text"
      placeholder="✍️ Ajouter une nouvelle tâche..."
      required
      class="input"
    />
    <button type="submit" class="button">Ajouter</button>
  </form>
</template>

<script setup>
import { ref } from 'vue'
const title = ref('')
const emit = defineEmits(['add'])

function submit() {
  const value = title.value.trim()
  if (!value) return
  emit('add', value)  // => App.vue écoute @add="addTask"
  title.value = ''
}
</script>

<style scoped>
.form { display: flex; gap: .5rem; }
.input {
  flex: 1;
  padding: .6rem .75rem;
  border: 1px solid #cfd8dc;
  border-radius: .6rem;
  background: #fff;
}
.button {
  padding: .6rem .9rem;
  border: none;
  border-radius: .6rem;
  background: linear-gradient(90deg,#00b09b,#96c93d);
  color: #fff;
  font-weight: 600;
  cursor: pointer;
}
.button:hover { filter: brightness(1.05); }
.sr-only {
  position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px;
  overflow: hidden; clip: rect(0,0,0,0); white-space: nowrap; border: 0;
}
</style>
