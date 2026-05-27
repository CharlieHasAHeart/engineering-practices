# Deployment Strategy

Strategy: Deployment Strategy  
Version: 1.1.0  
Status: Active  
Last Updated: 2026-05-27  
Maintainer: Charlie Mi <txqq3116934585@gmail.com>

## 1. Purpose

This document defines the deployment strategy for containerized web applications that use Docker Compose or a similar local-container deployment model.

It defines deployment services, runtime roles, gateway boundaries, environment strategy, network exposure, persistence, migration policy, reliability expectations, and AI coding agent constraints.

This document intentionally avoids most field-level Compose details. Those rules belong in `DOCKER_COMPOSE_FILE_CONVENTION.md`.

## 2. Core Idea: Deployment Services

A deployment is made of deployment services.

A deployment service is a runtime service that participates in serving, supporting, operating, or protecting the application.

Examples include:

```text
nginx
web
api
db
worker
scheduler
redis
queue
search
object-storage
```

The strategy must not assume that every web application has exactly one frontend, one backend, and one database. The minimal topology is a starting point, not an architectural limit.

## 3. Role-Based Deployment Model

This strategy standardizes runtime roles first and Compose service names second.

The minimal standard roles are:

| Role | Default Compose Service Name | Responsibility |
|---|---|---|
| Gateway | `nginx` | The only externally exposed HTTP entrypoint and reverse proxy. |
| Web Application | `web` | Browser-facing frontend application. |
| API Application | `api` | Backend HTTP API service. |
| Primary Database | `db` | Primary PostgreSQL database. |

For simple full-stack applications, use the default service names:

```text
nginx
web
api
db
```

These names are defaults for the minimal topology, not a universal limit on project architecture.

## 4. Minimal Standard Topology

The minimal standard topology is:

```text
Browser
  ↓
gateway / nginx
  ├── /       → web:3000
  └── /api/   → api:8000
                  ↓
                 db:5432
```

This topology is appropriate for applications with one browser-facing frontend, one backend HTTP API, one primary PostgreSQL database, and one external HTTP gateway.

Projects may extend the topology when the product architecture requires additional deployment services.

## 5. Deployment Service Responsibilities

Every deployment service must have a clear runtime responsibility.

A service may exist to provide:

1. External HTTP entry.
2. Frontend UI delivery.
3. Backend API behavior.
4. Persistent storage.
5. Background work.
6. Scheduled work.
7. Cache, queue, or search support.
8. Object storage or file-serving support.
9. Observability or operational support.

Do not add deployment services merely because a tool is common or because an AI coding agent generated it. Each service must be justified by project requirements.

## 6. Gateway Strategy

The gateway is the deployment service that receives external HTTP traffic.

The default gateway implementation is Nginx with the service name `nginx`.

Projects may use another gateway implementation, such as Caddy, Traefik, or Envoy, when the project explicitly adopts that deployment model.

A gateway implementation change must document:

1. The gateway service name.
2. The image or build source.
3. The exposed host port.
4. The route mapping behavior.
5. The differences from the default Nginx model.

Regardless of implementation, the gateway remains the only default externally exposed HTTP service.

## 7. Single External Entry Strategy

Only the gateway service may publish host ports by default.

The following services must remain internal unless explicitly justified by project architecture:

```text
web
api
db
worker
scheduler
redis
queue
search
```

This rule exists to:

1. Keep browser access under one origin.
2. Avoid avoidable CORS drift between environments.
3. Prevent direct host access to internal services.
4. Keep databases and infrastructure services private by default.
5. Make deployment behavior predictable for humans and AI coding agents.

Development is not an exception. Development may be convenient, but it should preserve the same external entry model.

## 8. Unified Browser Access Model

Browser traffic should enter through the gateway.

DNS-based production access may use:

```text
https://<deployment-domain>/
```

IP-based access is acceptable when no DNS name is available:

```text
http://<server-ip>:<published-gateway-port>/
```

For example, a server may publish gateway port `3015`:

```yaml
services:
  nginx:
    ports:
      - "3015:80"
```

The published port must not be mapped directly to `web`, `api`, `db`, or other internal services.

Browser-side API requests should use relative paths:

```text
/api/...
```

Browser-side code must not hardcode internal container addresses such as:

```text
http://localhost:8000
http://api:8000
```

## 9. Route Separation Strategy

The default route split is:

```text
/api/  → api:8000
/      → web:3000
```

The `/api/` prefix should be preserved unless the API design explicitly changes.

Example:

```text
Browser request:
  /api/projects

Gateway forwards to:
  api:8000/api/projects

API route:
  /api/projects
```

Do not introduce path rewriting that hides or changes the API contract without an explicit routing decision.

## 10. Service Naming Stability

Service names should remain stable once adopted by a project.

Routine deployment edits must not rename services.

A service rename is allowed only when:

1. The service role changes.
2. The project introduces multiple services in the same role category.
3. The project adopts a documented naming convention.
4. The gateway implementation changes.
5. The change is recorded as a topology-affecting change.

When no project-specific topology is documented, use the default service names:

```text
nginx
web
api
db
```

AI coding agents must not invent alternative names merely for style preference.

## 11. Topology Extension Policy

Projects may extend or specialize the deployment topology when product architecture requires it.

Examples include:

```text
admin-web
worker
scheduler
redis
queue
search
object-storage
```

Additional services must:

1. Have clear runtime responsibility.
2. Use stable service names.
3. Join the appropriate internal network.
4. Avoid host port exposure unless explicitly required.
5. Use named volumes for persistent data when needed.
6. Define healthchecks when readiness matters.
7. Document required environment variables.
8. Be documented in project deployment or architecture documentation.

## 12. Environment Strategy

The standard environments are:

```text
dev
test
prod
```

Recommended Compose files are:

```text
docker-compose.dev.yml      local development
docker-compose.test.yml     automated or production-like tests, optional
docker-compose.yml          production or production-like runtime
release/docker-compose.yml  packaged release deployment, optional
```

Different environments may differ in image source, commands, mounts, secrets, persistence behavior, CI behavior, and runtime hardening.

They should not casually differ in gateway ownership, external entry strategy, route model, or internal service exposure.

## 13. Development Deployment Strategy

Development deployment should optimize for fast local feedback while preserving the production access model.

Development Compose may use development-only convenience settings, including source-code bind mounts, hot reload commands, dependency volumes, framework cache volumes, host-aligned users, watcher polling, and safe local defaults.

Development services must still preserve these boundaries:

1. Only the gateway may publish host ports.
2. `web`, `api`, and `db` remain internal.
3. Browser access goes through the gateway.
4. API calls use `/api/...`.
5. Container-to-container communication uses service names.

Detailed examples live in `guides/DOCKER_COMPOSE_DEVELOPMENT_CONVENIENCE_GUIDE.md`.

## 14. Test Deployment Strategy

Test deployments should be deterministic, automation-friendly, and isolated from developer-local state.

Test deployments may use:

1. A dedicated `docker-compose.test.yml` file.
2. Ephemeral or resettable database volumes.
3. Test-specific commands.
4. Test-specific environment variables.
5. `CI=true` or `CI=1` for non-interactive behavior.

Test deployments must not:

1. Depend on direct host ports for internal services.
2. Depend on developer-local absolute paths unless explicitly required.
3. Reuse production secrets.
4. Persist test data into production volumes.
5. Require interactive input.

Test failure should be represented by process exit code.

## 15. Production Deployment Strategy

Production deployment should optimize for stability, repeatability, and small runtime exposure.

Production should:

1. Use pinned runtime images where possible.
2. Avoid source-code bind mounts.
3. Avoid hot reload commands.
4. Use stable restart policies.
5. Publish only the gateway host port.
6. Keep databases and infrastructure services internal.
7. Persist database data through named volumes.
8. Read production secrets from the deployment environment.
9. Enable non-interactive runtime behavior when required.

Recommended startup:

```bash
docker compose -p <project-slug> up -d
```

Production startup should not rely on `--build` unless the deployment process explicitly supports in-place image builds.

## 16. CI Mode Strategy

Production and test containers should avoid interactive behavior.

If the runtime, package manager, framework, or startup command behaves differently in CI or non-interactive mode, set an explicit CI flag.

Recommended default:

```yaml
environment:
  CI: "true"
```

Alternative accepted form:

```yaml
environment:
  CI: "1"
```

A project should choose one representation and use it consistently.

`CI` is a runtime behavior flag, not a secret. Do not add it to every service automatically. Add it to services that require non-interactive runtime or build behavior.

## 17. Runtime Image Strategy

Production and release deployments should use immutable or pinned runtime images.

Examples:

```yaml
web:
  image: <private-registry>/<project-slug>-web:<pinned-version>

api:
  image: <private-registry>/<project-slug>-api:<pinned-version>

nginx:
  image: <private-registry>/nginx:<pinned-version>

db:
  image: <private-registry>/postgres:<pinned-version>
```

Development may use local builds or upstream images when project policy allows.

Do not mix public and private image sources in production unless explicitly documented.

## 18. Network Strategy

Use one internal bridge network for the standard application topology.

Recommended Compose network key:

```text
app_net
```

Recommended actual network name:

```text
<project-slug>-app-net
```

Services should communicate through Compose service names:

```text
gateway → web:3000
gateway → api:8000
api     → db:5432
```

Do not hardcode container IP addresses.

Do not use `localhost` for container-to-container communication.

Do not use `host.docker.internal` unless the project explicitly requires host integration.

## 19. Persistence Strategy

Persistent data must use named volumes or an explicitly approved persistent storage mechanism.

For PostgreSQL, use a named volume.

Recommended volume key:

```text
postgres_data
```

Recommended actual volume names:

```text
<project-slug>-postgres-data-dev
<project-slug>-postgres-data-test
<project-slug>-postgres-data-prod
```

Development, test, and production database volumes must not be reused across environments.

Test volumes should be resettable or disposable.

## 20. Secret and Environment Variable Strategy

Projects should provide a safe `.env.example` documenting required variables.

Rules:

1. Do not commit real production secrets.
2. Use placeholders or safe local defaults in examples.
3. Provide production secrets through the deployment environment.
4. Document only variables actually required by the project.
5. Use Docker service names for container-to-container URLs.
6. Treat `CI` as a runtime behavior flag, not a secret.

Database URLs inside containers should use the database service name, such as `db:5432`.

## 21. Database Migration Strategy

Database schema migrations and seed scripts are deployment actions, not ordinary API startup behavior by default.

Production deployments should prefer explicit one-off migration jobs over implicit API startup migrations.

Development and test environments may provide convenience bootstrap commands.

Startup migration is allowed only when explicitly documented and safe for the project deployment model.

Detailed guidance lives in `guides/DATABASE_MIGRATION_AND_SEED_GUIDE.md`.

## 22. Runtime Reliability Strategy

Deployment services should not rely only on container start order.

Use healthchecks and readiness-aware dependency startup where readiness matters.

Recommended readiness relationships:

```text
api     depends on healthy db
gateway depends on healthy web and api
```

Production services should use a stable restart policy, such as:

```yaml
restart: unless-stopped
```

Where compatible, production services should use runtime hardening such as non-root users, read-only root filesystems, explicit writable paths, minimized capabilities, and disabled privilege escalation.

## 23. Release Deployment Strategy

If a project includes `release/docker-compose.yml`, it should remain behaviorally aligned with production `docker-compose.yml`.

Allowed differences:

1. Image tags.
2. Registry source.
3. Gateway host port.
4. Deployment domain or environment-specific values.

Forbidden differences:

1. Exposing internal services without documentation.
2. Changing service roles without a topology decision.
3. Weakening production hardening without explanation.
4. Reusing development or test database volumes.
5. Replacing the gateway model without documentation.

## 24. AI Coding Agent Constraints

When generating or editing deployment files, AI coding agents must:

1. Preserve the documented deployment service model.
2. Use default names `nginx`, `web`, `api`, and `db` when no project topology exists.
3. Avoid routine service renames.
4. Avoid adding services without project requirements.
5. Publish host ports only through the gateway by default.
6. Keep internal services on Docker networks.
7. Preserve same-origin browser access.
8. Preserve `/api/` routing unless explicitly changed.
9. Avoid hardcoding internal service addresses into browser-side code.
10. Avoid introducing Kubernetes, Helm, or cloud-specific deployment files unless requested.
11. Avoid adding unrelated environment variables.
12. Add `CI=true` or `CI=1` only where non-interactive behavior is required.
13. Keep deployment files readable and minimal.
