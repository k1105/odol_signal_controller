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
  background-color: #fff;
  border: 3px solid #f44336;
  box-shadow: 0 0 0 3px white;
  color: #f44336;
  font-weight: bold;
  font-size: 1.2rem;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
  text-transform: uppercase;
}

.stop-button:hover {
  background-color: #f5f5f5;
  transform: scale(1.05);
}

.stop-button:active {
  transform: scale(0.95);
}

.stop-label {
  user-select: none;
  text-align: center;
}
</style>
