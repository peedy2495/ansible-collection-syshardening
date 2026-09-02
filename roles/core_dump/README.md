# Core dump implementation role

`peedy2495.syshardening.core_dump` manages limits(5) and systemd-coredump
drop-ins without embedding a compliance policy. Both parts default to
`unchanged`.

```yaml
core_dump_limits_state: present
core_dump_limits:
  - domain: "*"
    type: hard
    item: core
    value: 0

core_dump_systemd_state: present
core_dump_systemd_settings:
  Storage: none
  ProcessSizeMax: 0
```

Use `absent` to remove the respective role-owned drop-in. Changes affect new
login sessions and future core dumps; existing sessions are not modified.
