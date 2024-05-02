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

const getWeatherColor = (weatherGroup) => {
  const weatherColors = {
    Thunderstorm: 'bg-blue-600/30',
    Drizzle: 'bg-blue-400/30',
    Rain: 'bg-blue-500/30',
    Snow: 'bg-white/30',
    Clear: 'bg-yellow-500/30',
    Clouds: 'bg-gray-300/30',
    Other: 'bg-gray-500/30',
  };

  const key = Object.keys(weatherColors).find((key) => key === weatherGroup);
  return weatherColors[key] || weatherColors.Other;
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
  >
    <template v-if="!loading">
      <div class="flex w-full rounded-lg">
        <div
          class="w-full rounded-lg py-6 pl-8 pr-32 text-white"
          :class="getWeatherColor(cityCurrent.weather[0].main)"
        >
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
            <div class="w-auto text-right">
              {{ cityCurrent.main.humidity }} %
            </div>
          </div>
          <div class="mb-8 flex w-full justify-between">
            <div class="w-auto font-bold uppercase">Wind</div>
            <div class="w-auto text-right">
              {{ cityCurrent.wind.speed }} m/s
            </div>
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
    </template>
    <template v-else>
      <svg
        class="h-32 w-32 animate-spin fill-gray-800 text-gray-300 dark:fill-white dark:text-gray-600"
        viewBox="0 0 100 101"
      >
        <path
          d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z"
          fill="currentColor"
        />
        <path
          d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z"
          fill="currentFill"
        />
      </svg>
    </template>
  </div>
</template>
