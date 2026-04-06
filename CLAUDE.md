# CLAUDE.md — project context for Claude and other AI assistants

This file mirrors the intent of **Cursor** project rules in **`.cursor/rules/*.mdc`** so behavior stays consistent across tools (Claude Code, Cursor, etc.). The **canonical extended guide** is **[`AGENTS.md`](./AGENTS.md)**; read it for commands, layout, and workflows.

**Repository:** Excalidraw monorepo (`excalidraw-monorepo`), Yarn **1.x** workspaces, **Node ≥ 18**. Root commands: `yarn start`, `yarn test:all`, `yarn fix`, `yarn build:packages`.

---

## Parallel to Cursor rules

| Topic | Cursor | Same policy here |
|--------|--------|------------------|
| Architecture | `.cursor/rules/project-architecture.mdc` | Canvas 2D for scene; **no** react-konva / fabric / pixi for drawing. Editor state via **`ActionManager`** → **`executeAction`** (see `packages/excalidraw/actions/manager.tsx`), **not** `dispatch()`. **No** Redux / Zustand / MobX for core editor state. `AppState` in `packages/excalidraw/types.ts`. |
| Conventions | `.cursor/rules/conventions.mdc` | Functional components + hooks; named exports; strict TS (`no any`, no `@ts-ignore` where rules forbid); colocated `*.test.tsx` where applicable. |
| Protected files | `.cursor/rules/do-not-touch.mdc` | Do **not** change without explicit approval + `yarn test:all` + manual QA: `packages/excalidraw/scene/Renderer.ts`, `packages/excalidraw/data/restore.ts`, `packages/excalidraw/actions/manager.tsx`, `packages/excalidraw/types.ts`. |
| Testing | `.cursor/rules/testing.mdc` | Vitest (`vitest.config.mts`). Root: `yarn test`, `yarn test:app --watch=false`, `yarn test:all`. |
| Security | `.cursor/rules/security.mdc` | No secrets in source; treat imports/clipboard/collab as untrusted; avoid `eval` / unsafe HTML; no new crypto/parsing deps without review. |
| Host app | `.cursor/rules/excalidraw-app.mdc` | Shell in `excalidraw-app/`; editor logic stays in `packages/excalidraw/`. |
| Data / restore | `.cursor/rules/data-restore.mdc` | Preserve `.excalidraw` compatibility; validate input; test migrations. |
| Element / math | `.cursor/rules/packages-element-math.mdc` | `packages/math` stays framework-agnostic; keep hot paths lean. |
| i18n | `.cursor/rules/i18n-locales.mdc` | `packages/excalidraw/locales/`; `en.json` as key source; valid JSON. |

Cursor-only **slash commands** (not duplicated here as files): `.cursor/commands/start-dev.md`, `verify-ci.md`, `test-watch.md` — use the equivalent **`yarn`** commands from `AGENTS.md`.

---

## What to run before proposing changes

1. **`yarn test:all`** — typecheck, ESLint, Prettier check, Vitest once.  
2. If touching packages and the app acts broken: **`yarn build:packages`** then **`yarn start`**.

---

## Rule validation

Cross-check brittle rule text against code when unsure: **[`dev-docs/ab-validation-rules.md`](./dev-docs/ab-validation-rules.md)** (example: `ActionManager` uses `executeAction`, not `dispatch`).

---

## Dependencies

**No new npm packages** without explicit maintainer / team approval (aligns with `.cursor/rules/project-architecture.mdc` and `AGENTS.md`).
