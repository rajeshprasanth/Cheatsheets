# Linux Essentials for Beginners - Complete Cheat Sheet

## 1. File System & Navigation

### Understanding the Linux Directory Structure
```
/           # Root directory (top of file system)
/home       # User home directories
/etc        # System configuration files
/var        # Variable files (logs, temp files)
/usr        # User programs and data
/bin        # Essential system programs
/tmp        # Temporary files
```

### Basic Navigation Commands
```bash
pwd                    # Print current directory (where am I?)
ls                     # List files in current directory
ls -l                  # Detailed list (permissions, size, date)
ls -la                 # Include hidden files (starting with .)
ls -lh                 # Human readable file sizes
cd /path/to/directory  # Change directory
cd ~                   # Go to home directory
cd ..                  # Go up one directory
cd -                   # Go to previous directory
```

### File and Directory Operations
```bash
# Creating
mkdir mydir            # Create directory
mkdir -p dir1/dir2     # Create nested directories
touch file.txt         # Create empty file or update timestamp

# Copying
cp file.txt backup.txt           # Copy file
cp file.txt /path/to/destination # Copy file to another location
cp -r mydir backup_dir          # Copy directory recursively
cp *.txt /backup/               # Copy all .txt files

# Moving/Renaming
mv file.txt newname.txt         # Rename file
mv file.txt /path/to/new/       # Move file to new location
mv olddir newdir                # Rename directory

# Deleting
rm file.txt            # Delete file
rm -i file.txt         # Delete with confirmation prompt
rm -r mydir            # Delete directory and contents
rm -rf mydir           # Force delete (be careful!)
rmdir emptydir         # Remove empty directory only
```

### File Permissions
```bash
# Understanding permissions (rwxrwxrwx)
# First set: owner, second: group, third: others
# r=read(4), w=write(2), x=execute(1)

chmod 755 file.txt     # rwxr-xr-x (owner: all, others: read+execute)
chmod 644 file.txt     # rw-r--r-- (owner: read+write, others: read)
chmod +x script.sh     # Add execute permission for all
chmod u+w file.txt     # Add write permission for user (owner)
chmod g-w file.txt     # Remove write permission for group
chmod o-r file.txt     # Remove read permission for others

# Change ownership
chown user:group file.txt      # Change owner and group
chown user file.txt            # Change owner only
chgrp group file.txt           # Change group only
```

### Finding Files
```bash
find . -name "*.txt"           # Find all .txt files in current directory
find /home -name "filename"    # Find specific file in /home
find . -type d -name "mydir"   # Find directories named "mydir"
find . -size +1M               # Find files larger than 1MB
find . -mtime -7               # Find files modified in last 7 days

locate filename                # Quick search (requires updatedb)
which command                  # Find location of command
whereis command                # Find command, source, and manual
```

---

## 2. Text Processing & Viewing

### Viewing File Contents
```bash
cat file.txt           # Display entire file
cat file1.txt file2.txt # Display multiple files
less file.txt          # View file page by page (q to quit)
more file.txt          # Similar to less (older version)
head file.txt          # Show first 10 lines
head -n 20 file.txt    # Show first 20 lines
tail file.txt          # Show last 10 lines
tail -n 20 file.txt    # Show last 20 lines
tail -f /var/log/syslog # Follow file in real-time (Ctrl+C to stop)
```

### Basic Text Editing with Nano
```bash
nano file.txt          # Open file in nano editor

# Inside nano:
Ctrl+O                 # Save file
Ctrl+X                 # Exit nano
Ctrl+K                 # Cut line
Ctrl+U                 # Paste line
Ctrl+W                 # Search
Ctrl+G                 # Help
```

### Searching Text with grep
```bash
grep "pattern" file.txt        # Search for pattern in file
grep -i "pattern" file.txt     # Case insensitive search
grep -n "pattern" file.txt     # Show line numbers
grep -v "pattern" file.txt     # Show lines that DON'T match
grep -r "pattern" /path/       # Search recursively in directory
grep "^start" file.txt         # Lines starting with "start"
grep "end$" file.txt           # Lines ending with "end"

# Practical examples
grep "error" /var/log/syslog   # Find errors in system log
grep -c "pattern" file.txt     # Count matching lines
ps aux | grep firefox          # Find Firefox processes
```

### Basic Text Processing
```bash
# Sorting and uniqueness
sort file.txt                  # Sort lines alphabetically
sort -n numbers.txt            # Sort numerically
sort -r file.txt               # Reverse sort
uniq sorted_file.txt           # Remove duplicate adjacent lines
sort file.txt | uniq           # Sort and remove duplicates

# Counting
wc file.txt                    # Count lines, words, characters
wc -l file.txt                 # Count lines only
wc -w file.txt                 # Count words only

# Cutting fields
cut -d',' -f1 data.csv         # Extract first column from CSV
cut -c1-10 file.txt            # Extract characters 1-10 from each line

# Simple replacements
tr 'a' 'A' < file.txt          # Replace 'a' with 'A'
tr -d ' ' < file.txt           # Delete spaces
```

---

## 3. Process Management

### Viewing Processes
```bash
ps                     # Show processes for current user
ps aux                 # Show all processes (detailed)
ps aux | grep firefox  # Find specific process
top                    # Real-time process monitor (q to quit)
htop                   # Enhanced version of top (if installed)

# Understanding ps output:
# PID = Process ID
# %CPU = CPU usage
# %MEM = Memory usage
# COMMAND = Process name/command
```

### Controlling Processes
```bash
# Starting processes
./script.sh            # Run script in foreground
./script.sh &          # Run script in background
nohup ./script.sh &    # Run script immune to hangups

# Stopping processes
Ctrl+C                 # Stop current foreground process
Ctrl+Z                 # Suspend current process
jobs                   # List background/suspended jobs
fg                     # Bring background job to foreground
bg                     # Resume suspended job in background

# Killing processes
kill PID               # Terminate process by PID
kill -9 PID            # Force kill process
killall firefox        # Kill all processes named firefox
pkill -f "script.sh"   # Kill processes matching pattern
```

### Process Monitoring
```bash
# System load and uptime
uptime                 # Show system uptime and load
w                      # Show who's logged in and what they're doing
who                    # Show logged in users

# Memory and CPU
free -h                # Show memory usage (human readable)
df -h                  # Show disk space usage
du -h /path/           # Show directory space usage

# Real-time monitoring
top                    # Live process monitor
# In top:
# k = kill process
# q = quit
# P = sort by CPU
# M = sort by memory
```

---

## 4. Basic System Information

### System Identity
```bash
whoami                 # Current username
id                     # User ID and group IDs
hostname               # Computer name
uname -a               # Complete system information
uname -r               # Kernel version
cat /etc/os-release    # Operating system information
```

### Date and Time
```bash
date                   # Current date and time
date +"%Y-%m-%d"       # Custom date format (2024-01-15)
cal                    # Calendar for current month
cal 2024               # Calendar for entire year
uptime                 # System uptime and load average
```

### Hardware Information
```bash
lscpu                  # CPU information
free -h                # Memory (RAM) information
df -h                  # Disk space usage
lsusb                  # USB devices
lspci                  # PCI devices
dmesg | tail           # Recent kernel messages
```

### Network Information
```bash
ip addr                # Network interfaces and IP addresses
ping google.com        # Test network connectivity
wget http://example.com/file.zip  # Download file from internet
curl http://api.example.com       # Make HTTP request
```

### File System Information
```bash
df -h                  # Disk space usage by filesystem
du -h ~                # Disk usage of home directory
du -sh *               # Size of each item in current directory
mount                  # Show mounted filesystems
```

---

## Essential Tips for Beginners

### Command Line Shortcuts
```bash
Tab                    # Auto-complete commands and filenames
Ctrl+C                 # Cancel current command
Ctrl+L                 # Clear screen (same as 'clear' command)
Ctrl+R                 # Search command history
↑/↓ arrows            # Navigate command history
!!                     # Repeat last command
!grep                  # Repeat last command starting with 'grep'
```

### Getting Help
```bash
man command            # Manual page for command
command --help         # Quick help for command
info command           # Info page (alternative to man)
which command          # Find location of command
type command           # Show what type of command it is
```

### Safe Practices
```bash
# Always be careful with:
rm -rf                 # Can delete everything!
sudo commands          # Run with admin privileges
> file.txt             # Overwrites file completely
>> file.txt            # Appends to file (safer)

# Use these flags for safety:
rm -i file.txt         # Prompt before deleting
cp -i file1 file2      # Prompt before overwriting
mv -i old new          # Prompt before overwriting
```

### Common Patterns
```bash
# Combining commands with pipes
ls -la | grep "txt"            # List files, show only .txt files
ps aux | grep firefox          # Show all processes, find Firefox
cat file.txt | sort | uniq     # Show file, sort it, remove duplicates

# Redirecting output
ls > files.txt                 # Save file list to file
grep "error" log.txt > errors.txt  # Save search results
command 2> errors.log          # Save error messages to file
command &> all_output.log      # Save both output and errors
```

### File Paths
```bash
/absolute/path         # Starts from root directory
relative/path          # Relative to current directory
~/Documents           # Tilde expands to home directory
.                     # Current directory
..                    # Parent directory
../..                 # Two directories up
```

### Common File Extensions
```bash
.txt                  # Text file
.log                  # Log file
.conf or .cfg         # Configuration file
.sh                   # Shell script
.tar, .gz, .zip       # Archive/compressed files
.deb, .rpm            # Package files
```

---

## Practice Exercises

### File Management Practice
```bash
# Try these commands to practice:
mkdir practice
cd practice
touch file1.txt file2.txt
echo "Hello World" > file1.txt
cp file1.txt backup.txt
ls -la
grep "Hello" *.txt
rm backup.txt
cd ..
```

### Process Practice
```bash
# Start a long-running process
sleep 300 &           # Sleep for 5 minutes in background
jobs                  # See the job
kill %1               # Kill the first background job
```

### Text Processing Practice
```bash
# Create a sample file and practice
echo -e "apple\nbanana\napple\ncherry" > fruits.txt
cat fruits.txt
sort fruits.txt
sort fruits.txt | uniq
grep "apple" fruits.txt
wc -l fruits.txt
```

Remember: Linux is case-sensitive, and there are no "undo" commands, so always double-check before running destructive commands!
