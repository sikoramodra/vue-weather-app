<script setup>
import LocationCard from '@/components/LocationCard.vue';
import SearchBar from '@/components/SearchBar.vue';
import { ref } from 'vue';

const q = ref();
</script>

<template>
  <header class="bg-gray-400 h-20 flex justify-evenly items-center">
    <LocationCard />
    <SearchBar v-model:q="q" />
  </header>
</template>
