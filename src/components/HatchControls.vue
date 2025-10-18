<script setup>
const props = defineProps({
  lineAngle: { type: Number, default: 0 },
  hatchStroke: { type: Number, default: 1 },
  hatchSpacing: { type: Number, default: 8 },
})

const emit = defineEmits([
  'update:lineAngle',
  'update:hatchStroke',
  'update:hatchSpacing',
])

const uid = Math.random().toString(36).slice(2, 8)
const angleId = `angle-${uid}`
const strokeId = `hstroke-${uid}`
const spacingId = `hspace-${uid}`
</script>

<template>
  <div class="hatch-controls">
    <div class="row">
      <label :for="angleId">Hatch angle</label>
      <input :id="angleId" type="range" min="0" max="180" step="1"
        :value="lineAngle"
        @input="e => emit('update:lineAngle', +(e.target.value))" />
      <input aria-label="Hatch angle number" type="number" min="0" max="180" step="1"
        :value="lineAngle"
        @input="e => emit('update:lineAngle', +(e.target.value))"
        class="num" />
      <span class="suffix">{{ lineAngle }}°</span>
    </div>

    <div class="row">
      <label :for="strokeId">Hatch stroke</label>
      <input :id="strokeId" type="range" min="1" max="10" step="1"
        :value="hatchStroke"
        @input="e => emit('update:hatchStroke', +(e.target.value))" />
      <input aria-label="Hatch stroke number" type="number" min="1" max="10" step="1"
        :value="hatchStroke"
        @input="e => emit('update:hatchStroke', +(e.target.value))"
        class="num" />
    </div>

    <div class="row">
      <label :for="spacingId">Hatch spacing</label>
      <input :id="spacingId" type="range" min="2" max="40" step="1"
        :value="hatchSpacing"
        @input="e => emit('update:hatchSpacing', +(e.target.value))" />
      <input aria-label="Hatch spacing number" type="number" min="2" max="40" step="1"
        :value="hatchSpacing"
        @input="e => emit('update:hatchSpacing', +(e.target.value))"
        class="num" />
    </div>
  </div>
</template>

<style scoped>
.hatch-controls {
  display: grid;
  gap: 0.5rem;
}
.row {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}
.row label {
  font-size: 0.9rem;
  min-width: 7.5rem;
}
.num {
  width: 4.5rem;
}
.suffix {
  opacity: 0.75;
  font-size: 0.9rem;
}
</style>
