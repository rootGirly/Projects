# Secure Offsite Backups: Verification & Hardening

> Goal: Validate and harden the Restic + SFTP + WireGuard backup architecture.

![Computer](img/computer.jpg)
>Photo by Tianyi Ma on Unsplash



Now that the backup system is functional, I moved to the Hardening Phase. The goal is Defense in Depth: even if one layer fails, others protect the data.

### Disable SSH Password Authentication

Passwords are vulnerable to brute-force attacks. Solution: Enforce key-only access.

**On the Backup LXC:**

I Modified:

```
# /etc/ssh/sshd_config
# Disable password authentication
PasswordAuthentication no

# Ensure public key auth is enabled
PubkeyAuthentication yes

# Disable root login
PermitRootLogin no

```

**Impact:** Attackers cannot guess passwords. They must possess the private key to access the server.


`systemctl restart ssh`

**Verification: Try to log in with a password (it should fail):**

```
ssh backupuser@192.168.x.xx
# Should return: Permission denied (publickey)
```

## Restrict backupuser Shell Access  (SFTP-Only)

If an account is compromised, an attacker could execute commands. Solution: Restrict the user to SFTP only using `ForceCommand`.

On the Backup LXC:
Change the shell to nologin:

`usermod -s /usr/sbin/nologin backupuser`

Add the Match User block to /etc/ssh/sshd_config:


```
# /etc/ssh/sshd_config
Match User backupuser
    ForceCommand internal-sftp
    PermitTunnel no
    AllowAgentForwarding no
    AllowTcpForwarding no
    X11Forwarding no    
```  

Even with a valid key, the user is "jailed" into the SFTP subsystem and cannot execute arbitrary code.  



**Verify:**

```
grep backupuser /etc/passwd
# Should show: backupuser:x:1000:1000::/home/backupuser:/usr/sbin/nologin
```

## Tighten WireGuard AllowedIPs

If the VPS is compromised, an attacker could pivot to other devices on the LAN. Solution: Restrict `AllowedIPs` to only the backup server IP.

**On the VPS (/etc/wireguard/wg0.conf):**

```
[Peer]
PublicKey = <LXC_PUBLIC_KEY>
# ONLY allow the backup LXC IP (not the whole subnet)
AllowedIPs = 10.10.0.2/32, 192.168.10.85/32 (e.g)
PersistentKeepalive = 25
```

**On the WireGuard LXC (/etc/wireguard/wg0.conf):**

I ensure the AllowedIPs for the VPS peer is just the VPS IP:

```
[Peer]
PublicKey = <VPS_PUBLIC_KEY>
AllowedIPs = 10.10.0.1/32
```
**Why?** This ensures that even if the VPS is fully compromised, the attacker can only reach the backup server, not my router, NAS, or other devices.

## Implementing Fail2Ban for SSH

Brute-force attempts on the SSH port. Solution: Automatically ban IPs that exceed the retry limit.

**On the Backup LXC:**

**Install Fail2Ban and rsyslog (required for logs):**

```
apt update
apt install fail2ban rsyslog -y
systemctl enable rsyslog
systemctl start rsyslog
```
Create `/etc/fail2ban/jail.local`:

I Added this configuration:

```
[DEFAULT]
bantime = 1h
findtime = 10m
maxretry = 3

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 1h
```

**Restart Fail2Ban:**

```
systemctl enable fail2ban
systemctl restart fail2ban
```

**Verification:**

```
fail2ban-client status
# Should show: Jail list: sshd
```


## Secure Credential Storage

The Restic encryption password could be stolen if stored insecurely. **Solution:** Store the password in a file with strict permissions (600).

**On the VPS:**

```
# Create the file (if not already done)
echo "your-very-strong-password" > /root/.restic-password

# Set strict permissions
chmod 600 /root/.restic-password
chown root:root /root/.restic-password
```

Prevents other users or processes on the VPS from stealing the decryption key.

**Verification:**

```
ls -l /root/.restic-password
# Should show: -rw------- (600) owned by root
```


## Network Isolation & Firewall (UFW)

Unnecessary ports exposed to the network. Solution: Allow SSH only from trusted subnets (WireGuard and LAN).

**On the Backup LXC:**

```
ufw default deny incoming
ufw default allow outgoing

# Allow SSH from LAN (adjust subnet as needed)
ufw allow from 192.168.x.xx/24 to any port 22 proto tcp

# Allow SSH from WireGuard subnet
ufw allow from 10.10.x.x/24 to any port 22 proto tcp

ufw enable
```
The backup server is invisible to the public internet and inaccessible from unauthorized networks.

**Verification**

`ufw status verbose`


## Lessons Learned & Troubleshooting
Here are the key challenges I faced and how I solved them:

### After Hardening

**The "Account Not Available" Error:**

* **Issue:** SSH authenticated successfully but immediately disconnected with This account is currently not available.
* **Cause:** The user shell was set to /usr/sbin/nologin, but the Match User block forcing internal-sftp was missing. SSH tried to start a shell and failed.
* **Solution:** Added ForceCommand internal-sftp to the Match User block. This tells SSH to ignore the shell setting and start the SFTP server instead.


**Chroot Complexity:**


* **Issue:** Using ChrootDirectory caused permission errors because the directory must be owned by root and not writable by the user.
* **Solution:** For simplicity and stability, I opted for ForceCommand internal-sftp without Chroot. I relied on file permissions and network isolation for security. (Chroot can be added later if stricter isolation is needed).

This project demonstrates a resilient, encrypted, and isolated backup architecture. By combining WireGuard for secure tunneling, Restic for deduplicated encryption, and SSH hardening for access control, I have created a system that survives total server failure.

**Fail2Ban & Missing Logs**

* **Issue:** Fail2Ban failed to start because /var/log/auth.log did not exist on the minimal LXC.
* **Solution:** Installed and started rsyslog to ensure log files were generated before starting Fail2Ban.



The journey from **"it works"** to **"it's secure"** required careful attention to detail, but the result is more than just a backup solution it is a **peace of mind**. I took this project very seriously because I know that a single misconfiguration or vulnerability could compromise my entire home network. In cybersecurity, the stakes are real: one weakness can let an attacker in, and one failure can mean losing everything.

**Be your own guru.**






