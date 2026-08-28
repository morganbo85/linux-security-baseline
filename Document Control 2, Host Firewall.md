### Control 2: Host Firewall

#### Requirement

The server should deny unsolicited inbound network traffic by default and explicitly permit only services required for its intended purpose.

At this stage of the project, SSH is the only service that requires inbound access.

#### Implementation

UFW was enabled with the following policy:

```
Default incoming: deny
Default outgoing: allow
Default routed: disabled
Logging: enabled
```

Inbound SSH was explicitly permitted:

```
22/tcp (OpenSSH)    ALLOW IN    Anywhere
22/tcp (OpenSSH)    ALLOW IN    Anywhere (v6)
```

This implements a default-deny approach for inbound connections while maintaining normal outbound connectivity.

#### Verification

Firewall status and policy were verified with:

```
sudo ufw status verbose
sudo ufw status numbered
```

Listening network services were independently checked with:

```
sudo ss -tulpn
```

The service inspection confirmed that SSH was listening on TCP port 22 and no additional externally listening application services were present.

A new SSH connection was successfully established after enabling UFW to verify that legitimate administrative access remained available.

#### Security Outcome

Unsolicited inbound traffic is denied by default. SSH is the only explicitly permitted inbound service, reducing the network attack surface of the host.
