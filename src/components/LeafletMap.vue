<template>
  <div id="map" ref="map"></div>
</template>

<script>
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';
import '@/assets/mapStyles.css';
import instagramPosts from '@/assets/instagramPosts.json'; // Import the Instagram posts mock data
import pinIcon from '@/assets/pin-icon.svg'; // Import the SVG file

export default {
  name: 'LeafletMap',
  data() {
    return {
      map: null,
      geojsonLayer: null,
      markers: [],
      visitedCountriesSet: new Set()
    };
  },
  mounted() {
    this.initMap();
  },
  methods: {
    initMap() {
      this.map = L.map(this.$refs.map, {
        minZoom: 2 // Prevent zooming out too much
      }).setView([-14.2350, -51.9253], 3); // Coordinates for Brazil and zoom level 3

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

      this.processInstagramPosts();
    },
    getRandomBlue() {
      const blueTones = [
        '#014ba0', '#0a5cb8', '#1466c3', '#2174d4', '#3b8eed'
      ];
      return blueTones[Math.floor(Math.random() * blueTones.length)];
    },
    styleFeature(feature) {
      const isVisited = this.visitedCountriesSet.has(feature.properties.name);
      const fillColor = this.getRandomBlue();

      return {
        fillColor: fillColor,
        fillOpacity: 0.7,
        color: isVisited ? '#ffffff' : fillColor,
        weight: isVisited ? 2 : 0,
        opacity: 1
      };
    },
    onEachFeature(feature, layer) {
      layer.on({
        mouseover: this.highlightFeature,
        mouseout: this.resetHighlight
      });
      layer.bindTooltip(feature.properties.name, {
        permanent: false,
        direction: 'auto',
        className: 'custom-tooltip',
        sticky: true
      });
    },
    highlightFeature(e) {
      const layer = e.target;
      layer.setStyle({
        weight: 3,
        color: '#ffffff',
        fillOpacity: 0.9
      });

      if (!L.Browser.ie && !L.Browser.opera && !L.Browser.edge) {
        layer.bringToFront();
      }
    },
    resetHighlight(e) {
      this.geojsonLayer.resetStyle(e.target);
    },
    processInstagramPosts() {
      const customIcon = L.divIcon({
        className: 'custom-pin',
        html: `<img src="${pinIcon}" style="width:30px;height:30px; background-color:white; border-radius:50%;" />`,
        iconSize: [30, 30],
        iconAnchor: [15, 30],
        popupAnchor: [0, -30]
      });

      instagramPosts.forEach(post => {
        const location = post.location;
        this.visitedCountriesSet.add(location.country);
        // Use the post.url for the iframe preview
        const tooltipContent = this.createTooltipContent(location.name, post.url);
        const marker = L.marker([location.latitude, location.longitude], { icon: customIcon })
          .bindTooltip(tooltipContent, {
            className: 'custom-tooltip',
            direction: 'top'
          })
          .on('click', () => {
            window.open(post.url, '_blank');
          })
          .addTo(this.map);
        this.markers.push(marker);
      });

      console.log('Markers added:', this.markers);
      console.log('Visited countries:', Array.from(this.visitedCountriesSet));
    },
    createTooltipContent(name, url) {
      const content = document.createElement('div');
      if (url) {
        content.innerHTML = `
          <div class="tooltip-content">
            <iframe src="${url+"/embed"}" width="200" height="150" style="border:none;" onload="this.contentWindow.scrollTo(this.contentWindow.document.body.scrollWidth/2 - this.clientWidth/2, this.contentWindow.document.body.scrollHeight/2 - this.clientHeight/2);"></iframe>
            <div class="city-name">${name}</div>
          </div>
        `;
      } else {
        content.innerHTML = `
          <div class="tooltip-content">
            <div class="city-name">${name}</div>
          </div>
        `;
  }
  return content;
}

  }
};
</script>

<style scoped>
#map {
  width: 100%;
  height: 100%;
  position: absolute;
}

.custom-tooltip {
  background-color: #001f3f;
  color: #ffffff;
  font-size: 14px;
  font-weight: bold;
  border: 1px solid #ffffff;
  border-radius: 5px;
  padding: 10px;
  text-align: center;
  max-width: 250px;
}

.custom-tooltip iframe {
  width: 100%;
  height: 150px;
  border: none;
  display: block;
  margin-bottom: 5px;
}

.custom-tooltip .city-name {
  font-size: 16px;
  color: #ffffff;
}

.leaflet-interactive {
  outline: none;
}

.custom-pin img {
  width: 30px;
  height: 30px;
  background-color: white;
  border-radius: 50%;
}
</style>
