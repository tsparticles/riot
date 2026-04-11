# Pitfalls Research

**Domain:** Wrapper-library and demo migration to tsParticles v4 beta in a pnpm monorepo (Riot + legacy Riot package)
**Researched:** 2026-04-10
**Confidence:** MEDIUM

## Critical Pitfalls

### Pitfall 1: Treating beta upgrades as simple version bumps

**What goes wrong:**
Maintainers update package ranges to `4.0.0-beta` but do not re-validate wrapper lifecycle contracts (`particlesInit`, `particlesLoaded`, load/destroy behavior), so published wrappers compile yet regress runtime behavior.

**Why it happens:**
Migration is scoped as dependency churn, not API/behavior migration. In wrapper ecosystems, breakage often appears in callback timing and container lifecycle, not TypeScript/build errors.

**How to avoid:**
Define and lock a wrapper contract checklist before dependency updates: callback invocation order, error-path behavior, multi-instance behavior, mount/unmount cleanup, URL/options loading parity. Require these checks in CI for both `@tsparticles/riot` and `riot-particles`.

**Warning signs:**
- PRs only touch `package.json`/lockfile with little or no wrapper test changes.
- Existing tests remain mostly static rendering checks with no async/error-path assertions.
- Migration notes say "build passes" but not "runtime contract parity verified."

**Phase to address:**
Phase 1 - Contract Baseline and Test Design

---

### Pitfall 2: Divergence between modern and legacy wrapper packages

**What goes wrong:**
`components/riot` and `components-legacy/riot` drift during migration, producing inconsistent behavior across package names (same feature request behaves differently depending on import path).

**Why it happens:**
Duplicated implementation is modified manually in parallel under deadline pressure.

**How to avoid:**
Extract shared runtime logic into one internal module and keep package-specific entry tags minimal. Add parity tests that run the same behavioral assertions against both wrappers.

**Warning signs:**
- Fixes land in only one wrapper path.
- Changelogs claim parity while source diffs show non-trivial differences.
- Legacy package has no dedicated tests or no parity gate.

**Phase to address:**
Phase 2 - Shared Wrapper Core Extraction

---

### Pitfall 3: Monorepo orchestration split-brain (Lerna vs Nx vs pnpm)

**What goes wrong:**
Different build entrypoints validate different parts of the workspace, so CI/local outcomes diverge and migration regressions escape.

**Why it happens:**
Historical tooling paths are retained during upgrade (fallback commands, mixed orchestrators), and no single canonical command is enforced.

**How to avoid:**
Choose one canonical CI path for migration validation, keep a single required build/test command, and make optional paths non-authoritative. Add a CI check that fails if alternate orchestrators become required for success.

**Warning signs:**
- `build:ci` contains fallback logic that can mask primary-path failures.
- Team reports "passes locally with command A, fails in CI with command B."
- Toolchain updates are applied to one orchestrator config but not the others.

**Phase to address:**
Phase 2 - Toolchain and Orchestration Consolidation

---

### Pitfall 4: Workspace range mistakes with prerelease dependencies

**What goes wrong:**
Packages accidentally resolve to registry versions or mismatched prerelease tags, causing wrappers and demo app to run against different tsParticles lines.

**Why it happens:**
Monorepo maintainers rely on loose semver ranges during a prerelease cycle and assume local linking behavior is deterministic.

**How to avoid:**
Use explicit workspace linking policy for internal packages and verify resolved versions in lockfile/CI. Pin beta families deliberately across `@tsparticles/*` dependencies and prevent mixed major lines in workspace checks.

**Warning signs:**
- Demo app and wrapper package show different resolved `@tsparticles/*` versions.
- Lockfile diffs repeatedly flip prerelease build identifiers.
- "Works in one package" issues tied to node_modules resolution differences.

**Phase to address:**
Phase 2 - Dependency Alignment and Lockfile Governance

---

### Pitfall 5: Shipping demo success while real integration paths are untested

**What goes wrong:**
Migration is declared complete because demo UI renders, but production-like scenarios fail (async load rejection, multiple component instances, cleanup leaks, remote URL config failures).

**Why it happens:**
Demo tests are mostly static rendering checks and do not exercise runtime integration behaviors that wrappers exist to guarantee.

**How to avoid:**
Add integration-focused tests around wrapper lifecycle, including failure cases and multi-instance mount/unmount cycles. Treat demo as an integration harness, not just visual smoke test.

**Warning signs:**
- No tests for `tsParticles.load(...)` rejection path.
- No multi-instance tests despite known single-instance fragility.
- Demo passes while consumers report runtime lifecycle issues.

**Phase to address:**
Phase 3 - Integration Coverage Expansion

---

### Pitfall 6: Ignoring beta-specific failure semantics and observability

**What goes wrong:**
Runtime failures in beta dependencies are swallowed or inconsistently surfaced, making downstream debugging difficult and causing silent production degradation.

**Why it happens:**
Wrapper APIs expose success callbacks but not explicit failure channels; migration teams prioritize happy path parity.

**How to avoid:**
Define explicit error handling contract in wrapper API (documented callback/event or thrown promise behavior), add failure telemetry hooks in demo, and test error propagation from invalid options/URL cases.

**Warning signs:**
- Promise chains use `.then(...)` without local `.catch(...)` in wrapper load flow.
- Consumer docs show success callbacks only.
- Bug reports describe "nothing happens" without actionable error context.

**Phase to address:**
Phase 3 - Error Contract and Runtime Hardening

---

### Pitfall 7: Release metadata drift during migration

**What goes wrong:**
Published package versions/changelogs/tags become inconsistent across workspace packages, complicating downstream debugging and rollback.

**Why it happens:**
Migration touches many manifests simultaneously and release metadata checks are not enforced per package.

**How to avoid:**
Add a release integrity gate: package version alignment checks, changelog update validation, and publish dry-run verification for both modern and legacy packages before release.

**Warning signs:**
- `package.json` versions lag behind changelog history.
- One package is bumped without corresponding wrapper pair or demo compatibility note.
- Consumers cannot map behavior to an unambiguous release set.

**Phase to address:**
Phase 4 - Release Preparation and Version Governance

---

## Technical Debt Patterns

Shortcuts that seem reasonable but create long-term problems.

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Keep duplicate wrapper logic in modern + legacy packages | Faster initial migration edits | Ongoing drift and double maintenance | Only for emergency hotfixes; remove in next phase |
| Keep Lerna/Nx fallback build paths indefinitely | Fewer immediate script changes | Non-deterministic build truth and slower debugging | Acceptable during transition sprint only |
| Skip async/error integration tests while updating betas | Faster PR throughput | Runtime regressions escape despite green CI | Never for prerelease-major migrations |
| Use broad prerelease ranges without lock discipline | Easy dependency bumping | Unexpected resolution changes and flaky behavior | Never in coordinated wrapper + demo releases |

## Integration Gotchas

Common mistakes when connecting to external services.

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| tsParticles engine loader | Assume success path only and omit rejection handling | Define and test explicit failure path for `load` and init flows |
| Consumer-provided remote config URL | Trust arbitrary URLs without guardrails | Validate/document allowed URL usage and provide safer inline options default |
| Demo-to-wrapper dependency linkage | Assume local workspace dependency always resolves expected package | Enforce workspace protocol policy and verify resolved versions in CI |

## Performance Traps

Patterns that work at small scale but fail as usage grows.

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Re-initializing engine per component mount | Increasing mount latency and repeated startup work | Initialize once per app lifecycle or shared module and reuse handles | Multi-widget pages or rapid route changes |
| Linear container lookup + shared mutable IDs | Cross-instance teardown and unpredictable cleanup | Track instance-scoped container refs and destroy per instance | More than one `<riot-particles>` instance |
| Missing unmount cleanup tests | Memory/container leaks over time | Add mount/unmount regression suite and leak checks | Long-lived SPAs and repeated navigation |

## Security Mistakes

Domain-specific security issues beyond general web security.

| Mistake | Risk | Prevention |
|---------|------|------------|
| Loading untrusted remote particle configs by default | Runtime ingestion of hostile or malformed config payloads | Require explicit opt-in, validate URL policy, document trust boundaries |
| Carrying deprecated transitive dependencies through migration | Known vulnerable packages remain in distributed artifacts | Add dependency audit gate and regular lockfile refresh on migration branch |
| Treating demo-only logging as sufficient observability | Production incidents lack actionable diagnostics | Add wrapper-level error reporting contract and reproducible failure logs |

## UX Pitfalls

Common user experience mistakes in this domain.

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| Silent particle load failure | Users see empty background with no fallback or explanation | Provide deterministic fallback UI path and failure callback guidance |
| Breaking callback timing across versions | Consumer code appears flaky after upgrade | Document callback lifecycle and preserve timing contract with tests |
| Inconsistent behavior between package names | Integrators lose trust in migration quality | Guarantee modern/legacy parity with shared core + parity suite |

## "Looks Done But Isn't" Checklist

Things that appear complete but are missing critical pieces.

- [ ] **Dependency bump PR:** Often missing runtime contract verification - verify callback order and failure-path tests.
- [ ] **Green build status:** Often missing cross-package parity - verify modern and legacy wrappers pass identical behavioral tests.
- [ ] **Demo renders particles:** Often missing multi-instance and unmount safety - verify lifecycle cleanup with repeated mount cycles.
- [ ] **Release notes drafted:** Often missing version coherence - verify manifest/changelog/tag alignment across workspace packages.

## Recovery Strategies

When pitfalls occur despite prevention, how to recover.

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Wrapper contract regression after beta bump | HIGH | Freeze dependency updates, reproduce with contract suite, patch shared core, release coordinated modern+legacy fix |
| Monorepo build-path inconsistency | MEDIUM | Pick canonical command, remove fallback reliance, update CI docs/scripts, backfill missing checks |
| Workspace prerelease mismatch | MEDIUM | Lock versions to one beta family, regenerate lockfile, run full workspace integration tests |
| Release metadata drift | MEDIUM | Publish corrective patch with synchronized versions/changelogs and explicit migration advisory |

## Pitfall-to-Phase Mapping

How roadmap phases should address these pitfalls.

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| Beta upgrade treated as version bump only | Phase 1 - Contract Baseline and Test Design | Contract checklist exists; tests cover success + failure + callback order |
| Modern/legacy wrapper divergence | Phase 2 - Shared Wrapper Core Extraction | Shared module in use and parity tests pass for both package names |
| Orchestration split-brain (Lerna/Nx/pnpm) | Phase 2 - Toolchain Consolidation | One canonical CI command; no fallback-only success path |
| Workspace prerelease mismatch | Phase 2 - Dependency Alignment | CI asserts single beta family across `@tsparticles/*` deps |
| Demo-only validation gap | Phase 3 - Integration Coverage Expansion | Integration suite includes multi-instance, cleanup, error-path scenarios |
| Missing beta error semantics | Phase 3 - Error Contract Hardening | Wrapper docs + tests include deterministic failure behavior |
| Release metadata drift | Phase 4 - Release Governance | Version/changelog consistency checks pass before publish |

## Sources

- Project context and constraints: `/Users/matteo/Projects/GitHub/tsparticles/riot/.planning/PROJECT.md`
- Current risk inventory: `/Users/matteo/Projects/GitHub/tsparticles/riot/.planning/codebase/CONCERNS.md`
- Current testing posture: `/Users/matteo/Projects/GitHub/tsparticles/riot/.planning/codebase/TESTING.md`
- Workspace dependency/linking behavior: https://pnpm.io/workspaces (Last updated 2026-03-30)
- Prerelease and semver rules reference: https://semver.org/

---
*Pitfalls research for: tsParticles Riot modernization and v4 beta migration*
*Researched: 2026-04-10*
