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
    class="bg-gray-50/70 dark:bg-gray-700/70 dark:text-gray-400 backdrop-blur-md h-min p-2 ml-4 text-sm rounded-lg min-w-fit"
  >
    <h2>{{ city.name }}</h2>
    <p><em class="dark:text-white">lat: </em>{{ city.lat }}</p>
    <p><em class="dark:text-white">lon: </em>{{ city.lon }}</p>
  </div>
</template>
