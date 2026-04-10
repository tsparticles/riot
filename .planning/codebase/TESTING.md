# Testing Patterns

**Analysis Date:** 2026-04-10

## Test Framework

**Runner:**
- Mocha ^10.2.0
- Config: no standalone `mocha` config file detected; runner configuration is provided in `apps/riot/package.json` script `test`.

**Assertion Library:**
- Chai (`expect` style), used in `apps/riot/src/components/**/*.spec.js` and `components/riot/test.js`.

**Run Commands:**
```bash
pnpm --filter @tsparticles/riot-demo test      # Run app demo tests with nyc + jsdom + @riotjs/register
pnpm --filter @tsparticles/riot-demo run cov   # Generate LCOV/text-lcov report from nyc data
pnpm --filter @tsparticles/riot-demo run cov-html  # Generate HTML coverage report
```

## Test File Organization

**Location:**
- Co-located with source Riot components in app demo package:
  - `apps/riot/src/components/global/my-component/my-component.spec.js`
  - `apps/riot/src/components/global/sidebar/sidebar.spec.js`
  - `apps/riot/src/components/includes/user/user.spec.js`
- Library package test is at package root (`components/riot/test.js`) instead of `src/`.

**Naming:**
- Use `*.spec.js` for app component tests (`apps/riot/src/components/**/*.spec.js`).
- Use `test.js` for the component package root test (`components/riot/test.js`).

**Structure:**
```
apps/riot/src/components/**/<component-name>.spec.js
components/riot/test.js
```

## Test Structure

**Suite Organization:**
```typescript
import ComponentTag from './component.riot'
import { expect } from 'chai'
import { component } from 'riot'

describe('Component Unit Test', () => {
  const mountComponent = component(ComponentTag)

  it('The component is properly rendered', () => {
    const div = document.createElement('div')
    const instance = mountComponent(div, { /* props */ })

    expect(instance.$('selector').innerHTML).to.be.equal('expected')
  })
})
```

**Patterns:**
- Setup pattern: compile mount function once per suite (`const mountX = component(X)`), then create a fresh `div` per test (`apps/riot/src/components/**/*.spec.js`, `components/riot/test.js`).
- Teardown pattern: explicit teardown hooks (`afterEach`, unmount calls) are not currently used in existing specs.
- Assertion pattern: direct DOM assertions via `instance.$(selector)` with strict equality in Chai (`to.be.equal(...)`).

## Mocking

**Framework:**
- Not applicable; no dedicated mocking/stubbing library detected.

**Patterns:**
```typescript
// Existing tests mount real Riot tags without mocks.
const mountSidebar = component(Sidebar)
const instance = mountSidebar(document.createElement('div'))
expect(instance.$('h1').innerHTML).to.be.equal('Sidebar')
```

**What to Mock:**
- Mocking patterns are not currently present; preserve current approach of testing rendered output with real Riot components in `apps/riot/src/components/**/*.spec.js`.

**What NOT to Mock:**
- Do not mock component template rendering for current unit tests; existing tests validate real rendered markup (`apps/riot/src/components/**/*.spec.js`, `components/riot/test.js`).

## Fixtures and Factories

**Test Data:**
```typescript
const instance = mountMyComponent(div, { message: 'hello' })
const user = mountUser(div, { name: 'Jack' })
```

**Location:**
- Inline in each spec file; no shared fixtures/factories directory detected.

## Coverage

**Requirements:**
- Coverage tooling exists through `nyc` scripts in `apps/riot/package.json`; no explicit minimum threshold enforcement is detected.

**View Coverage:**
```bash
pnpm --filter @tsparticles/riot-demo run cov-html
```

## Test Types

**Unit Tests:**
- Primary test type. Scope is component rendering and prop interpolation in Riot tags (`apps/riot/src/components/**/*.spec.js`, `components/riot/test.js`).

**Integration Tests:**
- Not detected.

**E2E Tests:**
- Not used (no Playwright/Cypress configuration detected).

## Common Patterns

**Async Testing:**
```typescript
// No async spec pattern is currently used in existing test files.
// Current tests are synchronous render/assert flows.
```

**Error Testing:**
```typescript
// No explicit error-path tests are currently implemented.
// Add describe/it blocks with invalid props when extending coverage.
```

---

*Testing analysis: 2026-04-10*
