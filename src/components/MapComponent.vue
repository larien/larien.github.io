<template>
  <div id="map" ref="map"></div>
</template>

<script>
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';
import '@/assets/mapStyles.css';
import visitedCountries from '@/assets/visitedCountries.json';

export default {
  name: 'MapComponent',
  data() {
    return {
      map: null,
      geojsonLayer: null
    };
  },
  mounted() {
    this.initMap();
  },
  methods: {
    initMap() {
      this.map = L.map(this.$refs.map, {
        minZoom: 2 // Prevent zooming out too much
      }).setView([20, 0], 2);

      L.tileLayer('https://stamen-tiles-{s}.a.ssl.fastly.net/toner-background/{z}/{x}/{y}.{ext}', {
        attribution: 'Map tiles by <a href="http://stamen.com">Stamen Design</a>, under <a href="http://creativecommons.org/licenses/by/3.0">CC BY 3.0</a>. Data by <a href="http://openstreetmap.org">OpenStreetMap</a>, under ODbL.',
        subdomains: 'abcd',
        minZoom: 0,
        maxZoom: 20,
        ext: 'png'
      }).addTo(this.map);

      fetch('https://raw.githubusercontent.com/johan/world.geo.json/master/countries.geo.json')
        .then(response => response.json())
        .then(data => {
          this.geojsonLayer = L.geoJson(data, {
            style: this.styleFeature,
            onEachFeature: this.onEachFeature
          }).addTo(this.map);
        });
    },
    styleFeature(feature) {
      const continentClass = this.getContinentClass(feature.properties.continent);
      const isVisited = visitedCountries.countries.includes(feature.properties.name);
      const visitedClass = isVisited ? 'country-visited' : 'country-unvisited';

      return {
        className: `country-default ${continentClass} ${visitedClass}`
      };
    },
    onEachFeature(feature, layer) {
      layer.on({
        mouseover: this.highlightFeature,
        mouseout: this.resetHighlight
      });
    },
    highlightFeature(e) {
      const layer = e.target;
      layer.setStyle({
        weight: 3,
        color: '#ffffff',
        fillOpacity: 0.7
      });

      if (!L.Browser.ie && !L.Browser.opera && !L.Browser.edge) {
        layer.bringToFront();
      }
    },
    resetHighlight(e) {
      this.geojsonLayer.resetStyle(e.target);
    },
    getContinentClass(continent) {
      switch (continent) {
        case 'North America':
          return 'country-north-america';
        case 'South America':
          return 'country-south-america';
        case 'Europe':
          return 'country-europe';
        case 'Africa':
          return 'country-africa';
        case 'Asia':
          return 'country-asia';
        case 'Oceania':
          return 'country-australia';
        default:
          return '';
      }
    }
  }
};
</script>

<style scoped>
#map {
  width: 100%;
  height: 100vh;
  position: relative;
}
</style>
