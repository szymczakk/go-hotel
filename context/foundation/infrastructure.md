---
project: go-hotel
researched_at: 2026-06-18
recommended_platform: Cloudflare Pages
runner_up: Netlify
context_type: mvp
tech_stack:
  language: TypeScript
  framework: React + Vite + Phaser.js
  runtime: browser SPA (no server-side runtime)
---

## Recommendation

**Deploy on Cloudflare Pages.**

Cloudflare Pages is the strongest free-tier static host for this stack: a pure Vite-built SPA deploys with a single `wrangler pages deploy dist/` command, static asset serving is unmetered at $0, and the developer already has hands-on Cloudflare familiarity (interview Q3). All three top-scoring platforms tied at 4.5/5 on the agent-friendly criteria, but Cloudflare wins on the cost constraint (minimize cost, Q2) and on the tie-breaker of existing familiarity — reinforced by `deployment_target: cloudflare-pages` already recorded in `tech-stack.md`.

## Platform Comparison

### Scoring Matrix

| Platform | CLI-first | Managed/Serverless | Agent docs | Stable deploy API | MCP | Total |
|---|---|---|---|---|---|---|
| **Cloudflare Pages** | Pass | Pass | Pass | Pass | Partial | **4.5** |
| **Netlify** | Pass | Pass | Pass | Pass | Partial | **4.5** |
| **Vercel** | Pass | Pass | Pass | Pass | Partial | **4.5** |
| Render | Partial | Pass | Pass | Partial | Partial | 3.5 |
| Railway | Partial | Partial | Partial | Partial | Pass | 3.0 |
| Fly.io | Partial | Partial | Partial | Partial | Partial | 2.5 |

**CLI-first notes:** Netlify and Railway lose half a point because CLI rollback is dashboard-only on both. Render loses half a point because rollback is via deploy hook/API, not a clean single CLI command. Fly.io loses half a point because rollback is a manual re-deploy of a prior image — no `fly rollback` command.

**Managed/Serverless notes:** Railway and Fly.io lose half a point because both require a persistent container (Caddy on Railway, nginx in Docker on Fly.io) — you manage a running process rather than just uploading build artifacts. Cloudflare, Netlify, Vercel, and Render all serve static files from a CDN/edge layer with zero container overhead.

**Agent docs notes:** Fly.io and Railway lose half a point — Fly.io's prose docs are HTML-rendered with no confirmed `llms.txt`; Railway has `.md` URL access but no explicit `llms.txt`. All other platforms publish `llms.txt` or `llms-full.txt` with agent-readable documentation.

**Stable deploy API notes:** Render and Railway lose half a point because rollback is not a deterministic single-command operation (Render: dashboard + deploy hook; Railway: dashboard only). Fly.io loses half a point for the same reason.

**MCP notes:** Cloudflare's MCP server is preview (August 2025). Vercel's is Public Beta (June 2026), read-only. Netlify's is production-available but not formally versioned (June 2026). Fly.io's is experimental (June 2026). Railway's MCP server is the most mature of the group — GA, both local and remote modes, dedicated Claude Code integration page. Render's MCP is GA for read operations but deploy triggers are not supported via MCP. All are Partial — none is fully GA with deploy mutation support.

**Soft weights applied (post-matrix):**

- **Minimize cost (Q2):** Fly.io (~$4+/month minimum, no free tier for new users) and Railway (~$5/month Hobby) are penalized. Cloudflare Pages, Netlify, and Vercel are all $0 at 10k–100k monthly page views for this workload.
- **Cloudflare familiarity (Q3):** Cloudflare Pages selected over Netlify and Vercel as the tie-breaker.
- **Single region fine (Q4):** No bonus applied for edge-native platforms; CDN is still preferable to a single-container deployment but not a differentiator between the three top scorers.
- **External services OK (Q5):** Co-location is a non-factor for shortlisting.

### Shortlisted Platforms

#### 1. Cloudflare Pages (Recommended)

**Why it won:** Pure CDN static hosting at $0 with unlimited static asset requests and no bandwidth cap on the free tier. Wrangler CLI (`wrangler pages deploy dist/`) is GA and deploys a Vite output directory in a single command. Docs are agent-readable via `developers.cloudflare.com/llms.txt`. The developer's existing Cloudflare familiarity is the decisive tie-breaker between three equally-scored platforms. The stack's own `tech-stack.md` already records `deployment_target: cloudflare-pages`.

#### 2. Netlify

**Why it scored second:** Equal raw score (4.5). $0 at current traffic scale under the 300 monthly credit allocation (comfortably covers 10k–100k page views). The strongest agent-accessible docs of any platform — `netlify/context-and-tools` GitHub repo publishes structured Markdown skill files covering CLI, deploy, Functions, and framework adapters including Vite. Drops to runner-up because CLI rollback is UI-only (no `netlify rollback` command), and the newer credit-based free tier model (September 2025+) is less transparent than a flat bandwidth cap.

#### 3. Vercel

**Why it scored third:** Equal raw score (4.5). $0 on Hobby plan, excellent DX, Vite zero-config detection, `vercel.com/docs/llms-full.txt` machine-readable docs. Two factors place it third: (1) the Hobby plan explicitly prohibits commercial use — a future risk if the game is ever published with IAP or subscriptions; (2) the 100 GB monthly bandwidth cap is tighter than Netlify or Cloudflare for a game with a large Phaser.js asset bundle (sprites, audio, WASM) at growing traffic.

## Anti-Bias Cross-Check: Cloudflare Pages

### Devil's Advocate — Weaknesses

1. **25 MB deployment size limit on the free tier (20,000 files max).** Marketing emphasizes "unlimited requests" and "unlimited bandwidth" but the per-deployment upload cap is buried in the platform limits documentation. A Phaser.js game naturally accumulates spritesheets, audio files, and particle textures — hitting this limit does not produce a clear error, it silently fails or rejects the deployment. Assets must be migrated to R2 before the bundle crosses this threshold.

2. **No single-command CLI rollback.** There is no `wrangler pages rollback`. Rollback requires either clicking "Rollback deployment" in the dashboard or manually listing deployments (`wrangler pages deployment list`) and re-deploying a prior artifact — a two-step workaround rather than a one-liner. For automated recovery this is friction.

3. **500 deployments/month cap is shared across all Pages projects on the account.** At roughly 17 commits/day across all projects the cap is exhausted. Preview deployments count toward the same account-level cap, not just production deploys.

4. **`wrangler pages deployment tail` only works for Pages Functions, not pure static serving.** A static SPA has no server-side logs. All runtime debugging is browser-console-only; an agent cannot programmatically verify correct serving via log tailing.

5. **`wrangler pages deploy` does not validate that `dist/index.html` exists before uploading.** A silent Vite build failure can result in Wrangler uploading an empty directory, exiting 0, and the platform serving a 404 — with no deployment-time guard to catch it.

### Pre-Mortem — How This Could Fail

*Six months after shipping go-hotel on Cloudflare Pages, the situation is quietly broken and nobody noticed for weeks.*

The game grew as planned. New hotels, a particle effects engine, three background music tracks, and a tileset pack pushed the bundled `dist/` to 41 MB. Deploys stopped working — Wrangler exited 0 each time, but the deployment was silently rejected at the platform boundary because the free-tier 25 MB file cap was hit. The live site continued serving the last successful deploy from three weeks prior. Players reported that the second-hotel spritesheet was missing on fresh installs; the developer attributed it to a caching bug and spent two days debugging the wrong thing.

Meanwhile, GitHub Actions was deploying on every push to `main` and every feature branch. With 4–5 pushes per day, the account hit the 500-deployment/month cap midway through month five. Pages started dropping deployments silently — no email notification, no visible error in the Actions log (the Wrangler step still exited 0). Two weeks of commits were never reflected in production. The developer discovered this only when a player screenshot showed the game missing an animation that had been in production for a month.

The root cause: two invisible failure modes — the 25 MB asset limit and the 500-deploy/month cap — operating silently in parallel, neither surfaced by Wrangler's exit codes.

### Unknown Unknowns

1. **`_redirects` routing is not validated at deploy time.** For a Phaser.js game at a single `/` entry point this may never matter — but the moment React Router is added (even for a settings screen), every direct-URL navigation outside of `/` 404s in production while working perfectly in local dev. Wrangler gives no warning.

2. **Cloudflare's free-tier SSL for custom domains can silently fail when the domain's registrar enforces CAA records that exclude Cloudflare's CA.** This delays go-live by 24–48 hours. The fix requires moving nameservers to Cloudflare or manually adding a CAA record — a DNS detail most deploy guides omit.

3. **The Cloudflare MCP server (`@cloudflare/mcp-server-cloudflare`) is preview-only as of August 2025.** Using it for agent-driven deploy automation today means depending on an unstable tool schema. The stable, GA path is Wrangler CLI only.

4. **`wrangler pages dev dist/` does not replicate the `_redirects` routing behavior of deployed Pages.** SPA routing bugs only surface post-deploy. Local development should use `npm run dev` (Vite dev server) — Wrangler's local mode is redundant for a pure static SPA.

5. **The 25 MB deployment limit applies to the compressed upload, not the source file sizes.** A game project that is 40 MB on disk may compress to 22 MB (pass) or 28 MB (fail) depending on asset types. There is no pre-deploy dry-run to check against the limit before uploading.

## Operational Story

- **Preview deploys:** Every `wrangler pages deploy dist/` without `--branch main` creates a unique preview URL (format: `<hash>.<project>.pages.dev`). Preview URLs are publicly accessible by default — protect with Cloudflare Access (GA) if the game should be invite-only before launch. Fork PRs from external contributors also create preview deploys.
- **Secrets:** Environment variables are set per-project/per-environment in the Cloudflare Pages dashboard under Settings → Environment Variables, or via `wrangler pages secret put VAR_NAME`. For CI (GitHub Actions), the Cloudflare API token goes in GitHub Secrets as `CLOUDFLARE_API_TOKEN`; Wrangler reads it from the environment. Never commit tokens in `wrangler.toml`.
- **Rollback:** In the dashboard: Deployments tab → three-dot menu on any prior deploy → "Rollback deployment" (instantaneous, promotes prior artifact). Via CLI: `wrangler pages deployment list --project-name <name>` to find the prior deployment ID, then `wrangler pages deploy dist/ --project-name <name>` from the prior commit. No `wrangler rollback` shortcut exists. Data caveats: localStorage game saves (in the player's browser) are not affected by platform rollback.
- **Approval:** An agent may deploy to Cloudflare Pages unattended (wrangler deploy, secret rotation). Human-only actions: deleting the project, modifying DNS/domain routing, billing tier changes, Cloudflare Access policy changes. Rotating the primary `CLOUDFLARE_API_TOKEN` should be done manually in the Cloudflare dashboard and updated in GitHub Secrets by hand.
- **Logs:** For a pure static SPA, there are no server-side logs. Runtime errors surface in the player's browser console. Pipeline (build) logs are accessible in the Pages dashboard → Deployments → specific deployment → "View build log". Via CLI: `wrangler pages deployment tail --project-name <name>` streams logs for Pages Functions invocations only — not applicable until Functions are added.

## Risk Register

| Risk | Source | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| `dist/` bundle exceeds 25 MB free-tier upload limit as game assets grow | Devil's advocate | M | H | Add a `du -sh dist/` check to the CI pipeline; migrate large assets (audio, large spritesheets) to Cloudflare R2 before the limit is reached |
| 500 deployment/month cap exhausted by frequent CI pushes across projects | Devil's advocate | M | M | Gate preview deploys: only deploy to Pages on merges to `main`; use `--branch main` flag; monitor account deployment count monthly |
| Wrangler exits 0 on a failed/empty deploy — no build-time guard | Devil's advocate | L | H | Add `test -f dist/index.html` as a CI step before `wrangler pages deploy`; fail the pipeline if the file is absent |
| SPA routing 404s post-deploy when React Router is added, no Wrangler warning | Unknown unknowns | M | M | Add `public/_redirects` with `/* /index.html 200` before adding any client-side router |
| SSL provisioning fails for custom domain due to CAA record mismatch | Unknown unknowns | L | M | Before pointing a custom domain, verify CAA records allow `letsencrypt.org` or move nameservers to Cloudflare |
| MCP server schema instability breaks agent automation | Unknown unknowns | L | L | Use Wrangler CLI (GA) as the only agent-controlled deploy path; treat MCP as future optionality only |
| Wrangler local dev gives false parity — routing bugs only appear post-deploy | Unknown unknowns | M | L | Use `npm run dev` for local development; treat `wrangler pages dev` as redundant for a pure static SPA |
| Six-month game growth causes two silent failure modes (25 MB + 500-deploy cap) in parallel | Pre-mortem | M | H | Instrument CI: add bundle size check + monthly deploy count alert; migrate assets to R2 proactively at ~15 MB bundle size |
| Credit card required after free-tier limits are hit; no soft billing alerts on free tier | Research finding | L | M | Enable Cloudflare billing alerts; set a monthly budget notification in the dashboard |

## Getting Started

1. **Install Wrangler** (requires Node.js 18+):
   ```bash
   npm install -g wrangler
   ```
   Verify: `wrangler --version` (expect Wrangler 3.x)

2. **Authenticate with your Cloudflare account:**
   ```bash
   wrangler login
   ```
   This opens a browser window to authorize the CLI. The token is stored in `~/.wrangler/config/default.toml`.

3. **Add the SPA routing redirect rule** before first deploy — create `public/_redirects`:
   ```
   /* /index.html 200
   ```
   Vite copies `public/` into `dist/` at build time, so this file will be present in every deployment.

4. **Build and deploy to Cloudflare Pages:**
   ```bash
   npm run build
   wrangler pages deploy dist/ --project-name go-hotel
   ```
   On first run, Wrangler creates the Pages project. Subsequent runs deploy to the same project. The CLI returns a preview URL immediately.

5. **Set up production branch mapping** in the Cloudflare dashboard (Pages → go-hotel → Settings → Builds & deployments): map `main` branch to production. Then `wrangler pages deploy dist/ --project-name go-hotel --branch main` promotes to your production domain (`go-hotel.pages.dev` or a custom domain).

## Out of Scope

The following were not evaluated in this research:
- Docker image configuration
- CI/CD pipeline setup (GitHub Actions wiring)
- Production-scale architecture (multi-region, HA, DR)
