# 🎨 vue-color-input

Beautiful, accessible color picker for Vue 3.

## Features

- 🎯 HEX, RGB, HSL support
- ♿ Full keyboard navigation & screen reader support
- 📦 Zero dependencies, <3kb gzipped
- 🎨 Customizable preset colors
- 🌙 Dark mode support
- 💅 CSS custom properties for theming

## Install

`bash
npm install vue-color-input
`

## Usage

`vue
<template>
  <ColorInput v-model="color" :presets="presets" />
  <p>Selected: {{ color }}</p>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { ColorInput } from 'vue-color-input'

const color = ref('#6366f1')
const presets = ['#ef4444', '#f59e0b', '#10b981', '#3b82f6', '#8b5cf6']
</script>
`

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| modelValue | string | '#000000' | Color value (v-model) |
| format | 'hex'\|'rgb'\|'hsl' | 'hex' | Output format |
| presets | string[] | [] | Preset color swatches |
| disabled | boolean | false | Disable input |
| size | 'sm'\|'md'\|'lg' | 'md' | Component size |

## License

MIT