<script setup>
import { ref, onMounted, onBeforeUnmount, watch } from 'vue'
import SvgCircle from './components/SvgCircle.vue'
import CombinedControls from './components/CombinedControls.vue'
import PreviewCircle from './components/PreviewCircle.vue'

// Circle A (left)
const aPattern = ref('hatch') // 'hatch' | 'concentric' | 'wavy' | 'perforated'
const aLineAngle = ref(70)
const aHatchStroke = ref(6)
const aHatchSpacing = ref(6)
const aConcStroke = ref(4)
const aConcSpacing = ref(9)
const aConcOffsetX = ref(0)
const aConcOffsetY = ref(0)
const aWavyAmplitude = ref(6)
const aWavyWavelength = ref(24)
const aWavySpacing = ref(12)
const aWavyStroke = ref(1)
const aPerforatedDotRadius = ref(4)
const aPerforatedSeparation = ref(12)
const aPerforatedInvert = ref(false)

// Circle B (right)
const bPattern = ref('concentric') // 'hatch' | 'concentric' | 'wavy' | 'perforated'
const bLineAngle = ref(0)
const bHatchStroke = ref(6)
const bHatchSpacing = ref(6)
const bConcStroke = ref(4)
const bConcSpacing = ref(9)
const bConcOffsetX = ref(25)
const bConcOffsetY = ref(30)
const bWavyAmplitude = ref(6)
const bWavyWavelength = ref(24)
const bWavySpacing = ref(12)
const bWavyStroke = ref(1)
const bPerforatedDotRadius = ref(4)
const bPerforatedSeparation = ref(12)
const bPerforatedInvert = ref(false)
const size = ref(300)
const aRef = ref(null)
const bRef = ref(null)
// Base reference radius for pattern scaling
const baseRefRadius = ref(200)
// Preview animation controls
const previewRotateB = ref(true)
const previewRotateSpeed = ref(20)
// Export options
const exportDpi = ref(300)

// Persist settings per circle/pattern
const SETTINGS_KEY = 'moireSettingsV1'

function setNum(refVar, val) { if (typeof val === 'number') refVar.value = val }
function setPattern(refVar, val) { if (val === 'hatch' || val === 'concentric' || val === 'wavy' || val === 'perforated') refVar.value = val }

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
    setNum(aWavyAmplitude, src.aWavyAmplitude)
    setNum(aWavyWavelength, src.aWavyWavelength)
    setNum(aWavySpacing, src.aWavySpacing)
    setNum(aWavyStroke, src.aWavyStroke)
  setNum(aPerforatedDotRadius, src.aPerforatedDotRadius)
  setNum(aPerforatedSeparation, src.aPerforatedSeparation)
  if (typeof src.aPerforatedInvert === 'boolean') aPerforatedInvert.value = src.aPerforatedInvert
  } else if (which === 'b') {
    setPattern(bPattern, src.pattern)
    setNum(bLineAngle, src.lineAngle)
    setNum(bHatchStroke, src.hatchStroke)
    setNum(bHatchSpacing, src.hatchSpacing)
    setNum(bConcStroke, src.concStroke)
    setNum(bConcSpacing, src.concSpacing)
    setNum(bConcOffsetX, src.concOffsetX)
    setNum(bConcOffsetY, src.concOffsetY)
    setNum(bWavyAmplitude, src.bWavyAmplitude)
    setNum(bWavyWavelength, src.bWavyWavelength)
    setNum(bWavySpacing, src.bWavySpacing)
    setNum(bWavyStroke, src.bWavyStroke)
  setNum(bPerforatedDotRadius, src.bPerforatedDotRadius)
  setNum(bPerforatedSeparation, src.bPerforatedSeparation)
  if (typeof src.bPerforatedInvert === 'boolean') bPerforatedInvert.value = src.bPerforatedInvert
  }
}

function loadSettings() {
  const raw = localStorage.getItem(SETTINGS_KEY)
  if (!raw) return
  try {
    const s = JSON.parse(raw)
    applyCircleSettings('a', s?.a)
    applyCircleSettings('b', s?.b)
    if (typeof s?.baseRefRadius === 'number' && s.baseRefRadius > 0) baseRefRadius.value = s.baseRefRadius
    if (typeof s?.exportDpi === 'number' && s.exportDpi > 0) exportDpi.value = Math.round(s.exportDpi)
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
      aWavyAmplitude: aWavyAmplitude.value,
      aWavyWavelength: aWavyWavelength.value,
      aWavySpacing: aWavySpacing.value,
      aWavyStroke: aWavyStroke.value,
  aPerforatedDotRadius: aPerforatedDotRadius.value,
  aPerforatedSeparation: aPerforatedSeparation.value,
  aPerforatedInvert: aPerforatedInvert.value,
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
      bWavyAmplitude: bWavyAmplitude.value,
      bWavyWavelength: bWavyWavelength.value,
      bWavySpacing: bWavySpacing.value,
      bWavyStroke: bWavyStroke.value,
  bPerforatedDotRadius: bPerforatedDotRadius.value,
  bPerforatedSeparation: bPerforatedSeparation.value,
  bPerforatedInvert: bPerforatedInvert.value,
    },
    baseRefRadius: baseRefRadius.value,
    exportDpi: exportDpi.value,
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

// Helper: remove stage background rects under the clipping group
function removeStageBackgroundRects(svgRoot) {
  const bgRects = svgRoot.querySelectorAll('g[clip-path] > rect')
  for (const rect of bgRects) rect.remove()
}

// Helper: remove white pattern tile backgrounds (direct children of pattern only)
function removeWhitePatternBackgrounds(svgRoot) {
  const selectors = [
    'defs pattern > rect[fill="white"]',
    'defs pattern > rect[fill="#fff"]',
    'defs pattern > rect[fill="#ffffff"]',
  ]
  const whiteRects = svgRoot.querySelectorAll(selectors.join(','))
  for (const r of whiteRects) r.remove()
}

function setSvgPixelSize(svgRoot, exportPx) {
  if (exportPx && Number.isFinite(exportPx)) {
    svgRoot.setAttribute('width', String(exportPx))
    svgRoot.setAttribute('height', String(exportPx))
  }
}

function serializeSvg(svgRoot) {
  const ser = new XMLSerializer()
  return ser.serializeToString(svgRoot)
}

// Helper: parse and sanitize SVG string, output black+transparent ready SVG string
function buildBlackTransparentSvgFromString(svgText, exportPx) {
  if (!svgText) return ''
  const parser = new DOMParser()
  const doc = parser.parseFromString(svgText, 'image/svg+xml')
  const svgRoot = doc.documentElement
  if (!svgRoot || svgRoot.nodeName.toLowerCase() !== 'svg') return ''
  svgRoot.setAttribute('xmlns', 'http://www.w3.org/2000/svg')
  removeStageBackgroundRects(svgRoot)
  removeWhitePatternBackgrounds(svgRoot)
  setSvgPixelSize(svgRoot, exportPx)
  return serializeSvg(svgRoot)
}

async function exportSvgRefToPng(refComp, fileBase) {
  const dpi = Math.max(1, Math.round(exportDpi.value || 300))
  const sizePx = Number(size.value) || 300
  const exportPx = Math.round((sizePx * dpi) / 96)
  // Prefer exposed serializer; fallback to component root element if available
  const originalSvg = refComp?.getSvgString?.() || refComp?.$el?.outerHTML || ''
  const svgString = buildBlackTransparentSvgFromString(originalSvg, exportPx)
  if (!svgString) { console.log('no svg string'); return }
  const blob = new Blob([svgString], { type: 'image/svg+xml;charset=utf-8' })
  const url = URL.createObjectURL(blob)
  try {
    const img = new Image()
    await new Promise((resolve, reject) => {
      const timer = setTimeout(() => {
        cleanup()
        reject(new Error('Image load timeout'))
      }, 10000)
      function cleanup() {
        clearTimeout(timer)
        img.onload = null
        img.onerror = null
      }
      img.onload = () => { cleanup(); resolve(null) }
      img.onerror = () => { cleanup(); reject(new Error('Image failed to load')) }
      img.src = url
    })

    const canvas = document.createElement('canvas')
    canvas.width = exportPx
    canvas.height = exportPx
    const ctx = canvas.getContext('2d', { willReadFrequently: true })
    if (!ctx) return
    ctx.imageSmoothingEnabled = true
    ctx.imageSmoothingQuality = 'high'
    ctx.clearRect(0, 0, canvas.width, canvas.height)
    ctx.drawImage(img, 0, 0, exportPx, exportPx)

    // Force RGB to black for all non-transparent pixels: guarantees only black + alpha
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height)
    const data = imageData.data
    for (let i = 0; i < data.length; i += 4) {
      const alpha = data[i + 3]
      if (alpha !== 0) {
        data[i] = 0 // R
        data[i + 1] = 0 // G
        data[i + 2] = 0 // B
      }
    }
    ctx.putImageData(imageData, 0, 0)

    await new Promise((resolve) => {
      canvas.toBlob(blob => {
        if (blob) {
          const pngUrl = URL.createObjectURL(blob)
          const a = document.createElement('a')
          a.href = pngUrl
          a.download = `${fileBase}-${dpi}dpi.png`
          document.body.appendChild(a)
          a.click()
          a.remove()
          URL.revokeObjectURL(pngUrl)
        }
        resolve()
      }, 'image/png')
    })
  } catch (e) {
    console.error('PNG export failed:', e)
    // Optional: lightweight user notification
    try { alert('PNG export failed. See console for details.') } catch {}
  } finally {
    URL.revokeObjectURL(url)
  }
}

function exportAPng() { exportSvgRefToPng(aRef.value, 'circle-a') }
function exportBPng() { exportSvgRefToPng(bRef.value, 'circle-b') }

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
  aWavyAmplitude, aWavyWavelength, aWavySpacing, aWavyStroke,
  aPerforatedDotRadius, aPerforatedSeparation, aPerforatedInvert,
  bPattern, bLineAngle, bHatchStroke, bHatchSpacing, bConcStroke, bConcSpacing, bConcOffsetX, bConcOffsetY,
  bWavyAmplitude, bWavyWavelength, bWavySpacing, bWavyStroke, bPerforatedDotRadius, bPerforatedSeparation, bPerforatedInvert,
  baseRefRadius, exportDpi,
], saveSettings, { deep: false })

function resetDefaults() {
  // Circle A defaults
  aPattern.value = 'hatch'
  aLineAngle.value = 70
  aHatchStroke.value = 6
  aHatchSpacing.value = 6
  aConcStroke.value = 4
  aConcSpacing.value = 9
  aConcOffsetX.value = 0
  aConcOffsetY.value = 0
  aWavyAmplitude.value = 6
  aWavyWavelength.value = 24
  aWavySpacing.value = 12
  aWavyStroke.value = 1
  aPerforatedDotRadius.value = 4
  aPerforatedSeparation.value = 12
  aPerforatedInvert.value = false

  // Circle B defaults
  bPattern.value = 'concentric'
  bLineAngle.value = 0
  bHatchStroke.value = 6
  bHatchSpacing.value = 6
  bConcStroke.value = 4
  bConcSpacing.value = 9
  bConcOffsetX.value = 25
  bConcOffsetY.value = 30
  bWavyAmplitude.value = 6
  bWavyWavelength.value = 24
  bWavySpacing.value = 12
  bWavyStroke.value = 1
  bPerforatedDotRadius.value = 4
  bPerforatedSeparation.value = 12
  bPerforatedInvert.value = false

  // Global defaults
  baseRefRadius.value = 200
  // Keep current size based on viewport; do not reset.
  // Animation defaults
  previewRotateB.value = true
  previewRotateSpeed.value = 20
  exportDpi.value = 300

  // Persist the defaults
  saveSettings()
}
</script>

<template>
  <div class="wrapper">
    <div class="card global-card">
      <div class="global-control">
        <label for="baseRefRadius">Base reference radius</label>
        <input id="baseRefRadius" class="num" type="number" min="1" step="1" v-model.number="baseRefRadius" />
          <label for="exportDpi">Export DPI</label>
          <input id="exportDpi" class="num" type="number" min="36" step="1" v-model.number="exportDpi" />
        <label for="rotateB">Rotate B</label>
        <input id="rotateB" type="checkbox" v-model="previewRotateB" />
        <label for="rotateSpeed">Speed</label>
        <input id="rotateSpeed" class="num" type="number" min="0" step="1" v-model.number="previewRotateSpeed" />
        <button type="button" @click="resetDefaults">Reset to defaults</button>
      </div>
    </div>
    <div class="card controls-card">
      <CombinedControls
      :a-pattern="aPattern" @update:aPattern="v => (aPattern = v)"
      :a-line-angle="aLineAngle" @update:aLineAngle="v => (aLineAngle = v)"
      :a-hatch-stroke="aHatchStroke" @update:aHatchStroke="v => (aHatchStroke = v)"
      :a-hatch-spacing="aHatchSpacing" @update:aHatchSpacing="v => (aHatchSpacing = v)"
      :a-conc-stroke="aConcStroke" @update:aConcStroke="v => (aConcStroke = v)"
      :a-conc-spacing="aConcSpacing" @update:aConcSpacing="v => (aConcSpacing = v)"
      :a-conc-offset-x="aConcOffsetX" @update:aConcOffsetX="v => (aConcOffsetX = v)"
      :a-conc-offset-y="aConcOffsetY" @update:aConcOffsetY="v => (aConcOffsetY = v)"
      :a-wavy-amplitude="aWavyAmplitude" @update:aWavyAmplitude="v => (aWavyAmplitude = v)"
      :a-wavy-wavelength="aWavyWavelength" @update:aWavyWavelength="v => (aWavyWavelength = v)"
      :a-wavy-spacing="aWavySpacing" @update:aWavySpacing="v => (aWavySpacing = v)"
      :a-wavy-stroke="aWavyStroke" @update:aWavyStroke="v => (aWavyStroke = v)"
  :a-perforated-dot-radius="aPerforatedDotRadius" @update:aPerforatedDotRadius="v => (aPerforatedDotRadius = v)"
  :a-perforated-separation="aPerforatedSeparation" @update:aPerforatedSeparation="v => (aPerforatedSeparation = v)"
  :a-perforated-invert="aPerforatedInvert" @update:aPerforatedInvert="v => (aPerforatedInvert = v)"

      :b-pattern="bPattern" @update:bPattern="v => (bPattern = v)"
      :b-line-angle="bLineAngle" @update:bLineAngle="v => (bLineAngle = v)"
      :b-hatch-stroke="bHatchStroke" @update:bHatchStroke="v => (bHatchStroke = v)"
      :b-hatch-spacing="bHatchSpacing" @update:bHatchSpacing="v => (bHatchSpacing = v)"
      :b-conc-stroke="bConcStroke" @update:bConcStroke="v => (bConcStroke = v)"
      :b-conc-spacing="bConcSpacing" @update:bConcSpacing="v => (bConcSpacing = v)"
      :b-conc-offset-x="bConcOffsetX" @update:bConcOffsetX="v => (bConcOffsetX = v)"
      :b-conc-offset-y="bConcOffsetY" @update:bConcOffsetY="v => (bConcOffsetY = v)"
      :b-wavy-amplitude="bWavyAmplitude" @update:bWavyAmplitude="v => (bWavyAmplitude = v)"
      :b-wavy-wavelength="bWavyWavelength" @update:bWavyWavelength="v => (bWavyWavelength = v)"
      :b-wavy-spacing="bWavySpacing" @update:bWavySpacing="v => (bWavySpacing = v)"
      :b-wavy-stroke="bWavyStroke" @update:bWavyStroke="v => (bWavyStroke = v)"
  :b-perforated-dot-radius="bPerforatedDotRadius" @update:bPerforatedDotRadius="v => (bPerforatedDotRadius = v)"
  :b-perforated-separation="bPerforatedSeparation" @update:bPerforatedSeparation="v => (bPerforatedSeparation = v)"
  :b-perforated-invert="bPerforatedInvert" @update:bPerforatedInvert="v => (bPerforatedInvert = v)"
      />
    </div>
      <div class="card stage">
        <div class="row">
          <SvgCircle ref="aRef" :size="size" :r="size/2 - 0.5 - 2"
            :use-hatch="aPattern === 'hatch'" :use-concentric="aPattern === 'concentric'" :use-wavy="aPattern === 'wavy'" :use-perforated="aPattern === 'perforated'"
            :line-angle="aLineAngle" :line-stroke-width="aHatchStroke" :line-spacing="aHatchSpacing"
            :concentric-stroke-width="aConcStroke" :concentric-spacing="aConcSpacing"
            :concentric-offset-x="aConcOffsetX" :concentric-offset-y="aConcOffsetY" :base-ref-radius="baseRefRadius"
            :wavy-amplitude="aWavyAmplitude" :wavy-wavelength="aWavyWavelength" :wavy-spacing="aWavySpacing" :wavy-stroke-width="aWavyStroke"
            :perforated-dot-radius="aPerforatedDotRadius" :perforated-separation="aPerforatedSeparation"
            :perforated-invert="aPerforatedInvert"
            aria-label="Circle A" />
          <SvgCircle ref="bRef" :size="size" :r="size/2 - 0.5 - 2"
            :use-hatch="bPattern === 'hatch'" :use-concentric="bPattern === 'concentric'" :use-wavy="bPattern === 'wavy'" :use-perforated="bPattern === 'perforated'"
            :line-angle="bLineAngle" :line-stroke-width="bHatchStroke" :line-spacing="bHatchSpacing"
            :concentric-stroke-width="bConcStroke" :concentric-spacing="bConcSpacing"
            :concentric-offset-x="bConcOffsetX" :concentric-offset-y="bConcOffsetY" :base-ref-radius="baseRefRadius"
            :wavy-amplitude="bWavyAmplitude" :wavy-wavelength="bWavyWavelength" :wavy-spacing="bWavySpacing" :wavy-stroke-width="bWavyStroke"
            :perforated-dot-radius="bPerforatedDotRadius" :perforated-separation="bPerforatedSeparation"
            :perforated-invert="bPerforatedInvert"
            aria-label="Circle B" />
        </div>
        <div class="preview">
          <PreviewCircle
            :size="size"
            :r="size/2 - 0.5 - 2"
            :base-ref-radius="baseRefRadius"
            :rotate-b-enabled="previewRotateB"
            :rotate-b-speed="previewRotateSpeed"
            :a-pattern="aPattern"
            :a-line-angle="aLineAngle"
            :a-hatch-stroke="aHatchStroke"
            :a-hatch-spacing="aHatchSpacing"
            :a-conc-stroke="aConcStroke"
            :a-conc-spacing="aConcSpacing"
            :a-conc-offset-x="aConcOffsetX"
            :a-conc-offset-y="aConcOffsetY"
            :a-wavy-amplitude="aWavyAmplitude"
            :a-wavy-wavelength="aWavyWavelength"
            :a-wavy-spacing="aWavySpacing"
            :a-wavy-stroke="aWavyStroke"
            :a-perforated-dot-radius="aPerforatedDotRadius"
            :a-perforated-separation="aPerforatedSeparation"
            :a-perforated-invert="aPerforatedInvert"

            :b-pattern="bPattern"
            :b-line-angle="bLineAngle"
            :b-hatch-stroke="bHatchStroke"
            :b-hatch-spacing="bHatchSpacing"
            :b-conc-stroke="bConcStroke"
            :b-conc-spacing="bConcSpacing"
            :b-conc-offset-x="bConcOffsetX"
            :b-conc-offset-y="bConcOffsetY"
            :b-wavy-amplitude="bWavyAmplitude"
            :b-wavy-wavelength="bWavyWavelength"
            :b-wavy-spacing="bWavySpacing"
            :b-wavy-stroke="bWavyStroke"
            :b-perforated-dot-radius="bPerforatedDotRadius"
            :b-perforated-separation="bPerforatedSeparation"
            :b-perforated-invert="bPerforatedInvert"
            aria-label="Preview Overlay"
          />
        </div>
      <div class="export-row">
        <button @click="exportACircle">Export SVG (A)</button>
        <button @click="exportBCircle">Export SVG (B)</button>
        <button @click="exportAPng">Export PNG (A)</button>
        <button @click="exportBPng">Export PNG (B)</button>
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
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 1rem;
  width: 100%;
  max-width: 95vw;
}

.preview {
  margin-top: 1rem;
  display: grid;
  place-items: center;
}

.global-control {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 0.5rem;
}
 .global-control .num { width: 6rem; }

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

/* Cards */
.card {
  background: rgba(255, 255, 255, 0.9);
  color: #222;
  padding: 0.75rem 1rem;
  border-radius: 12px;
  box-shadow: 0 6px 16px rgba(0,0,0,0.08);
  width: 100%;
  max-width: 1200px;
}
.global-card { display: block; }

.export-row {
  margin-top: 0.75rem;
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

/* Responsive adjustments */
@media (max-width: 900px) {
  .row {
    grid-template-columns: 1fr;
  }
}
@media (max-width: 600px) {
  .global-control { flex-wrap: wrap; }
  .global-control label { min-width: 120px; text-align: left; }
}
</style>
