<template>
  <v-app>
    <v-app-bar>
      <template v-slot:image>
        <v-img
          :gradient="appBarGradient"
        ></v-img>
      </template>
      <div class=" d-flex align-center flex-grow-1 mx-auto" style="max-width: 960px">
        <v-icon
          :icon="weatherIcon"
          :color="weatherColor"
        ></v-icon>
        <v-app-bar-title class="ml-2">
          Weather app 
          <span class="ml-2 text-h6">
            {{ actualUserCity }}, {{ actualUserCountry }} {{ actualCurrentTemp }}°C
          </span>
        </v-app-bar-title>
        <span class="text-h6 mr-4">
          {{ currentTime }}
        </span>
        <v-autocomplete
          style="max-width: 256px;"
          clearable
          density="compact"
          hide-details
          placeholder="Search city..."
          :items="citiesFormated"
          item-title="name"
          item-value="coordinates"
          v-model:model-value="selectedCity"
          @update:search="getCities"
          @update:model-value="getWeather"
        ></v-autocomplete>
      </div>
    </v-app-bar>
    <v-main :style="mainGradient">
      <v-container class="fill-height">
        <v-card
          class="mx-auto"
          width="512"
          max-height="512"
        >
          <v-card-item :title="`${userCity}, ${userCountry} | ${dayNames[(selectedDay + currentDay) % 7]} ${selectedHour.toString().padStart(2, 0)}:${currentMinutes}`">
            <template v-slot:subtitle>
              <v-icon
                :icon="weatherIcon"
                size="18"
                :color="weatherColor"
                class="me-1 pb-1"
              ></v-icon>

              {{ currentWeatherDescription }}
            </template>
          </v-card-item>

          <v-card-text class="py-0">
            <v-row align="center" no-gutters>
              <v-col
                class="text-h2"
                cols="6"
              >
                {{ currentTemp }}&deg;C
              </v-col>

              <v-col cols="6" class="text-right">
                <v-icon
                  :color="weatherColor"
                  :icon="weatherIcon"
                  size="88"
                ></v-icon>
              </v-col>
            </v-row>
            <v-row class="mt-0">
              <v-list-item
              density="compact"
              >
              <v-list-item-subtitle>Feels like: {{ currentApparentTemp }}&deg;C</v-list-item-subtitle>
              </v-list-item>
            </v-row>
          </v-card-text>

          <div class="d-flex mt-5 justify-space-between">
            <v-list-item
              prepend-icon="mdi-weather-windy"
            >
              <v-list-item-subtitle>{{ currentWindSpeed }}km/h</v-list-item-subtitle>
            </v-list-item>

            <v-list-item
              prepend-icon="mdi-compass"
            >
              <v-list-item-subtitle>{{ currentWindDirection }}&deg;</v-list-item-subtitle>
            </v-list-item>

            <v-list-item
              prepend-icon="mdi-weather-pouring"
            >
              <v-list-item-subtitle>{{ currentPrecipitationProb }}%</v-list-item-subtitle>
            </v-list-item>

            <v-list-item
              prepend-icon="mdi-water-percent"
            >
              <v-list-item-subtitle>{{ currentRelativeHumidity }}%</v-list-item-subtitle>
            </v-list-item>
          </div>
          <v-list-item class="justify-center">
            <v-list-item-subtitle>Surface pressure: {{ currentSurfacePressure }}hPa</v-list-item-subtitle>
          </v-list-item>

          <v-expand-transition>
            <div v-if="expand">
              <div class="py-3">
                <span class="text-caption ml-4">Move the slider to select a different hour</span>
                <v-slider
                  v-model="selectedHour"
                  :max="23"
                  :step="1"
                  :ticks="hourLabels"
                  tick-size="4"
                  class="mx-4"
                  color="primary"
                  density="compact"
                  hide-details
                  show-ticks="always"
                  thumb-size="10"
                  @update:model-value="updateAllInfo"
                ></v-slider>
              </div>

              <div class="d-flex flex-column align-center justify-center bg-grey-lighten-4 pa-1">
                <v-btn-toggle
                  v-model="selectedDay"
                  color="primary"
                  mandatory
                  @update:model-value="updateAllInfo"
                >
                  <v-btn :value="0">{{dayNames[new Date().getDay()]}}</v-btn>
                  <v-btn :value="1">{{dayNames[(new Date().getDay() + 1) % 7]}}</v-btn>
                  <v-btn :value="2">{{dayNames[(new Date().getDay() + 2) % 7]}}</v-btn>
                  <v-btn :value="3">{{dayNames[(new Date().getDay() + 3) % 7]}}</v-btn>
                  <v-btn :value="4">{{dayNames[(new Date().getDay() + 4) % 7]}}</v-btn>
                  <v-btn :value="5">{{dayNames[(new Date().getDay() + 5) % 7]}}</v-btn>
                  <v-btn :value="6">{{dayNames[(new Date().getDay() + 6) % 7]}}</v-btn>
                </v-btn-toggle>
              </div>
            </div>
          </v-expand-transition>

          <v-divider></v-divider>

          <v-card-actions>
            <v-btn @click="expand = !expand">
              {{ !expand ? 'More info' : 'Less info' }}
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import axios from 'axios';
import { weatherDescription } from './constant.js';
export default {
    data() {
      return {
          citiesFormated: [],
          selectedCity: undefined,
          userCity: undefined,
          userCountry: undefined,
          actualUserCity: undefined,
          actualUserCountry: undefined,
          actualCurrentTemp: undefined,
          weatherData: undefined,
          currentTemp: undefined,
          currentApparentTemp: undefined,
          currentWindSpeed: undefined,
          currentWindDirection: undefined,
          currentPrecipitationProb: undefined,
          currentSurfacePressure: undefined,
          currentRelativeHumidity: undefined,
          currentWeatherCode: undefined,
          currentWeatherDescription: undefined,
          mainGradient: undefined,
          appBarGradient: undefined,
          weatherIcon: undefined,
          weatherColor: undefined,
          expand: undefined,
          weekLabels: {},
          selectedDay: 0, // not related to 0 - Sunday; 6 - Saturday
          currentDay: new Date().getDay(),
          selectedHour: new Date().getHours(),
          currentTime: new Date().toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit', hour12: false }),
          currentMinutes: new Date().getMinutes(),
          dayNames: ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"]
      }
    },
    mounted() {
      console.log(new Date().getDay())
      const storedLocation = localStorage.getItem('userLocation');

      if (!storedLocation) {
        const options = {
          enableHighAccuracy: true,
          timeout: 7500,
          maximumAge: 60000,
        };
        navigator.geolocation.getCurrentPosition(this.saveLocation, null, options);
      } else {
        const userLocation = JSON.parse(storedLocation);
        this.userCity = userLocation.city;
        this.userCountry = userLocation.country;
        this.actualUserCity = userLocation.city;
        this.actualUserCountry = userLocation.country;
        this.getWeather({latitude: userLocation.latitude, longitude: userLocation.longitude});
      }

      const currentDay = new Date().getDay();
      for (let i = 0; i < this.dayNames.length; i++) {
        const dayIndex = (currentDay + i) % 7;
        this.weekLabels[i] = this.dayNames[dayIndex];
      }

      setInterval(() => {
        this.currentTime = new Date().toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit', hour12: false });
        this.currentMinutes = new Date().getMinutes();
      }, 60000);
    },
    methods: {
      saveLocation(position) {
        const coords = {
          latitude: position.coords.latitude.toFixed(5),
          longitude: position.coords.longitude.toFixed(5)
        };
        axios.get(`https://localhost:32768/WeatherForecast/getReverseLocation?Latitude=${coords.latitude}&Longitude=${coords.longitude}`)
        .then(result => {
          console.log(result);
          this.userCity = result.data[0].name;
          this.userCountry = result.data[0].country;
          this.actualUserCity = result.data[0].name;
          this.actualUserCountry = result.data[0].country;
          // cache in local storage
          const userLocation = {
            latitude: coords.latitude,
            longitude: coords.longitude,
            city: this.userCity,
            country: this.userCountry
          }
          localStorage.setItem('userLocation', JSON.stringify(userLocation));
        })
        .catch(err => {
          console.error(err);
        });
        
        this.getWeather(coords);
      },
      getCities(value) {
        if (value && value.length >= 2 && !value.includes(',')) {
          axios
            .get('https://localhost:32768/WeatherForecast/getLocation?cityName=' + value)
            .then(result => {
              console.log('api request');
              console.log(result);
              this.citiesFormated = [];
              result.data.forEach(city => {
                this.citiesFormated.push({
                  name: city.name + ', ' + city.country,
                  coordinates: {
                      latitude: city.latitude,
                      longitude: city.longitude
                  }
                });
              })
            })
            .catch(err => console.log(err));
        }
      },
      getWeather(coords) {
        if (coords) {
          axios
            .get(`https://localhost:32768/WeatherForecast?Latitude=${coords.latitude}&Longitude=${coords.longitude}&ForecastDays=7`)
            .then(result => {
              console.log('weather api request');
              console.log(result);
              this.weatherData = result.data;
            })
            .catch(err => console.log(err));
            
            const cityAndCountry = this.citiesFormated.find(obj =>
              obj.coordinates.latitude === this.selectedCity.latitude &&
              obj.coordinates.longitude === this.selectedCity.longitude
            );
            if (cityAndCountry) {
              const name = cityAndCountry.name;
              const [city, country] = name.split(',');

              this.userCity = city.trim();
              this.userCountry = country.trim();
            }
        }
      },
      updateAllInfo() {
        this.currentTemp = this.weatherData.temperatureC[this.selectedHour + this.selectedDay * 24];
        this.currentApparentTemp = this.weatherData.apparentTemperatureC[this.selectedHour + this.selectedDay * 24];
        this.currentPrecipitationProb = this.weatherData.precipitationProbability[this.selectedHour + this.selectedDay * 24];
        this.currentWindSpeed = this.weatherData.windSpeed[this.selectedHour + this.selectedDay * 24];
        this.currentWindDirection = this.weatherData.windDirection[this.selectedHour + this.selectedDay * 24];
        this.currentRelativeHumidity = this.weatherData.relativeHumidity[this.selectedHour + this.selectedDay * 24];
        this.currentSurfacePressure = this.weatherData.surfacePressure[this.selectedHour + this.selectedDay * 24];
        this.currentWeatherCode = this.weatherData.weatherCode[this.selectedHour + this.selectedDay * 24];
        this.currentWeatherDescription = weatherDescription[this.weatherData.weatherCode[this.selectedHour + this.selectedDay * 24]];
      }
    },
    computed: {
      adaptiveWeatherIcon() {
        if(this.currentTemp < 13) {
          if(this.currentWeatherCode === 2 || this.currentWeatherCode === 3) {
            return "mdi-weather-snowy";
          } else if (this.currentWeatherCode >= 51) {
            return "mdi-weather-snowy-rainy"
          } else {
            return "mdi-sun-snowflake-variant"
          }
        } else if (this.currentTemp >= 13 && this.currentTemp <= 25) {
          if(this.currentWeatherCode === 2 || this.currentWeatherCode === 3) {
            return "mdi-weather-cloudy";
          } else if (this.currentWeatherCode >= 51) {
            return "mdi-weather-pouring"
          } else {
            return "mdi-weather-sunny"
          }
        } else {
          if(this.currentWeatherCode === 2 || this.currentWeatherCode === 3) {
            return "mdi-weather-partly-cloudy";
          } else if (this.currentWeatherCode >= 51) {
            return "mdi-weather-lightning-rainy"
          } else {
            return "mdi-weather-sunny-alert"
          }
        }
      },
      hourLabels() {
        return Array.from({ length: 24 }, (_, i) => (i === this.selectedHour ? i : ''));
      }
    },
    watch: {
      weatherData(value) {
        const currentHour = new Date().getHours();
        if(!this.actualCurrentTemp) {
          this.actualCurrentTemp = value.temperatureC[currentHour];
        }
        this.currentTemp = value.temperatureC[currentHour];
        this.currentApparentTemp = value.apparentTemperatureC[currentHour];
        this.currentPrecipitationProb = value.precipitationProbability[currentHour];
        this.currentWindSpeed = value.windSpeed[currentHour];
        this.currentWindDirection = value.windDirection[currentHour];
        this.currentRelativeHumidity = value.relativeHumidity[currentHour];
        this.currentSurfacePressure = value.surfacePressure[currentHour];
        this.currentWeatherCode = value.weatherCode[currentHour];
        this.currentWeatherDescription = weatherDescription[value.weatherCode[currentHour]];
      },
      currentTemp(value) {
          if(value < 13) {
            this.appBarGradient = "to top right, rgba(0, 84, 130, 0.3), rgba(142, 213, 250, 0.3)";
            this.mainGradient = "background: linear-gradient(to bottom, rgba(166, 215, 252, 0.3), rgba(207, 233, 252, 0.3))";
            this.weatherIcon = this.adaptiveWeatherIcon;
            this.weatherColor = "cyan-lighten-4";
          } else if (value >= 13 && value <= 25) {
            this.appBarGradient = "to top right, rgba(255, 187, 0, 0.3), rgba(255, 235, 153, 0.3)";
            this.mainGradient = "background: linear-gradient(to bottom, rgba(255, 221, 128, 0.3), rgba(255, 241, 178, 0.3))";
            this.weatherIcon = this.adaptiveWeatherIcon;
            this.weatherColor = "yellow-darken-1";
          } else {
            this.appBarGradient = "to top right, rgba(204, 0, 0, 0.3), rgba(255, 102, 102, 0.3)";
            this.mainGradient = "background: linear-gradient(to bottom, rgba(255, 138, 128, 0.3), rgba(255, 205, 198, 0.3))";
            this.weatherIcon = this.adaptiveWeatherIcon;
            this.weatherColor = "orange-darken-4";
          }
      }
    }
}
</script>