<script lang="ts" setup>
import { ref, computed } from 'vue';
const props = defineProps<{
  index: number;
  id: number;
  name: string;
  isSelected: boolean;
}>();
const emit = defineEmits(['select']);

const onClick = () => {
  // Emit the select event with the id
  emit('select', props.id);
};

const toggleIndicator = computed(() => {
  return props.isSelected ? 'toggle-selected' : 'toggle-unselected';
});

</script>

<template>
  <button class="selector-button" @click="onClick">
    <p class="selector-label">
      {{ props.name }}      
    </p>
    <p class="selector-id">
      id: {{ props.id }}
    </p>
    <div :class="toggleIndicator"></div>
  </button>
</template>

<style scoped>
.selector-button {
  background-color: white; /* Green */
  border: 2px solid black; /* Green */
  padding: 1rem 0.25rem;
  text-decoration: none;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  margin: 4px 2px;
  cursor: pointer;
}

.selector-label {
  font-size: 1rem;
  font-weight: bold;
  color: black;
  margin: 0;
}

.selector-id {
  font-size: 1rem;
  color: gray;
  margin: 0;
}

.toggle-unselected {
  width: 4rem;
  height: 1rem;
  border: 2px solid black; /* Green */
  background-color: white; /* Green */
  position: relative;
}

.toggle-selected {
  width: 4rem;
  height: 1rem;
  border: 2px solid black; /* Green */
  background-color: green; /* Green */
}
</style>