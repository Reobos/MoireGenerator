<script setup>
const props = defineProps({
  // Circle A
  aPattern: { type: String, default: 'hatch' }, // 'hatch' | 'concentric'
  aLineAngle: { type: Number, default: 0 },
  aHatchStroke: { type: Number, default: 1 },
  aHatchSpacing: { type: Number, default: 8 },
  aConcStroke: { type: Number, default: 1 },
  aConcSpacing: { type: Number, default: 12 },
  aConcOffsetX: { type: Number, default: 0 },
  aConcOffsetY: { type: Number, default: 0 },
  // Circle B
  bPattern: { type: String, default: 'concentric' },
  bLineAngle: { type: Number, default: 0 },
  bHatchStroke: { type: Number, default: 1 },
  bHatchSpacing: { type: Number, default: 8 },
  bConcStroke: { type: Number, default: 1 },
  bConcSpacing: { type: Number, default: 12 },
  bConcOffsetX: { type: Number, default: 0 },
  bConcOffsetY: { type: Number, default: 0 },
})

const emit = defineEmits([
  'update:aPattern','update:aLineAngle','update:aHatchStroke','update:aHatchSpacing',
  'update:aConcStroke','update:aConcSpacing','update:aConcOffsetX','update:aConcOffsetY',
  'update:bPattern','update:bLineAngle','update:bHatchStroke','update:bHatchSpacing',
  'update:bConcStroke','update:bConcSpacing','update:bConcOffsetX','update:bConcOffsetY',
])

const uid = Math.random().toString(36).slice(2, 8)
// ids for labels
const ids = {
  a: {
    pattern: `apat-${uid}`,
    angle: `aang-${uid}`,
    hstroke: `ahst-${uid}`,
    hspace: `ahsp-${uid}`,
    cstroke: `acst-${uid}`,
    cspace: `acsp-${uid}`,
    coffx: `acox-${uid}`,
    coffy: `acoy-${uid}`,
  },
  b: {
    pattern: `bpat-${uid}`,
    angle: `bang-${uid}`,
    hstroke: `bhst-${uid}`,
    hspace: `bhsp-${uid}`,
    cstroke: `bcst-${uid}`,
    cspace: `bcsp-${uid}`,
    coffx: `bcox-${uid}`,
    coffy: `bcoy-${uid}`,
  }
}
</script>

<template>
  <div class="combined">
    <div class="col">
      <h3>Circle A</h3>
      <div class="row">
        <label :for="ids.a.pattern">Pattern</label>
        <select :id="ids.a.pattern" :value="aPattern" @change="e => emit('update:aPattern', e.target.value)">
          <option value="hatch">Hatch</option>
          <option value="concentric">Concentric</option>
        </select>
      </div>

      <template v-if="aPattern === 'hatch'">
        <div class="row">
          <label :for="ids.a.angle">Angle</label>
          <input :id="ids.a.angle" type="range" min="0" max="180" step="1" :value="aLineAngle" @input="e => emit('update:aLineAngle', +(e.target.value))" />
          <input class="num" type="number" min="0" max="180" step="1" :value="aLineAngle" @input="e => emit('update:aLineAngle', +(e.target.value))" />
        </div>
        <div class="row">
          <label :for="ids.a.hstroke">Stroke</label>
          <input :id="ids.a.hstroke" type="range" min="1" max="10" step="1" :value="aHatchStroke" @input="e => emit('update:aHatchStroke', +(e.target.value))" />
          <input class="num" type="number" min="1" max="10" step="1" :value="aHatchStroke" @input="e => emit('update:aHatchStroke', +(e.target.value))" />
        </div>
        <div class="row">
          <label :for="ids.a.hspace">Spacing</label>
          <input :id="ids.a.hspace" type="range" min="2" max="40" step="1" :value="aHatchSpacing" @input="e => emit('update:aHatchSpacing', +(e.target.value))" />
          <input class="num" type="number" min="2" max="40" step="1" :value="aHatchSpacing" @input="e => emit('update:aHatchSpacing', +(e.target.value))" />
        </div>
      </template>

      <template v-else>
        <div class="row">
          <label :for="ids.a.cstroke">Ring stroke</label>
          <input :id="ids.a.cstroke" type="range" min="1" max="10" step="1" :value="aConcStroke" @input="e => emit('update:aConcStroke', +(e.target.value))" />
          <input class="num" type="number" min="1" max="10" step="1" :value="aConcStroke" @input="e => emit('update:aConcStroke', +(e.target.value))" />
        </div>
        <div class="row">
          <label :for="ids.a.cspace">Ring spacing</label>
          <input :id="ids.a.cspace" type="range" min="2" max="40" step="1" :value="aConcSpacing" @input="e => emit('update:aConcSpacing', +(e.target.value))" />
          <input class="num" type="number" min="2" max="40" step="1" :value="aConcSpacing" @input="e => emit('update:aConcSpacing', +(e.target.value))" />
        </div>
        <div class="row">
          <label :for="ids.a.coffx">Offset X</label>
          <input :id="ids.a.coffx" type="range" min="-200" max="200" step="1" :value="aConcOffsetX" @input="e => emit('update:aConcOffsetX', +(e.target.value))" />
          <input class="num" type="number" min="-1000" max="1000" step="1" :value="aConcOffsetX" @input="e => emit('update:aConcOffsetX', +(e.target.value))" />
        </div>
        <div class="row">
          <label :for="ids.a.coffy">Offset Y</label>
          <input :id="ids.a.coffy" type="range" min="-200" max="200" step="1" :value="aConcOffsetY" @input="e => emit('update:aConcOffsetY', +(e.target.value))" />
          <input class="num" type="number" min="-1000" max="1000" step="1" :value="aConcOffsetY" @input="e => emit('update:aConcOffsetY', +(e.target.value))" />
        </div>
      </template>
    </div>

    <div class="col">
      <h3>Circle B</h3>
      <div class="row">
        <label :for="ids.b.pattern">Pattern</label>
        <select :id="ids.b.pattern" :value="bPattern" @change="e => emit('update:bPattern', e.target.value)">
          <option value="hatch">Hatch</option>
          <option value="concentric">Concentric</option>
        </select>
      </div>

      <template v-if="bPattern === 'hatch'">
        <div class="row">
          <label :for="ids.b.angle">Angle</label>
          <input :id="ids.b.angle" type="range" min="0" max="180" step="1" :value="bLineAngle" @input="e => emit('update:bLineAngle', +(e.target.value))" />
          <input class="num" type="number" min="0" max="180" step="1" :value="bLineAngle" @input="e => emit('update:bLineAngle', +(e.target.value))" />
        </div>
        <div class="row">
          <label :for="ids.b.hstroke">Stroke</label>
          <input :id="ids.b.hstroke" type="range" min="1" max="10" step="1" :value="bHatchStroke" @input="e => emit('update:bHatchStroke', +(e.target.value))" />
          <input class="num" type="number" min="1" max="10" step="1" :value="bHatchStroke" @input="e => emit('update:bHatchStroke', +(e.target.value))" />
        </div>
        <div class="row">
          <label :for="ids.b.hspace">Spacing</label>
          <input :id="ids.b.hspace" type="range" min="2" max="40" step="1" :value="bHatchSpacing" @input="e => emit('update:bHatchSpacing', +(e.target.value))" />
          <input class="num" type="number" min="2" max="40" step="1" :value="bHatchSpacing" @input="e => emit('update:bHatchSpacing', +(e.target.value))" />
        </div>
      </template>

      <template v-else>
        <div class="row">
          <label :for="ids.b.cstroke">Ring stroke</label>
          <input :id="ids.b.cstroke" type="range" min="1" max="10" step="1" :value="bConcStroke" @input="e => emit('update:bConcStroke', +(e.target.value))" />
          <input class="num" type="number" min="1" max="10" step="1" :value="bConcStroke" @input="e => emit('update:bConcStroke', +(e.target.value))" />
        </div>
        <div class="row">
          <label :for="ids.b.cspace">Ring spacing</label>
          <input :id="ids.b.cspace" type="range" min="2" max="40" step="1" :value="bConcSpacing" @input="e => emit('update:bConcSpacing', +(e.target.value))" />
          <input class="num" type="number" min="2" max="40" step="1" :value="bConcSpacing" @input="e => emit('update:bConcSpacing', +(e.target.value))" />
        </div>
        <div class="row">
          <label :for="ids.b.coffx">Offset X</label>
          <input :id="ids.b.coffx" type="range" min="-200" max="200" step="1" :value="bConcOffsetX" @input="e => emit('update:bConcOffsetX', +(e.target.value))" />
          <input class="num" type="number" min="-1000" max="1000" step="1" :value="bConcOffsetX" @input="e => emit('update:bConcOffsetX', +(e.target.value))" />
        </div>
        <div class="row">
          <label :for="ids.b.coffy">Offset Y</label>
          <input :id="ids.b.coffy" type="range" min="-200" max="200" step="1" :value="bConcOffsetY" @input="e => emit('update:bConcOffsetY', +(e.target.value))" />
          <input class="num" type="number" min="-1000" max="1000" step="1" :value="bConcOffsetY" @input="e => emit('update:bConcOffsetY', +(e.target.value))" />
        </div>
      </template>
    </div>
  </div>
</template>

<style scoped>
.combined {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
}
.col {
  background: rgba(255, 255, 255, 0.9);
  color: #222;
  padding: 0.5rem 0.75rem;
  border-radius: 8px;
  box-shadow: 0 4px 10px rgba(0,0,0,0.08);
}
.row {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 0.5rem;
}
.num { width: 4.5rem; }
select { padding: 0.2rem; }
</style>
