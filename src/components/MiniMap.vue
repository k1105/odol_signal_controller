<script setup lang="ts">

const props = defineProps<{
  effectList: Array<{ id: number; name: string }>;
  selectedEffect: { id: number; name: string } | null;
  page: number;
}>();

const totalPages = Math.ceil(props.effectList.length / 16);
function isCurrentPage(page: number): string {
  return page === props.page ? 'current-page' : '';
}

</script>

<template>
  <div class="minimap">
      <div  v-for="i in totalPages" >
        <div :class="isCurrentPage(i)" class="effect-grid">
          <div v-for="effect in props.effectList.slice((i - 1) * 16, i * 16)" 
             :key="effect.id" 
             >
            <div class="selected-effect" v-if="effect.id === selectedEffect?.id"></div>
            <div class="effect-item" v-else ></div>
          </div>
        </div>
      </div>
  </div>
</template>

<style scoped>
.minimap {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px;
}

.effect-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 2px;
}

.effect-item {
  width: 5px;
  height: 5px;
  background-color: #ccc;
  border: 1px solid #999;
}

.selected-effect {
  width: 5px;
  height: 5px;
  background-color: green;
  border: 1px solid #999;
}

.current-page {
  background-color: rgba(0, 255, 0, 0.2);
  border: 1px solid green;
  padding : 2px;
}
</style>