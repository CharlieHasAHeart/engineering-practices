# Docker Compose File Convention

Convention: Docker Compose File Convention  
Version: 1.1.0  
Status: Active  
Last Updated: 2026-05-27  
Maintainer: Charlie Mi <txqq3116934585@gmail.com>

## 1. Purpose

This document defines how Docker Compose files should be written for projects following the Deployment practice area.

It applies to:

```text
docker-compose.dev.yml
docker-compose.test.yml
docker-compose.yml
release/docker-compose.yml
```

For deployment principles, see `DEPLOYMENT_STRATEGY.md`.

## 2. Standard Compose Files

Recommended file responsibilities:

| File | Responsibility |
|---|---|
| `docker-compose.dev.yml` | Local development with hot reload and development-only convenience settings. |
| `docker-compose.test.yml` | Automated test setup, optional. |
| `docker-compose.yml` | Production or production-like runtime. |
| `release/docker-compose.yml` | Packaged release deployment, optional. |

## 3. Top-Level Order

Compose files should use this top-level order:

```yaml
services:
volumes:
networks:
```

Use comments sparingly and only when they explain project-specific choices.

## 4. Recommended Service Order

For the minimal topology, order services by external traffic flow and dependency depth:

```yaml
services:
  nginx:
  web:
  api:
  db:
```

For extended topology, keep the same principle:

```yaml
services:
  nginx:
  web:
  admin-web:
  api:
  worker:
  scheduler:
  db:
  redis:
```

## 5. Service Field Order

Within each service, use this order when practical:

```yaml
image or build
container_name
user
depends_on
environment
env_file
volumes
command
expose
ports
networks
healthcheck
restart
read_only
tmpfs
cap_drop
security_opt
profiles
```

Keep service definitions readable. Do not over-optimize for alphabetical order.

## 6. Gateway Port Publishing

Only the gateway service may use `ports` by default.

Example with default local port:

```yaml
services:
  nginx:
    ports:
      - "8080:80"
```

Example with IP-based production access on port `3015`:

```yaml
services:
  nginx:
    ports:
      - "3015:80"
```

Do not publish host ports for internal services:

```yaml
web:
  ports:
    - "3000:3000"

api:
  ports:
    - "8000:8000"

db:
  ports:
    - "5432:5432"
```

## 7. Internal Service Exposure

Internal services may use `expose` when useful for documentation or internal service visibility.

```yaml
web:
  expose:
    - "3000"

api:
  expose:
    - "8000"

db:
  expose:
    - "5432"
```

`expose` does not publish a host port.

## 8. Network Convention

Use one internal network for the standard topology.

```yaml
networks:
  app_net:
    name: <project-slug>-app-net
    driver: bridge
```

All core services should join `app_net`.

## 9. Volume Convention

Persistent service data should use named volumes.

PostgreSQL example:

```yaml
volumes:
  postgres_data:
    name: <project-slug>-postgres-data-prod
```

Development-only dependency and cache volumes are allowed in `docker-compose.dev.yml`, but must not be copied into production Compose files.

## 10. Container Naming Convention

Container names should follow:

```text
<project-slug>-<service>-<environment>
```

Examples:

```text
example-app-nginx-dev
example-app-web-dev
example-app-api-dev
example-app-db-dev
example-app-nginx-prod
```

If a project avoids explicit `container_name` to allow scaling, document that decision.

## 11. Environment Variables

Prefer explicit environment keys for important runtime behavior:

```yaml
environment:
  DATABASE_URL: ${DATABASE_URL}
  CI: "true"
```

Use `env_file` only when the file is part of the project deployment model.

Do not commit production secret values.

## 12. CI Mode

Use one consistent CI representation per project.

Recommended:

```yaml
environment:
  CI: "true"
```

Accepted alternative:

```yaml
environment:
  CI: "1"
```

Do not add `CI` to every service automatically. Add it where non-interactive behavior is required.

## 13. Development Compose Requirements

`docker-compose.dev.yml` may use:

1. Local `build` instructions.
2. Source-code bind mounts.
3. Hot reload commands.
4. Named volumes for dependencies and framework caches.
5. Host-aligned users.
6. Polling-based watcher settings.
7. Safe local defaults.

It must still:

1. Publish only the gateway host port.
2. Keep internal services internal.
3. Preserve `/api/` routing.
4. Use named volumes for database persistence.

## 14. Test Compose Requirements

`docker-compose.test.yml` may:

1. Use resettable or disposable database volumes.
2. Run migrations and test seed scripts automatically.
3. Set `CI=true` or `CI=1`.
4. Run test commands as the primary container command.

It must not:

1. Reuse production secrets.
2. Reuse production database volumes.
3. Require interactive input.
4. Depend on direct host ports for internal services.

## 15. Production Compose Requirements

`docker-compose.yml` should:

1. Use pinned runtime images where possible.
2. Avoid source-code bind mounts.
3. Avoid `node_modules` or framework cache runtime volumes.
4. Avoid hot reload commands.
5. Publish only the gateway port.
6. Keep `web`, `api`, `db`, and infrastructure services internal.
7. Use named volumes for persistent data.
8. Use stable restart policies.
9. Use healthchecks where readiness matters.
10. Apply runtime hardening where compatible.

## 16. Release Compose Requirements

`release/docker-compose.yml`, when present, must remain aligned with production Compose.

Allowed differences include image tags, registry source, gateway host port, and deployment domain.

Forbidden differences include exposing internal services, weakening hardening without explanation, or reusing development/test volumes.

## 17. Healthcheck Convention

Use healthchecks when readiness affects dependent services.

Database example:

```yaml
db:
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U <database-user> -d <database-name>"]
    interval: 10s
    timeout: 5s
    retries: 5
```

API example:

```yaml
api:
  healthcheck:
    test: ["CMD", "sh", "-c", "wget -qO- http://localhost:8000/health >/dev/null 2>&1 || exit 1"]
    interval: 30s
    timeout: 5s
    retries: 3
```

## 18. Runtime Hardening

Production services should apply hardening where compatible:

```yaml
read_only: true
tmpfs:
  - /tmp
cap_drop:
  - ALL
security_opt:
  - no-new-privileges:true
```

Do not apply hardening blindly if the application requires writable runtime paths. Instead, document explicit writable paths.

## 19. Profiles

Use Compose profiles for one-off or optional services such as migration, seed, bootstrap, or development tooling.

```yaml
profiles:
  - tools
```

Routine application startup should not require enabling unrelated tool profiles.

## 20. AI Coding Agent Constraints

AI coding agents must:

1. Preserve the existing Compose service model.
2. Avoid renaming services during routine edits.
3. Use `ports` only on the gateway by default.
4. Avoid adding direct host ports for `web`, `api`, `db`, or infrastructure services.
5. Avoid copying development-only volumes into production.
6. Avoid adding unrelated environment variables.
7. Keep examples project-neutral with placeholders.
