### Control 4: SSH Hardening

#### Requirement

OpenSSH should expose only the functionality required for remote administration while limiting unnecessary features and reducing opportunities for authentication abuse.

Security settings should be selected based on the server's intended use rather than blindly disabling all optional SSH functionality.

#### Initial Finding

The effective OpenSSH configuration included:

```
MaxAuthTries 6
ClientAliveInterval 0
ClientAliveCountMax 3
X11Forwarding yes
AllowTcpForwarding yes
```

X11 forwarding was unnecessary because the system is a headless server.

The default authentication attempt limit was also higher than required for an environment using public key authentication.

#### Implementation

A dedicated OpenSSH configuration file was created:

```
/etc/ssh/sshd_config.d/99-security-baseline.conf
```

The following settings were applied:

```
X11Forwarding no
MaxAuthTries 3

ClientAliveInterval 300
ClientAliveCountMax 2
```

The resulting effective configuration included:

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
PermitEmptyPasswords no

MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
X11Forwarding no
AllowTcpForwarding yes
```

TCP forwarding was intentionally retained because SSH tunneling can be useful for legitimate infrastructure administration. No project requirement justified disabling it.

#### Verification

Before reloading OpenSSH, configuration syntax was validated:

```
sudo sshd -t
```

The SSH service was then reloaded:

```
sudo systemctl reload ssh
```

Effective settings were verified with:

```
sudo sshd -T | grep -E \
'x11forwarding|maxauthtries|clientaliveinterval|clientalivecountmax'
```

The resulting configuration was:

```
maxauthtries 3
clientaliveinterval 300
clientalivecountmax 2
x11forwarding no
```

A new SSH session was successfully established after the configuration changes to confirm that legitimate remote administration remained functional.

#### Security Outcome

OpenSSH exposes fewer unnecessary capabilities, permits fewer authentication attempts per connection, and detects unresponsive SSH clients while preserving functionality useful for legitimate system administration.

`ClientAliveInterval` and `ClientAliveCountMax` are not treated as a true user inactivity timeout. They detect unresponsive clients rather than measuring keyboard or user activity.
