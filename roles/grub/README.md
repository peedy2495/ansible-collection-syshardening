# GRUB implementation role

`peedy2495.syshardening.grub` manages kernel parameters in the
double-quoted `GRUB_CMDLINE_LINUX` assignment in `/etc/default/grub`. It
preserves unrelated arguments and replaces an existing value for every managed
parameter.

When the file changes, the role runs the explicitly supplied GRUB update
command and can reboot the host before returning. A converged second execution
does not regenerate GRUB or reboot. Safe defaults perform no update or reboot;
a policy using non-empty settings must supply its platform command.

```yaml
grub_kernel_parameters:
  apparmor:
    state: present
    value: "1"
  security:
    state: present
    value: apparmor
```

Public variables:

- `grub_kernel_parameters`: parameter mapping; default `{}`.
- `grub_defaults_file`: platform configuration file; default empty and required
  with non-empty parameter settings.
- `grub_update_command`: platform update command; default `[]`.
- `grub_reboot_on_change`: defaults to `false`.
- `grub_reboot_timeout`: defaults to 600 seconds.
