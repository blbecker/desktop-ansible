# desktop-ansible

[![CI](https://github.com/blbecker/desktop-ansible/actions/workflows/ci.yml/badge.svg)](https://github.com/blbecker/desktop-ansible/actions/workflows/ci.yml)

## Bootstrapping a host

```bash
ansible-pull \
  -U git@github.com:blbecker/desktop-ansible.git \
  -d /tmp/desktop-ansible --ask-become-pass \
  -i ansible/inventory/hosts.yml \
  --limit "$(hostname -f)" \
  --tags bootstrap \
  --checkout main \
  ansible/local.yml
```
