# Playwright Test Review Checklist

Checklist: Playwright Test Review Checklist  
Version: 1.0.0  
Status: Draft  
Last Updated: 2026-05-04  
Maintainer: Charlie Mi <txqq3116934585@gmail.com>

## 1. Purpose

Use this checklist before accepting new or updated Playwright frontend tests.

The checklist is intended for coding agents, developers, and reviewers.

---

## 2. Test Scope

- [ ] The test protects user-visible frontend behavior.
- [ ] The test covers a critical flow, smoke path, regression path, or release-relevant path.
- [ ] The test is not trying to cover every component detail.
- [ ] The test is not an exhaustive edge-case matrix by default.
- [ ] The test belongs in Playwright rather than unit, component, API, or manual testing.

---

## 3. Formal Test Authority

- [ ] The behavior is implemented as an `@playwright/test` spec if it must be part of the baseline.
- [ ] Manual inspection is not used as the formal pass/fail result.
- [ ] Playwright CLI or Skills exploration is not treated as a substitute for the formal spec.
- [ ] The test can be executed through the Playwright test runner.

---

## 4. File Location and Naming

- [ ] Spec files are under `tests/e2e/**/*.spec.ts`.
- [ ] Helper files are under `tests/helpers/**/*.ts`.
- [ ] Fixture files are under `tests/fixtures/**/*.ts`.
- [ ] Spec names are descriptive and kebab-case.
- [ ] File names avoid vague names such as `test1`, `final`, `misc`, `temp`, or `data2`.

---

## 5. Spec Readability

- [ ] The spec reads as a clear user-flow narrative.
- [ ] The spec expresses user actions and visible outcomes.
- [ ] The test intent is not hidden behind excessive helper abstraction.
- [ ] The spec is not a giant mixed-purpose file.
- [ ] Temporary debugging code has been removed.

---

## 6. User Interaction

- [ ] The test opens pages as a user would.
- [ ] The test clicks controls as a user would.
- [ ] The test fills forms as a user would.
- [ ] The test submits actions as a user would.
- [ ] The test waits for navigation or visible UI state changes.
- [ ] The test prioritizes user-visible outcomes over implementation details.

---

## 7. Selectors

- [ ] The test prefers `data-testid`.
- [ ] Stable roles are used when appropriate.
- [ ] Stable labels are used when appropriate.
- [ ] Stable business text is used only when appropriate.
- [ ] The test does not primarily rely on CSS classes.
- [ ] The test does not primarily rely on deep DOM paths.
- [ ] The test does not rely on `nth-child` as a primary selector.
- [ ] The test does not depend on layout-specific selectors.
- [ ] Incidental or frequently changing copy is not overused as a selector.

---

## 8. Assertions

- [ ] Assertions are few, stable, and high-value.
- [ ] Assertions validate structural presence when relevant.
- [ ] Assertions validate flow success when relevant.
- [ ] Assertions validate key business values when relevant.
- [ ] Assertions validate stable user-visible outcomes.
- [ ] Assertions are not overbound to styling details.
- [ ] Assertions are not pixel-sensitive by default.
- [ ] Assertions do not depend on unstable DOM shape.
- [ ] Important assertions have not been weakened just to make the test pass.

---

## 9. Test Data

- [ ] UI-created data is used only when validating creation, edit, submission, or true end-to-end flows.
- [ ] Pre-seeded data is used for stable downstream behavior when appropriate.
- [ ] Test data has clear ownership.
- [ ] The test avoids hidden cross-test dependencies.
- [ ] The test is repeatable.
- [ ] The test is isolated where reasonably possible.

---

## 10. Helpers and Fixtures

- [ ] Helpers extract repeated actions, assertions, or setup patterns.
- [ ] Helpers do not hide the purpose of the test.
- [ ] Helpers are not heavy page object hierarchies by default.
- [ ] Fixtures remain data-focused.
- [ ] Fixtures do not mix unclear logic and data.

---

## 11. Diagnostics Defaults

- [ ] Assertion output is the default diagnostic entrypoint.
- [ ] Trace is not enabled by default.
- [ ] Screenshot is not enabled by default.
- [ ] Video is not enabled by default.
- [ ] HTML report is not enabled by default.
- [ ] Trace is used temporarily only when assertion output is insufficient.
- [ ] Screenshot, video, or HTML report are used only as temporary exceptions.

---

## 12. Execution

- [ ] The test can be run through the project package manager and script runner.
- [ ] The test can be run as part of `test:e2e`.
- [ ] The test can be run as a targeted spec.
- [ ] CI and release validation impact is clear.
- [ ] Package scripts remain stable and non-overlapping.

---

## 13. Maintenance

- [ ] If behavior changed intentionally, the spec was updated.
- [ ] If repeated actions changed, helpers were updated.
- [ ] If scenario data changed, fixtures were updated.
- [ ] If execution environment changed, config was updated.
- [ ] If command entrypoints changed, package scripts were updated.
- [ ] Flaky tests were diagnosed and stabilized instead of deleted without reason.

---

## 14. Final Acceptance

A Playwright test change is acceptable only if:

- [ ] it protects meaningful frontend behavior
- [ ] it is readable
- [ ] it is repeatable
- [ ] it is stable
- [ ] it follows selector and assertion rules
- [ ] it keeps diagnostics lightweight by default
- [ ] it can be executed by the formal Playwright test runner
