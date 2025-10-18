<script setup>
import { ref, onMounted, onBeforeUnmount, watch } from 'vue'
import SvgCircle from './components/SvgCircle.vue'
import CombinedControls from './components/CombinedControls.vue'
import PreviewCircle from './components/PreviewCircle.vue'

// Circle A (left)
const aPattern = ref('hatch')
const aLineAngle = ref(0)
const aHatchStroke = ref(1)
const aHatchSpacing = ref(8)
const aConcStroke = ref(1)
const aConcSpacing = ref(12)
const aConcOffsetX = ref(0)
const aConcOffsetY = ref(0)

// Circle B (right)
const bPattern = ref('concentric')
const bLineAngle = ref(0)
const bHatchStroke = ref(1)
const bHatchSpacing = ref(8)
const bConcStroke = ref(1)
const bConcSpacing = ref(12)
const bConcOffsetX = ref(0)
const bConcOffsetY = ref(0)
const size = ref(300)
const aRef = ref(null)
const bRef = ref(null)

// Persist settings per circle/pattern
const SETTINGS_KEY = 'moireSettingsV1'

function setNum(refVar, val) { if (typeof val === 'number') refVar.value = val }
function setPattern(refVar, val) { if (val === 'hatch' || val === 'concentric') refVar.value = val }

function applyCircleSettings(which, src) {
  if (!src) return
  if (which === 'a') {
    setPattern(aPattern, src.pattern)
    setNum(aLineAngle, src.lineAngle)
    setNum(aHatchStroke, src.hatchStroke)
    setNum(aHatchSpacing, src.hatchSpacing)
    setNum(aConcStroke, src.concStroke)
    setNum(aConcSpacing, src.concSpacing)
    setNum(aConcOffsetX, src.concOffsetX)
    setNum(aConcOffsetY, src.concOffsetY)
  } else if (which === 'b') {
    setPattern(bPattern, src.pattern)
    setNum(bLineAngle, src.lineAngle)
    setNum(bHatchStroke, src.hatchStroke)
    setNum(bHatchSpacing, src.hatchSpacing)
    setNum(bConcStroke, src.concStroke)
    setNum(bConcSpacing, src.concSpacing)
    setNum(bConcOffsetX, src.concOffsetX)
    setNum(bConcOffsetY, src.concOffsetY)
  }
}

function loadSettings() {
  const raw = localStorage.getItem(SETTINGS_KEY)
  if (!raw) return
  try {
    const s = JSON.parse(raw)
    applyCircleSettings('a', s?.a)
    applyCircleSettings('b', s?.b)
  } catch (e) {
    console.warn('Failed to parse settings:', e)
  }
}

function saveSettings() {
  const s = {
    a: {
      pattern: aPattern.value,
      lineAngle: aLineAngle.value,
      hatchStroke: aHatchStroke.value,
      hatchSpacing: aHatchSpacing.value,
      concStroke: aConcStroke.value,
      concSpacing: aConcSpacing.value,
      concOffsetX: aConcOffsetX.value,
      concOffsetY: aConcOffsetY.value,
    },
    b: {
      pattern: bPattern.value,
      lineAngle: bLineAngle.value,
      hatchStroke: bHatchStroke.value,
      hatchSpacing: bHatchSpacing.value,
      concStroke: bConcStroke.value,
      concSpacing: bConcSpacing.value,
      concOffsetX: bConcOffsetX.value,
      concOffsetY: bConcOffsetY.value,
    },
  }
  try {
    localStorage.setItem(SETTINGS_KEY, JSON.stringify(s))
  } catch (e) {
    console.warn('Failed to save settings:', e)
  }
}

function downloadSvg(filename, svgString) {
  const blob = new Blob([svgString], { type: 'image/svg+xml;charset=utf-8' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = filename
  document.body.appendChild(a)
  a.click()
  a.remove()
  URL.revokeObjectURL(url)
}

function exportACircle() {
  const svg = aRef.value?.getSvgString?.()
  if (!svg) return
  downloadSvg('circle-a.svg', svg)
}
function exportBCircle() {
  const svg = bRef.value?.getSvgString?.()
  if (!svg) return
  downloadSvg('circle-b.svg', svg)
}

function updateSize() {
  // Use a fraction of the viewport so the circle doesn't fill the entire screen
  const fraction = 0.7
  const target = Math.min(window.innerWidth, window.innerHeight) * fraction
  // Clamp to keep it usable on very small or very large screens
  size.value = Math.max(200, Math.min(target, 800))
}

onMounted(() => {
  loadSettings()
  updateSize()
  window.addEventListener('resize', updateSize)
})
onBeforeUnmount(() => {
  window.removeEventListener('resize', updateSize)
})

// Save whenever any relevant control changes
watch([
  aPattern, aLineAngle, aHatchStroke, aHatchSpacing, aConcStroke, aConcSpacing, aConcOffsetX, aConcOffsetY,
  bPattern, bLineAngle, bHatchStroke, bHatchSpacing, bConcStroke, bConcSpacing, bConcOffsetX, bConcOffsetY,
], saveSettings, { deep: false })
</script>

<template>
  <div class="wrapper">
    <CombinedControls
      :a-pattern="aPattern" @update:aPattern="v => (aPattern = v)"
      :a-line-angle="aLineAngle" @update:aLineAngle="v => (aLineAngle = v)"
      :a-hatch-stroke="aHatchStroke" @update:aHatchStroke="v => (aHatchStroke = v)"
      :a-hatch-spacing="aHatchSpacing" @update:aHatchSpacing="v => (aHatchSpacing = v)"
      :a-conc-stroke="aConcStroke" @update:aConcStroke="v => (aConcStroke = v)"
      :a-conc-spacing="aConcSpacing" @update:aConcSpacing="v => (aConcSpacing = v)"
      :a-conc-offset-x="aConcOffsetX" @update:aConcOffsetX="v => (aConcOffsetX = v)"
      :a-conc-offset-y="aConcOffsetY" @update:aConcOffsetY="v => (aConcOffsetY = v)"

      :b-pattern="bPattern" @update:bPattern="v => (bPattern = v)"
      :b-line-angle="bLineAngle" @update:bLineAngle="v => (bLineAngle = v)"
      :b-hatch-stroke="bHatchStroke" @update:bHatchStroke="v => (bHatchStroke = v)"
      :b-hatch-spacing="bHatchSpacing" @update:bHatchSpacing="v => (bHatchSpacing = v)"
      :b-conc-stroke="bConcStroke" @update:bConcStroke="v => (bConcStroke = v)"
      :b-conc-spacing="bConcSpacing" @update:bConcSpacing="v => (bConcSpacing = v)"
      :b-conc-offset-x="bConcOffsetX" @update:bConcOffsetX="v => (bConcOffsetX = v)"
      :b-conc-offset-y="bConcOffsetY" @update:bConcOffsetY="v => (bConcOffsetY = v)"
    />
      <div class="stage">
        <div class="row">
          <SvgCircle ref="aRef" :size="size" :r="size/2 - 0.5 - 2"
            :use-hatch="aPattern === 'hatch'" :use-concentric="aPattern === 'concentric'"
            :line-angle="aLineAngle" :line-stroke-width="aHatchStroke" :line-spacing="aHatchSpacing"
            :concentric-stroke-width="aConcStroke" :concentric-spacing="aConcSpacing"
            :concentric-offset-x="aConcOffsetX" :concentric-offset-y="aConcOffsetY"
            aria-label="Circle A" />
          <SvgCircle ref="bRef" :size="size" :r="size/2 - 0.5 - 2"
            :use-hatch="bPattern === 'hatch'" :use-concentric="bPattern === 'concentric'"
            :line-angle="bLineAngle" :line-stroke-width="bHatchStroke" :line-spacing="bHatchSpacing"
            :concentric-stroke-width="bConcStroke" :concentric-spacing="bConcSpacing"
            :concentric-offset-x="bConcOffsetX" :concentric-offset-y="bConcOffsetY"
            aria-label="Circle B" />
        </div>
        <div class="preview">
          <PreviewCircle
            :size="size"
            :r="size/2 - 0.5 - 2"
            :a-pattern="aPattern"
            :a-line-angle="aLineAngle"
            :a-hatch-stroke="aHatchStroke"
            :a-hatch-spacing="aHatchSpacing"
            :a-conc-stroke="aConcStroke"
            :a-conc-spacing="aConcSpacing"
            :a-conc-offset-x="aConcOffsetX"
            :a-conc-offset-y="aConcOffsetY"

            :b-pattern="bPattern"
            :b-line-angle="bLineAngle"
            :b-hatch-stroke="bHatchStroke"
            :b-hatch-spacing="bHatchSpacing"
            :b-conc-stroke="bConcStroke"
            :b-conc-spacing="bConcSpacing"
            :b-conc-offset-x="bConcOffsetX"
            :b-conc-offset-y="bConcOffsetY"
            aria-label="Preview Overlay"
          />
        </div>
      <div class="export-row">
        <button @click="exportACircle">Export SVG (A)</button>
        <button @click="exportBCircle">Export SVG (B)</button>
      </div>
      </div>
  </div>
</template>

<style scoped>
/* Layout wrapper with controls separated from the canvas */
.wrapper {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  padding: 1rem;
  gap: 0.75rem;
}

/* Center the stage and overlay the control panel */
.stage {
  width: 100%;
  flex: 1;
  display: grid;
  place-items: center;
}

.row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
  width: 100%;
  max-width: 95vw;
}

.preview {
  margin-top: 1rem;
  display: grid;
  place-items: center;
}

.controls {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 0.75rem;
}
.controls label {
  font-size: 0.9rem;
}
.controls .num {
  width: 4.5rem;
}

.controls-block {
  background: rgba(255, 255, 255, 0.9);
  color: #222;
  padding: 0.5rem 0.75rem;
  border-radius: 8px;
  box-shadow: 0 4px 10px rgba(0,0,0,0.08);
}
</style>
