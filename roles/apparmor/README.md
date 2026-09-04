# AppArmor implementation role

`peedy2495.syshardening.apparmor` manages the AppArmor service, enables profiles
represented by symlinks in a configurable disabled-profile directory, and
verifies kernel and profile state. It can install explicitly supplied packages,
but contains no distribution-specific package names, CIS policy, or BSI policy.

```yaml
apparmor_packages:
  - apparmor
  - apparmor-utils
apparmor_service_state: started
apparmor_service_enablement: enabled
apparmor_verify_kernel_enabled: true
apparmor_enable_disabled_profiles: true
apparmor_require_enforce_mode: false
```

`apparmor_packages` defaults to an empty list, so direct callers must supply
the package names appropriate for their platform. All defaults are no-op
values. `apparmor_require_enforce_mode` verifies that
`aa-status` reports no profiles in complain mode; it does not silently rewrite
organization-owned profile modes.
