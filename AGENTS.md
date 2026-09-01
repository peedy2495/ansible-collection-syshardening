# Ansible Collection Architecture

This repository contains reusable Ansible roles for system configuration,
security hardening, compliance, and policy enforcement.

The architecture defined here applies to all current and future
implementations, independently of:

- operating system
- distribution
- distribution version
- security framework
- compliance standard
- technical component
- vendor
- environment

## Core architecture

Separate technical implementation from policy and orchestration.

There are three conceptual role types:

1. implementation roles
2. policy roles
3. orchestration roles

## Implementation roles

Implementation roles configure exactly one technical component or capability.

Examples:

- ssh_client
- ssh_server
- auditd
- sysctl
- firewall
- pam
- logging

Implementation roles MUST:

- be reusable
- be independent of a specific compliance framework
- expose configuration through variables
- contain the technical implementation
- validate input where appropriate
- be idempotent

Implementation roles MUST NOT:

- contain CIS-specific policy
- contain BSI-specific policy
- contain STIG-specific policy
- contain organization-specific policy
- assume a particular compliance framework

A repeated execution with identical input MUST result in no changes.

## Policy roles

Policy roles define desired configuration according to a policy, standard,
baseline, platform, operating system, or version.

Policy roles SHOULD reuse implementation roles rather than duplicate their
technical implementation.

Generic naming pattern:

<policy>_<platform>_<major_version>_<component>

Examples:

cis_ubuntu_26_ssh_client
bsi_debian_13_ssh_server
stig_rhel_10_auditd
company_linux_ssh_client

A policy role SHOULD primarily contain:

- policy values
- policy-specific variables
- policy-specific conditions
- policy metadata
- calls to implementation roles

Example:

cis_ubuntu_26_ssh_client
    -> ssh_client

The policy role defines the desired policy.
The implementation role performs the technical configuration.

## Orchestration roles

Orchestration roles select and combine policy or implementation roles.

Examples:

cis_ubuntu
cis_linux
company_baseline
linux_hardening

An orchestration role MAY dynamically determine the applicable implementation
based on facts such as:

- ansible_distribution
- ansible_distribution_major_version
- ansible_os_family
- ansible_architecture

Orchestration roles MUST NOT duplicate technical implementation.

## Idempotency

All roles MUST be idempotent.

Running the same role multiple times with the same:

- host state
- variables
- facts
- inventory

MUST converge to the same target state.

After convergence, another execution MUST report no changes.

Prefer declarative Ansible modules over shell or command execution.

Avoid:

- shell
- command
- raw

when an idempotent Ansible module exists.

If command execution is unavoidable, explicit idempotency controls MUST be
implemented, for example:

- creates
- removes
- changed_when
- failed_when
- precondition checks

Handlers MUST only run when their notifying task actually changes state.

## Role boundaries

A role SHOULD have one clearly defined responsibility.

Prefer:

ssh_client
ssh_server
auditd
sysctl

instead of large monolithic roles containing unrelated functionality.

Roles MUST communicate through documented variables and role interfaces,
not through undocumented assumptions about internal files or task structure.

## Naming

Role names MUST:

- use lowercase characters
- use underscores
- be descriptive
- avoid hyphens

Generic implementation:

<component>

Examples:

ssh_client
auditd
firewall

Policy implementation:

<policy>_<platform>_<major_version>_<component>

Examples:

cis_ubuntu_26_ssh_client
stig_rhel_10_ssh_server

## General design principles

1. Technical implementation and policy MUST remain separated.
2. Implementation roles MUST be reusable.
3. Policy roles SHOULD compose implementation roles.
4. Orchestration roles SHOULD select appropriate policy roles.
5. All roles MUST be idempotent.
6. Duplicate technical implementation MUST be avoided.
7. Fully qualified Ansible collection names MUST be used.
8. Prefer ansible.builtin modules where applicable.
9. Every task MUST have a meaningful name.
10. Variables forming a public role interface MUST be documented.
11. Defaults SHOULD be safe and predictable.
12. OS- or version-specific logic SHOULD be isolated.
13. New frameworks and platforms MUST fit the same architecture without
    requiring structural redesign.