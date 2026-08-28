## Initial Security Assessment

Before implementing additional security controls, the server was reviewed to establish its initial security posture.

### Initial State

| Area | Initial State |
| --- | --- |
| Operating System | Ubuntu Server 24.04 LTS |
| Administrative Access | Root account available through SSH using public key authentication |
| SSH Password Authentication | Disabled |
| SSH Root Login | Permitted with public key authentication |
| Host Firewall | UFW installed but inactive |
| Exposed Services | SSH on TCP port 22 |
| Patch Status | 87 packages awaiting updates |
| Automatic Updates | Unattended upgrades installed and enabled |
| SSH Hardening | Mostly default OpenSSH configuration |
| Security Auditing | auditd not installed |
| Brute Force Protection | Fail2ban not installed |
| Internet Exposure | Repeated automated SSH scanning observed |

### Key Findings

The initial assessment identified several areas for improvement.

1. Direct root SSH access was still permitted using public key authentication.
2. The host firewall was inactive.
3. The system had 87 outstanding package updates, including security-related updates.
4. OpenSSH allowed unnecessary functionality such as X11 forwarding.
5. No dedicated Linux auditing system was configured.
6. No automated response existed for repeated SSH authentication attempts.
7. The public SSH service was receiving frequent automated login attempts from the Internet.

These findings were used to define the security controls implemented throughout this project.
