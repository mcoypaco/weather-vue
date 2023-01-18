<template>
  <div>
    <ManageLocationsModal
      v-if="showLocationsForm"
      class="hidden xl:block"
    />
    <MainLayout v-if="!showLocationsForm && !showSettings" />
    <SettingsLayout
      v-if="showSettings"
      class="xl:hidden"
    />
    <ManageLocationsLayout
      v-if="showLocationsForm"
      class="xl:hidden"
    />
  </div>
</template>

<script>
import axios from 'axios'
import { mapStores } from 'pinia'

import ManageLocationsModal from './components/ManageLocationsModal.vue'
import MainLayout from './layouts/MainLayout.vue'
import ManageLocationsLayout from './layouts/ManageLocationsLayout.vue'
import SettingsLayout from './layouts/SettingsLayout.vue'
import main from './stores/main'

const openWeatherUrl = import.meta.env.VITE_OPEN_WEATHER_URL
const openWeatherApiKey = import.meta.env.VITE_OPEN_WEATHER_API_KEY

export default {
  components: {
    ManageLocationsModal,
    MainLayout,
    SettingsLayout,
    ManageLocationsLayout
  },
  data () {
    return {
      //
    }
  },
  computed: {
    ...mapStores(main),

    /**
     *
     *
     */
    showLocationsForm () {
      try {
        return this.mainStore.showLocationsForm
      } catch (error) {
        return false
      }
    },

    /**
     *
     *
     */
    showSettings () {
      try {
        return this.mainStore.showSettings
      } catch (error) {
        return false
      }
    },

    /**
     *
     *
     */
    currentLocation () {
      try {
        return this.mainStore.currentLocation
      } catch (error) {
        return null
      }
    },

    /**
     *
     *
     */
    units () {
      try {
        return this.mainStore.units
      } catch (error) {
        return null
      }
    }
  },
  watch: {
    /**
     *
     *
     */
    currentLocation () {
      try {
        this.getWeatherForecast(this.mainStore.currentLocation.lat, this.mainStore.currentLocation.lon, this.mainStore.units)
      } catch (error) {
        console.error(error)
      }
    },

    /**
     *
     *
     */
    units () {
      try {
        this.getWeatherForecast(this.mainStore.currentLocation.lat, this.mainStore.currentLocation.lon, this.mainStore.units)
      } catch (error) {
        console.error(error)
      }
    }
  },
  created () {
    this.mainStore.$patch({ loading: true })

    this.setDefaultUnits()

    navigator.geolocation.getCurrentPosition(async (position) => {
      // Sets current latitude and longitude to pinia.
      this.mainStore.$patch({
        currentLatitude: position.coords.latitude,
        currentLongitude: position.coords.longitude
      })

      // Get the city from the latitude and longitude.
      const location = await axios.get(`${openWeatherUrl}/geo/1.0/reverse`, {
        params: {
          lat: position.coords.latitude,
          lon: position.coords.longitude,
          appid: openWeatherApiKey
        }
      }).then(({ data }) => data)

      // Store the current location.
      if (location) {
        this.storeLocation(location)
      }
    }, () => {
      let locations

      try {
        locations = JSON.parse(localStorage.getItem('locations')) || []
      } catch (error) {
        locations = []
      }

      this.mainStore.$patch({
        locations,
        showLocationsForm: true
      })
    })
  },
  methods: {
    /**
     *
     *
     */
    setDefaultUnits () {
      try {
        const units = localStorage.getItem('units')

        if (!units) {
          localStorage.setItem('units', 'metric')

          this.mainStore.$patch({ units: 'metric' })
        } else {
          this.mainStore.$patch({ units })
        }
      } catch (error) {
        console.error(error)
      }
    },

    /**
     *
     *
     */
    storeLocation (location) {
      let locations

      try {
        locations = JSON.parse(localStorage.getItem('locations')) || []
      } catch (error) {
        locations = []
      }

      const exists = locations.filter(item => {
        return item.name === location[0].name
      })

      if (!exists.length) locations.push(location[0])

      // Store to localstorage.
      localStorage.setItem('locations', JSON.stringify(locations))

      // Update the store.
      this.mainStore.$patch({
        currentLocation: location[0],
        locations
      })
    },

    /**
     *
     * @param string lat
     * @param string lon
     * @param units lon
     */
    async getWeatherForecast (lat, lon, units) {
      this.mainStore.$patch({ loading: true })

      const responses = await Promise.all([
        this.currentWeather(lat, lon, units),
        this.forecast(lat, lon, units),
        this.oneCall(lat, lon, units)
      ]).catch(() => {
        alert('Something went wrong!')
      })

      this.mainStore.$patch({
        currentWeather: responses[0],
        forecast: responses[1],
        oneCall: responses[2],
        loading: false
      })
    },

    /**
     *
     *
     */
    oneCall (lat, lon, units) {
      return axios.get(`${openWeatherUrl}/data/2.5/onecall`, {
        params: {
          lat,
          lon,
          exclude: 'alerts',
          units,
          appid: openWeatherApiKey
        }
      }).then(({ data }) => data)
    },

    /**
     *
     *
     */
    currentWeather (lat, lon, units) {
      return axios.get(`${openWeatherUrl}/data/2.5/weather`, {
        params: {
          lat,
          lon,
          units,
          appid: openWeatherApiKey
        }
      }).then(({ data }) => data)
    },

    /**
     *
     *
     */
    forecast (lat, lon, units) {
      return axios.get(`${openWeatherUrl}/data/2.5/forecast`, {
        params: {
          lat,
          lon,
          units,
          appid: openWeatherApiKey
        }
      }).then(({ data }) => data)
    }
  }
}
</script>
