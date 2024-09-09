<template>
  <div class="container">
    <h2 class="title-content">Previsão</h2>
    <div class="form-container">
      <div class="state-container form-input">
        <label>Estado</label>
        <select v-model="stateSelected" @change="handleChangeEstado">
          <option value="">Selecione o estado</option>
          <option value="AC">Acre</option>
          <option value="AL">Alagoas</option>
          <option value="AP">Amapá</option>
          <option value="AM">Amazonas</option>
          <option value="BA">Bahia</option>
          <option value="CE">Ceará</option>
          <option value="DF">Distrito Federal</option>
          <option value="ES">Espírito Santo</option>
          <option value="GO">Goiás</option>
          <option value="MA">Maranhão</option>
          <option value="MT">Mato Grosso</option>
          <option value="MS">Mato Grosso do Sul</option>
          <option value="MG">Minas Gerais</option>
          <option value="PA">Pará</option>
          <option value="PB">Paraíba</option>
          <option value="PR">Paraná</option>
          <option value="PE">Pernambuco</option>
          <option value="PI">Piauí</option>
          <option value="RJ">Rio de Janeiro</option>
          <option value="RN">Rio Grande do Norte</option>
          <option value="RS">Rio Grande do Sul</option>
          <option value="RO">Rondônia</option>
          <option value="RR">Roraima</option>
          <option value="SC">Santa Catarina</option>
          <option value="SP">São Paulo</option>
          <option value="SE">Sergipe</option>
          <option value="TO">Tocantins</option>
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
        <input type="text" disabled :value="weatherData ? weatherData.description : '-'"/>
      </div>
      <div class="maximum-container form-input">
        <label>Máxima</label>
        <input type="text" disabled :value="weatherData ? `${weatherData.forecast[0].max}°C` : '-'"/>
      </div>
      <div class="minimum-container form-input">
        <label>Mínima</label>
        <input type="text" disabled :value="weatherData ? `${weatherData.forecast[0].min}°C` : '-'"/>
      </div>
      <div class="moon-container form-input">
        <label>Fase da Lua</label>
        <input type="text" disabled :value="weatherData ? weatherData.moon_phase : '-'"/>
      </div>
      <div class="submit-btn">
        <button @click="handleFetchWeather">Consultar</button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import axios from 'axios';

const stateSelected = ref<string>('');
const citySelected = ref<string>('');
const cities = ref<string[]>([]);
const weatherData = ref<any>(null);

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
  // Emit city change event if needed
  // emit('cityChange', city);
};

const handleFetchWeather = async () => {
  if (!citySelected.value || !stateSelected.value) {
    console.error('City or state not selected');
    return;
  }

  const apiKey = '';
  const cityName = `${citySelected.value}`;
  const url = `https://api.hgbrasil.com/weather?key=${apiKey}&city_name=${cityName}&format=json-cors`;

  try {
    const response = await axios.get(url);
    const data = response.data.results;
    console.log('Weather data:', data);
    const translatedPhase = translateMoonPhase(response.data.results.moon_phase);
    weatherData.value = {
      ...data,
      moon_phase: translatedPhase
    };
  } catch (error) {
    console.error('Error fetching weather data:', error);
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
/* Adicione o CSS aqui, ou importe um arquivo CSS externo */
</style>


<style scoped>
.container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin: 0 auto;
  max-width: 600px;
  padding: 20px;
  width: 100%;
}

.form-container {
  background-color: #f9f9f9;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
}

.form-input {
  display: flex;
  flex-direction: column;
  margin-bottom: 20px;
}

.form-input label {
  font-size: 16px;
  margin-bottom: 8px;
}

.form-input select,
.form-input input {
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 10px;
  width: 100%;
}

.submit-btn button {
  background-color: #007BFF;
  border: none;
  border-radius: 4px;
  color: #fff;
  cursor: pointer;
  padding: 10px;
  text-align: center;
  width: 100%;
}

.submit-btn button:hover {
  background-color: #0056b3;
}

.title-content {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 20px;
  text-align: center;
}

/* Estilos responsivos */
@media (max-width: 768px) {
  .container {
    padding: 10px;
  }

  .form-input label {
    font-size: 14px;
  }

  .submit-btn button {
    padding: 12px;
  }
}

</style>
