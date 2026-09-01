# Role: ssh_client

Policy-neutral implementation role for modular, system-wide OpenSSH client
configuration below `/etc/ssh/ssh_config.d`. It never manages `sshd_config`,
restarts a service, or changes operating-system crypto policy.

## Requirements

- ansible-core 2.14 or newer
- OpenSSH client installed on the managed host
- `/etc/ssh/ssh_config` includes `/etc/ssh/ssh_config.d/*.conf` when
  `ssh_client_config_require_include` is enabled
- root privileges for system configuration

The role has no distribution policy and relies on the installed `ssh` binary to
validate option and algorithm compatibility before changing a fragment.

## Public interface

| Variable | Default | Purpose |
|---|---:|---|
| `ssh_client_config_dir` | `/etc/ssh/ssh_config.d` | Fragment directory |
| `ssh_client_config_vendor_file` | `/etc/ssh/ssh_config` | Main configuration inspected for the include |
| `ssh_client_config_require_include` | `true` | Require the fragment wildcard include |
| `ssh_client_config_fragment_suffix` | `.conf` | Managed filename suffix |
| `ssh_client_config_manifest_file` | `.ansible-ssh-client-config.manifest` | Private ownership ledger |
| `ssh_client_config_manage_removed_fragments` | `true` | Remove stale, role-owned fragments |
| `ssh_client_config_validate` | `true` | Validate candidates with `ssh -G` |
| `ssh_client_config_validation_host` | `ansible-validation.invalid` | Host used for local parsing |
| `ssh_client_config_binary` | `ssh` | Validation executable |
| `ssh_client_config_warn_crypto_policy` | `true` | Report an EL system crypto policy |
| `ssh_client_config_list_modes` | See defaults | List serialization by OpenSSH option |
| `ssh_client_config_fragments` | `{}` | Policy supplied by the caller |

The empty fragment default is intentional: implementation roles do not define
CIS, STIG, BSI, or organization policy.

## Fragment model

```yaml
ssh_client_config_fragments:
  security:
    priority: 10
    enabled: true
    references:
      - BSI TR-02102-4 (2026-01), Abschnitt 3.4, Tabelle 3
    options:
      ForwardAgent: false
      StrictHostKeyChecking: "yes"
```

Fragment names must match `^[a-z0-9][a-z0-9_-]*$`. Priorities are integers from
0 through 99 and become two-digit filename prefixes. `options` creates one
`Host *` block. Optional `references` are emitted as comments for policy
traceability. Use ordered `blocks` for host-specific configuration:

```yaml
ssh_client_config_fragments:
  destinations:
    priority: 5
    enabled: true
    blocks:
      - host: git.example.com
        options:
          User: git
          IdentityFile:
            - /etc/ssh/keys/current
            - /etc/ssh/keys/previous
      - host: "*"
        options:
          ForwardAgent: false
```

Do not set `options` and `blocks` together. Block order is preserved and options
within each block are sorted for deterministic output. Booleans render as
`yes`/`no`. Known algorithm lists are comma-separated; unknown lists render as
repeated directives unless their mode is set to `comma` or `space` in
`ssh_client_config_list_modes`.

OpenSSH generally uses the first value obtained for a parameter. Lower-numbered
fragments therefore have earlier evaluation and normally higher precedence.
User and command-line configuration may take precedence over system fragments.

## Ownership and validation

The role refuses to replace unrelated files. Managed basenames are recorded in
a mode `0600` manifest, and stale files are deleted only when they were recorded
there and still carry the role header. Headers created by the former
`ssh-client` role name are accepted and migrated on the next change.

Changed fragments are validated before atomic replacement with the equivalent
of:

```text
ssh -G -F <candidate> ansible-validation.invalid
```

This checks local parsing but cannot prove that a remote SSH server accepts the
policy. Set `ssh_client_config_validate: false` only in controlled chroot or test
environments.
