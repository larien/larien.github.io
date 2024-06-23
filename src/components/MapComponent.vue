<template>
  <div>
    <LeafletMap />
    <MapOverlay v-if="showOverlay" @close="hideOverlay" />
  </div>
</template>

<script>
import LeafletMap from './LeafletMap.vue';
import MapOverlay from './MapOverlay.vue';

export default {
  name: 'MapComponent',
  components: {
    LeafletMap,
    MapOverlay
  },
  data() {
    return {
      showOverlay: true // Overlay is initially shown
    };
  },
  methods: {
    hideOverlay() {
      this.showOverlay = false;
    }
  }
};
</script>
