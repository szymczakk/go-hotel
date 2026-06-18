---
project: go-hotel
deployed_at: 2026-06-18
platform: Cloudflare Pages
production_url: https://go-hotel.pages.dev
first_preview_url: https://231ba898.go-hotel.pages.dev
ci_workflow: .github/workflows/pages-deploy.yml
---

## First Deployment — Cloudflare Pages

**Platform decision:** `context/foundation/infrastructure.md`

### What was deployed

Pure static SPA: React 19 + Vite + TypeScript + Phaser.js 4. Build artifact: `dist/` (output of `npm run build`). No server-side runtime. No Pages Functions.

### Deploy commands

```bash
# Local one-off deploy (after `wrangler login`)
npm run build
wrangler pages deploy

# CI (GitHub Actions — triggers on push to main)
# .github/workflows/pages-deploy.yml
```

### Production branch mapping

Cloudflare Pages dashboard → go-hotel → Settings → Builds & deployments → Production branch: `main`

Any `wrangler pages deploy --branch main` promotes to `go-hotel.pages.dev`.

### Secrets wired

| Secret | Where | Used by |
|---|---|---|
| `CLOUDFLARE_API_TOKEN` | GitHub repo → Actions secrets | `cloudflare/wrangler-action@v3` |
| `CLOUDFLARE_ACCOUNT_ID` | GitHub repo → Actions secrets | `cloudflare/wrangler-action@v3` |

Token scope: Cloudflare Pages Edit (scoped to this account, not an account-level key).

### Risk mitigations active

- `test -f dist/index.html` in CI guards against silent empty-build deploys
- `public/_redirects` (`/* /index.html 200`) prevents SPA routing 404s
- CI triggers only on `main` pushes — no per-PR preview deploys (preserves 500-deploy/month free cap)
- `.wrangler/` in `.gitignore` — local Wrangler cache not committed
