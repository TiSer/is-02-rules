# Start dev server

Run the Excalidraw web app locally for interactive development.

## Do this

1. Use the repository root as the working directory (this monorepo uses Yarn workspaces).
2. Ensure dependencies are installed: `yarn install` (Node **>= 18** per `package.json`).
3. Start the Vite dev server:

```bash
yarn start
```

This runs `yarn --cwd ./excalidraw-app start`.

## When packages need a rebuild

If you changed low-level packages and see resolution or build issues, build workspace packages first, then start:

```bash
yarn build:packages
yarn start
```

## Orientation

- Host app: `excalidraw-app/`
- Core editor package: `packages/excalidraw/`
- Other packages: `packages/common`, `packages/element`, `packages/math`

Respect project rules: canvas rendering and `actionManager` state (see `.cursor/rules/project-architecture.mdc`).
