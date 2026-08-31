# Role: ssh-client

This role is part of the `peedy2495.syshardening` collection. It manages modular,
system-wide OpenSSH client policy fragments below `/etc/ssh/ssh_config.d`.
It never manages `sshd_config`, restarts no service, and does not change
operating-system crypto policies.

## Requirements

- ansible-core 2.15 or newer
- An installed OpenSSH client
- A vendor `/etc/ssh/ssh_config` that includes
  `/etc/ssh/ssh_config.d/*.conf` (the role verifies this by default)
- Privilege escalation sufficient to write system configuration as `root`

The supported operating-system variants are:

- Debian
- Ubuntu
- Red Hat Enterprise Linux
- Rocky Linux
- AlmaLinux

## OpenSSH option syntax

This role uses the option grammar documented by the current OpenSSH 10.x
[`ssh_config(5)` manual](https://man.openbsd.org/ssh_config). OpenSSH does not
publish a parallel supported-major-version matrix.

Options and algorithms available in OpenSSH 10.x are not assumed to exist in
every vendor client. Before replacing a fragment, the role validates it with the
OpenSSH client installed on the managed host. Client-side `Include` support and
an active vendor include for `ssh_config.d` are required. This runtime check is
the authoritative compatibility decision and avoids an unmaintainable embedded
option/version matrix.

## Configuration model

The default policy creates four fragments:

```text
10-crypto.conf
20-access.conf
30-security.conf
40-network.conf
```

Every variable is prefixed with `ssh_client_config_`. The primary input is a
mapping whose keys are safe fragment names. Names must match
`^[a-z0-9][a-z0-9_-]*$`; unsafe names are rejected rather than silently changed.
Priorities are integers from 0 through 99 and are rendered with two digits.

```yaml
ssh_client_config_fragments:
  crypto:
    priority: 10
    enabled: true
    options:
      Ciphers:
        - chacha20-poly1305@openssh.com
        - aes256-gcm@openssh.com
      MACs:
        - hmac-sha2-512-etm@openssh.com
        - hmac-sha2-256-etm@openssh.com
      KexAlgorithms:
        - curve25519-sha256

  access:
    priority: 20
    enabled: true
    options:
      PreferredAuthentications:
        - publickey
      PasswordAuthentication: false
      PubkeyAuthentication: true

  security:
    priority: 30
    enabled: true
    options:
      StrictHostKeyChecking: "yes"
      HashKnownHosts: true
      ForwardAgent: false

  network:
    priority: 40
    enabled: true
    options:
      ServerAliveInterval: 30
      ServerAliveCountMax: 3
      ConnectTimeout: 10
```

The short `options` form produces one `Host *` block. For host-specific policy,
use ordered `blocks` instead:

```yaml
ssh_client_config_fragments:
  destinations:
    priority: 5
    enabled: true
    blocks:
      - host: "git.example.com"
        options:
          User: git
          IdentityFile:
            - /etc/ssh/keys/current
            - /etc/ssh/keys/previous
      - host: "*"
        options:
          ForwardAgent: false
```

Do not specify `options` and `blocks` together. Block order is preserved;
options within a block are sorted for deterministic output. Booleans become
`yes`/`no` and numbers remain unquoted numbers.

Known algorithm/preference lists are comma-separated. Unknown list-valued
options are emitted as repeated directives because repeated syntax is safer
than assuming every OpenSSH directive accepts commas. Customize this explicitly:

```yaml
ssh_client_config_list_modes:
  Ciphers: comma
  CanonicalDomains: space
  IdentityFile: repeat
  MyVendorOption: comma
```

Valid modes are `comma`, `space`, and `repeat`.

## Priority and OpenSSH evaluation order

The vendor `ssh_config` expands matching `Include` files in lexical order.
OpenSSH generally uses the **first value obtained** for each parameter. Thus a
lower number is earlier and has higher precedence; `10-crypto.conf` does not get
overridden by the same `Host *` option in `40-network.conf`. This role deliberately
does not implement a fictitious last-wins merge.

Put narrow host matches before general defaults. Also remember that command-line
and user configuration are read before system-wide configuration and therefore
usually take precedence. The exact position of the vendor `Include` inside
`/etc/ssh/ssh_config` can also affect surrounding vendor defaults.

## Variables

| Variable | Default | Purpose |
|---|---:|---|
| `ssh_client_config_dir` | `/etc/ssh/ssh_config.d` | Fragment directory |
| `ssh_client_config_vendor_file` | `/etc/ssh/ssh_config` | Vendor config inspected for the include |
| `ssh_client_config_require_include` | `true` | Require the vendor wildcard include |
| `ssh_client_config_fragment_suffix` | `.conf` | Included suffix |
| `ssh_client_config_manifest_file` | `.ansible-ssh-client-config.manifest` | Private ownership ledger |
| `ssh_client_config_manage_removed_fragments` | `true` | Remove stale managed files |
| `ssh_client_config_validate` | `true` | Validate each candidate with `ssh -G` |
| `ssh_client_config_validation_host` | `ansible-validation.invalid` | Host argument for `ssh -G` |
| `ssh_client_config_binary` | `ssh` | OpenSSH client command/path |
| `ssh_client_config_warn_crypto_policy` | `true` | Report detected EL crypto policy |
| `ssh_client_config_list_modes` | See defaults | Per-option list serialization |
| `ssh_client_config_fragments` | Four policy fragments | Structured policy input |

See [`defaults/main.yml`](defaults/main.yml) for the complete defaults.

## Ownership and removal

Each fragment has a role-specific managed header. The role refuses to overwrite
a target file without that header, so a priority/name collision cannot silently
replace an administrator's file. It records exact managed basenames in a mode
`0600` manifest. Removed or disabled fragments are deleted only when their name
was in that manifest; unrelated `.conf` files are never scanned or deleted.

Set `ssh_client_config_manage_removed_fragments: false` to retain stale files.
They remain in the manifest, allowing a later run with removal enabled to clean
them up.

## Validation and safety

Before each changed fragment is atomically renamed into place, the
`ansible.builtin.template` validation hook runs:

```text
ssh -G -F <temporary-candidate> ansible-validation.invalid
```

This catches unknown directives, invalid values, and algorithms unsupported by
the installed client without opening a network connection. Validation of one
candidate cannot provide a transaction spanning several files, and `ssh -G`
cannot prove that a remote server accepts the resulting policy. The main vendor
include is checked separately. Set validation to `false` only for unusual test
or chroot environments.

On RHEL-compatible systems, system-wide crypto policies may further restrict
algorithms. The role detects `/etc/crypto-policies/config` and emits an optional
notice. It never runs `update-crypto-policies`, alters a global policy, or opts
OpenSSH out of that policy.

## Collection usage

Within the collection, reference the role by its collection-qualified name:

```yaml
- name: Configure SSH clients
  hosts: all
  become: true
  roles:
    - role: peedy2495.syshardening.ssh-client
```

## Testing

The default Molecule scenario covers Debian, Ubuntu, and Rocky Linux.
It tests rendering, filenames, boolean/list handling, disabled and stale
fragments, preservation of a foreign file, `ssh -G`, and idempotence. The
`invalid` scenario verifies a clear validation failure. Container images and a
working Docker or Podman runtime are required.

```bash
yamllint .
ansible-lint
molecule test -s default
molecule test -s invalid
```

Container tests validate local parsing only; they do not exercise remote SSH
negotiation or subscription-only RHEL repositories.
