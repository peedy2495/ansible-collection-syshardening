# Kernel implementation role

`peedy2495.syshardening.kernel` configures policy-neutral Linux kernel runtime
parameters and verifies build-time and boot-time protections. Empty defaults
make the role a no-op.

```yaml
kernel_sysctl_settings:
  kernel.randomize_va_space:
    value: 2
    state: present

kernel_config_requirements:
  CONFIG_STACKPROTECTOR_STRONG: y

kernel_forbidden_boot_parameters:
  - name: Disable ASLR
    pattern: '(^|\s)nokaslr(\s|$)'
```

## Public variables

- `kernel_sysctl_settings`: dictionary of sysctl names. Each item accepts
  `value` and `state` (`present` or `absent`).
- `kernel_config_requirements`: expected `CONFIG_*` values in the running
  kernel configuration.
- `kernel_forbidden_boot_parameters`: named regular expressions which must not
  match `/proc/cmdline`.
- `kernel_sysctl_file`: persistent configuration file; defaults to
  `/etc/sysctl.d/60-ansible-kernel.conf`.
- `kernel_apply_runtime`: apply values to the running kernel; defaults to
  `true`.
- `kernel_config_file`: running kernel build configuration; defaults to
  `/boot/config-{{ ansible_kernel }}`.

Build-time protections cannot safely be changed after compilation. Boot-time
protections are verified against the running command line so a policy cannot
claim compliance before the corrected boot configuration has taken effect.
The implementation deliberately does not edit a distribution-specific
bootloader.

Some sysctls, notably `kernel.modules_disabled` and
`kernel.kexec_load_disabled`, cannot be relaxed again without rebooting after
they are enabled. Policies should leave them unmanaged unless the operational
impact has been reviewed.
