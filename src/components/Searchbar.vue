<script setup lang="ts">
  import { ref, computed, watch, nextTick, onMounted, defineEmits, defineProps } from 'vue';
  import SearchIcon from './icons/SearchIcon.vue';
import type { ComponentPublicInstance } from 'vue';

  const apiUrl = import.meta.env.VITE_API_BASE_URL;

  interface Ingredient {
    id: number,
    english_name: string,
    tagalog_name: string
  };

  const searchTerm = ref ('');
  const ingredients = ref <Ingredient[]>([]);
  const focusedIndex = ref(-1);
  const focusedItem = ref<HTMLElement | null>(null);

  const props = defineProps<{
    selectedIngredients: Ingredient[]
  }>();
  const emits = defineEmits(['addIngredient', 'removeIngredient']);

  async function fetchIngredients () {
    try {
      const response = await fetch (`${apiUrl}/ingredients`);

      if (!response.ok) {
        throw new Error ('failed to fetch ingredients');
      }

      const data = await response.json();
      ingredients.value = data;
    }
    catch(error) {
      console.error("error fetching ingredients", error);
    }
  }

  onMounted(() => {
    fetchIngredients();
  });

  const filteredIngredients = computed(() => {
    if (!searchTerm.value) {
      return [];
    }
    return ingredients.value.filter(ingredient =>
      ingredient.english_name.toLowerCase().includes(searchTerm.value.toLowerCase()) ||
      ingredient.tagalog_name.toLowerCase().includes(searchTerm.value.toLowerCase())
    );
  });

  function addIngredient (ingredient: Ingredient) {
    emits('addIngredient', ingredient);
    searchTerm.value = '';
  };

  function removeIngredient (ingredient: Ingredient) {
    emits('removeIngredient', ingredient);
  };

  function moveDown() {
    if (!filteredIngredients.value.length) return;
    focusedIndex.value = (focusedIndex.value + 1 ) % filteredIngredients.value.length;
  }

  function moveUp() {
    if (!filteredIngredients.value.length) return;
    focusedIndex.value = (focusedIndex.value - 1 + filteredIngredients.value.length ) % filteredIngredients.value.length;
  }

  function selectItem() {
    if (focusedIndex.value < 0 || focusedIndex.value >= filteredIngredients.value.length) return;
    addIngredient(filteredIngredients.value[focusedIndex.value])
    focusedIndex.value = -1;
  }

  function setFocusedItem(el: Element | ComponentPublicInstance | null, index: number) {
  if (el instanceof HTMLElement && index === focusedIndex.value) {
    focusedItem.value = el;
  }
}

watch(focusedIndex, () => {
  nextTick(() => {
    if (focusedItem.value) {
      focusedItem.value.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
    }
  });
});

  watch(searchTerm, () => {
    focusedIndex.value = -1;
  })

</script>

<template>
  <div class="search-div">
    <div class="input-wrapper" @keydown.down.prevent="moveDown" @keydown.up.prevent="moveUp" @keydown.enter.prevent="selectItem">
      <input
        v-model="searchTerm"
        type="text"
        placeholder="Type an ingredient"
        class="search-text"
        :class="{ 'rounded-bottom': filteredIngredients.length }"
      />
      <SearchIcon class="search-icon" />
      <ul v-if="filteredIngredients.length && searchTerm">
        <li
          v-for="(ingredient, index) in filteredIngredients"
          :key="ingredient.id"
          @click="addIngredient(ingredient)"
          :class="{ focused: index === focusedIndex }"
          @mousedown.prevent="addIngredient(ingredient)"
          :ref="el => setFocusedItem(el, index)"
        >
          {{ ingredient.english_name }} <span class="tagalog-name">({{ ingredient.tagalog_name }})</span>
        </li>
      </ul>
    </div>
    <div class="selected-div">
      <div
        v-for="ingredient in props.selectedIngredients"
        :key="ingredient.id"
        class="selected-ingredient">
        <p>{{ ingredient.english_name }} <span class="tagalog-name">({{ ingredient.tagalog_name }})</span></p>
        <button @click="removeIngredient(ingredient)">x</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
  .search-div {
    width: 100%;
    position: relative;
    display: flex;
    flex-direction: column;
    padding-bottom: 10px;
  }
  .input-wrapper {
    position: relative;
    display: flex;
    flex-direction: column;
    width: 100%;
  }
  .search-text {
    width: 100%;
    padding: 8px;
    border-radius: 8px;
    border-color: black;
    border-width: 1px;
    font-size: 14px;
  }
  .search-text.rounded-bottom {
    border-radius: 8px 8px 0 0;
  }
  .search-icon {
    position: absolute;
    top: 10px;
    right: 10px;
    width: 15px;
    height: 15px;
    color: black;
    pointer-events: none;
  }
  .selected-div {
    margin-top: 10px;
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    row-gap: 10px;
    column-gap: 15px;
    font-size: 14px;
  }
  .selected-ingredient {
    background-color: rgba(231, 111, 81, 0.8);
    border-radius: 8px;
    color: white;
    padding: 4px;
    display: flex;
    flex-direction: row;
    justify-content: space-between;
  }
  button {
    background: none;
    border: none;
    color: inherit;
    cursor: pointer;
  }
  ul {
    position: absolute;
    top: 100%;
    left: 0;
    width: 100%;
    background: white;
    border: 1px solid black;
    border-radius: 0px 0px 8px 8px;
    z-index: 10;
    padding: 5px;
    list-style: none;
    max-height: 200px;
    overflow-y: auto;
    margin-top: 2px;
  }
  .tagalog-name{
    font-style: italic;
  }
  li {
    cursor: pointer;
    padding: 2px;
  }
  li:hover {
    background-color: rgba(231, 111, 81, 0.8);
    color: white;
    border-radius: 2px;
  }
  .focused {
    background-color: #f0f0f0;
  }

  @media (max-width: 768px) {
    .selected-div {
      grid-template-columns: repeat(2, 1fr);
    }
    .search-text {
      font-size: 16px;
    }
  }

  @media (max-width: 480px) {
    .selected-div {
      grid-template-columns: 1fr;
    }
  }

</style>


