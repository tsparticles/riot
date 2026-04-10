# Codebase Structure

**Analysis Date:** 2026-04-10

## Directory Layout

```text
riot/
├── apps/                     # Runnable/demo applications in workspace
│   └── riot/                 # Riot demo app package
├── components/               # Current publishable component packages
│   └── riot/                 # Scoped package @tsparticles/riot
├── components-legacy/        # Legacy package-name compatibility modules
│   └── riot/                 # Unscoped package riot-particles
├── .github/workflows/        # CI workflows
├── .planning/codebase/       # Generated architecture/quality planning docs
├── package.json              # Workspace root scripts and tool dependencies
├── pnpm-workspace.yaml       # Workspace package globs
├── lerna.json                # Lerna orchestration configuration
└── nx.json                   # Nx task defaults and caching config
```

## Directory Purposes

**`apps/riot/`:**
- Purpose: Host the browser demo/integration application for the Riot package.
- Contains: `src/` app source, webpack config, demo HTML/CSS, built output in `dist/`.
- Key files: `apps/riot/src/index.js`, `apps/riot/webpack.config.js`, `apps/riot/src/index.html`, `apps/riot/package.json`.

**`apps/riot/src/components/`:**
- Purpose: Define demo Riot tags used by mounted app.
- Contains: Global and feature-like component folders.
- Key files: `apps/riot/src/components/global/my-component/my-component.riot`, `apps/riot/src/components/global/sidebar/sidebar.riot`, `apps/riot/src/components/includes/user/user.riot`.

**`components/riot/`:**
- Purpose: Source of the primary reusable Riot tsParticles component package.
- Contains: Library source tag, package metadata, docs/changelog, build artifact folder.
- Key files: `components/riot/src/riot-particles.riot`, `components/riot/package.json`, `components/riot/README.md`, `components/riot/dist/riot-particles.js`.

**`components-legacy/riot/`:**
- Purpose: Maintain legacy distribution package (`riot-particles`) with equivalent implementation.
- Contains: Parallel Riot component source and package metadata.
- Key files: `components-legacy/riot/src/riot-particles.riot`, `components-legacy/riot/package.json`.

**`.github/workflows/`:**
- Purpose: Define automated CI execution.
- Contains: GitHub Actions YAML.
- Key files: `.github/workflows/nodejs.yml`.

## Key File Locations

**Entry Points:**
- `apps/riot/src/index.js`: Demo app runtime bootstrap (register + mount Riot tags).
- `apps/riot/src/index.html`: Demo mount root definitions consumed by Riot runtime.
- `components/riot/src/riot-particles.riot`: Primary library entry tag.
- `components-legacy/riot/src/riot-particles.riot`: Legacy-compatible library entry tag.

**Configuration:**
- `package.json`: Root build scripts (`build`, `build:ci`, `build:lerna`, `build:nx`) and workspace definitions.
- `pnpm-workspace.yaml`: Workspace package patterns (`apps/*`, `components/*`, `components-legacy/*`).
- `lerna.json`: Lerna package graph and release/version behavior.
- `nx.json`: Nx target caching defaults.
- `apps/riot/webpack.config.js`: Demo bundler rules and dev server behavior.

**Core Logic:**
- `components/riot/src/riot-particles.riot`: tsParticles init/load/destroy lifecycle wrapper.
- `apps/riot/src/components/global/my-component/my-component.riot`: Consumer usage pattern for `riot-particles` props/callbacks.
- `apps/riot/src/components/global/sidebar/sidebar.riot`: Demo stateful Riot component behavior.

**Testing:**
- `apps/riot/src/components/**/*.spec.js`: Demo component specs.
- `components/riot/test.js`: Library component spec.

## Naming Conventions

**Files:**
- Riot tags use kebab-case filenames: `riot-particles.riot`, `my-component.riot`.
- Tests use `.spec.js` suffix for app components: `sidebar.spec.js`, `user.spec.js`.
- Root/demo script files use lower-case names: `index.js`, `webpack.config.js`.

**Directories:**
- Workspace package grouping directories are plural and semantic: `apps/`, `components/`, `components-legacy/`.
- Demo component hierarchy groups by scope: `apps/riot/src/components/global/` and `apps/riot/src/components/includes/`.

## Where to Add New Code

**New Feature:**
- Primary code: add reusable Riot wrapper features in `components/riot/src/`.
- Demo integration usage: add or update demo behavior in `apps/riot/src/components/`.
- Tests: place demo behavior tests adjacent to component as `*.spec.js` in `apps/riot/src/components/**/`; place library package tests in `components/riot/` following `components/riot/test.js` pattern.

**New Component/Module:**
- Reusable published component: implement in `components/riot/src/` and export via package build pipeline.
- Demo-only component: implement under `apps/riot/src/components/` using existing folder segmentation (`global` or `includes`).

**Utilities:**
- Shared helper for library component logic: colocate inside `components/riot/src/`.
- Demo-only helper: colocate within `apps/riot/src/` near consuming components.

## Special Directories

**`apps/riot/dist/`:**
- Purpose: Webpack-generated demo bundles.
- Generated: Yes.
- Committed: Yes (directory is present in repository state).

**`components/riot/dist/`:**
- Purpose: Compiled library output (`main` target in `components/riot/package.json`).
- Generated: Yes.
- Committed: Yes (directory is present in repository state).

**`node_modules/`:**
- Purpose: Installed dependencies.
- Generated: Yes.
- Committed: No (`node_modules/` ignored in `.gitignore`).

**`.planning/codebase/`:**
- Purpose: Generated codebase mapping documents for planning/execution commands.
- Generated: Yes.
- Committed: Yes (intended output location for these docs).

---

*Structure analysis: 2026-04-10*
