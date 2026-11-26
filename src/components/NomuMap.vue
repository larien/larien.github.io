<template>
  <div class="nomu-map" :class="{ locked: !interactive }">
    <!-- Nomu share map, used as soon as the share page is published. -->
    <iframe
      v-if="live"
      class="embed"
      :src="src"
      :title="label"
      loading="lazy"
      referrerpolicy="strict-origin-when-cross-origin"
      :tabindex="interactive ? 0 : -1"
    ></iframe>

    <!-- Until then (share page 404s): a plain dotted background. -->
    <svg v-else class="grid" aria-hidden="true" focusable="false">
      <defs>
        <pattern id="nomu-dots" width="14" height="14" patternUnits="userSpaceOnUse">
          <circle class="gdot" cx="7" cy="7" r="1.3" />
        </pattern>
      </defs>
      <rect width="100%" height="100%" fill="url(#nomu-dots)" />
    </svg>
  </div>
</template>

<script>
import { LINKS } from "@/i18n";

export default {
  name: "NomuMap",
  props: {
    interactive: { type: Boolean, default: true },
    label: { type: String, default: "" },
  },
  data() {
    return {
      live: false,
      src: LINKS.shareMap,
    };
  },
  async mounted() {
    // share.nomu.land answers with Access-Control-Allow-Origin: *, so the
    // status is readable. Switch to the embed only once the page exists.
    try {
      const res = await fetch(LINKS.shareMap, { method: "GET", mode: "cors", cache: "no-store" });
      this.live = res.ok;
    } catch (e) {
      this.live = false;
    }
    this.$emit("status", this.live);
  },
};
</script>

<style scoped>
.nomu-map {
  position: fixed;
  inset: 0;
  z-index: 0;
  background: var(--bg);
}
.nomu-map.locked {
  pointer-events: none;
}
.embed {
  width: 100%;
  height: 100%;
  border: 0;
  display: block;
}
.grid {
  display: block;
  width: 100%;
  height: 100%;
}
.gdot {
  fill: var(--grid);
}
</style>
