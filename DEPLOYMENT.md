# Deployment

https://larien.dev is served by GitHub Pages from the `gh-pages` branch
(Settings → Pages → "Deploy from a branch", `gh-pages` / root).

## Automatic (GitHub Actions)

`.github/workflows/deploy.yml`:

- **Push to `main`** → `npm ci`, `npm run build`, publish `dist/` to `gh-pages`.
- **Pull request to `main`** → same build, no publish (a build check).
- **Manual** → Actions → "Deploy to GitHub Pages" → Run workflow.

The custom domain comes from `public/CNAME`, which the build copies into
`dist/`. The workflow also writes `larien.dev` if it is missing, so a deploy
never drops the domain.

## Manual (from your machine)

```bash
npm ci
npm run deploy   # build + push dist/ to gh-pages via the gh-pages package
```

`npm run deploy:quick` does the same without the cleanup and checks.

## Troubleshooting

- **Site shows a 404 or the github.io URL**: check that `dist/CNAME` (and the
  `gh-pages` branch) contains `larien.dev`.
- **Build fails with `ERR_OSSL_EVP_UNSUPPORTED`**: the build script already
  sets `NODE_OPTIONS=--openssl-legacy-provider` for webpack 4 on Node ≥17;
  run it through `npm run build`.
- **Workflow can't push**: the job needs `contents: write` (set in the
  workflow), and Settings → Actions → Workflow permissions must allow it.
