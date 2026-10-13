<script setup>
import { ref, computed, onMounted, watch } from 'vue'

// ESTADO REACTIVO
const entries = ref([])
const newMood = ref('feliz')
const newActivity = ref('ejercicio')
const newNote = ref('')

// Opciones disponibles
const moodOptions = [
  { value: 'feliz', label: '😊 Feliz', color: '#ffd1dc' },
  { value: 'tranquila', label: '🌿 Tranquila', color: '#d4f0f0' },
  { value: 'ansiosa', label: '☁️ Ansiosa', color: '#e6e6fa' },
  { value: 'cansada', label: '🛋️ Cansada', color: '#fff0f5' },
  { value: 'motivada', label: '✨ Motivada', color: '#ffe4e8' }
]

const activityOptions = [
  { value: 'sueño', label: '😴 Buen Sueño' },
  { value: 'insomnio', label: '👀 Mal Sueño' },
  { value: 'ejercicio', label: '🏃‍♀️ Ejercicio' },
  { value: 'estudio', label: '📚 Trabajo/Estudio' },
  { value: 'social', label: '👯‍♀️ Con Amigos/Familia' }
]

// Cargar datos guardados
onMounted(() => {
  const saved = localStorage.getItem('mind_patterns_data')
  if (saved) {
    entries.value = JSON.parse(saved)
  }
})

// Guardar en localStorage automáticamente al cambiar entradas
watch(entries, (newVal) => {
  localStorage.setItem('mind_patterns_data', JSON.stringify(newVal))
}, { deep: true })

// MANEJO DE EVENTOS: Agregar registro
const addEntry = () => {
  if (!newMood.value) return
  
  entries.value.unshift({
    id: Date.now(),
    date: new Date().toLocaleDateString('es-ES', { weekday: 'short', day: 'numeric', month: 'short' }),
    mood: newMood.value,
    activity: newActivity.value,
    note: newNote.value
  })

  // Limpiar campos
  newNote.value = ''
}

// PROPIEDADES COMPUTADAS (Análisis de Patrones Reactivos)
const totalEntries = computed(() => entries.value.length)

const topMood = computed(() => {
  if (entries.value.length === 0) return 'Sin datos'
  const counts = {}
  entries.value.forEach(e => counts[e.mood] = (counts[e.mood] || 0) + 1)
  const sorted = Object.keys(counts).sort((a,b) => counts[b] - counts[a])
  return sorted[0].toUpperCase()
})

const patternInsight = computed(() => {
  if (entries.value.length < 2) return 'Registra al menos 2 días para descubrir tus patrones de comportamiento.'
  
  const badSleepAndAnxious = entries.value.filter(e => e.activity === 'insomnio' && (e.mood === 'ansiosa' || e.mood === 'cansada')).length
  const exerciseAndHappy = entries.value.filter(e => e.activity === 'ejercicio' && (e.mood === 'feliz' || e.mood === 'motivada')).length

  if (exerciseAndHappy > 0) {
    return '💡 Patrón detectado: Hacer ejercicio se relaciona directamente con tus días más felices y motivados.'
  } else if (badSleepAndAnxious > 0) {
    return '💡 Patrón detectado: El mal sueño o insomnio está detonando tus estados de cansancio y ansiedad.'
  }
  return '💡 Consejo: Sigue registrando tus actividades diarias para revelar correlaciones ocultas.'
})

const deleteEntry = (id) => {
  entries.value = entries.value.filter(e => e.id !== id)
}
</script>

<template>
  <div class="app-container">
    <header class="header">
      <h1>🧠 MindPatterns</h1>
      <p>Registra tu ánimo y descubre patrones en tu comportamiento</p>
    </header>

    <main class="main-grid">
      <!-- FORMULARIO -->
      <section class="card form-card">
        <h2>✨ Nuevo Registro</h2>
        <form @submit.prevent="addEntry">
          <div class="form-group">
            <label>¿Cómo te sientes hoy?</label>
            <select v-model="newMood">
              <option v-for="m in moodOptions" :key="m.value" :value="m.value">{{ m.label }}</option>
            </select>
          </div>

          <div class="form-group">
            <label>Factor / Actividad clave:</label>
            <select v-model="newActivity">
              <option v-for="a in activityOptions" :key="a.value" :value="a.value">{{ a.label }}</option>
            </select>
          </div>

          <div class="form-group">
            <label>Nota reflexiva (opcional):</label>
            <textarea v-model="newNote" rows="2" placeholder="¿Qué ocurrió hoy?"></textarea>
          </div>

          <button type="submit" class="btn-primary">Guardar Registro</button>
        </form>
      </section>

      <!-- ANÁLISIS DE PATRONES (REACTIVO) -->
      <section class="card analysis-card">
        <h2>📊 Análisis de Patrones</h2>
        <div class="stats-grid">
          <div class="stat-box">
            <span class="stat-num">{{ totalEntries }}</span>
            <span class="stat-label">Registros</span>
          </div>
          <div class="stat-box">
            <span class="stat-num">{{ topMood }}</span>
            <span class="stat-label">Ánimo Predominante</span>
          </div>
        </div>
        <div class="insight-box">
          <p>{{ patternInsight }}</p>
        </div>
      </section>

      <!-- HISTORIAL -->
      <section class="card history-card">
        <h2>📜 Historial de Registros</h2>
        <div v-if="entries.length === 0" class="empty-msg">
          Aún no hay registros. ¡Añade el primero arriba!
        </div>
        <div v-else class="entries-list">
          <div v-for="item in entries" :key="item.id" class="entry-item">
            <div class="entry-header">
              <span class="entry-date">{{ item.date }}</span>
              <button @click="deleteEntry(item.id)" class="btn-delete">×</button>
            </div>
            <div class="entry-body">
              <span class="badge mood-badge">{{ item.mood }}</span>
              <span class="badge act-badge">{{ item.activity }}</span>
              <p v-if="item.note" class="entry-note">"{{ item.note }}"</p>
            </div>
          </div>
        </div>
      </section>
    </main>
  </div>
</template>

<style>
/* ESTILOS ROSA PASTEL Y MINIMALISTAS */
:root {
  --bg-color: #fde8ed;
  --card-bg: #ffffff;
  --primary-pink: #f48fb1;
  --text-color: #4a4a4a;
  --border-color: #f8bbd0;
}

body {
  font-family: 'Poppins', system-ui, sans-serif;
  background-color: var(--bg-color);
  color: var(--text-color);
  margin: 0;
  padding: 20px;
}

.app-container {
  max-width: 800px;
  margin: 0 auto;
}

.header {
  text-align: center;
  margin-bottom: 25px;
}

.header h1 {
  color: #c2185b;
  margin: 0;
  font-size: 32px;
}

.header p {
  color: #886b75;
  font-size: 14px;
}

.main-grid {
  display: grid;
  gap: 20px;
}

.card {
  background: var(--card-bg);
  padding: 22px;
  border-radius: 16px;
  border: 2px dashed var(--border-color);
  box-shadow: 0 4px 12px rgba(244, 143, 177, 0.12);
}

.card h2 {
  font-size: 18px;
  color: #c2185b;
  margin-top: 0;
}

.form-group {
  margin-bottom: 12px;
  display: flex;
  flex-direction: column;
}

.form-group label {
  font-size: 12px;
  font-weight: 600;
  margin-bottom: 4px;
}

select, textarea, input {
  padding: 10px;
  border-radius: 8px;
  border: 1px solid var(--border-color);
  font-family: inherit;
  outline: none;
}

.btn-primary {
  background: var(--primary-pink);
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  width: 100%;
}

.btn-primary:hover {
  background: #e91e63;
}

.stats-grid {
  display: flex;
  gap: 15px;
  margin-bottom: 15px;
}

.stat-box {
  flex: 1;
  background: #fff0f3;
  padding: 12px;
  border-radius: 10px;
  text-align: center;
}

.stat-num {
  display: block;
  font-size: 20px;
  font-weight: bold;
  color: #c2185b;
}

.stat-label {
  font-size: 11px;
  color: #886b75;
}

.insight-box {
  background: #fffbea;
  border: 1px solid #ffe082;
  padding: 12px;
  border-radius: 10px;
  font-size: 12px;
}

.entries-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.entry-item {
  background: #fff8f9;
  border: 1px solid #fce3e7;
  padding: 10px 14px;
  border-radius: 10px;
}

.entry-header {
  display: flex;
  justify-content: space-between;
  font-size: 11px;
  color: #aaa;
}

.btn-delete {
  background: none;
  border: none;
  color: #f44336;
  cursor: pointer;
  font-size: 16px;
}

.badge {
  display: inline-block;
  font-size: 11px;
  padding: 2px 8px;
  border-radius: 12px;
  margin-right: 6px;
  margin-top: 5px;
  text-transform: capitalize;
}

.mood-badge { background: #f8bbd0; color: #880e4f; }
.act-badge { background: #e1bee7; color: #4a148c; }

.entry-note {
  font-size: 12px;
  font-style: italic;
  margin: 6px 0 0 0;
  color: #666;
}

.empty-msg {
  font-size: 12px;
  color: #aaa;
  text-align: center;
}
</style>