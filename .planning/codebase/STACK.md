# Technology Stack

**Analysis Date:** 2026-04-10

## Languages

**Primary:**
- JavaScript (ES modules/CommonJS) - Runtime and build scripts in `apps/riot/src/index.js`, `apps/riot/webpack.config.js`, and workspace manifests in `package.json` files.
- Riot single-file component syntax (`.riot`) - Component implementation in `components/riot/src/riot-particles.riot`, `components-legacy/riot/src/riot-particles.riot`, and demo UI files under `apps/riot/src/components/`.

**Secondary:**
- YAML - Workspace and CI configuration in `pnpm-workspace.yaml` and `.github/workflows/nodejs.yml`.
- JSON - Tooling/orchestration configuration in `package.json`, `nx.json`, `lerna.json`, `renovate.json`, and `components/riot/typedoc.json`.

## Runtime

**Environment:**
- Node.js 20 in CI (`.github/workflows/nodejs.yml`).

**Package Manager:**
- pnpm 10.33.0 pinned at workspace root (`package.json` `packageManager`).
- Lockfile: present (`pnpm-lock.yaml`).

## Frameworks

**Core:**
- Riot `^7.1.0` - UI/component runtime in `components/riot/package.json`, `components-legacy/riot/package.json`, and `apps/riot/package.json`.
- tsParticles engine `^3.9.1` - Particle rendering core used by Riot component in `components/riot/src/riot-particles.riot` and `components-legacy/riot/src/riot-particles.riot`.

**Testing:**
- Mocha `^10.2.0` + Chai `^4.3.7` + jsdom/jsdom-global - Test runner/assertions/DOM simulation in `components/riot/package.json` and `apps/riot/package.json`.
- NYC `^15.1.0` - Coverage tooling via scripts in `apps/riot/package.json`.

**Build/Dev:**
- `@riotjs/cli` + `@riotjs/compiler` - Build Riot component packages (`riot -f umd src -o dist`) in `components/riot/package.json` and `components-legacy/riot/package.json`.
- Webpack 5 toolchain - Demo app bundling/dev server in `apps/riot/package.json` and `apps/riot/webpack.config.js`.
- Lerna `^6.3.0` + Nx `^20.7.2` - Monorepo orchestration in root `package.json`, `lerna.json`, and `nx.json`.
- Prettier + `@tsparticles/prettier-config` - Formatting in `components/riot/package.json` and `components-legacy/riot/package.json`.
- Husky + Commitlint - Commit message checks via `.husky/commit-msg` and root `package.json`.

## Key Dependencies

**Critical:**
- `@tsparticles/engine` `^3.9.1` - Core particle API consumed directly by Riot wrapper component (`components/riot/src/riot-particles.riot`).
- `@tsparticles/riot` workspace dependency - Demo app consumes the local package (`apps/riot/package.json`).
- `riot` `^7.1.0` - Required peer/runtime for all Riot components (`components/riot/package.json`, `components-legacy/riot/package.json`).

**Infrastructure:**
- `webpack`, `webpack-cli`, `webpack-dev-server`, `@riotjs/webpack-loader`, `html-webpack-plugin`, `mini-css-extract-plugin`, `css-loader` - Demo build/runtime infra in `apps/riot/package.json` and `apps/riot/webpack.config.js`.
- `lerna`, `nx`, `pnpm` - Workspace build orchestration in root `package.json`, `lerna.json`, `nx.json`, and `pnpm-workspace.yaml`.
- `renovate` config - Dependency update automation configuration in `renovate.json`.

## Configuration

**Environment:**
- No required environment variables detected in source/config (`apps/riot/src/**/*.js`, `components/**/src/*.riot`).
- `.env` files: Not detected at repository root (`.env*` glob returned no files).

**Build:**
- Workspace orchestration: `package.json`, `pnpm-workspace.yaml`, `lerna.json`, `nx.json`.
- Component package build scripts: `components/riot/package.json`, `components-legacy/riot/package.json`.
- Demo app build config: `apps/riot/webpack.config.js`.
- CI pipeline config: `.github/workflows/nodejs.yml`.

## Platform Requirements

**Development:**
- Node.js + pnpm workspace tooling required (`package.json`, `pnpm-workspace.yaml`).
- Riot compiler/webpack toolchain required for local demo (`apps/riot/package.json`, `apps/riot/webpack.config.js`).

**Production:**
- Library artifacts are produced to `dist/` in component packages (`components/riot/package.json`, `components-legacy/riot/package.json`) for package distribution.
- No server runtime or deployment platform configuration detected in repository manifests/workflows (`.github/workflows/nodejs.yml`, root `package.json`).

---

*Stack analysis: 2026-04-10*
