# Playwright Test Failure Diagnosis Playbook

Playbook: Playwright Test Failure Diagnosis Playbook  
Version: 1.0.0  
Status: Active  
Last Updated: 2026-05-04  
Maintainer: Charlie Mi <txqq3116934585@gmail.com>

## 1. Purpose

This playbook defines the step-by-step procedure for diagnosing failed Playwright frontend tests.

It follows the lightweight diagnostics strategy:

- assertion output first
- trace only when needed
- screenshot, video, and HTML report disabled by default

---

## 2. Diagnosis Principles

A failed formal Playwright test MUST be treated as the first source of truth.

Manual browser inspection MUST NOT override a formal failed test result.

Auxiliary tools MAY help investigation, but they MUST NOT replace formal Playwright execution.

Do not remove or weaken important assertions only to make a test pass.

---

## 3. Default Diagnosis Order

When a Playwright test fails, follow this order:

1. Read assertion output.
2. Identify the failing spec, test title, and assertion line.
3. Re-run the failing spec only.
4. Decide whether the product behavior is broken or the test is outdated.
5. Enable trace temporarily only if assertion output is insufficient.
6. Update the correct test asset.
7. Re-run the affected spec.
8. Re-run the required baseline if needed.

---

## 4. Step 1: Review Assertion Output

Review:

- failing test name
- failing file
- failing line
- expected value
- actual value
- timeout message
- locator error
- navigation or visibility error

Use assertion output as the default diagnostic entrypoint.

Do not enable trace, screenshot, video, or HTML report before assertion output has been reviewed.

---

## 5. Step 2: Re-run the Failing Spec

Run only the failing spec:

```bash
pnpm exec playwright test tests/e2e/<flow-name>.spec.ts
```

If needed, run a specific test by title:

```bash
pnpm exec playwright test tests/e2e/<flow-name>.spec.ts -g "<test title>"
```

Use targeted execution before running the full test suite again.

---

## 6. Step 3: Classify the Failure

Classify the failure as one of:

- product behavior broken
- expected behavior intentionally changed
- selector outdated
- assertion too strict
- fixture or test data invalid
- environment or configuration issue
- flaky timing or waiting issue
- test implementation issue

Do not update the test until the failure class is understood.

---

## 7. Step 4: Enable Trace Only If Needed

Enable trace temporarily only when assertion output is insufficient:

```bash
pnpm exec playwright test tests/e2e/<flow-name>.spec.ts --trace on
```

Trace SHOULD NOT be enabled by default in `playwright.config.ts`.

After diagnosis, do not leave trace enabled by default unless the project explicitly changes the diagnostics policy.

---

## 8. Step 5: Use Heavier Diagnostics Only as an Exception

Screenshot, video, or HTML report MAY be enabled temporarily only when assertion output and trace are insufficient.

They MUST NOT be part of the default diagnosis workflow.

If enabled temporarily, remove or disable them after diagnosis unless the project explicitly adopts them.

---

## 9. Step 6: Update the Correct Asset

When behavior changes intentionally:

- update the spec if expected behavior changed
- update helpers if repeated actions changed
- update fixtures if scenario data changed
- update config if execution environment changed
- update package scripts if command entrypoints changed

Do not fix a spec by hiding test intent inside helpers.

Do not weaken important assertions only to make the test pass.

---

## 10. Step 7: Stabilize Flaky Tests

For flaky tests:

- prefer stable selectors
- avoid fixed sleeps unless there is no better option
- wait for user-visible states
- assert stable outcomes
- isolate test data
- remove hidden cross-test dependencies

Do not delete flaky tests before diagnosing why they are flaky.

---

## 11. Step 8: Re-run Validation

After a fix:

1. Re-run the affected spec.
2. Re-run related specs if shared helpers or fixtures changed.
3. Re-run the required baseline if the change affects release or regression confidence.

Examples:

```bash
pnpm exec playwright test tests/e2e/<flow-name>.spec.ts
pnpm test:e2e
```

---

## 12. Final Rule

A Playwright failure is resolved only when:

- the cause is understood
- the correct asset is updated
- the affected spec passes through the formal Playwright runner
- no important assertion has been weakened without an intentional behavior change
