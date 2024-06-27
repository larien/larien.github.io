<template>
  <div class="map-container">
    <LeafletMap />
    <MapOverlay v-if="showOverlay" />
    <div class="icon-container">
      <img
        src="@/assets/map-icon.svg"
        alt="Map Icon"
        class="map-icon"
        @click="toggleOverlay"
        @mouseover="showTooltip"
        @mouseleave="hideTooltip"
      />
      <div v-if="showTooltipText" class="tooltip">Trips</div>
    </div>
    <div v-if="showOverlay" class="icon-container">
      <div class="linkedin-container">
        <a href="https://linkedin.larien.dev" target="_blank">
          <img
            src="@/assets/linkedin-icon.svg"
            alt="LinkedIn Icon"
            class="linkedin-icon"
          />
        </a>
      </div>
      <div class="blog-container">
        <a href="https://blog.larien.dev" target="_blank">
          <img src="@/assets/blog-icon.svg" alt="Blog Icon" class="blog-icon" />
        </a>
      </div>
    </div>
    <div v-if="!showOverlay" class="icon-container">
      <div class="blog-container">
        <a href="https://instagram.larien.dev" target="_blank">
          <img src="@/assets/instagram-icon.svg" alt="Instagram Icon" class="blog-icon" />
        </a>
      </div>
    </div>
  </div>
</template>

<script>
import LeafletMap from "./LeafletMap.vue";
import MapOverlay from "./MapOverlay.vue";

export default {
  name: "MapComponent",
  components: {
    LeafletMap,
    MapOverlay,
  },
  data() {
    return {
      showOverlay: true,
      showTooltipText: false,
    };
  },
  methods: {
    toggleOverlay() {
      this.showOverlay = !this.showOverlay;
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
  },
};
</script>

<style scoped>
.map-container {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
}

.icon-container {
  position: fixed;
  bottom: 20px;
  left: 20px;
  z-index: 1001; /* Ensure it's above the overlay */
}

.map-icon {
  width: 80px; /* Same size as the logo */
  height: auto;
  cursor: pointer;
  transition: background-color 0.3s ease, padding 0.3s ease,
    border-radius 0.3s ease;
  padding: 10px;
  border-radius: 50%;
}

.map-icon:hover {
  background-color: #1466c3; /* Set your desired blue tone */
}

.tooltip {
  position: fixed;
  bottom: 110px; /* Adjust position as needed */
  left: 20px;
  background-color: white;
  color: #001f3f;
  padding: 5px 10px;
  border-radius: 5px;
  font-size: 16px;
  display: inline-block;
  z-index: 1001; /* Ensure it's above the overlay */
}

.linkedin-container {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 1001; /* Ensure it's above the overlay */
}

.linkedin-icon {
  width: 70px; /* Adjust size as needed */
  height: auto;
  cursor: pointer;
  transition: background-color 0.3s ease, padding 0.3s ease,
    border-radius 0.3s ease;
  padding: 10px;
  border-radius: 10%;
}

.linkedin-icon:hover {
  background-color: #1466c3; /* Set your desired blue tone */
}

.blog-container {
  position: fixed;
  bottom: 20px;
  right: 20px;
  z-index: 1001; /* Ensure it's above the overlay */
}

.blog-icon {
  width: 80px; /* Adjust size as needed */
  height: auto;
  cursor: pointer;
  transition: background-color 0.3s ease, padding 0.3s ease,
    border-radius 0.3s ease;
  padding: 10px;
  border-radius: 10%;
}

.blog-icon:hover {
  background-color: #1466c3; /* Set your desired blue tone */
}

/* Media queries for mobile devices */
@media (max-width: 768px) {
  .map-icon,
  .blog-icon,
  .linkedin-icon {
    width: 60px; /* Adjust size for mobile */
    padding: 8px;
  }

  .tooltip {
    font-size: 14px;
  }
}
</style>
