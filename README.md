# Engineering Practices

A versioned library of engineering practice documents for consistent, reusable, and AI-friendly software development workflows.

## Purpose

This repository maintains reusable engineering practice documents.

A practice area may contain one or more document types, such as:

- strategy documents
- convention documents
- playbooks
- checklists
- templates
- references
- decision records

The documents are intended for both human developers and AI coding agents.

They define engineering direction, rules, workflows, review criteria, and maintenance expectations for specific engineering areas.

---

## Repository Model

Each practice area is managed independently under:

```text
practices/<practice-area>/
```

Each practice area should contain:

```text
<STRATEGY_DOCUMENT>.md
<CONVENTION_DOCUMENT>.md
<PLAYBOOK_DOCUMENT>.md
<CHECKLIST_DOCUMENT>.md
CHANGELOG.md
VERSION
```

Not every practice area must contain every document type.

A minimal practice area may contain only one document, such as a convention document.

Example:

```text
practices/
  frontend-playwright-testing/
    FRONTEND_PLAYWRIGHT_TESTING_STRATEGY.md
    FRONTEND_PLAYWRIGHT_TESTING_CONVENTION.md
    PLAYWRIGHT_TEST_FAILURE_DIAGNOSIS_PLAYBOOK.md
    PLAYWRIGHT_TEST_REVIEW_CHECKLIST.md
    CHANGELOG.md
    VERSION
```

Each practice area has its own:

1. document set
2. version number
3. changelog
4. Git tag
5. GitHub Release

---

## Practice Document Types

### Strategy

A strategy document defines direction, scope, priorities, trade-offs, defaults, and boundaries.

It answers:

```text
Why do we do this?
When do we do this?
What do we include by default?
What do we exclude by default?
What trade-offs are intentional?
```

### Convention

A convention document defines concrete rules for structure, naming, files, configuration, commands, and implementation constraints.

It answers:

```text
Where should files go?
How should things be named?
What must be configured?
What must not be done?
```

### Playbook

A playbook defines a step-by-step procedure for a recurring workflow.

It answers:

```text
What should be done first?
What should be done next?
How should failure or exception cases be handled?
```

### Checklist

A checklist defines review or acceptance criteria.

It answers:

```text
What must be checked before accepting, merging, releasing, or completing work?
```

### Template

A template provides a reusable starting point for new files, documents, tests, issues, pull requests, or other repeated work.

It answers:

```text
What should a new item start from?
What required sections or fields should it include?
```

### Reference

A reference document provides stable lookup information, such as commands, environment names, routes, configuration keys, or terminology.

It answers:

```text
Where can I find the exact command, value, name, or definition?
```

### Decision Record

A decision record documents an important technical or process decision, including context, alternatives, decision, consequences, and review conditions.

It answers:

```text
Why did we choose this approach?
What alternatives were considered?
What trade-offs did we accept?
When should this decision be revisited?
```

---

## Practice Areas

| Practice Area | Version | Status | Documents |
|---|---:|---|---|
| Deployment | 1.0.0 | Active | `practices/deployment/DEPLOYMENT_CONVENTION.md` |
| Frontend Playwright Testing | 1.0.0 | Draft | `practices/frontend-playwright-testing/FRONTEND_PLAYWRIGHT_TESTING_STRATEGY.md`, `FRONTEND_PLAYWRIGHT_TESTING_CONVENTION.md`, `PLAYWRIGHT_TEST_FAILURE_DIAGNOSIS_PLAYBOOK.md`, `PLAYWRIGHT_TEST_REVIEW_CHECKLIST.md` |

More practice areas may be added later, such as:

- API
- Frontend
- Backend
- Testing
- Release
- AI-assisted Development
- Performance
- Security

---

## Versioning

Each practice area is versioned independently using semantic versioning:

```text
MAJOR.MINOR.PATCH
```

Use the version number to describe the impact of a practice-area change.

### PATCH

Use `PATCH` for wording, formatting, typo fixes, or non-behavioral clarifications.

Example:

```text
1.0.0 -> 1.0.1
```

### MINOR

Use `MINOR` for new compatible rules, additional guidance, new checklists, new playbook steps, or new constraints that do not break the existing practice model.

Example:

```text
1.0.0 -> 1.1.0
```

### MAJOR

Use `MAJOR` for breaking changes to the practice model.

Example:

```text
1.0.0 -> 2.0.0
```

A breaking change is a change that requires existing users of the practice area to significantly adjust their documents, project structure, workflow, automation, or implementation.

The version number must stay consistent across:

1. the practice documents metadata
2. the `VERSION` file
3. the latest entry in `CHANGELOG.md`

---

## Tag Naming

Stable practice-area versions should be tagged with this pattern:

```text
<practice-area>-vX.Y.Z
```

Examples:

```text
deployment-v1.0.0
frontend-playwright-testing-v1.0.0
api-v1.0.0
```

Do not use generic repository-wide tags such as:

```text
v1.0.0
```

Each practice area is versioned independently, so tags should identify both the practice area and the version.

---

## Release Process

Use this process to publish a stable version of any practice area.

### 1. Update the practice documents

Update the relevant practice-area documents.

Example:

```text
practices/<practice-area>/<DOCUMENT>.md
```

Make sure each updated document metadata is correct.

Example strategy metadata:

```md
Strategy: <Strategy Name>  
Version: X.Y.Z  
Status: Active  
Last Updated: YYYY-MM-DD  
Maintainer: <maintainer-name-or-team>
```

Example convention metadata:

```md
Convention: <Convention Name>  
Version: X.Y.Z  
Status: Active  
Last Updated: YYYY-MM-DD  
Maintainer: <maintainer-name-or-team>
```

Example playbook metadata:

```md
Playbook: <Playbook Name>  
Version: X.Y.Z  
Status: Active  
Last Updated: YYYY-MM-DD  
Maintainer: <maintainer-name-or-team>
```

Example checklist metadata:

```md
Checklist: <Checklist Name>  
Version: X.Y.Z  
Status: Active  
Last Updated: YYYY-MM-DD  
Maintainer: <maintainer-name-or-team>
```

### 2. Update `VERSION`

Update the practice-area `VERSION` file:

```text
practices/<practice-area>/VERSION
```

The file should contain only the version number:

```text
X.Y.Z
```

### 3. Update `CHANGELOG.md`

Update the practice-area changelog:

```text
practices/<practice-area>/CHANGELOG.md
```

Add an entry for the new version:

```md
## [X.Y.Z] - YYYY-MM-DD

### Added

- ...

### Changed

- ...

### Fixed

- ...

### Migration Notes

- ...
```

Use `None.` when a section has no entries.

### 4. Commit the changes

```bash
git add .
git commit -m "<practice-area>: release vX.Y.Z"
```

Example:

```bash
git commit -m "frontend-playwright-testing: release v1.0.0"
```

### 5. Create a Git tag

```bash
git tag <practice-area>-vX.Y.Z
```

Example:

```bash
git tag frontend-playwright-testing-v1.0.0
```

### 6. Push the branch and tag

```bash
git push origin main
git push origin <practice-area>-vX.Y.Z
```

Example:

```bash
git push origin main
git push origin frontend-playwright-testing-v1.0.0
```

### 7. Create a GitHub Release

In GitHub, open:

```text
Releases -> Draft a new release
```

Use:

```text
Tag: <practice-area>-vX.Y.Z
Release title: <Practice Area> vX.Y.Z
```

For a stable version, do not select:

```text
Set as a pre-release
```

Release notes should summarize what changed in that practice-area version.

---

## Release Notes Template

```md
# <Practice Area> vX.Y.Z

Briefly describe this practice-area release.

## Highlights

- ...
- ...
- ...

## Notes

- ...
```

---

## Release Checklist

Before publishing a practice-area release, verify that:

- [ ] All updated documents have the correct version.
- [ ] The `VERSION` file matches the document versions.
- [ ] `CHANGELOG.md` contains an entry for the release version.
- [ ] The Git tag follows the practice-area tag format.
- [ ] The GitHub Release title matches the practice-area version.
- [ ] Stable releases are not marked as pre-releases.

---

## Status

Practice documents may use one of the following statuses:

```text
Draft
Active
Deprecated
Archived
```

### Draft

The document is still being discussed and should not be treated as stable.

### Active

The document is stable and can be used by projects.

### Deprecated

The document is no longer recommended, but remains available for compatibility or historical reference.

### Archived

The document is no longer maintained.

---

## Change Policy

All meaningful practice-area changes should update:

1. the affected practice documents
2. the practice-area `VERSION`
3. the practice-area `CHANGELOG.md`

Prefer focused changes.

Do not mix unrelated practice-area changes in a single release.

---

## License

To be decided.