<template>
  <v-app>
    <v-app-bar>
      <template v-slot:image>
        <v-img
          gradient="to top right, rgba(19, 84, 122, .8), rgba(128, 208, 199, .8)"
        ></v-img>
      </template>
      <div class=" d-flex align-center flex-grow-1 mx-auto" style="max-width: 960px">
        <v-app-bar-nav-icon>
          <v-icon
            icon="mdi-weather-partly-cloudy"
            color="black"
          ></v-icon>
        </v-app-bar-nav-icon>
        <v-app-bar-title>Weather app {{ userCity }}, {{ userCountry }} {{ currentTemp }}°</v-app-bar-title>
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
    <v-main>
      <v-container class="fill-height justify-center">
        <h1>hi</h1>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import axios from 'axios';
export default {
    data() {
      return {
          citiesFormated: [],
          selectedCity: undefined,
          userCity: undefined,
          userCountry: undefined,
          weatherData: undefined,
          currentTemp: undefined
      }
    },
    mounted() {
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
        this.getWeather({latitude: userLocation.latitude, longitude: userLocation.longitude});
      }
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
            .get(`https://localhost:32768/WeatherForecast?Latitude=${coords.latitude}&Longitude=${coords.longitude}&ForecastDays=1`)
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
      }
    },
    watch: {
      weatherData(value) {
        const currentHour = new Date().getHours();
        this.currentTemp = value.temperatureC[currentHour];
      }
    }
}
</script>
