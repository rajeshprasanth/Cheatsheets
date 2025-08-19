# Networking Complete Cheat Sheet

## Table of Contents

1. [Network Interfaces](#network-interfaces)
2. [IP Addressing & Subnets](#ip-addressing--subnets)
3. [Routing](#routing)
4. [VLANs & Bonding](#vlans--bonding)
5. [Tunnels & VPN](#tunnels--vpn)
6. [Firewall & Security](#firewall--security)
7. [DNS](#dns)
8. [DHCP](#dhcp)
9. [QoS & Traffic Shaping](#qos--traffic-shaping)
10. [Network Tools & Utilities](#network-tools--utilities)
11. [Packet Analysis](#packet-analysis)
12. [SNMP Monitoring](#snmp-monitoring)
13. [TCP/IP Troubleshooting](#tcpip-troubleshooting)
14. [Homelab Use Cases](#homelab-use-cases)
15. [Performance Tuning & Optimization](#performance-tuning--optimization)
16. [Useful Commands Summary](#useful-commands-summary)

---

## Network Interfaces

```bash
# Show all interfaces
ip link show
ifconfig -a

# Bring interface up/down
sudo ip link set eth0 up
sudo ip link set eth0 down

# Show IP addresses
ip addr show
ifconfig eth0

# Change IP temporarily
sudo ip addr add 192.168.1.10/24 dev eth0
sudo ip addr del 192.168.1.10/24 dev eth0

# Show VLANs and bonding
ip -d link show
```

---

## IP Addressing & Subnets

```bash
# Show routing table
ip route show
route -n

# Default gateway
ip route | grep default

# Ping host
ping 192.168.1.1

# Test connectivity to port
nc -zv 192.168.1.1 22

# Subnet calculation (CIDR /24)
# Network: 192.168.1.0
# Broadcast: 192.168.1.255
# Usable hosts: 192.168.1.1 - 192.168.1.254
```

---

## Routing

```bash
# Add static route
sudo ip route add 10.0.0.0/24 via 192.168.1.1

# Delete route
sudo ip route del 10.0.0.0/24
```

---

## VLANs & Bonding

```bash
# Create VLAN interface
sudo ip link add link eth0 name eth0.10 type vlan id 10
sudo ip link set dev eth0.10 up
sudo ip addr add 192.168.10.1/24 dev eth0.10

# Bonding interfaces
sudo modprobe bonding
sudo ip link add bond0 type bond
sudo ip link set eth0 master bond0
sudo ip link set eth1 master bond0
sudo ip link set bond0 up
```

---

## Tunnels & VPN

```bash
# GRE Tunnel
sudo ip tunnel add gre1 mode gre remote 192.168.2.1 local 192.168.1.1 ttl 255
sudo ip link set gre1 up
sudo ip addr add 10.0.0.1/30 dev gre1

# IPIP Tunnel
sudo ip tunnel add ipip1 mode ipip remote 192.168.2.1 local 192.168.1.1
sudo ip link set ipip1 up
sudo ip addr add 10.0.1.1/30 dev ipip1

# OpenVPN client
sudo openvpn --config client.ovpn

# WireGuard
sudo wg-quick up wg0
sudo wg show
```

---

## Firewall & Security

```bash
# Iptables
sudo iptables -L -v -n
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -s 10.0.0.5 -j DROP
sudo iptables-save > /etc/iptables/rules.v4

# Firewalld
sudo firewall-cmd --state
sudo firewall-cmd --list-all
sudo firewall-cmd --add-port=8080/tcp --permanent
sudo firewall-cmd --reload
```

---

## DNS

```bash
# Query DNS
dig example.com
nslookup example.com

# Query specific type
dig example.com A
dig example.com MX
dig example.com TXT

# Authoritative server
dig @8.8.8.8 example.com

# Reverse lookup
dig -x 192.168.1.1
```

---

## DHCP

```bash
# Check lease info
cat /var/lib/dhcp/dhclient.leases

# Release and renew IP
sudo dhclient -r eth0
sudo dhclient eth0
```

---

## QoS & Traffic Shaping

```bash
# Show tc qdisc
tc qdisc show dev eth0

# Add root HTB qdisc
sudo tc qdisc add dev eth0 root handle 1: htb default 12

# Add class with rate limit
sudo tc class add dev eth0 parent 1: classid 1:12 htb rate 10mbit ceil 10mbit

# Filter traffic to class
sudo tc filter add dev eth0 protocol ip parent 1:0 prio 1 u32 match ip dst 192.168.1.100 flowid 1:12
```

---

## Network Tools & Utilities

```bash
# Check open ports
ss -tuln
netstat -tulnp

# Bandwidth test
iperf3 -s
iperf3 -c 192.168.1.1

# Trace route
traceroute 8.8.8.8
mtr 8.8.8.8

# ARP table
arp -n
ip neigh show

# Active connections
lsof -i -P -n
```

---

## Packet Analysis

```bash
# Capture traffic
sudo tcpdump -i eth0
sudo tcpdump -i eth0 port 80
sudo tcpdump -i eth0 host 192.168.1.1 -w capture.pcap

# Analyze with Wireshark
wireshark capture.pcap
```

---

## SNMP Monitoring

```bash
# Walk OID subtree
snmpwalk -v2c -c public <host> system

# CPU, RAM, Disk usage
snmpwalk -v2c -c public <host> hrProcessorLoad
snmpwalk -v2c -c public <host> hrMemorySize
snmpwalk -v2c -c public <host> hrStorageUsed

# Interface stats
snmpwalk -v2c -c public <host> ifInOctets
snmpwalk -v2c -c public <host> ifOutOctets
snmpwalk -v2c -c public <host> ifOperStatus
```

---

## TCP/IP Troubleshooting

```bash
# Ping host
ping 192.168.1.1

# Trace route
traceroute 8.8.8.8
mtr 8.8.8.8

# Check ARP table
arp -n

# Test port
nc -zv 192.168.1.1 22

# Capture TCP only
sudo tcpdump -i eth0 tcp

# Show TCP statistics
ss -s
netstat -s
```

---

## Homelab Use Cases

```bash
# Monitor multiple switches
for ip in 10.0.0.{1..5}; do
  snmpwalk -v2c -c public $ip ifOperStatus
done

# Router WAN traffic
snmpget -v2c -c public 10.0.0.1 ifInOctets.1
snmpget -v2c -c public 10.0.0.1 ifOutOctets.1

# VM connectivity check
ping 10.0.1.10

# Network interface stats
cat /proc/net/dev | awk '{print $1, $2, $10}' | tail -n +3
```

---

## Performance Tuning & Optimization

```bash
# TCP buffers
sudo sysctl -w net.core.rmem_max=16777216
sudo sysctl -w net.core.wmem_max=16777216

# TCP window scaling
sudo sysctl -w net.ipv4.tcp_window_scaling=1

# Kernel network params
sysctl -a | grep tcp

# Test bandwidth with parallel streams
iperf3 -c 192.168.1.1 -P 4

# Real-time bandwidth monitor
iftop -i eth0
nload eth0
bmon
```

---

## Useful Commands Summary

| Command          | Description                  |
| ---------------- | ---------------------------- |
| `ip addr`        | Show IP addresses            |
| `ip link`        | Show network interfaces      |
| `ip -d link`     | VLANs & bonding info         |
| `ip route`       | Show routing table           |
| `ping`           | Connectivity test            |
| `traceroute`     | Trace path                   |
| `mtr`            | Real-time path & packet loss |
| `ss -tuln`       | Show listening ports         |
| `netstat -tulnp` | Network connections          |
| `tcpdump`        | Capture packets              |
| `dig`            | DNS query                    |
| `nslookup`       | DNS lookup                   |
| `snmpwalk`       | SNMP query                   |
| `iperf3`         | Bandwidth test               |
| `arp -n`         | ARP table                    |
| `tc`             | QoS / Traffic shaping        |
| `wg`             | WireGuard VPN                |
| `openvpn`        | OpenVPN client/server        |
| `iftop`          | Real-time bandwidth          |
| `nload`          | Network usage                |
| `bmon`           | Bandwidth monitor            |

