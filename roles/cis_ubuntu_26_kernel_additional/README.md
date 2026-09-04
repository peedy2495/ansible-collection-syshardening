# Additional CIS Ubuntu 26 kernel dispatcher

`peedy2495.syshardening.cis_ubuntu_26_kernel_additional` contains no policy
values and no technical implementation. It dispatches the independently
maintained component policies in their required order:

1. `cis_ubuntu_26_kernel_module`;
2. `cis_ubuntu_26_grub`;
3. `cis_ubuntu_26_apparmor`;
4. `cis_ubuntu_26_core_dump`.

The AppArmor policy supplies its Ubuntu package names to the generic AppArmor
implementation. GRUB configuration and a possible reboot complete before
AppArmor installation, service configuration, and runtime verification begin.

Each component has its own documented override namespace. The dispatcher
itself exposes no variables.
