<script lang="ts" setup>
const emit = defineEmits(['stop'])

const onClick = () => {
  // Emit the select event with the id
  emit('stop')
}
</script>

<template>
  <button class="stop-button" @click="onClick">
    <span class="stop-label">STOP</span>
  </button>
</template>

<style scoped>
.stop-button {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  background-color: #f44336;
  border: 3px solid #fff;
  color: #fff;
  font-weight: bold;
  font-size: 1.2rem;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
  text-transform: uppercase;
  box-shadow: 0 4px 8px rgba(244, 67, 54, 0.3);
}

.stop-button:hover {
  background-color: #e53935;
  transform: scale(1.05);
  box-shadow: 0 6px 12px rgba(244, 67, 54, 0.4);
}

.stop-button:active {
  transform: scale(0.95);
}

.stop-label {
  user-select: none;
  text-align: center;
}
</style>
