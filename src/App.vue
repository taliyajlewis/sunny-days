<template>
  <div id="SunnyDays" :class="weather && weather.current.is_day ? 'day' : 'night'">
    <h1>SunnyDays</h1>

    <input v-model="city" placeholder="Search your city here" />

    <button @click="UpdateWeather(city)">Search</button>
    <button @click="saveLocation">Save location</button>

  <div v-if="weather" class="top-section">

    <div class="weather-box">
      <h2>{{ weather.location.name }}</h2>
      <p>{{ weather.location.region }}, {{ weather.location.country }}</p>
      <p>{{ weather.current.condition.text }}</p>
      <p>{{ weather.current.temp_f }}F</p>
      <p>High: {{ weather.forecast.forecastday[0].day.maxtemp_f }}F</p>
      <p>Low: {{ weather.forecast.forecastday[0].day.mintemp_f }}F</p>
    </div>

    <div class="chart-box">
      <h3>Hourly Forecast (Today)</h3>
      <canvas id="weatherChart"></canvas>
  </div>
  
  </div>

    <div v-if="weather">
      <h3>5 Day Forecast</h3>

      <table>
        <tr>
          <th>Date</th>
          <th>Conditions</th>
          <th>High</th>
          <th>Low</th>
        </tr>

        <tr v-for="day in weather.forecast.forecastday" :key="day.date">
          <td>{{ day.date }}</td>
          <td>{{ day.day.condition.text }}</td>
          <td>{{ day.day.maxtemp_f }}F</td>
          <td>{{ day.day.mintemp_f }}F</td>
        </tr>
      </table>
    </div>

    <div v-if="locations.length">
      <h3>Saved Locations</h3>
      <ul>
        <li v-for="where in locations" :key="where">
          <span @click="UpdateWeather(where)">{{ where }}</span>
          <button @click="locations = locations.filter(l => l !== where)">Delete</button>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import Chart from "chart.js/auto"

export default {
  name: "SunnyDays",
  data() {
    return {
      city: "Troy",
      weather: null,
      locations: [],
      chart: null
    }
  },

  methods: {
    async UpdateWeather(cityName) {
      const key = "5adbfaae748240e0b52205124262804"
      const response = await fetch(`https://api.weatherapi.com/v1/forecast.json?key=${key}&q=${cityName || this.city}&days=5`)
      const data = await response.json()
      this.weather = data
      this.city = cityName || this.city
      this.$nextTick(() => this.createChart())
    },

    saveLocation() {
      if (!this.city) return
      if (!this.locations.includes(this.city)) this.locations.push(this.city)
    },

    createChart() {
      const ctx = document.getElementById("weatherChart")
      const hours = this.weather.forecast.forecastday[0].hour
      const labels = hours.map(h => h.time.split(" ")[1])
      const temps = hours.map(h => h.temp_f)
      const rain = hours.map(h => h.chance_of_rain)

      if (this.chart) this.chart.destroy()

      this.chart = new Chart(ctx, {
        type: "line",
        data: {
          labels,
          datasets: [
            { label: "Temperature (F)", data: temps },
            { label: "Rain Chance (%)", data: rain }
          ]
        }
      })
    }
  },

  mounted() {
    this.UpdateWeather(this.city)
  }
}
</script>

<style>
#SunnyDays {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

input {
  padding: 10px;
  width: 250px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

button {
  padding: 10px 12px;
  margin: 5px;
  border: none;
  border-radius: 5px;
  background: #2c3e50;
  color: white;
  cursor: pointer;
}

button:hover {
  opacity: 0.85;
}

#SunnyDays > div:nth-of-type(1) {
  border: 1px solid #ccc;
  width: 300px;
  margin: 20px auto;
  padding: 10px;
}

#SunnyDays > div:nth-of-type(2) {
  display: flex;
  justify-content: center;
  gap: 10px;
  flex-wrap: wrap;
}

#SunnyDays > div:nth-of-type(2) > div {
  border: 1px solid #ccc;
  width: 120px;
  padding: 10px;
}

canvas {
  width: 300px !important;
  height: 150px !important;
  margin: 20px auto;
  display: block;
}

ul {
  padding: 0;
}

li {
  list-style: none;
  margin: 5px 0;
}

table {
  margin: 15px auto;
  border-collapse: collapse;
  width: 60%;
  font-size: 14px;
}

th, td {
  border: 1px solid #ccc;
  padding: 6px 8px;
  text-align: center;
}

th {
  background: #2c3e50;
  color: white;
}

h3 {
  margin-bottom: 10px;
}

.weather-box h2 {
  margin-bottom: 5px;
}

.weather-box p {
  margin: 4px 0;
}

#SunnyDays h3 {
  width: 100%;
  text-align: center;
  display: block;
  margin: 20px auto 10px auto;
}

#SunnyDays {
  max-width: 1000px;
  margin: 40px auto;
  padding: 20px;
}

.weather-box,
table,
.saved-box {
  background: white;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0,0,0,.08);
}

</style>