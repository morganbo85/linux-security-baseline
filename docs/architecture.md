# Security Architecture

This document describes how the major security controls in the Linux Security Baseline work together.

## High-Level Architecture

```text
                         Internet
                            |
                            v
                     +-------------+
                     |     UFW     |
                     | Default Deny|
                     +-------------+
                            |
                      TCP 22 allowed
                            |
                            v
                     +-------------+
                     |   OpenSSH   |
                     | Key Auth    |
                     | No Root SSH |
                     +-------------+
                       |         |
                       |         +----------------+
                       v                          v
                +-------------+           +-------------+
                |   journald  |           | Named Admin |
                |  SSH Logs   |           |   Account   |
                +-------------+           +-------------+
                       |                          |
                       v                          v
                +-------------+               sudo
                |  Fail2ban   |                  |
                | SSH Filter  |                  v
                +-------------+           Privileged Action
                       |                          |
                repeated failures                |
                       |                          v
                       v                    +-----------+
                +-------------+             |  auditd   |
                |  nftables   |             | Audit Log |
                | Blocked IPs |             +-----------+
                +-------------+
```

## Control Relationships

### Network Layer

UFW provides the primary host firewall policy.

Inbound connections are denied by default, with SSH explicitly permitted on TCP port 22.

### Authentication Layer

OpenSSH provides remote administrative access.

Authentication requires SSH public keys. Direct root login and password authentication are disabled.

Administrators connect using named accounts and use `sudo` when elevated privileges are required.

### Detection and Response

SSH authentication activity is recorded in the systemd journal.

Fail2ban monitors these events for repeated authentication failures. When a source exceeds the configured threshold, Fail2ban adds the source address to an nftables set.

Traffic from addresses in that set is rejected before another SSH authentication attempt can occur.

### Administrative Auditing

auditd provides additional visibility into security-sensitive administrative changes.

The current baseline monitors changes involving:

- Local users and groups
- Password and group credential files
- OpenSSH configuration

Audit records preserve the authenticated user's audit identity even when an operation is executed with elevated root privileges through `sudo`.

### Patch Management

APT and unattended-upgrades provide operating system package maintenance.

Package metadata and approved updates are processed automatically, while system reboots remain administrator controlled.

## Defense in Depth

No individual control is treated as sufficient protection.

For example, SSH security uses several independent layers:

```text
UFW
 |
 +-- Limits exposed network services
 |
OpenSSH
 |
 +-- Requires public key authentication
 +-- Prevents direct root login
 +-- Restricts unnecessary SSH functionality
 |
Fail2ban
 |
 +-- Detects repeated authentication failures
 |
nftables
 |
 +-- Enforces temporary source bans
 |
auditd
 |
 +-- Records security-sensitive administrative changes
```

The goal is to combine preventive, detective, and responsive controls rather than relying on a single security mechanism.
