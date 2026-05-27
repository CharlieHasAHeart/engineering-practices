# Production Compose Checklist

Checklist: Production Compose Checklist  
Version: 1.1.0  
Status: Active  
Last Updated: 2026-05-27  
Maintainer: Charlie Mi <txqq3116934585@gmail.com>

## Production File Scope

- [ ] `docker-compose.yml` represents production or production-like runtime.
- [ ] It does not rely on local development assumptions.
- [ ] It does not require `--build` unless explicitly supported by the deployment process.

## Images

- [ ] Runtime images are pinned where possible.
- [ ] Image source is consistent and documented.
- [ ] Private registry usage is documented if required.
- [ ] Public and private image sources are not mixed without explanation.

## Ports

- [ ] Only the gateway service uses `ports` by default.
- [ ] `web` does not publish host ports.
- [ ] `api` does not publish host ports.
- [ ] `db` does not publish host ports.
- [ ] Published gateway port is documented, such as `80`, `443`, `8080`, or `3015`.

## Development-Only Settings Excluded

- [ ] No source-code bind mounts.
- [ ] No `node_modules` runtime volume.
- [ ] No framework cache runtime volume such as `.next`, `.vite`, or `.turbo`.
- [ ] No hot reload command.
- [ ] No polling watcher settings unless explicitly justified.
- [ ] No unsafe local development secrets.

## Environment and Secrets

- [ ] Required environment variables are documented.
- [ ] Production secrets are provided by the deployment environment.
- [ ] `CI=true` or `CI=1` is set only where required.
- [ ] Database URLs use internal service names.

## Persistence

- [ ] Production database uses a production-named volume or approved storage.
- [ ] Production volume is not shared with development or test.
- [ ] Backup and restore expectations are documented where applicable.

## Migrations

- [ ] Production migration command is explicit.
- [ ] Startup migration is not used unless documented and approved.
- [ ] Seed scripts do not insert demo or test data.

## Reliability and Hardening

- [ ] Services use stable restart policies.
- [ ] Healthchecks exist where readiness matters.
- [ ] Non-root users are used where compatible.
- [ ] Read-only root filesystems are used where compatible.
- [ ] Writable paths are explicit.
- [ ] Linux capabilities are minimized where compatible.
- [ ] Privilege escalation is disabled where compatible.
