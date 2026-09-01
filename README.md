# peedy2495.syshardening

Reusable Ansible roles for system hardening. The collection separates technical
implementation from policy and orchestration.

## Architecture

```text
roles/
├── ssh_client/                  # reusable implementation
├── firewall/                    # portable UFW/firewalld implementation
├── bsi_all_firewall/            # common BSI firewall policy
├── bsi_all/                     # common BSI dispatcher
├── bsi_ubuntu_26_ssh_client/    # BSI/Ubuntu 26 policy
└── bsi_ubuntu/                  # distribution/version orchestration
playbooks/
├── bsi_all.yml                  # common BSI entry point
├── bsi_ubuntu.yml               # Ubuntu BSI entry point
└── examples/                    # guarded deployment examples
```

- Implementation roles configure one component and contain no framework policy.
- Policy roles describe a named baseline for a platform and version.
- Orchestration roles select the applicable policy from Ansible facts.

Role names use lowercase letters and underscores. All role calls use their fully
qualified collection name.

## Current scope

The first platform target is the OpenSSH client on Ubuntu 26.04 LTS. The
`bsi_ubuntu_26_ssh_client` policy implements the applicable cryptographic
recommendations from BSI TR-02102-4 version 2026-01 and technical requirements
from BSI IT-Grundschutz OPS.1.2.5 Fernwartung. It uses strict allowlists,
public-key-only authentication, periodic rekeying, strict host-key checking, and
disabled forwarding. Technical rendering, ownership, validation, and cleanup
are delegated to `ssh_client`.

The policy-neutral `ssh_client` implementation role can also be used directly
with an explicitly supplied organization policy. See
[`roles/ssh_client/README.md`](roles/ssh_client/README.md).

The policy-neutral [`firewall`](roles/firewall/README.md) role detects UFW or
firewalld from operating-system facts. Its defaults make no changes. A policy
can define service state, portable directional defaults, and baseline inbound
or outbound rules; `firewall_additional_rules` appends host- and group-specific
rules through the same backend-independent schema.

The `bsi_all` dispatcher applies platform-independent BSI policies. Its current
`bsi_all_firewall` policy implements the technical baseline derived from BSI
IT-Grundschutz NET.3.2 with deny-by-default INPUT and FORWARD, selectable OUTPUT
protection, dual-stack handling, stateful filtering, and security logging.

## Usage

Apply the currently implemented BSI Ubuntu scope through orchestration:

```yaml
- name: Apply the BSI Ubuntu policy implemented by this collection
  hosts: all
  become: true
  roles:
    - role: peedy2495.syshardening.bsi_ubuntu
```

The same orchestration is available as a collection playbook:

```bash
ansible-playbook -i inventory peedy2495.syshardening.bsi_ubuntu
```

Guarded examples for the standard and increased BSI firewall protection levels
are documented in [`playbooks/README.md`](playbooks/README.md). They keep
organization-specific additions in inventory through
`firewall_additional_rules`.

Or configure the implementation role with an organization-owned policy:

```yaml
- name: Configure OpenSSH clients
  hosts: all
  become: true
  roles:
    - role: peedy2495.syshardening.ssh_client
      ssh_client_config_fragments:
        security:
          priority: 10
          enabled: true
          options:
            ForwardAgent: false
            StrictHostKeyChecking: "yes"
```

Build the collection locally with:

```bash
ansible-galaxy collection build .
```

## Migration from 1.0.0

The implementation role was renamed from `ssh-client` to `ssh_client` to follow
the collection naming rules. Update playbook role references accordingly. Its
public `ssh_client_config_*` variables are unchanged. Existing managed fragment
and manifest headers are recognized and migrated without taking ownership of
unrelated files.

## Adding coverage

Add technical behavior only to a component role such as `ssh_client`. Add
framework and version values to a policy role following
`<policy>_<platform>_<major_version>_<component>`, then register that policy in
the appropriate orchestration role. Public role variables must be documented
and every role must remain idempotent.
