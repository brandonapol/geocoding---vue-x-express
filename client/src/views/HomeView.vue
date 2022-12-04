<template>
  <div class="h-screen relative">
    <GeoErrorModal />
    <div id="map" class="h-full z-[1]"></div>
    
  </div>
</template>

<script>
// @ is an alias to /src
import leaflet from "leaflet";
import { onMounted, ref } from "vue";
import GeoErrorModal from "@/components/GeoErrorModal.vue";
// Like react, ref allows our data to be interactive from inside our setup() function
// "It allows us to obtain a direct reference to a specific DOM element or child
// component instance after it's mounted. 
// This may be useful when you want to, for example, programmatically focus an 
// input on component mount, or initialize a 3rd party library on an element."



export default {
  name: 'HomeView',
  components: {
    GeoErrorModal
  },
  setup() {
    let map;
    onMounted(() => {
      // make map
      map = leaflet.map('map').setView([28.538336, -81.379234], 10);
      // make tile layer
      leaflet
        .tileLayer( // per https://docs.mapbox.com/api/maps/styles/ 
            `https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=${process.env.VUE_APP_API_KEY}`,
            {
              maxZoom: 18,
              id: "mapbox/streets-v11",
              tileSize: 512,
              zoomOffset: -1,
              accessToken: process.env.VUE_APP_API_KEY,
            }
         )
      .addTo(map);   

      getGeolocation();
    });

    const coords = ref(null); // user coordinates
    const fetchCoords = ref(null); // loading animation between time we get coordinates after fetch
    const geoMarker = ref(null); // marker to plot on our coordinates on our map
    const geoError = ref(null);
    const geoErrorMsg = ref(null);

    const getGeolocation = () => {
      fetchCoords.value = true; // this is how you change value of a ref; have to have .value
      navigator.geolocation.getCurrentPosition(setCoords, getLocErr); 
      //https://developer.mozilla.org/en-US/docs/Web/API/Navigator/geolocation - success callback, error callback
      

      
    }
    const setCoords = (pos) => {
      console.log(pos) // Take a note of what's in the JSON! 
      //! FIRST TIME: JUST LOG IT 

      // Check sessionStorage for coords (DO THIS LATER)
      if (sessionStorage.getItem('coords')) {
        coords.value = JSON.parse(sessionStorage.getItem('coords'));
        plotGeolocation(coords.value);
        return;
      }
      // runs when we have success
      fetchCoords.value = null;
      // stashing coordinates to session storage so we don't have to keep asking for loc
      const setSessionCoords = {
        lat: pos.coords.latitude,
        lng: pos.coords.longitude,
      };

      sessionStorage.setItem('coords', JSON.stringify(setSessionCoords))

      // update coords ref to received values
      coords.value = setSessionCoords;

      plotGeolocation(coords.value);
    };
    const getLocErr = (err) => {
      // console.log(err);

      // after finishing setCoords:
      fetchCoords.value = null;
      geoError.value = true;
      geoErrorMsg.value = err.message;
    };

    const closeGeoError = () => {
      geoError.value = null;
      geoErrorMsg.value = null;
    }

    const plotGeolocation = (coords) => {
      // create custom marker
      const customMarker = leaflet.icon({
        iconUrl: require('../assets/map-marker-red.svg'),
        iconSize: [35, 35],
      });

      // create new marker with coords and the custom icon
      // https://leafletjs.com/reference.html#marker
      geoMarker.value = leaflet.marker([coords.lat, coords.lng], { icon: customMarker })
        .addTo(map);

      // setting map view to current loc
      map.setView([coords.lat, coords.lng], 10);



    }

    return { coords, geoMarker, closeGeoError };
  },
};
</script>
