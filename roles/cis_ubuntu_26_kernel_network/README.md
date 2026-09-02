# CIS Ubuntu 26 kernel network policy

`peedy2495.syshardening.cis_ubuntu_26_kernel_network` implements section 3.3
of CIS Ubuntu Linux 26.04 LTS Benchmark v1.0.0. It complements rather than
duplicates `bsi_ubuntu_26_kernel` and delegates all sysctl changes to the
policy-neutral `kernel` implementation role.

The role manages `/etc/sysctl.d/61-cis-ubuntu-26-kernel-network.conf` so its
provenance remains separate from the BSI kernel policy. It applies runtime
values immediately. IPv6 settings are skipped when the running kernel exposes
no IPv6 sysctl tree.

## Standard policy

| CIS | Options | Value | Reason and operational impact |
| --- | --- | --- | --- |
| 3.3.1 | IPv4 and IPv6 forwarding | `0` | Prevents unintended routing; routers and container networking require documented exceptions. |
| 3.3.2 | IPv4 redirect sending, `all` and `default` | `0` | Prevents the host from advertising alternate routes. |
| 3.3.3 | Ignore bogus IPv4 ICMP errors | `1` | Discards malformed ICMP diagnostics. |
| 3.3.4 | Ignore IPv4 broadcast echo | `1` | Prevents broadcast amplification. |
| 3.3.5 | Accept IPv4/IPv6 redirects, `all` and `default` | `0` | Prevents redirect-based route manipulation. |
| 3.3.6 | Accept secure IPv4 redirects, `all` and `default` | `0` | Rejects redirects even from configured gateways. |
| 3.3.7 | IPv4 reverse-path filtering, `all` and `default` | `1` | Rejects spoofed sources; strict mode can break asymmetric routing. |
| 3.3.8 | Accept IPv4/IPv6 source routes, `all` and `default` | `0` | Prevents senders from selecting packet routes. |
| 3.3.9 | Log IPv4 martians, `all` and `default` | `1` | Makes suspicious sources visible but can increase log volume. |
| 3.3.10 | TCP SYN cookies | `1` | Protects connection queues during SYN floods. |
| 3.3.11 | Accept IPv6 router advertisements, `all` and `default` | `0` | Prevents unsolicited IPv6 route configuration; SLAAC hosts need an exception. |

The canonical catalog in `vars/main.yml` records a reason and impact for every
individual sysctl.

## Inventory deviations

Only deviations are placed in `group_vars` or `host_vars`:

```yaml
cis_ubuntu_26_kernel_network_overrides:
  ipv4_forwarding:
    value: 1
```

Changing `value` retains the standard enabled state. Setting `enabled: false`
removes a previously managed persistent entry and then leaves the option
unmanaged; it does not actively restore a different runtime value.

Docker bridge networking normally requires IPv4 forwarding. A rootful Docker
host can use:

```yaml
# host_vars/docker01/kernel.yml
bsi_ubuntu_26_kernel_overrides: {}

cis_ubuntu_26_kernel_network_overrides:
  ipv4_forwarding:
    value: 1
```

If Docker IPv6 networking is explicitly enabled, add:

```yaml
  ipv6_forwarding:
    value: 1
```

These forwarding deviations deliberately fail CIS recommendation 3.3.1 and
must be recorded in the system's applicability and exception documentation.
The firewall FORWARD policy remains independent and should continue to allow
only the required container flows.

Reference: [CIS Ubuntu Linux Benchmarks](https://www.cisecurity.org/benchmark/ubuntu_linux)
