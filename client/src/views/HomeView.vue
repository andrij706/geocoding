<script setup>
import leaflet from 'leaflet'
import { onMounted, ref } from 'vue';
import GeoErrorModal from '../components/GeoErrorModal.vue'
import MapFeatures from '../components/MapFeatures.vue';

let map
onMounted(() => {
  //init map
  map = leaflet.map('map').setView([49.8397, 24.0297], 13);

  //add tile layer
  leaflet.tileLayer(`https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=${import.meta.env.VITE_APP_API_KEY}`,
          {
            maxZoom: 18,
            id: "mapbox/streets-v11",
            tileSize: 512,
            zoomOffset: -1,
            accessToken: import.meta.env.VUE_APP_API_KEY,
          }
        ).addTo(map);

        map.on('moveend', () => {
          closeSearchResults()
        })

        getGeolocation()
})

const coords = ref(null)
const fetchCoords = ref(null)
const geoMarker = ref(null)
const geoError = ref(null)
const geoErrorMsg = ref(null)

const getGeolocation = () => {
  if (coords.value){
    coords.value = null
    sessionStorage.removeItem('coords')
    map.removeLayer(geoMarker.value)
    return
  }
  //check session storage for coords
  if(sessionStorage.getItem('coords')){
    coords.value = JSON.parse(sessionStorage.getItem('coords'))
    plotGeolocation(coords.value)
    return
  }
  fetchCoords.value = true
  navigator.geolocation.getCurrentPosition(setCoords, getLockError)
}

const setCoords = (pos) => {
  //stop fetching coords
  fetchCoords.value = null

  const setSessionCords = {
    lat: pos.coords.latitude,
    lng: pos.coords.longitude
  }

  sessionStorage.setItem('coords', JSON.stringify(setSessionCords))

  //set ref coords value
  coords.value = setSessionCords

  plotGeolocation(coords.value)
}

const getLockError = (err) => {
  fetchCoords.value = null
  geoError.value = true
  geoErrorMsg.value = err.message
}

const closeGeoError = () => {
  geoError.value = null
  geoErrorMsg.value = null
}

const plotGeolocation = (coords) => {
  //create custom marker
  const customMarker = leaflet.icon({
    iconUrl: './src/assets/map-marker-red.svg',
    iconSize: [35, 35]
  })

  //create new marker with coords and custom icon
  geoMarker.value = leaflet.marker([coords.lat, coords.lng], {icon: customMarker}).addTo(map)

  map.setView([coords.lat, coords.lng], 10)
}

const resultMarker = ref(null)
const plotResult = (coords) => {
  //Check to see if resultMarker has value
  if(resultMarker.value){
    map.removeLayer(resultMarker.value)
  }
    //create custom marker
    const customMarker = leaflet.icon({
    iconUrl: './src/assets/map-marker-blue.svg',
    iconSize: [35, 35]
  })

  //create new marker with coords and custom icon
  resultMarker.value = leaflet.marker([coords.coordinates[1], coords.coordinates[0]], {icon: customMarker}).addTo(map)

  map.setView([coords.coordinates[1], coords.coordinates[0]], 14)

  closeSearchResults()
}

const searchResults = ref(null)

const toggleSearchResults = () => {
  searchResults.value = !searchResults.value
}

const closeSearchResults = () => {
  searchResults.value =null
}

const removeResult = () => {
  map.removeLayer(resultMarker.value)
}

</script>

<template>
  <div class="h-screen relative">
    <GeoErrorModal 
      v-if="geoError" 
      :geoErrorMsg="geoErrorMsg" 
      @closeGeoError="closeGeoError"
    />
    <MapFeatures 
      @getGeolocation="getGeolocation" 
      :coords="coords" 
      :fetchCoords="fetchCoords"
      @plotResult="plotResult"
      :searchResults="searchResults"
      @toggleSearchResults="toggleSearchResults"
      @removeResult="removeResult"  
    />
    <div id="map" class="h-full z-[1]"></div>
  </div>
</template>
