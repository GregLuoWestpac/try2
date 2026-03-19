# Copilot Instructions for W1C2 Workspace

These instructions help AI coding agents be immediately productive across this multi-microfrontend workspace. Follow the project-specific workflows and conventions below.

## Big Picture
- Architecture: Multiple microfrontends (e.g., `galaxy-account`, `galaxy-entitlements`, `galaxy-payment`, `galaxymakerchecker-v1-widget`) plus a consuming SPA (`galaxy-singlepageapplication`). Each MFE has `api` (Express) and `ui` (React/Nx) workspaces.
- Contracts: OpenAPI-driven EXP layer. Source in [galaxy-account-microfrontend/openapi_ref.yaml](galaxy-account-microfrontend/openapi_ref.yaml) → dereferenced [galaxy-account-microfrontend/openapi.yaml](galaxy-account-microfrontend/openapi.yaml).
- Mesh: Components defined in Mesh YAML, e.g. [galaxy-account-microfrontend/component.yaml](galaxy-account-microfrontend/component.yaml), and built/deployed with Mesh CLI.
- Consumption: SPA consumes MFE `remoteEntry.js`; register SPA in [galaxy-account-microfrontend/consumption.yaml](galaxy-account-microfrontend/consumption.yaml).

## Node & Tooling
- Node: Always use Node 22. On macOS, run `nvm use 22` before any command.
- Nx: MFEs use Nx with workspaces `api` and `ui`; see [galaxy-account-microfrontend/nx.json](galaxy-account-microfrontend/nx.json).
- Testing: Jest with coverage via Nx; see [galaxy-account-microfrontend/jest.config.js](galaxy-account-microfrontend/jest.config.js).

## Dev Workflow (per MFE)
- Install: From the MFE root, `npm ci`. VS Code task "Install Dependencies" runs `nvm use 22 && npm ci` at workspace root.
- Clean install: Use task "Clean and Install Dependencies" or run `rm -rf node_modules && npm ci` (UI/API too).
- Run locally: `npm start` (executes `nx run-many -t start --skip-nx-cache`). UI serves `remoteEntry.js`; API runs on `localhost:3002` (see `app-config.json`). Verify API health: `http://localhost:3002/exp/[componentName]/reference/v2/healthcheck`.
- Tests: `npm run test` for all (Nx run-many). Update snapshots: `npm run test:update`. UI-only: `npx nx run <componentName>:test`.

## Contracts & Codegen (typical sequence)
- Validate contract: `npm run contract.validate` (validates [openapi_ref.yaml](galaxy-account-microfrontend/openapi_ref.yaml)).
- Dereference: `npm run contract.dereference` → writes [openapi.yaml](galaxy-account-microfrontend/openapi.yaml).
- Generate EXP clients/routes:
  - `npm run api-clients` → [api/server/generated/api-clients](galaxy-account-microfrontend/api/server/generated/api-clients)
  - `npm run routes` → [api/server/generated/routes](galaxy-account-microfrontend/api/server/generated/routes)
  - `npm run policies` → entitlements wiring from [dependencies.yaml](galaxy-account-microfrontend/dependencies.yaml)
- Redux artifacts: `npm run redux-artifacts` → [ui/src/store/generated](galaxy-account-microfrontend/ui/src/store/generated).
- Swagger sync: `npm run mv-swagger` to refresh shared swagger assets.

## Mocking & Local APIs
- Mockoon: `npm run mockoon` uses env data under `api/env/*/mockoon-data*.json` (example PAC: [galaxymakerchecker-v1-widget/api/env/sit-w1c2/mockoon-data-pac.json](galaxymakerchecker-v1-widget/api/env/sit-w1c2/mockoon-data-pac.json)).
- Static downstream mocks: `npm run mock.downstream` (serves `api/mock-api/static-mock` on port 8002).

## Build & Deploy (Mesh)
- Build all UI: `npm run build` (Nx run-many with cache, outputs under `build/ui`).
- Mesh CLI: use VS Code tasks or scripts:
  - Task "Mesh Build": `mesh build --component_name galaxy-account --revision <rev> -v`.
  - Task "Mesh Deploy": `mesh deploy --component_name galaxy-account --environment <env> [--swimlane <lane>] --build_number <num> --wait`.
  - Script: [galaxy-account-microfrontend/mesh_build_and_deploy.sh](galaxy-account-microfrontend/mesh_build_and_deploy.sh) maps inputs (e.g. `sit-w1c2`) to environment/swimlane, extracts build number, and runs deploy.
  - Check status: `mesh status --component_name galaxy-account`.

## Conventions & Patterns
- Workspaces: each MFE root `package.json` declares `workspaces: ["api","ui"]` and `"engines": {"node": ">=22"}`.
- Nx plugins: webpack and jest configured in [galaxy-account-microfrontend/nx.json](galaxy-account-microfrontend/nx.json).
- CI-friendly tests: jest runs with coverage, `passWithNoTests: true` set via Nx target defaults.
- SPA: `galaxy-singlepageapplication` uses CRA; it consumes MFE runtime bundles (`remoteEntry.js`). See [galaxy-singlepageapplication/README.md](galaxy-singlepageapplication/README.md).

## Component-Specific Guidance
- Some MFEs include their own Copilot instructions: 
  - [galaxy-account-microfrontend/.github/copilot-instructions.md](galaxy-account-microfrontend/.github/copilot-instructions.md)
  - [galaxy-entitlements-microfrontend/.github/copilot-instructions.md](galaxy-entitlements-microfrontend/.github/copilot-instructions.md)
  - [galaxy-payment-microfrontend/.github/copilot-instructions.md](galaxy-payment-microfrontend/.github/copilot-instructions.md)
  - [multiverse-ui-common-library/.github/copilot-instructions.md](multiverse-ui-common-library/.github/copilot-instructions.md)
  - [galaxy-singlepageapplication/.github/copilot-instructions.md](galaxy-singlepageapplication/.github/copilot-instructions.md)

## VS Code Tasks (workspace root)
- Clean Node Modules, Install Dependencies, Clean and Install Dependencies.
- Mesh Build, Mesh Deploy, Mesh Build and Deploy.
- Update Galaxy Common Components.

If any section is unclear or missing (e.g., per-MFE ports, app-config paths, or additional Mesh flags), let me know and I’ll refine this file with specifics.