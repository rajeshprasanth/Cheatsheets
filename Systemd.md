# **Systemd Complete Cheat Sheet**

## **Table of Contents**

1. [Overview](#overview)
2. [Unit Types](#unit-types)
3. [Basic Commands](#basic-commands)
4. [Service Management](#service-management)
5. [Creating Custom Unit Files](#creating-custom-unit-files)
6. [Enable / Disable Services](#enable--disable-services)
7. [Inspecting Units](#inspecting-units)
8. [Journal / Logs](#journal--logs)
9. [Timers](#timers)
10. [Sockets & Path Units](#sockets--path-units)
11. [Advanced Options](#advanced-options)
12. [Security & Resource Management](#security--resource-management)
13. [Dependencies & Ordering](#dependencies--ordering)
14. [Troubleshooting](#troubleshooting)
15. [Tips & Best Practices](#tips--best-practices)

---

## **Overview**

Systemd is the **init system and service manager** in most modern Linux distributions. It handles:

* Boot process
* Service startup / shutdown
* Timers
* Logging & journal management
* Dependency tracking

Example placeholder:

```yaml
user:
  name: John Doe
  address:
    city: Houston
    state: Texas
    country: USA
```

---

## **Unit Types**

| Unit Type   | Description                     |
| ----------- | ------------------------------- |
| `service`   | System or application service   |
| `socket`    | Socket activation unit          |
| `target`    | Grouping units (like runlevels) |
| `timer`     | Time-based activation           |
| `mount`     | Mount points                    |
| `automount` | Automount points                |
| `path`      | File path watches               |
| `swap`      | Swap devices                    |
| `device`    | Device unit                     |
| `scope`     | External processes              |

---

## **Basic Commands**

```bash
# List all active units
systemctl list-units

# List all loaded units (active + inactive)
systemctl list-unit-files

# Show status of systemd
systemctl status

# Show systemd version
systemctl --version
```

---

## **Service Management**

```bash
# Start a service
systemctl start nginx.service

# Stop a service
systemctl stop nginx.service

# Restart a service
systemctl restart nginx.service

# Reload configuration without restart
systemctl reload nginx.service

# Check service status
systemctl status nginx.service

# List all active services
systemctl list-units --type=service

# Mask a service (prevents it from starting)
systemctl mask apache2.service

# Unmask a service
systemctl unmask apache2.service
```

---

## **Creating Custom Unit Files**

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Custom App
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/myapp --config /etc/myapp/config.yml
Restart=on-failure
RestartSec=5
User=john
Group=john
Environment="ENV=production"
WorkingDirectory=/opt/myapp

[Install]
WantedBy=multi-user.target
```

```bash
# Reload systemd after creating/updating unit
systemctl daemon-reload

# Start custom service
systemctl start myapp.service

# Enable service at boot
systemctl enable myapp.service

# Check service status
systemctl status myapp.service
```

---

## **Enable / Disable Services**

```bash
# Enable service at boot
systemctl enable nginx.service

# Disable service at boot
systemctl disable nginx.service

# Enable & start immediately
systemctl enable --now nginx.service

# Disable & stop immediately
systemctl disable --now nginx.service
```

---

## **Inspecting Units**

```bash
# Show detailed info
systemctl show nginx.service

# List dependencies
systemctl list-dependencies nginx.service

# View startup logs for a service
journalctl -u nginx.service

# Check all failed units
systemctl --failed
```

---

## **Journal / Logs**

```bash
# View all logs
journalctl

# Follow logs in real-time
journalctl -f

# Show logs for today
journalctl --since today

# Show last 50 log lines
journalctl -n 50

# Show logs for a specific boot
journalctl -b -1

# Filter logs by priority
journalctl -p err..emerg
```

---

## **Timers**

```ini
# /etc/systemd/system/myjob.timer
[Unit]
Description=Run MyJob every 10 minutes

[Timer]
OnBootSec=5min
OnUnitActiveSec=10min
Unit=myjob.service

[Install]
WantedBy=timers.target
```

```bash
# Enable and start timer
systemctl enable --now myjob.timer

# Check timers
systemctl list-timers

# Stop a timer
systemctl stop myjob.timer
```

---

## **Sockets & Path Units**

```ini
# Socket unit example: myapp.socket
[Unit]
Description=My App Socket

[Socket]
ListenStream=12345

[Install]
WantedBy=sockets.target

# Path unit example: watch /tmp/file.txt
[Unit]
Description=Watch file

[Path]
PathExists=/tmp/file.txt

[Install]
WantedBy=multi-user.target
```

```bash
# Start socket
systemctl start myapp.socket

# Enable socket at boot
systemctl enable myapp.socket

# Check active sockets
systemctl list-sockets
```

---

## **Advanced Options**

```ini
# Restart policies
Restart=always
Restart=on-failure
RestartSec=10

# Logging
StandardOutput=journal
StandardError=journal

# Resource limits
MemoryMax=512M
CPUQuota=50%
TasksMax=200

# Environment variables
Environment="ENV=production" "DEBUG=false"

# Dependencies
Requires=network.target
After=network.target
Wants=network-online.target
```

---

## **Security & Resource Management**

```ini
# Run service in restricted mode
ProtectSystem=full
ProtectHome=yes
NoNewPrivileges=yes
PrivateTmp=yes
PrivateDevices=yes
ReadOnlyPaths=/etc
InaccessiblePaths=/root

# Limit processes and memory
TasksMax=100
MemoryMax=256M
CPUQuota=50%
```

---

## **Dependencies & Ordering**

* `After=` → Service starts **after** target/unit
* `Requires=` → Service **fails if dependency fails**
* `Wants=` → Service **optional dependency**
* `Before=` → Service starts **before** target/unit

---

## **Troubleshooting**

```bash
# Debug failed service
systemctl status myapp.service
journalctl -xe

# Test unit without enabling
systemctl start myapp.service --no-block

# Check unit configuration
systemd-analyze verify /etc/systemd/system/myapp.service

# Analyze boot performance
systemd-analyze blame
systemd-analyze critical-chain
```

