# Frontend Playwright Testing Convention

Convention: Frontend Playwright Testing Convention  
Version: 1.0.0  
Status: Draft  
Last Updated: 2026-05-04  
Maintainer: Charlie Mi <txqq3116934585@gmail.com>

## 1. Purpose

This convention defines concrete file, naming, selector, assertion, data, configuration, and command rules for Playwright-based frontend tests.

It applies to Playwright tests created after frontend implementation work.

---

## 2. Package Manager and Script Runner

Each project MUST choose one canonical package manager and script runner.

Examples:

- `pnpm`
- `npm`
- `yarn`
- `bun`

The chosen tool MUST be used consistently in documentation, local commands, package scripts, and CI commands.

This convention uses `pnpm` in examples only.

---

## 3. Required Assets

A Playwright frontend testing baseline SHOULD include:

- `playwright.config.ts`
- `package.json` test scripts
- Playwright spec files
- helper files
- fixture files
- minimal reporter output
- trace artifacts only when explicitly enabled for diagnosis

Screenshot, video, and HTML report artifacts SHOULD NOT be enabled by default.

---

## 4. Directory Structure

Recommended structure:

```text
tests/
  e2e/
  helpers/
  fixtures/
playwright.config.ts
```

Spec files SHOULD live under:

```text
tests/e2e/**/*.spec.ts
```

Helper files SHOULD live under:

```text
tests/helpers/**/*.ts
```

Fixture files SHOULD live under:

```text
tests/fixtures/**/*.ts
```

---

## 5. File Responsibilities

### 5.1 `playwright.config.ts`

`playwright.config.ts` SHOULD define:

- test directories
- frontend application base URL
- browser projects
- minimal reporter configuration
- failure diagnostics policy
- retries
- timeouts
- local vs CI behavior

Failure diagnostics policy SHOULD follow these defaults:

- assertion output is the default diagnostic entrypoint
- trace SHOULD be disabled by default
- trace MAY be enabled temporarily when assertion output is insufficient
- screenshot SHOULD be disabled by default
- video SHOULD be disabled by default
- HTML report SHOULD NOT be enabled by default unless explicitly required

`playwright.config.ts` MUST NOT contain business logic, page flow logic, test assertions, or ad hoc debugging hacks.

### 5.2 `package.json`

`package.json` SHOULD define stable Playwright command entrypoints.

Recommended scripts:

```json
{
  "scripts": {
    "playwright:install": "playwright install",
    "test:e2e": "playwright test",
    "test:e2e:headed": "playwright test --headed",
    "test:e2e:ui": "playwright test --ui",
    "test:e2e:debug": "playwright test --debug"
  }
}
```

`package.json` MUST NOT contain test assertions, page flow logic, or helper logic.

### 5.3 Spec Files

Spec files MUST express user-facing flows, interact through browser behavior, contain formal assertions, and remain readable as the narrative of the test.

Spec files MUST NOT become giant mixed-purpose files, hide intent behind excessive helper abstraction, or act as temporary debugging scratchpads.

### 5.4 Helper Files

Helpers SHOULD extract repeated page actions, repeated form actions, repeated assertions, and repeated setup patterns.

Helpers MUST NOT hide the intent of a spec, become uncontrolled abstraction layers, or introduce heavy page object hierarchies by default.

### 5.5 Fixture Files

Fixtures SHOULD contain scenario data, minimal valid input sets, standard variants, and negative or exception case data.

Fixtures MUST remain data-focused.

Fixtures MUST NOT mix unclear logic and data.

---

## 6. Naming Rules

Spec files SHOULD use descriptive kebab-case names:

```text
<flow-name>.spec.ts
```

Examples:

- `login-flow.spec.ts`
- `checkout-flow.spec.ts`
- `app-smoke.spec.ts`
- `release-validation.spec.ts`

Avoid vague names such as `test1.spec.ts`, `final.spec.ts`, `misc.spec.ts`, `temp.ts`, and `data2.ts`.

Helper and fixture files SHOULD be named by responsibility or scenario.

Examples:

- `navigation.ts`
- `forms.ts`
- `assertions.ts`
- `auth.ts`
- `minimal-user.ts`
- `checkout-scenarios.ts`
- `invalid-form-inputs.ts`

---

## 7. User Interaction Rules

Playwright tests SHOULD simulate important user interactions through automated browser control.

Tests SHOULD interact with the application by opening pages, clicking controls, filling forms, submitting actions, waiting for navigation or UI state changes, and asserting visible results.

Tests SHOULD prioritize user-facing outcomes over implementation details.

Tests MUST NOT primarily validate internal component state, private implementation details, or unstable DOM structure.

Preferred pattern:

```text
user action -> visible system response -> stable assertion
```

---

## 8. Selector Rules

Default selector strategy:

```text
data-testid
```

Selector preference order SHOULD be:

1. `data-testid`
2. stable roles
3. stable labels
4. stable business text
5. other selectors only when necessary

Tests MUST NOT rely primarily on CSS classes, deep DOM paths, `nth-child`, unstable copy, or layout-dependent selectors.

Text selectors and text assertions MAY be used for stable business-critical labels, statuses, and results.

Text selectors SHOULD NOT be used broadly for incidental or frequently changing copy.

---

## 9. Assertion Rules

Assertions SHOULD be few, stable, and high-value.

Assertions SHOULD prioritize:

1. structural presence
2. flow success
3. key business values
4. stable user-visible outcomes

Preferred assertions include visible critical sections, critical buttons, successful submits, successful saves, route transitions, modal open/close behavior, expected statuses, and important persisted values.

Tests SHOULD NOT overbind assertions to exact styling details, fragile copy, pixel-sensitive behavior, unstable DOM shape, or low-value implementation details.

Assertions MUST validate what the user or business flow cares about.

---

## 10. Test Data Rules

A Playwright test suite MAY use both UI-created data and pre-seeded data.

Use UI-created data for creation flows, edit flows, form submission flows, and true end-to-end user paths.

Use pre-seeded data for list pages, detail pages, reporting pages, comparison pages, search/filter behavior, and stable downstream regression behavior.

Tests SHOULD be repeatable, isolated, deterministic where possible, and minimally coupled to other tests.

Test data SHOULD have clear ownership.

Tests SHOULD avoid hidden cross-test dependencies.

---

## 11. Execution Rules

Recommended command usage:

```bash
pnpm install
pnpm playwright:install
pnpm test:e2e
pnpm test:e2e:headed
pnpm test:e2e:ui
pnpm test:e2e:debug
```

Targeted spec execution SHOULD be supported:

```bash
pnpm exec playwright test tests/e2e/login-flow.spec.ts
```

Temporary trace diagnosis MAY be run when assertion output is insufficient:

```bash
pnpm exec playwright test tests/e2e/login-flow.spec.ts --trace on
```

CI and release validation requirements MUST be explicit in the project.

HTML report commands SHOULD NOT be part of the default baseline unless explicitly required.
