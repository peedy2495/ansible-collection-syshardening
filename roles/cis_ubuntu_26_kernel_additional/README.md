# Additional CIS Ubuntu 26 kernel policy

`peedy2495.syshardening.cis_ubuntu_26_kernel_additional` contains the
kernel-related CIS Ubuntu Linux 26.04 LTS Benchmark v1.0.0 controls not already
covered by `bsi_ubuntu_26_kernel` and `cis_ubuntu_26_kernel_network`.

It delegates technical work to the policy-neutral `kernel_module`, `kernel`,
and `core_dump` roles. The policy covers filesystem and network protocol module
availability, AppArmor activation at boot, and the process core-file limit.
ASLR, `ptrace_scope`, and `fs.suid_dumpable` remain owned by the BSI policy.

## Profiles

`cis_ubuntu_26_kernel_additional_profile` defaults to `level_1_server`.
`level_2_server` also disables overlayfs, squashfs, UDF, DCCP, TIPC, RDS, and
SCTP. These controls have significant compatibility impact and their inactive
Level 1 state is represented explicitly in the policy catalog.

The AppArmor boot control verifies the running kernel command line. It does not
edit a bootloader. If `apparmor=1` and `security=apparmor` are missing, configure
the system bootloader and reboot before applying the role again.

## Public variables

- `cis_ubuntu_26_kernel_additional_profile`: `level_1_server` or
  `level_2_server`.
- `cis_ubuntu_26_kernel_additional_overrides`: recursive per-control deviations;
  each entry accepts only `enabled`.
- `cis_ubuntu_26_kernel_additional_module_config_file`: managed modprobe file.
- `cis_ubuntu_26_kernel_additional_limits_file`: managed limits drop-in.
- `cis_ubuntu_26_kernel_additional_unload_modules`: unload disabled modules at
  runtime; disabling this is intended for controlled tests only.

For example, Docker requires overlayfs to remain available even under Level 2:

```yaml
cis_ubuntu_26_kernel_additional_overrides:
  overlay_filesystem_module:
    enabled: false
```

An exception deliberately makes the corresponding CIS recommendation
non-compliant and should be documented with the host's operational policy.
