<script setup>
import LocationCard from '@/components/LocationCard.vue';
import SearchBar from '@/components/SearchBar.vue';
import { reactive, ref } from 'vue';

const q = ref();
const city = reactive({
  name: 'London',
  lat: 51.5073219,
  lon: -0.1276474,
});

const updateCity = ({ name, lat, lon }) => {
  city.name = name;
  city.lat = lat;
  city.lon = lon;
};
</script>

<template>
  <header
    class="bg-gray-300 dark:bg-gray-600 h-24 flex justify-evenly items-center"
  >
    <LocationCard :city="city" />
    <SearchBar v-model:q="q" @update:city="updateCity" />
  </header>
</template>
