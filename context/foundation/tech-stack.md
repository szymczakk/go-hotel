---
starter_id: vite-react
package_manager: npm
project_name: go-hotel
hints:
  language_family: js
  team_size: solo
  deployment_target: cloudflare-pages
  ci_provider: github-actions
  ci_default_flow: auto-deploy-on-merge
  bootstrapper_confidence: verified
  path_taken: custom
  quality_override: true
  self_check_answers:
    typed: true
    from_official_starter: true
    conventions: true
    docs_current: true
    can_judge_agent: false
  has_auth: false
  has_payments: false
  has_realtime: false
  has_ai: false
  has_background_jobs: false
---

## Why this stack

Solo developer building a 2D browser hotel game — no SSR, no backend, no database in MVP. vite-react is the only registry starter that is a pure client-side SPA shell without meta-framework overhead; it scaffolds Vite + React + TypeScript in one command. React handles the UI layer (HUD, progress bar, unlock buttons) while Phaser.js (added manually post-scaffold) drives the game loop, Canvas rendering, and sprite animation. All four agent-friendly criteria pass except convention_based — acceptable for a game project because structural conventions come from Phaser.js scene architecture, not from web routing. quality_override is recorded; bootstrapper will add Phaser.js-specific conventions to the project instruction file. Deployment targets Cloudflare Pages (free CDN, static asset hosting, global edge) with auto-deploy on merge via GitHub Actions.
