<template>
  <div class="home-container">
    <div class="forecast-container">
      <Form @city-change="setSelectedCity" />
    </div>
    <div class="map-container">
      <Map :city-name="selectedCity" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import Form from '@/components/Form.vue';
import Map from '@/components/Map.vue';

const selectedCity = ref<string>('');

const setSelectedCity = (city: string) => {
  selectedCity.value = city;
};
</script>

<style scoped>
.home-container {
  /* Adicione seus estilos aqui */
}
</style>
