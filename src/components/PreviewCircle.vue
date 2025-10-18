<script setup>
import { computed } from 'vue'

const props = defineProps({
  size: { type: Number, default: 200 },
  r: { type: Number, default: 60 },
  cx: { type: Number, default: null },
  cy: { type: Number, default: null },
  stroke: { type: String, default: '#000' },
  background: { type: String, default: 'white' },
  ariaLabel: { type: String, default: 'Preview circle' },

  // Circle A pattern selection + settings
  aPattern: { type: String, default: 'hatch' }, // 'hatch' | 'concentric'
  aLineAngle: { type: Number, default: 0 },
  aHatchStroke: { type: Number, default: 1 },
  aHatchSpacing: { type: Number, default: 8 },
  aConcStroke: { type: Number, default: 1 },
  aConcSpacing: { type: Number, default: 12 },
  aConcOffsetX: { type: Number, default: 0 },
  aConcOffsetY: { type: Number, default: 0 },

  // Circle B pattern selection + settings
  bPattern: { type: String, default: 'concentric' },
  bLineAngle: { type: Number, default: 0 },
  bHatchStroke: { type: Number, default: 1 },
  bHatchSpacing: { type: Number, default: 8 },
  bConcStroke: { type: Number, default: 1 },
  bConcSpacing: { type: Number, default: 12 },
  bConcOffsetX: { type: Number, default: 0 },
  bConcOffsetY: { type: Number, default: 0 },
})

const uid = Math.random().toString(36).slice(2, 8)
const titleId = `previewTitle-${uid}`
const clipId = `previewClip-${uid}`

// A pattern ids
const aHatchId = `aHatch-${uid}`
const aConcentricId = `aConcentric-${uid}`
// B pattern ids
const bHatchId = `bHatch-${uid}`
const bConcentricId = `bConcentric-${uid}`

const viewBox = computed(() => `0 0 ${props.size} ${props.size}`)
const centerX = computed(() => (props.cx == null ? props.size / 2 : props.cx))
const centerY = computed(() => (props.cy == null ? props.size / 2 : props.cy))

const clampedR = computed(() => {
  const outline = 1
  const maxR = props.size / 2 - outline / 2
  return Math.max(0, Math.min(props.r, maxR))
})

const aOffset = computed(() => Math.hypot(props.aConcOffsetX, props.aConcOffsetY))
const aRings = computed(() => {
  if (props.aConcSpacing <= 0) return 0
  const requiredR = aOffset.value + clampedR.value
  return Math.max(0, Math.floor(requiredR / props.aConcSpacing))
})

const bOffset = computed(() => Math.hypot(props.bConcOffsetX, props.bConcOffsetY))
const bRings = computed(() => {
  if (props.bConcSpacing <= 0) return 0
  const requiredR = bOffset.value + clampedR.value
  return Math.max(0, Math.floor(requiredR / props.bConcSpacing))
})
</script>

<template>
  <svg
    :width="size"
    :height="size"
    :viewBox="viewBox"
    :aria-labelledby="titleId"
    xmlns="http://www.w3.org/2000/svg"
  >
    <title :id="titleId">{{ ariaLabel }}</title>
    <defs>
      <clipPath :id="clipId">
        <rect :width="size" :height="size" />
      </clipPath>

      <!-- A hatch pattern (transparent background for overlay) -->
      <pattern :id="aHatchId" patternUnits="userSpaceOnUse" :width="aHatchSpacing" :height="aHatchSpacing" :patternTransform="`rotate(${aLineAngle})`">
        <line x1="0" y1="0" :x2="aHatchSpacing" y2="0" :stroke="'#000'" :stroke-width="aHatchStroke" />
      </pattern>

      <!-- A concentric pattern (transparent background for overlay) -->
      <pattern :id="aConcentricId" patternUnits="userSpaceOnUse" :width="size * 2" :height="size * 2">
        <g :transform="`translate(${centerX + aConcOffsetX}, ${centerY + aConcOffsetY})`">
          <template v-for="i in aRings" :key="i">
            <circle :cx="0" :cy="0" :r="i * aConcSpacing" fill="none" stroke="#000" :stroke-width="aConcStroke" />
          </template>
        </g>
      </pattern>

      <!-- B hatch pattern (transparent) -->
      <pattern :id="bHatchId" patternUnits="userSpaceOnUse" :width="bHatchSpacing" :height="bHatchSpacing" :patternTransform="`rotate(${bLineAngle})`">
        <line x1="0" y1="0" :x2="bHatchSpacing" y2="0" :stroke="'#000'" :stroke-width="bHatchStroke" />
      </pattern>

      <!-- B concentric pattern (transparent) -->
      <pattern :id="bConcentricId" patternUnits="userSpaceOnUse" :width="size * 2" :height="size * 2">
        <g :transform="`translate(${centerX + bConcOffsetX}, ${centerY + bConcOffsetY})`">
          <template v-for="i in bRings" :key="i">
            <circle :cx="0" :cy="0" :r="i * bConcSpacing" fill="none" stroke="#000" :stroke-width="bConcStroke" />
          </template>
        </g>
      </pattern>
    </defs>

    <g :clip-path="`url(#${clipId})`">
      <rect :width="size" :height="size" :fill="background" />
      <!-- Layer A fill if active -->
      <circle v-if="aPattern === 'hatch'" :cx="centerX" :cy="centerY" :r="clampedR" :fill="`url(#${aHatchId})`" />
      <circle v-else :cx="centerX" :cy="centerY" :r="clampedR" :fill="`url(#${aConcentricId})`" />

      <!-- Layer B fill if active -->
      <circle v-if="bPattern === 'hatch'" :cx="centerX" :cy="centerY" :r="clampedR" :fill="`url(#${bHatchId})`" />
      <circle v-else :cx="centerX" :cy="centerY" :r="clampedR" :fill="`url(#${bConcentricId})`" />

      <!-- Outline on top -->
      <circle :cx="centerX" :cy="centerY" :r="clampedR" :stroke="stroke" stroke-width="1" fill="none" />
    </g>
  </svg>

</template>

<style scoped>
svg { display: block; max-width: 100%; height: auto; }
</style>
