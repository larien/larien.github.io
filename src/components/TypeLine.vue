<template>
  <div class="typed">
    <!-- Visual typewriter. Hidden from assistive tech: the live region below
         announces each whole phrase once instead of every keystroke. The
         link is mouse-only (tabindex -1) because the same destinations are
         reachable as the project cards right below. -->
    <span aria-hidden="true"
      >{{ shown.pre
      }}<a v-if="shown.lk" :href="phrase.href" tabindex="-1" class="lk">{{ shown.lk }}</a
      >{{ shown.post }}<span class="caret"></span
    ></span>
    <span class="sr-only" aria-live="polite">{{ full }}</span>
  </div>
</template>

<script>
const TICK = 65; // ms per typed / erased character
const HOLD_FULL = 30; // ticks (~2s) holding the full phrase
const HOLD_EMPTY = 6; // ticks pause before the next phrase
const REDUCED_SWAP = 4000; // ms between phrases with reduced motion

export default {
  name: "TypeLine",
  props: {
    phrases: { type: Array, required: true },
    reduced: { type: Boolean, default: false },
  },
  data() {
    return { i: 0, n: 0, deleting: false, hold: 0 };
  },
  computed: {
    phrase() {
      return this.phrases[this.i % this.phrases.length];
    },
    full() {
      const s = this.phrase;
      return s.pre + s.link + s.post;
    },
    shown() {
      const s = this.phrase;
      const n = this.reduced ? this.full.length : this.n;
      const a = s.pre.length;
      const b = a + s.link.length;
      return {
        pre: this.full.slice(0, Math.min(n, a)),
        lk: n > a ? this.full.slice(a, Math.min(n, b)) : "",
        post: n > b ? this.full.slice(b, n) : "",
      };
    },
  },
  watch: {
    phrases() {
      this.restart();
    },
    reduced() {
      this.restart();
    },
  },
  mounted() {
    this.start();
  },
  beforeDestroy() {
    clearInterval(this.timer);
  },
  methods: {
    start() {
      clearInterval(this.timer);
      this.timer = this.reduced
        ? setInterval(() => (this.i = (this.i + 1) % this.phrases.length), REDUCED_SWAP)
        : setInterval(this.tick, TICK);
    },
    restart() {
      this.i = 0;
      this.n = 0;
      this.deleting = false;
      this.hold = 0;
      this.start();
    },
    tick() {
      if (this.hold > 0) {
        this.hold--;
        return;
      }
      if (!this.deleting) {
        if (this.n < this.full.length) this.n++;
        else {
          this.deleting = true;
          this.hold = HOLD_FULL;
        }
      } else if (this.n > 0) {
        this.n--;
      } else {
        this.deleting = false;
        this.i = (this.i + 1) % this.phrases.length;
        this.hold = HOLD_EMPTY;
      }
    },
  },
};
</script>

<style scoped>
.typed {
  min-height: 1.4em;
}
.lk {
  border-bottom: 2px solid var(--accent);
}
.lk:hover {
  text-decoration: none;
}
.caret {
  display: inline-block;
  width: 0.55em;
  height: 1.1em;
  background: var(--accent);
  vertical-align: -0.18em;
  margin-left: 2px;
  animation: blink 1s steps(1) infinite;
}
@keyframes blink {
  50% {
    opacity: 0;
  }
}
@media (prefers-reduced-motion: reduce) {
  .caret {
    animation: none;
  }
}
@media (max-width: 720px) {
  .typed {
    min-height: 2.7em;
  }
}
/* Narrow phones: the longest phrases need a third line. */
@media (max-width: 480px) {
  .typed {
    min-height: 4.05em;
  }
}
</style>
