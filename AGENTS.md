# AGENTS.md

## Repository purpose

- Ansible configuration for Benjamin's Debian-family workstations, applied by an hourly
  `ansible-pull` job on each host. This is infrastructure config, not an application:
  there is no build step and no real test suite.
- Main entrypoint: `ansible/local.yml`.
- Inventory: `ansible/inventory/hosts.yml` (`hermes` = laptop, `deimos` = desktop).
  Shared vars: `inventory/group_vars/all.yml`; per-host vars: `inventory/host_vars/`.
- Roles: `bootstrap` (ansible user, passwordless sudo, ansible-core, systemd pull
  service/timer, ntfy failure notification) and `common` (packages). Tags match role names.

## Commands

- Set up local Ansible: `python3 -m venv .venv && .venv/bin/pip install -r ansible/requirements.txt`
- Syntax check: `.venv/bin/ansible-playbook --syntax-check -i ansible/inventory/hosts.yml ansible/local.yml`
- Run playbook: `.venv/bin/ansible-playbook -i ansible/inventory/hosts.yml ansible/local.yml --ask-become-pass --vault-password-file ansible/vault-password`
  Add `--limit <host>` and/or `--tags bootstrap|common` to scope it.
- There is no `ansible.cfg` on this branch; always pass `-i` explicitly and run from the repo root.
- Linting is driven by `prek` (not `pre-commit`): `prek run --all-files`.
  Run one hook with e.g. `prek run ansible-lint` or `prek run yamllint`.
  CI runs the same hooks through `j178/prek-action`.
- `ansible-lint` and `markdownlint-cli2` run with `--fix` and will modify files.
- `ansible/roles/*/tests/` are stock placeholders and are excluded from ansible-lint;
  do not treat them as meaningful tests.

## Conventions and gotchas

- YAML must start with `---` (strict yamllint), use LF, and max line length is 160
  (warning only). `**/vault.yml` is excluded from yamllint.
- Secrets live in `ansible/roles/bootstrap/vars/vault.yml` (encrypted, provides
  `vault_ntfy_token`). The local password file is `ansible/vault-password` and is
  gitignored. Keep secrets encrypted; tasks touching them use `no_log`.
- Manage the vault with:
  `ansible-vault edit ansible/roles/bootstrap/vars/vault.yml --vault-password-file ansible/vault-password`
- On hosts, the systemd unit loads `/etc/credstore.encrypted/ansible-vault-password`
  via `LoadCredentialEncrypted`, decrypted by systemd at runtime. That credential is
  not provisioned by this playbook and must already exist before the timer can run.
- `ansible-pull.service.j2` pins `--checkout` (currently `feature/ansible-vault`), while
  the README bootstrap command uses `main`. Keep them in sync when merging/releasing,
  otherwise hosts keep running the wrong branch.
- `bootstrap` makes persistent, privileged changes (passwordless sudo for the `ansible`
  user, hourly pull timer). Verify with `--check`/`--limit` before applying broadly.
