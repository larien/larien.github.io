<template>
  <div id="app" class="shell">
    <div class="veil" :class="{ hidden: exploring }" aria-hidden="true"></div>

    <div class="overlay">
      <header class="glass bar">
        <a href="https://larien.dev" class="logo" :aria-label="t.home">
          <svg width="36" height="36" viewBox="0 0 962 962" fill="currentColor" aria-hidden="true" focusable="false">
            <rect width="173.48" height="962" />
            <rect y="785.02" width="962" height="176.98" />
            <rect x="262.84" width="699.16" height="173.48" />
            <rect x="788.52" width="173.48" height="700.91" />
            <rect x="262.84" y="261.09" width="697.16" height="175.23" />
            <rect x="262.84" y="523.93" width="697.16" height="176.98" />
          </svg>
        </a>
        <div class="hdr-ctl">
          <nav class="navlinks">
            <a :href="links.nomu">./nomu</a>
            <a :href="links.blog">./blog</a>
            <a :href="links.linkedin">./linkedin</a>
          </nav>
          <div class="langgrp" role="group" :aria-label="t.lang">
            <button
              v-for="l in langs"
              :key="l.key"
              type="button"
              class="ctl"
              :class="{ on: l.key === lang }"
              :aria-pressed="String(l.key === lang)"
              :lang="l.htmlLang"
              @click="setLang(l.key)"
            >
              {{ l.label }}
            </button>
          </div>
          <div ref="picker" class="picker">
            <button
              ref="pickerBtn"
              type="button"
              class="ctl icon"
              :aria-label="t.acc"
              :aria-expanded="String(pickerOpen)"
              aria-controls="accent-pop"
              @click="pickerOpen = !pickerOpen"
            >
              <span class="acc-dot"></span>
            </button>
            <div v-if="pickerOpen" id="accent-pop" class="pop" role="group" :aria-label="t.acc">
              <span class="pop-title">{{ t.acc }}</span>
              <div class="swatches">
                <button
                  v-for="k in accents"
                  :key="k"
                  type="button"
                  class="sw"
                  :class="['sw-' + k, { on: k === accent }]"
                  :aria-label="t.cn[k]"
                  :aria-pressed="String(k === accent)"
                  :title="t.cn[k]"
                  @click="pickAccent(k)"
                >
                  <span class="dot"></span>
                </button>
              </div>
            </div>
          </div>
          <button type="button" class="ctl icon" :aria-label="theme === 'dark' ? t.toLight : t.toDark" @click="toggleTheme">
            <svg v-if="theme === 'dark'" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" aria-hidden="true" focusable="false">
              <circle cx="12" cy="12" r="4" />
              <path d="M12 2v2M12 20v2M4.9 4.9l1.4 1.4M17.7 17.7l1.4 1.4M2 12h2M20 12h2M4.9 19.1l1.4-1.4M17.7 6.3l1.4-1.4" />
            </svg>
            <svg v-else width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true" focusable="false">
              <path d="M21 12.8A9 9 0 1 1 11.2 3a7 7 0 0 0 9.8 9.8z" />
            </svg>
          </button>
        </div>
      </header>

      <main class="main">
        <section v-if="!exploring" class="glass panel">
          <div><span class="prompt">~/larien.dev $</span> whoami</div>
          <h1 class="hello">{{ t.hello }}<span class="accent">Lauren</span>,</h1>
          <TypeLine class="typeline" :phrases="t.S" :reduced="reduced" />
          <p class="based">{{ t.based }}</p>
          <div class="ls"><span class="prompt">~/larien.dev $</span> ls projects/</div>
          <div class="cards">
            <a :href="links.nomu" class="card featured">
              <span class="card-name">nomu.land/</span>
              <span class="card-desc">{{ t.nomu }}</span>
              <span class="card-cta">iOS + Android →</span>
            </a>
            <a :href="links.blog" class="card">
              <span class="card-name">blog/</span>
              <span class="card-desc">{{ t.blog }}</span>
              <span class="card-cta">{{ t.blogCta }}</span>
            </a>
            <a :href="links.linkedin" class="card">
              <span class="card-name">linkedin/</span>
              <span class="card-desc">{{ t.li }}</span>
              <span class="card-cta">{{ t.liCta }}</span>
            </a>
          </div>
        </section>
      </main>

      <footer class="foot">
        <div class="glass chip legal">
          <span>© 2026 Lauren Ferreira · @larienmf</span>
          <span class="ltda">LARIEN DEV LTDA</span>
        </div>
        <button type="button" class="ctl glass explore" :aria-pressed="String(exploring)" @click="exploring = !exploring">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true" focusable="false">
            <path d="M9 4 3 6v14l6-2 6 2 6-2V4l-6 2-6-2z" />
            <path d="M9 4v14M15 6v14" />
          </svg>
          {{ exploring ? t.back : t.ex }}
        </button>
      </footer>
    </div>

    <NomuMap :interactive="!mobile || exploring" :label="t.map" />
  </div>
</template>

<script>
import TypeLine from "./components/TypeLine.vue";
import NomuMap from "./components/NomuMap.vue";
import { T, LANGS, ACCENTS, LINKS } from "./i18n";

const KEY = "larien.";
const store = {
  get(k) {
    try {
      return localStorage.getItem(KEY + k);
    } catch (e) {
      return null;
    }
  },
  set(k, v) {
    try {
      localStorage.setItem(KEY + k, v);
    } catch (e) {
      /* storage unavailable: choice lasts for this visit only */
    }
  },
};

const mq = (q) => (window.matchMedia ? window.matchMedia(q) : { matches: false, addEventListener() {}, removeEventListener() {} });
const KONAMI = ["ArrowUp", "ArrowUp", "ArrowDown", "ArrowDown", "ArrowLeft", "ArrowRight", "ArrowLeft", "ArrowRight", "b", "a"];

// The boot script in public/index.html resolves stored/system defaults
// before first paint; read them back so there is one source of truth.
function initial() {
  const root = document.documentElement;
  const lang = root.getAttribute("data-lang");
  const accent = ACCENTS.find((a) => root.classList.contains("acc-" + a));
  return {
    lang: T[lang] ? lang : "en",
    theme: root.classList.contains("light") ? "light" : "dark",
    accent: accent || "blue",
  };
}

export default {
  name: "App",
  components: { TypeLine, NomuMap },
  data() {
    return {
      ...initial(),
      pickerOpen: false,
      exploring: false,
      mobile: mq("(max-width: 720px)").matches,
      reduced: mq("(prefers-reduced-motion: reduce)").matches,
      langs: LANGS,
      accents: ACCENTS,
      links: LINKS,
      konami: 0,
    };
  },
  computed: {
    t() {
      return T[this.lang];
    },
  },
  watch: {
    lang: { handler: "applyLang", immediate: true },
    theme: "applyRootClasses",
    accent: "applyRootClasses",
  },
  mounted() {
    this.applyRootClasses();
    this.mqMobile = mq("(max-width: 720px)");
    this.mqReduced = mq("(prefers-reduced-motion: reduce)");
    this.onMobile = (e) => (this.mobile = e.matches);
    this.onReduced = (e) => (this.reduced = e.matches);
    this.mqMobile.addEventListener("change", this.onMobile);
    this.mqReduced.addEventListener("change", this.onReduced);
    document.addEventListener("keydown", this.onKey);
    document.addEventListener("click", this.onDocClick);
  },
  beforeDestroy() {
    this.mqMobile.removeEventListener("change", this.onMobile);
    this.mqReduced.removeEventListener("change", this.onReduced);
    document.removeEventListener("keydown", this.onKey);
    document.removeEventListener("click", this.onDocClick);
  },
  methods: {
    setLang(k) {
      this.lang = k;
      store.set("lang", k);
    },
    toggleTheme() {
      this.theme = this.theme === "dark" ? "light" : "dark";
      store.set("theme", this.theme);
    },
    pickAccent(k) {
      this.accent = k;
      store.set("accent", k);
      this.closePicker();
    },
    closePicker(refocus = true) {
      if (!this.pickerOpen) return;
      this.pickerOpen = false;
      if (refocus) this.$nextTick(() => this.$refs.pickerBtn && this.$refs.pickerBtn.focus());
    },
    applyLang() {
      const meta = LANGS.find((l) => l.key === this.lang);
      document.documentElement.lang = meta.htmlLang;
      document.documentElement.setAttribute("data-lang", this.lang);
      document.title = this.t.title;
    },
    applyRootClasses() {
      const root = document.documentElement;
      root.classList.remove("dark", "light", ...ACCENTS.map((a) => "acc-" + a));
      root.classList.add(this.theme, "acc-" + this.accent);
      const bg = getComputedStyle(root).getPropertyValue("--bg").trim();
      const meta = document.querySelector('meta[name="theme-color"]');
      if (meta && bg) meta.setAttribute("content", bg);
    },
    onDocClick(e) {
      if (this.pickerOpen && this.$refs.picker && !this.$refs.picker.contains(e.target)) this.closePicker(false);
    },
    onKey(e) {
      if (e.key === "Escape") this.closePicker();
      // ↑↑↓↓←→←→BA → the travel itinerary
      const want = KONAMI[this.konami];
      if (e.key && e.key.toLowerCase() === want.toLowerCase()) {
        this.konami++;
        if (this.konami === KONAMI.length) {
          this.konami = 0;
          window.open("https://trips.larien.dev", "_blank", "noopener");
        }
      } else {
        this.konami = e.key === KONAMI[0] ? 1 : 0;
      }
    },
  },
};
</script>

<style scoped>
.shell {
  position: relative;
  height: 100vh;
  height: 100dvh;
  overflow: hidden;
  color: var(--text);
  font-family: "JetBrains Mono", ui-monospace, monospace;
}
.veil {
  position: fixed;
  inset: 0;
  z-index: 1;
  pointer-events: none;
  background: linear-gradient(90deg, var(--veil) 0%, transparent 55%);
  transition: opacity 0.3s;
}
.veil.hidden {
  opacity: 0;
}
.overlay {
  position: relative;
  z-index: 2;
  height: 100vh;
  height: 100dvh;
  display: flex;
  flex-direction: column;
  pointer-events: none;
}
.glass {
  background: var(--glass);
  -webkit-backdrop-filter: blur(12px);
  backdrop-filter: blur(12px);
  border: 1px solid var(--line);
  pointer-events: auto;
}

/* Header */
.bar {
  position: relative;
  z-index: 10;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 16px;
  padding: 12px clamp(16px, 5vw, 72px);
  border-width: 0 0 1px 0;
}
.logo {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-height: 44px;
  min-width: 44px;
  color: var(--accent);
  border-radius: 8px;
}
.hdr-ctl {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 20px;
  margin-left: auto;
}
.navlinks {
  display: flex;
  gap: 24px;
  font-size: 14px;
}
.navlinks a {
  display: inline-flex;
  align-items: center;
  min-height: 44px;
}
.langgrp {
  display: flex;
  align-items: center;
  gap: 2px;
  padding-left: 16px;
  border-left: 1px solid var(--line);
}
.ctl {
  font: 500 13px "JetBrains Mono", ui-monospace, monospace;
  min-height: 44px;
  min-width: 44px;
  padding: 0 10px;
  background: transparent;
  border: 1px solid transparent;
  border-radius: 8px;
  color: var(--muted);
  cursor: pointer;
}
.ctl:hover {
  color: var(--fg);
  border-color: var(--border);
}
.ctl.on {
  color: var(--fg);
  border-color: var(--accent);
}
.ctl.icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
}
.acc-dot {
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: var(--accent);
  box-shadow: 0 0 0 2px var(--glass), 0 0 0 3px var(--border);
}

/* Accent picker */
.picker {
  position: relative;
}
.pop {
  position: absolute;
  right: 0;
  top: calc(100% + 8px);
  z-index: 20;
  width: max-content;
  display: flex;
  flex-direction: column;
  gap: 6px;
  padding: 10px;
  background: var(--panel);
  border: 1px solid var(--line);
  border-radius: 12px;
  box-shadow: 0 12px 32px rgba(0, 0, 0, 0.18);
}
.pop-title {
  font-size: 11px;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  white-space: nowrap;
  color: var(--muted);
  padding: 0 6px;
}
.swatches {
  display: grid;
  grid-template-columns: repeat(3, 44px);
  gap: 4px;
}
.sw {
  width: 44px;
  height: 44px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  background: transparent;
  border: 0;
  border-radius: 50%;
  cursor: pointer;
  padding: 0;
}
.sw .dot {
  width: 22px;
  height: 22px;
  border-radius: 50%;
  background: var(--sw);
  box-shadow: 0 0 0 2px var(--panel);
  transition: transform 0.15s;
}
.sw.on .dot {
  box-shadow: 0 0 0 2px var(--panel), 0 0 0 4px var(--fg);
}
.sw:hover .dot {
  transform: scale(1.12);
}

/* Main panel */
.main {
  flex: 1;
  display: flex;
  align-items: center;
  width: 100%;
  max-width: 1440px;
  margin: 0 auto;
  box-sizing: border-box;
  min-height: 0;
  padding: clamp(12px, 3vh, 48px) clamp(16px, 5vw, 72px);
}
.panel {
  flex: 0 1 720px;
  min-width: 0;
  display: flex;
  flex-direction: column;
  gap: clamp(8px, 2vh, 20px);
  max-height: 100%;
  box-sizing: border-box;
  overflow-y: auto;
  overscroll-behavior: contain;
  padding: clamp(18px, 4vh, 40px) clamp(18px, 3vw, 40px);
  border-radius: 18px;
  font-size: clamp(15px, min(1.4vw, 2.2vh), 19px);
  line-height: 1.6;
}
.prompt {
  color: var(--dim);
}
.hello {
  margin: 0;
  font-size: clamp(28px, min(4vw, 6.5vh), 56px);
  line-height: 1.15;
  color: var(--fg);
  font-weight: 700;
  letter-spacing: -0.02em;
}
.accent {
  color: var(--accent);
}
.typeline {
  font-size: clamp(18px, min(2.4vw, 3.8vh), 32px);
  line-height: 1.35;
  color: var(--fg);
}
.based {
  margin: 0;
  color: var(--muted);
}
.ls {
  margin-top: clamp(0px, 1.5vh, 24px);
}
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(170px, 1fr));
  gap: 12px;
}
.card {
  display: flex;
  flex-direction: column;
  gap: 6px;
  padding: 18px;
  border: 1px solid var(--border);
  border-radius: 10px;
}
.card:hover {
  text-decoration: none;
  border-color: var(--accent);
}
.card.featured {
  border-color: var(--accent);
  background: var(--card);
}
.card-name {
  font-weight: 700;
  color: var(--fg);
}
.card-desc {
  font-size: 13px;
  color: var(--muted);
  line-height: 1.5;
}
.card-cta {
  margin-top: auto;
  font-size: 12px;
  color: var(--accent);
}
.card:hover .card-cta {
  color: var(--accentHover);
}

/* Footer */
.foot {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  padding: clamp(10px, 2vh, 20px) clamp(16px, 5vw, 72px);
}
.explore {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 0 16px;
  color: var(--fg);
  border-color: var(--line);
}
.explore:hover {
  border-color: var(--border);
}
.chip {
  display: inline-flex;
  align-items: center;
  min-height: 44px;
  box-sizing: border-box;
  padding: 8px 12px;
  border-radius: 8px;
  font-size: 12px;
}
.legal {
  display: inline-flex;
  flex-wrap: wrap;
  gap: 6px 16px;
  color: var(--dim);
}
.ltda {
  letter-spacing: 0.08em;
}

@media (max-width: 720px) {
  .navlinks {
    display: none;
  }
  .langgrp {
    border-left: 0;
    padding-left: 0;
  }
  .hdr-ctl {
    gap: 4px;
  }
  .ctl {
    padding: 0 6px;
  }
  .bar {
    padding-top: 4px;
    padding-bottom: 4px;
  }
  .main {
    align-items: flex-start;
    padding-top: 12px;
    padding-bottom: 8px;
  }
  .panel {
    flex-basis: 100%;
    border-radius: 14px;
  }
  .cards {
    grid-template-columns: 1fr;
    gap: 8px;
  }
  .card {
    display: grid;
    grid-template-columns: 1fr auto;
    align-items: baseline;
    gap: 2px 12px;
    padding: 10px 14px;
  }
  .card-desc {
    grid-column: 1 / -1;
    grid-row: 2;
    font-size: 12px;
  }
  .card-cta {
    grid-column: 2;
    grid-row: 1;
  }
  .foot {
    gap: 8px;
  }
  .panel {
    gap: 12px;
    padding: 18px;
  }
  .ls {
    margin-top: 0;
  }
  .explore {
    order: -1;
    margin-left: auto;
    padding: 0 14px;
  }
  .legal {
    flex: 1 1 100%;
    justify-content: center;
  }
  .chip {
    font-size: 11px;
    padding: 6px 10px;
  }
  .legal {
    gap: 2px 10px;
  }
}
/* Short phones (e.g. 360×640): drop card blurbs so the page still fits. */
@media (max-width: 720px) and (max-height: 720px) {
  .card-desc {
    display: none;
  }
  .card {
    padding: 8px 14px;
  }
  .hello {
    font-size: 26px;
  }
  .panel {
    gap: 10px;
  }
}
/* Landscape phones / very short screens: hide the location prompt gap. */
@media (max-height: 560px) {
  .ls {
    margin-top: 0;
  }
}
</style>
