<template>
  <div
    ref="viewportRef"
    class="pooling-viewport"
    role="img"
    :aria-label="ariaLabel"
  >
    <div ref="stageRef" class="pooling-diagram" :style="stageStyle">
      <!-- Frase de entrada -->
      <div class="node node--input">Frase: '{{ phrase }}'</div>
      <div class="arrow" />

      <!-- Modelo -->
      <div class="node node--model">{{ model }}</div>
      <div class="line" />

      <!-- Tokens (se bifurca en tantas columnas como tokens haya) -->
      <div class="branch" :style="{ '--cols': tokens.length }">
        <div class="branch__bar branch__bar--top" />
        <div v-for="(token, i) in tokens" :key="i" class="branch__col">
          <div class="arrow arrow--short" />
          <div class="node node--token">
            Token {{ i + 1 }}: [{{ formatVector(token.vector) }}]
            <span class="node__meta">({{ dimensions }} números)</span>
          </div>
          <div class="line line--fill" />
        </div>
        <div class="branch__bar branch__bar--bottom" />
      </div>
      <div class="arrow" />

      <!-- Decisión -->
      <div class="decision">
        <div class="decision__inner">{{ question }}</div>
      </div>
      <div class="arrow" />

      <!-- Resultado -->
      <div class="node node--output">
        Vector único que<br />representa la frase
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref, onMounted, onBeforeUnmount } from 'vue'

const props = defineProps({
  phrase: { type: String, default: 'Hola mundo' },
  model: { type: String, default: 'Transformer' },
  tokens: {
    type: Array,
    default: () => [
      { vector: [0.1, 0.2, 0.3, 0.5] },
      { vector: [0.4, 0.5, 0.6, 0.8] },
    ],
  },
  dimensions: { type: Number, default: 768 },
  question: { type: String, default: '¿POOLING?' },
})

// [0.1, 0.2, 0.3, 0.5] -> "0.1, 0.2, 0.3, ..., 0.5"
function formatVector(v) {
  if (v.length < 2) return v.join(', ')
  return [...v.slice(0, -1), '...', v[v.length - 1]].join(', ')
}

const ariaLabel = computed(
  () =>
    `La frase "${props.phrase}" pasa por un ${props.model}, que genera ` +
    `${props.tokens.length} vectores de ${props.dimensions} números; ` +
    `el pooling los combina en un único vector que representa la frase.`
)

const viewportRef = ref(null)
const stageRef = ref(null)
const scale = ref(1)

const stageStyle = computed(() => ({
  transform: `scale(${scale.value})`,
}))

// Ajusta el diagrama al espacio disponible sin deformarlo.
function fit() {
  const viewport = viewportRef.value
  const stage = stageRef.value
  if (!viewport || !stage) return

  const availableWidth = viewport.clientWidth
  const availableHeight = viewport.clientHeight
  // offsetWidth/Height no se ven afectados por el transform.
  const naturalWidth = stage.offsetWidth
  const naturalHeight = stage.offsetHeight

  if (!availableWidth || !availableHeight || !naturalWidth || !naturalHeight) {
    return
  }

  scale.value = Math.min(
    1,
    availableWidth / naturalWidth,
    availableHeight / naturalHeight
  )
}

let resizeObserver = null

onMounted(() => {
  fit()
  if (typeof ResizeObserver !== 'undefined') {
    resizeObserver = new ResizeObserver(fit)
    if (viewportRef.value) resizeObserver.observe(viewportRef.value)
    if (stageRef.value) resizeObserver.observe(stageRef.value)
  }
  document.fonts?.ready.then(fit)
})

onBeforeUnmount(() => {
  resizeObserver?.disconnect()
  resizeObserver = null
})
</script>

<style scoped>
/* Hueco reservado en la diapositiva; recorta solo el espacio sobrante. */
.pooling-viewport {
  width: 100%;
  height: 340px;
  display: flex;
  align-items: flex-start;
  justify-content: center;
  overflow: hidden;
}

.pooling-diagram {
  --line: #555;
  --arrow-h: 16px;
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  max-width: 640px;
  flex: 0 0 auto;
  padding: 8px;
  font-family: system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif;
  font-size: 12px;
  color: #000;
  transform-origin: top center;
  will-change: transform;
}

/* ---------- Nodos ---------- */
.node {
  padding: 7px 13px;
  border: 2px solid;
  border-radius: 6px;
  text-align: center;
  line-height: 1.4;
}
.node__meta {
  display: block;
  font-size: 10px;
  opacity: 0.75;
}
.node--input  { background: #e1f5ff; border-color: #0288d1; }
.node--model  { background: #fff3e0; border-color: #f57c00; }
.node--token  {
  background: #f3e5f5;
  border-color: #7b1fa2;
  font-family: ui-monospace, Menlo, monospace;
  font-size: 11px;
  padding: 7px 9px;
}
.node--output { background: #e8f5e9; border-color: #388e3c; }

/* ---------- Conectores ---------- */
.line,
.arrow {
  width: 2px;
  height: var(--arrow-h);
  background: var(--line);
  position: relative;
  flex-shrink: 0;
}
.arrow { margin-bottom: 2px; }
.arrow::after {
  content: '';
  position: absolute;
  left: 50%;
  bottom: -2px;
  transform: translateX(-50%);
  border: 6px solid transparent;
  border-top: 8px solid var(--line);
  border-bottom: 0;
}
.arrow--short { height: 11px; }
.line--fill { flex: 1; min-height: 10px; }

/* ---------- Bifurcación / unión ---------- */
.branch {
  position: relative;
  display: grid;
  grid-template-columns: repeat(var(--cols), 1fr);
  width: 100%;
}
.branch__col {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 0 8px;
}
.branch__bar {
  position: absolute;
  height: 2px;
  background: var(--line);
  /* va del centro de la primera columna al centro de la última */
  left: calc(50% / var(--cols));
  right: calc(50% / var(--cols));
}
.branch__bar--top    { top: 0; }
.branch__bar--bottom { bottom: 0; }

/* ---------- Rombo de decisión ---------- */
.decision {
  position: relative;
  width: 160px;
  height: 76px;
  background: #fbc02d; /* borde */
  clip-path: polygon(50% 0, 100% 50%, 50% 100%, 0 50%);
}
.decision__inner {
  position: absolute;
  inset: 4px 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #fff9c4;
  clip-path: polygon(50% 0, 100% 50%, 50% 100%, 0 50%);
  font-weight: 600;
}
</style>