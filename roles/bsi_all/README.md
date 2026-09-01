# Common BSI dispatcher

`peedy2495.syshardening.bsi_all` dispatches BSI policy roles that apply to all
supported platforms. It currently includes
`peedy2495.syshardening.bsi_all_firewall`.

```yaml
- name: Apply common BSI policies
  hosts: all
  become: true
  gather_facts: true
  roles:
    - role: peedy2495.syshardening.bsi_all
```

Distribution- and version-specific dispatchers remain separate and may compose
this role with their platform-specific policies.
