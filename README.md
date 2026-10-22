# desktop-ansible

## Bootstrapping a host
```bash
ansible-pull \
  -U git@github.com:blbecker/desktop-ansible.git \
  -d /tmp/desktop-ansible --ask-become-pass \
  -i ansible/inventory/hosts.yml \
  --limit "$(hostname -f)" \
  --tags bootstrap \
  --checkout main \
  ansible/playbooks/local.yml
```
