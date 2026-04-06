# Test watch (Vitest)

Run unit tests in watch mode while editing—default workflow for `packages/*` and app code covered by Vitest.

## Do this

From the repository root:

```bash
yarn test
```

This runs `vitest` (watch mode is the typical default for local iteration).

## Useful variants

**Single run (no watch), e.g. for scripts or CI-like local checks:**

```bash
yarn test:app --watch=false
```

**Update snapshots** (only when you intentionally changed expected output):

```bash
yarn test:update
```

**Coverage:**

```bash
yarn test:coverage
```

**Vitest UI:**

```bash
yarn test:ui
```

## Notes

- Config: `vitest.config.mts` (aliases for `@excalidraw/*` packages).
- For the **full** gate including typecheck, ESLint, and Prettier, use `/verify-ci` instead of only Vitest.
