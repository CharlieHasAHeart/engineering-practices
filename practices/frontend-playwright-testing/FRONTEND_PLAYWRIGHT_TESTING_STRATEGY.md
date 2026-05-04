# Frontend Playwright Testing Strategy

Strategy: Frontend Playwright Testing Strategy  
Version: 1.0.0  
Status: Draft  
Last Updated: 2026-05-04  
Maintainer: Charlie Mi <txqq3116934585@gmail.com>

## 1. Strategy Positioning

Frontend Playwright testing is a **lightweight, user-flow-oriented, post-implementation validation strategy**.

This strategy applies after frontend implementation work is completed or updated.

It defines how coding agents and developers should decide what to test, how to test it, how to execute tests, how to diagnose failures, and how to maintain Playwright test assets.

This strategy is not intended to explain what Playwright is. It defines how Playwright-based frontend testing should be used.

---

## 2. Core Strategy

Frontend Playwright tests MUST validate frontend behavior from the user perspective.

Playwright tests SHOULD focus on:

- critical user flows
- user-visible behavior
- important page states
- important business results
- previously working behavior that must not regress
- release-critical frontend behavior

Playwright tests SHOULD NOT attempt to cover every frontend quality concern by default.

The default Playwright baseline MUST remain lightweight, repeatable, readable, maintainable, stable, and aligned with real user flows.

---

## 3. Formal Test Authority

`@playwright/test` is the formal test execution authority.

Formal test status MUST come from the Playwright test runner.

The following MUST NOT be treated as formal pass/fail evidence:

- manual browser inspection
- Playwright CLI exploration
- Skills-based exploration
- ad hoc browser interaction
- visual observation without a formal spec result

Any behavior that must be protected by the testing baseline MUST be implemented as an `@playwright/test` spec.

CI, regression validation, and release validation MUST rely on formal Playwright test execution.

If a formal Playwright test fails, manual verification MUST NOT override the failed test result.

---

## 4. Test Scope Strategy

Playwright tests SHOULD cover frontend behavior that is user-visible, business-important, likely to break during changes, important for release confidence, and worth protecting over time.

Playwright tests SHOULD cover:

- page availability
- critical route accessibility
- login or authentication flows, when applicable
- submit, save, create, edit, search, filter, and navigation flows
- visible results after user actions
- key status changes
- key data display
- previously fixed bugs that may regress

Playwright tests SHOULD NOT cover by default:

- every component detail
- every edge-case input combination
- every styling detail
- pixel-level visual consistency
- full performance validation
- full accessibility validation
- large cross-browser matrices
- large mobile-device matrices

Optional areas MAY be adopted separately when explicitly required.

---

## 5. Test Layering Strategy

A frontend Playwright test suite SHOULD be organized into layers.

### 5.1 Smoke Layer

The smoke layer SHOULD verify that the application is basically usable.

It SHOULD cover application entry page loading, critical routes, critical layout regions, and obvious startup or deployment failures.

Smoke tests SHOULD be small, fast, and stable.

### 5.2 Core User Flow Layer

The core user flow layer SHOULD verify important user journeys.

It SHOULD cover login or entry flows, form submission flows, create/edit/save/delete flows, search and filter flows, key navigation flows, and business-critical page transitions.

Core user flow tests SHOULD interact with the application as a user would.

### 5.3 Regression Layer

The regression layer SHOULD protect previously working behavior from accidental breakage.

It SHOULD cover stable high-value behavior, previously fixed bugs, refactor-sensitive flows, and paths that must remain reliable across feature changes.

Regression tests SHOULD be repeatable and stable.

Regression tests SHOULD NOT become exhaustive edge-case matrices by default.

### 5.4 Release Validation Layer

The release validation layer SHOULD define the minimum frontend behavior required before merge, release, or deployment.

It MAY include smoke tests, selected core user flow tests, and selected regression tests.

The release validation baseline MUST be explicit in the project.

### 5.5 Optional Extended Layers

The following layers are optional and MUST NOT be assumed by default:

- visual regression checks
- accessibility checks
- performance checks
- cross-browser checks
- mobile viewport checks
- Playwright component tests

Optional layers MAY be added only when explicitly adopted.

---

## 6. User Interaction Strategy

Playwright tests SHOULD simulate important user interactions through automated browser control.

Tests SHOULD open pages, click controls, fill forms, submit actions, wait for navigation or UI state changes, and assert visible results.

Tests SHOULD prioritize user-facing outcomes over implementation details.

Tests MUST NOT primarily validate internal component state, private implementation details, or unstable DOM structure.

Preferred pattern:

```text
user action -> visible system response -> stable assertion
```

---

## 7. Diagnostics Strategy

The diagnostics strategy MUST be lightweight by default.

Default diagnostic policy:

- assertion output is the default diagnostic entrypoint
- trace is disabled by default
- screenshot is disabled by default
- video is disabled by default
- HTML report is disabled by default

When a Playwright test fails, review diagnostics in this order:

1. assertion output
2. trace, only if assertion output is insufficient and trace was explicitly enabled

Trace MAY be enabled temporarily when assertion output is insufficient.

Screenshot, video, or HTML report MAY be enabled temporarily only when assertion output and trace are insufficient.

Screenshot, video, and HTML report MUST NOT be part of the default baseline unless explicitly required.

---

## 8. Final Strategy Rule

Frontend Playwright testing MUST remain lightweight by default, formalized through `@playwright/test`, user-flow-oriented, stable, repeatable, readable, maintainable, focused on critical frontend behavior, and explicit about local, CI, regression, and release validation requirements.
