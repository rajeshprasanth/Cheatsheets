# Linux Networking & Disk Administration Cheat Sheet

## Network Interface Management

### Modern Network Tools (ip command)
```bash
# Display network interfaces
ip link show                    # Show all interfaces
ip link show eth0              # Show specific interface
ip -s link show                # Show with statistics

# Interface control
ip link set eth0 up            # Bring interface up
ip link set eth0 down          # Bring interface down
ip link set eth0 mtu 9000      # Set MTU size

# IP address management
ip addr show                   # Show all IP addresses
ip addr show eth0              # Show specific interface IPs
ip addr add 192.168.1.100/24 dev eth0    # Add IP address
ip addr del 192.168.1.100/24 dev eth0    # Remove IP address

# Multiple IPs on same interface
ip addr add 192.168.1.101/24 dev eth0:1  # Add alias
ip addr show eth0              # View all IPs on interface
```

### Legacy Network Tools (ifconfig)
```bash
# Interface information
ifconfig                       # Show all active interfaces
ifconfig -a                    # Show all interfaces (active/inactive)
ifconfig eth0                  # Show specific interface

# Configure interfaces
ifconfig eth0 192.168.1.100 netmask 255.255.255.0    # Set IP and netmask
ifconfig eth0 up               # Bring interface up
ifconfig eth0 down             # Bring interface down
ifconfig eth0:1 192.168.1.101  # Create alias interface
```

### Network Configuration Files

#### Debian/Ubuntu (/etc/netplan/ - newer systems)
```yaml
# /etc/netplan/01-network-manager-all.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses: [192.168.1.100/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
    eth1:
      dhcp4: yes

# Apply configuration
sudo netplan apply
sudo netplan try               # Test configuration
```

#### Debian/Ubuntu (/etc/network/interfaces - legacy)
```bash
# /etc/network/interfaces
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 192.168.1.100
netmask 255.255.255.0
gateway 192.168.1.1
dns-nameservers 8.8.8.8 8.8.4.4

auto eth1
iface eth1 inet dhcp

# Restart networking
sudo systemctl restart networking
sudo /etc/init.d/networking restart
```

#### Red Hat/CentOS (/etc/sysconfig/network-scripts/)
```bash
# /etc/sysconfig/network-scripts/ifcfg-eth0
TYPE=Ethernet
BOOTPROTO=static
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=192.168.1.100
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
DNS2=8.8.4.4

# Restart network interface
sudo ifdown eth0 && sudo ifup eth0
sudo systemctl restart network
```

## Routing and Gateway Management

### Routing Table Management
```bash
# View routing table
ip route show                  # Current routing table
ip route show table all        # All routing tables
route -n                       # Legacy command (numeric)

# Add routes
ip route add 10.0.0.0/8 via 192.168.1.1 dev eth0        # Add network route
ip route add default via 192.168.1.1 dev eth0           # Add default route
ip route add 172.16.0.0/12 via 192.168.1.254            # Route via gateway

# Delete routes
ip route del 10.0.0.0/8 via 192.168.1.1 dev eth0        # Delete specific route
ip route del default                                     # Delete default route

# Route with specific metric (priority)
ip route add 0.0.0.0/0 via 192.168.1.1 dev eth0 metric 100

# Legacy routing commands
route add -net 10.0.0.0 netmask 255.0.0.0 gw 192.168.1.1
route del -net 10.0.0.0 netmask 255.0.0.0
```

### Advanced Routing
```bash
# Policy routing
ip rule list                   # Show routing rules
ip route show table main       # Main routing table
ip route show table local      # Local routing table

# Create custom routing table
echo "200 custom" >> /etc/iproute2/rt_tables
ip route add 10.0.0.0/8 via 192.168.1.1 table custom
ip rule add from 192.168.1.100 table custom

# Route based on source IP
ip rule add from 192.168.1.0/24 table 100
ip route add default via 192.168.1.1 table 100
```

## Network Diagnostics and Monitoring

### Connectivity Testing
```bash
# Ping operations
ping -c 4 google.com           # Ping 4 times
ping -i 0.2 -c 10 host         # Ping every 0.2 seconds
ping -s 1024 host              # Ping with specific packet size
ping6 google.com               # IPv6 ping

# Traceroute
traceroute google.com          # Trace route to destination
traceroute -n google.com       # No hostname resolution
tracepath google.com           # Alternative to traceroute
mtr google.com                 # Continuous traceroute

# Path MTU discovery
tracepath -m 30 google.com     # Discover path MTU
```

### Port and Service Testing
```bash
# Check open ports
netstat -tuln                  # Show listening ports
netstat -tulnp                 # Show with process IDs
netstat -rn                    # Show routing table
netstat -i                     # Show interface statistics

# Modern replacement for netstat
ss -tuln                       # Show listening ports
ss -tulnp                      # Show with process IDs
ss -s                          # Show socket statistics
ss -t state established        # Show established TCP connections

# Test specific ports
telnet host 80                 # Test HTTP port
nc -zv host 22                 # Test SSH port with netcat
nmap -p 22,80,443 host         # Port scanning
```

### Network Monitoring
```bash
# Interface statistics
ip -s link show               # Interface statistics
cat /proc/net/dev             # Network device statistics
sar -n DEV 1                  # Network statistics every second

# Real-time monitoring
iftop                         # Real-time interface monitoring
nethogs                       # Process network usage
vnstat                        # Network statistics
tcpdump -i eth0               # Packet capture
wireshark                     # GUI packet analyzer

# Bandwidth testing
iperf3 -s                     # Start iperf3 server
iperf3 -c server_ip           # Connect to iperf3 server
speedtest-cli                 # Internet speed test
```

### DNS Configuration and Testing
```bash
# DNS configuration
cat /etc/resolv.conf          # View DNS servers
systemd-resolve --status      # systemd DNS status

# DNS testing
nslookup google.com           # Basic DNS lookup
dig google.com                # Detailed DNS query
dig @8.8.8.8 google.com       # Query specific DNS server
dig google.com MX             # Query MX records
host google.com               # Simple DNS lookup

# Reverse DNS lookup
dig -x 8.8.8.8               # Reverse lookup
nslookup 8.8.8.8             # Reverse lookup

# DNS cache management
sudo systemd-resolve --flush-caches    # Flush DNS cache
sudo /etc/init.d/nscd restart          # Restart nscd
```

## Firewall Management

### iptables
```bash
# View current rules
iptables -L                   # List all rules
iptables -L -v -n             # Verbose with numbers
iptables -L INPUT             # List INPUT chain
iptables -t nat -L            # List NAT table

# Basic rules
iptables -A INPUT -p tcp --dport 22 -j ACCEPT      # Allow SSH
iptables -A INPUT -p tcp --dport 80 -j ACCEPT      # Allow HTTP
iptables -A INPUT -p tcp --dport 443 -j ACCEPT     # Allow HTTPS
iptables -A INPUT -j DROP     # Drop everything else

# Source-based rules
iptables -A INPUT -s 192.168.1.0/24 -j ACCEPT     # Allow from subnet
iptables -A INPUT -s 10.0.0.100 -j DROP           # Block specific IP

# Save and restore rules
iptables-save > /etc/iptables/rules.v4            # Save rules
iptables-restore < /etc/iptables/rules.v4         # Restore rules

# Clear all rules
iptables -F                   # Flush all rules
iptables -X                   # Delete custom chains
iptables -t nat -F            # Flush NAT table
```

### UFW (Uncomplicated Firewall)
```bash
# UFW management
sudo ufw status               # Show firewall status
sudo ufw enable               # Enable firewall
sudo ufw disable              # Disable firewall

# Basic rules
sudo ufw allow 22             # Allow SSH
sudo ufw allow ssh            # Allow SSH (by name)
sudo ufw allow 80/tcp         # Allow HTTP
sudo ufw allow from 192.168.1.0/24    # Allow from subnet

# Advanced rules
sudo ufw allow from 192.168.1.0/24 to any port 22
sudo ufw deny from 10.0.0.100
sudo ufw delete allow 80     # Remove rule

# Application profiles
sudo ufw app list            # List application profiles
sudo ufw allow 'Apache Full' # Allow Apache
```

### firewalld
```bash
# firewalld management
sudo systemctl status firewalld        # Check status
sudo firewall-cmd --state              # Check if running

# Zone management
sudo firewall-cmd --get-zones          # List zones
sudo firewall-cmd --get-default-zone   # Show default zone
sudo firewall-cmd --set-default-zone=home    # Set default zone

# Service management
sudo firewall-cmd --list-services      # List allowed services
sudo firewall-cmd --add-service=http   # Allow HTTP
sudo firewall-cmd --add-service=http --permanent    # Make permanent
sudo firewall-cmd --reload             # Reload configuration

# Port management
sudo firewall-cmd --add-port=8080/tcp  # Allow port
sudo firewall-cmd --list-ports         # List open ports
```

## Disk Management and Partitioning

### Disk Information
```bash
# List block devices
lsblk                         # Tree view of block devices
lsblk -f                      # Show filesystem information
fdisk -l                      # List all disks and partitions
parted -l                     # List partitions (GPT and MBR)

# Disk usage
df -h                         # Human-readable disk usage
df -i                         # Inode usage
du -sh /path/*                # Directory sizes
du -sh . | sort -h            # Sort by size

# Detailed disk information
hdparm -I /dev/sda            # Disk information
smartctl -a /dev/sda          # SMART disk health
iostat -x 1                   # I/O statistics
```

### Partitioning

#### fdisk (MBR partitioning)
```bash
# Start fdisk
sudo fdisk /dev/sdb

# fdisk commands:
# p - print partition table
# n - create new partition
# d - delete partition
# t - change partition type
# w - write changes and exit
# q - quit without saving

# Example: Create new partition
sudo fdisk /dev/sdb
# n (new) -> p (primary) -> 1 (partition number) -> Enter -> Enter -> w (write)

# List partition types
sudo fdisk -l /dev/sdb
```

#### parted (GPT partitioning)
```bash
# Start parted
sudo parted /dev/sdb

# Parted commands:
# print                       # Show partition table
# mklabel gpt                 # Create GPT partition table
# mkpart primary ext4 1MiB 100%    # Create partition
# rm 1                        # Remove partition 1
# quit                        # Exit

# Non-interactive usage
sudo parted /dev/sdb mklabel gpt
sudo parted /dev/sdb mkpart primary ext4 1MiB 100%
sudo parted /dev/sdb print
```

#### gdisk (GPT partitioning)
```bash
# Start gdisk
sudo gdisk /dev/sdb

# gdisk commands:
# p - print partition table
# n - create new partition
# d - delete partition
# t - change partition type
# w - write changes and exit
# q - quit without saving
```

### Filesystem Management

#### Creating Filesystems
```bash
# Create ext4 filesystem
sudo mkfs.ext4 /dev/sdb1
sudo mkfs.ext4 -L "MyDisk" /dev/sdb1    # With label

# Create other filesystems
sudo mkfs.xfs /dev/sdb1               # XFS filesystem
sudo mkfs.btrfs /dev/sdb1             # Btrfs filesystem
sudo mkfs.ntfs /dev/sdb1              # NTFS filesystem
sudo mkfs.vfat -F 32 /dev/sdb1        # FAT32 filesystem

# Create swap
sudo mkswap /dev/sdb2                 # Create swap partition
sudo swapon /dev/sdb2                 # Enable swap
```

#### Mounting Filesystems
```bash
# Manual mounting
sudo mkdir /mnt/mydisk
sudo mount /dev/sdb1 /mnt/mydisk
sudo mount -t ext4 /dev/sdb1 /mnt/mydisk    # Specify filesystem type

# Mount with options
sudo mount -o rw,noatime /dev/sdb1 /mnt/mydisk
sudo mount -o ro /dev/sdb1 /mnt/mydisk      # Read-only

# View mounted filesystems
mount                         # Show all mounted filesystems
mount | grep sdb              # Show specific device
findmnt                       # Tree view of mounts
findmnt /mnt/mydisk          # Show specific mount

# Unmounting
sudo umount /mnt/mydisk
sudo umount /dev/sdb1
sudo umount -l /mnt/mydisk    # Lazy unmount
```

#### /etc/fstab Configuration
```bash
# /etc/fstab format:
# device  mountpoint  filesystem  options  dump  pass

# Examples:
UUID=12345678-1234-1234-1234-123456789012  /mnt/data  ext4  defaults  0  2
/dev/sdb1  /mnt/backup  ext4  defaults,noatime  0  2
/dev/sdb2  none  swap  sw  0  0

# Get UUID
sudo blkid /dev/sdb1

# Test fstab
sudo mount -a                # Mount all in fstab
sudo findmnt --verify        # Verify fstab syntax
```

### Logical Volume Management (LVM)

#### Physical Volumes
```bash
# Create physical volume
sudo pvcreate /dev/sdb1 /dev/sdc1
sudo pvs                     # List physical volumes
sudo pvdisplay               # Detailed PV information
sudo pvdisplay /dev/sdb1     # Specific PV info
```

#### Volume Groups
```bash
# Create volume group
sudo vgcreate vg_data /dev/sdb1 /dev/sdc1
sudo vgs                     # List volume groups
sudo vgdisplay               # Detailed VG information
sudo vgdisplay vg_data       # Specific VG info

# Extend volume group
sudo vgextend vg_data /dev/sdd1
```

#### Logical Volumes
```bash
# Create logical volume
sudo lvcreate -L 10G -n lv_data vg_data        # 10GB logical volume
sudo lvcreate -l 100%VG -n lv_backup vg_data   # Use all space

# List logical volumes
sudo lvs                     # List LVs
sudo lvdisplay               # Detailed LV information
sudo lvdisplay /dev/vg_data/lv_data    # Specific LV info

# Extend logical volume
sudo lvextend -L +5G /dev/vg_data/lv_data      # Add 5GB
sudo lvextend -l +100%FREE /dev/vg_data/lv_data # Use all free space

# Resize filesystem after extending LV
sudo resize2fs /dev/vg_data/lv_data            # For ext4
sudo xfs_growfs /mnt/data                      # For XFS
```

#### LVM Snapshots
```bash
# Create snapshot
sudo lvcreate -L 1G -s -n lv_data_snap /dev/vg_data/lv_data

# Mount snapshot
sudo mkdir /mnt/snapshot
sudo mount /dev/vg_data/lv_data_snap /mnt/snapshot

# Remove snapshot
sudo umount /mnt/snapshot
sudo lvremove /dev/vg_data/lv_data_snap
```

### RAID Management

#### Software RAID (mdadm)
```bash
# Create RAID arrays
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1  # RAID 1
sudo mdadm --create /dev/md1 --level=5 --raid-devices=3 /dev/sdb1 /dev/sdc1 /dev/sdd1  # RAID 5

# View RAID status
cat /proc/mdstat
sudo mdadm --detail /dev/md0
sudo mdadm --query /dev/md0

# Add spare disk
sudo mdadm --add /dev/md0 /dev/sde1

# Remove failed disk
sudo mdadm --fail /dev/md0 /dev/sdb1
sudo mdadm --remove /dev/md0 /dev/sdb1

# Stop and start arrays
sudo mdadm --stop /dev/md0
sudo mdadm --assemble /dev/md0 /dev/sdb1 /dev/sdc1

# Configuration file
sudo mdadm --detail --scan >> /etc/mdadm/mdadm.conf
```

### Disk Performance and Monitoring

#### I/O Monitoring
```bash
# Real-time I/O monitoring
iostat -x 1                  # Extended I/O stats every second
iotop                        # Top-like I/O monitor
iotop -o                     # Show only active processes

# Disk activity
sudo iotop -a                # Show accumulated I/O
pidstat -d 1                 # Per-process I/O stats
```

#### Performance Testing
```bash
# Disk speed testing
sudo hdparm -T /dev/sda      # Cache read speed
sudo hdparm -t /dev/sda      # Direct read speed

# dd benchmarks
sudo dd if=/dev/zero of=/tmp/testfile bs=1G count=1 oflag=direct    # Write test
sudo dd if=/tmp/testfile of=/dev/null bs=1G count=1 iflag=direct    # Read test

# Random I/O testing with fio
sudo apt install fio
fio --name=randwrite --ioengine=libaio --rw=randwrite --bs=4k --direct=1 --size=1G --numjobs=1 --runtime=60
```

### Filesystem Maintenance

#### Filesystem Checks
```bash
# Check filesystem (unmounted)
sudo fsck /dev/sdb1
sudo fsck.ext4 /dev/sdb1     # Specific filesystem type
sudo fsck -f /dev/sdb1       # Force check

# Check all filesystems
sudo fsck -A                 # Check all in /etc/fstab
sudo fsck -AR                # Check all except root

# XFS filesystem check
sudo xfs_repair /dev/sdb1    # Repair XFS filesystem
```

#### Filesystem Tuning
```bash
# ext4 tuning
sudo tune2fs -l /dev/sdb1    # Show filesystem parameters
sudo tune2fs -L "NewLabel" /dev/sdb1    # Set label
sudo tune2fs -c 30 /dev/sdb1 # Set max mount count for check
sudo tune2fs -i 0 /dev/sdb1  # Disable time-based checks

# Enable/disable features
sudo tune2fs -O ^has_journal /dev/sdb1    # Disable journaling
sudo tune2fs -O has_journal /dev/sdb1     # Enable journaling
```

## Network Storage

### NFS (Network File System)
```bash
# NFS Server setup
sudo apt install nfs-kernel-server
echo "/srv/nfs 192.168.1.0/24(rw,sync,no_subtree_check)" >> /etc/exports
sudo exportfs -ra            # Reload exports
sudo systemctl restart nfs-kernel-server

# NFS Client
sudo apt install nfs-common
sudo mkdir /mnt/nfs
sudo mount -t nfs server:/srv/nfs /mnt/nfs

# Permanent NFS mount in /etc/fstab
server:/srv/nfs /mnt/nfs nfs defaults 0 0

# Show NFS exports
showmount -e server_ip
```

### CIFS/SMB
```bash
# Install CIFS utilities
sudo apt install cifs-utils

# Mount CIFS share
sudo mkdir /mnt/share
sudo mount -t cifs //server/share /mnt/share -o username=user

# Permanent CIFS mount
//server/share /mnt/share cifs username=user,password=pass,uid=1000,gid=1000 0 0

# Credentials file (more secure)
echo "username=user" > ~/.cifs-credentials
echo "password=pass" >> ~/.cifs-credentials
chmod 600 ~/.cifs-credentials
//server/share /mnt/share cifs credentials=/home/user/.cifs-credentials,uid=1000 0 0
```

### iSCSI
```bash
# iSCSI Initiator setup
sudo apt install open-iscsi

# Discover targets
sudo iscsiadm --mode discovery --type sendtargets --portal server_ip

# Login to target
sudo iscsiadm --mode node --targetname iqn.target.name --portal server_ip --login

# List sessions
sudo iscsiadm --mode session

# Logout from target
sudo iscsiadm --mode node --targetname iqn.target.name --portal server_ip --logout
```

## Network Troubleshooting Tools

### Advanced Network Diagnostics
```bash
# Packet capture and analysis
sudo tcpdump -i eth0 -n                    # Capture packets on eth0
sudo tcpdump -i eth0 port 80                # Capture HTTP traffic
sudo tcpdump -i eth0 host 192.168.1.100    # Capture specific host traffic
sudo tcpdump -i eth0 -w capture.pcap       # Write to file

# Network scanning
nmap -sn 192.168.1.0/24                    # Ping scan
nmap -p 22,80,443 192.168.1.100            # Port scan
nmap -sS -O 192.168.1.100                  # OS detection

# ARP table management
arp -a                                      # Show ARP table
ip neigh show                               # Modern ARP table
sudo arping 192.168.1.1                    # ARP ping
```

### Network Configuration Verification
```bash
# Network namespace (for containers/VMs)
ip netns list                               # List network namespaces
sudo ip netns add testns                    # Create namespace
sudo ip netns exec testns ip link list     # Execute in namespace

# Bridge management
brctl show                                  # Show bridges
sudo brctl addbr br0                        # Create bridge
sudo brctl addif br0 eth0                   # Add interface to bridge

# VLAN configuration
sudo vconfig add eth0 100                   # Create VLAN interface
ip link add link eth0 name eth0.100 type vlan id 100    # Modern VLAN creation
```

## Quick Reference Tables

### Common Network Ports
| Port | Service | Protocol |
|------|---------|----------|
| 22 | SSH | TCP |
| 23 | Telnet | TCP |
| 25 | SMTP | TCP |
| 53 | DNS | TCP/UDP |
| 80 | HTTP | TCP |
| 110 | POP3 | TCP |
| 143 | IMAP | TCP |
| 443 | HTTPS | TCP |
| 993 | IMAPS | TCP |
| 995 | POP3S | TCP |

### Filesystem Types
| Filesystem | Use Case | Max File Size |
|------------|----------|---------------|
| ext4 | General purpose | 16 TiB |
| XFS | High performance | 8 EiB |
| Btrfs | Copy-on-write | 16 EiB |
| ZFS | Enterprise storage | 16 EiB |
| NTFS | Windows compatibility | 256 TiB |
| exFAT | Cross-platform | 128 PiB |

### RAID Levels
| RAID | Min Disks | Fault Tolerance | Capacity |
|------|-----------|-----------------|----------|
| 0 | 2 | None | 100% |
| 1 | 2 | 1 disk | 50% |
| 5 | 3 | 1 disk | (n-1)/n |
| 6 | 4 | 2 disks | (n-2)/n |
| 10 | 4 | 1 per mirror | 50% |
