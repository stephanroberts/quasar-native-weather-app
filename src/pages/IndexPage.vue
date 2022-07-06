<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input @keyup.enter="getWeatherBySearch" v-model="data.search" placeholder="Search" dark borderless>
        <template v-slot:before>
          <q-icon @click="getLocation" name="my_location" />
        </template>
        <template v-slot:append>
          <q-btn @click="getWeatherBySearch" round dense flat icon="search" />
        </template>
      </q-input>
    </div>

    <template v-if="data.weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">
          {{ data.weatherData.name }}
        </div>
        <div class="text-h6 text-weight-light">
          {{ data.weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg  relative-position">
          <span>{{ Math.round(data.weatherData.main.temp) }}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
      </div>

      <div class="col text-center">
        <img :src="`http://openweathermap.org/img/wn/${data.weatherData.weather[0].icon}@2x.png`">
      </div>
    </template>
    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">
          Quasar<br>Weather
        </div>
        <q-btn @click="getLocation" class="col" flat>
          <q-icon left size="3em" name="my_location" />
          <div>Find my location</div>
        </q-btn>
      </div>
    </template>

    <div class="col skyline">

    </div>

  </q-page>
</template>

<script setup>
import { reactive, computed } from 'vue';
import { useQuasar } from 'quasar';
import axios from 'axios';

const data = reactive({
  search: '',
  weatherData: null,
  lat: null,
  long: null,
  apiKey: 'a8ed26007a5432ba287e1083a12dbd9d',
});

const bgClass = computed(() => {
  if (!data.weatherData) return 'bg-day';
  if (data.weatherData.weather[0].icon.endsWith('n')) {
    return 'bg-night';
  } else {
  }
  return 'bg-day';
});

const $q = useQuasar();

function getLocation() {
  $q.loading.show();
  navigator.geolocation.getCurrentPosition(position => {
    data.lat = position.coords.latitude;
    data.long = position.coords.longitude;
    getWeatherByCoords();
  })
}

function getWeatherByCoords() {
  $q.loading.show();
  axios.get(`http://api.openweathermap.org/data/2.5/weather?lat=${data.lat}&lon=${data.long}&limit=${1}&appid=${data.apiKey}&units=metric`).then(response => {
    data.weatherData = response.data;
    $q.loading.hide();
  })
}

function getWeatherBySearch() {
  $q.loading.show();
  if (data.search === '') return;
  axios.get(`http://api.openweathermap.org/data/2.5/weather?q=${data.search}&appid=${data.apiKey}&units=metric`).then(response => {
    data.weatherData = response.data;
    $q.loading.hide();
  })
  data.search = '';
}

</script>
<style lang="scss">
.q-page.bg-day {
  background: linear-gradient(to top, #43c6ac, #191654);
}

.bg-night.bg-night {
  background: linear-gradient(to bottom, #141e30, #243b55);
}

.degree {
  top: -44px;
}

.skyline {
  flex: 0 0 100px;
  background: url(/citySkyline.png);
  background-size: contain;
  background-position: center bottom;
}
</style>
