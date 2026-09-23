# saltbox

Custom Saltbox roles (via [saltbox_mod](https://github.com/saltyorg/saltbox_mod)) for apps not covered by the main Saltbox catalog.

## Requirements on the Saltbox host

- A working Saltbox installation (domain + DNS already configured), up to date: run `sb update` first. The roles use the `role_web` lookup plugin, added to Saltbox on 2026-08-24, and fail with `The lookup plugin 'role_web' was not found` on older installations.
- `saltbox_mod` installed: `sb install saltbox-mod` (or `git clone https://github.com/saltyorg/saltbox_mod.git /opt/saltbox_mod`).

## Installation (generic steps, any role)

1. Get this repo on the host and copy the role into the host's `saltbox_mod` installation:

   ```bash
   git clone https://github.com/Mathod95/saltbox.git ~/saltbox-roles
   sudo cp -r ~/saltbox-roles/roles/<role> /opt/saltbox_mod/roles/<role>
   ```

2. Register the role in `/opt/saltbox_mod/saltbox_mod.yml`, under `roles:`:

   ```yaml
       - { role: <role>, tags: ['<role>'] }
   ```

3. Configure the host's Inventory (`/srv/git/saltbox/inventories/host_vars/localhost.yml`) as documented in that role's own README — **never commit secrets to this repo**.

4. Deploy:

   ```bash
   sb install mod-<role>
   ```

If `sb install mod-<role>` is not available on your version, run the playbook directly from `/opt/saltbox_mod`:

```bash
cd /opt/saltbox_mod && sudo ansible-playbook saltbox_mod.yml --tags <role>
```

## Updating a role

```bash
git -C ~/saltbox-roles pull
sudo cp -r ~/saltbox-roles/roles/<role> /opt/saltbox_mod/roles/
sb install mod-<role>
```

## Roles

| Role | Description |
|---|---|
| [`backstage`](roles/backstage/) | Backstage developer portal, backed by a dedicated Postgres instance. |
| [`tracearr`](roles/tracearr/) | Tracearr, copied unmodified from [saltyorg/Sandbox](https://github.com/saltyorg/Sandbox/tree/master/roles/tracearr) (author: connorgallopo, GPL-3.0, upstream commit `65efcf4c1e`). Register it with `- { role: tracearr, tags: ['tracearr', 'tracearr-claim'] }`. It uses Saltbox's native `redis` and `timescaledb` roles. |

Each role's own README (`roles/<role>/README.md`) covers what it deploys, its specific requirements, configuration variables, and persistence guarantees.
