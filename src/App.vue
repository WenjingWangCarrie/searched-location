<template>
  <div id="app">
    <div class="button-container">
      <button @click="getUserLocation">Get Current Location</button>
      <input v-model="searchQuery" @keyup.enter="searchLocation" placeholder="Search by name here..." />
      <button @click="searchLocation">Search</button>
    </div>

    <div v-if="latestLocation">
      <strong>Latest Searched Location</strong>
      <div>Time Zone: {{ latestLocation.timeZone }}</div>
      <div>Local Time: {{ latestLocation.localTime }}</div>
    </div>

    <div class="map-container">
      <LocationMap :markers="markers" :map="map" @mapInitialized="mapInitialized" />
    </div>

    <div class="table-container">
      <LocationTable :locations="locations" @deleteLocations="deleteLocations" />
    </div>
  </div>
</template>

<script>
import LocationMap from './components/LocationMap.vue'
import LocationTable from './components/LocationTable.vue';
import L from "leaflet";
import axios from 'axios';
import { DateTime } from 'luxon';

export default {
  name: 'App',
  components: {
    LocationMap, LocationTable,
  },
  data() {
    return {
      searchQuery: '',
      locations: [],
      map: null,
      markers: [],
      latestLocation: null,
    };
  },
  methods: {
    mapInitialized(map) {
      this.map = map;
    },
    getUserLocation() {
      if ("geolocation" in navigator) {
        navigator.geolocation.getCurrentPosition(
          async (position) => {
            const latitude = position.coords.latitude;
            const longitude = position.coords.longitude;
            const timeZone = await this.getTimeZone(latitude, longitude);
            const localTime = DateTime.now().setZone(timeZone).toLocaleString(DateTime.DATETIME_FULL);
            
            const location = {
              name: "Current Location",
              latitude: latitude,
              longitude: longitude,
              timeZone: timeZone,
              localTime: localTime,
              isSelected: false, 
            };

            this.locations.push(location);
            this.latestLocation = location;

            if (this.map) {
              // add a marker on the map
              const marker = L.marker([latitude, longitude]).addTo(this.map);
              marker.bindPopup("Current Location", { closeOnClick: false }).openPopup();
              
              // keep track of the markers
              const markerData = {
                location: location,
                marker: marker,
              };
              this.markers.push(markerData);
            } else {
              const map = L.map("leafletmap").setView([latitude, longitude], 13);

              L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
                maxZoom: 19,
                attribution: "&copy; <a href='http://www.openstreetmap.org/copyright'>OpenStreetMap</a>"
              }).addTo(map);

              this.map = map; 
            }
          }, (error) => {
            console.error("Error getting user location:", error);
          }
        );
      } else {
        alert("Geolocation is not available.");
      }
    },
    async getTimeZone(lat, lng) {
      try {
        const KEY = "replace key in here";
        const response = await fetch(
          `https://maps.googleapis.com/maps/api/timezone/json?timestamp=1331766000&location=${lat},${lng}&key=${KEY}`
        );
        
        const data = await response.json();
        return data.timeZoneId;
      } catch (error) {
        console.error('Error fetching time zone:', error);
        return null;
      }
    },
    async searchLocation() {
      if (this.searchQuery.trim() === "") 
        return;
      
      const geoLocation = await axios.get('https://maps.googleapis.com/maps/api/geocode/json', {
        params: {
          address: this.searchQuery,
          key: 'replace key in here',  
        },
      });

      if (geoLocation.data.status != "OK") {
        alert("Unable to obtain geometric position, please confirm whether the position is correct");
        return;
      } else {
        const { lat, lng } = geoLocation.data.results[0].geometry.location;
        const timeZone = await this.getTimeZone(lat, lng);
        const localTime = DateTime.now().setZone(timeZone).toLocaleString(DateTime.DATETIME_FULL);
        const searchedLocation = {
          name: this.searchQuery,
          latitude: lat,
          longitude: lng,
          timeZone: timeZone,
          localTime: localTime,
          isSelected: false,
        };

        this.locations.push(searchedLocation);
        this.latestLocation = searchedLocation;

        if (this.map) {
          const marker = L.marker([lat, lng]).addTo(this.map);
          marker.bindPopup(this.searchQuery, { closeOnClick: false }).openPopup();

          const markerData = {
            location: searchedLocation,
            marker: marker,
          };
          this.markers.push(markerData);
        } else {
          const map = L.map("leafletmap").setView([lat, lng], 13);

          L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
            maxZoom: 19,
            attribution: "&copy; <a href='http://www.openstreetmap.org/copyright'>OpenStreetMap</a>"
          }).addTo(map);

          this.map = map;
        }

        this.searchQuery = "";
      }
    },
    deleteLocations(selectedLocations) {
      selectedLocations.forEach((location) => {
        const marker = this.markers.find((marker) => marker.location.name === location.name);

        if (marker) {
          marker.marker.remove();

          const markerIndex = this.markers.indexOf(marker);
          if (markerIndex !== -1) {
            this.markers.splice(markerIndex, 1);
          }
        }
      });

      this.locations = this.locations.filter((location) => !location.isSelected);
    },
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 60px 80px;
}

button {
  padding: 20px 60px;
  background-color: #007bff;
  color: #fff;
  border: none;
  cursor: pointer;
  margin-right: 20px;
}

input {
  padding: 20px 60px;
  border: 1px solid #ccc;
}

.button-container {
  margin-bottom: 20px;
}

.map-container {
	height: 500px;
	width: 100%;
	max-width: 100%;
	max-height: 100%;
  margin-top: 20px;
 }

.table-container{
  height: 500px;
	width: 100%;
	max-width: 100%;
	max-height: 100%;
}

.leaflet-marker-icon, .leaflet-marker-shadow {
  background-image: url('@/../public/images/marker-icon-2x.png');
  background-size: contain;
  background-repeat: no-repeat;
  width: 25px;
  height: 41px;
}
</style>
