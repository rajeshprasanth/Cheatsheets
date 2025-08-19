# Firewall Cheat Sheet

## Table of Contents

1. [Basics](#basics)
2. [Iptables Commands](#iptables-commands)
3. [Common Iptables Rules](#common-iptables-rules)
4. [Nftables Commands](#nftables-commands)
5. [Firewalld Commands](#firewalld-commands)
6. [UFW Commands](#ufw-commands)
7. [Firewall Logging & Troubleshooting](#firewall-logging--troubleshooting)
8. [Homelab Use Cases](#homelab-use-cases)

---

## Basics

```bash
# Show current firewall rules (iptables)
sudo iptables -L -v -n

# Default policies
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Flush all rules
sudo iptables -F
sudo iptables -X
sudo iptables -Z
```

---

## Iptables Commands

```bash
# List rules
sudo iptables -L -v -n
sudo iptables -S

# Add rules
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p icmp -j ACCEPT

# Insert rule at specific position
sudo iptables -I INPUT 1 -s 10.0.0.5 -j DROP

# Delete rule
sudo iptables -D INPUT -p tcp --dport 22 -j ACCEPT

# Save rules (Debian/Ubuntu)
sudo iptables-save > /etc/iptables/rules.v4

# Restore rules
sudo iptables-restore < /etc/iptables/rules.v4
```

---

## Common Iptables Rules

```bash
# Allow SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow DNS
sudo iptables -A INPUT -p udp --dport 53 -j ACCEPT

# Allow Ping
sudo iptables -A INPUT -p icmp -j ACCEPT

# Block IP
sudo iptables -A INPUT -s 192.168.1.100 -j DROP

# Limit SSH connections (Brute force protection)
sudo iptables -A INPUT -p tcp --dport 22 -m connlimit --connlimit-above 3 -j REJECT
```

---

## Nftables Commands

```bash
# List rules
sudo nft list ruleset

# Add table
sudo nft add table inet filter

# Add chain
sudo nft add chain inet filter input { type filter hook input priority 0 \; }

# Add rules
sudo nft add rule inet filter input tcp dport 22 accept
sudo nft add rule inet filter input ip saddr 192.168.1.0/24 drop

# Delete rules
sudo nft delete rule inet filter input handle <handle-number>

# Flush table
sudo nft flush table inet filter
```

---

## Firewalld Commands

```bash
# Check firewall status
sudo firewall-cmd --state

# List zones and rules
sudo firewall-cmd --list-all
sudo firewall-cmd --get-zones

# Add permanent rule
sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent
sudo firewall-cmd --reload

# Remove rule
sudo firewall-cmd --zone=public --remove-port=8080/tcp --permanent

# Allow service
sudo firewall-cmd --zone=public --add-service=http --permanent

# Remove service
sudo firewall-cmd --zone=public --remove-service=http --permanent
```

---

## UFW Commands (Ubuntu / Debian)

```bash
# Enable / disable firewall
sudo ufw enable
sudo ufw disable

# Check status
sudo ufw status verbose

# Allow traffic
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Deny traffic
sudo ufw deny from 192.168.1.100

# Delete rule
sudo ufw delete allow 22/tcp

# Reset firewall
sudo ufw reset
```

---

## Firewall Logging & Troubleshooting

```bash
# View dropped packets (iptables)
sudo dmesg | grep IPTABLES
sudo tail -f /var/log/kern.log

# Track connections
sudo iptables -L INPUT -v -n

# Test firewall rules
nc -zv 192.168.1.1 22
ping 192.168.1.1

# Check nftables logs
sudo journalctl -f | grep nft
```

---

## Homelab Use Cases

```bash
# Allow internal network only
sudo iptables -A INPUT -s 10.0.0.0/24 -j ACCEPT
sudo iptables -A INPUT -j DROP

# Allow access to web server from specific IPs
sudo iptables -A INPUT -p tcp --dport 80 -s 192.168.1.50 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -s 192.168.1.50 -j ACCEPT

# Rate-limit ICMP to prevent ping flood
sudo iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 5/second -j ACCEPT

# Protect SSH from brute force
sudo iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
sudo iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 3 -j DROP
```

---

## Useful Command Summary

| Command                   | Description             |
| ------------------------- | ----------------------- |
| `iptables -L -v -n`       | List all rules          |
| `iptables -A INPUT ...`   | Append rule             |
| `iptables -I INPUT ...`   | Insert rule at top      |
| `iptables -D INPUT ...`   | Delete rule             |
| `iptables-save`           | Save rules              |
| `iptables-restore`        | Restore rules           |
| `nft list ruleset`        | List nftables rules     |
| `nft add rule ...`        | Add nftables rule       |
| `firewall-cmd --add-port` | Add permanent port rule |
| `firewall-cmd --reload`   | Reload firewalld rules  |
| `ufw allow`               | Allow traffic in UFW    |
| `ufw deny`                | Deny traffic in UFW     |
| `ufw status`              | Check UFW status        |

