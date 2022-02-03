<script setup lang="ts"></script>

<template>
  <div class="loading" v-if="state.loading">読込中...</div>
  <div ref="map" class="map"/>
</template>

<script lang="ts">
import {Loader} from "@googlemaps/js-api-loader"
import {onMounted, reactive, ref} from 'vue'

const loader = new Loader({
  apiKey: import.meta.env.VITE_APP_MAP_API_KEY,
  version: "weekly",
  libraries: ["places"]
})

interface State {
  loading: boolean
}

export default {
  setup() {
    const map = ref(null)
    const state = reactive<State>({
      loading: true
    })

    const success = (position) => {

      const posLatLng = {
        lat: position.coords.latitude,
        lng: position.coords.longitude
      }

      const mapOptions = {
        center: posLatLng,
        zoom: 16
      };

      loader.load()
          .then((google) => {
            const googleMap = new google.maps.Map(map.value, mapOptions)

            new google.maps.Marker({
              position: posLatLng,
              map: googleMap
            })

            const overlay = new google.maps.OverlayView()
            overlay.onAdd = function () {
              const panes = this.getPanes()

              const svgWrapper = document.createElement('div')
              svgWrapper.setAttribute('class', 'overlay-wrapper')
              panes.markerLayer.appendChild(svgWrapper)

              const svg = document.createElement('img')
              svg.setAttribute('class', 'overlay')
              svg.setAttribute('src', '/compass.svg')
              svgWrapper.appendChild(svg)
            }
            overlay.setMap(googleMap)
            state.loading = false
          })
          .catch(e => {
            console.error(e)
          })
    }

    const error = (error) => {
      switch (error.code) {
        case 1: //PERMISSION_DENIED
          alert('位置情報の利用が許可されていません')
          break
        case 2: //POSITION_UNAVAILABLE
          alert('現在位置が取得できませんでした')
          break
        case 3: //TIMEOUT
          alert('タイムアウトになりました')
          break
        default:
          alert('現在位置が取得できませんでした')
          break
      }
    }

    const getLocation = () => {
      if (!navigator.geolocation) {
        alert('お使いのブラウザでは現在地情報を利用できません')
        return
      }

      const options = {
        enableHighAccuracy: false,
        timeout: 5000,
        maximumAge: 0
      }
      navigator.geolocation.getCurrentPosition(success, error, options)
    }

    onMounted(() => {
      if (navigator.permissions && navigator.permissions.query) {
        navigator.permissions.query({name: 'geolocation'}).then(function (result) {
          const permission = result.state
          if (permission === 'granted' || permission === 'prompt') {
            getLocation()
          } else {
            alert('お使いのブラウザでは現在地情報を利用できません')
          }
        })
      } else {
        getLocation()
      }
    })

    return {
      state,
      map
    }
  },
}
</script>

<style lang="scss">
.loading {
  margin-top: 50px;
}

.overlay-wrapper {
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);

  .overlay {
    max-width: 90vw;
  }
}

.map {
  width: 100vw;
  height: 100vh;
}
</style>
