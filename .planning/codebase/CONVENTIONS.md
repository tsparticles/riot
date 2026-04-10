# Coding Conventions

**Analysis Date:** 2026-04-10

## Naming Patterns

**Files:**
- Use kebab-case for Riot component files and directories (for example `apps/riot/src/components/global/my-component/my-component.riot`, `apps/riot/src/components/global/sidebar/sidebar.riot`).
- Use `.spec.js` suffix for component unit tests colocated with the component (for example `apps/riot/src/components/global/my-component/my-component.spec.js`, `apps/riot/src/components/includes/user/user.spec.js`).
- Use lowercase `index.js` for application entrypoints (`apps/riot/src/index.js`).

**Functions:**
- Use camelCase for local functions and callbacks (for example `toggleUser` in `apps/riot/src/components/global/sidebar/sidebar.riot`, `particlesInit` and `particlesLoaded` in `apps/riot/src/components/global/my-component/my-component.riot`).
- Use `mount<ComponentName>` naming for test helper mount functions (for example `mountMyComponent`, `mountSidebar`, `mountUser` in `apps/riot/src/components/**/*.spec.js`).

**Variables:**
- Use camelCase for local variables and state keys (for example `oldId` in `components/riot/src/riot-particles.riot`, `showUser` in `apps/riot/src/components/global/sidebar/sidebar.riot`).
- Use descriptive names for mounted tag instances in tests (for example `component` in `apps/riot/src/components/global/sidebar/sidebar.spec.js`).

**Types:**
- Not applicable in current source modules; files in this repository focus on `.js` and `.riot` implementation (`apps/riot/src/**/*.js`, `components/riot/src/*.riot`, `components-legacy/riot/src/*.riot`).

## Code Style

**Formatting:**
- Use Prettier via package-level scripts and shared config reference in `package.json` files.
- Formatting enforcement is configured through scripts in `components/riot/package.json` and `components-legacy/riot/package.json`:
  - `prettify:src`, `prettify:readme`
  - `prettify:ci:src`, `prettify:ci:readme`
- Adopt existing component formatting style per package:
  - 2-space indentation in app component/test files (`apps/riot/src/components/global/sidebar/sidebar.riot`, `apps/riot/src/components/global/sidebar/sidebar.spec.js`)
  - 4-space indentation in publishable component package (`components/riot/src/riot-particles.riot`, `components/riot/test.js`)

**Linting:**
- ESLint configuration files are not detected (`.eslintrc*`, `eslint.config.*` not present in repository root/workspaces).
- Use formatting + test/build checks as quality gates:
  - formatting checks in `components/riot/package.json` and `components-legacy/riot/package.json`
  - CI build pipeline in `.github/workflows/nodejs.yml`

## Import Organization

**Order:**
1. External package imports first (for example `import { mount, register } from "riot"` in `apps/riot/src/index.js`, `import { tsParticles } from "@tsparticles/engine"` in `components/riot/src/riot-particles.riot`).
2. Local component imports second (for example `import MyComponent from "./components/global/my-component/my-component.riot"` in `apps/riot/src/index.js`).
3. No side-effect imports except explicit runtime setup imports (for example `import "@riotjs/hot-reload"` in `apps/riot/src/index.js`).

**Path Aliases:**
- Not detected. Use relative imports inside app/components (for example `../../includes/user/user.riot` in `apps/riot/src/components/global/sidebar/sidebar.riot`).

## Error Handling

**Patterns:**
- Use guard checks before optional callbacks (for example `if (props.particlesInit)`, `if (props.particlesLoaded)` in `components/riot/src/riot-particles.riot`).
- Use fallback behavior when required props are missing (for example invoking `props.particlesLoaded(undefined)` when `props.id` is absent in `components/riot/src/riot-particles.riot`).
- No centralized try/catch strategy is implemented in current files.

## Logging

**Framework:** console

**Patterns:**
- Logging appears in demo-level callback usage (`console.log(container)` in `apps/riot/src/components/global/my-component/my-component.riot`).
- Prefer keeping published component package logic callback-driven and free of direct logging (`components/riot/src/riot-particles.riot`).

## Comments

**When to Comment:**
- Use concise intent comments around registration/bootstrap sections (for example `// register` and mount comment block in `apps/riot/src/index.js`).
- Use explanatory comments in README usage examples for initialization semantics (`README.md`, `components/riot/README.md`).

**JSDoc/TSDoc:**
- Not detected in runtime source (`apps/riot/src`, `components/riot/src`, `components-legacy/riot/src`).

## Function Design

**Size:**
- Keep component methods short and single-purpose (for example `toggleUser()` in `apps/riot/src/components/global/sidebar/sidebar.riot`).
- Lifecycle handlers can be moderate-length when orchestrating integration setup (for example `async onMounted(props)` in `components/riot/src/riot-particles.riot`).

**Parameters:**
- Pass Riot `props` objects into lifecycle handlers and use property guards (`components/riot/src/riot-particles.riot`).
- Pass explicit prop objects in tests when behavior depends on props (for example `{ message: 'hello' }` and `{ name: 'Jack' }` in `apps/riot/src/components/**/*.spec.js`).

**Return Values:**
- Component methods typically mutate/update state via `this.update(...)` rather than returning values (`apps/riot/src/components/global/sidebar/sidebar.riot`).
- Async lifecycle methods perform side effects and callback invocation without explicit return payloads (`components/riot/src/riot-particles.riot`).

## Module Design

**Exports:**
- Use default export objects inside Riot `<script>` blocks for component behavior (`apps/riot/src/components/global/sidebar/sidebar.riot`, `components/riot/src/riot-particles.riot`).
- Register components centrally in app entrypoint with explicit tag names (`apps/riot/src/index.js`).

**Barrel Files:**
- Not detected for app components (`apps/riot/src/components/**`).
- Package root entry exists for publishable component import target used by tests (`components/riot/test.js` imports `./`).

---

*Convention analysis: 2026-04-10*
