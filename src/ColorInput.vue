<template>
  <div class="color-input" :class="[size, { disabled }]">
    <div class="color-preview" :style="{ backgroundColor: modelValue }"
         @click="togglePicker" role="button" tabindex="0"
         :aria-label="'Color picker, current: ' + modelValue"
         @keydown.enter="togglePicker" @keydown.space.prevent="togglePicker">
    </div>
    <input type="text" :value="displayValue" @input="onInput"
           :disabled="disabled" :aria-label="'Color value'" class="color-text" />
    <div v-if="isOpen" class="picker-dropdown" role="dialog" aria-label="Color picker">
      <canvas ref="canvas" width="256" height="256" class="color-canvas"
              @mousedown="startDrag" @mousemove="onDrag" @mouseup="stopDrag" />
      <div class="hue-slider">
        <input type="range" min="0" max="360" :value="hue" @input="onHueChange"
               aria-label="Hue" class="hue-range" />
      </div>
      <div v-if="presets.length" class="preset-colors" role="listbox" aria-label="Preset colors">
        <button v-for="c in presets" :key="c" class="preset-swatch"
                :style="{ backgroundColor: c }" @click="selectPreset(c)"
                :aria-label="'Select color ' + c" :aria-selected="c === modelValue" />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted, onUnmounted } from 'vue'

interface Props {
  modelValue?: string
  format?: 'hex' | 'rgb' | 'hsl'
  presets?: string[]
  disabled?: boolean
  size?: 'sm' | 'md' | 'lg'
}

const props = withDefaults(defineProps<Props>(), {
  modelValue: '#000000',
  format: 'hex',
  presets: () => [],
  disabled: false,
  size: 'md',
})

const emit = defineEmits<{ 'update:modelValue': [value: string] }>()

const isOpen = ref(false)
const isDragging = ref(false)
const canvas = ref<HTMLCanvasElement>()
const hue = ref(0)

const displayValue = computed(() => {
  if (props.format === 'hex') return props.modelValue
  if (props.format === 'rgb') return hexToRgb(props.modelValue)
  return hexToHsl(props.modelValue)
})

function togglePicker() {
  if (!props.disabled) isOpen.value = !isOpen.value
}

function onInput(e: Event) {
  const val = (e.target as HTMLInputElement).value
  if (/^#[0-9a-fA-F]{6}$/.test(val)) emit('update:modelValue', val)
}

function selectPreset(color: string) {
  emit('update:modelValue', color)
  isOpen.value = false
}

function onHueChange(e: Event) {
  hue.value = parseInt((e.target as HTMLInputElement).value)
  drawCanvas()
}

function startDrag() { isDragging.value = true }
function stopDrag() { isDragging.value = false }
function onDrag(e: MouseEvent) {
  if (!isDragging.value || !canvas.value) return
  const rect = canvas.value.getBoundingClientRect()
  const x = Math.max(0, Math.min(255, e.clientX - rect.left))
  const y = Math.max(0, Math.min(255, e.clientY - rect.top))
  const s = x / 255
  const v = 1 - y / 255
  emit('update:modelValue', hsvToHex(hue.value, s, v))
}

function drawCanvas() {
  if (!canvas.value) return
  const ctx = canvas.value.getContext('2d')!
  const w = 256, h = 256
  for (let x = 0; x < w; x++) {
    for (let y = 0; y < h; y++) {
      const s = x / w, v = 1 - y / h
      ctx.fillStyle = hsvToHex(hue.value, s, v)
      ctx.fillRect(x, y, 1, 1)
    }
  }
}

function hsvToHex(h: number, s: number, v: number): string {
  const f = (n: number) => {
    const k = (n + h / 60) % 6
    return v - v * s * Math.max(Math.min(k, 4 - k, 1), 0)
  }
  const toHex = (x: number) => Math.round(x * 255).toString(16).padStart(2, '0')
  return '#' + toHex(f(5)) + toHex(f(3)) + toHex(f(1))
}

function hexToRgb(hex: string): string {
  const r = parseInt(hex.slice(1, 3), 16)
  const g = parseInt(hex.slice(3, 5), 16)
  const b = parseInt(hex.slice(5, 7), 16)
  return +""+gb(+""+${r}, , )+""+`
}

function hexToHsl(hex: string): string {
  let r = parseInt(hex.slice(1, 3), 16) / 255
  let g = parseInt(hex.slice(3, 5), 16) / 255
  let b = parseInt(hex.slice(5, 7), 16) / 255
  const max = Math.max(r, g, b), min = Math.min(r, g, b)
  const l = (max + min) / 2
  if (max === min) return +""+hsl(0, 0%, +""+${Math.round(l * 100)}%)+""+`
  const d = max - min
  const s = l > 0.5 ? d / (2 - max - min) : d / (max + min)
  let h = 0
  if (max === r) h = ((g - b) / d + (g < b ? 6 : 0)) / 6
  else if (max === g) h = ((b - r) / d + 2) / 6
  else h = ((r - g) / d + 4) / 6
  return +""+hsl(+""+${Math.round(h * 360)}, +""+${Math.round(s * 100)}%, +""+${Math.round(l * 100)}%)+""+`
}

function handleClickOutside(e: MouseEvent) {
  const el = (e.target as HTMLElement).closest('.color-input')
  if (!el) isOpen.value = false
}

onMounted(() => {
  document.addEventListener('click', handleClickOutside)
  drawCanvas()
})
onUnmounted(() => document.removeEventListener('click', handleClickOutside))
watch(hue, drawCanvas)
</script>