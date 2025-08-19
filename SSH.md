# **SSH Cheat Sheet**

## **Table of Contents**

1. [Overview](#overview)
2. [Basic Commands](#basic-commands)
3. [Key Management](#key-management)
4. [SSH Config File](#ssh-config-file)
5. [Port Forwarding & Tunneling](#port-forwarding--tunneling)
6. [Agent & Forwarding](#agent--forwarding)
7. [Advanced Options](#advanced-options)
8. [Multi-hop / ProxyJump](#multi-hop--proxyjump)
9. [SSHFS / File Mounting](#sshfs--file-mounting)
10. [Security Best Practices](#security-best-practices)
11. [Troubleshooting](#troubleshooting)
12. [Automation / Scripts](#automation--scripts)

---

## **Overview**

SSH (Secure Shell) is used for:

* Secure remote login
* Encrypted file transfers (SCP/SFTP)
* Port forwarding & tunneling
* Agent forwarding for key reuse
* Automation scripts for DevOps / homelab management

It ensures **confidentiality, integrity, and authentication** across untrusted networks.

---

## **Basic Commands**

```bash
# Connect to remote server
ssh john@192.168.1.10

# Connect on a custom port
ssh -p 2222 john@192.168.1.10

# Run a single remote command
ssh john@192.168.1.10 'uptime && df -h'

# Copy file to remote host
scp file.txt john@192.168.1.10:/home/john/

# Copy file from remote host
scp john@192.168.1.10:/var/log/syslog ./syslog_backup

# Copy directories recursively
scp -r ./project john@192.168.1.10:/home/john/project/
```

---

## **Key Management**

```bash
# Generate RSA key (4096-bit)
ssh-keygen -t rsa -b 4096 -C "john.doe@usa.com"

# Generate ed25519 key (modern preferred)
ssh-keygen -t ed25519 -C "john.doe@usa.com"

# Specify key file and passphrase
ssh-keygen -f ~/.ssh/id_ed25519_custom -N "mysecurepass"

# Copy public key to remote host for passwordless login
ssh-copy-id -i ~/.ssh/id_ed25519 john@192.168.1.10

# List keys in ssh-agent
ssh-add -l

# Add key to agent
ssh-add ~/.ssh/id_ed25519

# Remove key from agent
ssh-add -d ~/.ssh/id_ed25519
```

---

## **SSH Config File**

`~/.ssh/config` simplifies connections and supports advanced options:

```ini
# ~/.ssh/config
Host prod-server
    HostName 192.168.1.20
    User john
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    ForwardAgent yes
    ServerAliveInterval 60
    ServerAliveCountMax 3

Host dev-server
    HostName dev.internal.local
    User john
    Port 22
    IdentityFile ~/.ssh/id_rsa
```

**Notes:**

* `Host` → Alias for the SSH target
* `IdentityFile` → Private key to use
* `ForwardAgent` → Forward local keys for jump hosts
* `ServerAliveInterval` → Keep connection alive
* `ServerAliveCountMax` → Maximum retries before disconnect

---

## **Port Forwarding & Tunneling**

```bash
# Local port forwarding (access remote service locally)
ssh -L 8080:internalhost:80 john@192.168.1.10
# Access http://localhost:8080 → internalhost:80

# Remote port forwarding (make local service available remotely)
ssh -R 9090:localhost:3000 john@192.168.1.10
# Remote host can access local port 3000 via 192.168.1.10:9090

# Dynamic SOCKS5 proxy (acts like VPN)
ssh -D 1080 john@192.168.1.10
# Configure browser to use SOCKS5 proxy at localhost:1080
```

---

## **Agent & Forwarding**

```bash
# Start ssh-agent
eval "$(ssh-agent -s)"

# Add key to agent
ssh-add ~/.ssh/id_ed25519

# Forward agent to remote server
ssh -A john@192.168.1.10
```

---

## **Advanced Options**

```bash
# Enable compression
ssh -C john@192.168.1.10

# Verbose/debug mode
ssh -v john@192.168.1.10
ssh -vv john@192.168.1.10
ssh -vvv john@192.168.1.10

# Restrict command execution in authorized_keys
command="/usr/local/bin/backup.sh" ssh-ed25519 AAAA...

# Use specific cipher
ssh -c aes256-gcm@openssh.com john@192.168.1.10
```

---

## **Multi-hop / ProxyJump**

```bash
# Single jump host / bastion
ssh -J jumpuser@jumphost john@internal.host

# Multiple jumps
ssh -J jump1@host1,jump2@host2 john@internal.host

# Using config file
Host internal
    HostName internal.host
    User john
    ProxyJump jumpuser@jumphost
```

---

## **SSHFS / File Mounting**

```bash
# Mount remote directory locally
sshfs john@192.168.1.10:/remote/path /local/mountpoint

# Unmount
fusermount -u /local/mountpoint

# Mount with key authentication
sshfs -o IdentityFile=~/.ssh/id_ed25519 john@192.168.1.10:/remote /mnt/remote
```

---

## **Security Best Practices**

* Use **ed25519** keys (preferred over RSA)
* Disable password authentication: `PasswordAuthentication no` in `/etc/ssh/sshd_config`
* Disable root login: `PermitRootLogin no`
* Limit access: `AllowUsers john admin`
* Enable fail2ban to prevent brute-force attacks
* Rotate keys regularly and remove unused ones
* Consider two-factor authentication (2FA) for sensitive servers

---

## **Troubleshooting**

```bash
# Debug connection
ssh -vvv john@192.168.1.10

# Verify which key is being used
ssh -v john@192.168.1.10

# Test port forwarding
ssh -L 8080:localhost:80 john@192.168.1.10

# Check SSH server config
sshd -T

# Check ssh-agent status
ssh-add -l
```

---

## **Automation / Scripts**

```bash
# Run multiple commands on remote host
ssh john@192.168.1.10 <<'EOF'
uptime
df -h
free -m
exit
EOF

# Batch key deployment
for host in host1 host2 host3; do
    ssh-copy-id john@$host
done

# Integrate with Ansible
ansible all -m ping -u john
```

