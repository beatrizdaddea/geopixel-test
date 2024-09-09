<template>
  <div class="home-container">
    <div class="forecast-container">
      <RequestForm @city-change="setSelectedCity" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import RequestForm from '@/components/RequestForm.vue';

const selectedCity = ref<string>('');

const setSelectedCity = (city: string) => {
  selectedCity.value = city;
};
</script>

