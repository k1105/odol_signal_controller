<script lang="ts" setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import p5 from 'p5'

const props = defineProps<{
  label: string
  id: number
  color: string
  isSelected: boolean
}>()

const emit = defineEmits<{
  select: [id: number]
}>()

const canvasContainer = ref<HTMLDivElement | null>(null)
let p5Instance: p5 | null = null
let isHovered = ref(false)
let scale = ref(1)

const onClick = () => {
  emit('select', props.id)
  // クリックアニメーション
  scale.value = 0.95
  setTimeout(() => {
    scale.value = 1
  }, 100)
}

const drawButton = (p: p5) => {
  p.clear()

  const centerX = p.width / 2
  const centerY = p.height / 2
  const buttonSize = 60
  const currentScale = scale.value * (isHovered.value ? 1.05 : 1)

  p.push()
  p.translate(centerX, centerY)
  p.scale(currentScale)

  // 色をパース
  const c = p.color(props.color)

  p.noFill()
  p.stroke(c)
  p.strokeWeight(6)

  if (props.isSelected) {
    p.beginShape()
    for (let i = 0; i < 10; i++) {
      const angle = p.map(i, 0, 10, 0, p.TWO_PI) - p.HALF_PI
      const r = i % 2 === 0 ? buttonSize / 2 : (buttonSize / 2) * 0.5
      const x = r * p.cos(angle)
      const y = r * p.sin(angle)
      p.vertex(x, y)
    }
    p.endShape(p.CLOSE)
  } else {
    p.circle(0, 0, buttonSize)
  }

  p.pop()
}

onMounted(() => {
  if (!canvasContainer.value) return

  const sketch = (p: p5) => {
    p.setup = () => {
      p.createCanvas(70, 70)
      p.frameRate(30)
    }

    p.draw = () => {
      drawButton(p)
    }

    p.mousePressed = () => {
      if (p.mouseX >= 0 && p.mouseX <= p.width && p.mouseY >= 0 && p.mouseY <= p.height) {
        onClick()
        return false
      }
    }
  }

  p5Instance = new p5(sketch, canvasContainer.value)
})

onUnmounted(() => {
  p5Instance?.remove()
})

// props の変更を監視して再描画
watch(
  () => [props.color, props.isSelected, scale.value, isHovered.value],
  () => {
    if (p5Instance) {
      drawButton(p5Instance)
    }
  },
)

const onMouseEnter = () => {
  isHovered.value = true
}

const onMouseLeave = () => {
  isHovered.value = false
}
</script>

<template>
  <div
    ref="canvasContainer"
    class="signal-button-container"
    @mouseenter="onMouseEnter"
    @mouseleave="onMouseLeave"
  ></div>
</template>

<style scoped>
.signal-button-container {
  width: 70px;
  height: 70px;
  cursor: pointer;
  display: inline-block;
  position: relative;
}

.signal-button-container :deep(canvas) {
  display: block;
}
</style>
