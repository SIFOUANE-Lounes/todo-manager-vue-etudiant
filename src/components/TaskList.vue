<template>
  <section>
    <header class="list__header">
      <h2>📋 Mes Tâches <span class="muted">— {{ tasks.length }}</span></h2>
    </header>

    <p v-if="!tasks.length" class="empty">Aucune tâche pour l’instant. Ajoute-en une ci-dessus !</p>

    <ul v-else class="list">
      <li v-for="t in tasks" :key="t.id" class="item">
        <div class="item__left">
          <span class="badge" :data-status="t.status">
            {{ label(t.status) }}
          </span>
          <span class="item__title">{{ t.title }}</span>
        </div>

        <div class="item__right">
          <label class="sr-only" :for="`status-${t.id}`">Statut</label>
          <select
            :id="`status-${t.id}`"
            class="select"
            :value="t.status"
            @change="onStatus(t.id, $event.target.value)"
          >
            <option value="to do">À faire</option>
            <option value="in progress">En cours</option>
            <option value="done">Terminée</option>
          </select>

          <button class="button danger" @click="onRemove(t.id)" :aria-label="`Supprimer ${t.title}`">
            Supprimer
          </button>
        </div>
      </li>
    </ul>
  </section>
</template>

<script setup>
const props = defineProps({
  tasks: { type: Array, required: true }
})
const emit = defineEmits(['remove', 'status'])

function onRemove(id) { emit('remove', id) }
function onStatus(id, status) { emit('status', id, status) }

function label(status) {
  if (status === 'to do') return 'À faire'
  if (status === 'in progress') return 'En cours'
  if (status === 'done') return 'Terminée'
  return status
}
</script>

<style scoped>
.list__header { display: flex; align-items: baseline; justify-content: space-between; margin: .25rem 0 .5rem; }
.muted { color: #607d8b; font-weight: normal; font-size: .95em; }
.empty { color: #607d8b; font-style: italic; margin: .5rem 0; }

.list { display: grid; gap: .6rem; list-style: none; padding: 0; margin: 0; }
.item {
  display: flex; gap: .75rem; align-items: center; justify-content: space-between;
  padding: .7rem .8rem; border: 1px solid #e0e0e0; border-radius: .7rem; background: #fff;
  box-shadow: 0 2px 6px rgba(0,0,0,.04);
}
.item__left { display: flex; gap: .6rem; align-items: center; min-width: 0; }
.item__title { font-weight: 600; color: #263238; word-break: break-word; }

.badge {
  font-size: .75rem; padding: .2rem .5rem; border-radius: .5rem; border: 1px solid #cfd8dc; color: #455a64;
  background: #eceff1;
}
.badge[data-status="in progress"] { background: #fff3cd; border-color: #ffe08a; color: #8a6d3b; }
.badge[data-status="done"]       { background: #e6ffed; border-color: #a6f4c5; color: #046c4e; }

.item__right { display: flex; gap: .5rem; align-items: center; }
.select, .button { padding: .45rem .6rem; border: 1px solid #cfd8dc; border-radius: .5rem; }
.button.danger {
  background: #ef5350; color: white; border-color: #ef5350; cursor: pointer;
}
.button.danger:hover { filter: brightness(1.05); }

.sr-only {
  position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px;
  overflow: hidden; clip: rect(0,0,0,0); white-space: nowrap; border: 0;
}
</style>
