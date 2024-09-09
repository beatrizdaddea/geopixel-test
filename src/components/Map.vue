<template>
  <div id="map" class="map-container" :style="{ width: '800px', height: '500px', border: '1px solid green' }"></div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue';
import axios from 'axios';
import Map from 'ol/Map';
import View from 'ol/View';
import TileLayer from 'ol/layer/Tile';
import VectorLayer from 'ol/layer/Vector';
import VectorSource from 'ol/source/Vector';
import { fromLonLat } from 'ol/proj';
import OSM from 'ol/source/OSM';
import Feature from 'ol/Feature';
import { Point } from 'ol/geom';
import Style from 'ol/style/Style';
import Icon from 'ol/style/Icon';

// Props
const props = defineProps<{
  cityName: string;
}>();

const coordinates = ref<[number, number] | null>(null);

// Fetch coordinates when cityName changes
const fetchCoordinates = async (cityName: string) => {
  try {
    const apiKey = ''; // Insert your OpenCageData API key here
    const response = await axios.get(`https://api.opencagedata.com/geocode/v1/json`, {
      params: {
        q: cityName,
        key: apiKey
      }
    });

    const { lat, lng } = response.data.results[0].geometry;
    coordinates.value = [lng, lat];
  } catch (error) {
    console.error('Error fetching coordinates:', error);
  }
};

// Watch for cityName prop change
watch(() => props.cityName, (newCityName) => {
  if (newCityName) {
    fetchCoordinates(newCityName);
  }
});

// Initialize map when coordinates change
onMounted(() => {
  if (coordinates.value) {
    const mapElement = document.getElementById('map');
    if (mapElement) {
      const pointFeature = new Feature({
        geometry: new Point(fromLonLat(coordinates.value)),
      });

      pointFeature.setStyle(new Style({
        image: new Icon({
          src: 'https://openlayers.org/en/latest/examples/data/icon.png',
          scale: 0.1
        })
      }));

      const initialFeaturesLayer = new VectorLayer({
        source: new VectorSource({
          features: [pointFeature]
        })
      });

      const initialMap = new Map({
        target: mapElement,
        layers: [
          new TileLayer({
            source: new OSM()
          }),
          initialFeaturesLayer
        ],
        view: new View({
          projection: 'EPSG:3857',
          center: fromLonLat(coordinates.value),
          zoom: 10
        }),
        controls: []
      });

      // Cleanup function
      return () => {
        initialMap.dispose();
      };
    }
  }
});
</script>

<style scoped>
/* Add your CSS here or import from an external stylesheet */
</style>
