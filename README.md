# Engineering Conventions

A versioned library of engineering convention documents for consistent, reusable, and AI-friendly software development workflows.

## Purpose

This repository maintains reusable engineering convention documents.

Each convention defines rules, boundaries, naming patterns, file responsibilities, and implementation constraints for a specific engineering area.

The documents are intended for both human developers and AI coding agents.

---

## Repository Model

Each convention is managed independently under:

```text
conventions/<convention-name>/
```

Each convention directory should contain:

```text
<CONVENTION_DOCUMENT>.md
CHANGELOG.md
VERSION
```

Example:

```text
conventions/
  deployment/
    DEPLOYMENT_CONVENTION.md
    CHANGELOG.md
    VERSION
```

Each convention has its own:

1. convention document
2. version number
3. changelog
4. Git tag
5. GitHub Release

---

## Conventions

| Convention            | Version | Status | Document                                          |
| --------------------- | ------: | ------ | ------------------------------------------------- |
| Deployment Convention |   1.0.0 | Active | `conventions/deployment/DEPLOYMENT_CONVENTION.md` |

More conventions may be added later, such as:

* API Convention
* Frontend Convention
* Backend Convention
* Testing Convention
* Release Convention
* AI-assisted Development Convention

---

## Versioning

Each convention is versioned independently using semantic versioning:

```text
MAJOR.MINOR.PATCH
```

Use the version number to describe the impact of a convention change.

### PATCH

Use `PATCH` for wording, formatting, typo fixes, or non-behavioral clarifications.

Example:

```text
1.0.0 → 1.0.1
```

### MINOR

Use `MINOR` for new compatible rules, additional guidance, or new constraints that do not break the existing convention model.

Example:

```text
1.0.0 → 1.1.0
```

### MAJOR

Use `MAJOR` for breaking changes to the convention model.

Example:

```text
1.0.0 → 2.0.0
```

A breaking change is a change that requires existing users of the convention to significantly adjust their documents, project structure, workflow, or implementation.

The version number must stay consistent across:

1. the convention document metadata
2. the `VERSION` file
3. the latest entry in `CHANGELOG.md`

---

## Tag Naming

Stable convention versions should be tagged with this pattern:

```text
<convention-name>-convention-vX.Y.Z
```

Examples:

```text
deployment-convention-v1.0.0
api-convention-v1.0.0
frontend-convention-v1.0.0
```

Do not use generic repository-wide tags such as:

```text
v1.0.0
```

Each convention is versioned independently, so tags should identify both the convention and the version.

---

## Release Process

Use this process to publish a stable version of any convention.

### 1. Update the convention document

Update the relevant convention document.

Example:

```text
conventions/<convention-name>/<CONVENTION_DOCUMENT>.md
```

Make sure the document metadata is correct:

```md
Convention: <Convention Name>  
Version: X.Y.Z  
Status: Active  
Last Updated: YYYY-MM-DD  
Maintainer: <maintainer-name-or-team>
```

### 2. Update `VERSION`

Update the convention `VERSION` file:

```text
conventions/<convention-name>/VERSION
```

The file should contain only the version number:

```text
X.Y.Z
```

### 3. Update `CHANGELOG.md`

Update the convention changelog:

```text
conventions/<convention-name>/CHANGELOG.md
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
git commit -m "<convention-name>: release convention vX.Y.Z"
```

Example:

```bash
git commit -m "deployment: release convention v1.0.0"
```

### 5. Create a Git tag

```bash
git tag <convention-name>-convention-vX.Y.Z
```

Example:

```bash
git tag deployment-convention-v1.0.0
```

### 6. Push the branch and tag

```bash
git push origin main
git push origin <convention-name>-convention-vX.Y.Z
```

Example:

```bash
git push origin main
git push origin deployment-convention-v1.0.0
```

### 7. Create a GitHub Release

In GitHub, open:

```text
Releases → Draft a new release
```

Use:

```text
Tag: <convention-name>-convention-vX.Y.Z
Release title: <Convention Name> vX.Y.Z
```

For a stable version, do not select:

```text
Set as a pre-release
```

Release notes should summarize what changed in that convention version.

---

## Release Notes Template

```md
# <Convention Name> vX.Y.Z

Briefly describe this convention release.

## Highlights

- ...
- ...
- ...

## Notes

- ...
```

---

## Release Checklist

Before publishing a convention release, verify that:

* [ ] The convention document has the correct version.
* [ ] The `VERSION` file matches the document version.
* [ ] `CHANGELOG.md` contains an entry for the release version.
* [ ] The Git tag follows the convention-specific tag format.
* [ ] The GitHub Release title matches the convention version.
* [ ] Stable releases are not marked as pre-releases.

---

## Status

Convention documents may use one of the following statuses:

```text
Draft
Active
Deprecated
Archived
```

### Draft

The convention is still being discussed and should not be treated as stable.

### Active

The convention is stable and can be used by projects.

### Deprecated

The convention is no longer recommended, but remains available for compatibility or historical reference.

### Archived

The convention is no longer maintained.

---

## Change Policy

All meaningful convention changes should update:

1. the convention document
2. the convention `VERSION`
3. the convention `CHANGELOG.md`

Prefer focused changes.

Do not mix unrelated convention changes in a single release.

---

## License

To be decided.
