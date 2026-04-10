# tsParticles Riot Modernization (v4 Beta Alignment)

## What This Is

This project modernizes the `tsparticles` Riot workspace so it aligns with the `4.0.0` beta line currently under development. It focuses on upgrading package versions, migrating to more modern syntax and patterns, and keeping the Riot wrappers reliable for both current and legacy package consumers. The primary users are maintainers and downstream developers integrating tsParticles with Riot.

## Core Value

Ship a stable, modern Riot integration that is fully aligned with the tsParticles 4.0.0 beta ecosystem.

## Requirements

### Validated

- ✓ Riot wrapper package exists and integrates tsParticles lifecycle hooks (`components/riot/src/riot-particles.riot`) — existing
- ✓ Legacy Riot wrapper package exists for backward package-name compatibility (`components-legacy/riot/src/riot-particles.riot`) — existing
- ✓ Demo app validates package integration and local development flow (`apps/riot/src/index.js`) — existing
- ✓ Monorepo build orchestration is established with pnpm/Lerna/Nx (`package.json`, `pnpm-workspace.yaml`, `lerna.json`, `nx.json`) — existing

### Active

- [ ] Upgrade workspace and package dependencies to latest compatible versions, centered on tsParticles `4.0.0` beta packages.
- [ ] Migrate code to modern syntax and conventions across component and app codepaths.
- [ ] Preserve runtime compatibility and expected behavior for Riot consumers while modernizing internals.
- [ ] Keep build, test, and CI flows passing after migration.

### Out of Scope

- Full framework rewrite away from Riot — not required to achieve v4 beta alignment.
- New major product features unrelated to modernization — defer until post-migration stability.

## Context

- The repository is an existing brownfield monorepo with modern and legacy Riot packages plus a demo app.
- A codebase map already exists in `.planning/codebase/`, including architecture and stack analysis.
- Current package manifests are still based on 3.x-era tsParticles dependencies and older syntax patterns.
- Primary objective from the idea document: update everything to latest versions, use the 4.0.0 beta packages being developed, and migrate to modern syntax.

## Constraints

- **Compatibility**: Preserve consumer-facing Riot package behavior while upgrading internals — avoid breaking integration contracts.
- **Dependency Strategy**: Prioritize `4.0.0` beta alignment for tsParticles packages — this is the central migration target.
- **Quality Gate**: Build/test/CI must remain green after changes — modernization cannot regress reliability.
- **Scope Control**: Focus on upgrade and syntax migration, not net-new feature expansion — keeps delivery bounded.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Use auto mode with fast planning depth | Goal is to move quickly from initialization to executable phases | — Pending |
| Treat this as modernization of an existing codebase | Existing monorepo and codebase map already define current baseline | — Pending |
| Anchor migration on tsParticles `4.0.0` beta | Requested primary objective and most important technical driver | — Pending |

---
*Last updated: 2026-04-10 after initialization*
