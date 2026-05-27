# Database Migration and Seed Guide

Guide: Database Migration and Seed Guide  
Version: 1.1.0  
Status: Active  
Last Updated: 2026-05-27  
Maintainer: Charlie Mi <txqq3116934585@gmail.com>

## 1. Purpose

This guide defines how database migrations, seed scripts, bootstrap scripts, and test database setup should be handled in Docker Compose deployments.

## 2. Migration, Seed, and Startup Responsibilities

Separate these responsibilities conceptually:

| Action | Responsibility |
|---|---|
| `migrate` | Apply schema changes. |
| `seed` | Insert required initial data or environment-specific data. |
| `start` | Start the application service. |

Do not hide schema or data changes inside ordinary API startup unless the project explicitly adopts that deployment model.

## 3. Default Policy

Default policy:

```text
API startup should not implicitly run schema migrations by default.
```

Production migrations should preferably run as explicit one-off jobs.

Development and test environments may provide convenience bootstrap flows.

## 4. Recommended Production Pattern: One-Off Migration Job

Define a dedicated migration service using the API image:

```yaml
services:
  migrate:
    image: <private-registry>/<project-slug>-api:<pinned-version>
    command: <migration-command>
    depends_on:
      db:
        condition: service_healthy
    environment:
      DATABASE_URL: ${DATABASE_URL}
      CI: "true"
    networks:
      - app_net
    profiles:
      - tools
```

Run it explicitly:

```bash
docker compose --profile tools run --rm migrate
```

Then start or update application services:

```bash
docker compose up -d
```

## 5. Development Bootstrap Pattern

Development may provide a bootstrap service that runs migrations and development seed data:

```yaml
services:
  bootstrap-dev:
    build:
      context: ./apps/api
      dockerfile: Dockerfile
    command: sh -c "<migration-command> && <dev-seed-command>"
    depends_on:
      db:
        condition: service_healthy
    environment:
      DATABASE_URL: ${DATABASE_URL}
      CI: "true"
    networks:
      - app_net
    profiles:
      - tools
```

Run it explicitly:

```bash
docker compose -f docker-compose.dev.yml --profile tools run --rm bootstrap-dev
```

## 6. Test Database Setup Pattern

Test environments may automatically reset, migrate, seed fixtures, and run tests because test data is disposable.

Example:

```yaml
services:
  test:
    build:
      context: ./apps/api
      dockerfile: Dockerfile
    command: sh -c "<migration-command> && <test-seed-command> && <test-command>"
    depends_on:
      db:
        condition: service_healthy
    environment:
      DATABASE_URL: ${TEST_DATABASE_URL}
      CI: "true"
    networks:
      - app_net
```

Test failures must be represented by process exit code.

## 7. Production Seed Policy

Production seed scripts must be limited to required bootstrap data.

Allowed production seed examples:

1. Required roles.
2. Required permissions.
3. Required system settings.
4. Required lookup tables.
5. Required feature flag defaults.

Forbidden default production seed examples:

1. Demo users.
2. Fake projects.
3. Test fixtures.
4. Sample orders.
5. Tutorial-only data.

## 8. Startup Migration Policy

Startup migration means the API startup command runs migrations before starting the API.

Example:

```bash
<migration-command> && <api-start-command>
```

This is allowed only when explicitly documented and safe.

Requirements:

1. The deployment is single-instance or migration concurrency is controlled.
2. The migration tool is safe for repeated execution.
3. Seed scripts are idempotent if included.
4. Failure causes the container to fail fast.
5. The behavior is visible in deployment documentation.
6. The production risk is accepted by the project.

Do not use startup migration as the default pattern for production without a decision.

## 9. Idempotency Requirements

Seed scripts must be idempotent.

Running the same seed script multiple times must not create duplicate or inconsistent data.

Use stable keys, upsert behavior, existence checks, or migration-managed data inserts.

## 10. Failure Behavior

Migration and seed failures must be explicit.

Do not silently ignore migration failures.

Do not continue to API startup after a failed required migration.

Recommended shell behavior:

```bash
set -e
<migration-command>
<seed-command>
```

## 11. Multi-Instance and Concurrency Risks

If multiple API containers start at the same time, startup migration may run concurrently.

Risks include:

1. Lock contention.
2. Duplicate seed data.
3. Partial migrations.
4. Confusing startup failures.
5. Multiple containers attempting the same destructive operation.

Prefer one-off migration jobs for production multi-instance deployments.

## 12. Rollback and Recovery Considerations

Schema changes may not be reversible.

Before production migration, projects should know:

1. Whether the migration is backward-compatible.
2. Whether the previous application version can run against the new schema.
3. Whether a rollback migration exists.
4. How backups are created and restored.
5. Who approves destructive migrations.

## 13. Common Migration Commands

Examples are project-specific and should not be treated as mandatory.

```bash
alembic upgrade head
pnpm prisma migrate deploy
npm run migrate
python manage.py migrate
```

Choose the command that matches the backend framework and migration tool.

## 14. AI Coding Agent Constraints

AI coding agents must:

1. Not add automatic production startup migrations unless requested.
2. Prefer one-off migration jobs for production examples.
3. Keep migration, seed, and application startup responsibilities clear.
4. Not insert demo or test data into production seed flows.
5. Make seed scripts idempotent when generating them.
6. Add `CI=true` or `CI=1` when non-interactive behavior is required.
7. Document any startup-migration model clearly.
