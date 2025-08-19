# Linux Advanced/Specialized - Complete Cheat Sheet

## 1. Performance Monitoring & Optimization

### CPU Performance Analysis
```bash
# Real-time CPU monitoring
top                              # Traditional process monitor
htop                            # Enhanced interactive process viewer
atop                            # Advanced system & process monitor
iotop                           # I/O usage by process
pidstat 1                       # Per-process statistics every second
mpstat 1                        # Multi-processor statistics

# CPU utilization breakdown
vmstat 1                        # Virtual memory statistics
sar -u 1 10                     # CPU utilization (10 samples)
iostat -c 1                     # CPU statistics
nmon                            # Comprehensive system monitor

# Process-specific CPU analysis
perf top                        # Real-time performance counter profiling
perf record -g ./application    # Record call graph
perf report                     # Analyze recorded data
strace -c -p PID               # System call statistics for process
ltrace -c -p PID               # Library call statistics

# CPU frequency and scaling
cpufreq-info                    # CPU frequency information
cat /proc/cpuinfo              # Detailed CPU information
lscpu                          # CPU architecture info
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

### Memory Performance Analysis
```bash
# Memory usage overview
free -h                         # Human-readable memory info
cat /proc/meminfo              # Detailed memory statistics
vmstat 1                       # Virtual memory stats
sar -r 1 10                    # Memory utilization over time

# Process memory analysis
pmap -x PID                    # Memory map of process
smem                           # Memory usage with shared memory
ps_mem                         # Memory usage by process
valgrind --tool=massif ./app   # Memory profiler

# Memory performance tuning
echo 3 > /proc/sys/vm/drop_caches    # Clear page cache
sysctl vm.swappiness=10              # Reduce swap usage
echo madvise > /sys/kernel/mm/transparent_hugepage/enabled  # THP settings

# Memory leak detection
valgrind --leak-check=full ./app     # Detailed leak detection
```

### Disk I/O Performance
```bash
# I/O monitoring
iotop                          # I/O usage by process
iostat -x 1                    # Extended I/O statistics
sar -d 1 10                    # Disk activity statistics
dstat                          # Versatile system stats tool

# Disk performance testing
dd if=/dev/zero of=testfile bs=1M count=1000  # Write test
dd if=testfile of=/dev/null bs=1M             # Read test
hdparm -t /dev/sda             # Disk read speed test
hdparm -T /dev/sda             # Buffer cache read speed

# Advanced I/O analysis
blktrace /dev/sda              # Block I/O tracing
btrace /dev/sda                # Simplified blktrace
fio --name=test --rw=randread --size=1G --filename=testfile  # Flexible I/O tester

# File system performance
tune2fs -l /dev/sda1           # Ext filesystem parameters
xfs_info /mount/point          # XFS filesystem info
```

### Network Performance
```bash
# Network monitoring
iftop -i eth0                  # Interface bandwidth usage
nethogs                       # Network usage by process
vnstat -i eth0                # Network statistics
ss -i                         # Socket statistics with details

# Network performance testing
iperf3 -s                     # Start iperf server
iperf3 -c server_ip           # Client connection test
netperf -H remote_host        # Network performance benchmark
ping -f -c 1000 host          # Flood ping test

# Network tuning
echo 'net.core.rmem_max = 16777216' >> /etc/sysctl.conf  # Increase buffer
sysctl -w net.ipv4.tcp_window_scaling=1                  # Enable window scaling
ethtool -G eth0 rx 4096 tx 4096                         # Increase ring buffers
```

---

## 2. Security & Permissions

### Advanced File Permissions
```bash
# Extended attributes
lsattr file.txt                # List file attributes
chattr +i file.txt             # Make file immutable
chattr +a file.txt             # Append-only file
chattr -i file.txt             # Remove immutable attribute

# Access Control Lists (ACL)
getfacl file.txt               # Display ACL
setfacl -m u:john:rw file.txt  # Set user permissions
setfacl -m g:staff:r file.txt  # Set group permissions
setfacl -m d:u:john:rw dir/    # Set default ACL for directory
setfacl -R -m u:john:rw dir/   # Recursive ACL
setfacl -b file.txt            # Remove all ACL entries
setfacl -k file.txt            # Remove default ACL

# Special permissions
find /usr -perm -4000          # Find setuid files
find /tmp -perm -1000          # Find sticky bit files
chmod 4755 /usr/bin/sudo       # Set setuid bit
chmod 2755 /shared/directory   # Set setgid bit
chmod 1777 /tmp                # Set sticky bit
```

### SELinux Security
```bash
# SELinux status and modes
getenforce                     # Current SELinux mode
setenforce 0                   # Set to permissive mode
setenforce 1                   # Set to enforcing mode
sestatus                       # Detailed SELinux status

# Context management
ls -Z file.txt                 # Show SELinux context
ps -Z                          # Show process contexts
id -Z                          # Show current user context
chcon -t httpd_exec_t /path/file   # Change file context
restorecon -R /var/www/        # Restore default contexts

# Policy management
setsebool httpd_can_network_connect on    # Set boolean
getsebool -a                   # List all booleans
semanage port -a -t http_port_t -p tcp 8080  # Add port to policy
semodule -l                    # List loaded modules

# Troubleshooting
ausearch -m avc -ts recent     # Search for recent denials
audit2allow -a                 # Generate policy from denials
sealert -a /var/log/audit/audit.log  # Analyze audit log
```

### Firewall Management

#### iptables
```bash
# Basic iptables operations
iptables -L                    # List rules
iptables -L -n --line-numbers  # List with line numbers
iptables -F                    # Flush all rules
iptables -P INPUT DROP         # Set default policy

# Adding rules
iptables -A INPUT -p tcp --dport 22 -j ACCEPT      # Allow SSH
iptables -I INPUT 1 -s 192.168.1.0/24 -j ACCEPT   # Insert at position 1
iptables -A INPUT -p tcp --dport 80 -m limit --limit 25/minute --limit-burst 100 -j ACCEPT

# Deleting rules
iptables -D INPUT 3            # Delete rule number 3
iptables -D INPUT -p tcp --dport 80 -j ACCEPT      # Delete specific rule

# NAT and forwarding
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE  # Enable NAT
iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT         # Forward between interfaces
echo 1 > /proc/sys/net/ipv4/ip_forward                # Enable IP forwarding

# Save and restore
iptables-save > /etc/iptables/rules.v4    # Save rules
iptables-restore < /etc/iptables/rules.v4  # Restore rules
```

#### firewalld
```bash
# Zone management
firewall-cmd --get-default-zone      # Show default zone
firewall-cmd --set-default-zone=public  # Set default zone
firewall-cmd --get-active-zones      # Show active zones
firewall-cmd --list-all-zones        # List all zones

# Service management
firewall-cmd --list-services         # List allowed services
firewall-cmd --add-service=http      # Add service temporarily
firewall-cmd --add-service=http --permanent  # Add service permanently
firewall-cmd --remove-service=http   # Remove service
firewall-cmd --reload               # Reload configuration

# Port management
firewall-cmd --add-port=8080/tcp    # Open port temporarily
firewall-cmd --add-port=8080/tcp --permanent  # Open port permanently
firewall-cmd --list-ports           # List open ports
```

### SSH Security Hardening
```bash
# SSH key management
ssh-keygen -t ed25519 -C "comment"     # Generate Ed25519 key
ssh-keygen -t rsa -b 4096              # Generate 4096-bit RSA key
ssh-copy-id -i ~/.ssh/key.pub user@host  # Copy specific key
ssh-add ~/.ssh/private_key             # Add key to agent

# SSH configuration (/etc/ssh/sshd_config)
# Disable root login: PermitRootLogin no
# Change port: Port 2222
# Disable password auth: PasswordAuthentication no
# Allow specific users: AllowUsers user1 user2
# Enable fail2ban integration

# SSH tunneling
ssh -L 8080:localhost:80 user@host     # Local port forwarding
ssh -R 8080:localhost:80 user@host     # Remote port forwarding
ssh -D 1080 user@host                  # SOCKS proxy
```

---

## 3. Container/Virtualization

### Docker Container Management
```bash
# Container lifecycle
docker run -d --name myapp nginx      # Run container in background
docker run -it ubuntu bash            # Run interactive container
docker run -p 8080:80 nginx          # Port mapping
docker run -v /host/path:/container/path nginx  # Volume mounting
docker run --rm nginx                 # Auto-remove after exit

# Container operations
docker ps                             # List running containers
docker ps -a                          # List all containers
docker logs -f container_name         # Follow logs
docker exec -it container_name bash   # Execute command in container
docker cp file.txt container:/path/   # Copy file to container
docker stats                          # Real-time resource usage

# Image management
docker images                         # List images
docker pull ubuntu:20.04             # Pull specific tag
docker build -t myapp .               # Build image from Dockerfile
docker tag myapp:latest myapp:v1.0    # Tag image
docker push myapp:v1.0                # Push to registry
docker rmi image_id                   # Remove image
docker system prune -a               # Clean up unused images/containers

# Docker Compose
docker-compose up -d                  # Start services in background
docker-compose down                   # Stop and remove services
docker-compose logs -f service_name   # Follow service logs
docker-compose scale web=3            # Scale service to 3 replicas
docker-compose exec web bash          # Execute in service container
```

### Advanced Docker Operations
```bash
# Multi-stage builds (Dockerfile)
FROM node:16 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html

# Docker networks
docker network create mynetwork       # Create custom network
docker network ls                     # List networks
docker run --network mynetwork nginx  # Connect container to network
docker network inspect mynetwork      # Inspect network details

# Docker volumes
docker volume create myvolume         # Create named volume
docker volume ls                      # List volumes
docker run -v myvolume:/data nginx    # Use named volume
docker volume inspect myvolume        # Inspect volume details
```

### Podman (Docker Alternative)
```bash
# Podman basics (rootless containers)
podman run -d --name myapp nginx      # Run container
podman ps                             # List containers
podman images                         # List images
podman logs myapp                     # View logs

# Podman pods (Kubernetes-like)
podman pod create --name mypod -p 8080:80  # Create pod with port mapping
podman run -d --pod mypod nginx       # Add container to pod
podman pod ps                         # List pods
podman generate kube mypod            # Generate Kubernetes YAML

# Rootless operation
podman system migrate                 # Migrate to rootless
podman system reset                   # Reset podman storage
```

### KVM/QEMU Virtualization
```bash
# VM management with virsh
virsh list                            # List running VMs
virsh list --all                      # List all VMs
virsh start vm_name                   # Start VM
virsh shutdown vm_name                # Graceful shutdown
virsh destroy vm_name                 # Force stop VM
virsh reboot vm_name                  # Restart VM

# VM creation and modification
virt-install --name myvm --ram 2048 --disk size=20 --os-variant ubuntu20.04
virsh edit vm_name                    # Edit VM configuration
virsh attach-disk vm_name /path/to/disk.img vdb  # Attach disk
virsh detach-disk vm_name vdb         # Detach disk

# Snapshots
virsh snapshot-create vm_name         # Create snapshot
virsh snapshot-list vm_name           # List snapshots
virsh snapshot-revert vm_name snapshot_name  # Revert to snapshot
virsh snapshot-delete vm_name snapshot_name  # Delete snapshot

# Resource management
virsh setmem vm_name 4G               # Set memory
virsh setvcpus vm_name 4              # Set CPU count
virsh dominfo vm_name                 # Show VM info
```

---

## 4. System Services (systemd)

### Service Management
```bash
# Service operations
systemctl start service_name          # Start service
systemctl stop service_name           # Stop service
systemctl restart service_name        # Restart service
systemctl reload service_name         # Reload configuration
systemctl enable service_name         # Enable at boot
systemctl disable service_name        # Disable at boot
systemctl mask service_name           # Prevent service from starting

# Service status and information
systemctl status service_name         # Detailed service status
systemctl is-active service_name      # Check if running
systemctl is-enabled service_name     # Check if enabled
systemctl list-units --type=service   # List all services
systemctl list-units --failed         # List failed services
```

### Creating Custom Services
```bash
# Create service file (/etc/systemd/system/myapp.service)
cat > /etc/systemd/system/myapp.service << EOF
[Unit]
Description=My Application
After=network.target
Wants=network.target

[Service]
Type=simple
User=myuser
Group=mygroup
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/myapp
Restart=always
RestartSec=5
Environment=NODE_ENV=production
EnvironmentFile=-/etc/myapp/environment

[Install]
WantedBy=multi-user.target
EOF

# Reload and start service
systemctl daemon-reload               # Reload systemd configuration
systemctl enable myapp                # Enable service
systemctl start myapp                 # Start service
```

### Advanced Service Configuration
```bash
# Service dependencies and ordering
# After= : Start after specified units
# Before= : Start before specified units  
# Requires= : Hard dependency
# Wants= : Soft dependency
# Conflicts= : Cannot run with specified units

# Service types
# Type=simple : Default, process runs in foreground
# Type=forking : Process forks and parent exits
# Type=oneshot : Process exits after completion
# Type=notify : Service notifies systemd when ready
# Type=dbus : Service takes name on D-Bus

# Resource limits
systemctl edit myapp                  # Create override file
# Add in override file:
[Service]
LimitNOFILE=65536
MemoryLimit=1G
CPUQuota=50%
```

### Timer Units (Cron Alternative)
```bash
# Create timer unit (/etc/systemd/system/mybackup.timer)
cat > /etc/systemd/system/mybackup.timer << EOF
[Unit]
Description=Run backup daily
Requires=mybackup.service

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
EOF

# Timer management
systemctl enable mybackup.timer       # Enable timer
systemctl start mybackup.timer        # Start timer
systemctl list-timers                 # List all timers
systemctl status mybackup.timer       # Timer status
```

### Logging and Journald
```bash
# Journal viewing
journalctl                            # View all logs
journalctl -u service_name            # Logs for specific service
journalctl -f                         # Follow logs in real-time
journalctl --since "2024-01-01"       # Logs since date
journalctl --until "2024-01-02"       # Logs until date
journalctl -p err                     # Error level and above
journalctl -k                         # Kernel messages only

# Journal management
journalctl --disk-usage               # Show disk usage
journalctl --vacuum-time=2d           # Keep only 2 days of logs
journalctl --vacuum-size=100M         # Keep only 100MB of logs
systemctl restart systemd-journald    # Restart logging service

# Persistent logging
mkdir -p /var/log/journal             # Enable persistent storage
systemctl restart systemd-journald
```

---

## Advanced Integration Examples

### Performance Monitoring Script
```bash
#!/bin/bash
# Comprehensive system monitor

LOG_FILE="/var/log/system_monitor.log"
THRESHOLD_CPU=80
THRESHOLD_MEM=90
THRESHOLD_DISK=85

log_message() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> "$LOG_FILE"
}

check_cpu() {
    local cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
    if (( $(echo "$cpu_usage > $THRESHOLD_CPU" | bc -l) )); then
        log_message "HIGH CPU: ${cpu_usage}%"
        # Alert or take action
    fi
}

check_memory() {
    local mem_usage=$(free | grep Mem | awk '{printf "%.0f", ($3/$2)*100}')
    if [ "$mem_usage" -gt "$THRESHOLD_MEM" ]; then
        log_message "HIGH MEMORY: ${mem_usage}%"
        # Alert or take action
    fi
}

check_disk() {
    while read -r filesystem usage mountpoint; do
        if [ "$usage" -gt "$THRESHOLD_DISK" ]; then
            log_message "HIGH DISK: $mountpoint at ${usage}%"
        fi
    done < <(df -h | awk 'NR>1 {print $1, $5, $6}' | sed 's/%//')
}

# Run checks
check_cpu
check_memory  
check_disk
```

### Container Security Hardening
```bash
# Docker security best practices
docker run -d \
  --read-only \                        # Read-only filesystem
  --tmpfs /tmp \                       # Writable temp directory
  --cap-drop ALL \                     # Drop all capabilities
  --cap-add NET_BIND_SERVICE \         # Add only needed capabilities
  --user 1000:1000 \                   # Run as non-root user
  --security-opt no-new-privileges \   # Prevent privilege escalation
  --memory 512m \                      # Memory limit
  --cpus="0.5" \                       # CPU limit
  nginx
```

### Automated Service Deployment
```bash
#!/bin/bash
# Blue-green deployment script

SERVICE_NAME="myapp"
NEW_VERSION="$1"
OLD_PORT=8080
NEW_PORT=8081

# Build new version
docker build -t "${SERVICE_NAME}:${NEW_VERSION}" .

# Start new version on different port
docker run -d --name "${SERVICE_NAME}-new" \
  -p $NEW_PORT:80 "${SERVICE_NAME}:${NEW_VERSION}"

# Health check
for i in {1..30}; do
    if curl -f "http://localhost:$NEW_PORT/health"; then
        echo "New version is healthy"
        break
    fi
    sleep 2
done

# Switch traffic (nginx config update)
sed -i "s/:$OLD_PORT/:$NEW_PORT/g" /etc/nginx/sites-available/myapp
nginx -s reload

# Stop old version
docker stop "${SERVICE_NAME}-old" 2>/dev/null || true
docker rm "${SERVICE_NAME}-old" 2>/dev/null || true

# Rename containers
docker rename "$SERVICE_NAME" "${SERVICE_NAME}-old" 2>/dev/null || true
docker rename "${SERVICE_NAME}-new" "$SERVICE_NAME"

echo "Deployment completed successfully"
```

---

## Security Best Practices Checklist

### System Hardening
- [ ] Disable root SSH login
- [ ] Use SSH keys instead of passwords
- [ ] Configure fail2ban for brute force protection
- [ ] Enable and configure firewall
- [ ] Keep system updated
- [ ] Remove unnecessary services and packages
- [ ] Configure log monitoring and alerting
- [ ] Implement proper backup strategy
- [ ] Use file integrity monitoring (AIDE/Tripwire)
- [ ] Configure SELinux/AppArmor

### Container Security
- [ ] Use minimal base images
- [ ] Run containers as non-root users
- [ ] Implement proper resource limits
- [ ] Use read-only filesystems where possible
- [ ] Regularly update container images
- [ ] Scan images for vulnerabilities
- [ ] Use container orchestration security features
- [ ] Network segmentation for containers

### Performance Optimization
- [ ] Monitor system resources continuously
- [ ] Optimize kernel parameters
- [ ] Configure proper swap settings
- [ ] Implement log rotation
- [ ] Use appropriate filesystems for workload
- [ ] Configure CPU governors
- [ ] Optimize network settings
- [ ] Implement caching strategies

This advanced cheat sheet covers complex system administration tasks that require deep Linux knowledge and careful implementation. Always test in non-production environments first!
