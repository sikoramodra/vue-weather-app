<script setup>
import { ref, watch } from 'vue';

const props = defineProps({
  q: {
    type: String,
  },
});

const cityList = ref();
const showList = ref(false);

const emit = defineEmits(['update:q', 'update:city']);

const fetchCityData = async (q) => {
  try {
    const response = await fetch(
      `https://api.openweathermap.org/geo/1.0/direct?appid=deaf4353cc00ef0e58e1fe493454be49&q=${q}&limit=5`,
    );
    if (!response.ok) {
      throw new Error(`Failed to fetch city: ${q}`);
    }
    return await response.json();
  } catch (e) {
    console.error(e.message);
  }
};

const autocomplete = (name, lat, lon) => {
  emit('update:city', { name, lat, lon });
  emit('update:q', null);
  showList.value = false;
  cityList.value = null;
};

const inputFocusOut = (event) => {
  if (
    event.relatedTarget !== document.querySelector('input') &&
    event.relatedTarget !== document.querySelector('div[tabindex="0"]')
  ) {
    emit('update:q', null);
  }
};

watch(
  () => props.q,
  async () => {
    if (props.q) {
      cityList.value = await fetchCityData(props.q);
      showList.value = true;
    } else {
      showList.value = false;
      cityList.value = null;
    }
  },
);
</script>

<template>
  <div class="relative mx-4 w-full min-w-64">
    <div
      class="pointer-events-none absolute inset-y-0 start-0 flex items-center ps-3"
    >
      <svg
        class="z-20 h-4 w-4 text-gray-500 dark:text-gray-400"
        fill="none"
        viewBox="0 0 20 20"
      >
        <path
          stroke="currentColor"
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
          d="m19 19-4-4m0-7A7 7 0 1 1 1 8a7 7 0 0 1 14 0Z"
        />
      </svg>
    </div>
    <input
      type="text"
      class="block w-full rounded-lg border border-gray-300 bg-gray-50/70 p-2.5 ps-10 text-sm text-gray-900 placeholder-gray-500 outline-none backdrop-blur-md focus:border-blue-500 focus:ring-blue-500 dark:border-gray-600 dark:bg-gray-700/70 dark:text-white dark:placeholder-gray-400 dark:focus:border-blue-500 dark:focus:ring-blue-500"
      placeholder="Search for city"
      :value="q"
      @input="$emit('update:q', $event.target.value)"
      @focusout="inputFocusOut"
    />
    <div
      tabindex="0"
      v-if="showList"
      id="dropdown-menu"
      class="absolute left-0 mt-2 w-full space-y-1 rounded-lg bg-gray-50/70 p-1 shadow-lg backdrop-blur-md dark:bg-gray-700/70"
    >
      <template v-if="cityList.length !== 0">
        <p
          class="block cursor-pointer rounded-lg px-4 py-2 text-gray-900 hover:bg-gray-300/30 dark:text-white dark:hover:bg-gray-700/30"
          v-for="(city, index) in cityList"
          :key="index"
          @click="autocomplete(city.name, city.lat, city.lon)"
        >
          {{ city.name }}
        </p>
      </template>
      <template v-else>
        <p class="block rounded-lg px-4 py-2 text-gray-500 dark:text-white">
          No cities found
        </p>
      </template>
    </div>
  </div>
</template>
