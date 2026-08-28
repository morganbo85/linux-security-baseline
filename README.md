# Linux Security Baseline

A hands-on Linux security project demonstrating the assessment, hardening, auditing, and monitoring of an internet-facing Ubuntu Server.

The project began with a minimally configured Ubuntu 24.04 LTS cloud server. I assessed its initial security posture, identified weaknesses, implemented security controls individually, and verified that each control worked as intended.

The goal was not to run a prebuilt hardening script. Each control was implemented manually so I could understand the underlying Linux security mechanisms and document how they interact.

## What This Project Demonstrates

- Linux server administration
- Command-line troubleshooting
- SSH security and public key authentication
- Host firewall configuration
- Linux patch management
- Security auditing with auditd
- Log analysis with journald
- Automated abuse detection with Fail2ban
- nftables firewall enforcement
- Security control verification
- Technical documentation

## Environment

| Component | Technology |
| --- | --- |
| Operating System | Ubuntu Server 24.04 LTS |
| Remote Administration | OpenSSH |
| Host Firewall | UFW |
| Firewall Enforcement | nftables |
| Security Auditing | auditd |
| Log Management | systemd journal |
| Abuse Detection | Fail2ban |
| Package Management | APT / unattended-upgrades |

## Security Controls

The baseline currently implements six primary controls.

| Control | Purpose | Documentation |
| --- | --- | --- |
| Administrative Access | Named administrator accounts, sudo, SSH keys, and disabled root SSH | [View Control](docs/controls/01-administrative-access.md) |
| Host Firewall | Default-deny inbound firewall policy | [View Control](docs/controls/02-host-firewall.md) |
| Patch Management | Current packages and automated security updates | [View Control](docs/controls/03-patch-management.md) |
| SSH Hardening | Reduce unnecessary SSH functionality and authentication exposure | [View Control](docs/controls/04-ssh-hardening.md) |
| Security Auditing | Monitor security-sensitive system changes | [View Control](docs/controls/05-security-auditing.md) |
| SSH Abuse Protection | Detect and block repeated SSH authentication failures | [View Control](docs/controls/06-ssh-abuse-protection.md) |

## Before and After

The initial assessment identified several areas that required improvement.

| Initial State | Hardened State |
| --- | --- |
| Root SSH permitted with public keys | Direct root SSH disabled |
| UFW inactive | Default-deny inbound firewall enabled |
| 87 package updates pending | System fully patched |
| Default SSH settings | Hardened SSH configuration |
| No dedicated audit rules | Security-sensitive files monitored with auditd |
| No automated SSH abuse response | Fail2ban automatically blocks abusive sources |

See the complete [Initial Security Assessment](docs/controls/00-initial-assessment.md).

## Architecture

The controls are designed as layers rather than independent configuration changes.

```text
Internet
   |
   v
  UFW
   |
   v
OpenSSH --------------------+
   |                        |
   v                        v
journald                Named Admin
   |                        |
   v                       sudo
Fail2ban                     |
   |                         v
   v                       auditd
nftables
   |
   v
Temporary source blocking
```

See the [Security Architecture](docs/architecture.md) for a detailed explanation.

## Configuration Examples

Reusable configuration examples from the project are included in this repository:

```text
configs/
├── audit/
│   └── security-baseline.rules
├── fail2ban/
│   └── security-baseline.local
└── ssh/
    └── 99-security-baseline.conf
```

These files are examples from the lab environment and should be reviewed before being applied to another system.

## Verification Approach

A control was not considered complete simply because a configuration file existed.

Each control was independently verified using the underlying system where possible.

Examples include:

- Inspecting effective OpenSSH configuration with `sshd -T`
- Comparing listening services with firewall policy
- Verifying the running kernel after patching
- Generating controlled account changes and locating them in auditd
- Tracing Fail2ban enforcement through to the nftables ruleset

This verification process also exposed implementation details that were not obvious from the configuration alone. For example, Fail2ban reported successful bans, but inspecting its effective configuration showed that nftables rather than iptables was providing the actual enforcement layer.

## Project Status

Version 1 of the Linux Security Baseline focuses on manually implementing and verifying core host security controls.

Future iterations may explore configuration automation, additional auditing, centralized logging, vulnerability assessment, and repeatable deployment.

## Disclaimer

This project was built in a personal lab environment for educational and portfolio purposes.

It does not contain employer infrastructure, proprietary configurations, production credentials, private IP addressing, or other company-specific information.