
<template>
  <div id="app">
    <div class="map-container" style="position: relative; height: 100vh;">
      <div class="filter-container">
        <poi-filter :poiTypes="poiTypes" @update-filter="filterPois"></poi-filter>
      </div>
      <l-map ref="mapRef" :zoom="zoom" :center="center" style="height: 100vh">
        <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
        <l-marker v-for="poi in filteredPois" :key="poi.id" :lat-lng="poi.latLng" :icon="getAwesomeMarkerIcon(poi.type)">
          <l-popup>
            <div class="popup-content">
              <div class="popup-header">
                <h3>{{ poi.name }}</h3>
                <span class="poi-type">{{ poi.type }}</span>
              </div>
              <div class="info-table">
                <div class="info-row" v-if="poi.address">
                  <div class="info-icon">
                    <img src="@/assets/icons/home.svg" alt="Address Icon" class="small-icon">
                  </div>
                  <div class="info-text">
                    <a :href="'https://maps.google.com/?q=' + encodeURI(poi.name + ', ' + poi.address)" target="_blank">{{ poi.address
                    }}</a>
                  </div>
                </div>
                <div class="info-row" v-if="poi.website">
                  <div class="info-icon">
                    <img src="@/assets/icons/language.svg" alt="Website Icon" class="small-icon">
                  </div>
                  <div class="info-text">
                    <a :href="poi.website" target="_blank">{{ poi.website
                    }}</a>
                  </div>
                </div>
                <div class="info-row" v-if="poi.openingHours">
                  <div class="info-icon">
                    <img src="@/assets/icons/schedule.svg" alt="Clock Icon" class="small-icon">
                  </div>
                  <div class="info-text">
                    {{ poi.openingHours }}
                  </div>
                </div>
                <div class="info-row" v-if="poi.note">
                  <div class="info-icon">
                    <img src="@/assets/icons/description.svg" alt="Note Icon" class="small-icon">
                  </div>
                  <div class="info-text">
                    {{ poi.note }}
                  </div>
                </div>
              </div>
            </div>
          </l-popup>

        </l-marker>
        <l-marker v-if="userLocation" :lat-lng="userLocation.latLng" :icon="getUserLocationIcon()"/>
      </l-map>
    </div>
  </div>
</template>

<script>
import { LMap, LTileLayer, LMarker, LPopup } from '@vue-leaflet/vue-leaflet';
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';
import PoiFilter from './components/PoiFilter.vue';
import pois from './assets/data/pois.json';
import 'leaflet.awesome-markers/dist/leaflet.awesome-markers.js';
import 'leaflet.awesome-markers/dist/leaflet.awesome-markers.css';
import 'leaflet.awesome-markers';

export default {
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPopup,
    PoiFilter
  },
  data() {
    return {
      center: [51.164, 10.454],
      zoom: 6,
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      attribution: 'Map data &copy; OpenStreetMap contributors',
      pois,
      filteredPois: pois,
      poiTypes: Array.from(new Set(pois.map(poi => poi.type))),
      userLocation: null,
    };
  },
  methods: {
    filterPois(selectedTypes) {
      this.filteredPois = this.pois.filter(poi => selectedTypes.includes(poi.type));
    },
    getCurrentLocation() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(position => {
          const userLat = position.coords.latitude;
          const userLng = position.coords.longitude;

          /*this.$refs.mapRef.mapObject.flyTo([userLat, userLng], 13, {
            duration: 1,
            animate: true,
            easeLinearity: 0.25,
          });*/
          this.center = [userLat, userLng];
          this.zoom = 13;

          this.userLocation = { latLng: [userLat, userLng], name: 'Ihre Position' };

        }, () => {
          alert("Zugriff auf den Standort verweigert. Standardansicht wird verwendet.");
        });
      } else {
        alert("Geolocation wird von diesem Browser nicht unterstützt.");
      }
    },
    getAwesomeMarkerIcon(type) {
      const markerColor = this.getColorForType(type);

      return L.AwesomeMarkers.icon({
        // does not work yet icon: 'star', // Ersetzen Sie dies durch das entsprechende Symbol
        markerColor: markerColor,
        //prefix: 'glyphicon', 
      });
    },
    getColorForType(type) {
      const hash = Array.from(type).reduce((acc, char) => char.charCodeAt(0) + acc, 0);
      const colors = ['red', 'darkred', 'orange', 'green', 'darkgreen', 'blue', 'purple', 'darkpurple', 'cadetblue'];
      const randomIndex = Math.floor(Math.abs(Math.sin(hash) * colors.length) % colors.length);
      return colors[randomIndex];
    },
    getUserLocationIcon() {
      const iconSize = [32, 32];
      const iconAnchor = [16, 32];

      const customIcon = L.icon({
        iconUrl: '/assets/icons/my_location.svg',
        iconSize: iconSize,
        iconAnchor: iconAnchor,
      });

      return customIcon;
    },
  },
  mounted() {
    this.getCurrentLocation();
  }
};
</script>


<style>
html,
body {
  padding: 0;
  margin: 0;
  height: 100%;
  width: 100%;
}

#app {
  height: 100%;
}

/* Stellen Sie sicher, dass die Karten-Icons korrekt geladen werden */
.leaflet-container {
  background-color: #f0f0f0;
}

/* Korrigiert einen häufigen Fehler, bei dem Leaflet-Icons nicht angezeigt werden */
.leaflet-marker-icon {
  filter: drop-shadow(0 0 1px #454545);
}

.map-container {
  height: 100vh;
  width: 100vw;
}

.filter-container {
  position: absolute;
  top: 10px;
  right: 10px;
  z-index: 1000;
  background-color: rgba(255, 255, 255, 0.9);
  padding: 10px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
  margin-left: 15%;
}

/* Popup-Container Stil */
.l-popup-content {
  font-family: 'Arial', sans-serif;
  color: #333;
  max-width: 250px;
}

.l-popup-content h3 {
  color: #007BFF;
  margin-bottom: 8px;
}

.l-popup-content p {
  margin-bottom: 4px;
  font-size: 14px;
}

.l-popup-content a {
  display: inline-flex;
  align-items: center;
  text-decoration: none;
  color: #007BFF;
  margin-top: 8px;
}

.l-popup-content a img {
  width: 20px;
  margin-right: 5px;
}

.l-popup-content a:hover {
  text-decoration: underline;
}

.popup-content {
  position: relative;
}

.popup-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.info-table {
  display: flex;
  flex-direction: column;
  padding: 5px;
  background-color: #fafafa;
  border-radius: 10px;
}

.info-row {
  display: flex;
  align-items: normal;
  margin: 4px;
}

.info-icon {
  display: table-cell;
  padding-right: 10px;
}

.info-text {
  display: table-cell;
}

.info-text a {
  color: black;
}

.poi-type {
  background-color: #f0f0f0;
  border-radius: 10px;
  padding: 2px 6px;
  font-size: 0.8em;
  margin-left: 10px;
}

.icon-link {
  margin-left: 8px;
}

.small-icon {
  width: 16px;
  height: 16px;
}
</style>
