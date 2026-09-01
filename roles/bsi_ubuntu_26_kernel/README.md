# BSI Ubuntu 26 kernel policy

`peedy2495.syshardening.bsi_ubuntu_26_kernel` translates the technology-neutral
kernel requirements from BSI IT-Grundschutz SYS.1.3, Edition 2023, into Ubuntu
26.04 controls. It delegates sysctl configuration and kernel verification to
the reusable `kernel` implementation role.

BSI SYS.1.3.A4 explicitly requires ASLR and DEP/NX and prohibits disabling
kernel and library protections. SYS.1.3.A10, A14, A16, and A17 describe
additional confinement, information hiding, syscall reduction, memory
protection, and filesystem protection. They do not prescribe concrete Linux
parameter values; the values below are the Ubuntu-specific policy realization.

## Standard policy

| Option | State | Value/check | Reason and operational impact |
| --- | --- | --- | --- |
| `aslr` | enabled | `kernel.randomize_va_space=2` | Full process ASLR; fixed-address legacy software can fail. |
| `dep_nx` | enabled | reject `noexec=off` | Preserves NX; executable-data software needs explicit mappings. |
| `stack_protector` | enabled | `CONFIG_STACKPROTECTOR_STRONG=y` | Detects stack corruption with minimal overhead. |
| `strict_kernel_memory_permissions` | enabled | `CONFIG_STRICT_KERNEL_RWX=y` | Enforces kernel W^X; incompatible kernel code can fail. |
| `strict_module_memory_permissions` | enabled | `CONFIG_STRICT_MODULE_RWX=y` | Enforces module W^X; incompatible modules can fail. |
| `hardened_usercopy` | enabled | `CONFIG_HARDENED_USERCOPY=y` | Checks user/kernel copies and exposes invalid drivers. |
| `fortify_source` | enabled | `CONFIG_FORTIFY_SOURCE=y` | Adds bounds checks with minimal overhead. |
| `kernel_aslr` | enabled | `CONFIG_RANDOMIZE_BASE=y` | Enables KASLR; addresses change after reboot. |
| `kernel_aslr_boot_guard` | enabled | reject `nokaslr` | Prevents disabling KASLR; stable-address debugging is unavailable. |
| `vmapped_kernel_stack` | enabled | `CONFIG_VMAP_STACK=y` | Adds stack guard pages with slight VM overhead. |
| `cpu_vulnerability_mitigations` | enabled | reject disabling boot flags | Preserves CPU mitigations with workload-dependent cost. |
| `dmesg_restriction` | enabled | `kernel.dmesg_restrict=1` | Hides diagnostics; viewing requires privileges. |
| `kernel_pointer_restriction` | enabled | `kernel.kptr_restrict=1` | Hides pointers; low-level diagnostics need privileges. |
| `ptrace_restriction` | enabled | `kernel.yama.ptrace_scope=1` | Limits tracing; cross-process debugging needs privileges. |
| `minimum_mmap_address` | enabled | `vm.mmap_min_addr=65536` | Protects low memory; old applications can fail. |
| `privileged_core_dumps` | enabled | `fs.suid_dumpable=0` | Stops privileged dumps; reduces crash diagnostics. |
| `protected_hardlinks` | enabled | `fs.protected_hardlinks=1` | Blocks unsafe hardlinks; restricts links to foreign files. |
| `protected_symlinks` | enabled | `fs.protected_symlinks=1` | Blocks shared-directory symlink races. |
| `protected_fifos` | enabled | `fs.protected_fifos=2` | Protects FIFOs, including group-writable sticky directories. |
| `protected_regular_files` | enabled | `fs.protected_regular=2` | Protects regular files in shared sticky directories. |
| `tty_line_discipline_autoload` | enabled | `dev.tty.ldisc_autoload=0` | Reduces autoload attack surface; uncommon devices may need admin loading. |
| `unprivileged_bpf` | enabled | `kernel.unprivileged_bpf_disabled=2` | Blocks unprivileged BPF but remains reversible by an administrator. |
| `bpf_jit_hardening` | enabled | `net.core.bpf_jit_harden=2` | Hardens all BPF JIT output with compilation overhead. |
| `unprivileged_userfaultfd` | enabled | `vm.unprivileged_userfaultfd=0` | Restricts userfaultfd; some migration tools need privileges. |
| `perf_event_restriction` | enabled | `kernel.perf_event_paranoid=4` | Prevents information disclosure; profiling needs privileges. |
| `apparmor_user_namespace_restriction` | enabled | `kernel.apparmor_restrict_unprivileged_userns=1` | Confines namespaces; applications need suitable profiles. |
| `sysrq` | enabled | `kernel.sysrq=0` | Removes a powerful interface; SysRq recovery is unavailable. |
| `disable_unprivileged_user_namespaces` | disabled | `kernel.unprivileged_userns_clone=0` | Optional stronger restriction; breaks namespace users. |
| `disable_io_uring` | disabled | `kernel.io_uring_disabled=2` | Optional syscall reduction; breaks io_uring applications. |
| `lock_kexec_loading` | disabled | `kernel.kexec_load_disabled=1` | Optional irreversible lock until reboot; affects crash workflows. |
| `lock_module_loading` | disabled | `kernel.modules_disabled=1` | Optional irreversible lock until reboot; affects drivers and updates. |

The canonical catalog in `vars/main.yml` contains the precise BSI requirement,
paragraph, concise reason, and impact for every option. Disabled controls are
not forced to an insecure value; they remain unmanaged until explicitly
enabled.

## Inventory deviations

Define only the deviation in `group_vars` or `host_vars`. The role recursively
merges it with the complete standard policy:

```yaml
bsi_ubuntu_26_kernel_overrides:
  sysrq:
    value: 176
  disable_io_uring:
    enabled: true
  lock_module_loading:
    enabled: true
```

Changing `value` retains the standard enabled state. Setting `enabled: false`
stops the policy from managing or verifying that option. Enabling the module or
`kexec` loading locks is irreversible until reboot and requires prior testing.

Build-time options are verified in `/boot/config-{{ ansible_kernel }}`. Forbidden
boot flags are checked against the running `/proc/cmdline`; this prevents a
system from reporting compliance before a corrected boot configuration is
active.

Reference: [BSI IT-Grundschutz SYS.1.3 Server unter Linux und Unix, Edition 2023](https://www.bsi.bund.de/SharedDocs/Downloads/DE/BSI/Grundschutz/IT-GS-Kompendium_Einzel_PDFs_2023/07_SYS_IT_Systeme/SYS_1_3_Server_unter_Linux_und_Unix_Edition_2023.pdf)

Ubuntu mapping references:
[security-feature overview](https://documentation.ubuntu.com/security/security-features/security-features-overview/),
[process and memory protections](https://documentation.ubuntu.com/security/security-features/process-memory/),
and [kernel protections](https://documentation.ubuntu.com/security/security-features/kernel-protections/).
