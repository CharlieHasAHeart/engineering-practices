# DEPLOYMENT_CONVENTION.md

Convention: Deployment Convention  
Version: 1.0.0  
Status: Active  
Last Updated: 2026-05-02  
Maintainer: Charlie Mi <txqq3116934585@gmail.com>

## 1. Purpose

This document defines deployment conventions for containerized full-stack web applications.

It is intended to guide both human developers and AI coding agents when creating or modifying deployment-related files.

This convention applies to projects that use:

1. Docker Compose
2. Nginx as the single external gateway
3. A frontend web service
4. A backend API service
5. PostgreSQL as the database service

This document is not a general deployment tutorial. It is a reusable engineering convention.

---

## 2. Core Deployment Model

The application should be deployed as a container-first web application using Docker Compose.

The standard service topology is:

```text
Browser
  ↓
nginx
  ├── /       → web:3000
  └── /api/   → api:8000
                  ↓
                 db:5432
```

The standard Compose services are:

```text
nginx
web
api
db
```

Only `nginx` may be accessed from the host machine.

`web`, `api`, and `db` must communicate only through the Docker internal network.

---

## 3. Project Variables

Project-specific values must be represented using placeholders in reusable conventions.

Recommended placeholders:

```text
<project-name>
<project-slug>
<database-name>
<database-user>
<private-registry>
<environment>
```

Example values:

```text
<project-name>      → Example App
<project-slug>      → example-app
<database-name>     → example
<database-user>     → example
<private-registry>  → registry.example.com/org
<environment>       → dev | prod | test
```

Reusable convention documents must not hardcode a specific product name unless the document is intentionally project-specific.

---

## 4. Unified Access Model

### 4.1 Browser Entry

The application should be accessed through a single external origin.

Default local development access:

```text
http://localhost:8080/
```

API requests should use the same origin:

```text
http://localhost:8080/api/...
```

---

### 4.2 Route Separation

Nginx must route requests as follows:

```text
/api/  → api:8000
/      → web:3000
```

The `/api/` prefix must be preserved when forwarding requests to the API service.

Example:

```text
Browser request:
  /api/projects

Nginx forwards to:
  api:8000/api/projects

API route:
  /api/projects
```

Do not rewrite `/api/...` into `/v1/...` unless the API design is explicitly changed.

---

### 4.3 Forbidden Browser Access Patterns

The browser must not directly access:

```text
http://localhost:3000
http://localhost:8000
http://localhost:5432
```

Frontend browser-side code must not hardcode:

```text
http://localhost:8000
http://api:8000
```

Browser-side API requests should use relative paths:

```text
/api/...
```

---

## 5. Fixed Compose Service Names

The following Compose service names are standard and must be used exactly:

```text
nginx
web
api
db
```

Do not rename them to:

```text
frontend
backend
postgres
database
gateway
server
client
```

Reasons:

1. Nginx upstream rules rely on stable service names.
2. Docker internal DNS uses Compose service names.
3. Documentation and automation depend on predictable names.
4. AI coding agents must not invent alternative names.

---

## 6. Container Naming Convention

### 6.1 Required Pattern

Container names should follow this pattern:

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
example-app-web-prod
example-app-api-prod
example-app-db-prod
```

Allowed environment suffixes:

```text
dev
prod
test
```

For a minimal deployment profile, `dev` and `prod` are usually sufficient.

---

### 6.2 Naming Rules

Container names must:

1. Use lowercase letters.
2. Use hyphens.
3. Start with `<project-slug>`.
4. Include the service name.
5. Include the environment suffix.
6. Avoid underscores.
7. Avoid random or generated names when stable names are expected.

---

### 6.3 Compose Project Name

The Compose project name should be:

```text
<project-slug>
```

Recommended command:

```bash
docker compose -p <project-slug> -f docker-compose.dev.yml up --build
```

If a `.env` file is used for Compose, it may include:

```env
COMPOSE_PROJECT_NAME=<project-slug>
```

---

## 7. Network Naming Convention

### 7.1 Required Network

Use one internal bridge network.

Recommended Compose network key:

```text
app_net
```

Recommended actual network name:

```text
<project-slug>-app-net
```

All core services must join this network:

1. `nginx`
2. `web`
3. `api`
4. `db`

---

### 7.2 Network Rules

Services must communicate through Compose service names:

```text
nginx → web:3000
nginx → api:8000
api   → db:5432
```

Do not hardcode container IP addresses.

Do not create multiple networks unless explicitly required.

Do not expose `web`, `api`, or `db` to the host network.

---

## 8. Volume Naming Convention

### 8.1 PostgreSQL Data Volume

PostgreSQL data must be persisted with a named Docker volume.

Recommended Compose volume key:

```text
postgres_data
```

Recommended actual volume names:

```text
<project-slug>-postgres-data-dev
<project-slug>-postgres-data-prod
```

---

### 8.2 Volume Rules

The `db` service must store PostgreSQL data in a named volume.

Do not use anonymous volumes for PostgreSQL data.

Do not store PostgreSQL data only inside the container filesystem.

Do not mount host-specific absolute paths for database storage unless the project explicitly requires it.

---

## 9. File Responsibility Boundaries

### 9.1 `docker-compose.dev.yml`

Purpose:

1. Local development container orchestration.
2. Support frontend and backend hot reload.
3. Allow source-code mounts.
4. Use development commands.
5. Expose only `nginx`.
6. Preserve the same browser access model as production.

Development access should remain:

```text
http://localhost:8080/
```

---

### 9.2 `docker-compose.yml`

Purpose:

1. Production or production-like container orchestration.
2. Use pinned production runtime images.
3. Avoid source-code hot reload.
4. Avoid source directory mounts.
5. Expose only `nginx`.
6. Use stable restart policies.
7. Apply production runtime hardening where compatible.

Production startup should normally use:

```bash
docker compose -p <project-slug> up -d
```

Do not rely on `--build` for production startup unless the deployment process explicitly supports in-place image builds.

---

### 9.3 `release/docker-compose.yml`

Purpose:

1. Release-ready production Compose file.
2. Remain behaviorally aligned with root `docker-compose.yml`.
3. Allow environment-specific host port mapping for `nginx`.
4. Use immutable or pinned runtime image references.
5. Preserve production security and network policies.

---

### 9.4 `apps/web/Dockerfile`

Purpose:

1. Build the frontend service image.
2. Support production build.
3. Support development mode through Compose command override.
4. Avoid hardcoded API hostnames.
5. Avoid database initialization logic.
6. Avoid Nginx gateway logic.
7. Run the runtime stage as a non-root user where possible.

---

### 9.5 `apps/api/Dockerfile`

Purpose:

1. Build the backend API service image.
2. Install backend dependencies.
3. Run the API application.
4. Support reload mode in development through Compose command override.
5. Avoid hardcoded host machine addresses.
6. Avoid uncontrolled database migrations during image build.

---

### 9.6 `infra/nginx/nginx.conf`

Purpose:

1. Provide the single external gateway.
2. Reverse proxy `/api/` to `api`.
3. Reverse proxy `/` to `web`.
4. Preserve same-origin browser access.
5. Pass common proxy headers.
6. Avoid business logic.

---

### 9.7 `.env.example`

Purpose:

1. Document required environment variables.
2. Provide safe local development defaults.
3. Avoid real secrets.
4. Document only variables that are actually required by the project.

---

## 10. Service Specifications

### 10.1 `nginx`

#### Role

`nginx` is the only externally exposed service.

Responsibilities:

1. Accept browser traffic.
2. Route `/api/` to `api:8000`.
3. Route all other paths to `web:3000`.
4. Keep frontend and backend under one origin.

#### Required Port Exposure

Only `nginx` may use `ports`.

Recommended local mapping:

```yaml
ports:
  - "8080:80"
```

---

### 10.2 `web`

#### Role

`web` runs the frontend application.

The `web` service may use any frontend framework, as long as it:

1. Listens on the expected internal port.
2. Is reachable by `nginx` through the Docker internal network.
3. Does not expose host ports directly.
4. Uses same-origin relative paths for browser-side API requests.

Container internal port:

```text
3000
```

The `web` service should use:

```yaml
expose:
  - "3000"
```

It must not use host port mappings such as:

```yaml
ports:
  - "3000:3000"
```

---

### 10.3 `api`

#### Role

`api` runs the backend API application.

The `api` service may use any backend framework, as long as it:

1. Listens on the expected internal port.
2. Is reachable by `nginx` through the Docker internal network.
3. Handles API paths routed by `nginx` without requiring direct browser access.
4. Connects to the database through the Docker service name.

Container internal port:

```text
8000
```

The `api` service should use:

```yaml
expose:
  - "8000"
```

It must not use host port mappings such as:

```yaml
ports:
  - "8000:8000"
```

---

### 10.4 `db`

#### Role

`db` runs PostgreSQL.

Use the official PostgreSQL upstream image or an approved mirrored copy from a private registry.

Development example:

```yaml
image: postgres:16
```

Production or release example:

```yaml
image: <private-registry>/postgres:<pinned-version>
```

Container internal port:

```text
5432
```

The `db` service may use:

```yaml
expose:
  - "5432"
```

It must not use host port mappings such as:

```yaml
ports:
  - "5432:5432"
```

Database data must be stored in a named volume.

---

## 11. Port Exposure Rules

Only `nginx` may expose host ports.

The following services must not expose host ports:

```text
web
api
db
```

This rule applies to both development and production Compose files.

Development is not an exception.

Forbidden:

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

Use `expose` for internal container visibility when needed.

---

## 12. Nginx Configuration Rules

### 12.1 Required Upstreams

Recommended upstream names:

```nginx
upstream web_upstream {
    server web:3000;
}

upstream api_upstream {
    server api:8000;
}
```

---

### 12.2 Required Location Rules

Nginx must explicitly separate API and web traffic:

```nginx
location /api/ {
    proxy_pass http://api_upstream;
}

location / {
    proxy_pass http://web_upstream;
}
```

The `/api/` prefix must be preserved.

---

### 12.3 Required Proxy Headers

Nginx should pass common proxy headers:

```nginx
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

---

### 12.4 Nginx Must Not

Nginx must not:

1. Implement business rules.
2. Connect to PostgreSQL.
3. Replace the frontend service.
4. Serve backend responses from static files.
5. Merge `web` and `api` into one container.
6. Introduce complex caching rules without explicit project requirements.
7. Introduce path rewriting that hides `/api/`.

---

## 13. Web Service Rules

Frontend browser-side API calls must use relative paths:

```text
/api/...
```

Do not hardcode:

```text
http://localhost:8000
http://api:8000
```

Production should run the built frontend application.

Development may run a dev server, such as:

```bash
pnpm dev
```

The development server must listen on:

```text
0.0.0.0:3000
```

so that Nginx can reach it inside the Docker network.

---

## 14. API Service Rules

The `api` service runs the backend API application.

The API service must:

1. Listen on `0.0.0.0:8000` inside the container.
2. Be reachable by `nginx` through the Docker internal network.
3. Preserve the API path prefix expected by the Nginx routing rules.
4. Connect to dependent services through Docker service names, not host addresses.

Development Compose may override the startup command to enable framework-specific reload mode.

---

## 15. Database Rules

### 15.1 PostgreSQL Image

Use the official PostgreSQL upstream image or an approved mirrored copy.

Do not build a custom database image unless explicitly required.

---

### 15.2 Required Environment Variables

The `db` service should use:

```env
POSTGRES_DB=<database-name>
POSTGRES_USER=<database-user>
POSTGRES_PASSWORD=<safe-development-password>
```

These may be overridden by `.env`.

---

### 15.3 API Database Connection

The API must connect to PostgreSQL using the Docker service name:

```text
db:5432
```

Recommended format:

```env
DATABASE_URL=postgresql+psycopg://<database-user>:<database-password>@db:5432/<database-name>
```

Do not use:

```text
localhost:5432
```

inside the API container.

---

### 15.4 Persistence

Use a named volume:

```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data
```

The actual volume name should follow this convention:

```text
<project-slug>-postgres-data-<environment>
```

---

### 15.5 External Access

PostgreSQL must not expose a host port.

Developers should interact with the database through Compose:

```bash
docker compose -p <project-slug> -f docker-compose.dev.yml exec db psql -U <database-user> -d <database-name>
```

or through the API container.

---

## 16. Environment Variables

### 16.1 Required `.env.example`

Each project should provide a safe `.env.example` file that documents only the variables required to run the standard Compose deployment.

Recommended minimal structure:

```env
COMPOSE_PROJECT_NAME=<project-slug>

POSTGRES_DB=<database-name>
POSTGRES_USER=<database-user>
POSTGRES_PASSWORD=<database-password>
```

If the API uses a database connection string, it should use the Docker service name `db`:

```env
DATABASE_URL=postgresql+psycopg://<database-user>:<database-password>@db:5432/<database-name>
```

Project-specific runtime variables may be added only when they are actually used by the project.

Rules:

1. Keep `.env.example` minimal.
2. Use placeholders instead of real secrets.
3. Add application-specific variables only when they are actually used.
4. Do not use `localhost` for container-to-container database access.
5. Do not require generic projects to use a specific environment variable name such as `APP_ENV` or `API_PREFIX`.

---

### 16.2 Secret Handling

Do not commit real production secrets.

Example files may use safe local development defaults.

Production secrets must be provided by the deployment environment.

---

## 17. Compose File Structure Rules

### 17.1 Top-Level Order

Compose files should use this order:

```yaml
services:
volumes:
networks:
```

---

### 17.2 Recommended Service Order

Inside `services`, use:

```yaml
nginx:
web:
api:
db:
```

This order reflects traffic flow from external gateway to internal dependencies.

---

### 17.3 Service Field Order

Within each service, use this field order when practical:

```yaml
image or build
container_name
user
depends_on
environment
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
```

---

---
## 18. Compose Requirements

This section defines how Compose files should be structured across development, production, and release environments.

### 18.1 Development Compose

`docker-compose.dev.yml` must:

1. Include the standard services: `nginx`, `web`, `api`, and `db`.
2. Use fixed service names.
3. Use container names with the `-dev` suffix.
4. Expose only `nginx` to the host.
5. Keep `web`, `api`, and `db` on the Docker internal network.
6. Mount source code only when hot reload or local development requires it.
7. Enable framework-specific development or reload commands through Compose overrides.
8. Use PostgreSQL with a named volume.
9. Preserve the same Nginx routing model used by production.
10. Avoid host permission drift for bind-mounted source directories.

Services that write to bind-mounted source code should run as a host-aligned non-root user:

```yaml
user: "${UID:-1000}:${GID:-1000}"
```

Do not run package manager or build commands as `root` if they write to bind-mounted project source paths.

For frontend development containers, prefer volume-backed artifact paths such as:

```text
node_modules
.next
dist
```

when they reduce host ownership conflicts.

---

### 18.2 Production Compose

`docker-compose.yml` must:

1. Include the standard services: `nginx`, `web`, `api`, and `db`.
2. Use fixed service names.
3. Use container names with the `-prod` suffix.
4. Expose only `nginx` to the host.
5. Use production runtime images for `web` and `api`.
6. Avoid source-code mounts.
7. Use PostgreSQL with a named volume.
8. Use a single internal network.
9. Use a stable restart policy, such as `unless-stopped`.
10. Apply runtime hardening where compatible.

Production startup should normally use:

```bash
docker compose -p <project-slug> up -d
```

Do not rely on `--build` for production startup unless the deployment process explicitly supports in-place image builds.

---

### 18.3 Release Compose

If the project includes `release/docker-compose.yml`, it must remain behaviorally aligned with root `docker-compose.yml`.

Required alignment:

1. Service topology:
   - services must include `nginx`, `web`, `api`, and `db`
   - standard service names must remain identical
2. Runtime network model:
   - one internal network for service-to-service communication
   - `web`, `api`, and `db` must not publish host ports
3. Runtime security baseline:
   - non-root runtime users where compatible
   - read-only filesystem where compatible
   - explicit writable runtime paths
   - minimized Linux capabilities
   - `no-new-privileges` where compatible
4. Persistent storage:
   - named PostgreSQL data volume
5. Stability controls:
   - restart policy semantics aligned with production

Allowed differences:

1. Host port mapping for `nginx` may differ by deployment environment.
2. Image tags may differ by release version.
3. Image source may differ if explicitly documented.

Forbidden differences:

1. Weakening production security hardening without explanation.
2. Exposing `web`, `api`, or `db` to host ports.
3. Changing the standard service topology without a major convention update.

---

### 18.4 Additional Services

The standard deployment topology includes:

```text
nginx
web
api
db
```

Projects may add additional services when required by product, architecture, or project-specific deployment documents.

Additional services must:

1. Use stable Compose service names.
2. Join the appropriate Docker internal network.
3. Avoid host port exposure unless explicitly required.
4. Use named volumes for persistent data when needed.
5. Define healthchecks when they affect runtime readiness.
6. Document required environment variables in `.env.example`.
7. Be documented in project-specific deployment or architecture documentation.

---

## 19. Runtime Reliability and Security

### 19.1 Production Security Baseline

For production and production-like deployment, apply the following baseline where compatible:

1. `web` and `api` must not run as `root`.
2. Root filesystems should be read-only for `nginx`, `web`, and `api` where application behavior allows.
3. Writable runtime paths must be explicit and minimal.
4. Prefer `tmpfs` for temporary writable paths.
5. Linux capabilities should be minimized.
6. Privilege escalation must be disabled.
7. Bind-mounted configuration files should be read-only by default.
8. Database persistent storage remains writable only through the dedicated database volume.

Recommended service hardening example:

```yaml
web:
  user: "1000:1000"
  read_only: true
  tmpfs:
    - /tmp
  cap_drop:
    - ALL
  security_opt:
    - no-new-privileges:true

api:
  user: "1000:1000"
  read_only: true
  tmpfs:
    - /tmp
  cap_drop:
    - ALL
  security_opt:
    - no-new-privileges:true
```

---

### 19.2 Runtime Image Policy

Production and release deployments should use pinned runtime images.

If a private registry is required, production and release Compose files must use private registry images instead of public registry addresses.

Example:

```yaml
nginx:
  image: <private-registry>/nginx:<pinned-version>

db:
  image: <private-registry>/postgres:<pinned-version>
```

Development Compose may use official upstream images or approved mirrored images, depending on project policy.

Do not mix public and private image sources in production unless explicitly documented.

---

### 19.3 Healthcheck and Readiness Policy

Compose files should define minimal runtime healthchecks and readiness-based dependency startup.

Required rules:

1. `db` must define a healthcheck using `pg_isready`.
2. `api` must define a healthcheck that verifies local service readiness.
3. `web` must define a healthcheck that verifies local service readiness.
4. `nginx` should define a lightweight healthcheck.
5. Dependency startup should use `depends_on` with `condition: service_healthy` for critical upstream services.

Recommended dependency relationships:

```text
api   depends on healthy db
nginx depends on healthy web and api
```

`web` should depend on `api` only if the web container startup requires API availability.

Do not rely on container start order alone for application readiness.

---

### 19.4 Deployment Readiness Check

Before release or deployment, verify that deployment files are valid and aligned with this convention.

At minimum, check:

1. Compose files render successfully.
2. Required services are present.
3. Only `nginx` exposes host ports.
4. `web`, `api`, and `db` remain internal.
5. Nginx routing preserves the `/api/` path.
6. Required environment variables are documented.
7. Healthchecks and readiness dependencies are present where required.
8. Release files remain aligned with production files, if applicable.

---

## 20. Dockerfile Rules

### 20.1 Web Dockerfile

The web Dockerfile should:

1. Be located at `apps/web/Dockerfile`.
2. Target the frontend framework used by the project.
3. Support production builds.
4. Allow development command override from Compose.
5. Avoid hardcoded API hostnames.
6. Avoid Nginx gateway logic.
7. Avoid database logic.
8. Run the runtime stage as a non-root user where possible.
9. Keep the runtime image minimal.

The web Dockerfile must not:

1. Install PostgreSQL.
2. Run database migrations.
3. Include Nginx.
4. Hardcode `http://localhost:8000`.
5. Hardcode `http://api:8000` for browser-side calls.
6. Add unrelated application services.

---

### 20.2 API Dockerfile

The API Dockerfile should:

1. Be located at `apps/api/Dockerfile`.
2. Target the backend framework used by the project.
3. Install backend dependencies.
4. Run the API server.
5. Allow reload mode through Compose in development.
6. Avoid Nginx logic.
7. Avoid frontend build logic.

The API Dockerfile must not:

1. Include Nginx.
2. Include frontend build logic.
3. Hardcode host machine IPs.
4. Hardcode `localhost:5432` as the container database host.
5. Run uncontrolled destructive migration commands.
6. Add unrelated hardware or protocol clients.

---

## 21. Database Migration Convention

If database migrations are used, they should be explicit.

Migration tools are project-specific and should be selected by the backend implementation.

Development example:

```bash
docker compose -p <project-slug> -f docker-compose.dev.yml exec api <migration-command>
```

Production-like example:

```bash
docker compose -p <project-slug> exec api <migration-command>
```

Do not make every API container startup automatically run migrations unless explicitly approved.

Avoid entrypoints that always combine migrations with API startup, such as:

```bash
<migration-command> && <api-start-command>
```

Reasons:

1. It hides schema changes.
2. It makes startup side effects harder to control.
3. It can be unsafe for production-like deployments.

---

## 22. Standard Workflows

### 22.1 Development Startup

Recommended development startup:

```bash
docker compose -p <project-slug> -f docker-compose.dev.yml up --build
```

Then open:

```text
http://localhost:8080/
```

---

### 22.2 Database Shell

Developers should access PostgreSQL through Compose:

```bash
docker compose -p <project-slug> -f docker-compose.dev.yml exec db psql -U <database-user> -d <database-name>
```

Do not connect through `localhost:5432`, because the database must not expose a host port.

---

### 22.3 API Tests

Recommended API test command:

```bash
docker compose -p <project-slug> -f docker-compose.dev.yml exec api <test-command>
```

---

### 22.4 Production-Like Startup

Recommended production-like startup:

```bash
docker compose -p <project-slug> up -d
```

Then open:

```text
http://localhost:8080/
```

or the configured deployment domain.

---

## 23. Examples

### 23.1 Example Development Compose Skeleton

This is a development-oriented structural example.

Production and release Compose files must additionally follow production image, hardening, and release alignment rules.

```yaml
services:
  nginx:
    image: nginx:stable
    container_name: <project-slug>-nginx-dev
    depends_on:
      web:
        condition: service_healthy
      api:
        condition: service_healthy
    volumes:
      - ./infra/nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
    ports:
      - "8080:80"
    networks:
      - app_net
    healthcheck:
      test: ["CMD", "nginx", "-t"]
      interval: 30s
      timeout: 5s
      retries: 3

  web:
    build:
      context: ./apps/web
      dockerfile: Dockerfile
    container_name: <project-slug>-web-dev
    user: "${UID:-1000}:${GID:-1000}"
    expose:
      - "3000"
    networks:
      - app_net
    healthcheck:
      test: ["CMD", "sh", "-c", "wget -qO- http://localhost:3000 >/dev/null 2>&1 || exit 1"]
      interval: 30s
      timeout: 5s
      retries: 3

  api:
    build:
      context: ./apps/api
      dockerfile: Dockerfile
    container_name: <project-slug>-api-dev
    user: "${UID:-1000}:${GID:-1000}"
    environment:
      DATABASE_URL: postgresql+psycopg://<database-user>:<database-password>@db:5432/<database-name>
    expose:
      - "8000"
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app_net
    healthcheck:
      test: ["CMD", "sh", "-c", "wget -qO- http://localhost:8000/health >/dev/null 2>&1 || exit 1"]
      interval: 30s
      timeout: 5s
      retries: 3

  db:
    image: postgres:16
    container_name: <project-slug>-db-dev
    environment:
      POSTGRES_DB: <database-name>
      POSTGRES_USER: <database-user>
      POSTGRES_PASSWORD: <database-password>
    volumes:
      - postgres_data:/var/lib/postgresql/data
    expose:
      - "5432"
    networks:
      - app_net
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U <database-user> -d <database-name>"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
    name: <project-slug>-postgres-data-dev

networks:
  app_net:
    name: <project-slug>-app-net
    driver: bridge
```

---

### 23.2 Example Nginx Skeleton

```nginx
upstream web_upstream {
    server web:3000;
}

upstream api_upstream {
    server api:8000;
}

server {
    listen 80;

    location /api/ {
        proxy_pass http://api_upstream;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location / {
        proxy_pass http://web_upstream;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## 24. AI Coding Agent Constraints

When generating or editing deployment files, AI coding agents must follow these rules:

1. Do not rename standard services unless explicitly requested.
2. Use `nginx`, `web`, `api`, and `db` for the standard topology.
3. Do not expose `web`, `api`, or `db` host ports.
4. Only `nginx` may use `ports`.
5. Preserve `/api/` routing through Nginx.
6. Preserve the same browser access model in development and production.
7. Do not hardcode browser API calls to `localhost:8000`.
8. Do not hardcode container IPs.
9. Do not merge `nginx` with `web` or `api`.
10. Do not replace PostgreSQL with another database unless explicitly requested.
11. Do not customize the PostgreSQL image unless explicitly required.
12. Do not add additional runtime services unless explicitly required by the project.
13. Do not add Kubernetes or Helm unless explicitly requested.
14. Do not add unrelated environment variables.
15. Do not add hardware or industrial protocol services unless explicitly requested.
16. Do not hide migrations as uncontrolled startup side effects.
17. Keep deployment files readable and minimal.
18. Use project placeholders in reusable convention documents.

---

## 25. Final Summary

This deployment convention defines a standard Docker Compose topology for containerized full-stack web applications:

```text
nginx
web
api
db
```

Nginx is the only externally exposed gateway.

Routing must be:

```text
/api/  → api:8000
/      → web:3000
```

PostgreSQL must remain internal to Docker networking, persist data in a named volume, and avoid host port exposure.

Development and production should share the same browser access model.

The deployment model should remain readable, container-first, and suitable for both human developers and AI coding agents.
