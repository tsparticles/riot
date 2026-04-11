# Feature Research

**Domain:** JS wrapper package modernization for tsParticles Riot (v4 beta to stable)
**Researched:** 2026-04-10
**Confidence:** HIGH

## Feature Landscape

### Table Stakes (Users Expect These)

Features users assume exist. Missing these = product feels incomplete.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| API compatibility contract across beta (id/options/url/particlesInit/particlesLoaded/class/style) | Wrapper consumers expect upgrades to preserve integration shape while internals modernize | MEDIUM | Keep behavior stable while aligning to `@tsparticles/engine` v4 beta; document any unavoidable changes with deprecations first |
| Legacy + scoped package continuity (`riot-particles` and `@tsparticles/riot`) | Existing installs depend on historical package names during transition windows | MEDIUM | Publish both packages from one shared implementation to prevent drift |
| Correct lifecycle and multi-instance safety | Wrapper components are expected to mount/unmount independently without cross-instance teardown | HIGH | Must remove module-level shared state (`oldId`) and track container per instance |
| Explicit error handling path for async load failures | Consumers expect failed remote/local config loads to be catchable and observable | MEDIUM | Standardize callback/event for errors (or documented promise rejection behavior) |
| Beta release channel hygiene (SemVer prereleases + npm dist-tags) | Maintainers and consumers expect safe beta adoption without breaking `latest` consumers | LOW | Use prerelease versions and `beta` tag before moving stable to `latest` |
| Build/test/CI reliability parity with declared toolchain | Maintainers expect reproducible installs and green CI before promotion to stable | MEDIUM | Align CI package manager/runtime versions with workspace declarations |
| Migration docs with valid, copy-pastable examples | Developers expect concise upgrade docs during major transitions | LOW | Fix invalid snippets, show beta install path, and include "from v3 to v4 beta" mapping |
| Cross-wrapper behavioral consistency with tsParticles ecosystem norms | Users expect Riot wrapper behavior comparable to React/Vue wrappers (init once, options/url patterns) | MEDIUM | Match callback semantics and initialization patterns where feasible |

### Differentiators (Competitive Advantage)

Features that set the product apart. Not required, but valuable.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Single shared core used by modern + legacy Riot packages | Reduces maintenance cost and prevents subtle behavior drift between published package names | MEDIUM | Extract lifecycle/load logic into internal module, keep thin package entry wrappers |
| One-time engine initialization strategy | Better runtime performance and cleaner app integration than per-mount init | MEDIUM | Align with ecosystem guidance used in other wrappers to initialize once per app lifecycle |
| Defensive URL config policy (validation + documented allowlist patterns) | Improves security posture for teams loading remote configs | MEDIUM | Provide safe defaults and explicit docs for trusted URL usage |
| Beta quality gates with promotion checklist (beta -> rc -> stable) | Increases confidence for maintainers and enterprise adopters | LOW | Add explicit promotion criteria: test pass rates, known issue threshold, docs completeness |
| Compatibility test suite against realistic consumer scenarios | Demonstrates reliability beyond unit tests (multi-instance, unmount cycles, async failure) | HIGH | Add scenario tests for both package names and real Riot mounting flows |

### Anti-Features (Commonly Requested, Often Problematic)

Features that seem good but create problems.

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| API redesign during beta (renaming core props/events) | Looks like a chance to "clean everything up" before v4 stable | Multiplies migration cost, slows adoption, and obscures whether regressions come from engine or wrapper | Preserve API contract; add deprecations with clear timelines if change is unavoidable |
| Multiple long-term build orchestrators for the same workflow | Feels safer to keep fallback tooling forever | Creates divergent CI/local behavior and raises maintenance burden | Choose one canonical build path; keep temporary fallback only for migration window |
| Wrapper-owned remote config proxy/transform service | Seems convenient for users with URL configs | Expands scope from wrapper library to backend/security surface area | Keep wrapper client-only; document secure hosting and URL validation practices |
| Framework rewrite beyond Riot modernization scope | Tempting during major-version work | Delays stabilization and dilutes objective of v4 alignment | Complete Riot modernization first; evaluate broader rewrites post-stable |
| Divergent wrapper behavior from other tsParticles adapters | Could optimize for Riot-specific preferences | Increases cognitive load for multi-framework users and maintainers | Keep a shared behavior baseline; isolate Riot-specific additions as optional |

## Feature Dependencies

```text
API compatibility contract
    -> Legacy + scoped package continuity
    -> Migration docs with valid examples

Lifecycle and multi-instance safety
    -> Explicit error handling path
    -> Compatibility scenario test suite

Build/test/CI reliability parity
    -> Beta release channel hygiene
    -> Beta quality gate promotion checklist

Single shared core (modern + legacy)
    -> Cross-wrapper behavioral consistency
    -> Reduced regression surface in beta-to-stable promotion

Defensive URL config policy
    -> Migration docs and security guidance
```

### Dependency Notes

- **Release hygiene depends on CI reliability:** you cannot safely promote `beta` to `latest` without deterministic build/test outcomes.
- **Legacy/scoped continuity depends on API stability:** both package names must expose the same contract or migration guidance becomes contradictory.
- **Differentiators build on table stakes:** shared-core refactor and one-time init are only meaningful after lifecycle correctness is guaranteed.
- **Security guidance depends on docs quality:** URL validation controls must be paired with examples or consumers will bypass them.

## MVP Definition

### Launch With (v1)

Minimum viable modernization for v4 beta confidence.

- [ ] API compatibility contract preserved and documented
- [ ] Lifecycle correctness + multi-instance safety fixed
- [ ] Explicit load error handling path implemented
- [ ] Legacy + scoped package continuity via shared behavior
- [ ] CI/toolchain parity and green build/test gates
- [ ] Valid migration docs for beta consumers

### Add After Validation (v1.x)

- [ ] One-time engine initialization optimization
- [ ] Defensive URL validation + allowlist guidance
- [ ] Promotion checklist automation (beta -> stable)

### Future Consideration (v2+)

- [ ] Advanced observability hooks for container lifecycle telemetry
- [ ] Optional framework-specific ergonomics that do not alter core contract

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| API compatibility contract | HIGH | MEDIUM | P1 |
| Lifecycle and multi-instance safety | HIGH | HIGH | P1 |
| Explicit error handling path | HIGH | MEDIUM | P1 |
| Legacy + scoped package continuity | HIGH | MEDIUM | P1 |
| CI/toolchain parity | HIGH | MEDIUM | P1 |
| Migration docs and examples | HIGH | LOW | P1 |
| Single shared core refactor | MEDIUM | MEDIUM | P2 |
| One-time engine initialization | MEDIUM | MEDIUM | P2 |
| Defensive URL policy | MEDIUM | MEDIUM | P2 |
| Beta promotion checklist automation | MEDIUM | LOW | P2 |

**Priority key:**
- P1: Must have for stable-ready migration
- P2: Strongly recommended improvements after baseline stability
- P3: Future exploration

## Competitor Feature Analysis

| Feature | Competitor A | Competitor B | Our Approach |
|---------|--------------|--------------|--------------|
| App-lifetime engine initialization guidance | `@tsparticles/react` documents one-time initialization (`initParticlesEngine`) | `@tsparticles/vue3` documents app plugin `init` usage | Add equivalent Riot guidance and avoid per-mount init |
| Config input modes (`options` object vs `url`) | Supported | Supported | Keep both, add error and security guidance for URL mode |
| Clear installation and migration paths | Mature package docs with straightforward install + usage | Mature package docs plus explicit Vue2 -> Vue3 migration section | Add explicit v3 -> v4 beta -> stable Riot migration map |
| Ecosystem naming consistency | Scoped modern package naming | Scoped modern package naming | Maintain scoped + legacy package continuity during transition, then simplify post-stable |

## Sources

- Local project context: `/Users/matteo/Projects/GitHub/tsparticles/riot/.planning/PROJECT.md` (HIGH)
- Local codebase risks: `/Users/matteo/Projects/GitHub/tsparticles/riot/.planning/codebase/CONCERNS.md` (HIGH)
- Current wrapper implementation: `/Users/matteo/Projects/GitHub/tsparticles/riot/components/riot/src/riot-particles.riot` (HIGH)
- Current legacy implementation: `/Users/matteo/Projects/GitHub/tsparticles/riot/components-legacy/riot/src/riot-particles.riot` (HIGH)
- Current docs/examples quality: `/Users/matteo/Projects/GitHub/tsparticles/riot/README.md` (HIGH)
- Semantic Versioning 2.0.0 spec: https://semver.org/ (HIGH)
- npm dist-tags guidance: https://docs.npmjs.com/adding-dist-tags-to-packages (HIGH)
- tsParticles ecosystem docs: https://particles.js.org/docs/ (MEDIUM; broad docs page rendering)
- tsParticles React wrapper README/repo usage patterns: https://github.com/tsparticles/react (MEDIUM)
- tsParticles Vue3 wrapper README/repo usage patterns: https://github.com/tsparticles/vue3 (MEDIUM)

---
*Feature research for: tsParticles Riot modernization and v4 beta migration*
*Researched: 2026-04-10*
