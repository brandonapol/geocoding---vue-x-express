<template>
  <div class="h-screen relative">
    <div id="map" class="h-full z-[1]"></div>
    
  </div>
</template>

<script>
// @ is an alias to /src
import leaflet from "leaflet";
import { onMounted } from "vue";



export default {
  name: 'HomeView',
  components: {

  },
  setup() {
    let map;
    onMounted(() => {
      // make map
      map = leaflet.map('map').setView([28.538336, -81.379234], 10);
    })
  }
}
</script>
