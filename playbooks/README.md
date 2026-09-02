# Collection playbooks

The collection provides three deployment entry points:

- `peedy2495.syshardening.bsi_all` applies platform-independent BSI policies;
- `peedy2495.syshardening.bsi_ubuntu` additionally selects the supported Ubuntu
  version policies;
- `peedy2495.syshardening.ubuntu_26_kernel_hardening` combines the BSI Ubuntu
  26 kernel policy with CIS Ubuntu 26 network and additional kernel controls.

After installing or building the collection, run an entry point by its fully
qualified collection name:

```bash
ansible-playbook -i inventory peedy2495.syshardening.bsi_ubuntu
```

For the complementary kernel policies:

```bash
ansible-playbook -i inventory \
  peedy2495.syshardening.ubuntu_26_kernel_hardening
```

## Firewall examples

The files below `playbooks/examples/` demonstrate the BSI firewall policy for
standard and increased protection. Copy an example into the deployment
repository and adapt it there. They intentionally require organization-owned
endpoint variables and fail before enabling deny policies when those values are
missing.

For the standard example, inventory must define:

```yaml
bsi_example_management_network: 192.0.2.0/24
```

For the increased-protection example, inventory must additionally define:

```yaml
bsi_example_dns_resolver: 192.0.2.53/32
bsi_example_ntp_server: 192.0.2.123/32
bsi_example_https_proxy: 192.0.2.10/32
```

The addresses are documentation examples and MUST be replaced. Define further
host- or group-specific rules with the documented `firewall_additional_rules`
interface. Do not redefine that variable in these playbooks, because play-level
variable precedence would prevent inventory from extending it as intended.

Review console access and all required inbound and outbound communication paths
before rollout. Apply firewall changes serially to a test group first.
