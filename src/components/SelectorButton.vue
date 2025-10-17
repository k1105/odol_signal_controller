<script lang="ts" setup>
import { ref } from 'vue'

const props = defineProps<{
  index: number
  id: number
  name: string
  imageUrl?: string
  isSelected: boolean
  borderColor: string
  disabled: boolean
}>()
const emit = defineEmits(['select'])

const imageError = ref(false)

const onClick = () => {
  // Emit the select event with the id
  emit('select', props.id)
}

const onImageError = () => {
  imageError.value = true
}
</script>

<template>
  <button
    :class="['selector-button', { 'selector-button-selected': props.isSelected }]"
    :disabled="props.disabled"
    @click="onClick"
  >
    <div class="effect-label">{{ props.name }}</div>
    <div class="effect-preview">
      <img
        v-if="props.imageUrl && !imageError"
        :src="props.imageUrl"
        :alt="props.name"
        class="preview-image"
        @error="onImageError"
      />
      <img v-else src="/effect_placeholder.png" :alt="props.name" class="preview-image" />
    </div>
    <div class="effect-index">{{ props.index + 1 }}</div>
  </button>
</template>

<style scoped>
.selector-button {
  background-color: #000;
  border: 2px solid v-bind('borderColor');
  text-decoration: none;
  display: flex;
  flex-direction: column;
  align-items: stretch;
  justify-content: space-between;
  cursor: pointer;
  transition: all 0.2s ease;
  position: relative;
  height: 100%;
  border-radius: 0 8px 0 8px;
}

.selector-button:hover {
  background-color: #222;
}

.selector-button-selected {
  background-color: #222;
  border-width: 3px;
}

.selector-button:disabled {
  opacity: 0.3;
  cursor: not-allowed;
  pointer-events: none;
}

.selector-button:focus {
  border: 2px solid #fff;
  outline-offset: 2px;

  .effect-label {
    background-color: rgba(255, 255, 255, 1);
  }
}

.effect-label {
  position: absolute;
  top: 0;
  left: 0;
  background-color: rgba(255, 255, 255, 0.5);
  color: #000;
  padding: 2px 6px;
  font-size: 0.7rem;
  font-weight: bold;
  z-index: 2;
}

.effect-preview {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  border-radius: 4px;
  position: relative;
}

.preview-image {
  width: 100%;
  height: 20vh;
  object-fit: cover;
  object-position: center;
  border-radius: 4px;
}

.effect-index {
  position: absolute;
  bottom: 8px;
  right: 8px;
  color: #fff;
  font-size: 0.8rem;
  font-weight: bold;
  z-index: 2;
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
}
</style>
