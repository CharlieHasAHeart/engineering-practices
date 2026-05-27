# Docker Compose Development Convenience Guide

Guide: Docker Compose Development Convenience Guide  
Version: 1.1.0  
Status: Active  
Last Updated: 2026-05-27  
Maintainer: Charlie Mi <txqq3116934585@gmail.com>

## 1. Purpose

This guide documents development-only Docker Compose patterns that improve local feedback speed while preserving deployment boundaries.

These patterns must not be copied into production Compose files unless explicitly justified.

## 2. Development Convenience Principle

Development Compose may optimize for fast local feedback.

Common convenience settings include:

1. Source-code bind mounts.
2. Hot reload commands.
3. Dependency named volumes.
4. Framework cache named volumes.
5. Package-manager cache volumes.
6. Host-aligned users.
7. Polling-based file watchers.
8. Safe local environment defaults.

Development convenience must not break the gateway-only external entry model.

## 3. Source-Code Bind Mounts

Hot reload normally requires source-code bind mounts.

Example:

```yaml
services:
  web:
    volumes:
      - ./apps/web:/app
```

Without the bind mount, changes on the host are not automatically visible inside the container and the image usually needs to be rebuilt.

Bind mounts are development-only and should not appear in production Compose files.

## 4. Ownership Risk from Bind Mounts

Bind mounts expose host files to container processes.

If a container process runs as `root` and writes to a bind-mounted source path, the host may receive root-owned files such as:

```text
.next
dist
coverage
node_modules
generated
```

This can cause `permission denied` errors on the host.

To reduce ownership drift, use one or more of these patterns:

1. Run the service as a host-aligned user.
2. Store dependency and cache directories in named volumes.
3. Keep generated files outside bind-mounted source paths.

## 5. Host-Aligned User Pattern

Recommended development pattern:

```yaml
services:
  web:
    user: "${UID:-1000}:${GID:-1000}"
    environment:
      HOME: /tmp
```

This makes container writes use the host developer's numeric UID/GID when writing bind-mounted files.

Use explicit values when needed:

```bash
UID=$(id -u) GID=$(id -g) docker compose -f docker-compose.dev.yml up
```

Manual commands should also use the host-aligned user when writing bind-mounted paths:

```bash
docker compose exec --user "$(id -u):$(id -g)" web pnpm install
```

This pattern does not solve every permission issue. Some images expect a real user entry or writable home directory. Use `HOME=/tmp` or explicit cache directories when needed.

## 6. Dependency Named Volume Pattern

Frontend dependencies can be stored in a named Docker volume.

Example:

```yaml
services:
  web:
    volumes:
      - ./apps/web:/app
      - web_node_modules:/app/node_modules

volumes:
  web_node_modules:
    name: <project-slug>-web-node-modules-dev
```

This avoids the common issue where the host bind mount hides image-built `node_modules` or where host and container dependencies differ by platform.

This is development-only. Production images should contain their runtime dependencies at image build time.

## 7. Stale Dependency Volume Risk

Dependency volumes can become stale when dependency manifests change.

Examples:

```text
package.json
pnpm-lock.yaml
package-lock.json
yarn.lock
```

When dependency manifests change, developers may need to recreate development volumes:

```bash
docker compose -f docker-compose.dev.yml down -v
docker compose -f docker-compose.dev.yml up --build
```

Projects should document a safe reset command.

## 8. Framework Cache Volumes

Framework cache volumes can improve repeated startup and rebuild speed.

Common examples:

```text
.next
.nuxt
.vite
.turbo
node_modules/.cache
```

Use only the caches needed by the project.

Do not copy framework cache volumes into production Compose files.

## 9. Next.js Development Example

```yaml
services:
  web:
    build:
      context: ./apps/web
      dockerfile: Dockerfile
    container_name: <project-slug>-web-dev
    user: "${UID:-1000}:${GID:-1000}"
    working_dir: /app
    command: pnpm dev --hostname 0.0.0.0 --port 3000
    environment:
      HOME: /tmp
      WATCHPACK_POLLING: "true"
      CHOKIDAR_USEPOLLING: "true"
    volumes:
      - ./apps/web:/app
      - web_node_modules:/app/node_modules
      - web_next_cache:/app/.next
    expose:
      - "3000"
    networks:
      - app_net

volumes:
  web_node_modules:
    name: <project-slug>-web-node-modules-dev
  web_next_cache:
    name: <project-slug>-web-next-cache-dev
```

Use polling only when file changes are not detected reliably.

## 10. Vite Development Example

```yaml
services:
  web:
    build:
      context: ./apps/web
      dockerfile: Dockerfile
    container_name: <project-slug>-web-dev
    user: "${UID:-1000}:${GID:-1000}"
    working_dir: /app
    command: pnpm dev --host 0.0.0.0 --port 3000
    environment:
      HOME: /tmp
      CHOKIDAR_USEPOLLING: "true"
    volumes:
      - ./apps/web:/app
      - web_node_modules:/app/node_modules
      - web_vite_cache:/app/node_modules/.vite
    expose:
      - "3000"
    networks:
      - app_net

volumes:
  web_node_modules:
    name: <project-slug>-web-node-modules-dev
  web_vite_cache:
    name: <project-slug>-web-vite-cache-dev
```

## 11. pnpm Store Volume Example

```yaml
services:
  web:
    environment:
      PNPM_STORE_DIR: /pnpm/store
    volumes:
      - ./apps/web:/app
      - web_node_modules:/app/node_modules
      - web_pnpm_store:/pnpm/store

volumes:
  web_node_modules:
    name: <project-slug>-web-node-modules-dev
  web_pnpm_store:
    name: <project-slug>-web-pnpm-store-dev
```

This can reduce repeated package download and install time.

## 12. Polling Watcher Settings

Some host/container file sharing systems do not reliably emit filesystem events.

Development Compose may enable polling:

```yaml
environment:
  CHOKIDAR_USEPOLLING: "true"
  WATCHPACK_POLLING: "true"
```

Polling can increase CPU usage. Enable it only when needed.

## 13. API Development Reload Pattern

Backend services may use reload commands in development.

Example:

```yaml
services:
  api:
    build:
      context: ./apps/api
      dockerfile: Dockerfile
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
    volumes:
      - ./apps/api:/app
    expose:
      - "8000"
```

The service must listen on `0.0.0.0`, not `localhost`, so other containers can reach it.

## 14. Development Database Volume

Development may persist local database state:

```yaml
volumes:
  postgres_data:
    name: <project-slug>-postgres-data-dev
```

Projects should document reset instructions:

```bash
docker compose -f docker-compose.dev.yml down -v
```

Development database volumes must not be reused for test or production.

## 15. Production Exclusions

Do not copy these development-only settings into production Compose files:

1. Source-code bind mounts.
2. `node_modules` runtime volumes.
3. Framework cache runtime volumes.
4. Hot reload commands.
5. Polling watcher settings.
6. Unsafe local defaults.
7. Direct host ports for internal services.
