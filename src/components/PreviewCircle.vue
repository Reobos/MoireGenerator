<script setup>
import { computed, ref, onMounted, onBeforeUnmount, watch } from 'vue'

const props = defineProps({
  size: { type: Number, default: 200 },
  r: { type: Number, default: 60 },
  cx: { type: Number, default: null },
  cy: { type: Number, default: null },
  stroke: { type: String, default: '#000' },
  background: { type: String, default: 'white' },
  ariaLabel: { type: String, default: 'Preview circle' },
  baseRefRadius: { type: Number, default: 150 },
  // animation to rotate circle B pattern
  rotateBEnabled: { type: Boolean, default: true },
  rotateBSpeed: { type: Number, default: 20 }, // degrees per second

  // Circle A pattern selection + settings
  aPattern: { type: String, default: 'hatch' }, // 'hatch' | 'concentric'
  aLineAngle: { type: Number, default: 0 },
  aHatchStroke: { type: Number, default: 1 },
  aHatchSpacing: { type: Number, default: 8 },
  aConcStroke: { type: Number, default: 1 },
  aConcSpacing: { type: Number, default: 12 },
  aConcOffsetX: { type: Number, default: 0 },
  aConcOffsetY: { type: Number, default: 0 },
  aWavyAmplitude: { type: Number, default: 6 },
  aWavyWavelength: { type: Number, default: 24 },
  aWavySpacing: { type: Number, default: 12 },
  aWavyStroke: { type: Number, default: 1 },
  aPerforatedDotRadius: { type: Number, default: 4 },
  aPerforatedSeparation: { type: Number, default: 12 },
  aPerforatedInvert: { type: Boolean, default: false },

  // Circle B pattern selection + settings
  bPattern: { type: String, default: 'concentric' },
  bLineAngle: { type: Number, default: 0 },
  bHatchStroke: { type: Number, default: 1 },
  bHatchSpacing: { type: Number, default: 8 },
  bConcStroke: { type: Number, default: 1 },
  bConcSpacing: { type: Number, default: 12 },
  bConcOffsetX: { type: Number, default: 0 },
  bConcOffsetY: { type: Number, default: 0 },
  bWavyAmplitude: { type: Number, default: 6 },
  bWavyWavelength: { type: Number, default: 24 },
  bWavySpacing: { type: Number, default: 12 },
  bWavyStroke: { type: Number, default: 1 },
  bPerforatedDotRadius: { type: Number, default: 4 },
  bPerforatedSeparation: { type: Number, default: 12 },
  bPerforatedInvert: { type: Boolean, default: false },
})

const uid = Math.random().toString(36).slice(2, 8)
const titleId = `previewTitle-${uid}`
const clipId = `previewClip-${uid}`

// A pattern ids
const aHatchId = `aHatch-${uid}`
const aConcentricId = `aConcentric-${uid}`
const aWavyId = `aWavy-${uid}`
const aPerforatedId = `aPerforated-${uid}`
// B pattern ids
const bHatchId = `bHatch-${uid}`
const bConcentricId = `bConcentric-${uid}`
const bWavyId = `bWavy-${uid}`
const bPerforatedId = `bPerforated-${uid}`
const aPerforatedMaskId = `aPerforatedMask-${uid}`
const bPerforatedMaskId = `bPerforatedMask-${uid}`

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
// Scaled wavy A/B
const aScaledWavyAmplitude = computed(() => props.aWavyAmplitude * scale.value)
const aScaledWavyWavelength = computed(() => Math.max(1, props.aWavyWavelength * scale.value))
const aScaledWavySpacing = computed(() => Math.max(1, props.aWavySpacing * scale.value))
const aScaledWavyStroke = computed(() => props.aWavyStroke * scale.value)
// A perforated scaled (interpret separation as hex edge length 'a')
const aScaledDotR = computed(() => Math.max(0.5, props.aPerforatedDotRadius * scale.value))
const aScaledHexA = computed(() => Math.max(1, props.aPerforatedSeparation * scale.value))
const aColStep = computed(() => Math.sqrt(3) * aScaledHexA.value)
const aRowStep = computed(() => 1.5 * aScaledHexA.value)

const bScaledWavyAmplitude = computed(() => props.bWavyAmplitude * scale.value)
const bScaledWavyWavelength = computed(() => Math.max(1, props.bWavyWavelength * scale.value))
const bScaledWavySpacing = computed(() => Math.max(1, props.bWavySpacing * scale.value))
const bScaledWavyStroke = computed(() => props.bWavyStroke * scale.value)
// B perforated scaled (interpret separation as hex edge length 'a')
const bScaledDotR = computed(() => Math.max(0.5, props.bPerforatedDotRadius * scale.value))
const bScaledHexA = computed(() => Math.max(1, props.bPerforatedSeparation * scale.value))
const bColStep = computed(() => Math.sqrt(3) * bScaledHexA.value)
const bRowStep = computed(() => 1.5 * bScaledHexA.value)

function makeWavePath(width, amplitude, samples, baselineY) {
  const n = Math.max(32, Math.min(1024, samples || Math.round(width * 2)))
  const w = Math.max(1, width)
  const A = amplitude
  let d = ''
  for (let i = 0; i < n; i++) {
    const x = (i / (n - 1)) * w
    const y = baselineY + (A ? Math.sin((2 * Math.PI * x) / w) * A : 0)
    d += (i === 0 ? `M ${x} ${y}` : ` L ${x} ${y}`)
  }
  return d
}
const bOffset = computed(() => Math.hypot(bScaledOffsetX.value, bScaledOffsetY.value))
const bRings = computed(() => {
  if (bScaledConcSpacing.value <= 0) return 0
  const requiredR = bOffset.value + clampedR.value
  return Math.max(0, Math.floor(requiredR / bScaledConcSpacing.value))
})

// Animated rotation state for B
const bAngle = ref(0)
let rafId = 0
let lastTs = 0
function loop(ts) {
  if (!lastTs) lastTs = ts
  const dt = (ts - lastTs) / 1000
  lastTs = ts
  bAngle.value = (bAngle.value + dt * (props.rotateBSpeed || 0)) % 360
  rafId = requestAnimationFrame(loop)
}
function startAnim() {
  if (rafId) cancelAnimationFrame(rafId)
  lastTs = 0
  rafId = requestAnimationFrame(loop)
}
function stopAnim() {
  if (rafId) cancelAnimationFrame(rafId)
  rafId = 0
}
onMounted(() => {
  if (props.rotateBEnabled) startAnim()
})
onBeforeUnmount(() => {
  stopAnim()
})
watch(() => props.rotateBEnabled, (en) => {
  if (en) startAnim(); else stopAnim()
})

// B pattern rotation transforms around the circle center
const bHatchAngle = computed(() => (props.bLineAngle || 0) + bAngle.value)
const bHatchTransform = computed(() => `rotate(${bHatchAngle.value}, ${centerX.value}, ${centerY.value})`)
const bConcentricTransform = computed(() => `rotate(${bAngle.value}, ${centerX.value}, ${centerY.value})`)
const bWavyTransform = computed(() => `rotate(${bAngle.value}, ${centerX.value}, ${centerY.value})`)
const bPerforatedTransform = computed(() => `rotate(${bAngle.value}, ${centerX.value}, ${centerY.value})`)
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

      <!-- A wavy pattern (transparent) -->
      <pattern :id="aWavyId" patternUnits="userSpaceOnUse" :width="aScaledWavyWavelength" :height="Math.max(aScaledWavySpacing, 2 * aScaledWavyAmplitude + aScaledWavyStroke)">
        <path :d="makeWavePath(aScaledWavyWavelength, aScaledWavyAmplitude, 60, Math.max(aScaledWavySpacing, 2 * aScaledWavyAmplitude + aScaledWavyStroke) / 2)" fill="none" stroke="#000" :stroke-width="aScaledWavyStroke" stroke-linecap="round" stroke-linejoin="round" />
      </pattern>

      <!-- A perforated honeycomb dots (transparent background) -->
      <pattern :id="aPerforatedId" patternUnits="userSpaceOnUse" :width="aColStep" :height="aRowStep">
        <template v-if="!aPerforatedInvert">
          <!-- transparent bg with black dots -->
          <!-- base dots -->
          <circle :cx="aScaledDotR" :cy="aScaledDotR" :r="aScaledDotR" fill="#000" />
          <circle :cx="(aColStep/2) + aScaledDotR" :cy="(aRowStep/2) + aScaledDotR" :r="aScaledDotR" fill="#000" />
          <!-- seam-safe duplicates from neighboring tiles (shifted negative) -->
          <circle :cx="aScaledDotR - aColStep" :cy="aScaledDotR" :r="aScaledDotR" fill="#000" />
          <circle :cx="aScaledDotR" :cy="aScaledDotR - aRowStep" :r="aScaledDotR" fill="#000" />
          <circle :cx="aScaledDotR - aColStep" :cy="aScaledDotR - aRowStep" :r="aScaledDotR" fill="#000" />
          <circle :cx="(aColStep/2) + aScaledDotR - aColStep" :cy="(aRowStep/2) + aScaledDotR" :r="aScaledDotR" fill="#000" />
          <circle :cx="(aColStep/2) + aScaledDotR" :cy="(aRowStep/2) + aScaledDotR - aRowStep" :r="aScaledDotR" fill="#000" />
          <circle :cx="(aColStep/2) + aScaledDotR - aColStep" :cy="(aRowStep/2) + aScaledDotR - aRowStep" :r="aScaledDotR" fill="#000" />
        </template>
        <template v-else>
          <!-- transparent bg with transparent holes cut from a black rect via mask -->
          <mask :id="aPerforatedMaskId">
            <rect :width="aColStep" :height="aRowStep" fill="white" />
            <!-- base holes -->
            <circle :cx="aScaledDotR" :cy="aScaledDotR" :r="aScaledDotR" fill="black" />
            <circle :cx="(aColStep/2) + aScaledDotR" :cy="(aRowStep/2) + aScaledDotR" :r="aScaledDotR" fill="black" />
            <!-- seam-safe duplicates from neighboring tiles (shifted negative) -->
            <circle :cx="aScaledDotR - aColStep" :cy="aScaledDotR" :r="aScaledDotR" fill="black" />
            <circle :cx="aScaledDotR" :cy="aScaledDotR - aRowStep" :r="aScaledDotR" fill="black" />
            <circle :cx="aScaledDotR - aColStep" :cy="aScaledDotR - aRowStep" :r="aScaledDotR" fill="black" />
            <circle :cx="(aColStep/2) + aScaledDotR - aColStep" :cy="(aRowStep/2) + aScaledDotR" :r="aScaledDotR" fill="black" />
            <circle :cx="(aColStep/2) + aScaledDotR" :cy="(aRowStep/2) + aScaledDotR - aRowStep" :r="aScaledDotR" fill="black" />
            <circle :cx="(aColStep/2) + aScaledDotR - aColStep" :cy="(aRowStep/2) + aScaledDotR - aRowStep" :r="aScaledDotR" fill="black" />
          </mask>
          <rect :width="aColStep" :height="aRowStep" fill="#000" :mask="`url(#${aPerforatedMaskId})`" />
        </template>
      </pattern>

      <!-- B hatch pattern (transparent) -->
      <pattern :id="bHatchId" patternUnits="userSpaceOnUse" :width="bScaledHatchSpacing" :height="bScaledHatchSpacing" :patternTransform="bHatchTransform">
        <line x1="0" y1="0" :x2="bScaledHatchSpacing" y2="0" :stroke="'#000'" :stroke-width="bScaledHatchStroke" />
      </pattern>

      <!-- B concentric pattern (transparent) -->
      <pattern :id="bConcentricId" patternUnits="userSpaceOnUse" :width="size * 2" :height="size * 2" :patternTransform="bConcentricTransform">
        <g :transform="`translate(${centerX + bScaledOffsetX}, ${centerY + bScaledOffsetY})`">
          <template v-for="i in bRings" :key="i">
            <circle :cx="0" :cy="0" :r="i * bScaledConcSpacing" fill="none" stroke="#000" :stroke-width="bScaledConcStroke" />
          </template>
        </g>
      </pattern>

      <!-- B wavy pattern (transparent) -->
      <pattern :id="bWavyId" patternUnits="userSpaceOnUse" :width="bScaledWavyWavelength" :height="Math.max(bScaledWavySpacing, 2 * bScaledWavyAmplitude + bScaledWavyStroke)" :patternTransform="bWavyTransform">
        <path :d="makeWavePath(bScaledWavyWavelength, bScaledWavyAmplitude, 60, Math.max(bScaledWavySpacing, 2 * bScaledWavyAmplitude + bScaledWavyStroke) / 2)" fill="none" stroke="#000" :stroke-width="bScaledWavyStroke" stroke-linecap="round" stroke-linejoin="round" />
      </pattern>

      <!-- B perforated (transparent) -->
      <pattern :id="bPerforatedId" patternUnits="userSpaceOnUse" :width="bColStep" :height="bRowStep" :patternTransform="bPerforatedTransform">
        <template v-if="!bPerforatedInvert">
          <!-- base dots -->
          <circle :cx="bScaledDotR" :cy="bScaledDotR" :r="bScaledDotR" fill="#000" />
          <circle :cx="(bColStep/2) + bScaledDotR" :cy="(bRowStep/2) + bScaledDotR" :r="bScaledDotR" fill="#000" />
          <!-- seam-safe duplicates from neighboring tiles (shifted negative) -->
          <circle :cx="bScaledDotR - bColStep" :cy="bScaledDotR" :r="bScaledDotR" fill="#000" />
          <circle :cx="bScaledDotR" :cy="bScaledDotR - bRowStep" :r="bScaledDotR" fill="#000" />
          <circle :cx="bScaledDotR - bColStep" :cy="bScaledDotR - bRowStep" :r="bScaledDotR" fill="#000" />
          <circle :cx="(bColStep/2) + bScaledDotR - bColStep" :cy="(bRowStep/2) + bScaledDotR" :r="bScaledDotR" fill="#000" />
          <circle :cx="(bColStep/2) + bScaledDotR" :cy="(bRowStep/2) + bScaledDotR - bRowStep" :r="bScaledDotR" fill="#000" />
          <circle :cx="(bColStep/2) + bScaledDotR - bColStep" :cy="(bRowStep/2) + bScaledDotR - bRowStep" :r="bScaledDotR" fill="#000" />
        </template>
        <template v-else>
          <mask :id="bPerforatedMaskId">
            <rect :width="bColStep" :height="bRowStep" fill="white" />
            <!-- base holes -->
            <circle :cx="bScaledDotR" :cy="bScaledDotR" :r="bScaledDotR" fill="black" />
            <circle :cx="(bColStep/2) + bScaledDotR" :cy="(bRowStep/2) + bScaledDotR" :r="bScaledDotR" fill="black" />
            <!-- seam-safe duplicates from neighboring tiles (shifted negative) -->
            <circle :cx="bScaledDotR - bColStep" :cy="bScaledDotR" :r="bScaledDotR" fill="black" />
            <circle :cx="bScaledDotR" :cy="bScaledDotR - bRowStep" :r="bScaledDotR" fill="black" />
            <circle :cx="bScaledDotR - bColStep" :cy="bScaledDotR - bRowStep" :r="bScaledDotR" fill="black" />
            <circle :cx="(bColStep/2) + bScaledDotR - bColStep" :cy="(bRowStep/2) + bScaledDotR" :r="bScaledDotR" fill="black" />
            <circle :cx="(bColStep/2) + bScaledDotR" :cy="(bRowStep/2) + bScaledDotR - bRowStep" :r="bScaledDotR" fill="black" />
            <circle :cx="(bColStep/2) + bScaledDotR - bColStep" :cy="(bRowStep/2) + bScaledDotR - bRowStep" :r="bScaledDotR" fill="black" />
          </mask>
          <rect :width="bColStep" :height="bRowStep" fill="#000" :mask="`url(#${bPerforatedMaskId})`" />
        </template>
      </pattern>
    </defs>

    <g :clip-path="`url(#${clipId})`">
      <rect :width="size" :height="size" :fill="background" />
      <!-- Layer A fill if active -->
  <circle v-if="aPattern === 'hatch'" :cx="centerX" :cy="centerY" :r="clampedR" :fill="`url(#${aHatchId})`" />
  <circle v-else-if="aPattern === 'concentric'" :cx="centerX" :cy="centerY" :r="clampedR" :fill="`url(#${aConcentricId})`" />
  <circle v-else-if="aPattern === 'wavy'" :cx="centerX" :cy="centerY" :r="clampedR" :fill="`url(#${aWavyId})`" />
  <circle v-else-if="aPattern === 'perforated'" :cx="centerX" :cy="centerY" :r="clampedR" :fill="`url(#${aPerforatedId})`" />

      <!-- Layer B fill if active -->
  <circle v-if="bPattern === 'hatch'" :cx="centerX" :cy="centerY" :r="clampedR" :fill="`url(#${bHatchId})`" />
  <circle v-else-if="bPattern === 'concentric'" :cx="centerX" :cy="centerY" :r="clampedR" :fill="`url(#${bConcentricId})`" />
  <circle v-else-if="bPattern === 'wavy'" :cx="centerX" :cy="centerY" :r="clampedR" :fill="`url(#${bWavyId})`" />
  <circle v-else-if="bPattern === 'perforated'" :cx="centerX" :cy="centerY" :r="clampedR" :fill="`url(#${bPerforatedId})`" />

      <!-- Outline on top -->
      <circle :cx="centerX" :cy="centerY" :r="clampedR" :stroke="stroke" stroke-width="1" fill="none" />
    </g>
  </svg>

</template>

<style scoped>
svg { display: block; max-width: 100%; height: auto; }
</style>
