# Firewall implementation role

`peedy2495.syshardening.firewall` is a policy-neutral implementation role. It
detects the backend from `ansible_os_family`: Debian uses UFW; Red Hat and SUSE
families use firewalld. There is deliberately no public backend selector.

The role does not install packages, enable a service, change default policies,
or add rules with its defaults. Facts must be gathered, and the operating
system's firewall package plus its Ansible module dependencies must already be
installed.

## Public variables

| Variable | Default | Meaning |
| --- | --- | --- |
| `firewall_state` | `unchanged` | `unchanged`, `enabled`, or `disabled` |
| `firewall_default_policies` | `{}` | Optional defaults for `input`, `output`, and `forward` |
| `firewall_loopback` | `unchanged` | `unchanged` or explicitly `allow` loopback |
| `firewall_stateful` | `unchanged` | `unchanged` or require backend state tracking |
| `firewall_ipv6` | `unchanged` | `unchanged` or require IPv4/IPv6 filtering with `enabled` |
| `firewall_log_denied` | `unchanged` | `unchanged`, `enabled`, or `disabled` denied-traffic logging |
| `firewall_log_configuration_changes` | `false` | Write a target syslog event after changes |
| `firewall_rules` | `[]` | Policy-owned rules |
| `firewall_additional_rules` | `[]` | Inventory-owned rules appended to policy rules |
| `firewall_additional_rules_<scope>` | undefined | Independently named inventory rule lists, discovered and appended automatically |

All default directions use the same values:

```yaml
firewall_default_policies:
  input: deny       # allow, deny, or reject
  output: allow     # allow, deny, or reject
  forward: deny     # allow, deny, or reject
```

UFW maps these values to its incoming, outgoing, and routed defaults. firewalld
uses dedicated policy objects: `ANY` to `HOST` for input, `HOST` to `ANY` for
output, and `ANY` to `ANY` for forwarding. This avoids treating a firewalld zone
target as a global OUTPUT policy.

Each item in either rule list uses the same backend-independent schema:

```yaml
- name: Allow SSH from administration network
  action: allow                    # allow, deny, reject
  direction: input                # input, output, forward; default: input
  protocol: tcp                    # any, tcp, udp, sctp; default: any
  source: 192.0.2.0/24             # default: any
  source_port: "1024-65535"        # optional, N or N-N; not with destination_port
  destination: any                 # default: any
  destination_port: "22"           # optional, N or N-N
  log: false                       # default: false
  state: present                   # present or absent; default: present
```

A port requires an explicit transport protocol. UFW commands and firewalld
policy XML are generated only inside their respective backend implementation.
Rules without `direction` remain compatible and default to `input`.

## Policy plus inventory additions

A policy role supplies the baseline without replacing local additions:

```yaml
- name: Apply organization firewall baseline
  ansible.builtin.include_role:
    name: peedy2495.syshardening.firewall
  vars:
    firewall_state: enabled
    firewall_default_policies:
      input: deny
      output: allow
      forward: deny
    firewall_rules:
      - name: Allow SSH from administration network
        action: allow
        protocol: tcp
        source: 192.0.2.0/24
        destination_port: "22"
```

Inventory can add host- or group-specific rules without redefining the policy:

```yaml
firewall_additional_rules:
  - name: Allow application health checks
    action: allow
    protocol: tcp
    source: 198.51.100.10/32
    destination_port: "8080"
```

Ansible replaces a list when a more specific inventory group defines the same
variable. For hosts belonging to several groups, give every scope an independent
variable name. The role automatically discovers variables matching
`firewall_additional_rules_<scope>` and appends all of them:

```yaml
# group_vars/all/firewall.yml
firewall_additional_rules_all:
  - name: Allow SSH from administration network
    action: allow
    direction: input
    protocol: tcp
    source: 192.0.2.0/24
    destination_port: "22"
```

```yaml
# group_vars/docker/firewall.yml
firewall_additional_rules_docker:
  - name: Allow Docker service from application network
    action: allow
    direction: input
    protocol: tcp
    source: 198.51.100.0/24
    destination_port: "2376"
```

```yaml
# group_vars/ipa/firewall.yml
firewall_additional_rules_ipa:
  - name: Allow IPA HTTPS from client network
    action: allow
    direction: input
    protocol: tcp
    source: 203.0.113.0/24
    destination_port: "443"
```

A host in both `docker` and `ipa` receives the `all`, `docker`, and `ipa` rule
sets. Hostvars can contribute another uniquely named list, for example
`firewall_additional_rules_host`. Scope suffixes must contain only letters,
digits, and underscores.

Rule order is `firewall_rules`, `firewall_additional_rules`, and then the scoped
lists sorted by variable name. Keep names stable; UFW stores them as comments.
To remove a UFW rule reliably, retain its full definition temporarily and set
`state: absent`.

The firewalld backend requires firewalld 0.9.0 or newer for policy objects. It
manages three reserved policy files named `ansible-input`, `ansible-output`, and
`ansible-forward`. It validates permanent configuration with firewalld's online
or offline configuration checker and reloads only when firewalld was already
running. If it was stopped, the files take effect on the next start. firewalld
implements established/related and loopback acceptance in its native base
chains; the role relies on those backend invariants instead of duplicating
them in a policy object.

## Implementation references

- [community.general.ufw module](https://docs.ansible.com/projects/ansible/latest/collections/community/general/ufw_module.html)
- [firewalld policy concepts](https://firewalld.org/documentation/man-pages/firewalld.policies)
- [firewalld policy XML format](https://firewalld.org/documentation/man-pages/firewalld.policy.html)
