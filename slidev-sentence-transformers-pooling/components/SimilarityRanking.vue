<template>
  <div class="sr" role="img" :aria-label="ariaLabel">
    <!-- Pregunta -->
    <div class="sr__query">
      <span class="sr__tag">Pregunta</span>
      <span class="sr__text">"{{ query.text }}"</span>
      <span class="sr__vec">[{{ fmtVec(query.vector) }}]</span>
    </div>

    <div class="sr__op">
      <div class="sr__arrow" />
      <span class="sr__op-name">similitud del cosinus amb cada frase</span>
    </div>

    <!-- Rànquing -->
    <div class="sr__list">
      <div
        v-for="(item, i) in ranked"
        :key="item.text"
        class="sr__item"
        :class="{ 'is-best': i === 0 }"
      >
        <div class="sr__row">
          <span class="sr__rank">{{ i + 1 }}</span>
          <span class="sr__text">"{{ item.text }}"</span>
          <span class="sr__score">{{ item.score.toFixed(2) }}</span>
        </div>
        <div class="sr__bar">
          <div class="sr__fill" :style="{ width: `${Math.max(0, item.score) * 100}%` }" />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  query: {
    type: Object,
    default: () => ({ text: 'Gats negres', vector: [0.12, -0.45, 0.67] }),
  },
  sentences: {
    type: Array,
    default: () => [
      { text: 'Els cotxes negres', vector: [0.5, 0.2, -0.1] },
      { text: 'El gat negre és maco', vector: [0.11, -0.43, 0.65] },
      { text: 'El felí fosc miola', vector: [0.2, -0.3, 0.6] },
    ],
  },
})

const dot = (a, b) => a.reduce((s, x, i) => s + x * b[i], 0)
const norm = (a) => Math.sqrt(dot(a, a))
const cosine = (a, b) => dot(a, b) / (norm(a) * norm(b))

const ranked = computed(() =>
  props.sentences
    .map((s) => ({ ...s, score: cosine(props.query.vector, s.vector) }))
    .sort((a, b) => b.score - a.score)
)

const fmtVec = (v) => v.map((x) => x.toFixed(2)).join(', ')

const ariaLabel = computed(
  () =>
    `Rànquing per similitud amb "${props.query.text}": ` +
    ranked.value.map((r) => `"${r.text}" ${r.score.toFixed(2)}`).join(', ') +
    '.'
)
</script>

<style scoped>
.sr {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  gap: 8px;
  width: 100%;
  max-width: 420px;
  font-size: 13px;
  color: #f8f8f2;
}

.sr__query {
  display: flex;
  flex-wrap: wrap;
  align-items: baseline;
  gap: 8px;
  padding: 8px 12px;
  border: 2px solid #8be9fd;
  border-radius: 8px;
}
.sr__tag {
  font-size: 11px;
  font-weight: 700;
  color: #8be9fd;
  text-transform: uppercase;
}
.sr__vec {
  font-family: ui-monospace, Menlo, monospace;
  font-size: 11px;
  opacity: 0.75;
}

.sr__op {
  display: flex;
  align-items: center;
  gap: 10px;
  padding-left: 20px;
}
.sr__op-name {
  font-family: ui-monospace, Menlo, monospace;
  font-size: 12px;
  color: #ffb86c;
}
.sr__arrow {
  width: 2px;
  height: 24px;
  background: #6272a4;
  position: relative;
}
.sr__arrow::after {
  content: '';
  position: absolute;
  left: 50%;
  bottom: -2px;
  transform: translateX(-50%);
  border: 6px solid transparent;
  border-top: 8px solid #6272a4;
  border-bottom: 0;
}

.sr__list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.sr__item {
  padding: 6px 10px;
  border: 2px solid transparent;
  border-radius: 8px;
  opacity: 0.7;
}
.sr__item.is-best {
  border-color: #50fa7b;
  opacity: 1;
}
.sr__row {
  display: flex;
  align-items: baseline;
  gap: 8px;
}
.sr__rank {
  font-weight: 700;
  color: #bd93f9;
}
.sr__score {
  margin-left: auto;
  font-family: ui-monospace, Menlo, monospace;
  font-weight: 700;
}
.sr__bar {
  margin-top: 4px;
  height: 8px;
  border-radius: 4px;
  background: #44475a;
  overflow: hidden;
}
.sr__fill {
  height: 100%;
  background: #bd93f9;
}
.sr__item.is-best .sr__fill { background: #50fa7b; }
</style>
