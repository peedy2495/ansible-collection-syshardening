# Role: bsi_ubuntu

Orchestration role that selects implemented BSI policy roles using Ansible
distribution facts. Current coverage is Ubuntu 26.x and
the `bsi_ubuntu_26_kernel` and `bsi_ubuntu_26_ssh_client` policies. It first
composes the platform-independent `bsi_all` dispatcher, which currently applies
`bsi_all_firewall`.

The common firewall opens no incoming management service implicitly. Supply a
source-restricted SSH server rule through `firewall_additional_rules` before
running this dispatcher remotely.

Unsupported distributions and versions fail explicitly. Applying this role does
not by itself establish BSI IT-Grundschutz conformity because organizational,
server-side, network, identity-management, and logging requirements remain.
