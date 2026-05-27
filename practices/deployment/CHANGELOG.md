# Deployment Practice Changelog

## [1.1.0] - 2026-05-27

### Added

- Added `DEPLOYMENT_STRATEGY.md` as the main deployment strategy document.
- Added `DOCKER_COMPOSE_FILE_CONVENTION.md` for Docker Compose file-writing rules.
- Added development convenience guide for bind mounts, dependency volumes, framework cache volumes, host-aligned users, and watcher settings.
- Added database migration and seed guide.
- Added deployment service naming reference.
- Added environment variable reference.
- Added deployment and production Compose review checklists.
- Added Docker Compose example files for Next.js, Vite, production, and test setups.

### Changed

- Reorganized the Deployment practice area into a document system rather than a single long convention document.
- Clarified that `nginx`, `web`, `api`, and `db` are default minimal service names, not a universal limit on project architecture.
- Clarified that only the gateway service should publish host ports.
- Clarified that production access may use either a DNS name or an IP address with a published gateway port.

### Fixed

- None.

### Migration Notes

- Existing projects using `DEPLOYMENT_CONVENTION.md` can gradually adopt the new document system.
- No immediate project structure migration is required.
- Future updates should prefer editing the specialized document that matches the topic instead of expanding a single convention document indefinitely.

## [1.0.0] - 2026-05-02

### Added

- Added initial active deployment convention.
- Defined standard Docker Compose topology with `nginx`, `web`, `api`, and `db`.
- Defined unified browser access model through `nginx`.
- Defined service naming, container naming, network naming, and volume naming conventions.
- Defined Nginx routing rules for `/api/` and `/`.
- Defined port exposure rules.
- Defined Compose, Dockerfile, environment variable, database, healthcheck, and release alignment guidance.
- Added AI coding agent constraints for deployment-related file generation and editing.

### Changed

- None.

### Fixed

- None.

### Migration Notes

- None. This is the initial stable version.
