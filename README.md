# 🎓 To‑Do Manager Étudiant (Vue 3) 
---

## 0) Prérequis

* Node.js 18+ (ou 20+)
* Git installé

Vérifie :

```bash
node -v
npm -v
git --version
```

---

## 1) Créer le projet avec Vite

**Pourquoi ?** Vite démarre vite et configure Vue 3 automatiquement.

```bash
npm create vite@latest todo-manager -- --template vue
cd todo-manager
npm i
```

**Test** :

```bash
npm run dev
```

Ouvre `http://localhost:5173` → la page Vite/Vue apparaît.

**Commit** :

```bash
git init
git add .
git commit -m "chore: init vite + vue"
```

---

## 2) Point d’entrée de l’app

**Pourquoi ?** On monte l’app Vue dans `#app`.

**Fichier :** `src/main.js`

```js
import { createApp } from 'vue'
import App from './App.vue'

createApp(App).mount('#app')
```

**Fichier :** `index.html`

```html
<!doctype html>
<html lang="fr">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>To‑Do Manager Étudiant</title>
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/main.js"></script>
  </body>
</html>
```

**Commit** :

```bash
git add .
git commit -m "feat: point d'entrée de l'app"
```

---

## 3) Créer le squelette `App.vue`

**Pourquoi ?** C’est le conteneur : header, formulaire, liste, footer.

**Fichier :** `src/App.vue`

```vue
<template>
  <div class="app">
    <header class="app__header">
      <h1>🎓 To‑Do Manager Étudiant</h1>
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

      <TaskList :tasks="filteredTasks" @remove="removeTask" @status="setStatus" />

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

// État
const tasks = ref(loadTasks())
const query = ref('')
const filter = ref('all')

// Persistance locale
watch(tasks, (val) => localStorage.setItem('tasks', JSON.stringify(val)), { deep: true })

function loadTasks() {
  try { return JSON.parse(localStorage.getItem('tasks')) || [] } catch { return [] }
}

function addTask(title) {
  tasks.value.push({ id: Date.now(), title, status: 'to do', createdAt: new Date().toLocaleString() })
}
function removeTask(id) { tasks.value = tasks.value.filter(t => t.id !== id) }
function setStatus(id, status) { const t = tasks.value.find(t => t.id === id); if (t) t.status = status }

// Dérivés
const filteredTasks = computed(() => {
  const q = query.value.toLowerCase()
  return tasks.value.filter(t => (filter.value === 'all' || t.status === filter.value) && t.title.toLowerCase().includes(q))
})

const stats = computed(() => {
  const total = tasks.value.length
  const done = tasks.value.filter(t => t.status === 'done').length
  const progress = total ? Math.round((done / total) * 100) : 0
  return { total, done, progress }
})
</script>

<style scoped>
.app { max-width: 700px; margin: 0 auto; padding: 1rem; font-family: "Poppins", sans-serif; background: #fefefe; border-radius: 10px; box-shadow: 0 0 12px rgba(0,0,0,0.1); }
.app__header { text-align: center; color: #222; }
.subtitle { color: #777; font-size: 0.9rem; margin-top: -5px; }
.app__main { display: grid; gap: 1rem; margin-top: 1rem; }
.toolbar { display: flex; gap: .5rem; }
.input, .select { flex: 1; padding: .6rem; border: 1px solid #ccc; border-radius: .5rem; }
.progress { text-align: center; font-weight: 500; color: #333; }
.bar { height: 8px; background: #e0e0e0; border-radius: 5px; overflow: hidden; }
.bar__fill { height: 100%; background: linear-gradient(90deg,#00b09b,#96c93d); transition: width 0.3s; }
.app__footer { text-align: center; margin-top: 1.5rem; font-size: 0.85rem; color: #555; }
</style>
```

**Test** : l’app se lance toujours (`npm run dev`).

**Commit** :

```bash
git add src/App.vue
git commit -m "feat: squelette App.vue + état de base"
```

---

## 4) Créer le formulaire d’ajout

**Pourquoi ?** Saisir une nouvelle tâche et l’envoyer au parent.

**Fichier :** `src/components/TaskForm.vue`

```vue
<template>
  <form @submit.prevent="submit" class="form">
    <label class="sr-only" for="task-title">Titre de la tâche</label>
    <input id="task-title" v-model.trim="title" type="text" placeholder="✍️ Ajouter une nouvelle tâche..." required class="input" />
    <button type="submit" class="button">Ajouter</button>
  </form>
</template>

<script setup>
import { ref } from 'vue'
const title = ref('')
const emit = defineEmits(['add'])
function submit() { const v = title.value.trim(); if (!v) return; emit('add', v); title.value = '' }
</script>

<style scoped>
.form { display: flex; gap: .5rem; }
.input { flex: 1; padding: .6rem .75rem; border: 1px solid #cfd8dc; border-radius: .6rem; background: #fff; }
.button { padding: .6rem .9rem; border: none; border-radius: .6rem; background: linear-gradient(90deg,#00b09b,#96c93d); color: #fff; font-weight: 600; cursor: pointer; }
.button:hover { filter: brightness(1.05); }
.sr-only { position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px; overflow: hidden; clip: rect(0,0,0,0); white-space: nowrap; border: 0; }
</style>
```

**Test** : tape un titre → Entrée → la tâche apparaît.

**Commit** :

```bash
git add src/components/TaskForm.vue
git commit -m "feat: formulaire d'ajout + émission d'événement"
```

---

## 5) Créer la liste des tâches

**Pourquoi ?** Affiche chaque tâche + changer statut + supprimer.

**Fichier :** `src/components/TaskList.vue`

```vue
<template>
  <section>
    <header class="list__header">
      <h2>📋 Mes Tâches <span class="muted">— {{ tasks.length }}</span></h2>
    </header>

    <p v-if="!tasks.length" class="empty">Aucune tâche pour l’instant. Ajoute-en une ci-dessus !</p>

    <ul v-else class="list">
      <li v-for="t in tasks" :key="t.id" class="item">
        <div class="item__left">
          <span class="badge" :data-status="t.status">{{ label(t.status) }}</span>
          <span class="item__title">{{ t.title }}</span>
        </div>
        <div class="item__right">
          <label class="sr-only" :for="`status-${t.id}`">Statut</label>
          <select :id="`status-${t.id}`" class="select" :value="t.status" @change="onStatus(t.id, $event.target.value)">
            <option value="to do">À faire</option>
            <option value="in progress">En cours</option>
            <option value="done">Terminée</option>
          </select>
          <button class="button danger" @click="onRemove(t.id)">Supprimer</button>
        </div>
      </li>
    </ul>
  </section>
</template>

<script setup>
const props = defineProps({ tasks: { type: Array, required: true } })
const emit = defineEmits(['remove', 'status'])
function onRemove(id) { emit('remove', id) }
function onStatus(id, status) { emit('status', id, status) }
function label(s) { if (s==='to do') return 'À faire'; if (s==='in progress') return 'En cours'; if (s==='done') return 'Terminée'; return s }
</script>

<style scoped>
.list__header { display: flex; align-items: baseline; justify-content: space-between; margin: .25rem 0 .5rem; }
.muted { color: #607d8b; font-weight: normal; font-size: .95em; }
.empty { color: #607d8b; font-style: italic; margin: .5rem 0; }
.list { display: grid; gap: .6rem; list-style: none; padding: 0; margin: 0; }
.item { display: flex; gap: .75rem; align-items: center; justify-content: space-between; padding: .7rem .8rem; border: 1px solid #e0e0e0; border-radius: .7rem; background: #fff; box-shadow: 0 2px 6px rgba(0,0,0,.04); }
.item__left { display: flex; gap: .6rem; align-items: center; min-width: 0; }
.item__title { font-weight: 600; color: #263238; word-break: break-word; }
.badge { font-size: .75rem; padding: .2rem .5rem; border-radius: .5rem; border: 1px solid #cfd8dc; color: #455a64; background: #eceff1; }
.badge[data-status="in progress"] { background: #fff3cd; border-color: #ffe08a; color: #8a6d3b; }
.badge[data-status="done"] { background: #e6ffed; border-color: #a6f4c5; color: #046c4e; }
.item__right { display: flex; gap: .5rem; align-items: center; }
.select, .button { padding: .45rem .6rem; border: 1px solid #cfd8dc; border-radius: .5rem; }
.button.danger { background: #ef5350; color: white; border-color: #ef5350; cursor: pointer; }
.button.danger:hover { filter: brightness(1.05); }
.sr-only { position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px; overflow: hidden; clip: rect(0,0,0,0); white-space: nowrap; border: 0; }
</style>
```

**Test** :

* Ajouter une tâche → elle apparaît
* Changer le statut → le badge change
* Supprimer → la ligne disparaît

**Commit** :

```bash
git add src/components/TaskList.vue
git commit -m "feat: liste des tâches (afficher/changer statut/supprimer)"
```

---

## 6) Persistance locale (localStorage)

**Pourquoi ?** Conserver les tâches quand on rafraîchit la page.

➡️ C’est déjà fait via :

```js
watch(tasks, (val) => localStorage.setItem('tasks', JSON.stringify(val)), { deep: true })
```

Et via :

```js
function loadTasks() { return JSON.parse(localStorage.getItem('tasks')) || [] }
```

**Test** : ajoute 2 tâches → recharge la page → elles sont toujours là.

**Commit** :

```bash
git add src/App.vue
git commit -m "feat: persistance locale via localStorage"
```

---

## 7) Recherche + Filtre

**Pourquoi ?** Améliorer l’UX pour retrouver rapidement.

➡️ Déjà inclus : `query` + `filter` + `filteredTasks`.

**Test** :

* Tape une recherche → la liste se filtre
* Choisis un statut dans le `<select>` → la liste se filtre

**Commit** :

```bash
git add src/App.vue
git commit -m "feat: recherche et filtre par statut"
```

---

## 8) Barre de progression

**Pourquoi ?** Montrer l’avancement (nb de tâches terminées).

➡️ Déjà inclus : `stats.progress` + `<div class="bar">`.

**Test** :

* Marque des tâches en "Terminée" → la barre progresse

**Commit** :

```bash
git add src/App.vue
git commit -m "feat: barre de progression (statistiques)"
```

---

## 9) Préparer le dépôt GitHub & push

**Pourquoi ?** Publier ton travail pour le montrer.

```bash
git remote add origin https://github.com/SIFOUANE-Lounes/todo-manager-vue-etudiant.git
git branch -M main
git push -u origin main
```

**Tip** : ajoute un **README.md** (ce fichier !) à la racine, puis :

```bash
git add README.md
git commit -m "docs: tutoriel pas à pas"
git push
```

---

## 10) Démo en ligne (facultatif mais recommandé)

* **Vercel** ou **Netlify** : connecte ton repo, clique "Deploy"
* **GitHub Pages (vite)** :

  * `npm i -D gh-pages`
  * Ajoute dans `package.json` : `"homepage": "."` et scripts `"deploy": "vite build && gh-pages -d dist"`
  * `npm run deploy`

---

## 🤝 Remerciements & Licence

* Auteur : **Lounes Sifouane** — projet étudiant
* Licence : **MIT**

---

## Annexe A — Checklist de debug rapide

* Formulaire : `@submit.prevent="submit"` et bouton `type="submit"`
* Pas de balises HTML oubliées (chaque `<div>` a son `</div>`)
* Imports de composants corrects : `./components/TaskForm.vue`
* Console du navigateur propre (pas d’erreur rouge)

---

## Annexe B — Structure finale

```
todo-manager/
├─ index.html
├─ package.json
├─ src/
│  ├─ main.js
│  ├─ App.vue
│  └─ components/
│     ├─ TaskForm.vue
│     └─ TaskList.vue
└─ README.md (ce document)
```
