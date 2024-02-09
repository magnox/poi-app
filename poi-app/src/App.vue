
<template>
  <div id="app">
    <div class="map-container" style="position: relative; height: 100vh;">
      <div class="filter-container">
        <poi-filter :poiTypes="poiTypes" @update-filter="filterPois"></poi-filter>
      </div>
      <l-map :zoom="zoom" :center="center" style="height: 100vh">
        <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
        <l-marker v-for="poi in filteredPois" :key="poi.id" :lat-lng="poi.latLng" :icon="getAwesomeMarkerIcon(poi.type)">
          <l-popup>
            <div class="popup-content">
              <div class="popup-header">
                <h3>{{ poi.name }}</h3>
                <span class="poi-type">{{ poi.type }}</span>
              </div>
              <p v-if="poi.address"><strong>Adresse:</strong> {{ poi.address }}</p>
              <p v-if="poi.openingHours"><strong>Öffnungszeiten:</strong> {{ poi.openingHours }}</p>
              <p v-if="poi.note"><strong>Notiz:</strong> {{ poi.note }}</p>
              <div class="popup-footer">
                <a v-if="poi.website" :href="poi.website" target="_blank" title="Website" class="icon-link">
                  <img src="@/assets/icons/language.svg" alt="Website">
                </a>
                <a v-if="poi.googleMapsLink" :href="poi.googleMapsLink" target="_blank" title="Google Maps"
                  class="icon-link">
                  <img src="@/assets/icons/directions.svg" alt="Google Maps">
                </a>
              </div>
            </div>
          </l-popup>

        </l-marker>

        <l-marker v-if="userLocation" :lat-lng="userLocation.latLng">
          <l-popup>{{ userLocation.name }}</l-popup>
        </l-marker>
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
      center: [50.115, 8.690],
      zoom: 13,
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

          this.center = [userLat, userLng];
          this.zoom = 13;

          // Aktualisieren Sie die aktuelle Position des Benutzers separat
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

.poi-type {
  background-color: #f0f0f0;
  border-radius: 10px;
  padding: 2px 6px;
  font-size: 0.8em;
}

.popup-footer {
  text-align: right;
}

.icon-link {
  margin-left: 8px;
}

/* Optional: Stile für Hover-Effekte auf den Icons */
.icon-link:hover {
  opacity: 0.7;
}
</style>
