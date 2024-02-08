
<template>
  <div id="app">
    <div class="map-container" style="position: relative; height: 100vh;">
      <div class="filter-container">
        <poi-filter :poiTypes="poiTypes" @update-filter="filterPois"></poi-filter>
      </div>
      <l-map :zoom="zoom" :center="center" style="height: 100vh">
        <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
        <l-marker v-for="poi in filteredPois" :key="poi.id" :lat-lng="poi.latLng">
          <l-popup>{{ poi.name }}</l-popup>
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
import 'leaflet/dist/leaflet.css';
import PoiFilter from './components/PoiFilter.vue';
import pois from './assets/data/pois.json';

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
      userLocation: null, // Neue Property für die aktuelle Position des Benutzers
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
  },
  mounted() {
    this.getCurrentLocation();
  }
};
</script>

<style>

html, body {
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
}
</style>
