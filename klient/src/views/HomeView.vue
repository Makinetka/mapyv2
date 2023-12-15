
<template>
 <div class="h-screen relative">
  <GeoErrMod @closeGeoErr="closeGeoErr" v-if="geoErrorMess" :geoErrorMess="geoErrorMess" />
  <MapFeat @removeResult="removeResult" @getGeolocation="getGeolocation" @togSearchRes="togSearchRes" @plotResult="plotResult" :coords="coords" :fetchCoords="fetchCoords" :searchResult="searchResult" />
  <router-view></router-view>
    <div id="map" class="h-full z-[1]">
      
    </div>
 </div>
</template>

<script>

import leaflet from 'leaflet';
import { onMounted, ref} from 'vue';
import GeoErrMod from '@/components/GeoErrMod.vue';
import MapFeat from '@/components/MapFeat.vue';
export default {
  name: 'HomeView',
  components: {GeoErrMod, MapFeat
    
  },
  setup(){
    let map;
    onMounted(()=>{
      map=leaflet.map('map', {zoomControl: true,zoom:1,zoomAnimation:false,fadeAnimation:true,markerZoomAnimation:true}).setView([51.505, -0.09], 13);

      leaflet
        .tileLayer(
          `https://api.mapbox.com/styles/v1/magdalenaplaw/clp0bjpmf006u01qufiz88a0r/tiles/256/{z}/{x}/{y}@2x?access_token=${process.env.VUE_APP_API_KEY}`,
          {
            maxZoom: 17,
            id: "mapbox/streets-v11",
            tileSize: 512,
            zoomOffset: -1,
            accessToken: process.env.VUE_APP_API_KEY,
          }
        )
        .addTo(map);
        getGeolocation();
        map.on("moveend", () =>{
          closeSearchRes();
        });
    })
    const coords = ref(null);
    const fetchCoords = ref(null);
    const geoMaker = ref(null);
    const geoError = ref(null);
    const geoErrorMess = ref(null);
    const resultZnacz = ref(null);

    const getGeolocation = () => {
      if (coords.value) {
        coords.value=null;
        sessionStorage.removeItem("coords");
        map.removeLayer(geoMaker.value);
        return;
      }
      if(sessionStorage.getItem('coords')){
        coords.value = JSON.parse(sessionStorage.getItem("coords"));
        plotGeolocation(coords.value);
        return;
      }
      fetchCoords.value = true;
      navigator.geolocation.getCurrentPosition(setCoords, getLocE);
    }
    const setCoords = (pos) => {
      fetchCoords.value = null;
      const setSessionCoords = {
        lat: pos.coords.latitude,
        lng: pos.coords.longitude,
      };
      sessionStorage.setItem('coords', JSON.stringify(setSessionCoords));
      coords.value = setSessionCoords;
      plotGeolocation(coords.value);
    }
    const getLocE = (err) => {
      fetchCoords.value = null;
      geoError.value = true;
      geoErrorMess.value = err.message;
    }
    const plotGeolocation = () => {
      const customMarker = leaflet.icon({
        iconUrl: require("../assets/znacznik.svg"),
        iconSize: [40,40],
      });
      geoMaker.value=leaflet.marker([coords.value.lat, coords.value.lng], {icon: customMarker}).addTo(map);
      map.setView([coords.value.lat, coords.value.lng], 15);
    }
    const closeGeoErr = () => {
      geoError.value = null;
      geoErrorMess.value = null;
    }
    
    const plotResult = (coords) => {
        if(resultZnacz.value){
          map.removeLayer(resultZnacz.value);
        }
        const customMarker = leaflet.icon({
        iconUrl: require("../assets/znacznik.svg"),
        iconSize: [40,40],
      });
      resultZnacz.value=leaflet.marker([coords.coordinates[1], coords.coordinates[0]], {icon: customMarker}).addTo(map);
      map.setView([coords.coordinates[1], coords.coordinates[0]], 14);
      closeSearchRes();
    }

    
    const searchResult = ref(null);
    const togSearchRes = () => {
      searchResult.value = !searchResult.value;
    };

    const closeSearchRes = () => {
      searchResult.value = null;
    };

    const removeResult = () => {
      map.removeLayer(resultZnacz.value);
    };
  




    return{coords, fetchCoords, geoMaker, closeGeoErr, geoErrorMess, getGeolocation, plotResult, searchResult, togSearchRes, closeSearchRes, removeResult}
  }
}
</script>
