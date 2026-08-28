### Control 5: Security Auditing

#### Requirement

Security-relevant administrative activity should be recorded in a way that allows an administrator to determine what changed, when it changed, and which authenticated user initiated the action.

Particular attention should be given to changes affecting local identities and remote administrative access.

#### Initial Finding

The Linux audit subsystem was not configured.

After `auditd` was installed, the audit subsystem was active but contained no custom audit rules:

```text
enabled 1
failure 1
lost 0
```

```text
No rules
```

Standard SSH authentication logging was already available through the system journal, but dedicated audit rules were needed to provide additional visibility into security-sensitive configuration changes.

#### Implementation

`auditd` and its supporting plugins were installed.

A security baseline rules file was created:

```text
/etc/audit/rules.d/security-baseline.rules
```

Local identity files were monitored for writes and attribute changes:

```text
-w /etc/passwd -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/gshadow -p wa -k identity
```

OpenSSH configuration was also monitored:

```text
-w /etc/ssh/sshd_config -p wa -k ssh_config
-w /etc/ssh/sshd_config.d/ -p wa -k ssh_config
```

The `identity` and `ssh_config` keys provide searchable labels for related audit events.

Rules were loaded using:

```bash
sudo augenrules --load
```

#### Verification

Loaded audit rules were verified with:

```bash
sudo auditctl -l
```

A temporary account was then intentionally created to generate a controlled audit event:

```bash
sudo useradd audit-test
```

The resulting events were queried using:

```bash
sudo ausearch -k identity -ts recent -i
```

The audit trail identified:

```text
auid=bo-admin
uid=root
comm=useradd
exe=/usr/sbin/useradd
key=identity
```

The temporary account was subsequently removed:

```bash
sudo userdel audit-test
```

The resulting audit records identified:

```text
auid=bo-admin
uid=root
comm=userdel
exe=/usr/sbin/userdel
key=identity
```

This demonstrated an important distinction between the effective user and the audit user.

`uid=root` showed that the operation executed with elevated privileges through `sudo`.

`auid=bo-admin` preserved the identity of the administrator whose authenticated session initiated the privileged operation.

#### Security Outcome

Changes to local identity files and OpenSSH configuration are now monitored by the Linux audit subsystem.

Privileged account-management activity can be traced back to the authenticated administrator who initiated it, providing stronger accountability and useful evidence for security investigations.
