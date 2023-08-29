<template>
  <div v-for="city in savedCities" :key="city.id">
    <CityCard :city="city" @click="goToCityView(city)" />
  </div>

  <p v-if="savedCities.length === 0">
    No location added yet. Search for a city above to get started.
  </p>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";
import CityCard from "./CityCard.vue";
import { useRouter } from "vue-router";
const savedCities = ref([]);
const openWeatherAPIKey = import.meta.env.VITE_openWeatherAPIKey;

const getCities = async () => {
  if (localStorage.getItem("savedCities")) {
    savedCities.value = JSON.parse(localStorage.getItem("savedCities"));

    const requests = [];
    savedCities.value.forEach((city) => {
      requests.push(
        axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${city.coord.lat}&lon=${city.coord.lng}&appid=${openWeatherAPIKey}&units=imperial`)
      );
    });

    const weatherData = await Promise.all(requests);

    await new Promise((resolve) => setTimeout(resolve, 1000));

    weatherData.forEach((city, index) => {
      savedCities.value[index].weather = city.data;
    });
  }
};


await getCities();

const router = useRouter();
const goToCityView = (city) => {
  router.push({
    name: 'CityView',
    params: {
      state: city.state,
      city: city.city
    },
    query: {
      id: city.id,
      lat: city.coord.lat,
      lng: city.coord.lng
    }
  })
}
</script>