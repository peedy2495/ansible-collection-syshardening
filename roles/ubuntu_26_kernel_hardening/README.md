# Ubuntu 26 kernel hardening orchestration

`peedy2495.syshardening.ubuntu_26_kernel_hardening` combines three independent
policy roles:

1. `bsi_ubuntu_26_kernel` for the BSI SYS.1.3-derived process, memory,
   filesystem, build-time, and boot-time kernel controls;
2. `cis_ubuntu_26_kernel_network` for CIS Ubuntu Linux 26.04 LTS Benchmark
   v1.0.0 section 3.3 network kernel parameters;
3. `cis_ubuntu_26_kernel_additional` as a dispatcher for the independent
   kernel-module, GRUB, AppArmor, and core-dump policy roles.

The orchestration contains no technical implementation. Inventory overrides
retain their policy-specific namespaces:

```yaml
bsi_ubuntu_26_kernel_overrides: {}
cis_ubuntu_26_kernel_network_overrides: {}
cis_ubuntu_26_kernel_module_overrides: {}
cis_ubuntu_26_grub_overrides: {}
cis_ubuntu_26_apparmor_overrides: {}
cis_ubuntu_26_core_dump_overrides: {}
```

Use the CIS network override interface to document router, Docker, asymmetric
routing, or SLAAC exceptions. Use the kernel-module policy interface for
module exceptions such as Docker's required overlayfs support. Such deviations
can make individual CIS checks non-compliant even when operationally required.

The additional CIS policy can reboot a host once when required AppArmor kernel
parameters are missing. The reboot happens only after the managed GRUB defaults
change and completes before runtime verification continues. Roll the policy out
serially where simultaneous host reboots would affect availability.

## Docker hosts

Keep all Docker deviations together in `group_vars/docker/` or the applicable
`host_vars/` file. A normal rootful Docker host with the `overlay2` storage
driver needs IPv4 forwarding and overlayfs. Explicitly retaining module loading
also protects the host if an increased BSI policy enables the irreversible
module-loading lock elsewhere in inventory.

IPv6 forwarding is required only when Docker IPv6 networking is enabled. SCTP
is required only by workloads that use SCTP. Rootless Docker additionally
needs unprivileged user namespaces; prefer a compatible AppArmor profile over
disabling Ubuntu's AppArmor user-namespace restriction.

Inventory dictionaries do not recursively merge between multiple Ansible group
levels by default. Define each policy's complete host-specific override mapping
at the winning inventory precedence level, or assemble it there from separately
named organization variables.

```yaml
bsi_ubuntu_26_kernel_overrides:
  lock_module_loading:
    enabled: false

cis_ubuntu_26_kernel_network_overrides:
  ipv4_forwarding:
    value: 1

cis_ubuntu_26_kernel_module_overrides:
  overlay_filesystem_module:
    enabled: false
```
