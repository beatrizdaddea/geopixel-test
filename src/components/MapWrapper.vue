<template>
  <div ref="mapContainer" class="map"></div>
</template>

<script setup lang="ts">
import { onMounted, ref, watch } from 'vue';
import 'ol/ol.css';
import { Map, View } from 'ol';
import TileLayer from 'ol/layer/Tile';
import OSM from 'ol/source/OSM';
import { fromLonLat } from 'ol/proj';

const props = defineProps<{
  coordinates: [number, number] | null;
}>();

const map = ref<Map | null>(null);
const mapContainer = ref<HTMLElement | null>(null);

onMounted(() => {
  map.value = new Map({
    target: mapContainer.value as HTMLElement,
    layers: [
      new TileLayer({
        source: new OSM()
      })
    ],
    view: new View({
      center: fromLonLat([-51.9253, -14.235]), // Coordenadas do Brasil
      zoom: 4
    })
  });
});

watch(
  () => props.coordinates,
  (newCoordinates) => {
    if (newCoordinates && map.value) {
      const view = map.value.getView();
      const lonLatCoords = fromLonLat(newCoordinates);
      view.setCenter(lonLatCoords);
      view.setZoom(10);
    }
  }
);
</script>

<style scoped>
.map {
  height: 100%;
  width: 100%;
}

</style>
