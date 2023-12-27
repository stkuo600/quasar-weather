<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        @keyup.enter="getWeatherBySearch"
        placeholder="Search"
        dark
        borderless
      >
        <template v-slot:before>
          <q-icon name="my_location" @click="getLocationWeather" />
        </template>

        <template v-slot:hint> Field hint </template>

        <template v-slot:append>
          <q-btn round dense flat icon="search" @click="getWeatherBySearch" />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData !== null">
      <div class="col text-center text-white">
        <div class="text-h4 text-weight-light">{{ weatherData.name }}</div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg">
          <span>{{ Math.round(weatherData.main.temp) }}</span
          ><span class="text-h4 relative-position degree">&degC</span>
        </div>
      </div>

      <div class="col text-center">
        <img
          :src="`https://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`"
        />
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-light">Quasar<br />Weather</div>
        <q-btn class="col" flat @click="getLocationWeather">
          <q-icon left size="3em" name="my_location" />
          <div>Find my location</div>
        </q-btn>
      </div>
    </template>
    <div class="col skyline"></div>
  </q-page>
</template>

<script setup lang="ts">
import { computed, ref } from "vue";
import axios from "axios";
import { WeatherModel } from "../models/weather_model";
import { useQuasar } from "quasar";

const apiUrl = "https://api.openweathermap.org/data/2.5/weather";
const search = ref("");
let weatherData = ref<WeatherModel | null>(null);
let latitude: number | null = null;
let longitude: number | null = null;
const apiKey = "a45ea032a8afe65bd40f29137c3741ff";
const httpClient = axios.create({
  baseURL: "https://api.openweathermap.org/data/2.5/weather",
  timeout: 1000,
});

const $q = useQuasar();

const bgClass = computed(() => {
  if (weatherData.value !== null) {
    if (weatherData.value.weather[0].icon.endsWith("n")) {
      return "bg-night";
    } else {
      return "bg-day";
    }
  }
});

const getLocationWeather = () => {
  $q.loading.show();
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition((position) => {
      console.log(position.coords.latitude, position.coords.longitude);
      latitude = position.coords.latitude;
      longitude = position.coords.longitude;
      getWeather();
    });
  }
};

const getWeather = async () => {
  $q.loading.show();
  const res = await httpClient
    .get(apiUrl, {
      params: {
        lat: latitude,
        lon: longitude,
        appid: apiKey,
        units: "metric",
      },
    })
    .then((res) => {
      weatherData.value = JSON.parse(JSON.stringify(res.data));
    })
    .catch((err) => {
      console.log(err);
    })
    .finally(() => {
      $q.loading.hide();
    });
};

const getWeatherBySearch = async () => {
  $q.loading.show();

  const res = await httpClient
    .get(apiUrl, {
      params: {
        q: search.value,
        appid: apiKey,
        units: "metric",
      },
    })
    .then((res) => {
      weatherData.value = JSON.parse(JSON.stringify(res.data));
    })
    .catch((err) => {
      console.log(err);
    })
    .finally(() => {
      $q.loading.hide();
    });
};
</script>

<style lang="sass" scoped>
.q-page
  background: linear-gradient(to bottom, #136a8a, #267871)
  &.bg-night
    background: linear-gradient(to bottom, #232526, #414345)
  &.bg-day
    background: linear-gradient(to bottom, #00b4db, #0083b0)
.degree
  top: -44px
.skyline
  flex:0 0 100px
  background: url(../assets/skyline.png)
  background-size: contain
  background-position: center bottom
</style>
```
