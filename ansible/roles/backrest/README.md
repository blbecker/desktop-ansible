# Backrest Role

Installs and configures Backrest (garethgeorge/backrest) for workstation backups.

## Purpose

This role installs Backrest as a client for multihost sync with a central Backrest instance in a k8s cluster. Workstations are *clients* that:

- Run backup-only plans (prune/check/compact live on the cluster instance)
- Receive repo config (B2 location, repo password) via shared repos from the central server
- Store backup config locally (protected by FDE)

## What This Role Does

1. Downloads the backrest binary from GitHub releases
2. Installs it to `/usr/local/bin/backrest`
3. Creates minimal config at `~/.config/backrest/config.json` with instance ID
4. Sets up systemd service running as the desktop user
5. Binds UI to `127.0.0.1:9898` (localhost only)

## What This Role Does NOT Do

- Commit secrets or repo credentials to git (these come via shared repos)
- Create backup plans declaratively (pairing and plan creation is manual via UI)
- Manage vault files or example passwords

## Role Variables

| Variable | Default | Description |
| --- | --- | --- |
| `backrest_version` | `"1.14.1"` | GitHub release tag to install |
| `backrest_tarball_sha256` | pinned v1.14.1 x86_64 sha256 | Checksum for `backrest_Linux_x86_64.tar.gz`; set to `""` to skip validation (warns). Update alongside `backrest_version` |
| `backrest_user` | `"{{ ansible_user_id }}"` | User to run backrest as (overridden in host_vars) |
| `backrest_instance_id` | `"{{ inventory_hostname.split('.') \| first }}"` | Unique instance ID (hermes/deimos) |
| `backrest_data_dir` | `/home/{{ backrest_user }}/.local/share/backrest` | Data directory |
| `backrest_config_path` | `/home/{{ backrest_user }}/.config/backrest/config.json` | Config file path |
| `backrest_ui_port` | `9898` | UI port |
| `backrest_ui_bind` | `"127.0.0.1"` | Bind address (localhost only) |
| `backrest_backup_paths` | `["/home/{{ backrest_user }}"]` | Default backup paths |
| `backrest_excludes` | List of common excludes | Standard workstation excludes (.cache, .var, snap, etc.) |
| `backrest_exclude_if_present` | `["node_modules", ".venv", ...]` | Exclude-if-present patterns |
| `backrest_schedule` | `"0 */12 * * *"` | Default cron schedule |

## Post-Install Setup

1. **SSH tunnel to UI**: `ssh -L 9898:localhost:9898 hermes` (or deimos)
2. **Open UI**: `http://localhost:9898`
3. **Pair with central instance**:
   - Settings → Multihost → paste pairing token from central instance
   - Accept shared repo when offered
4. **Create backup plan**:
   - Name: `home`
   - Paths: `/home/{{ backrest_user }}`
   - Excludes: as templated (review and adjust)
   - Schedule: per host (hermes: `30 6 * * *`, deimos: `0 */12 * * *`)
   - Retention: disabled (managed centrally)
   - Prune/check: disabled (managed centrally)

## Notes

- Repo password lands in client's local config after pairing, protected only by FDE
- `~/.ssh` remains in scope (home dir backup covers it)
- The role templates a minimal config; plans/repos are added via UI or multihost sync
- Check mode (`--check`) should not crash
- All tasks are idempotent

## Inventory Wiring

Add to `host_vars/hermes.tartarus.us.yml`:

```yaml
backrest_user: benjamin
backrest_schedule: "30 6 * * *"  # daily at 6:30am
```

Add to `host_vars/deimos.tartarus.us.yml`:

```yaml
backrest_user: benjamin
backrest_schedule: "0 */12 * * *"  # every 12 hours
```
