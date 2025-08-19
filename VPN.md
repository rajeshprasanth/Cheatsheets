# VPN Cheat Sheet

## Table of Contents

1. [Basics](#basics)
2. [OpenVPN](#openvpn)
3. [WireGuard](#wireguard)
4. [Key Management](#key-management)
5. [Client Configuration](#client-configuration)
6. [Testing & Troubleshooting](#testing--troubleshooting)
7. [Homelab Use Cases](#homelab-use-cases)

---

## Basics

```text
# VPN overview
VPN (Virtual Private Network) → secure communication tunnel over the internet or LAN

# Common use
Remote access, site-to-site connectivity, homelab remote management

# Protocols
OpenVPN → SSL/TLS based, widely supported
WireGuard → Modern, lightweight, fast, uses public-key cryptography
```

---

## OpenVPN

```bash
# Install OpenVPN (Debian/Ubuntu)
sudo apt install openvpn easy-rsa

# Generate server keys & certificates
make-cadir ~/openvpn-ca
cd ~/openvpn-ca
./easyrsa init-pki
./easyrsa build-ca
./easyrsa gen-req server nopass
./easyrsa sign-req server server

# Start OpenVPN server
sudo systemctl start openvpn@server
sudo systemctl enable openvpn@server

# Client config
# Copy client.ovpn from server to client machine
scp /etc/openvpn/client/client1.ovpn user@client:/home/user/

# Test connection
sudo openvpn --config client1.ovpn
```

### Common OpenVPN Commands

```bash
# Check OpenVPN status
sudo systemctl status openvpn@server

# Restart OpenVPN
sudo systemctl restart openvpn@server

# Show active VPN connections
sudo cat /var/log/openvpn/status.log
```

---

## WireGuard

```bash
# Install WireGuard
sudo apt install wireguard

# Generate server keys
wg genkey | tee server_private.key | wg pubkey > server_public.key

# Generate client keys
wg genkey | tee client_private.key | wg pubkey > client_public.key

# Server config (/etc/wireguard/wg0.conf)
[Interface]
PrivateKey = <server_private.key>
Address = 10.0.0.1/24
ListenPort = 51820

[Peer]
PublicKey = <client_public.key>
AllowedIPs = 10.0.0.2/32

# Start WireGuard
sudo wg-quick up wg0
sudo systemctl enable wg-quick@wg0
```

### WireGuard Commands

```bash
# Show status
sudo wg show

# Bring interface up/down
sudo wg-quick up wg0
sudo wg-quick down wg0

# Check interface IPs
ip addr show wg0
```

---

## Key Management

```bash
# OpenVPN: CA, server, client certificates
# WireGuard: private/public key pairs

# Backup keys
cp *.key ~/vpn-backup/
cp *.crt ~/vpn-backup/
```

---

## Client Configuration

```text
# OpenVPN client (.ovpn)
client
dev tun
proto udp
remote <server-ip> 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
cert client.crt
key client.key
remote-cert-tls server
cipher AES-256-CBC
auth SHA256
```

```text
# WireGuard client (/etc/wireguard/wg0.conf)
[Interface]
PrivateKey = <client_private.key>
Address = 10.0.0.2/24
DNS = 10.0.0.1

[Peer]
PublicKey = <server_public.key>
Endpoint = <server-ip>:51820
AllowedIPs = 0.0.0.0/0, ::/0
PersistentKeepalive = 25
```

---

## Testing & Troubleshooting

```bash
# Ping VPN server from client
ping 10.0.0.1

# Check VPN interface
ip addr show tun0      # OpenVPN
ip addr show wg0       # WireGuard

# Check routing table
ip route

# Show VPN logs
sudo journalctl -u openvpn@server
sudo journalctl -u wg-quick@wg0
```

---

## Homelab Use Cases

```bash
# Remote access to internal homelab network
# OpenVPN or WireGuard client connects to home server
# Access 10.0.0.x network resources securely

# Site-to-site VPN
# WireGuard between two homelab locations
[Peer]
PublicKey = <remote-server-key>
AllowedIPs = 10.0.1.0/24   # remote subnet
Endpoint = <remote-ip>:51820
PersistentKeepalive = 25

# Split tunneling
AllowedIPs = 10.0.0.0/24   # only route internal network traffic via VPN
```

---

## Useful Command Summary

| Command                                   | Description                    |
| ----------------------------------------- | ------------------------------ |
| `sudo systemctl start openvpn@server`     | Start OpenVPN server           |
| `sudo systemctl status openvpn@server`    | Check OpenVPN status           |
| `sudo wg-quick up wg0`                    | Bring up WireGuard interface   |
| `sudo wg-quick down wg0`                  | Bring down WireGuard interface |
| `sudo wg show`                            | Show WireGuard status          |
| `ping 10.0.0.1`                           | Test VPN connectivity          |
| `ip addr show tun0`                       | Check OpenVPN interface        |
| `ip addr show wg0`                        | Check WireGuard interface      |
| `ip route`                                | Check routing table            |
| `scp client.ovpn user@client:/home/user/` | Copy OpenVPN client config     |


