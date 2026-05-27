# Deployment Practice Area

Practice Area: Deployment  
Version: 1.1.0  
Status: Active  
Last Updated: 2026-05-27  
Maintainer: Charlie Mi <txqq3116934585@gmail.com>

## Purpose

This practice area defines reusable deployment strategy, Docker Compose conventions, development convenience patterns, database migration and seed guidance, review checklists, references, and examples for containerized web applications.

The documentation is designed for both human developers and AI coding agents.

## Document Map

| Document | Type | Purpose |
|---|---|---|
| `DEPLOYMENT_STRATEGY.md` | Strategy | Defines deployment services, runtime roles, environment strategy, gateway boundaries, persistence, reliability, migration policy summary, and AI constraints. |
| `DOCKER_COMPOSE_FILE_CONVENTION.md` | Convention | Defines how Docker Compose files should be written across development, test, production, and release contexts. |
| `guides/DOCKER_COMPOSE_DEVELOPMENT_CONVENIENCE_GUIDE.md` | Guide | Provides development-only patterns such as bind mounts, named volumes for dependencies, Next.js/Vite examples, polling watchers, and host-aligned users. |
| `guides/DATABASE_MIGRATION_AND_SEED_GUIDE.md` | Guide | Defines migration, seed, bootstrap, and test database setup patterns. |
| `references/DEPLOYMENT_SERVICE_NAMING_REFERENCE.md` | Reference | Defines default and extended deployment service names. |
| `references/ENVIRONMENT_VARIABLE_REFERENCE.md` | Reference | Defines common environment variables used by Compose deployment files. |
| `checklists/DEPLOYMENT_REVIEW_CHECKLIST.md` | Checklist | General deployment review checklist. |
| `checklists/PRODUCTION_COMPOSE_CHECKLIST.md` | Checklist | Production Compose-specific checklist. |
| `examples/` | Examples | Copyable examples for common Compose patterns. |

## Reading Order

Recommended reading order:

1. `DEPLOYMENT_STRATEGY.md`
2. `DOCKER_COMPOSE_FILE_CONVENTION.md`
3. `guides/DOCKER_COMPOSE_DEVELOPMENT_CONVENIENCE_GUIDE.md`
4. `guides/DATABASE_MIGRATION_AND_SEED_GUIDE.md`
5. `checklists/DEPLOYMENT_REVIEW_CHECKLIST.md`
6. `checklists/PRODUCTION_COMPOSE_CHECKLIST.md`
7. `references/`
8. `examples/`

## Normative Strength

The strategy and convention documents are normative for this practice area.

Guides provide recommended implementation patterns and examples.

References provide stable lookup information.

Examples are illustrative starting points and may be adapted to project-specific needs.

## Core Principle

Deployment should be described through stable deployment services and runtime roles.

The default minimal Compose topology is:

```text
nginx
web
api
db
```

These names are defaults for simple full-stack applications, not a universal limit on project architecture.

Projects may extend or specialize the deployment topology when the architecture requires it, but service names must be stable, role-revealing, and documented.
