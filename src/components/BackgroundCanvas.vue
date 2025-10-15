<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'
import p5 from 'p5'

const props = defineProps<{
  selectedColor: string
}>()

const canvasContainer = ref<HTMLDivElement | null>(null)
let p5Instance: p5 | null = null

// HEX色をRGBに変換する関数
function hexToRgb(hex: string): { r: number; g: number; b: number } {
  // #を削除
  const cleanHex = hex.replace('#', '')

  // RGBに変換
  const r = parseInt(cleanHex.substring(0, 2), 16)
  const g = parseInt(cleanHex.substring(2, 4), 16)
  const b = parseInt(cleanHex.substring(4, 6), 16)

  return { r, g, b }
}

onMounted(() => {
  if (!canvasContainer.value) return

  const sketch = (p: p5) => {
    p.setup = () => {
      p.createCanvas(p.windowWidth, p.windowHeight)
      p.background(0)
    }

    p.draw = () => {
      // 選択された色をRGBに変換
      const rgb = hexToRgb(props.selectedColor)

      // 背景色を動的に変更
      p.background(rgb.r, rgb.g, rgb.b)

      // ランダムな円を描画（色も動的に）
      p.noStroke()
      // 選択色を少し透明にして円を描画
      p.fill(rgb.r, rgb.g, rgb.b, 30)
      const x = p.random(p.width)
      const y = p.random(p.height)
      p.circle(x, y, 50)
    }

    p.windowResized = () => {
      p.resizeCanvas(p.windowWidth, p.windowHeight)
    }
  }

  p5Instance = new p5(sketch, canvasContainer.value)
})

onUnmounted(() => {
  if (p5Instance) {
    p5Instance.remove()
  }
})
</script>

<template>
  <div ref="canvasContainer" class="background-canvas"></div>
</template>

<style scoped>
.background-canvas {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: -1;
  pointer-events: none;
}
</style>
