<template>
  <div>
    <div id="map" ref="map"></div>

    <!-- Modal -->
    <div v-if="showModal" class="modal" @click="closeModalOnOutsideClick">
      <div class="modal-content" @click.stop>
        <!-- Title/Header -->
        <h2 class="modal-title">{{ currentCityName }}</h2>
        <button class="close-button" @click="closeModal">X</button>

        <!-- Modal Content -->
        <div v-html="modalContent"></div>
      </div>
    </div>
  </div>
</template>

<script>
import L from "leaflet";
import "leaflet/dist/leaflet.css";
import "@/assets/styles.css";
import instagramPosts from "@/assets/posts.json"; // Import the Instagram posts mock data
import pinIcon from "@/assets/icons/pin.svg"; // Import the SVG file

export default {
  name: "LeafletMap",
  data() {
    return {
      map: null,
      geojsonLayer: null,
      markers: [],
      visitedCountriesSet: new Set(),
      showModal: false,
      modalContent: null,
    };
  },
  mounted() {
    this.initMap();
  },
  methods: {
    initMap() {
      this.map = L.map(this.$refs.map, {
        minZoom: 2, // Prevent zooming out too much
      }).setView([-14.235, -51.9253], 3); // Coordinates for Brazil and zoom level 3

      L.tileLayer(
        "https://stamen-tiles-{s}.a.ssl.fastly.net/toner-background/{z}/{x}/{y}.{ext}",
        {
          attribution:
            'Map tiles by <a href="http://stamen.com">Stamen Design</a>, under <a href="http://creativecommons.org/licenses/by/3.0">CC BY 3.0</a>. Data by <a href="http://openstreetmap.org">OpenStreetMap</a>, under ODbL.',
          subdomains: "abcd",
          minZoom: 0,
          maxZoom: 20,
          ext: "png",
        }
      ).addTo(this.map);

      fetch(
        "https://raw.githubusercontent.com/johan/world.geo.json/master/countries.geo.json"
      )
        .then((response) => response.json())
        .then((data) => {
          this.geojsonLayer = L.geoJson(data, {
            style: this.styleFeature,
            onEachFeature: this.onEachFeature,
          }).addTo(this.map);
        });

      this.processPosts();
    },
    getRandomBlue() {
      const blueTones = ["#014ba0", "#0a5cb8", "#1466c3", "#2174d4", "#3b8eed"];
      return blueTones[Math.floor(Math.random() * blueTones.length)];
    },
    styleFeature(feature) {
      const isVisited = this.visitedCountriesSet.has(feature.properties.name);
      const fillColor = this.getRandomBlue();

      return {
        fillColor: fillColor,
        fillOpacity: 0.7,
        color: isVisited ? "#ffffff" : fillColor,
        weight: isVisited ? 2 : 0,
        opacity: 1,
      };
    },
    onEachFeature(feature, layer) {
      const countryName = feature.properties.name || "Unknown Country";

      // Bind a tooltip to the country
      layer.bindTooltip(countryName, {
        permanent: false, // Tooltip only visible on hover
        direction: "auto",
        className: "custom-tooltip",
      });

      // Highlight on hover
      layer.on({
        mouseover: (e) => {
          this.highlightFeature(e);
        },
        mouseout: (e) => {
          this.resetHighlight(e);
        },
      });
    },
    highlightFeature(e) {
      const layer = e.target;
      layer.setStyle({
        weight: 3,
        color: "#ffffff",
        fillOpacity: 0.9,
      });

      if (!L.Browser.ie && !L.Browser.opera && !L.Browser.edge) {
        layer.bringToFront();
      }
    },
    resetHighlight(e) {
      this.geojsonLayer.resetStyle(e.target);
    },
    processPosts() {
      const customIcon = L.divIcon({
        className: "custom-pin",
        html: `<img src="${pinIcon}" style="width:30px;height:30px; background-color:white; border-radius:50%;" />`,
        iconSize: [30, 30],
        iconAnchor: [15, 30],
        popupAnchor: [0, -30],
      });

      instagramPosts.forEach((post) => {
        const location = post.location;

        // eslint-disable-next-line
        const marker = L.marker([location.latitude, location.longitude], { 
          icon: customIcon,
        }).on("click", () => {
            // Set the city name and modal content
            this.currentCityName = location.name || "Unknown City";
            this.modalContent = post.url
              ? `<iframe src="${post.url}/embed" width="300" height="400" style="border:none;"></iframe>`
              : `<p>Didn't post this one. Yet. Or not. LOL.</p>`;
            this.showModal = true;

            // Zoom into the location
            this.map.flyTo([location.latitude, location.longitude], 6, {
              animate: true,
              duration: 1.5, // Animation duration in seconds
            });
          })
          .addTo(this.map);
      });
    },
    createTooltipContent(name, url) {
      const content = document.createElement("div");
      if (url) {
        content.innerHTML = `
        <div class="tooltip-content">
          <div class="city-name">${name}</div>
          <iframe class="tooltip-iframe" src="${
            url + "/embed"
          }" width="300" height="400" style="border:none; display: none;"></iframe>
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
    },
    showTooltip(e) {
      const layer = e.target;
      const tooltip = layer.getTooltip();
      if (tooltip) {
        tooltip.getElement().onmouseover = () => {
          layer.openTooltip();
        };
        tooltip.getElement().onmouseout = () => {
          layer.closeTooltip();
        };
      }
    },
    hideTooltip(e) {
      const layer = e.target;
      const tooltip = layer.getTooltip();
      if (tooltip) {
        tooltip.getElement().onmouseover = null;
        tooltip.getElement().onmouseout = null;
      }
    },
    closeModalOnOutsideClick(event) {
      if (event.target.classList.contains("modal")) {
        this.closeModal();
      }
    },
    closeModal() {
      this.showModal = false;
      this.modalContent = null;

       // Zoom out slightly more than the last zoom level
  const currentZoom = this.map.getZoom(); // Get the current zoom level
  const targetZoom = Math.max(currentZoom - 2, 3); // Ensure it doesn't zoom out below level 3
  this.map.flyTo(this.map.getCenter(), targetZoom, { animate: true, duration: 0.7 });
      },
  },
};
</script>

<style scoped>
#map {
  width: 100%;
  height: 100%;
  position: absolute;
}

.tooltip-content {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.tooltip-iframe {
  display: none;
}

.city-name {
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

.custom-tooltip {
  background-color: #001f3f;
  color: #ffffff;
  font-size: 14px;
  font-weight: bold;
  border: 1px solid #ffffff;
  border-radius: 5px;
  padding: 5px 10px;
  text-align: center;
  max-width: 150px;
}

.city-name {
  font-size: 16px;
  color: #ffffff;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 10px;
  max-width: 400px;
  text-align: center;
}

.close-button {
  position: absolute;
  top: 10px;
  right: 10px;
  background: transparent;
  border: none;
  font-size: 18px;
  cursor: pointer;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 10px;
  max-width: 400px;
  text-align: center;
}

.close-button {
  position: absolute;
  top: 10px;
  right: 10px;
  background: transparent;
  border: none;
  font-size: 18px;
  cursor: pointer;
}

.modal-title {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 10px;
  text-align: center;
  color: #333;
}
</style>
