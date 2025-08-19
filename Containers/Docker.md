# Docker Cheat Sheet

## Basic Commands

### Images
```bash
# List images
docker images
docker image ls

# Pull image from registry
docker pull <image>:<tag>

# Build image from Dockerfile
docker build -t <image>:<tag> .
docker build -t <image>:<tag> <path>

# Remove image
docker rmi <image>
docker image rm <image>

# Remove all unused images
docker image prune
```

### Containers

#### Running Containers
```bash
# Run container
docker run <image>

# Run container in background (detached)
docker run -d <image>

# Run with port mapping
docker run -p <host-port>:<container-port> <image>

# Run with volume mount
docker run -v <host-path>:<container-path> <image>

# Run with environment variables
docker run -e VAR=value <image>

# Run interactively with shell
docker run -it <image> /bin/bash

# Run with name
docker run --name <container-name> <image>

# Run and remove after exit
docker run --rm <image>
```

#### Managing Containers
```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Stop container
docker stop <container>

# Start stopped container
docker start <container>

# Restart container
docker restart <container>

# Remove container
docker rm <container>

# Remove all stopped containers
docker container prune

# Force remove running container
docker rm -f <container>
```

#### Container Information
```bash
# Show container logs
docker logs <container>

# Follow logs in real-time
docker logs -f <container>

# Execute command in running container
docker exec <container> <command>

# Open interactive shell in container
docker exec -it <container> /bin/bash

# Show container details
docker inspect <container>

# Show container resource usage
docker stats <container>
```

## Dockerfile

### Common Instructions
```dockerfile
# Base image
FROM ubuntu:20.04

# Set working directory
WORKDIR /app

# Copy files
COPY . .
COPY src/ /app/src/

# Add files (with extraction for archives)
ADD archive.tar.gz /app/

# Run commands during build
RUN apt-get update && apt-get install -y python3

# Set environment variables
ENV NODE_ENV=production

# Expose port
EXPOSE 3000

# Define volume mount point
VOLUME ["/data"]

# Set default user
USER node

# Command to run when container starts
CMD ["python3", "app.py"]

# Entry point (cannot be overridden)
ENTRYPOINT ["python3"]

# Labels for metadata
LABEL version="1.0"
LABEL description="My application"
```

### Multi-stage Build Example
```dockerfile
# Build stage
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM node:16-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm ci --only=production
CMD ["node", "dist/index.js"]
```

## Docker Compose

### Basic docker-compose.yml
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    volumes:
      - .:/app
    depends_on:
      - db
    
  db:
    image: postgres:13
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  postgres_data:
```

### Docker Compose Commands
```bash
# Start services
docker-compose up

# Start in background
docker-compose up -d

# Build and start
docker-compose up --build

# Stop services
docker-compose down

# Stop and remove volumes
docker-compose down -v

# View logs
docker-compose logs

# Follow logs
docker-compose logs -f

# Scale service
docker-compose up --scale web=3

# Execute command in service
docker-compose exec web bash
```

## Networking

```bash
# List networks
docker network ls

# Create network
docker network create <network-name>

# Connect container to network
docker network connect <network> <container>

# Disconnect container from network
docker network disconnect <network> <container>

# Remove network
docker network rm <network>

# Inspect network
docker network inspect <network>
```

## Volumes

```bash
# List volumes
docker volume ls

# Create volume
docker volume create <volume-name>

# Remove volume
docker volume rm <volume-name>

# Remove all unused volumes
docker volume prune

# Inspect volume
docker volume inspect <volume-name>

# Run with volume
docker run -v <volume-name>:/path <image>
```

## Registry Operations

```bash
# Login to registry
docker login
docker login <registry-url>

# Tag image for registry
docker tag <image> <registry>/<image>:<tag>

# Push image to registry
docker push <registry>/<image>:<tag>

# Pull image from registry
docker pull <registry>/<image>:<tag>
```

## System Management

```bash
# Show Docker system info
docker info

# Show disk usage
docker system df

# Clean up everything
docker system prune

# Clean up with volumes
docker system prune -a --volumes

# Show Docker version
docker version

# Show running processes in container
docker top <container>
```

## Useful Flags

| Flag | Description |
|------|-------------|
| `-d` | Run in detached mode (background) |
| `-it` | Interactive with TTY |
| `-p` | Port mapping |
| `-v` | Volume mount |
| `-e` | Environment variable |
| `--name` | Container name |
| `--rm` | Remove after exit |
| `--network` | Connect to network |
| `--restart` | Restart policy |
| `-f` | Force operation |

## Common Patterns

### Development Setup
```bash
# Hot reload for development
docker run -v $(pwd):/app -p 3000:3000 node:16 npm run dev

# Database with persistent data
docker run -d --name postgres \
  -e POSTGRES_PASSWORD=password \
  -v postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 postgres:13
```

### Production Deployment
```bash
# Multi-stage build for smaller image
docker build -t app:prod --target production .

# Run with resource limits
docker run -d --name app \
  --memory="512m" \
  --cpus="0.5" \
  -p 80:3000 \
  app:prod
```

## Troubleshooting

```bash
# Debug container that won't start
docker logs <container>
docker run --rm -it <image> /bin/bash

# Check resource usage
docker stats

# Inspect container configuration
docker inspect <container>

# Check container processes
docker exec <container> ps aux

# Access container filesystem
docker exec -it <container> /bin/bash
```
