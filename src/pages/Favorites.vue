<script setup>
import { onMounted, ref } from "vue";
import axios from "axios";
import CardList from "../components/CardList.vue";

const favorites = ref([]);
const isLoading = ref(true);

onMounted(async () => {
  try {
    isLoading.value = true;

    // Загружаем избранное
    const { data: favoritesData } = await axios.get(
      "https://5c4f68a7b58c636d.mokky.dev/favorites"
    );


    if (!favoritesData || favoritesData.length === 0) {
      favorites.value = [];
      return;
    }

    // Получаем ID всех избранных товаров
    const itemIds = favoritesData.map((fav) => fav.item_id);

    // Загружаем все спортивные костюмы
    const { data: sportsuits } = await axios.get(
      "https://5c4f68a7b58c636d.mokky.dev/sportsuits"
    );


    // Фильтруем только избранные товары
    const favoriteItems = sportsuits.filter((item) =>
      itemIds.includes(item.id)
    );


    // Форматируем данные
    favorites.value = favoriteItems.map((item) => {
      const favorite = favoritesData.find((fav) => fav.item_id === item.id);

      // Получаем сохраненный размер из favoritesData или из localStorage
      let savedSize = favorite?.selectedSize || "";

      // Если в favoritesData нет размера, пробуем получить из localStorage
      if (!savedSize) {
        const savedSizes = JSON.parse(
          localStorage.getItem("selectedSizes") || "{}"
        );
        savedSize = savedSizes[item.id] || "";
      }

      // Если сохраненного размера нет в доступных, берем первый доступный
      const availableSizes = item.sizes ? item.sizes.split(",") : [];
      if (savedSize && !availableSizes.includes(savedSize)) {
        savedSize = availableSizes[0] || "";
      } else if (!savedSize) {
        savedSize = availableSizes[0] || "";
      }

      return {
        ...item,
        isFavorite: true,
        favoritesId: favorite?.id || null,
        availableSizes: availableSizes,
        selectedSize: savedSize, // Добавляем сохраненный размер
      };
    });

  } catch (error) {
    console.error("Ошибка загрузки избранного:", error);
    favorites.value = [];
  } finally {
    isLoading.value = false;
  }
});
</script>

<template>
  <div>
    <h2 class="text-3xl font-bold mb-8">Мои закладки</h2>

    <!-- Индикатор загрузки -->
    <div v-if="isLoading" class="text-center py-12">
      <div
        class="inline-block animate-spin rounded-full h-12 w-12 border-t-2 border-b-2 border-blue-500"
      ></div>
      <p class="mt-4 text-gray-600">Загрузка избранного...</p>
    </div>

    <!-- Если нет избранных товаров -->
    <div v-else-if="favorites.length === 0" class="text-center py-12">
      <div class="text-5xl mb-4 opacity-60">❤️</div>
      <h3 class="text-xl font-semibold mb-2 text-gray-700">
        Закладок пока нет
      </h3>
      <p class="text-gray-500 max-w-md mx-auto">
        Добавляйте понравившиеся спортивные костюмы в избранное, нажимая на
        сердечко ❤️
      </p>
      <router-link
        to="/"
        class="inline-block mt-6 px-6 py-3 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition font-medium"
      >
        Перейти к покупкам
      </router-link>
    </div>

    <!-- Отображение избранных товаров -->
    <div v-else>
      <p class="text-gray-600 mb-6">
        Найдено {{ favorites.length }}
        {{
          favorites.length === 1
            ? "товар"
            : favorites.length > 1 && favorites.length < 5
            ? "товара"
            : "товаров"
        }}
        в избранном
      </p>
      <CardList :items="favorites" :is-favorites="true" />
    </div>
  </div>
</template>
