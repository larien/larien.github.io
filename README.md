# larien.dev

Personal landing page for Lauren Ferreira, served by GitHub Pages at https://larien.dev.

"Prompt" layout: a terminal-style intro panel with a typewriter line, over a full-screen background map. Design reference: [`design/reference.html`](design/reference.html) (design-tool export, not production code).

## Stack

Vue 2.7 + Vue CLI. No runtime dependencies besides Vue.

```
public/index.html          tokens (dark/light + 6 accents), pre-paint boot script, SEO tags, no-JS fallback
public/CNAME               custom domain, copied into dist by the build
src/App.vue                header, panel, cards, footer, theme/language/accent state
src/i18n.js                all copy in EN / BR / ES, links
src/components/TypeLine.vue  typewriter (65ms/char, ~2s hold, reduced-motion aware)
src/components/NomuMap.vue   background: share.nomu.land/maps/larien embed, dotted fallback while it 404s
src/assets/data/posts.json   trip list (source data)
```

Visitor choices (theme, language, accent) are stored in `localStorage` under `larien.*`.

## Commands

```bash
npm install
npm run dev       # dev server
npm run build     # production build into dist/
npm run deploy    # build + publish dist/ to the gh-pages branch
```
