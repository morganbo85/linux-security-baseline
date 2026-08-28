## Security Controls

### Control 1: Administrative Access

#### Requirement

Administrative access should use individual named accounts rather than direct root login. Remote authentication should require SSH public keys, while privileged operations should be performed through `sudo`.

This provides better accountability and reduces the risk associated with directly exposing the root account for remote administration.

#### Implementation

A dedicated administrative account was created and granted `sudo` privileges.

OpenSSH was configured to:

- Disable direct root login
- Disable password authentication
- Require public key authentication
- Prohibit empty passwords

Effective SSH configuration:

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
PermitEmptyPasswords no

Administrative workflow:

SSH public key
      |
      v
Named administrator account
      |
      v
sudo
      |
      v
Privileged operation
Verification

The effective OpenSSH configuration was verified with:

sudo sshd -T | grep -E \
'permitrootlogin|passwordauthentication|pubkeyauthentication|permitemptypasswords'

The administrative account was also verified to have functional privilege elevation:

whoami
sudo whoami

Expected result:

bo-admin
root

A new SSH session was successfully established after the configuration changes to confirm that remote administrative access remained functional.

Security Outcome

Direct remote root authentication is prohibited. Administrators authenticate using individual accounts and SSH keys, with privileged actions performed through sudo.
