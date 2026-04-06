# Verify like CI (full check)

Run the full pre-merge validation suite before a PR or after large refactors.

## Do this

From the repository root:

```bash
yarn test:all
```

This script runs, in order:

| Step | Script | What it runs |
|------|--------|----------------|
| 1 | `yarn test:typecheck` | `tsc` |
| 2 | `yarn test:code` | ESLint (`.js`, `.ts`, `.tsx`, max warnings 0) |
| 3 | `yarn test:other` | Prettier `--list-different` on configured globs |
| 4 | `yarn test:app --watch=false` | Vitest once (no watch) |

## If something fails

- **Typecheck**: fix TypeScript errors; re-run `yarn test:typecheck` for a fast loop.
- **ESLint**: auto-fix where safe: `yarn fix:code`, then re-run `yarn test:code`.
- **Prettier**: `yarn fix:other`, then re-run `yarn test:other`.
- **Vitest**: run a single file or pattern with Vitest’s CLI options if needed.

## One-shot “fix formatting + lint” (optional)

```bash
yarn fix
```

Then run `yarn test:all` again to confirm.
