# Linux Cheat Sheet

## File and Directory Operations

### Basic Navigation
```bash
# Print current working directory
pwd

# List directory contents
ls
ls -l          # Long format
ls -la         # Include hidden files
ls -lh         # Human readable sizes
ls -lS         # Sort by size
ls -lt         # Sort by time
ls -R          # Recursive listing

# Change directory
cd /path/to/directory
cd ~           # Home directory
cd -           # Previous directory
cd ..          # Parent directory
cd ../..       # Two levels up

# Create directories
mkdir dirname
mkdir -p path/to/nested/dirs  # Create parent dirs
mkdir -m 755 dirname          # Set permissions

# Remove directories
rmdir dirname           # Remove empty directory
rm -r dirname          # Remove directory and contents
rm -rf dirname         # Force remove (be careful!)
```

### File Operations
```bash
# Create files
touch filename
touch file1 file2 file3        # Multiple files
touch "file with spaces.txt"

# Copy files and directories
cp source destination
cp -r source_dir dest_dir      # Recursive copy
cp -p source dest              # Preserve permissions/timestamps
cp -v source dest              # Verbose output
cp -i source dest              # Interactive (prompt before overwrite)

# Move/rename files
mv oldname newname
mv source destination
mv file1 file2 /target/dir     # Move multiple files

# Remove files
rm filename
rm -i filename         # Interactive removal
rm -f filename         # Force removal
rm *.txt              # Remove all .txt files
rm -rf dirname        # Remove directory recursively (dangerous!)

# View file contents
cat filename           # Display entire file
less filename          # View file page by page
more filename          # Similar to less
head filename          # First 10 lines
head -n 20 filename    # First 20 lines
tail filename          # Last 10 lines
tail -n 20 filename    # Last 20 lines
tail -f filename       # Follow file changes (useful for logs)
```

### File Information and Search
```bash
# File information
ls -l filename         # Detailed info
stat filename          # File statistics
file filename          # File type
du -sh filename        # File/directory size
du -h dirname          # Directory contents sizes
df -h                  # Disk usage

# Find files and directories
find /path -name "filename"
find . -name "*.txt"               # Find .txt files
find /home -user username          # Find files by user
find . -type f -size +100M         # Files larger than 100MB
find . -type d -name "cache"       # Find directories named cache
find . -mtime -7                   # Files modified in last 7 days
find . -name "*.log" -delete       # Find and delete log files

# Search within files
grep "pattern" filename
grep -r "pattern" directory/       # Recursive search
grep -i "pattern" filename         # Case insensitive
grep -n "pattern" filename         # Show line numbers
grep -v "pattern" filename         # Invert match (exclude pattern)
grep -c "pattern" filename         # Count matches
grep -l "pattern" *.txt            # List files containing pattern

# Locate command (faster than find)
locate filename
updatedb                           # Update locate database
```

## File Permissions and Ownership

### Understanding Permissions
```bash
# Permission format: rwxrwxrwx (user group other)
# r=read(4), w=write(2), x=execute(1)

# View permissions
ls -l filename

# Symbolic notation
chmod u+x filename         # Add execute for user
chmod g-w filename         # Remove write for group
chmod o=r filename         # Set other to read only
chmod a+r filename         # Add read for all

# Numeric notation
chmod 755 filename         # rwxr-xr-x
chmod 644 filename         # rw-r--r--
chmod 600 filename         # rw-------
chmod 777 filename         # rwxrwxrwx (use with caution)

# Recursive permissions
chmod -R 755 directory/

# Change ownership
chown user:group filename
chown username filename
chgrp groupname filename
chown -R user:group directory/

# Special permissions
chmod +t directory         # Sticky bit
chmod g+s filename         # Set group ID
chmod u+s filename         # Set user ID
```

## Text Processing

### Text Editors
```bash
# Nano (beginner-friendly)
nano filename
# Ctrl+X to exit, Ctrl+O to save

# Vim
vim filename
# Press 'i' for insert mode
# Press 'Esc' then ':wq' to save and quit
# Press 'Esc' then ':q!' to quit without saving

# Basic vim commands
:w                         # Save
:q                         # Quit
:wq                        # Save and quit
:q!                        # Quit without saving
/pattern                   # Search forward
?pattern                   # Search backward
dd                         # Delete current line
yy                         # Copy current line
p                          # Paste
```

### Text Manipulation
```bash
# Sort text
sort filename
sort -r filename           # Reverse sort
sort -n filename           # Numeric sort
sort -k2 filename          # Sort by second column

# Remove duplicates
sort filename | uniq
uniq filename              # Remove consecutive duplicates
sort filename | uniq -c    # Count unique lines

# Text substitution
sed 's/old/new/' filename          # Replace first occurrence per line
sed 's/old/new/g' filename         # Replace all occurrences
sed 's/old/new/gi' filename        # Case insensitive replacement
sed -i 's/old/new/g' filename      # In-place editing

# Cut columns
cut -d',' -f1 filename             # First field (comma delimiter)
cut -c1-10 filename                # Characters 1-10
cut -f1,3 filename                 # Fields 1 and 3

# Text processing with awk
awk '{print $1}' filename          # Print first column
awk -F',' '{print $2}' filename    # Use comma as delimiter
awk '{sum+=$1} END {print sum}' file # Sum first column

# Join files
paste file1 file2                  # Join files side by side
join file1 file2                   # Join on common field

# Count words, lines, characters
wc filename
wc -l filename             # Count lines
wc -w filename             # Count words
wc -c filename             # Count characters
```

## System Information

### System Status
```bash
# System information
uname -a                   # System information
uname -r                   # Kernel version
hostname                   # System hostname
uptime                     # System uptime and load
whoami                     # Current username
id                         # User and group IDs
who                        # Logged in users
w                          # Detailed user information

# Hardware information
lscpu                      # CPU information
lsmem                      # Memory information
lsblk                      # Block devices
lsusb                      # USB devices
lspci                      # PCI devices
dmidecode                  # Hardware details (requires root)

# Memory and storage
free -h                    # Memory usage
df -h                      # Disk space usage
du -sh *                   # Directory sizes
fdisk -l                   # Disk partitions (requires root)
```

### Process Management
```bash
# View processes
ps                         # Current user processes
ps aux                     # All processes (detailed)
ps -ef                     # All processes (different format)
pstree                     # Process tree
top                        # Real-time process monitor
htop                       # Enhanced top (if installed)

# Process control
kill PID                   # Terminate process
kill -9 PID               # Force kill
killall processname       # Kill by name
pkill pattern             # Kill by pattern

# Job control
jobs                      # List background jobs
bg %1                     # Put job 1 in background
fg %1                     # Bring job 1 to foreground
nohup command &           # Run command immune to hangups

# Process priorities
nice -n 10 command        # Run with lower priority
renice 10 PID            # Change priority of running process
```

## Network Operations

### Network Information
```bash
# Network interfaces
ip addr show              # Show IP addresses
ip link show              # Show network interfaces
ifconfig                  # Network configuration (deprecated)

# Network connectivity
ping hostname             # Test connectivity
ping -c 4 hostname        # Ping 4 times
traceroute hostname       # Trace network path
netstat -tuln             # Show listening ports
ss -tuln                  # Modern replacement for netstat

# DNS operations
nslookup domain.com       # DNS lookup
dig domain.com            # DNS lookup (more detailed)
host domain.com           # Simple DNS lookup

# Download files
wget URL                  # Download file
wget -O newname URL       # Save with different name
curl URL                  # Transfer data from/to server
curl -O URL               # Save file with original name
```

### Remote Access
```bash
# SSH operations
ssh user@hostname         # Connect to remote host
ssh -p 2222 user@host     # Connect to specific port
ssh-keygen                # Generate SSH key pair
ssh-copy-id user@host     # Copy public key to remote host

# File transfers
scp file user@host:/path  # Copy file to remote host
scp user@host:/path/file . # Copy file from remote host
rsync -av source dest     # Sync directories
rsync -av --delete src dst # Sync and delete extra files
```

## Package Management

### Debian/Ubuntu (APT)
```bash
# Update package list
sudo apt update

# Upgrade packages
sudo apt upgrade
sudo apt full-upgrade     # More comprehensive upgrade

# Install packages
sudo apt install package-name
sudo apt install package1 package2

# Remove packages
sudo apt remove package-name
sudo apt purge package-name    # Remove with config files
sudo apt autoremove           # Remove unused packages

# Search packages
apt search keyword
apt show package-name         # Show package info

# Package files
dpkg -l                       # List installed packages
dpkg -L package-name          # List package files
```

### Red Hat/CentOS (YUM/DNF)
```bash
# Update packages
sudo yum update
sudo dnf update              # For newer versions

# Install packages
sudo yum install package-name
sudo dnf install package-name

# Remove packages
sudo yum remove package-name
sudo dnf remove package-name

# Search packages
yum search keyword
dnf search keyword

# Package information
yum info package-name
dnf info package-name

# List packages
yum list installed
dnf list installed
```

## System Services

### Systemd (Modern Linux)
```bash
# Service control
sudo systemctl start service-name
sudo systemctl stop service-name
sudo systemctl restart service-name
sudo systemctl reload service-name

# Service status
systemctl status service-name
systemctl is-active service-name
systemctl is-enabled service-name

# Enable/disable services
sudo systemctl enable service-name    # Start at boot
sudo systemctl disable service-name   # Don't start at boot

# List services
systemctl list-units --type=service
systemctl list-units --state=failed   # Failed services

# Logs
journalctl -u service-name            # Service logs
journalctl -f                         # Follow logs
journalctl --since "2 hours ago"      # Recent logs
```

### Traditional Init (Older Systems)
```bash
# Service control
sudo service service-name start
sudo service service-name stop
sudo service service-name restart
sudo service service-name status

# Enable/disable services
sudo chkconfig service-name on
sudo chkconfig service-name off
sudo update-rc.d service-name enable
sudo update-rc.d service-name disable
```

## Environment and Variables

### Environment Variables
```bash
# View environment variables
env                        # All environment variables
printenv                   # Same as env
echo $PATH                 # Specific variable
echo $HOME
echo $USER

# Set variables
export VAR_NAME=value      # Set for current session
VAR_NAME=value command     # Set for single command

# Configuration files
~/.bashrc                  # User bash configuration
~/.profile                 # User login configuration
/etc/environment           # System-wide variables
/etc/profile               # System-wide login configuration

# Source configuration
source ~/.bashrc           # Reload bash config
. ~/.bashrc               # Same as source
```

### PATH Management
```bash
# View PATH
echo $PATH

# Add to PATH (temporary)
export PATH=$PATH:/new/path

# Add to PATH (permanent)
echo 'export PATH=$PATH:/new/path' >> ~/.bashrc

# Common PATH locations
/usr/local/bin             # Local binaries
/usr/bin                   # System binaries
/bin                       # Essential binaries
~/bin                      # User binaries
```

## Archive and Compression

### Tar Archives
```bash
# Create archives
tar -cf archive.tar files/         # Create tar archive
tar -czf archive.tar.gz files/     # Create gzipped archive
tar -cjf archive.tar.bz2 files/    # Create bzip2 archive

# Extract archives
tar -xf archive.tar                # Extract tar archive
tar -xzf archive.tar.gz            # Extract gzipped archive
tar -xjf archive.tar.bz2           # Extract bzip2 archive

# List archive contents
tar -tf archive.tar                # List contents
tar -tzf archive.tar.gz            # List gzipped archive

# Extract specific files
tar -xf archive.tar file1 file2    # Extract specific files

# Extract to specific directory
tar -xf archive.tar -C /target/dir
```

### Other Compression
```bash
# Gzip compression
gzip filename              # Compress file
gunzip filename.gz         # Decompress file
zcat filename.gz           # View compressed file

# Zip archives
zip archive.zip files/     # Create zip archive
zip -r archive.zip dir/    # Recursive zip
unzip archive.zip          # Extract zip archive
unzip -l archive.zip       # List zip contents
```

## Input/Output Redirection

### Redirection Operators
```bash
# Output redirection
command > file             # Redirect stdout to file (overwrite)
command >> file            # Redirect stdout to file (append)
command 2> file            # Redirect stderr to file
command &> file            # Redirect both stdout and stderr
command > file 2>&1        # Redirect stderr to stdout, then to file

# Input redirection
command < file             # Use file as input
command << EOF             # Here document
some text
EOF

# Pipes
command1 | command2        # Pipe output of cmd1 to input of cmd2
command | tee file         # Output to both stdout and file
```

### Examples
```bash
# Common redirection examples
ls -l > listing.txt        # Save listing to file
echo "Hello" >> file.txt   # Append text to file
grep error log.txt > errors.txt # Save errors to file
find / -name "*.conf" 2>/dev/null # Hide error messages

# Complex pipes
ps aux | grep nginx | wc -l      # Count nginx processes
cat file.txt | sort | uniq -c    # Sort and count unique lines
ls -l | awk '{print $5,$9}' | sort -n # List files by size
```

## Cron and Task Scheduling

### Crontab Management
```bash
# Edit crontab
crontab -e                 # Edit current user's crontab
sudo crontab -u user -e    # Edit specific user's crontab

# List crontab
crontab -l                 # List current user's crontab
sudo crontab -u user -l    # List specific user's crontab

# Remove crontab
crontab -r                 # Remove current user's crontab
```

### Cron Syntax
```bash
# Format: minute hour day month day_of_week command
# * = any value
# */n = every n units
# n,m = specific values
# n-m = range of values

# Examples:
0 0 * * *           /path/to/script.sh      # Daily at midnight
0 */2 * * *         /path/to/script.sh      # Every 2 hours
30 9 * * 1-5        /path/to/script.sh      # Weekdays at 9:30 AM
0 0 1 * *           /path/to/script.sh      # First day of month
@reboot             /path/to/script.sh      # At startup
@daily              /path/to/script.sh      # Daily (same as 0 0 * * *)
@weekly             /path/to/script.sh      # Weekly
@monthly            /path/to/script.sh      # Monthly
```

## System Monitoring and Logs

### System Monitoring
```bash
# Real-time monitoring
top                        # Process monitor
htop                       # Enhanced process monitor
iotop                      # I/O monitor
iftop                      # Network monitor

# Load average
uptime                     # System uptime and load
w                          # Users and load

# Memory monitoring
free -h                    # Memory usage
vmstat 1                   # Virtual memory statistics
iostat 1                   # I/O statistics

# Disk monitoring
df -h                      # Disk space usage
du -sh /*                  # Directory space usage
iotop                      # Real-time I/O usage
```

### Log Files
```bash
# Common log locations
/var/log/syslog           # System messages
/var/log/auth.log         # Authentication logs
/var/log/kern.log         # Kernel messages
/var/log/apache2/         # Apache logs
/var/log/nginx/           # Nginx logs

# View logs
tail -f /var/log/syslog   # Follow system log
journalctl -f             # Follow systemd journal
less /var/log/auth.log    # View auth log
grep "error" /var/log/syslog # Search for errors

# Log rotation
logrotate -f /etc/logrotate.conf # Force log rotation
```

## Security and User Management

### User Management
```bash
# User operations
sudo adduser username      # Add new user (interactive)
sudo useradd username      # Add user (basic)
sudo userdel username      # Delete user
sudo userdel -r username   # Delete user and home directory

# Password management
passwd                     # Change current user password
sudo passwd username       # Change user password
sudo passwd -l username    # Lock user account
sudo passwd -u username    # Unlock user account

# Group management
groups                     # Show current user groups
sudo groupadd groupname    # Create group
sudo groupdel groupname    # Delete group
sudo usermod -aG groupname username # Add user to group

# Switch users
su username                # Switch to user
su -                       # Switch to root with environment
sudo command               # Run command as root
sudo su -                  # Become root user
```

### File Security
```bash
# File attributes
lsattr filename            # List file attributes
chattr +i filename         # Make file immutable
chattr -i filename         # Remove immutable attribute

# Access Control Lists (ACL)
getfacl filename           # Get file ACL
setfacl -m u:user:rwx file # Set user permissions
setfacl -m g:group:r file  # Set group permissions
setfacl -x u:user file     # Remove user permissions
```

## Quick Reference

### Essential Commands
| Category | Command | Description |
|----------|---------|-------------|
| Navigation | `pwd` | Print working directory |
| | `ls -la` | List files (detailed, all) |
| | `cd ~` | Go to home directory |
| Files | `cp -r src dst` | Copy recursively |
| | `mv old new` | Move/rename |
| | `rm -rf dir` | Remove directory |
| Text | `grep -r pattern dir` | Search recursively |
| | `sed 's/old/new/g' file` | Replace text |
| | `awk '{print $1}' file` | Print first column |
| System | `ps aux` | List all processes |
| | `top` | Process monitor |
| | `df -h` | Disk usage |
| Network | `ping host` | Test connectivity |
| | `ssh user@host` | Remote login |
| | `wget url` | Download file |

### Useful Aliases
Add these to your `~/.bashrc`:
```bash
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias grep='grep --color=auto'
alias ..='cd ..'
alias ...='cd ../..'
alias h='history'
alias df='df -h'
alias du='du -h'
alias free='free -h'
alias psg='ps aux | grep'
alias please='sudo'
alias update='sudo apt update && sudo apt upgrade'
```

### Keyboard Shortcuts
| Shortcut | Description |
|----------|-------------|
| `Ctrl+C` | Interrupt/kill current command |
| `Ctrl+Z` | Suspend current command |
| `Ctrl+D` | Exit/logout |
| `Ctrl+L` | Clear screen |
| `Ctrl+A` | Move to beginning of line |
| `Ctrl+E` | Move to end of line |
| `Ctrl+R` | Search command history |
| `Tab` | Auto-complete |
| `!!` | Repeat last command |
| `!n` | Execute command n from history |
