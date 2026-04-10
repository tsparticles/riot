# External Integrations

**Analysis Date:** 2026-04-10

## APIs & External Services

**Particle Engine Runtime:**
- tsParticles engine - Client-side particle container lifecycle and rendering in `components/riot/src/riot-particles.riot` and `components-legacy/riot/src/riot-particles.riot`.
  - SDK/Client: `@tsparticles/engine` (`components/riot/package.json`, `components-legacy/riot/package.json`)
  - Auth: Not applicable

**Remote Particle Config Loading (consumer-provided):**
- Arbitrary JSON URL support via component prop (`url`) passed to `tsParticles.load(...)` in `components/riot/src/riot-particles.riot` and `components-legacy/riot/src/riot-particles.riot`.
  - SDK/Client: `@tsparticles/engine` loader API
  - Auth: Not handled by this repository; consumer must provide reachable URL and any auth strategy externally.

**Community/Documentation Links:**
- External links to project website/Discord/Telegram/CodePen in `README.md` and `components/riot/README.md`.
  - SDK/Client: Not applicable
  - Auth: Not applicable

## Data Storage

**Databases:**
- Not detected.
  - Connection: Not applicable
  - Client: Not applicable

**File Storage:**
- Local filesystem only for build outputs (`dist/` from package build scripts in `components/riot/package.json` and `components-legacy/riot/package.json`).

**Caching:**
- Build/test cache through Nx local cache under `node_modules/.cache/nx/` (configured by `nx.json`), no external cache service detected.

## Authentication & Identity

**Auth Provider:**
- None detected.
  - Implementation: Not applicable

## Monitoring & Observability

**Error Tracking:**
- None detected (no Sentry/Bugsnag/etc. integrations in manifests/source).

**Logs:**
- Console logging in demo component callback (`particlesLoaded`) at `apps/riot/src/components/global/my-component/my-component.riot`.

## CI/CD & Deployment

**Hosting:**
- Not explicitly configured in this repository.

**CI Pipeline:**
- GitHub Actions workflow in `.github/workflows/nodejs.yml`.
  - Uses `actions/checkout@v4`, `actions/setup-node@v4`, `pnpm/action-setup@v3.0.0`.
  - Runs `pnpm install` and `pnpm run build:ci`.

## Environment Configuration

**Required env vars:**
- Not detected in source or configuration (`apps/riot/src/**/*.js`, `components/**/src/*.riot`, manifests/config files).

**Secrets location:**
- No secrets files detected from root `.env*` scan.
- `.gitignore` includes secret-prone patterns such as `*.pfx` and publish profiles in `.gitignore`, but no active secret management configuration file is present.

## Webhooks & Callbacks

**Incoming:**
- None detected.

**Outgoing:**
- None detected (no webhook emitters, HTTP client integrations, or backend callback handlers found).

---

*Integration audit: 2026-04-10*
