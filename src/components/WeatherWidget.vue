<script setup>
import { ref, watch } from 'vue';

const props = defineProps({
  city: {
    type: Object,
    required: true,
  },
});

const cityForecast = ref();
const cityCurrent = ref();
const loading = ref(true);

const fetchCityForecast = async (city) => {
  try {
    const response = await fetch(
      `https://api.openweathermap.org/data/2.5/forecast?appid=deaf4353cc00ef0e58e1fe493454be49&lat=${city.lat}&lon=${city.lon}&units=metric&lang=pl`,
    );
    if (!response.ok) {
      console.error(`Failed to fetch forecast for city: ${city.name}`);
      return;
    }
    const data = await response.json();
    data.list = data.list.filter(
      (item, index) => index % 8 === 0 && index !== 0,
    );
    data.cnt = data.list.length;
    data.list.forEach((item, index) => {
      data.list[index].main.temp_F = item.main.temp * 1.8 + 32;
    });
    cityForecast.value = data;
  } catch (e) {
    console.error(e.message);
  }
};

const fetchCityCurrent = async (city) => {
  try {
    const response = await fetch(
      `https://api.openweathermap.org/data/2.5/weather?appid=deaf4353cc00ef0e58e1fe493454be49&lat=${city.lat}&lon=${city.lon}&units=metric&lang=pl`,
    );
    if (!response.ok) {
      console.error(`Failed to fetch current weather for city: ${city.name}`);
      return;
    }
    const data = await response.json();
    data.main.temp_F = data.main.temp * 1.8 + 32;
    cityCurrent.value = data;
  } catch (e) {
    console.error(e.message);
  }
};

const formatDate = (dt, tz, format) => {
  const date = new Date(dt * 1000);
  date.setMinutes(date.getMinutes() - tz / 60);
  return date.toLocaleDateString('pl-PL', format);
};

watch(
  () => props.city,
  async () => {
    loading.value = true;
    await Promise.all([
      fetchCityForecast(props.city),
      fetchCityCurrent(props.city),
    ]);
    loading.value = false;
  },
  { immediate: true, deep: true },
);
</script>

<template>
  <div
    class="container mx-auto flex w-full max-w-3xl flex-wrap justify-center px-6"
    v-if="!loading"
  >
    <div class="flex w-full rounded-lg">
      <div class="w-full rounded-lg bg-blue-400 py-6 pl-8 pr-32 text-white">
        <!-- TODO: dynamic background -->
        <div class="mb-12">
          <h2 class="pb-1 text-3xl font-bold leading-none">
            {{
              formatDate(cityCurrent.dt, cityCurrent.timezone, {
                weekday: 'long',
                hour: 'numeric',
                minute: 'numeric',
              })
            }}
          </h2>
          <h3 class="pb-2 pl-1 leading-none">
            {{
              formatDate(cityCurrent.dt, cityCurrent.timezone, {
                day: '2-digit',
                month: 'short',
                year: '2-digit',
              })
            }}
          </h3>
          <p class="flex opacity-75">
            <svg class="mr-1 inline w-4" viewBox="0 0 425 425">
              <path
                d="M213.285,0h-0.608C139.114,0,79.268,59.826,79.268,133.361c0,48.202,21.952,111.817,65.246,189.081   c32.098,57.281,64.646,101.152,64.972,101.588c0.906,1.217,2.334,1.934,3.847,1.934c0.043,0,0.087,0,0.13-0.002   c1.561-0.043,3.002-0.842,3.868-2.143c0.321-0.486,32.637-49.287,64.517-108.976c43.03-80.563,64.848-141.624,64.848-181.482   C346.693,59.825,286.846,0,213.285,0z M274.865,136.62c0,34.124-27.761,61.884-61.885,61.884   c-34.123,0-61.884-27.761-61.884-61.884s27.761-61.884,61.884-61.884C247.104,74.736,274.865,102.497,274.865,136.62z"
                fill="#FFFFFF"
              />
            </svg>
            {{ cityCurrent.name }}, {{ cityCurrent.sys.country }}
          </p>
        </div>
        <div>
          <img
            class="mb-4 w-24 rounded-full bg-white/30"
            :src="`https://openweathermap.org/img/wn/${cityCurrent.weather[0].icon}@4x.png`"
            alt="weather icon"
          />
          <strong class="font-weight-bolder block text-5xl leading-none"
            >{{ Math.round(cityCurrent.main.temp) }}ºC /
            {{ Math.round(cityCurrent.main.temp_F) }}ºF</strong
          >
          <b class="block text-xl font-bold">{{
            cityCurrent.weather[0].description
          }}</b>
        </div>
      </div>
    </div>

    <div class="flex w-full">
      <div
        class="mx-3 w-full rounded-b-lg bg-gray-300 p-8 dark:bg-gray-600 dark:text-white"
      >
        <div class="mb-4 flex w-full justify-between">
          <div class="w-auto font-bold uppercase">Clouds</div>
          <div class="w-auto text-right">{{ cityCurrent.clouds.all }} %</div>
        </div>
        <div class="mb-4 flex w-full justify-between">
          <div class="w-auto font-bold uppercase">Humidity</div>
          <div class="w-auto text-right">{{ cityCurrent.main.humidity }} %</div>
        </div>
        <div class="mb-8 flex w-full justify-between">
          <div class="w-auto font-bold uppercase">Wind</div>
          <div class="w-auto text-right">{{ cityCurrent.wind.speed }} m/s</div>
        </div>
        <div class="flex flex-row gap-3">
          <div
            class="flex w-1/4 flex-col items-center rounded-lg bg-gray-50/30 pb-4 dark:bg-gray-700/70"
            v-for="(forecast, index) in cityForecast.list"
            :key="index"
          >
            <img
              class="my-3 w-14 rounded-full bg-white/30 text-center"
              :src="`https://openweathermap.org/img/wn/${forecast.weather[0].icon}.png`"
              alt="weather icon"
            />
            <div class="text-center">
              <b class="font-normal">{{
                formatDate(forecast.dt, cityCurrent.timezone, {
                  weekday: 'short',
                })
              }}</b
              ><br />
              <strong class="text-xl"
                >{{ Math.round(forecast.main.temp) }}ºC /
                {{ Math.round(forecast.main.temp_F) }}ºF</strong
              >
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
