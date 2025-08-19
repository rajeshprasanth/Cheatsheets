# Linux Intermediate Level - Complete Cheat Sheet

## 1. Network Management

### Network Configuration Commands

#### Modern IP Command (preferred)
```bash
# Interface management
ip addr show                    # Show all network interfaces
ip addr show eth0               # Show specific interface
ip link show                    # Show link layer information
ip link set eth0 up             # Bring interface up
ip link set eth0 down           # Bring interface down

# IP address management
ip addr add 192.168.1.100/24 dev eth0    # Add IP address
ip addr del 192.168.1.100/24 dev eth0    # Remove IP address
ip addr flush dev eth0                    # Remove all IPs from interface

# Routing
ip route show                   # Show routing table
ip route add default via 192.168.1.1     # Add default route
ip route add 10.0.0.0/8 via 192.168.1.1  # Add specific route
ip route del 10.0.0.0/8                  # Delete route
```

#### Legacy ifconfig (still useful to know)
```bash
ifconfig                        # Show all interfaces
ifconfig eth0                   # Show specific interface
ifconfig eth0 192.168.1.100     # Set IP address
ifconfig eth0 up                # Bring interface up
ifconfig eth0 down              # Bring interface down
```

#### NetworkManager (nmcli)
```bash
nmcli device status             # Show device status
nmcli connection show           # Show connections
nmcli connection up "MyWiFi"    # Activate connection
nmcli connection down "MyWiFi"  # Deactivate connection
nmcli device wifi list          # List WiFi networks
nmcli device wifi connect "SSID" password "password"  # Connect to WiFi
```

### Network Diagnostic Tools
```bash
# Connectivity testing
ping -c 4 google.com            # Send 4 pings
ping6 -c 4 google.com           # IPv6 ping
traceroute google.com           # Trace route to destination
mtr google.com                  # Real-time traceroute

# DNS tools
nslookup google.com             # DNS lookup
dig google.com                  # Detailed DNS query
dig @8.8.8.8 google.com         # Query specific DNS server
dig -x 8.8.8.8                  # Reverse DNS lookup
host google.com                 # Simple DNS lookup

# Network connections
netstat -tuln                   # Show listening ports
netstat -i                      # Show interface statistics
ss -tuln                        # Modern replacement for netstat
ss -p                           # Show processes using sockets
lsof -i :80                     # Show what's using port 80
```

### File Transfer and Remote Access
```bash
# SSH - Secure Shell
ssh user@hostname               # Connect to remote host
ssh -p 2222 user@hostname       # Connect on specific port
ssh -i ~/.ssh/key user@hostname # Use specific SSH key
ssh -L 8080:localhost:80 user@host  # Local port forwarding

# SCP - Secure Copy
scp file.txt user@host:/path/   # Copy file to remote host
scp user@host:/path/file.txt .  # Copy file from remote host
scp -r directory/ user@host:/path/  # Copy directory recursively

# Rsync - Advanced file synchronization
rsync -av source/ dest/         # Archive mode, verbose
rsync -av --delete source/ dest/  # Delete extra files in destination
rsync -av file user@host:/path/   # Sync to remote host
rsync -av --progress large_file user@host:/path/  # Show progress

# Wget and Curl
wget https://example.com/file.zip     # Download file
wget -c https://example.com/file.zip  # Continue partial download
curl -O https://example.com/file.zip  # Download with curl
curl -L https://bit.ly/shortened      # Follow redirects
curl -H "Authorization: Bearer token" api.example.com  # Custom headers
```

### Network Monitoring
```bash
# Interface monitoring
iftop                           # Real-time interface usage
nethogs                         # Network usage by process
iotop                           # I/O usage by process
nload                           # Network load monitor

# Traffic analysis
tcpdump -i eth0                 # Capture packets on interface
tcpdump -i eth0 port 80         # Capture HTTP traffic
tcpdump -w capture.pcap -i eth0 # Save capture to file
```

---

## 2. Package Management

### APT (Debian/Ubuntu)
```bash
# Package information
apt list --installed            # List installed packages
apt list --upgradable           # List upgradable packages
apt search keyword              # Search for packages
apt show package_name           # Show package details
apt depends package_name        # Show package dependencies

# Package installation/removal
apt update                      # Update package index
apt upgrade                     # Upgrade all packages
apt full-upgrade               # Upgrade with dependency resolution
apt install package_name        # Install package
apt install ./local_package.deb # Install local .deb file
apt remove package_name         # Remove package (keep config)
apt purge package_name          # Remove package and config
apt autoremove                  # Remove unused dependencies

# Repository management
apt edit-sources               # Edit sources.list
apt-key list                   # List GPG keys (deprecated)
apt policy package_name        # Show package versions available

# Low-level dpkg commands
dpkg -l                        # List installed packages
dpkg -L package_name           # List files in package
dpkg -S /path/to/file          # Find which package owns file
dpkg -i package.deb            # Install .deb package
dpkg -r package_name           # Remove package
dpkg --configure -a            # Fix broken packages
```

### YUM/DNF (RHEL/CentOS/Fedora)
```bash
# Package information
dnf list installed             # List installed packages
dnf list available             # List available packages
dnf search keyword             # Search packages
dnf info package_name          # Show package info
dnf provides /path/to/file     # Find package providing file

# Package management
dnf update                     # Update all packages
dnf install package_name       # Install package
dnf remove package_name        # Remove package
dnf autoremove                 # Remove unused dependencies
dnf clean all                  # Clean package cache

# Repository management
dnf repolist                   # List repositories
dnf config-manager --add-repo url  # Add repository

# RPM low-level commands
rpm -qa                        # List all installed packages
rpm -qi package_name           # Query package info
rpm -ql package_name           # List package files
rpm -qf /path/to/file          # Find package owning file
rpm -ivh package.rpm           # Install RPM package
```

### Snap Packages (Universal)
```bash
snap list                      # List installed snaps
snap find keyword              # Search for snaps
snap install package_name      # Install snap
snap remove package_name       # Remove snap
snap refresh                   # Update all snaps
snap info package_name         # Show snap information
```

---

## 3. User & Group Management

### User Management
```bash
# User information
id username                    # Show user ID and groups
groups username                # Show user's groups
finger username                # User information (if available)
last username                  # Show user's login history
w                             # Show currently logged in users
who                           # Show logged in users (simple)

# Creating users
useradd username               # Create user with defaults
useradd -m username            # Create user with home directory
useradd -m -s /bin/bash username  # Specify shell
useradd -m -G sudo,audio username  # Add to groups
adduser username               # Interactive user creation (Debian)

# Modifying users
usermod -aG groupname username # Add user to additional group
usermod -s /bin/zsh username   # Change user shell
usermod -d /new/home username  # Change home directory
usermod -l newname oldname     # Change username
passwd username                # Change user password
chage -l username              # Show password aging info
chage -M 90 username           # Set password to expire in 90 days

# Deleting users
userdel username               # Delete user (keep home)
userdel -r username            # Delete user and home directory
```

### Group Management
```bash
# Group information
cat /etc/group                 # List all groups
getent group                   # List groups (includes LDAP/NIS)
groups                         # Show current user's groups

# Creating and modifying groups
groupadd groupname             # Create group
groupmod -n newname oldname    # Rename group
gpasswd -a username groupname  # Add user to group
gpasswd -d username groupname  # Remove user from group
groupdel groupname             # Delete group

# Group passwords (rarely used)
gpasswd groupname              # Set group password
newgrp groupname               # Switch to group temporarily
```

### Permission and Access Control
```bash
# Advanced permissions
chmod u+s file                 # Set setuid bit
chmod g+s file                 # Set setgid bit
chmod +t directory             # Set sticky bit
find /path -perm -u+s          # Find setuid files

# Access Control Lists (ACL)
getfacl file.txt               # Show ACL permissions
setfacl -m u:username:rw file.txt  # Grant user read/write
setfacl -m g:group:r file.txt   # Grant group read access
setfacl -x u:username file.txt  # Remove user ACL
setfacl -b file.txt            # Remove all ACLs

# Sudo configuration
visudo                         # Edit sudo configuration safely
sudo -l                        # List sudo privileges
sudo -u username command       # Run command as different user
sudo -i                        # Login as root
sudo su -                      # Switch to root with environment
```

---

## 4. Shell Scripting Basics

### Script Structure and Basics
```bash
#!/bin/bash                    # Shebang line (script interpreter)

# Variables
name="John"                    # String variable (no spaces around =)
age=25                         # Numeric variable
readonly PI=3.14159            # Read-only variable
unset name                     # Remove variable

# Special variables
$0                            # Script name
$1, $2, $3                    # Command line arguments
$#                            # Number of arguments
$@                            # All arguments as separate words
$*                            # All arguments as single string
$?                            # Exit status of last command
$$                            # Process ID of script
$!                            # PID of last background process
```

### Input/Output and User Interaction
```bash
# Reading user input
read -p "Enter your name: " name
read -s password              # Silent input (for passwords)
read -t 10 response           # Timeout after 10 seconds

# Output
echo "Hello, $name"           # Print text
printf "Number: %d\n" 42      # Formatted output
echo -e "Line 1\nLine 2"      # Enable escape sequences
echo -n "No newline"          # No trailing newline
```

### Control Structures

#### Conditional Statements
```bash
# if-then-else
if [ "$age" -gt 18 ]; then
    echo "Adult"
elif [ "$age" -eq 18 ]; then
    echo "Just became adult"
else
    echo "Minor"
fi

# File tests
if [ -f "/etc/passwd" ]; then
    echo "File exists"
fi

if [ -d "/home" ]; then
    echo "Directory exists"
fi

# String comparisons
if [ "$name" = "John" ]; then
    echo "Hello John"
fi

if [ -z "$variable" ]; then      # Test if variable is empty
    echo "Variable is empty"
fi

if [ -n "$variable" ]; then      # Test if variable is not empty
    echo "Variable has value"
fi

# Numeric comparisons
# -eq (equal), -ne (not equal), -gt (greater), -lt (less)
# -ge (greater or equal), -le (less or equal)
```

#### Case Statements
```bash
case "$choice" in
    1|one)
        echo "You chose one"
        ;;
    2|two)
        echo "You chose two"
        ;;
    *)
        echo "Invalid choice"
        ;;
esac
```

#### Loops
```bash
# for loop
for file in *.txt; do
    echo "Processing $file"
done

for i in {1..10}; do
    echo "Number: $i"
done

for ((i=1; i<=10; i++)); do
    echo "Counter: $i"
done

# while loop
counter=1
while [ $counter -le 5 ]; do
    echo "Iteration: $counter"
    ((counter++))
done

# until loop
until [ $counter -gt 10 ]; do
    echo "Counter: $counter"
    ((counter++))
done
```

### Functions
```bash
# Function definition
greet() {
    local name=$1              # Local variable
    echo "Hello, $name!"
    return 0                   # Return status
}

# Function with parameters
calculate() {
    local num1=$1
    local num2=$2
    local result=$((num1 + num2))
    echo $result
}

# Calling functions
greet "World"
result=$(calculate 5 3)
echo "Result: $result"
```

### Error Handling
```bash
# Exit on error
set -e                        # Exit if any command fails
set -u                        # Exit on undefined variable
set -o pipefail              # Exit on pipe failure

# Error checking
if ! command -v git >/dev/null; then
    echo "Git is not installed"
    exit 1
fi

# Trap signals
trap 'echo "Script interrupted"' INT TERM
trap 'cleanup_function' EXIT
```

### Practical Script Examples

#### System Information Script
```bash
#!/bin/bash
echo "=== System Information ==="
echo "Hostname: $(hostname)"
echo "Date: $(date)"
echo "Uptime: $(uptime -p)"
echo "Disk Usage:"
df -h | grep -E '^/dev/'
echo "Memory Usage:"
free -h
```

#### File Backup Script
```bash
#!/bin/bash
SOURCE="/home/user/documents"
BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_$DATE.tar.gz"

if [ ! -d "$BACKUP_DIR" ]; then
    mkdir -p "$BACKUP_DIR"
fi

echo "Creating backup..."
tar -czf "$BACKUP_DIR/$BACKUP_FILE" -C "$(dirname "$SOURCE")" "$(basename "$SOURCE")"

if [ $? -eq 0 ]; then
    echo "Backup created successfully: $BACKUP_DIR/$BACKUP_FILE"
else
    echo "Backup failed!"
    exit 1
fi
```

#### Log Monitor Script
```bash
#!/bin/bash
LOGFILE="/var/log/syslog"
KEYWORD="error"

tail -f "$LOGFILE" | while read line; do
    if echo "$line" | grep -i "$KEYWORD" >/dev/null; then
        echo "[ALERT] $(date): $line"
        # Could send email or notification here
    fi
done
```

### String Manipulation
```bash
# String operations
string="Hello World"
echo ${#string}               # Length of string
echo ${string:6:5}            # Substring (start 6, length 5)
echo ${string/World/Universe} # Replace first occurrence
echo ${string//l/L}           # Replace all occurrences
echo ${string^}               # Capitalize first character
echo ${string^^}              # Convert to uppercase
echo ${string,,}              # Convert to lowercase
```

### Arrays
```bash
# Array creation
fruits=("apple" "banana" "cherry")
numbers=(1 2 3 4 5)

# Array access
echo ${fruits[0]}             # First element
echo ${fruits[@]}             # All elements
echo ${#fruits[@]}            # Array length

# Adding elements
fruits+=("orange")            # Append element

# Iterating arrays
for fruit in "${fruits[@]}"; do
    echo "Fruit: $fruit"
done
```

---

## Integration and Advanced Tips

### Combining Network and Package Management
```bash
# Update system over SSH
ssh user@server 'sudo apt update && sudo apt upgrade -y'

# Install package on remote system
ssh user@server 'sudo apt install -y nginx'

# Check service status remotely
ssh user@server 'systemctl status nginx'
```

### User Management with Scripting
```bash
#!/bin/bash
# Bulk user creation script
while IFS=, read -r username fullname; do
    if ! id "$username" &>/dev/null; then
        useradd -m -c "$fullname" "$username"
        echo "Created user: $username"
    else
        echo "User $username already exists"
    fi
done < users.csv
```

### Network Monitoring Script
```bash
#!/bin/bash
# Simple network monitor
HOSTS=("google.com" "github.com" "stackoverflow.com")

for host in "${HOSTS[@]}"; do
    if ping -c 1 "$host" &>/dev/null; then
        echo "✓ $host is reachable"
    else
        echo "✗ $host is unreachable"
    fi
done
```

### Package Management Automation
```bash
#!/bin/bash
# System maintenance script
echo "Updating package lists..."
apt update

echo "Upgrading packages..."
apt upgrade -y

echo "Removing unused packages..."
apt autoremove -y

echo "Cleaning package cache..."
apt autoclean

echo "System update complete!"
```

---

## Best Practices and Security

### Script Security
```bash
# Always validate input
if [[ ! "$input" =~ ^[a-zA-Z0-9]+$ ]]; then
    echo "Invalid input"
    exit 1
fi

# Use full paths for commands in scripts
/bin/rm instead of rm
/usr/bin/find instead of find
```

### Network Security
```bash
# SSH key management
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"  # Generate key
ssh-copy-id user@hostname                               # Copy key to server
ssh-add ~/.ssh/id_rsa                                   # Add key to agent
```

### User Security
```bash
# Password policies
chage -M 90 -W 7 username     # 90 day expiry, 7 day warning
passwd -l username            # Lock user account
passwd -u username            # Unlock user account
```

This intermediate cheat sheet bridges the gap between basic Linux usage and advanced system administration. Master these concepts to become proficient in Linux system management!
