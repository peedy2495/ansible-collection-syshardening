# Kernel module implementation role

`peedy2495.syshardening.kernel_module` manages one role-owned modprobe policy
file and optionally unloads modules selected as disabled. It contains no CIS or
BSI policy.

```yaml
kernel_module_settings:
  cramfs:
    state: disabled
  overlay:
    state: available
```

`disabled` writes both an `install ... /bin/false` rule and a blacklist entry.
`available` removes this role's prohibition but does not load the module. The
role refuses invalid module names and uses `community.general.modprobe` for
idempotent unloading.

Public variables:

- `kernel_module_settings`: module mapping with `state` set to `disabled` or
  `available`; default `{}`.
- `kernel_module_config_file`: role-owned modprobe file; default
  `/etc/modprobe.d/60-ansible-kernel-modules.conf`.
- `kernel_module_unload_disabled`: unload disabled modules; default `true`.
