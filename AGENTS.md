# Repository Guidelines

## Project Structure & Module Organization
- Vite-powered Vue 3 app; all runtime code sits in `src/`.
- `src/main.js` bootstraps Vue, Pinia stores from `src/stores/`, and routing from `src/router/index.js`.
- Smart, shareable pieces live in `src/components/`; routed pages belong in `src/view/` (one folder per screen).
- Keep board assets or static JSON under `public/`; they are copied verbatim at build time.
- Generated bundles land in `dist/` (git-ignored); never edit files there.

## Build, Test, and Development Commands
- `npm install` — install Node 20+ dependencies recorded in `package-lock.json`.
- `npm run dev` — launches Vite dev server with hot reload and auto-opens the board UI.
- `npm run build` — emits optimized static assets into `dist/`; run before tagging a release.
- `npm run preview` — serves the latest production build for smoke-testing.
- `npm run lint` — runs ESLint (Vue essential + JS recommended) in fix-and-cache mode; use before every commit.

## Coding Style & Naming Conventions
- Follow the flat ESLint config in `eslint.config.js`: 2-space indentation, single quotes, semicolons omitted.
- Vue SFCs should prefer `<script setup>` and PascalCase filenames (e.g., `GobangBoard.vue`); composables use `useXyz`.
- Pinia stores export `useThingStore` helpers from `src/stores/thing.js`.
- Route ids are kebab-case, matching folder names in `src/view/`. Keep constants and enums in dedicated modules under `src/components` or `/stores` to avoid circular imports.

## Testing Guidelines
- No automated suite is wired up yet; when introducing Vitest + Vue Test Utils, co-locate specs as `*.spec.js` beside the component.
- Prefer shallow component tests for board logic and Pinia store tests for move validation rules.
- Manual regression passes should at least cover opening `/` via `npm run dev`, creating a room, and completing a five-in-a-row scenario.

## Commit & Pull Request Guidelines
- History shows Conventional Commits (`feat: …`, `fix: …`); continue using that format for clarity and changelog automation.
- Each PR should: (1) describe the user-facing change, (2) link the related issue/task, (3) list commands run (`npm run dev`, `npm run lint`, etc.), and (4) attach before/after screenshots for UI-facing tweaks.
- Keep PRs scoped to one feature or fix; start from an updated `main`, re-run `npm run build` for anything touching production output, and resolve lint warnings before requesting review.
