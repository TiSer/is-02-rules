# A/B validation — Cursor rules vs. codebase

This document records **A/B validation** for project rules: a **stated rule (A)** is checked against **what the repository actually implements (B)** using grep and source inspection.

---

## Rule under test

**File:** `.cursor/rules/project-architecture.mdc`  
**Section:** State Management (as validated on 2026-04-06)

**Original claim (variant A):**  
“State updates: `actionManager.dispatch()` ONLY”

**Competing hypothesis (variant B):**  
The Excalidraw `ActionManager` class exposes a different public API for running actions (e.g. `executeAction`), and the codebase calls that API—not `dispatch`.

---

## Test scenario

1. **Objective:** Determine whether `actionManager.dispatch` appears in editor code and whether `ActionManager` defines `dispatch`.
2. **Commands run** (from repository root):

   ```bash
   rg 'actionManager\.dispatch' packages/excalidraw --glob '*.{ts,tsx}'
   rg 'executeAction' packages/excalidraw/actions/manager.tsx
   rg 'actionManager\.executeAction|\.actionManager\.executeAction' packages/excalidraw --glob '*.{ts,tsx}' --count
   ```

3. **Source check:** Open `packages/excalidraw/actions/manager.tsx` and list public methods used for running actions.

---

## Results

### Variant A — Rule text (`dispatch`)

| Check | Result |
|-------|--------|
| `actionManager.dispatch` in `packages/excalidraw` | **No matches** |
| `dispatch` as a method on `ActionManager` in `manager.tsx` | **Not present** (the class has `executeAction`, `handleKeyDown`, `renderAction`, etc.) |

**Interpretation:** The original wording **does not match** this fork’s implementation. Assistants following “call `dispatch()`” would search for an API that does not exist on `ActionManager`.

### Variant B — Codebase (`executeAction`)

| Check | Result |
|-------|--------|
| `executeAction` in `packages/excalidraw/actions/manager.tsx` | **Present** — method at line ~132 (`executeAction<T extends Action>(...)`) |
| Call sites `actionManager.executeAction` / `app.actionManager.executeAction` | **Many files** (components, tests, wysiwyg, command palette, etc.) |

**Interpretation:** State transitions driven through the action system use **`executeAction`**, consistent with the `ActionManager` implementation.

---

## Conclusion

- **Variant A (dispatch-only)** is **invalid** for this repository: there is **no** `actionManager.dispatch()` API in `packages/excalidraw`.
- **Variant B (executeAction)** is **valid**: it matches `ActionManager` and widespread usage.

**Action taken:** `.cursor/rules/project-architecture.mdc` was updated to describe **`actionManager.executeAction(...)`** and to point at `packages/excalidraw/actions/manager.tsx`, so the rule aligns with code and avoids misleading agents.

**How to re-run this check:** Repeat the `rg` commands in the test scenario after large refactors; if `ActionManager` is renamed or split, update the rule and this note.

---

## Follow-up A/B ideas (not yet run)

| Rule | Possible A/B check |
|------|---------------------|
| `conventions.mdc` — “named exports only” | `rg '^export default'` in `packages/**/*.tsx` — count default exports vs rule |
| `security.mdc` | `rg 'dangerouslySetInnerHTML|eval\(' packages/excalidraw excalidraw-app` — ensure new usages are reviewed |
