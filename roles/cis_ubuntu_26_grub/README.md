# CIS Ubuntu 26 GRUB policy

This policy owns the CIS AppArmor bootloader values and Ubuntu's GRUB update
command. It delegates configuration to `grub`, reboots only after a change, and
then delegates running-command-line verification to `kernel`.

```yaml
cis_ubuntu_26_grub_reboot_timeout: 900
```

Public variables:

- `cis_ubuntu_26_grub_overrides`: per-control `enabled` deviations;
- `cis_ubuntu_26_grub_defaults_file`: Ubuntu GRUB defaults file;
- `cis_ubuntu_26_grub_update_command`: command used to regenerate GRUB;
- `cis_ubuntu_26_grub_reboot_timeout`: reconnect timeout after the required
  reboot.
