# AGENTS.md — guidance for automated assistants and contributors

This repository is the **Excalidraw** monorepo (`name`: `excalidraw-monorepo` in root `package.json`): a hand-drawn style whiteboard with a publishable editor package and a first-party web app. Official upstream docs: [docs.excalidraw.com](https://docs.excalidraw.com), contributing: [Contributing](https://docs.excalidraw.com/docs/introduction/contributing).

**Cross-tool context:** **[`CLAUDE.md`](./CLAUDE.md)** summarizes the same policies as **`.cursor/rules/`** for Claude Code and other assistants that do not read Cursor rule files.

---

## 1. Overview

| Item | Detail |
|------|--------|
| **Monorepo tool** | Yarn **1.x** workspaces (`packageManager`: `yarn@1.22.22`) |
| **Workspaces** | `excalidraw-app`, `packages/*`, `examples/*` |
| **Node** | `>= 18` (engines in root `package.json`) |
| **Primary languages** | TypeScript, TSX |
| **App shell** | Vite-based host in `excalidraw-app/` |
| **Core editor package** | `packages/excalidraw` (published as `@excalidraw/excalidraw`) |
| **Tests** | Vitest (`vitest.config.mts` at repo root) |

---

## 2. Repository structure

| Path | Role |
|------|------|
| **`excalidraw-app/`** | Deployable web app (Vite, PWA, collaboration, Firebase, etc.). Run via `yarn start` from the repo root. |
| **`packages/common/`** | Shared utilities consumed by other packages. |
| **`packages/element/`** | Element model, geometry, bindings (`@excalidraw/element`). |
| **`packages/math/`** | Math primitives (`@excalidraw/math`) — keep framework-agnostic. |
| **`packages/excalidraw/`** | Main editor: React UI, canvas/scene rendering, actions, data, locales, tests. |
| **`packages/utils/`** | Additional shared helpers (see its `package.json` for scope). |
| **`examples/`** | Sample integrations (e.g. `with-script-in-browser`, `with-nextjs`). |
| **`scripts/`** | Release, locales coverage, and other tooling. |
| **`vitest.config.mts`** | Vitest + path aliases for `@excalidraw/*`. |
| **`tsconfig.json`** | Root TypeScript project (`yarn test:typecheck` runs `tsc`). |
| **`.cursor/rules/*.mdc`** | Cursor project rules (architecture, testing, security, etc.). |
| **`.cursor/commands/*.md`** | Cursor slash commands (`/start-dev`, `/verify-ci`, `/test-watch`). |

---

## 3. Prerequisites and setup

1. Install **Node.js ≥ 18** and **Yarn 1** (Classic), matching `packageManager` / `engines`.
2. From the **repository root**, install dependencies:

   ```bash
   yarn install
   ```

3. Optional: Husky is wired via `prepare` — `yarn install` runs `husky install` when applicable.

---

## 4. Development workflow

| Goal | Command (from repo root) |
|------|---------------------------|
| **Dev server (main app)** | `yarn start` → runs `yarn --cwd ./excalidraw-app start` |
| **Rebuild workspace packages** | `yarn build:packages` — `common` → `math` → `element` → `excalidraw` (ESM builds) |
| **Example: script in browser** | `yarn start:example` — builds packages then starts `examples/with-script-in-browser` |
| **Production-like local** | `yarn start:production` (see `excalidraw-app` scripts) |

If you change low-level packages and see resolution issues, run `yarn build:packages` before `yarn start`.

---

## 5. Build and release

| Goal | Command |
|------|---------|
| **Full app build** | `yarn build` — delegates to `excalidraw-app` |
| **Build app artifact only** | `yarn build:app` |
| **Docker-related app build** | `yarn build:app:docker` |
| **Individual packages** | `yarn build:common`, `yarn build:math`, `yarn build:element`, `yarn build:excalidraw` |
| **Clean build outputs** | `yarn rm:build` |
| **Release scripts** | `yarn release`, `yarn release:test`, `yarn release:next`, `yarn release:latest` |

---

## 6. Testing, linting, and formatting

| Goal | Command |
|------|---------|
| **Full CI-equivalent gate** | `yarn test:all` — runs `test:typecheck` → `test:code` → `test:other` → `test:app --watch=false` |
| **Typecheck only** | `yarn test:typecheck` (`tsc`) |
| **ESLint** | `yarn test:code` (`eslint --max-warnings=0 --ext .js,.ts,.tsx .`) |
| **Prettier check** | `yarn test:other` |
| **Vitest (watch)** | `yarn test` or `yarn test:app` |
| **Vitest once** | `yarn test:app --watch=false` |
| **Snapshot update** | `yarn test:update` |
| **Coverage** | `yarn test:coverage` / `yarn test:coverage:watch` |
| **Vitest UI** | `yarn test:ui` |
| **Auto-fix** | `yarn fix` — Prettier write + ESLint fix |

Run **`yarn test:all`** before opening a PR.

---

## 7. Architecture expectations (editor)

These match `.cursor/rules/project-architecture.mdc` and related rules:

- **State**: Editor state is driven by the Excalidraw **action manager** (`ActionManager` in `packages/excalidraw/actions/manager.tsx`). Run user-visible actions via **`executeAction`** (there is no `actionManager.dispatch()` on this class). Do not introduce Redux, Zustand, or MobX for core editor state.
- **Rendering**: Scene content is drawn with **Canvas 2D**, not React DOM; avoid adding `react-konva`, `fabric.js`, or `pixi.js` for the canvas pipeline.
- **Types**: `AppState` and core types live in **`packages/excalidraw/types.ts`**.
- **Host app**: `excalidraw-app` may use its own patterns (e.g. Jotai for shell UI); keep editor concerns in `packages/excalidraw`.

---

## 8. Cursor rules and commands

- **Rules** (`.cursor/rules/`): Always-on and glob-scoped MDC files — architecture, conventions, testing, security, i18n, `excalidraw-app`, `data/`, `element`/`math`, and **`do-not-touch.mdc`** for protected paths.
- **Commands** (`.cursor/commands/`):
  - `/start-dev` — local dev server steps
  - `/verify-ci` — `yarn test:all` breakdown
  - `/test-watch` — Vitest-focused workflow

Follow **`.cursor/rules/do-not-touch.mdc`** for files that require explicit approval and extra verification.

- **Rule vs. code checks:** See **`dev-docs/ab-validation-rules.md`** for documented A/B validation (e.g. rule wording vs. `rg` / source inspection).

---

## 9. Protected and high-risk areas

Do **not** modify these without explicit approval and full regression checks (see `do-not-touch.mdc`):

| File | Risk |
|------|------|
| `packages/excalidraw/scene/Renderer.ts` | Render pipeline |
| `packages/excalidraw/data/restore.ts` | File format / restore compatibility |
| `packages/excalidraw/actions/manager.tsx` | Action dispatch |
| `packages/excalidraw/types.ts` | Core type definitions |

Changes require: understanding dependents, **`yarn test:all`**, and manual QA where noted in rules.

---

## 10. Security and dependencies

- Do not commit secrets, API keys, or Firebase private config; use existing env / deployment patterns.
- Treat imported `.excalidraw` JSON, clipboard payloads, and collaboration data as **untrusted**; use established validation/restore paths in `packages/excalidraw/data/`.
- **No new npm packages** without explicit approval (team / maintainer process).

---

## 11. Internationalization

- Locale JSON lives under **`packages/excalidraw/locales/`**; **`en.json`** is the usual source for keys (other locales often sync via Crowdin — see project docs).
- Helper scripts: `yarn locales-coverage`, `yarn locales-coverage:description`.

---

## 12. Contributing and documentation

- High-level contributing pointer: [Contributing](https://docs.excalidraw.com/docs/introduction/contributing).
- Development setup for this repo: [Development](https://docs.excalidraw.com/docs/introduction/development).
- Root **`CONTRIBUTING.md`** links to the same documentation hub.

---

## 13. Troubleshooting

| Issue | Suggestion |
|-------|------------|
| **Module resolution / stale package output** | `yarn build:packages`, then retry `yarn start`. |
| **Lint / format failures** | `yarn fix` then `yarn test:all`. |
| **Type errors** | `yarn test:typecheck` for full `tsc` output. |
| **Clean install** | `yarn clean-install` (removes workspace `node_modules` and reinstalls). |

---

## 14. Quick reference (copy-paste)

```bash
yarn install          # deps
yarn start            # dev server
yarn test:all         # full check before PR
yarn fix              # prettier + eslint --fix
yarn build:packages   # rebuild @excalidraw packages (ESM)
```
