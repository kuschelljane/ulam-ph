<script setup lang="ts">
  defineProps<{
    tags?: string[]
  }>()
</script>

<template>
    <div class="tags-container">
      <div v-if="tags?.includes('savory')" class="savory">
        <p class="text">savory</p>
      </div>
      <div  v-if="tags?.includes('quick')" class="quick">
        <p class="text">quick</p>
      </div>
      <div  v-if="tags?.includes('healthy')" class="healthy">
        <p class="text">healthy</p>
      </div>
    </div>
</template>

<style scoped>
  .tags-container {
    display: grid;
    grid-template-columns: 70px 70px 70px;
    gap: 5px;
    color: white;
    font-size: 12px;
  }
  .savory, .quick, .healthy {
    border-radius: 20px;
    padding: 3px;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .savory{
    background-color:  #E76F51;
  }
  .quick {
    background-color: #FF9D23;
  }
  .healthy {
    background-color: #2D4F2B;
  }
  .text {
    font-size: 12px;
  }
</style>
