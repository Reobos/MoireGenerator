<script setup>
import { computed, ref } from 'vue'

const props = defineProps({
  // overall square canvas size
  size: { type: Number, default: 200 },
  // circle radius
  r: { type: Number, default: 60 },
  // circle center (defaults to canvas center)
  cx: { type: Number, default: null },
  cy: { type: Number, default: null },
  // styling
  stroke: { type: String, default: '#000' },
  fill: { type: String, default: 'white' },
  // optional background of the SVG
  background: { type: String, default: 'white' },
  // hatch pattern options
  useHatch: { type: Boolean, default: true },
  lineSpacing: { type: Number, default: 8 },
  lineColor: { type: String, default: '#000' },
  lineStrokeWidth: { type: Number, default: 1 },
  lineAngle: { type: Number, default: 0 }, // degrees, 0 = horizontal lines
  // concentric circles options
  useConcentric: { type: Boolean, default: false },
  concentricSpacing: { type: Number, default: 12 },
  concentricColor: { type: String, default: '#000' },
  concentricStrokeWidth: { type: Number, default: 1 },
  concentricOffsetX: { type: Number, default: 0 },
  concentricOffsetY: { type: Number, default: 0 },
  // base reference radius for scaling pattern sizes
  baseRefRadius: { type: Number, default: 150 },
  // ARIA label for accessibility
  ariaLabel: { type: String, default: 'Generated circle' }
})

// Unique ids per instance for accessibility and defs
const uid = Math.random().toString(36).slice(2, 8)
const titleId = `svgCircleTitle-${uid}`
const frameClipId = `svgCircleFrame-${uid}`
const hatchPatternId = `circleHatch-${uid}`
const concentricPatternId = `circleConcentric-${uid}`

const viewBox = computed(() => `0 0 ${props.size} ${props.size}`)
const centerX = computed(() => (props.cx == null ? props.size / 2 : props.cx))
const centerY = computed(() => (props.cy == null ? props.size / 2 : props.cy))

// Ensure the circle fits within the SVG even with stroke
const clampedR = computed(() => {
  // Outline stroke width is fixed at 1
  const outline = 1
  const maxR = props.size / 2 - outline / 2
  return Math.max(0, Math.min(props.r, maxR))
})

const hatchUrl = computed(() => `url(#${hatchPatternId})`)
const concentricUrl = computed(() => `url(#${concentricPatternId})`)

// Keep pattern scale relative to circle radius using a base reference radius (configurable)
const scale = computed(() => {
  const base = props.baseRefRadius > 0 ? props.baseRefRadius : 150
  return clampedR.value / base
})

// Scaled hatch values
const scaledLineSpacing = computed(() => props.lineSpacing * scale.value)
const scaledLineStrokeWidth = computed(() => props.lineStrokeWidth * scale.value)

// Scaled concentric values
const scaledConcentricSpacing = computed(() => props.concentricSpacing * scale.value)
const scaledConcentricStrokeWidth = computed(() => props.concentricStrokeWidth * scale.value)
const scaledConcentricOffsetX = computed(() => props.concentricOffsetX * scale.value)
const scaledConcentricOffsetY = computed(() => props.concentricOffsetY * scale.value)

const offsetDistance = computed(() => Math.hypot(scaledConcentricOffsetX.value, scaledConcentricOffsetY.value))
const ringsCount = computed(() => {
  if (scaledConcentricSpacing.value <= 0) return 0
  // cover entire circle even when the ring center is offset: need up to (offset + radius)
  const requiredR = offsetDistance.value + clampedR.value
  return Math.max(0, Math.floor(requiredR / scaledConcentricSpacing.value))
})

// Expose SVG for export
const svgEl = ref(null)
function getSvgString() {
  if (!svgEl.value) return ''
  const clone = svgEl.value.cloneNode(true)
  // Ensure xmlns is present on the root
  clone.setAttribute('xmlns', 'http://www.w3.org/2000/svg')
  // Serialize
  const serializer = new XMLSerializer()
  return serializer.serializeToString(clone)
}

defineExpose({ getSvgString, svgEl })
</script>

<template>
  <svg
    ref="svgEl"
    :width="size"
    :height="size"
    :viewBox="viewBox"
    :aria-labelledby="titleId"
    xmlns="http://www.w3.org/2000/svg"
  >
    <title :id="titleId">{{ ariaLabel }}</title>
    <defs>
      <clipPath :id="frameClipId">
        <rect :width="size" :height="size" />
      </clipPath>
      <pattern
        :id="hatchPatternId"
        patternUnits="userSpaceOnUse"
        :width="scaledLineSpacing"
        :height="scaledLineSpacing"
        :patternTransform="`rotate(${lineAngle})`"
      >
        <!-- white background for the hatch pattern -->
        <rect :width="scaledLineSpacing" :height="scaledLineSpacing" fill="white" />
  <!-- a single line; pattern repeats to make parallel lines -->
  <line x1="0" y1="0" :x2="scaledLineSpacing" y2="0" :stroke="lineColor" :stroke-width="scaledLineStrokeWidth" />
      </pattern>

      <pattern
        :id="concentricPatternId"
        patternUnits="userSpaceOnUse"
        :width="size * 2"
        :height="size * 2"
      >
        <rect :width="size * 2" :height="size * 2" fill="white" />
        <g :transform="`translate(${centerX + scaledConcentricOffsetX}, ${centerY + scaledConcentricOffsetY})`">
          <template v-for="i in ringsCount" :key="i">
            <circle :cx="0" :cy="0" :r="i * scaledConcentricSpacing" :fill="'none'" :stroke="concentricColor" :stroke-width="scaledConcentricStrokeWidth" />
          </template>
        </g>
      </pattern>
    </defs>

    <g :clip-path="`url(#${frameClipId})`">
      <rect :width="size" :height="size" :fill="background" />
      <circle
        :cx="centerX"
        :cy="centerY"
        :r="clampedR"
        :stroke="stroke"
        stroke-width="1"
        :fill="useConcentric ? concentricUrl : (useHatch ? hatchUrl : fill)"
      />
    </g>
  </svg>
</template>

<style scoped>
svg {
  display: block;
  max-width: 100%;
  height: auto;
}
</style>
