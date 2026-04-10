# Codebase Concerns

**Analysis Date:** 2026-04-10

## Tech Debt

**Duplicated component implementation (modern + legacy):**
- Issue: The same logic is duplicated in `components/riot/src/riot-particles.riot` and `components-legacy/riot/src/riot-particles.riot`, including lifecycle, container lookup, and load flow.
- Files: `components/riot/src/riot-particles.riot`, `components-legacy/riot/src/riot-particles.riot`
- Impact: Fixes require changing two files, increasing drift risk and inconsistent behavior between packages.
- Fix approach: Extract shared logic into a single internal module consumed by both package entry files, keeping package-specific template differences minimal.

**Dual orchestrator build setup increases maintenance surface:**
- Issue: Workspace scripts support both Lerna and Nx (`build:lerna`, `build:nx`, fallback logic in `build:ci`) while package configuration is still strongly Lerna-centric.
- Files: `package.json`, `lerna.json`, `nx.json`, `README.md`
- Impact: CI/build troubleshooting is harder because multiple build paths can produce different outcomes.
- Fix approach: Standardize on one orchestrator for CI and local builds, keeping one canonical build command and reducing fallback branching.

**Outdated package metadata in workspace vs package changelog:**
- Issue: Multiple package manifests are pinned at `2.8.0`, while changelogs document releases up to `2.9.3`.
- Files: `components/riot/package.json`, `components-legacy/riot/package.json`, `apps/riot/package.json`, `components/riot/CHANGELOG.md`, `apps/riot/CHANGELOG.md`
- Impact: Version traceability is unclear, and release/debug workflows can target stale metadata.
- Fix approach: Align `package.json` versions with release tags/changelog generation flow and enforce via release automation checks.

## Known Bugs

**Multiple component instances can destroy each other:**
- Symptoms: Mounting a second `<riot-particles>` instance can destroy the previous container because cleanup uses a module-level `oldId` shared by all instances.
- Files: `components/riot/src/riot-particles.riot`, `components-legacy/riot/src/riot-particles.riot`
- Trigger: Render more than one instance with different `id` values in the same runtime.
- Workaround: Use only one mounted instance per page/session, or patch component to store instance-scoped references.

**Unhandled load rejection from particle engine:**
- Symptoms: Failures from `tsParticles.load(...)` are not caught, so consumers have no standardized error callback path.
- Files: `components/riot/src/riot-particles.riot`, `components-legacy/riot/src/riot-particles.riot`
- Trigger: Invalid config payload, unreachable URL config, or runtime engine errors during load.
- Workaround: Wrap `particlesInit` and external config setup with caller-side try/catch and avoid remote URL configs that can fail unpredictably.

**README usage snippets contain invalid markup:**
- Symptoms: Documentation includes malformed component tags (`<@tsparticles/riot` and duplicated closing `/>`), which can cause copy/paste integration failures.
- Files: `README.md`, `components/riot/README.md`
- Trigger: Following README snippets verbatim.
- Workaround: Use `<riot-particles ... />` with valid Riot syntax and remove extra closing line.

## Security Considerations

**Dependency tree includes deprecated/vulnerability-prone transitive packages:**
- Risk: Lockfile includes deprecated package notices (for example old `glob`, old `tar`, and pre-v4 `rimraf`), increasing supply-chain exposure.
- Files: `pnpm-lock.yaml`
- Current mitigation: Renovate configuration exists (`renovate.json`) and workflow builds on every push/PR (`.github/workflows/nodejs.yml`).
- Recommendations: Run regular dependency audits, refresh lockfile with updated transitive dependencies, and block CI on known critical vulnerabilities.

**No guardrails around externally provided `url` option in component API:**
- Risk: Arbitrary config URL can be passed to the engine load call from consuming apps, enabling untrusted remote configuration ingestion at runtime.
- Files: `components/riot/src/riot-particles.riot`, `components-legacy/riot/src/riot-particles.riot`
- Current mitigation: Not detected.
- Recommendations: Document strict allowlist guidance, validate/sanitize `props.url` before load, and provide a safer default path using inline options.

## Performance Bottlenecks

**Engine initialization runs on every mount:**
- Problem: `tsParticles.init()` is executed in `onMounted` for each component instance.
- Files: `components/riot/src/riot-particles.riot`, `components-legacy/riot/src/riot-particles.riot`
- Cause: Initialization is not memoized or centralized.
- Improvement path: Initialize engine once per app lifecycle and reuse across component mounts.

**Container lookup is linear and repeated:**
- Problem: Cleanup uses `tsParticles.dom().find(...)` each mount.
- Files: `components/riot/src/riot-particles.riot`, `components-legacy/riot/src/riot-particles.riot`
- Cause: Lookup scans current container list each time instead of maintaining direct references.
- Improvement path: Keep per-instance container handles and destroy directly during unmount/update.

## Fragile Areas

**Lifecycle cleanup is incomplete:**
- Files: `components/riot/src/riot-particles.riot`, `components-legacy/riot/src/riot-particles.riot`
- Why fragile: Component only handles mount path; there is no explicit unmount lifecycle cleanup, increasing risk of leaked container state.
- Safe modification: Add an unmount hook that destroys the mounted container for the current instance only, then add regression tests for mount/unmount cycles.
- Test coverage: Gaps in `components/riot/test.js` and no legacy component tests detected.

**CI toolchain version mismatch:**
- Files: `package.json`, `.github/workflows/nodejs.yml`
- Why fragile: Repository declares `pnpm@10.33.0` in `packageManager`, but CI installs `pnpm` version `8`.
- Safe modification: Pin CI pnpm to the same major/minor family as `packageManager` and validate lockfile with frozen install.
- Test coverage: No CI guard detected for pnpm/toolchain parity.

## Scaling Limits

**Component state management is single-instance oriented:**
- Current capacity: Reliable behavior is oriented around one active `<riot-particles>` instance due to shared `oldId` state.
- Limit: Concurrent/multi-instance usage can cause cross-instance teardown.
- Scaling path: Move to per-instance container tracking and keyed lifecycle handling.

## Dependencies at Risk

**Lerna legacy package-management chain:**
- Risk: Lockfile references `@lerna/legacy-package-management` and Lerna dependency chain with deprecation messaging.
- Impact: Future tooling upgrades can break monorepo scripts or force abrupt migration.
- Migration plan: Shift build/release orchestration to one maintained path (Nx or pure pnpm workspaces) and remove legacy Lerna-only behaviors.

**Aging test/build ecosystem dependencies in package manifests:**
- Risk: Component and demo packages rely on older major versions of tooling (for example `prettier@^2.8.1`, `webpack-dev-server@^4.11.1`) while workspace tooling evolved.
- Impact: Inconsistent local environments and harder upgrades across packages.
- Migration plan: Define a centralized dependency policy and update package-level dev dependencies in lockstep.

## Missing Critical Features

**No explicit error callback API for component load failures:**
- Problem: Consumers cannot reliably subscribe to or handle `tsParticles.load` failures through the component contract.
- Blocks: Robust production observability and graceful UI fallback behavior in consuming applications.

**No runtime validation for required `id`/configuration contract:**
- Problem: Component behavior silently degrades when props are missing or malformed (for example calls `particlesLoaded(undefined)` when `id` is absent).
- Blocks: Predictable integration behavior and easier debugging for downstream teams.

## Test Coverage Gaps

**Core particle-loading behavior untested in published component package:**
- What's not tested: Asynchronous load success/failure paths, callback invocation order, multi-instance behavior, and cleanup behavior.
- Files: `components/riot/src/riot-particles.riot`, `components/riot/test.js`
- Risk: Regressions in lifecycle behavior can ship without detection.
- Priority: High

**Legacy package has no dedicated tests detected:**
- What's not tested: Any behavior of `components-legacy/riot/src/riot-particles.riot`.
- Files: `components-legacy/riot/src/riot-particles.riot`
- Risk: Legacy package can drift silently from modern package behavior.
- Priority: High

**Demo tests cover only static rendering:**
- What's not tested: End-to-end particle integration and runtime interaction/performance behavior.
- Files: `apps/riot/src/components/global/my-component/my-component.spec.js`, `apps/riot/src/components/global/sidebar/sidebar.spec.js`, `apps/riot/src/components/includes/user/user.spec.js`
- Risk: Demo can pass tests while integration scenarios fail in real browser usage.
- Priority: Medium

---

*Concerns audit: 2026-04-10*
