# Role: bsi_ubuntu_26_ssh_client

Policy role for a hardened system-wide OpenSSH client on Ubuntu 26.04. It
implements applicable settings from:

- [BSI TR-02102-4, Version 2026-01](https://www.bsi.bund.de/SharedDocs/Downloads/DE/BSI/Publikationen/TechnischeRichtlinien/TR02102/BSI-TR-02102-4.pdf?__blob=publicationFile)
- [BSI IT-Grundschutz OPS.1.2.5 Fernwartung, Edition 2023](https://www.bsi.bund.de/SharedDocs/Downloads/DE/BSI/Grundschutz/IT-GS-Kompendium_Einzel_PDFs_2023/04_OPS_Betrieb/OPS_1_2_5_Fernwartung_Edition_2023.pdf?__blob=publicationFile&v=3)

The role validates Ubuntu major version 26 and delegates technical configuration
to `peedy2495.syshardening.ssh_client`. Every generated fragment contains its
policy references as comments.

## Implemented requirements

| Fragment | Configuration | Reference |
|---|---|---|
| `crypto` | Strict allowlists for KEX, ciphers, and MACs | TR-02102-4 sections 3.3-3.5; OPS.1.2.5.A8 paragraph 1 |
| `rekey` | Rekey after 1 GiB or one hour | TR-02102-4 section 3.3.1 paragraph 1 sentence 3 |
| `server_authentication` | ECDSA NIST host keys and strict host-key checking | TR-02102-4 section 3.6 table 5; section 4.1 paragraph 2 |
| `client_authentication` | Public-key-only client authentication using ECDSA NIST | TR-02102-4 section 3.7 paragraph 3 and section 3.6 table 5 |
| `connection_scope` | Disable forwarding and persistent multiplexed sessions | OPS.1.2.5.A3 paragraph 1 sentences 1-2 |

`diffie-hellman-group15-sha512` is absent because OpenSSH 10.2 does not
implement it. `diffie-hellman-group-exchange-sha256` is intentionally absent
because `ssh_config` cannot enforce the minimum group parameters required by the
remarks in TR-02102-4 section 3.3. The policy therefore uses only the remaining
recommended, enforceable KEX algorithms.

TR-02102-4 version 2026-01 does not yet recommend an SSH post-quantum algorithm.
Section 3.1.3 states that hybrid algorithms are to be recommended after suitable
standards have been adopted; section 3.3 names proposed algorithms for a future
recommendation. Ubuntu's OpenSSH post-quantum defaults are therefore excluded by
this strict BSI allowlist.

## Operational requirements outside ssh_config

The following requirements cannot be completely enforced by an SSH client
configuration and remain part of the operating concept:

- OPS.1.2.5.A3: install remote-maintenance software only where needed and close
  every connection after maintenance
- OPS.1.2.5.A17 paragraph 1: enforce multi-factor authentication and document
  the selected method
- OPS.1.2.5.A20 paragraph 3: log all remote-maintenance operations
- OPS.1.2.5.A25: use a jump server for access from outside management networks
- TR-02102-4 section 4.1 paragraph 1: protect private keys against copying,
  misuse, and manipulation, preferably using suitable hardware

The configured public-key-only authentication can use a hardware-backed ECDSA
key, but the role cannot prove hardware use or multi-factor enforcement.

## Compatibility impact

Remote servers must offer an ECDSA NIST host key and one of the configured KEX,
cipher, and MAC combinations. Users need an accepted ECDSA key. RSA, Ed25519,
password-only, and keyboard-interactive-only environments will not connect.
Test this policy against all destinations before production rollout.

## Variable

| Variable | Default | Purpose |
|---|---|---|
| `bsi_ubuntu_26_ssh_client_fragments` | Five referenced fragments | Complete policy mapping passed to `ssh_client_config_fragments` |

Site-specific tailoring can override the mapping, but deviations should be
recorded in the institution's cryptographic and remote-maintenance policies.
