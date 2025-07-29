<script setup lang="ts">
  import { ref } from 'vue';
  import Card from './components/Card.vue';
  import Searchbar from './components/Searchbar.vue';
  import Chatbox from './components/Chatbox.vue';

  interface Ingredient {
    id: number,
    english_name: string,
    tagalog_name: string
  }

  const selectedIngredients = ref <Ingredient[]>([]);

  const addIngredient = (ingredient: Ingredient) => {
    const selected = selectedIngredients.value.find(i => i.id === ingredient.id)
    if (!selected) {
      selectedIngredients.value.push(ingredient);
    }
  };

  const removeIngredient = (ingredient: Ingredient) => {
    selectedIngredients.value = selectedIngredients.value.filter(i => i.id !== ingredient.id)
  };


</script>

<template>
  <div>
    <h1 class="logo-name">ma, anong ulam?</h1>
    <Searchbar
      :selectedIngredients="selectedIngredients"
      @addIngredient="addIngredient"
      @removeIngredient="removeIngredient"
      class="search-div"
    />
  <div>
    <Card
      :selectedIngredients="selectedIngredients"
    />
  </div>
  <div>
    <Chatbox />
  </div>
  </div>
</template>

<style scoped>
  @import url('https://fonts.googleapis.com/css2?family=Caprasimo&family=Grandstander:ital,wght@0,100..900;1,100..900&display=swap');
  .logo-name {
    font-family: 'Caprasimo', serif;
    color: #E76F51;
    font-weight: 400;
  }
</style>
