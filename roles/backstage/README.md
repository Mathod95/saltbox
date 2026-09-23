# Backstage

## Overview

[Backstage](https://backstage.io) is a developer portal (internal developer platform) that centralizes a service catalog, technical documentation (TechDocs), and scaffolding templates. This role deploys a stateless Backstage container built and published by GitHub Actions from [Mathod95/backstage](https://github.com/Mathod95/backstage) (`ghcr.io/mathod95/backstage`), backed by a dedicated Postgres instance.

The Postgres instance is deployed by the role itself through Saltbox's native `postgres` role (named `backstage-postgres`), following the same pattern as the mainline `authentik` role. Its password is generated once and persisted with `saltbox_facts`, so nothing has to be written in the Inventory.

The stock Backstage image only ships the `guest` auth provider, which provides no real security. The role therefore puts Authelia in front of Backstage by default (`backstage_role_traefik_sso_middleware`). Once a real auth provider is configured in the image, set that variable to `""` in the Inventory to remove the Authelia layer.

## Requirements

- A working Saltbox installation with `saltbox_mod` and this role registered (see the [repo root README](../../README.md)).
- Authelia installed on the host, unless you set `backstage_role_traefik_sso_middleware: ""`.

## Deployment

```shell
sb install mod-backstage
```

Or directly:

```shell
sudo ansible-playbook saltbox_mod.yml --tags backstage
```

Re-running the command recreates the Backstage container with the current image (pull + remove + recreate). The Postgres data is never touched.

## Usage

Visit `https://backstage.YOUR_DOMAIN`.

The public URL is injected automatically through `APP_CONFIG_app_baseUrl` and `APP_CONFIG_backend_baseUrl`, which override the `localhost` values of the image's `app-config.production.yaml` (`APP_CONFIG_*` variables have the highest configuration precedence). Changing the domain does not require rebuilding the image.

## Configuration

Variables follow the standard Saltbox convention (`backstage_role_*`, see `defaults/main.yml`). Override them in the Inventory (`/srv/git/saltbox/inventories/host_vars/localhost.yml`). Common ones:

| Variable | Purpose | Default |
|---|---|---|
| `backstage_role_web_subdomain` | subdomain | `backstage` |
| `backstage_role_docker_image_repo` / `_tag` | image to deploy | `ghcr.io/mathod95/backstage` / `latest` |
| `backstage_role_traefik_sso_middleware` | Authelia protection | Saltbox default SSO middleware |
| `backstage_role_postgres_deploy` | deploy the Postgres instance | `true` |
| `backstage_role_postgres_docker_image_tag` | Postgres image tag | `17-alpine` |
| `backstage_role_docker_envs_custom` | extra environment variables for the app | `{}` |

Pin `backstage_role_docker_image_tag` to a commit sha (the pipeline tags every image with its sha) for reproducible deployments.

## Data persistence

Postgres data lives on a persistent host path (`<appdata>/backstage/postgres`) managed by Saltbox's native `postgres` role. Redeploying either role only removes and recreates containers, and major-version upgrades are migrated safely with an automatic backup of the previous data directory. The only real risks are manually deleting that directory or renaming the instance.
