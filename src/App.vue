<template>
  <div class="app">
    <div class="top-bar">
      <h1>{{ t.title }}</h1>
      <button class="reset-btn" @click="reset">{{ t.reset }}</button>
    </div>
    <div class="table">
      <div class="row header">
        <div class="cell cell-type"></div>
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
  </div>
</template>

<script setup>
import { reactive, ref, computed } from 'vue'

const isFrench = navigator.language.startsWith('fr')

const t = {
  title:     isFrench ? 'Score Akropolis' : 'Akropolis Score',
  reset:     isFrench ? 'Réinitialiser'   : 'Reset',
  plazas:    isFrench ? 'Places'          : 'Plazas',
  districts: isFrench ? 'Quartiers'       : 'Districts',
  stones:    isFrench ? 'Pierres'         : 'Stones',
  total:     'Total',
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

function inc(row, field) { row[field]++ }
function dec(row, field) { if (row[field] > 0) row[field]-- }
function decStones() { if (stones.value > 0) stones.value-- }
function reset() {
  rows.forEach(row => { row.plazas = 0; row.districts = 0 })
  stones.value = 0
}
</script>

<style>
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

body {
  font-family: system-ui, sans-serif;
  background: #111;
  color: #fff;
  min-height: 100svh;
  display: flex;
  justify-content: center;
  padding: 1.5rem 1rem 3rem;
}

.app {
  width: 100%;
  max-width: 600px;
}

.top-bar {
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  margin-bottom: 1.5rem;
}

h1 {
  font-size: 1.5rem;
  font-weight: 700;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  color: #e2c97e;
}

.reset-btn {
  position: absolute;
  right: 0;
  background: none;
  border: none;
  color: #888;
  font-size: 0.8rem;
  cursor: pointer;
  text-decoration: underline;
  text-underline-offset: 3px;
  padding: 0;
}

.reset-btn:hover { color: #ccc; }

.table {
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 8px 32px rgba(0,0,0,0.5);
}

.row {
  display: grid;
  grid-template-columns: 5rem 1fr 5.5rem 4rem;
  align-items: center;
}

.header {
  background: #1e1e1e;
  font-size: 0.7rem;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: #888;
  padding: 0.5rem 0;
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
  background: #1e1e1e;
  border-top: 2px solid #e2c97e;
  min-height: 60px;
}

.total-row .cell-score {
  font-size: 1.5rem;
  color: #e2c97e;
}

.total-row .cell-type {
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: #e2c97e;
}

/* Responsive */
@media (max-width: 400px) {
  .row {
    grid-template-columns: 4.5rem 1fr 4.5rem 3rem;
  }

  .cell { padding: 0.4rem 0.4rem; }

  .cell-type { font-size: 0.7rem; }

  .num { font-size: 1rem; }

  .arrow { width: 24px; }
}
</style>
