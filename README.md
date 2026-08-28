# Linux Security Baseline

A hands-on project documenting the deployment and hardening of an internet-facing Ubuntu 24.04 LTS server.

The goal of this project is to build a practical Linux security baseline while focusing on why each security control exists, how it is implemented, and how its effectiveness can be verified.

Rather than applying a prebuilt hardening script or blindly following a checklist, each control is evaluated, implemented, tested, and documented individually.

## Environment

- Ubuntu Server 24.04 LTS
- Cloud-hosted virtual machine
- Public IPv4 and IPv6 connectivity
- OpenSSH for remote administration
- UFW host firewall
- auditd for security auditing
- Fail2ban for automated SSH abuse protection
- nftables for Fail2ban enforcement
