# Engineering Conventions

A versioned library of engineering convention documents for consistent, reusable, and AI-friendly software development workflows.

## Purpose

This repository maintains engineering convention documents that can be reused across projects.

Each convention defines rules, boundaries, naming patterns, file responsibilities, and implementation constraints for a specific engineering area.

## Conventions

| Convention | Status | Document |
|---|---|---|
| Deployment Convention | Active | `conventions/deployment/DEPLOYMENT_CONVENTION.md` |

More conventions may be added later, such as:

- API Convention
- Frontend Convention
- Backend Convention
- Testing Convention
- Release Convention
- AI-assisted Development Convention

## Versioning

Each convention is versioned independently.

Every convention directory should include:

```text
CONVENTION_DOCUMENT.md
CHANGELOG.md
VERSION
```

Stable versions should be tagged with the following pattern:

```text
<convention-name>-vX.Y.Z
```

Example:

```text
deployment-convention-v1.0.0
```

## Status

Convention documents may use one of the following statuses:

```text
Draft
Active
Deprecated
Archived
```

## Repository Structure

```text
engineering-conventions/
  README.md
  conventions/
    deployment/
      DEPLOYMENT_CONVENTION.md
      CHANGELOG.md
      VERSION
```

## Change Policy

All meaningful convention changes should update:

1. The convention document
2. The convention `VERSION`
3. The convention `CHANGELOG.md`

Use semantic versioning:

```text
MAJOR.MINOR.PATCH
```

* `PATCH`: wording fixes or non-behavioral clarifications
* `MINOR`: new compatible rules or constraints
* `MAJOR`: breaking changes to the convention model

## License

To be decided.
