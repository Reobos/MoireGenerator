<script setup>
const props = defineProps({
  concentricSpacing: { type: Number, default: 12 },
  concentricStroke: { type: Number, default: 1 },
  concentricOffsetX: { type: Number, default: 0 },
  concentricOffsetY: { type: Number, default: 0 },
})

const emit = defineEmits([
  'update:concentricSpacing',
  'update:concentricStroke',
  'update:concentricOffsetX',
  'update:concentricOffsetY',
])

const uid = Math.random().toString(36).slice(2, 8)
const spacingId = `cspace-${uid}`
const strokeId = `cstroke-${uid}`
const offsetXId = `coffx-${uid}`
const offsetYId = `coffy-${uid}`
</script>

<template>
  <div class="concentric-controls">
    <div class="row">
      <label :for="spacingId">Ring spacing</label>
      <input :id="spacingId" type="range" min="2" max="40" step="1"
        :value="concentricSpacing"
        @input="e => emit('update:concentricSpacing', +(e.target.value))" />
      <input aria-label="Ring spacing number" type="number" min="2" max="40" step="1"
        :value="concentricSpacing"
        @input="e => emit('update:concentricSpacing', +(e.target.value))"
        class="num" />
    </div>

    <div class="row">
      <label :for="strokeId">Ring stroke</label>
      <input :id="strokeId" type="range" min="1" max="10" step="1"
        :value="concentricStroke"
        @input="e => emit('update:concentricStroke', +(e.target.value))" />
      <input aria-label="Ring stroke number" type="number" min="1" max="10" step="1"
        :value="concentricStroke"
        @input="e => emit('update:concentricStroke', +(e.target.value))"
        class="num" />
    </div>

    <div class="row">
      <label :for="offsetXId">Offset X</label>
      <input :id="offsetXId" type="range" min="-200" max="200" step="1"
        :value="concentricOffsetX"
        @input="e => emit('update:concentricOffsetX', +(e.target.value))" />
      <input aria-label="Offset X number" type="number" min="-1000" max="1000" step="1"
        :value="concentricOffsetX"
        @input="e => emit('update:concentricOffsetX', +(e.target.value))"
        class="num" />
    </div>

    <div class="row">
      <label :for="offsetYId">Offset Y</label>
      <input :id="offsetYId" type="range" min="-200" max="200" step="1"
        :value="concentricOffsetY"
        @input="e => emit('update:concentricOffsetY', +(e.target.value))" />
      <input aria-label="Offset Y number" type="number" min="-1000" max="1000" step="1"
        :value="concentricOffsetY"
        @input="e => emit('update:concentricOffsetY', +(e.target.value))"
        class="num" />
    </div>
  </div>
</template>

<style scoped>
.concentric-controls {
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
</style>
