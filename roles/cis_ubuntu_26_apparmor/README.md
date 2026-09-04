# CIS Ubuntu 26 AppArmor policy

This policy owns Ubuntu package names plus the AppArmor runtime, service, and
profile requirements. It delegates installation and configuration to the
generic `apparmor` implementation role.

`level_1_server` installs `apparmor` and `apparmor-utils`, enables the service,
verifies kernel activation, and enables profiles disabled through marker
symlinks. `level_2_server` additionally requires that no loaded profile remains
in complain mode.

```yaml
cis_ubuntu_26_apparmor_overrides:
  packages:
    enabled: false
  profiles_enforcing:
    enabled: false
```

Public variables:

- `cis_ubuntu_26_apparmor_profile`: `level_1_server` or `level_2_server`;
- `cis_ubuntu_26_apparmor_overrides`: per-control `enabled` deviations.
