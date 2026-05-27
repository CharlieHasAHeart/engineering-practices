# Deployment Review Checklist

Checklist: Deployment Review Checklist  
Version: 1.1.0  
Status: Active  
Last Updated: 2026-05-27  
Maintainer: Charlie Mi <txqq3116934585@gmail.com>

## Deployment Service Model

- [ ] Deployment services are clearly identified.
- [ ] Service names are stable and role-revealing.
- [ ] Default service names are used when no project-specific topology exists.
- [ ] Additional services have documented responsibilities.
- [ ] AI-generated changes did not introduce unnecessary services.

## Gateway and Access

- [ ] Only the gateway publishes host ports by default.
- [ ] `web`, `api`, `db`, and infrastructure services remain internal.
- [ ] IP-based access, if used, maps the published port to the gateway only.
- [ ] Browser access goes through the gateway.
- [ ] Browser-side API calls use `/api/...` or another documented gateway route.

## Routing

- [ ] Gateway routes `/` to the web service.
- [ ] Gateway routes `/api/` to the API service.
- [ ] `/api/` prefix behavior is documented.
- [ ] No unexpected rewrite hides the API contract.

## Environment Separation

- [ ] Development, test, and production use separate data volumes.
- [ ] Test does not reuse production secrets.
- [ ] Development-only convenience settings are not copied into production.
- [ ] CI/non-interactive mode is used where required.

## Persistence and Secrets

- [ ] PostgreSQL uses a named volume or approved persistent storage.
- [ ] Real production secrets are not committed.
- [ ] `.env.example` uses placeholders or safe local defaults.
- [ ] Database URLs use service names inside containers.

## Migration and Seed

- [ ] Migration strategy is documented.
- [ ] Production does not implicitly run migrations on API startup unless explicitly approved.
- [ ] One-off migration job exists if production requires controlled migrations.
- [ ] Seed scripts are environment-aware and idempotent.
- [ ] Demo/test seed data is not inserted into production by default.

## Reliability

- [ ] Readiness-dependent services have healthchecks.
- [ ] Critical startup dependencies use readiness-aware ordering where needed.
- [ ] Production services use stable restart policies.
- [ ] Runtime hardening is applied where compatible.
