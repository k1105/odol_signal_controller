<script lang="ts" setup>
import { computed } from 'vue'
const props = defineProps<{
  index: number
  id: number
  isSelected: boolean
}>()
const emit = defineEmits(['select'])

const onClick = () => {
  // Emit the select event with the id
  emit('select', props.id)
}

const toggleIndicator = computed(() => {
  return props.isSelected ? 'toggle-selected' : 'toggle-unselected'
})
</script>

<template>
  <button class="selector-button" @click="onClick">
    <p class="selector-id">id: {{ props.id }}</p>
    <div :class="toggleIndicator"></div>
  </button>
</template>

<style scoped>
.selector-button {
  background-color: #000;
  border: 2px solid #fff;
  padding: 1rem;
  text-decoration: none;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  cursor: pointer;
  transition: all 0.2s ease;
}

.selector-button:hover {
  background-color: #222;
}

.selector-label {
  font-size: 0.9rem;
  font-weight: bold;
  color: #fff;
  margin: 0;
}

.selector-id {
  font-size: 0.8rem;
  color: #ccc;
  margin: 0;
}

.toggle-unselected {
  width: 3rem;
  height: 0.8rem;
  border: 2px solid #fff;
  background-color: transparent;
  position: relative;
}

.toggle-selected {
  width: 3rem;
  height: 0.8rem;
  border: 2px solid #fff;
  background-color: #f44336;
}
</style>
