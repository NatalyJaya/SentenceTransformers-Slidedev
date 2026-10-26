<template>
  <div class="ts" role="img" :aria-label="ariaLabel">
    <!-- Shape d'entrada -->
    <div class="ts__shape">
      <span class="ts__paren">(</span>
      <template v-for="(d, i) in dims" :key="i">
        <span class="ts__dim" :class="[`ts__dim--${i}`, { 'is-reduced': i === reduceDim }]">
          <span class="ts__value">{{ d.size }}</span>
          <span class="ts__name">dim={{ i }} · {{ d.name }}</span>
        </span>
        <span v-if="i < dims.length - 1" class="ts__comma">,</span>
      </template>
      <span class="ts__paren">)</span>
    </div>

    <!-- Una frase: tokens × embedding -->
    <div class="ts__sentence">
      <div class="ts__caption">1 frase = {{ dims[1].size }} tokens × {{ dims[2].size }} números</div>
      <div class="ts__grid" :style="{ '--cols': cols }">
        <div v-for="r in rows" :key="r" class="ts__row">
          <div
            v-for="c in cols"
            :key="c"
            class="ts__cell"
            :style="{ opacity: 0.35 + cell(r, c) * 0.65 }"
          />
        </div>
      </div>
    </div>

    <!-- Operació -->
    <div class="ts__op">
      <div class="ts__arrow" />
      <code class="ts__op-name">{{ op }}(dim={{ reduceDim }})</code>
    </div>

    <!-- Resultat -->
    <div class="ts__sentence">
      <div class="ts__grid" :style="{ '--cols': cols }">
        <div class="ts__row">
          <div v-for="c in cols" :key="c" class="ts__cell ts__cell--out" />
        </div>
      </div>
      <div class="ts__caption ts__caption--out">1 vector de {{ dims[2].size }} números</div>
    </div>

    <div class="ts__shape ts__shape--out">
      <span class="ts__paren">(</span>
      <template v-for="(d, i) in outDims" :key="i">
        <span class="ts__dim" :class="`ts__dim--${d.index}`">
          <span class="ts__value">{{ d.size }}</span>
        </span>
        <span v-if="i < outDims.length - 1" class="ts__comma">,</span>
      </template>
      <span class="ts__paren">)</span>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  dims: {
    type: Array,
    default: () => [
      { size: 32, name: 'batch' },
      { size: 50, name: 'tokens' },
      { size: 768, name: 'embedding' },
    ],
  },
  reduceDim: { type: Number, default: 1 },
  op: { type: String, default: '.mean' },
})

// Quadrícula representativa (no a escala)
const rows = 5
const cols = 10

// Valors pseudoaleatoris fixos perquè la quadrícula no canviï a cada render
const cell = (r, c) => ((r * 7 + c * 13) % 10) / 10

const outDims = computed(() =>
  props.dims
    .map((d, index) => ({ ...d, index }))
    .filter((d) => d.index !== props.reduceDim)
)

const ariaLabel = computed(
  () =>
    `Tensor de shape (${props.dims.map((d) => d.size).join(', ')}); ` +
    `${props.op}(dim=${props.reduceDim}) elimina la dimensió dels tokens i ` +
    `queda (${outDims.value.map((d) => d.size).join(', ')}).`
)
</script>

<style scoped>
.ts {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  font-family: ui-monospace, Menlo, monospace;
  color: #f8f8f2;
}

/* ---------- Shape ---------- */
.ts__shape {
  display: flex;
  align-items: flex-start;
  gap: 4px;
  font-size: 22px;
}
.ts__shape--out { font-size: 20px; }
.ts__paren,
.ts__comma { color: #6272a4; }
.ts__dim {
  display: flex;
  flex-direction: column;
  align-items: center;
  transition: opacity 0.3s;
}
.ts__value { font-weight: 700; }
.ts__name {
  font-size: 10px;
  opacity: 0.8;
}
.ts__dim--0 { color: #8be9fd; }
.ts__dim--1 { color: #ffb86c; }
.ts__dim--2 { color: #bd93f9; }
.ts__dim.is-reduced .ts__value {
  text-decoration: underline wavy #ff5555;
  text-underline-offset: 5px;
}

/* ---------- Quadrícula ---------- */
.ts__sentence {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
}
.ts__caption {
  font-size: 11px;
  opacity: 0.75;
}
.ts__caption--out {
  color: #50fa7b;
  opacity: 1;
}
.ts__grid {
  display: flex;
  flex-direction: column;
  gap: 3px;
  padding: 4px;
  border-left: 3px solid #ffb86c;
}
.ts__row {
  display: grid;
  grid-template-columns: repeat(var(--cols), 18px);
  gap: 3px;
}
.ts__cell {
  height: 14px;
  border-radius: 3px;
  background: #bd93f9;
}
.ts__cell--out {
  background: #50fa7b;
}

/* ---------- Operació ---------- */
.ts__op {
  display: flex;
  align-items: center;
  gap: 10px;
}
.ts__op-name {
  font-size: 12px;
  color: #ffb86c;
}
.ts__arrow {
  width: 2px;
  height: 24px;
  background: #6272a4;
  position: relative;
}
.ts__arrow::after {
  content: '';
  position: absolute;
  left: 50%;
  bottom: -2px;
  transform: translateX(-50%);
  border: 6px solid transparent;
  border-top: 8px solid #6272a4;
  border-bottom: 0;
}
</style>
