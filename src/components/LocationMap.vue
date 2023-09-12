<template>
  <div id="leafletmap"></div>
</template>

<script>
import L from "leaflet";

export default {
  props: ['markers', 'map'],
  mounted() {
    if (this.map) {
      L.marker([45.4215, -75.6919]).addTo(this.map);
    } else {
      const map = L.map("leafletmap").setView([45.4215, -75.6919], 13);

      L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
        maxZoom: 19,
        attribution: "&copy; <a href='http://www.openstreetmap.org/copyright'>OpenStreetMap</a>"
      }).addTo(map);

      L.marker([45.4215, -75.6919]).addTo(map);
      this.$emit('mapInitialized', map);
    }
  },
};
</script>

<style scoped>
@import "~leaflet/dist/leaflet.css";

#leafletmap {
  height: 100%;
  width: 100%;
}
</style>