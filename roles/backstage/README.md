# Backstage

## Overview

[Backstage](https://backstage.io) is a developer portal (internal developer platform) that centralizes a service catalog, technical documentation (TechDocs), and scaffolding templates. This role deploys the Mathod.io Backstage instance: a stateless application container, image built and published by GitHub Actions from [Mathod95/backstage](https://github.com/Mathod95/backstage), with Postgres as the persistence backend (catalog, scaffolding tasks, TechDocs metadata).

Backstage handles its own authentication (GitHub OAuth): this role does not put an Authelia layer in front of it. No homemade Postgres container either: this role reuses Saltbox's **native** `postgres` role, which already handles persistence, multiple instances, and major-version migrations correctly. Backstage becomes an additional named instance of that role rather than its own bundled database.

## Requirements

- This repo's role copied and registered per the [repo root README](../../README.md).
- A Postgres instance named `backstage` provisioned through Saltbox's native `postgres` role (see Configuration below).

## Deployment

Deploy the Postgres instance first, then the app:

```shell
sb install postgres
sb install mod-backstage
```

Or directly:

```shell
sudo ansible-playbook saltbox_mod.yml --tags backstage
```

Re-running `sb install mod-backstage` at any time recreates the container with the latest config/image (pull + remove + recreate), without touching the Postgres data. To change versions, override `backstage_docker_image_tag` in the Inventory rather than editing this repo, or pin an exact tag/digest for a reproducible production deployment.

## Usage

Visit `https://backstage.YOUR_DOMAIN`.

## Configuration

All variables follow the standard Saltbox `backstage_*` convention (see `defaults/main.yml`), overridable without touching the role through the host's Inventory (`/srv/git/saltbox/inventories/host_vars/localhost.yml`):

| Variable | Purpose | Where to set it |
|---|---|---|
| `backstage_web_subdomain` | subdomain (default: `backstage`) | Inventory, if needed |
| `backstage_docker_image_repo` / `_tag` | GHCR image to deploy | Inventory, to pin an exact tag/digest in production |
| `backstage_docker_env_user` / `_password` / `_db` | Postgres credentials, shared with the matching `postgres` instance | Inventory, required, never committed |
| `backstage_docker_envs_custom` | application secrets (`backend.auth.keys`, GitHub OAuth client...) | Inventory, required, never committed |

Full Inventory example — **never put these values in this repo**:

```yaml
# Add a Postgres instance dedicated to Backstage (in addition to the default "postgres" instance)
postgres_instances: ["postgres", "backstage"]

# Credentials for that Postgres instance — used both by the native postgres role
# (to create the instance) and by the backstage role (to connect to it), single source of truth
backstage_docker_env_user: "backstage"
backstage_docker_env_password: "<generated password, never committed>"
backstage_docker_env_db: "backstage"

# Secrets specific to the Backstage app itself (backend.auth.keys, GitHub OAuth client, etc.)
# — exact names to be set once the real app-config.yaml is written in the backstage repo
backstage_docker_envs_custom:
  BACKEND_SECRET: "<to be defined>"
```

## Data persistence

Yes, Postgres data survives any redeploy/update of both the `backstage` role **and** the `postgres` role. Saltbox's native `postgres` role:

- mounts data on a persistent host path (`{{ server_appdata_path }}/backstage`), not an anonymous Docker volume;
- only ever `remove`s and `recreate`s the container on each run, never touching data on disk;
- even handles Postgres major-version migrations safely (automatically backing up the previous data directory before switching over).

The only real risk: manually deleting the data directory on the host, or changing `postgres_instances`/the instance name without a deliberate migration.

## Still to confirm once the `backstage` repo (image/app-config) is actually written

- The exact port exposed by the Backstage backend in production (`backstage_web_port`, currently defaulting to `7007`, to be confirmed against the real `app-config.yaml`).
- The exact GHCR repo name (`backstage_docker_image_repo`, currently `ghcr.io/mathod95/backstage`).
- The exact list of application secrets to pass via `backstage_docker_envs_custom` (backend.auth.keys, GitHub OAuth client id/secret...).
