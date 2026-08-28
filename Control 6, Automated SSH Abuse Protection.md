### Control 6: Automated SSH Abuse Protection

#### Requirement

The server should automatically detect and temporarily block sources generating repeated failed SSH authentication attempts.

This control supplements SSH public key authentication and the host firewall. It is intended to reduce repeated automated scanning, abusive connection attempts, and unnecessary authentication log activity.

#### Initial Finding

Because SSH is exposed to the Internet, automated authentication attempts began appearing almost immediately.

During one observation period, 153 failed or invalid SSH authentication attempts were recorded within approximately one hour.

Several individual source addresses generated repeated attempts, including one source responsible for more than 80 events.

No automated mechanism existed to respond to this behavior.

#### Implementation

Fail2ban was installed and configured to monitor OpenSSH authentication activity through the systemd journal.

The SSH jail uses the following policy:

```
Maximum failures: 5
Detection window:  10 minutes
Ban duration:      1 hour
```

A local configuration override was created rather than modifying Fail2ban's packaged `jail.conf`:

```
/etc/fail2ban/jail.d/security-baseline.local
```

Configuration:

```
[sshd]
enabled = true
maxretry = 5
findtime = 10m
bantime = 1h
```

The configuration was validated before use:

```
sudo fail2ban-client -t
```

The effective settings were then verified:

```
sudo fail2ban-client get sshd maxretry
sudo fail2ban-client get sshd findtime
sudo fail2ban-client get sshd bantime
```

Result:

```
5
600
3600
```

#### Detection Verification

After Fail2ban was enabled, repeated Internet SSH attempts triggered automatic bans.

Jail status was inspected with:

```
sudo fail2ban-client status sshd
```

Multiple abusive source addresses were automatically identified and banned without generating artificial authentication attacks for the test.

#### Enforcement Verification

Fail2ban reported that addresses were banned, but the control was also verified at the firewall enforcement layer rather than relying only on application status.

Initial inspection of iptables did not reveal Fail2ban rules. The effective Fail2ban configuration was therefore examined:

```
sudo fail2ban-client get sshd actions
sudo fail2ban-client -d
```

This showed that the SSH jail was using a native nftables action rather than an iptables action.

Fail2ban created:

```
table: f2b-table
chain: f2b-chain
set:   addr-set-sshd
```

The active nftables configuration was inspected directly:

```
sudo nft list table inet f2b-table
```

The resulting rule included:

```
tcp dport 22 ip saddr @addr-set-sshd reject
```

A currently banned source address was also present in `addr-set-sshd`, confirming that Fail2ban was inserting abusive sources into the nftables enforcement set.

#### Security Outcome

Repeated SSH authentication failures are automatically detected and offending source addresses are temporarily blocked for one hour.

The complete response path was verified:

```
SSH authentication failure
        |
        v
systemd journal
        |
        v
Fail2ban SSH filter
        |
        v
Source exceeds threshold
        |
        v
IP added to nftables set
        |
        v
Subsequent SSH traffic rejected
```

Verification at the nftables layer confirmed that a Fail2ban "banned" status represented an actual firewall enforcement action rather than only an application-level state.
