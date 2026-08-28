### Control 3: Patch Management

#### Requirement

The server should regularly retrieve package updates and automatically install approved updates to reduce exposure to known vulnerabilities.

Updates requiring a system reboot should use a controlled administrator-initiated reboot rather than automatically restarting the server.

#### Initial Finding

During the initial assessment, the server had 87 packages awaiting upgrades.

The pending updates included security-relevant components such as:

- OpenSSL
- PAM
- systemd
- curl
- rsyslog
- Linux kernel packages

The installed kernel was:

```text
6.8.0-134-generic
```

#### Implementation

The system was brought fully up to date using Ubuntu's APT package management system.

Ubuntu's automatic update configuration was verified:

```text
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

The configured unattended upgrade origins included Ubuntu security updates.

Automatic rebooting was intentionally not enabled. Reboots required by kernel or other system updates are performed as controlled administrative actions to reduce the risk of unexpected service interruption.

After patching, the server was rebooted to load the updated kernel.

#### Verification

Outstanding updates were checked with:

```
apt list --upgradable
```

After patching, no packages were reported as awaiting upgrades.

The running kernel was verified with:

```
uname -r
```

Result:

```
6.8.0-138-generic
```

Automatic update scheduling was verified with:

```
systemctl list-timers --all | grep apt
```

Both of the expected APT timers were active:

```
apt-daily.timer
apt-daily-upgrade.timer
```

The unattended upgrades service was also verified:

```
systemctl status unattended-upgrades --no-pager
```

#### Security Outcome

The server was brought from 87 outstanding package upgrades to a fully patched state.

Package metadata refreshes and unattended upgrades are automatically scheduled, while system reboots remain administrator controlled to balance security patching with system availability.
