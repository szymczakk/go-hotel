# Repository Guidelines

Go-hotel is a 2D browser hotel management game (idle + tycoon hybrid) built on Vite 8, React 19, TypeScript 6, and Phaser 4. The entire game runs client-side — no server, no backend, no SSR.

## Hard Rules

- **Do not modify `context/`.** It holds 10x chain metadata (PRD, tech-stack hand-off, shape-notes). It is not game source. Adding game files here or altering its contents breaks the chain trail.
- **No backend, no SSR in MVP.** All game state must live in the browser (localStorage target). Do not introduce API routes, server functions, or external data fetching in MVP scope.
- **`npm run build` must pass before any merge.** It runs `tsc -b && vite build`. TypeScript is strict — `noUnusedLocals`, `noUnusedParameters`, `noFallthroughCasesInSwitch`, `erasableSyntaxOnly`. Build failures are not tolerated.
- **`npm run lint` must exit clean.** Config is at `@eslint.config.js`: typescript-eslint recommended + react-hooks + react-refresh. Fix all warnings, not just errors.

## Project Structure

- `src/` — React UI components (HUD, menus) and Phaser scenes / game logic
- `public/` — static assets served verbatim by Vite
- `index.html` — Vite HTML entry point; Phaser mounts its Canvas here
- `context/` — 10x workflow docs; read-only for game development

See `@context/foundation/prd.md` for the full feature list and the one-sentence business rule that drives the game loop.

## Build, Test, and Dev Commands

- `npm run dev` — start Vite dev server with HMR
- `npm run build` — TypeScript compile + production bundle (the merge-readiness gate)
- `npm run lint` — ESLint across all `.ts` / `.tsx` files
- `npm run preview` — serve the production bundle locally for manual testing

No automated test framework is installed in MVP.

## Coding Style and Naming

Use `.tsx` for React UI components and `.ts` for Phaser scenes and game logic. Target ES2023, module ESNext. PascalCase for React components and Phaser scene class names; camelCase for variables and functions. Do not use `any` — TypeScript strict mode will flag it via `typescript-eslint`.

## Architecture

Phaser 4 owns the game loop and Canvas rendering (scenes, sprites, animations, pathfinding). React wraps the Phaser canvas and renders UI overlays on top. Game state persists to `localStorage`; the key invariant from the PRD: a room cycles through `free → occupied → dirty → free` and earns money on each completed cycle. See `@context/foundation/prd.md` § Business Logic.

## Commit Guidelines

No commit history exists yet. Use Conventional Commits prefixes (`feat:`, `fix:`, `chore:`, `refactor:`) from the first commit to establish a consistent log.
