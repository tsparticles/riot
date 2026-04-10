# Architecture

**Analysis Date:** 2026-04-10

## Pattern Overview

**Overall:** Monorepo workspace with package-oriented architecture (publishable component package + legacy package + demo application).

**Key Characteristics:**
- Build orchestration is centralized at workspace root using Lerna/Nx scripts in `package.json`, `lerna.json`, and `nx.json`.
- Runtime feature logic is encapsulated in Riot tag modules (`.riot` files), with the reusable component implementation in `components/riot/src/riot-particles.riot`.
- Demo app composes local Riot components and the workspace package through explicit registration in `apps/riot/src/index.js`.

## Layers

**Workspace Orchestration Layer:**
- Purpose: Coordinate multi-package builds and CI execution.
- Location: `package.json`, `pnpm-workspace.yaml`, `lerna.json`, `nx.json`, `.github/workflows/nodejs.yml`
- Contains: Workspace package definitions, build targets, CI pipeline steps.
- Depends on: `pnpm`, `lerna`, `nx`, GitHub Actions.
- Used by: All packages under `apps/*`, `components/*`, and `components-legacy/*`.

**Reusable Component Package Layer:**
- Purpose: Provide the Riot wrapper for tsParticles as a publishable package.
- Location: `components/riot/`
- Contains: Source tag in `components/riot/src/riot-particles.riot`, build output in `components/riot/dist/riot-particles.js`, package metadata in `components/riot/package.json`.
- Depends on: `@tsparticles/engine` (runtime), Riot compiler/CLI (build).
- Used by: Demo app via `@tsparticles/riot` in `apps/riot/src/index.js` and `apps/riot/src/components/global/my-component/my-component.riot`.

**Legacy Compatibility Package Layer:**
- Purpose: Keep legacy package name and equivalent runtime behavior.
- Location: `components-legacy/riot/`
- Contains: Parallel component source in `components-legacy/riot/src/riot-particles.riot`, package metadata in `components-legacy/riot/package.json`.
- Depends on: Same tsParticles/Riot stack as the modern package.
- Used by: External consumers of `riot-particles` package name.

**Demo Application Layer:**
- Purpose: Demonstrate component integration and serve as integration verification target.
- Location: `apps/riot/`
- Contains: App entry in `apps/riot/src/index.js`, Riot components under `apps/riot/src/components/`, HTML shell in `apps/riot/src/index.html`, bundling config in `apps/riot/webpack.config.js`.
- Depends on: `@tsparticles/riot`, `riot`, `tsparticles`, webpack toolchain.
- Used by: Local development (`start`) and CI build flow (`build:ci`).

## Data Flow

**Component Registration and Mount Flow:**

1. Webpack loads `apps/riot/src/index.js` as configured by `apps/riot/webpack.config.js`.
2. `apps/riot/src/index.js` registers tags (`my-component`, `sidebar`, `user`, `riot-particles`) using Riot `register`.
3. Riot mounts DOM nodes marked with `data-riot-component` from `apps/riot/src/index.html` via `mount('[data-riot-component]')`.

**Particles Initialization Flow:**

1. `apps/riot/src/components/global/my-component/my-component.riot` passes `id`, `options`, `particlesInit`, and `particlesLoaded` props to `<riot-particles>`.
2. `components/riot/src/riot-particles.riot` runs `onMounted(props)`, calls `tsParticles.init()`, then awaits `props.particlesInit(tsParticles)` when provided.
3. The component calls `tsParticles.load({ id, url, options })`, then triggers `props.particlesLoaded(container)` in the resolved callback.

**State Management:**
- Local component state is handled inside Riot tag exports (for example `state` and `this.update(...)` in `apps/riot/src/components/global/sidebar/sidebar.riot`).
- Cross-component shared state store is not present.

## Key Abstractions

**Riot Particle Wrapper Tag:**
- Purpose: Abstract tsParticles lifecycle behind Riot props.
- Examples: `components/riot/src/riot-particles.riot`, `components-legacy/riot/src/riot-particles.riot`
- Pattern: Riot custom tag exposing hook-driven lifecycle (`onMounted`) and callback props (`particlesInit`, `particlesLoaded`).

**Composable Riot App Components:**
- Purpose: Compose page-level demo behavior from small tags.
- Examples: `apps/riot/src/components/global/my-component/my-component.riot`, `apps/riot/src/components/global/sidebar/sidebar.riot`, `apps/riot/src/components/includes/user/user.riot`
- Pattern: Parent-child component composition through `components: { ... }` and props.

**Workspace Package Boundaries:**
- Purpose: Separate reusable library code from demo and legacy compatibility surface.
- Examples: `components/riot/package.json`, `components-legacy/riot/package.json`, `apps/riot/package.json`
- Pattern: One package per directory, wired through workspace dependencies (`"@tsparticles/riot": "workspace:^"` in `apps/riot/package.json`).

## Entry Points

**Demo Runtime Entry:**
- Location: `apps/riot/src/index.js`
- Triggers: Bundled and executed in browser as webpack app entry.
- Responsibilities: Register Riot tags and mount application roots.

**Demo HTML Entry:**
- Location: `apps/riot/src/index.html`
- Triggers: Served/generated by HtmlWebpackPlugin from `apps/riot/webpack.config.js`.
- Responsibilities: Provide mount points with `data-riot-component` attributes.

**Library Component Entry:**
- Location: `components/riot/src/riot-particles.riot`
- Triggers: Imported as `@tsparticles/riot` by consumers.
- Responsibilities: Initialize tsParticles engine and load/destroy containers based on props.

**Legacy Library Component Entry:**
- Location: `components-legacy/riot/src/riot-particles.riot`
- Triggers: Imported as `riot-particles` by legacy consumers.
- Responsibilities: Provide equivalent tsParticles wrapper behavior under legacy package naming.

## Error Handling

**Strategy:** Minimal inline conditional checks with no centralized error pipeline.

**Patterns:**
- Optional callback guards (`if (props.particlesInit)`, `if (props.particlesLoaded)`) in `components/riot/src/riot-particles.riot`.
- Promise continuation using `.then(cb)` on `tsParticles.load(...)` without local `.catch(...)` in `components/riot/src/riot-particles.riot` and `components-legacy/riot/src/riot-particles.riot`.

## Cross-Cutting Concerns

**Logging:** `console.log(container)` is used in demo component callback at `apps/riot/src/components/global/my-component/my-component.riot`.
**Validation:** Prop validation schema is not present; props are consumed directly in `components/riot/src/riot-particles.riot`.
**Authentication:** Not applicable in current architecture.

---

*Architecture analysis: 2026-04-10*
