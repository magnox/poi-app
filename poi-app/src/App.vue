
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
      center: [51.505, -0.09],
      zoom: 13,
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      attribution: 'Map data &copy; OpenStreetMap contributors',
      pois,
      filteredPois: pois,
      poiTypes: Array.from(new Set(pois.map(poi => poi.type)))
    };
  },
  methods: {
    filterPois(selectedTypes) {
      this.filteredPois = this.pois.filter(poi => selectedTypes.includes(poi.type));
    }
  }
};
</script>

<style>
/* Stellen Sie sicher, dass die Karten-Icons korrekt geladen werden */
.leaflet-container {
  background-color: #f0f0f0;
}

/* Korrigiert einen häufigen Fehler, bei dem Leaflet-Icons nicht angezeigt werden */
.leaflet-marker-icon {
  filter: drop-shadow(0 0 1px #454545);
}

.map-container {
  position: relative;
  height: 100vh;
}

.filter-container {
  position: absolute;
  top: 10px;
  right: 10px;
  z-index: 1000;
  background-color: rgba(255, 255, 255, 0.9);
  padding: 10px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.3);
}
</style>
