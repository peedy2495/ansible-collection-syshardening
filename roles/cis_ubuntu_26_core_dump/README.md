# CIS Ubuntu 26 core dump policy

This policy owns the CIS core-file size limit and delegates its implementation
to the generic `core_dump` role.

```yaml
cis_ubuntu_26_core_dump_overrides:
  core_file_size:
    enabled: false
```

Public variables:

- `cis_ubuntu_26_core_dump_overrides`: per-control `enabled` deviations;
- `cis_ubuntu_26_core_dump_limits_file`: managed limits drop-in.
