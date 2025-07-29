<script setup lang="ts">
  import { ref, watch, computed, onMounted, defineProps } from 'vue';
  import Tags from './Tags.vue';

  import ArrowLeftIcon from './icons/ArrowLeftIcon.vue';
  import ArrowRightIcon from './icons/ArrowRightIcon.vue';

  const apiUrl = import.meta.env.VITE_API_BASE_URL;


  interface Ingredient {
    id: number,
    english_name: string,
    tagalog_name: string
  };

  interface Menu {
    id: number,
    name: string,
    category: string,
    ingredients: string[],
    tags: string[],
    url: string
  }

  const menu = ref <Menu[]>([]);
  const props = defineProps<{
    selectedIngredients: Ingredient[]
  }>();

  async function fetchMenus () {
    try {
      const response = await fetch (`${apiUrl}/menus`);

      if (!response.ok) {
        throw new Error ('failed to fetch menus');
      }

      const data = await response.json();
      menu.value = data;
    }
    catch(error) {
      console.error("error fetching menus", error);
    }
  };

  onMounted(() => {
    fetchMenus();
  });

  const filteredMenus = computed(() => {
    if (!props.selectedIngredients.length) {
      return menu.value
    }

    return menu.value.filter(menuItem => {
      return props.selectedIngredients.every(selectedIngredient =>
        menuItem.ingredients.includes(selectedIngredient.english_name))
    })
  });

  const isSelected = (ingredient: string): boolean => {
    return props.selectedIngredients.some(
      i => i.english_name === ingredient
    );
  };

  const currentPage = ref<number>(1);
  const itemsPerPage = 9;

  const totalPages = computed<number>(() => {
    return Math.ceil(filteredMenus.value.length / itemsPerPage);
  });

  const paginatedMenus = computed<Menu []>(() => {
    const start = (currentPage.value - 1) * itemsPerPage;
    const end = (start + itemsPerPage);
    return filteredMenus.value.slice(start, end);
  });

  function goToPage(page:number) :void {
    if (page >= 1 && page <= totalPages.value) {
      currentPage.value = page;
    }
  };

  watch(filteredMenus, () => {
    currentPage.value = 1;
  });

  function imgSrc(category:string) {
    switch(category) {
      case 'vegetable':
        return new URL('@/assets/vegetable.svg', import.meta.url).href
      case 'pork':
        return new URL('@/assets/pork.svg', import.meta.url).href
      case 'beef':
        return new URL('@/assets/beef.svg', import.meta.url).href
      case 'chicken':
        return new URL('@/assets/chicken.svg', import.meta.url).href
      case 'fish':
        return new URL('@/assets/fish.svg', import.meta.url).href
    }
  }

</script>

<template>
  <div v-if="filteredMenus.length">
    <div class="main-card">
      <div v-for="item in paginatedMenus" :key="item.id" class="container">
        <img :src="imgSrc(item.category)" alt="icon" class="card-image" />
        <div class="title">
          <p class="menu">{{ item.name }}</p>
          <p class="category">{{ item.category }}</p>
        </div>
        <div class="content">
          <p class="ingredients">
            Ingredients:
            <span v-for="(ingredient, index) in item.ingredients" :key="index">
              <span :class="{ highlight: isSelected(ingredient)}">{{ ingredient }}</span><span v-if="index < item.ingredients.length - 1">, </span>
            </span>
          </p>
        </div>

        <div class="bottom-section">
          <div class="footer-div">
            <a :href="item.url" target="_blank" class="text">View full recipe > </a>
          </div>
          <div class="tags-div">
            <Tags :tags="item.tags" />
          </div>
        </div>

      </div>
    </div>

    <div class="pagination-controls">
      <button class="page-button" @click="goToPage(currentPage - 1)" :disabled="currentPage === 1">
        <ArrowLeftIcon class="page-icon" />
      </button>
      <span style="color: #A9A9AC;">Page {{ currentPage }} of {{ totalPages }}</span>
      <button class="page-button" @click="goToPage(currentPage + 1)" :disabled="currentPage === totalPages">
        <ArrowRightIcon class="page-icon" />
      </button>
    </div>

  </div>

  <div v-else-if="props.selectedIngredients.length > 0">
    <div class="loader-container">
      <p class="loader-text">No results found.</p>
    </div>
  </div>

  <div v-else>
    <div class="loader-container">
      <div class="loader"></div>
      <p class="loader-text">Loading ...</p>
    </div>
  </div>
</template>

<style scoped>
  .main-card {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 15px;
  }
  .container {
    display: grid;
    grid-template-rows: auto 1fr auto;
    gap: 10px;
    color: black;
    border: 2px solid #F5F5F5;
    padding: 5%;
    border-radius: 10px;
    height: 100%;
  }
  .title {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
  }
  .content {
    min-height: 70px;
  }
  .bottom-section {
    display: flex;
    flex-direction: column;
    gap: 10px;
    margin-top: auto;
  }
  .menu, .ingredients{
    font-weight: 700;
  }
  .loader-container {
    display: flex;
    flex-direction: column;
    gap: 10px;
    justify-content: center;
    align-items: center;
    height: 80vh;
  }
  .loader {
    border: 5px solid #f3f3f3;
    border-top: 5px solid #E76F51;
    border-radius: 50%;
    width: 50px;
    height: 50px;
    animation: spin 2s linear infinite;
  }
  .highlight {
    color: #E76F51;
    font-weight: 700;
  }
  .category {
    color: darkgray;
  }
  .pagination-controls {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 20px;
    margin-bottom: 40px;
  }
  .page-button {
    background: none;
    border: none;
    color: #E76F51;
    border-radius: 3px;
    padding: 5px;
    cursor: pointer;
  }
  .page-button:hover {
    background-color: #E76F51;
    color: white;
  }
  .page-icon{
    width: 15px;
    height: 15px;
    cursor: pointer;
  }
  .text{
    color: #E76F51;
    text-decoration: none;
  }
  .card-image {
    width: 100%;
    height: 200px;
    object-fit: contain;
  }


  @keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
  }

  @media (max-width: 768px) {
    .main-card {
      grid-template-columns: 1fr;
    }
  }
</style>

