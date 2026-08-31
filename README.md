# peedy2495.syshardening

Ansible collection for hardening IT systems.

## Contents

- `ssh-client`: OpenSSH client hardening via managed `ssh_config.d` fragments.

## Structure

```text
.
├── galaxy.yml
├── README.md
├── LICENSE
├── meta/
│   └── runtime.yml
└── roles/
    └── ssh-client/
```

## Usage

Reference the role in a playbook:

```yaml
- hosts: all
  become: true
  roles:
    - role: peedy2495.syshardening.ssh-client
```

Build the collection locally:

```bash
ansible-galaxy collection build .
```

## Notes

Detailed configuration and role behavior are documented in the role-specific README files.
