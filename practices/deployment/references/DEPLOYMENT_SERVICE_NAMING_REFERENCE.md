# Deployment Service Naming Reference

Reference: Deployment Service Naming Reference  
Version: 1.1.0  
Status: Active  
Last Updated: 2026-05-27  
Maintainer: Charlie Mi <txqq3116934585@gmail.com>

## Purpose

This reference defines default and extended deployment service naming patterns.

## Default Minimal Service Names

| Role | Default Service Name | Notes |
|---|---|---|
| Gateway | `nginx` | Default external HTTP gateway. |
| Web Application | `web` | Default browser-facing frontend service. |
| API Application | `api` | Default backend HTTP API service. |
| Primary Database | `db` | Default PostgreSQL service. |

Use these names when the project has no documented custom topology.

## Extended Service Names

Use role-revealing names when a project has multiple services in the same category.

Examples:

| Service Name | Responsibility |
|---|---|
| `admin-web` | Admin frontend. |
| `public-web` | Public frontend. |
| `billing-api` | Billing API service. |
| `notification-worker` | Background notification worker. |
| `scheduler` | Scheduled jobs. |
| `redis` | Redis cache or queue backend. |
| `queue` | Message queue. |
| `search` | Search service. |
| `object-storage` | Object storage service. |

## Naming Rules

Service names should:

1. Be lowercase.
2. Use hyphens for multi-word names.
3. Reveal runtime role.
4. Remain stable once adopted.
5. Avoid implementation ambiguity.

Avoid names that are too generic when multiple services exist:

```text
service
server
app
backend2
new-api
```

## Rename Policy

Routine edits should not rename services.

Renames are allowed only for documented topology changes, role changes, implementation changes, or project naming convention adoption.
