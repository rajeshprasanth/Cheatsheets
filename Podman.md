# Podman Cheat Sheet

## Table of Contents

1. [Basics](#basics)
2. [Images](#images)
3. [Containers](#containers)
4. [Podman Pods](#podman-pods)
5. [Networking](#networking)
6. [Volumes](#volumes)
7. [Registries](#registries)
8. [System Management](#system-management)
9. [Useful Flags](#useful-flags)
10. [Troubleshooting](#troubleshooting)
11. [Homelab Use Cases](#homelab-use-cases)

---

## Basics

```text
# Podman overview
Podman → daemonless container engine, Docker-compatible CLI
Rootless → run containers without root privileges
Pod → group of containers sharing network namespace
```

---

## Images

```bash
# List images
podman images
podman image ls

# Pull image from registry
podman pull <image>:<tag>

# Build image from Dockerfile
podman build -t <image>:<tag> .
podman build -f <Dockerfile> -t <image>:<tag> .

# Remove image
podman rmi <image>
podman image rm <image>

# Remove all unused images
podman image prune
```

---

## Containers

### Running Containers

```bash
# Run container
podman run <image>

# Run container in background (detached)
podman run -d <image>

# Run with port mapping
podman run -p <host-port>:<container-port> <image>

# Run with volume mount
podman run -v <host-path>:<container-path> <image>

# Run with environment variables
podman run -e VAR=value <image>

# Run interactively with shell
podman run -it <image> /bin/bash

# Run with name
podman run --name <container-name> <image>

# Run and remove after exit
podman run --rm <image>
```

### Managing Containers

```bash
# List running containers
podman ps

# List all containers (including stopped)
podman ps -a

# Stop container
podman stop <container>

# Start stopped container
podman start <container>

# Restart container
podman restart <container>

# Remove container
podman rm <container>

# Remove all stopped containers
podman container prune

# Force remove running container
podman rm -f <container>
```

### Container Information

```bash
# Show container logs
podman logs <container>

# Follow logs in real-time
podman logs -f <container>

# Execute command in running container
podman exec <container> <command>

# Open interactive shell in container
podman exec -it <container> /bin/bash

# Show container details
podman inspect <container>

# Show container resource usage
podman stats <container>
```

---

## Podman Pods

```bash
# Create pod
podman pod create --name <pod-name> -p <host-port>:<container-port>

# List pods
podman pod ps

# Run container in pod
podman run -dt --pod <pod-name> <image>

# Inspect pod
podman pod inspect <pod-name>

# Stop pod (stops all containers)
podman pod stop <pod-name>

# Remove pod
podman pod rm <pod-name>
```

---

## Networking

```bash
# List networks
podman network ls

# Create network
podman network create <network-name>

# Connect container to network
podman network connect <network> <container>

# Disconnect container from network
podman network disconnect <network> <container>

# Remove network
podman network rm <network>

# Inspect network
podman network inspect <network>
```

---

## Volumes

```bash
# List volumes
podman volume ls

# Create volume
podman volume create <volume-name>

# Remove volume
podman volume rm <volume-name>

# Remove all unused volumes
podman volume prune

# Inspect volume
podman volume inspect <volume-name>

# Run container with volume
podman run -v <volume-name>:/path <image>
```

---

## Registries

```bash
# Login to registry
podman login
podman login <registry-url>

# Tag image for registry
podman tag <image> <registry>/<image>:<tag>

# Push image to registry
podman push <registry>/<image>:<tag>

# Pull image from registry
podman pull <registry>/<image>:<tag>
```

---

## System Management

```bash
# Show Podman info
podman info

# Show disk usage
podman system df

# Clean up unused resources
podman system prune

# Show Podman version
podman version
```

---

## Useful Flags

| Flag        | Description                |
| ----------- | -------------------------- |
| `-d`        | Detached mode (background) |
| `-it`       | Interactive with TTY       |
| `-p`        | Port mapping               |
| `-v`        | Volume mount               |
| `-e`        | Environment variable       |
| `--name`    | Container name             |
| `--rm`      | Remove after exit          |
| `--network` | Connect to network         |
| `--pod`     | Run in pod                 |
| `--restart` | Restart policy             |
| `-f`        | Force operation            |

---

## Troubleshooting

```bash
# Debug container that won't start
podman logs <container>
podman run --rm -it <image> /bin/bash

# Inspect container configuration
podman inspect <container>

# Check resource usage
podman stats
```

---

## Homelab Use Cases

```bash
# Run monitoring stack in pod
podman pod create --name monitor -p 3000:3000
podman run -dt --pod monitor prometheus
podman run -dt --pod monitor grafana

# Persistent storage for app
podman volume create app_data
podman run -v app_data:/data -d myapp

# Rootless container for testing
podman run --rm -it alpine /bin/sh
```
