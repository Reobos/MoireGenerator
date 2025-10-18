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
  baseRefRadius: { type: Number, default: 150 },

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

// Keep pattern scale relative to circle radius using a base reference radius (configurable)
const scale = computed(() => {
  const base = props.baseRefRadius > 0 ? props.baseRefRadius : 150
  return clampedR.value / base
})

// Scaled A values
const aScaledHatchSpacing = computed(() => props.aHatchSpacing * scale.value)
const aScaledHatchStroke = computed(() => props.aHatchStroke * scale.value)
const aScaledConcSpacing = computed(() => props.aConcSpacing * scale.value)
const aScaledConcStroke = computed(() => props.aConcStroke * scale.value)
const aScaledOffsetX = computed(() => props.aConcOffsetX * scale.value)
const aScaledOffsetY = computed(() => props.aConcOffsetY * scale.value)
const aOffset = computed(() => Math.hypot(aScaledOffsetX.value, aScaledOffsetY.value))
const aRings = computed(() => {
  if (aScaledConcSpacing.value <= 0) return 0
  const requiredR = aOffset.value + clampedR.value
  return Math.max(0, Math.floor(requiredR / aScaledConcSpacing.value))
})

// Scaled B values
const bScaledHatchSpacing = computed(() => props.bHatchSpacing * scale.value)
const bScaledHatchStroke = computed(() => props.bHatchStroke * scale.value)
const bScaledConcSpacing = computed(() => props.bConcSpacing * scale.value)
const bScaledConcStroke = computed(() => props.bConcStroke * scale.value)
const bScaledOffsetX = computed(() => props.bConcOffsetX * scale.value)
const bScaledOffsetY = computed(() => props.bConcOffsetY * scale.value)
const bOffset = computed(() => Math.hypot(bScaledOffsetX.value, bScaledOffsetY.value))
const bRings = computed(() => {
  if (bScaledConcSpacing.value <= 0) return 0
  const requiredR = bOffset.value + clampedR.value
  return Math.max(0, Math.floor(requiredR / bScaledConcSpacing.value))
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
      <pattern :id="aHatchId" patternUnits="userSpaceOnUse" :width="aScaledHatchSpacing" :height="aScaledHatchSpacing" :patternTransform="`rotate(${aLineAngle})`">
        <line x1="0" y1="0" :x2="aScaledHatchSpacing" y2="0" :stroke="'#000'" :stroke-width="aScaledHatchStroke" />
      </pattern>

      <!-- A concentric pattern (transparent background for overlay) -->
      <pattern :id="aConcentricId" patternUnits="userSpaceOnUse" :width="size * 2" :height="size * 2">
        <g :transform="`translate(${centerX + aScaledOffsetX}, ${centerY + aScaledOffsetY})`">
          <template v-for="i in aRings" :key="i">
            <circle :cx="0" :cy="0" :r="i * aScaledConcSpacing" fill="none" stroke="#000" :stroke-width="aScaledConcStroke" />
          </template>
        </g>
      </pattern>

      <!-- B hatch pattern (transparent) -->
      <pattern :id="bHatchId" patternUnits="userSpaceOnUse" :width="bScaledHatchSpacing" :height="bScaledHatchSpacing" :patternTransform="`rotate(${bLineAngle})`">
        <line x1="0" y1="0" :x2="bScaledHatchSpacing" y2="0" :stroke="'#000'" :stroke-width="bScaledHatchStroke" />
      </pattern>

      <!-- B concentric pattern (transparent) -->
      <pattern :id="bConcentricId" patternUnits="userSpaceOnUse" :width="size * 2" :height="size * 2">
        <g :transform="`translate(${centerX + bScaledOffsetX}, ${centerY + bScaledOffsetY})`">
          <template v-for="i in bRings" :key="i">
            <circle :cx="0" :cy="0" :r="i * bScaledConcSpacing" fill="none" stroke="#000" :stroke-width="bScaledConcStroke" />
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
