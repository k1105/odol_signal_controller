<script lang="ts" setup>
import { onMounted, ref} from 'vue';
const props = defineProps<{
  index: number;
  id: number;
  name: string;
}>();
const label = ref<string>(props.name);
const id = ref<number>(props.id);
const emit = defineEmits(['edit']);

const onChange = () => {
  // Emit the select event with the id
  const emitProps = {
    index: props.index,
    name: label.value,
    id: id.value
  };
  emit('edit', emitProps);
};

</script>

<template>
  <div class="selector-button">
    <input type="text" v-model="label" class="selector-label" @change="onChange()"/>
    <p class="selector-id">
     id: 
    </p>
    <input type="number" v-model.number="id" @change="onChange()"/>
  </div>
</template>

<style scoped>
.selector-button {
  background-color: white; /* Green */
  border: 2px solid black; /* Green */
  padding: 2rem 0.25rem;
  text-decoration: none;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  margin: 4px 2px;
}

.selector-label {
  max-width: calc(100svw / 4 - 4rem);
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