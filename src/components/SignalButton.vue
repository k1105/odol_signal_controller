<script lang="ts" setup>
const props = defineProps<{
  label: string
  id: number
  color: string
  isSelected: boolean
}>()

const emit = defineEmits<{
  select: [id: number]
}>()

const onClick = () => {
  emit('select', props.id)
}
</script>

<template>
  <button
    :class="['signal-button', isSelected ? 'signal-button-selected' : '']"
    :style="{
      backgroundColor: isSelected ? color : '#000',
      borderColor: color,
      color: isSelected ? '#fff' : color,
    }"
    @click="onClick"
  >
    <span class="signal-label">{{ label }}</span>
  </button>
</template>

<style scoped>
.signal-button {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  border: 2px solid;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-size: 0.7rem;
  transition: all 0.2s ease;
  text-transform: uppercase;
  position: relative;
}

.signal-button:hover {
  transform: scale(1.05);
}

.signal-button:active {
  transform: scale(0.95);
}

.signal-label {
  user-select: none;
  text-align: center;
  color: inherit;
}

/* Color indicator dot */
.signal-button::before {
  content: '';
  position: absolute;
  top: 6px;
  right: 6px;
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background-color: v-bind('color');
  border: 1px solid #fff;
}
</style>
