# CIS Ubuntu 26 kernel module policy

This policy owns CIS filesystem and network protocol module availability and
delegates the implementation to `kernel_module`. Level 1 is the default;
Level 2 additionally disables overlayfs, squashfs, UDF, DCCP, TIPC, RDS, and
SCTP.

```yaml
cis_ubuntu_26_kernel_module_profile: level_2_server
cis_ubuntu_26_kernel_module_overrides:
  overlay_filesystem_module:
    enabled: false
```

Public variables:

- `cis_ubuntu_26_kernel_module_profile`: `level_1_server` or
  `level_2_server`;
- `cis_ubuntu_26_kernel_module_overrides`: per-control `enabled` deviations;
- `cis_ubuntu_26_kernel_module_config_file`: managed modprobe file;
- `cis_ubuntu_26_kernel_module_unload_disabled`: unload disabled modules from
  the running kernel.
