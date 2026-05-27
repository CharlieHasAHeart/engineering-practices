# Environment Variable Reference

Reference: Environment Variable Reference  
Version: 1.1.0  
Status: Active  
Last Updated: 2026-05-27  
Maintainer: Charlie Mi <txqq3116934585@gmail.com>

## Purpose

This reference defines common environment variables used by Docker Compose deployment files.

Variables listed here are references, not mandatory requirements for every project.

## Compose Variables

| Variable | Environment | Purpose |
|---|---|---|
| `COMPOSE_PROJECT_NAME` | dev/test/prod | Stable Compose project name. |
| `UID` | dev | Host user ID for host-aligned user. |
| `GID` | dev | Host group ID for host-aligned user. |

## Database Variables

| Variable | Environment | Purpose |
|---|---|---|
| `POSTGRES_DB` | dev/test/prod | PostgreSQL database name. |
| `POSTGRES_USER` | dev/test/prod | PostgreSQL user. |
| `POSTGRES_PASSWORD` | dev/test/prod | PostgreSQL password. Must not be a real production secret in committed files. |
| `DATABASE_URL` | dev/test/prod | Application database connection string. |
| `TEST_DATABASE_URL` | test | Test database connection string. |

Container database URLs should use the Compose service name:

```env
DATABASE_URL=postgresql+psycopg://<database-user>:<database-password>@db:5432/<database-name>
```

Do not use `localhost:5432` inside application containers.

## CI Mode Variables

| Variable | Environment | Purpose |
|---|---|---|
| `CI` | test/prod/selected dev tooling | Enables non-interactive behavior where required. |

Recommended value:

```env
CI=true
```

Accepted alternative:

```env
CI=1
```

A project should choose one representation and use it consistently.

## Frontend Development Variables

| Variable | Environment | Purpose |
|---|---|---|
| `CHOKIDAR_USEPOLLING` | dev | Enables polling watcher for frameworks using Chokidar. |
| `WATCHPACK_POLLING` | dev | Enables polling watcher for Webpack/Next.js. |
| `PNPM_STORE_DIR` | dev | Sets pnpm store path, often used with a named volume. |
| `HOME` | dev | May be set to a writable path such as `/tmp` when running as a numeric UID. |

Polling variables should be enabled only when file changes are not detected reliably.

## Secret Rules

1. Do not commit real production secrets.
2. Use placeholders in `.env.example`.
3. Provide production secrets through the deployment environment.
4. Do not treat `CI` as a secret.
