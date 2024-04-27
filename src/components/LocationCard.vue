<script setup>
defineProps({
  city: {
    type: Object,
    required: true,
  },
});
</script>

<template>
  <div
    class="ml-4 h-min min-w-fit rounded-lg bg-gray-50/70 p-2 text-sm text-gray-500 backdrop-blur-md dark:bg-gray-700/70 dark:text-gray-400"
  >
    <h2>{{ city.name }}</h2>
    <p><em class="text-gray-900 dark:text-white">lat: </em>{{ city.lat }}</p>
    <p><em class="text-gray-900 dark:text-white">lon: </em>{{ city.lon }}</p>
  </div>
</template>
