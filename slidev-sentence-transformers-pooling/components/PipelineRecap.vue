<template>
  <div class="pr" role="img" :aria-label="ariaLabel">
    <template v-for="(step, i) in steps" :key="i">
      <div class="pr__step" :class="{ 'is-focus': step.focus }" :style="{ '--c': step.color }">
        <span class="pr__title">{{ step.title }}</span>
        <span class="pr__detail">{{ step.detail }}</span>
      </div>
      <div v-if="i < steps.length - 1" class="pr__arrow" />
    </template>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  steps: {
    type: Array,
    default: () => [
      { title: 'Frase', detail: '"Gat negre"', color: '#8be9fd' },
      { title: 'Transformer', detail: '1 vector per token', color: '#ffb86c' },
      { title: 'Pooling', detail: 'Mean · CLS · Max', color: '#ff79c6', focus: true },
      { title: 'Embedding', detail: '1 vector per frase', color: '#bd93f9' },
      { title: 'Semantic search', detail: 'comparar per significat', color: '#50fa7b' },
    ],
  },
})

const ariaLabel = computed(() => props.steps.map((s) => s.title).join(' → '))
</script>

<style scoped>
.pr {
  display: flex;
  flex-direction: column;
  align-items: center;
  color: #f8f8f2;
}

.pr__step {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-width: 220px;
  padding: 6px 14px;
  border: 2px solid var(--c);
  border-radius: 8px;
}
.pr__step.is-focus {
  background: color-mix(in srgb, var(--c) 20%, transparent);
  transform: scale(1.08);
}
.pr__title {
  font-weight: 700;
  color: var(--c);
}
.pr__detail {
  font-size: 12px;
  opacity: 0.8;
}

.pr__arrow {
  width: 2px;
  height: 18px;
  margin: 2px 0 4px;
  background: #6272a4;
  position: relative;
}
.pr__arrow::after {
  content: '';
  position: absolute;
  left: 50%;
  bottom: -2px;
  transform: translateX(-50%);
  border: 5px solid transparent;
  border-top: 7px solid #6272a4;
  border-bottom: 0;
}
</style>
