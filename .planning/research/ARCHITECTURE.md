# Architecture Research

**Domain:** tsParticles Riot modernization and v4 beta migration
**Researched:** 2026-04-10
**Confidence:** HIGH

## Standard Architecture

### System Overview

```text
┌─────────────────────────────────────────────────────────────────────┐
│                    Workspace Orchestration Layer                    │
├─────────────────────────────────────────────────────────────────────┤
│  pnpm workspace  │  Lerna/Nx task graph  │  CI workflow gates      │
└───────────────┬───────────────────────────────┬─────────────────────┘
                │                               │
┌───────────────▼───────────────────────────────▼─────────────────────┐
│                      Package Facade Layer                           │
├─────────────────────────────────────────────────────────────────────┤
│  @tsparticles/riot facade         riot-particles (legacy) facade   │
│  (consumer API contract)          (package-name compatibility)      │
└───────────────┬───────────────────────────────┬─────────────────────┘
                │ shared lifecycle + adapter API│
┌───────────────▼───────────────────────────────▼─────────────────────┐
│                      Shared Core Adapter Layer                      │
├─────────────────────────────────────────────────────────────────────┤
│  components/riot-core                                                 │
│  - normalize props (id/url/options)                                 │
│  - init/load/destroy orchestration                                  │
│  - runtime bridge for v3/v4 package differences                     │
└───────────────┬───────────────────────────────┬─────────────────────┘
                │                               │
┌───────────────▼───────────────────────────────▼─────────────────────┐
│                      Runtime + Verification Layer                    │
├─────────────────────────────────────────────────────────────────────┤
│  tsParticles engine/packages  │  apps/riot demo  │  contract tests   │
└─────────────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|----------------|------------------------|
| Workspace orchestration | Build order, task caching, CI quality gates | Root scripts + Lerna/Nx target graph |
| `@tsparticles/riot` facade | Primary public Riot wrapper API | Thin `.riot` tag delegating to shared core |
| `riot-particles` facade | Legacy package-name compatibility | Thin `.riot` tag delegating to shared core |
| `components/riot-core` | Single source of lifecycle truth for init/load/destroy | TS module(s) with adapter interface |
| Demo/integration app | Runtime verification + usage examples | `apps/riot` registered components + smoke checks |
| Contract test suite | Prevent breaking API contracts during migration | Shared tests executed against both facades |

## Recommended Project Structure

```text
riot/
├── apps/
│   └── riot/                              # integration demo and runtime verification
├── components/
│   ├── riot/                              # modern public facade package (@tsparticles/riot)
│   │   └── src/
│   │       ├── riot-particles.riot        # facade only
│   │       └── bridge.ts                  # package-local wiring to core
│   └── riot-core/                         # new internal shared core package
│       └── src/
│           ├── lifecycle.ts               # mount/load/destroy orchestration
│           ├── adapter.ts                 # v3/v4 runtime abstraction
│           └── contracts.ts               # normalized prop + callback types
├── components-legacy/
│   └── riot/
│       └── src/
│           ├── riot-particles.riot        # legacy facade only
│           └── bridge.ts                  # package-local wiring to core
└── .planning/
    └── research/                          # migration architecture decisions
```

### Structure Rationale

- **Facade packages stay where they are:** keeps current import paths and package ownership stable while internals change.
- **New `components/riot-core` package:** removes duplicated logic between modern and legacy wrappers, reducing migration drift risk.
- **Bridge files per facade:** isolate package-specific imports/exports so v4 beta changes are absorbed without changing consumer API.
- **Contract tests at package boundaries:** enforce identical behavior between modern and legacy wrappers through all phases.

## Architectural Patterns

### Pattern 1: Facade + Shared Core

**What:** Public Riot tags remain thin facades; all lifecycle behavior lives in one shared core package.
**When to use:** Immediately, before dependency upgrades, to reduce duplicate refactor risk.
**Trade-offs:** Adds one package and wiring complexity, but sharply reduces long-term breakage risk.

**Example:**
```typescript
// facade bridge (conceptual)
import { createParticlesLifecycle } from "@tsparticles/riot-core";

export const lifecycle = createParticlesLifecycle({ runtime: "v4-beta" });
```

### Pattern 2: Adapter for Runtime Version Drift

**What:** Core calls a runtime adapter interface instead of calling v3/v4 APIs directly from Riot tags.
**When to use:** During dual-support phases where package names/signatures change.
**Trade-offs:** Slight indirection cost; major gain in controlled migration sequencing.

**Example:**
```typescript
interface RuntimeAdapter {
  init(): Promise<void>;
  load(input: { id: string; url?: string; options?: unknown }): Promise<unknown>;
  destroy(id: string): void;
}
```

### Pattern 3: Strangler Migration by Boundary

**What:** Move one behavior slice at a time (init, load, destroy, callbacks) from wrappers into core, keeping API stable.
**When to use:** Brownfield modernization with existing consumers.
**Trade-offs:** More incremental PRs, but safer rollback and easier regression isolation.

## Data Flow

### Runtime Request Flow

```text
[Consumer Riot Component Props]
    ↓
[Facade Tag: @tsparticles/riot or riot-particles]
    ↓
[Bridge]
    ↓
[Shared Core Lifecycle]
    ↓
[Runtime Adapter (v3/v4)]
    ↓
[tsParticles Engine]
    ↓
[Container Result]
    ↓
[particlesLoaded Callback to Consumer]
```

### Build and Validation Flow

```text
[Root build command]
    ↓
[Build/Test components/riot-core]
    ↓
[Build/Test components/riot + components-legacy/riot]
    ↓
[Build/Test apps/riot integration]
    ↓
[CI gate passes and package publish eligibility]
```

### Key Data Flows

1. **Props normalization flow:** consumer props move from facade to core contracts, then to adapter-specific load input.
2. **Lifecycle callback flow:** engine container result moves from adapter to core, then back to consumer via `particlesLoaded`.
3. **Compatibility flow:** both facade packages call the same core lifecycle, ensuring behavior parity across package names.

## Scaling Considerations

| Scale | Architecture Adjustments |
|-------|--------------------------|
| Current maintainer scale | Keep monorepo; enforce package boundaries and contract tests |
| More wrappers/framework ports | Reuse `riot-core` pattern to avoid per-wrapper lifecycle duplication |
| Broad v4 rollout | Add migration telemetry/log hooks in demo and CI smoke matrix for regression detection |

### Scaling Priorities

1. **First bottleneck:** duplicated behavior between modern/legacy wrappers; fix by extracting shared core first.
2. **Second bottleneck:** runtime API drift across beta updates; fix by adapter interface and pinned compatibility tests.

## Anti-Patterns

### Anti-Pattern 1: Dual Wrapper Divergence

**What people do:** Apply fixes directly to one wrapper and copy manually to the other later.
**Why it's wrong:** Creates behavior mismatch and hidden consumer regressions.
**Do this instead:** Route both wrappers through one shared lifecycle core with boundary tests.

### Anti-Pattern 2: Big-Bang Dependency Upgrade

**What people do:** Upgrade all tsParticles/Riot/runtime dependencies and syntax in one pass.
**Why it's wrong:** Failure surface is too large to isolate; rollback becomes expensive.
**Do this instead:** Migrate in dependency order (core, facades, demo), with passing tests at each boundary.

## Integration Points

### External Services

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| tsParticles engine/packages | Adapter-driven runtime calls from shared core | Keeps API changes out of Riot facades |
| Riot runtime/compiler | Facade tags expose stable props/events | Preserve consumer contracts while modernizing internals |
| CI (GitHub Actions) | Root build pipeline validates package graph order | Use as migration safety gate per phase |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| `apps/riot` ↔ facade packages | Package imports + Riot props/callbacks | Demo acts as integration canary |
| facade packages ↔ `riot-core` | Explicit bridge API | Main seam for modernization without consumer breakage |
| `riot-core` ↔ tsParticles runtime | Adapter interface | Decouples v4 beta churn from package API |

## Suggested Build Order Through Migration Phases

1. **Phase 1 - Safety rails first:** add contract tests around current wrapper behavior and CI ordering; no runtime changes.
2. **Phase 2 - Extract shared core:** move duplicated lifecycle logic into `components/riot-core`; facades become thin delegates.
3. **Phase 3 - Introduce v4 adapter:** implement adapter layer in core and run dual-compat tests to validate parity.
4. **Phase 4 - Modern syntax refactor:** update syntax/tooling in core and facades after boundaries are stable.
5. **Phase 5 - Default to v4 beta path:** switch primary adapter default to v4 beta, retain legacy facade package-name compatibility.
6. **Phase 6 - Cleanup and hardening:** remove dead compatibility branches only after downstream verification window closes.

Dependency implication: each phase depends on the previous boundary being stable; do not modernize facade syntax before shared core and contract tests exist.

## Sources

- `/Users/matteo/Projects/GitHub/tsparticles/riot/.planning/PROJECT.md`
- `/Users/matteo/Projects/GitHub/tsparticles/riot/.planning/codebase/ARCHITECTURE.md`
- `/Users/matteo/Projects/GitHub/tsparticles/riot/.planning/codebase/STRUCTURE.md`

---
*Architecture research for: tsParticles Riot modernization and v4 beta alignment*
*Researched: 2026-04-10*
