# BSI firewall policy

`peedy2495.syshardening.bsi_all_firewall` applies the common firewall portion
of BSI IT-Grundschutz `NET.3.2 Firewall`, Edition 2023, through the reusable
`firewall` implementation role.

The policy enables the detected firewall and configures:

- INPUT deny with stateful return traffic and loopback allowed;
- no implicitly opened service;
- OUTPUT allow for standard clients or deny for increased protection;
- FORWARD deny;
- equivalent family-neutral IPv4 and IPv6 policy;
- logging of denied traffic and Ansible firewall configuration changes.

`bsi_all_firewall_required_rules` contains policy-owned required services and is
empty by default. Host and group inventory can append rules through the common
`firewall_additional_rules` interface:

> **Remote-access warning:** no SSH server port is opened implicitly. Define a
> narrowly scoped management rule before applying this role remotely, or the
> INPUT deny policy can terminate subsequent administrative access.

```yaml
bsi_all_firewall_required_rules:
  - name: Allow SSH from administration network
    action: allow
    direction: input
    protocol: tcp
    source: 192.0.2.0/24
    destination_port: "22"

firewall_additional_rules:
  - name: Allow HTTPS egress through organization proxy
    action: allow
    direction: output
    protocol: tcp
    destination: 198.51.100.20/32
    destination_port: "443"
```

For increased protection, set:

```yaml
bsi_all_firewall_protection_level: increased
```

OUTPUT then defaults to deny, so every required outbound connection must be
listed explicitly, including DNS, time synchronization, proxies, repositories,
and management services as applicable. The standard level defaults OUTPUT to
allow, matching the requested client baseline.

`standard` therefore represents an institutional decision that general client
egress is authorized. Where NET.3.2.A2 requires individual outbound
communication relationships to be allowlisted, use `increased` and document
the corresponding OUTPUT rules.

The role covers technical packet filtering. Rule ownership and justification,
log retention and evaluation, alerting, time synchronization, and monitoring
remain part of the institution's ISMS and operating processes.

Reference: [BSI IT-Grundschutz NET.3.2 Firewall, Edition 2023](https://www.bsi.bund.de/SharedDocs/Downloads/DE/BSI/Grundschutz/IT-GS-Kompendium_Einzel_PDFs_2023/09_NET_Netze_und_Kommunikation/NET_3_2_Firewall_Edition_2023.pdf)
