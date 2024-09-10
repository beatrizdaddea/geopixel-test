<template>
  <div class="container">
    <div class="main-container">
      <div class="form-container">
        <h2 class="title-content">Previsão</h2>
        <div class="state-container form-input">
          <label>Estado</label>
          <select v-model="stateSelected" @change="handleChangeEstado">
            <option value="">Selecione o estado</option>
            <option v-for="(state, index) in states" :key="index" :value="state.code">
              {{ state.name }}
            </option>
          </select>
        </div>
        <div v-if="stateSelected" class="city-container form-input">
          <label>Cidade</label>
          <select v-model="citySelected" @change="handleChangeCity">
            <option value="">Selecione a cidade</option>
            <option v-for="(city, index) in cities" :key="index" :value="city">
              {{ city }}
            </option>
          </select>
        </div>
        <div class="climate-container form-input">
          <label>Clima</label>
          <div class="climate-content">
            <input type="text" disabled :value="weatherData ? weatherData.description : '-'" />
            <img v-if="weatherData && weatherData.condition_slug"
              :src="`https://assets.hgbrasil.com/weather/icons/conditions/${weatherData.condition_slug}.svg`"
              :alt="weatherData.description">
          </div>
        </div>
        <div class="maximum-container form-input">
          <label>Máxima</label>
          <input type="text" disabled :value="weatherData ? `${weatherData.forecast[0].max}°C` : '-'" />
        </div>
        <div class="minimum-container form-input">
          <label>Mínima</label>
          <input type="text" disabled :value="weatherData ? `${weatherData.forecast[0].min}°C` : '-'" />
        </div>
        <div class="moon-container form-input">
          <label>Fase da Lua</label>
          <input type="text" disabled :value="weatherData ? weatherData.moon_phase : '-'" />
          <img v-if="weatherData && weatherData.moon_phase"
            :src="`https://assets.hgbrasil.com/weather/moon/${weatherData.moon_phase}.svg`"
            :alt="weatherData.moon_phase">
        </div>
        <div class="submit-btn">
          <button @click="handleFetchWeather">Consultar</button>
        </div>
      </div>
      <div class="map-section">
        <MapWrapper class="display-map" :coordinates="cityCoordinates" />
      </div>
    </div>

  </div>
  <div v-if="forecastData.length" class="forecast-section">
    <div class="forecast-container">
      <h2>Previsão dos Próximos 3 Dias</h2>
      <div v-for="(forecast, index) in forecastData" :key="index" class="forecast-item">
        <div class="forecast-date">{{ forecast.date }}</div>
        <div class="forecast-temperature">
          Máxima: {{ forecast.max }}°C
          Mínima: {{ forecast.min }}°C
        </div>
        <div class="forecast-condition">
          Condição: {{ forecast.description }}
        </div>
        <div class="forecast-chance-rain">
          Chance de chuva: {{ forecast.rain_probability }}%
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import axios from 'axios';
import MapWrapper from './MapWrapper.vue';

const stateSelected = ref<string>('');
const citySelected = ref<string>('');
const cities = ref<string[]>([]);
const weatherData = ref<any>(null);
const forecastData = ref<any[]>([]);
const cityCoordinates = ref<[number, number] | null>(null);
const states = ref<{ code: string, name: string }[]>([
  { code: 'AC', name: 'Acre' }, { code: 'AL', name: 'Alagoas' }, { code: 'AP', name: 'Amapá' },
  { code: 'AM', name: 'Amazonas' }, { code: 'BA', name: 'Bahia' }, { code: 'CE', name: 'Ceará' },
  { code: 'DF', name: 'Distrito Federal' }, { code: 'ES', name: 'Espírito Santo' }, { code: 'GO', name: 'Goiás' },
  { code: 'MA', name: 'Maranhão' }, { code: 'MT', name: 'Mato Grosso' }, { code: 'MS', name: 'Mato Grosso do Sul' },
  { code: 'MG', name: 'Minas Gerais' }, { code: 'PA', name: 'Pará' }, { code: 'PB', name: 'Paraíba' },
  { code: 'PR', name: 'Paraná' }, { code: 'PE', name: 'Pernambuco' }, { code: 'PI', name: 'Piauí' },
  { code: 'RJ', name: 'Rio de Janeiro' }, { code: 'RN', name: 'Rio Grande do Norte' }, { code: 'RS', name: 'Rio Grande do Sul' },
  { code: 'RO', name: 'Rondônia' }, { code: 'RR', name: 'Roraima' }, { code: 'SC', name: 'Santa Catarina' },
  { code: 'SP', name: 'São Paulo' }, { code: 'SE', name: 'Sergipe' }, { code: 'TO', name: 'Tocantins' }
]);

const handleChangeEstado = async (event: Event) => {
  const target = event.target as HTMLSelectElement;
  const state = target.value;
  stateSelected.value = state;

  const url = `https://servicodados.ibge.gov.br/api/v1/localidades/estados/${state}/municipios?orderBy=nome`;

  try {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error('Error to fetch cities');
    }
    const data: { nome: string }[] = await response.json();
    const namesCity: string[] = data.map(city => city.nome);
    cities.value = namesCity;
  } catch (err) {
    console.error(err);
    cities.value = [];
  }
};

const handleChangeCity = (event: Event) => {
  const target = event.target as HTMLSelectElement;
  const city = target.value;
  citySelected.value = city;
};

const handleFetchWeather = async () => {
  if (!citySelected.value || !stateSelected.value) {
    console.error('City or state not selected');
    return;
  }

  // Busca as coordenadas da cidade pela API de Geoencoding da OpenWeather
  const apiKey = '0b696f84c6de101cf0f2b3c8b1a666b3';
  const cityName = `${citySelected.value},BR`;
  const geoUrl = `http://api.openweathermap.org/geo/1.0/direct?q=${cityName}&limit=1&appid=${apiKey}`;

  try {
    const geoResponse = await axios.get(geoUrl);
    if (geoResponse.data.length > 0) {
      const { lat, lon } = geoResponse.data[0];
      cityCoordinates.value = [lon, lat];  // Atualiza as coordenadas
    } else {
      console.error('City not found');
    }

    // Buscar os dados de clima
    const weatherApiKey = "f0fc03f7"
    const weatherUrl = `https://api.hgbrasil.com/weather?key=${weatherApiKey}&city_name=${cityName}&format=json-cors`;
    const weatherResponse = await axios.get(weatherUrl);
    const data = weatherResponse.data.results;
    weatherData.value = {
      ...data,
      moon_phase: translateMoonPhase(data.moon_phase)
    };
    forecastData.value = data.forecast.slice(0, 3);  // Previsão para 3 dias
  } catch (error) {
    console.error('Error fetching weather or coordinates data:', error);

    console.log('Moon Phase:', weatherData.value.moon_phase);
    console.log('Image URL:', `https://assets.hgbrasil.com/weather/icons/moon/${weatherData.value.moon_phase}.png`);
  }
};

const translateMoonPhase = (phase: string): string => {
  const phases: { [key: string]: string } = {
    new: 'Lua nova',
    waxing_crescent: 'Lua crescente',
    first_quarter: 'Quarto crescente',
    waxing_gibbous: 'Gibosa crescente',
    full: 'Lua cheia',
    waning_gibbous: 'Gibosa minguante',
    last_quarter: 'Quarto minguante',
    waning_crescent: 'Lua minguante'
  };

  return phases[phase] || phase;
};

</script>

<style scoped>
.main-container {
  display: flex;
  justify-content: space-between;
  gap: 20px;
  flex-wrap: wrap;
  width: 100%;
}

.climate-container {
  display: flex;
  flex-direction: column;
  margin-bottom: 15px;

}

.climate-container label {
  margin-bottom: 5px;
}

.climate-content {
  align-items: center;
  display: flex;
  gap: 10px;
}

.climate-content input {
  flex: 1;
}

.climate-content img {
  height: auto;
  max-width: 32px;
}

.form-container {
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  padding: 1em;
  width: 25%;
}

.map-section {
  width: 60%;
}

.title-content {
  font-size: 24px;
  margin-bottom: 20px;
  color: #333;
  text-align: center;
}

.form-input {
  margin-bottom: 15px;
}

.form-input label {
  display: block;
  margin-bottom: 5px;
  font-weight: 600;
  color: #555;
}

.form-input select,
.form-input input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 16px;
  box-sizing: border-box;
}

.form-input img {
  display: inline-flex;
}

.form-input input[disabled] {
  background-color: #f5f5f5;
}

.submit-btn {
  text-align: center;
  margin-top: 20px;
}

.submit-btn button {
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 5px;
  padding: 12px 20px;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.3s, transform 0.2s;
}

.submit-btn button:hover {
  background-color: #0056b3;
}

.submit-btn button:active {
  transform: scale(0.98);
}

.forecast-section {
  background-color: #f8f9fa;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  margin-top: 20px;
}

.forecast-container {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.forecast-container h2 {
  font-size: 24px;
  color: #333;
  margin-bottom: 15px;
  text-align: center;
}

.forecast-item {
  background-color: #ffffff;
  border-radius: 8px;
  padding: 15px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.forecast-date {
  font-size: 18px;
  font-weight: bold;
  color: #333;
}

.forecast-temperature,
.forecast-condition,
.forecast-chance-rain {
  font-size: 16px;
  color: #555;
}

.forecast-temperature span,
.forecast-condition span,
.forecast-chance-rain span {
  font-weight: 600;
}

@media (max-width: 768px) {
  .main-container {
    display: flex;
    flex-direction: column;
  }

  .form-container {
    width: 100%;
  }

  .map-section {
    height: 300px;
    width: 100%;
  }

  .forecast-section {
    margin-top: 20px;
  }
}
</style>