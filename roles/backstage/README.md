# Backstage

## Overview

[Backstage](https://backstage.io) is a developer portal (internal developer platform) that centralizes a service catalog, technical documentation (TechDocs), and scaffolding templates. This role deploys the Mathod.io Backstage instance: a stateless application container, image built and published by GitHub Actions from [Mathod95/backstage](https://github.com/Mathod95/backstage), with Postgres as the persistence backend (catalog, scaffolding tasks, TechDocs metadata).

Backstage handles its own authentication (GitHub OAuth): this role does not put an Authelia layer in front of it.

## Deployment

Requirements: role registered in `saltbox_mod.yml` (see the [repo README](../../README.md) for the full installation), and a Postgres instance named `backstage` provisioned through Saltbox's native `postgres` role.

```shell
sb install mod-backstage
```

Or directly:

```shell
sudo ansible-playbook saltbox_mod.yml --tags backstage
```

Re-running this command at any time recreates the container with the latest config/image (pull + remove + recreate), without touching the Postgres data.

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

Full details (requirements, install order, persistence guarantees): see the [repo root README](../../README.md).
