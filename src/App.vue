<template>
  <div class="app">
    <div class="table">
      <div class="row header">
        <div class="cell cell-type title">{{ t.title }}</div>
        <div class="cell cell-plazas">{{ t.plazas }}</div>
        <div class="cell cell-districts">{{ t.districts }}</div>
        <div class="cell cell-score"></div>
      </div>

      <div
        v-for="row in rows"
        :key="row.name"
        class="row district-row"
        :style="{ '--row-bg': row.bg, '--row-text': row.text }"
      >
        <div class="cell cell-type">{{ row.name }}</div>

        <div class="cell cell-plazas">
          <div class="counter">
            <button class="arrow" @click="inc(row, 'plazas')">▲</button>
            <div class="plaza-value">
              <span class="num">{{ row.plazas }}</span>
              <span class="times">×</span>
              <span class="badge">
                <span v-for="i in row.stars" :key="i" class="star">★</span>
              </span>
            </div>
            <button class="arrow" @click="dec(row, 'plazas')">▼</button>
          </div>
        </div>

        <div class="cell cell-districts">
          <div class="counter">
            <button class="arrow" @click="inc(row, 'districts')">▲</button>
            <span class="num">{{ row.districts }}</span>
            <button class="arrow" @click="dec(row, 'districts')">▼</button>
          </div>
        </div>

        <div class="cell cell-score">{{ score(row) }}</div>
      </div>

      <div class="row stones-row">
        <div class="cell cell-type">{{ t.stones }}</div>
        <div class="cell cell-plazas"></div>
        <div class="cell cell-districts">
          <div class="counter">
            <button class="arrow" @click="stones++">▲</button>
            <span class="num">{{ stones }}</span>
            <button class="arrow" @click="decStones">▼</button>
          </div>
        </div>
        <div class="cell cell-score">{{ stones }}</div>
      </div>

      <div class="row total-row">
        <div class="cell cell-type">{{ t.total }}</div>
        <div class="cell cell-plazas"></div>
        <div class="cell cell-districts"></div>
        <div class="cell cell-score">{{ total }}</div>
      </div>
    </div>

    <div class="history">
      <span class="history-label">{{ t.history }}</span>
      <button
        v-for="(entry, i) in history"
        :key="i"
        class="history-btn"
        :class="{ active: activeEntry === entry }"
        @click="restore(entry)"
      >{{ entry.total }}</button>
      <button class="new-btn" @click="newEntry" :title="t.newEntry">＋</button>
      <button class="trash-btn" @click="clearHistory" :title="t.clearHistory">🗑</button>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref, computed, watchEffect } from 'vue'

const isFrench = navigator.language.startsWith('fr')

const t = {
  title:          isFrench ? 'Score Akropolis'              : 'Akropolis Score',
  plazas:         isFrench ? 'Places'                       : 'Plazas',
  districts:      isFrench ? 'Quartiers'                    : 'Districts',
  stones:         isFrench ? 'Pierres'                      : 'Stones',
  total:          'Total',
  history:      isFrench ? 'Scores'        : 'Scores',
  newEntry:     isFrench ? 'Nouveau score' : 'New score',
  clearHistory: isFrench ? 'Tout supprimer': 'Clear all',
}

const DISTRICT_NAMES = isFrench
  ? ['Maisons', 'Marchés', 'Casernes', 'Temples', 'Jardins']
  : ['Houses',  'Markets', 'Barracks', 'Temples', 'Gardens']

const rows = reactive([
  { name: DISTRICT_NAMES[0], stars: 1, bg: '#2563EB', text: '#fff',    plazas: 0, districts: 0 },
  { name: DISTRICT_NAMES[1], stars: 2, bg: '#CA8A04', text: '#1a1a1a', plazas: 0, districts: 0 },
  { name: DISTRICT_NAMES[2], stars: 2, bg: '#DC2626', text: '#fff',    plazas: 0, districts: 0 },
  { name: DISTRICT_NAMES[3], stars: 2, bg: '#7C3AED', text: '#fff',    plazas: 0, districts: 0 },
  { name: DISTRICT_NAMES[4], stars: 3, bg: '#16A34A', text: '#fff',    plazas: 0, districts: 0 },
])

const stones = ref(0)

function score(row) {
  if (row.plazas === 0) return 0
  return row.districts * row.plazas * row.stars
}

const total = computed(() =>
  rows.reduce((sum, row) => sum + score(row), 0) + stones.value
)

const history = reactive([])
const activeEntry = ref(null)

function applyEntry(entry) {
  entry.snapshot.forEach((s, i) => { rows[i].plazas = s.plazas; rows[i].districts = s.districts })
  stones.value = entry.stones
  activeEntry.value = entry
}

function newEntry() {
  const entry = { snapshot: rows.map(() => ({ plazas: 0, districts: 0 })), stones: 0, total: 0 }
  history.push(entry)
  applyEntry(entry)
}

// Persist state across reloads
try {
  const saved = JSON.parse(localStorage.getItem('akropolis-state'))
  if (saved?.history?.length) {
    history.push(...saved.history)
    applyEntry(history[Math.max(0, saved.activeEntryIndex ?? 0)])
  }
} catch (e) {}

if (history.length === 0) newEntry()

// Auto-save active entry on every change
watchEffect(() => {
  if (!activeEntry.value) return
  activeEntry.value.snapshot = rows.map(r => ({ plazas: r.plazas, districts: r.districts }))
  activeEntry.value.stones = stones.value
  activeEntry.value.total = total.value
})

watchEffect(() => {
  localStorage.setItem('akropolis-state', JSON.stringify({
    history,
    activeEntryIndex: activeEntry.value ? history.indexOf(activeEntry.value) : 0,
  }))
})

function inc(row, field) { row[field]++ }
function dec(row, field) { if (row[field] > 0) row[field]-- }
function decStones() { if (stones.value > 0) stones.value-- }

function restore(entry) {
  applyEntry(entry)
}

function clearHistory() {
  history.splice(0)
  activeEntry.value = null
  newEntry()
}
</script>

<style>
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; user-select: none; }

body {
  font-family: system-ui, sans-serif;
  color: #fff;
  min-height: 100svh;
  display: flex;
  justify-content: center;
  background: #111;
  padding: 1rem 1rem 3rem;
}

.app {
  width: min(100%, 600px);
  min-width: 0;
}

.table {
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 8px 32px rgba(0,0,0,0.3);
}

.row {
  display: grid;
  grid-template-columns: 5rem 1fr 5.5rem 4rem;
  align-items: center;
}

/* Header row doubles as title */
.header {
  background: #1a1a2e;
  padding: 0.6rem 0;
}

.header .cell-plazas,
.header .cell-districts {
  font-size: 0.7rem;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: #888;
}

.title {
  font-size: 1rem;
  font-weight: 700;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  color: #e2c97e;
}

.cell {
  padding: 0.5rem 0.75rem;
  display: flex;
  align-items: center;
  justify-content: center;
}

.cell-type {
  justify-content: flex-start;
  font-size: 0.8rem;
  font-weight: 600;
  letter-spacing: 0.03em;
}

.cell-score {
  font-size: 1.2rem;
  font-weight: 700;
  justify-content: flex-end;
}

/* District rows */
.district-row {
  background-color: var(--row-bg);
  color: var(--row-text);
  border-top: 1px solid rgba(0,0,0,0.15);
  min-height: 70px;
}

/* Counter */
.counter {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
}

.arrow {
  background: rgba(0,0,0,0.2);
  border: none;
  color: inherit;
  font-size: 0.65rem;
  width: 28px;
  height: 20px;
  border-radius: 4px;
  cursor: pointer;
  line-height: 1;
  transition: background 0.1s;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0;
  padding-bottom: 1px;
}

.arrow:hover { background: rgba(0,0,0,0.35); }
.arrow:active { background: rgba(0,0,0,0.5); }

.num {
  font-size: 1.1rem;
  font-weight: 700;
  min-width: 1.5rem;
  text-align: center;
  line-height: 1.4;
}

/* Plaza value display */
.plaza-value {
  display: flex;
  align-items: center;
  gap: 4px;
}

.times {
  font-size: 0.75rem;
  opacity: 0.7;
}

.badge {
  display: inline-flex;
  align-items: center;
  gap: 1px;
  background: rgba(0,0,0,0.2);
  border: 1.5px solid rgba(255,255,255,0.35);
  border-radius: 5px;
  padding: 2px 5px;
  min-width: 24px;
  justify-content: center;
}

.district-row[style*="CA8A04"] .badge,
.district-row[style*="1a1a1a"] .badge {
  border-color: rgba(0,0,0,0.3);
}

.star {
  font-size: 0.75rem;
  color: #FFD700;
  text-shadow: 0 0 3px rgba(0,0,0,0.4);
}

/* Stones row */
.stones-row {
  background: #374151;
  color: #e5e7eb;
  border-top: 1px solid rgba(0,0,0,0.3);
  min-height: 70px;
}

/* Total row */
.total-row {
  background: #1a1a2e;
  border-top: 2px solid #e2c97e;
  min-height: 60px;
}

.total-row .cell-score {
  font-size: 1.5rem;
  color: #e2c97e;
}

.total-row .cell-type {
  color: #e2c97e;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.1em;
}

/* History */
.history {
  margin-top: 0.5rem;
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 0.4rem;
  padding-left: 0.25rem;
  width: 100%;
  overflow: hidden;
}

.history-label {
  font-size: 0.7rem;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: #555;
  margin-right: 0.2rem;
}

.history-btn, .new-btn, .trash-btn {
  height: 28px;
  border-radius: 6px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0 0.5rem;
  font-size: 0.85rem;
  font-weight: 700;
  line-height: 1;
  transition: background 0.1s, border-color 0.1s, color 0.1s;
}

.history-btn {
  background: #1a1a2e;
  border: 1px solid #333;
  color: #e2c97e;
}

.history-btn:hover, .history-btn.active {
  background: #252540;
  border-color: #e2c97e;
}

.new-btn {
  background: none;
  border: 1px solid #444;
  color: #888;
}

.new-btn:hover { border-color: #e2c97e; color: #e2c97e; }

.trash-btn {
  background: none;
  border: 1px solid #444;
  color: #888;
}

.trash-btn:hover { border-color: #ef4444; color: #ef4444; }

/* Responsive */
@media (max-width: 400px) {
  .row {
    grid-template-columns: 4.5rem 1fr 4.5rem 3rem;
  }

  .cell { padding: 0.4rem 0.4rem; }

  .cell-type { font-size: 0.7rem; }

  .num { font-size: 1rem; }

  .arrow { width: 24px; }

  .title { font-size: 0.8rem; }
}
</style>
