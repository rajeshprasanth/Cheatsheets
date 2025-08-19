# Docker Swarm Cheat Sheet

## What is Docker Swarm?

Docker Swarm is Docker's native clustering and orchestration solution. It turns a pool of Docker hosts into a single virtual Docker host, providing high availability, load balancing, and scaling capabilities.

## Swarm Cluster Management

### Initialize Swarm
```bash
# Initialize swarm on current node (becomes manager)
docker swarm init

# Initialize with specific IP (for multi-interface hosts)
docker swarm init --advertise-addr <manager-ip>

# Initialize with external CA
docker swarm init --external-ca protocol=cfssl,url=https://ca.example.com

# Get join token for workers
docker swarm join-token worker

# Get join token for managers
docker swarm join-token manager

# Rotate join token
docker swarm join-token --rotate worker
```

### Join/Leave Swarm
```bash
# Join as worker node
docker swarm join --token <worker-token> <manager-ip>:2377

# Join as manager node
docker swarm join --token <manager-token> <manager-ip>:2377

# Leave swarm (from worker node)
docker swarm leave

# Leave swarm (from manager node)
docker swarm leave --force

# Remove node from swarm (from manager)
docker node rm <node-id>

# Force remove node
docker node rm --force <node-id>
```

### Node Management
```bash
# List nodes
docker node ls

# Inspect node details
docker node inspect <node-id>
docker node inspect self

# Promote worker to manager
docker node promote <node-id>

# Demote manager to worker
docker node demote <node-id>

# Update node availability
docker node update --availability drain <node-id>
docker node update --availability active <node-id>
docker node update --availability pause <node-id>

# Add labels to node
docker node update --label-add environment=production <node-id>
docker node update --label-add role=database <node-id>

# Remove labels from node
docker node update --label-rm environment <node-id>
```

## Service Management

### Create Services
```bash
# Create simple service
docker service create --name web nginx

# Create service with replicas
docker service create --name web --replicas 3 nginx

# Create service with port mapping
docker service create --name web -p 80:80 --replicas 3 nginx

# Create service with environment variables
docker service create --name app \
  --env NODE_ENV=production \
  --env DB_HOST=database \
  my-app:latest

# Create service with resource constraints
docker service create --name app \
  --limit-cpu 0.5 \
  --limit-memory 512M \
  --reserve-cpu 0.25 \
  --reserve-memory 256M \
  my-app:latest

# Create service with volume mounts
docker service create --name app \
  --mount type=volume,source=app-data,target=/data \
  --mount type=bind,source=/host/path,target=/container/path \
  my-app:latest
```

### Service Configuration
```bash
# Create service with placement constraints
docker service create --name database \
  --constraint 'node.labels.role==database' \
  --constraint 'node.role==worker' \
  postgres:13

# Create service with placement preferences
docker service create --name web \
  --placement-pref 'spread=node.labels.datacenter' \
  --replicas 6 nginx

# Create service with update configuration
docker service create --name app \
  --update-delay 10s \
  --update-parallelism 2 \
  --update-failure-action rollback \
  --rollback-delay 5s \
  my-app:v1.0

# Create service with restart policy
docker service create --name app \
  --restart-condition on-failure \
  --restart-delay 5s \
  --restart-max-attempts 3 \
  my-app:latest
```

### Service Operations
```bash
# List services
docker service ls

# Inspect service
docker service inspect <service-name>
docker service inspect --pretty <service-name>

# View service logs
docker service logs <service-name>
docker service logs -f <service-name>
docker service logs --tail 100 <service-name>

# Scale service
docker service scale <service-name>=5
docker service scale web=3 api=2 database=1

# Update service
docker service update --image my-app:v2.0 <service-name>
docker service update --env-add NEW_VAR=value <service-name>
docker service update --publish-add 8080:80 <service-name>

# Rollback service
docker service rollback <service-name>

# Remove service
docker service rm <service-name>
```

### Service Tasks
```bash
# List service tasks
docker service ps <service-name>

# List all tasks
docker service ps --no-trunc <service-name>

# List failed tasks
docker service ps --filter "desired-state=running" <service-name>

# Get logs from specific task
docker logs <task-id>
```

## Stack Management

### Docker Compose Integration
```yaml
# docker-stack.yml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    deploy:
      replicas: 3
      placement:
        constraints:
          - node.role == worker
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
      rollback_config:
        parallelism: 1
        delay: 5s
    networks:
      - webnet
    configs:
      - source: nginx_config
        target: /etc/nginx/nginx.conf
    secrets:
      - ssl_cert
      - ssl_key

  api:
    image: my-api:latest
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
    deploy:
      replicas: 2
      placement:
        preferences:
          - spread: node.labels.datacenter
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: pause
        monitor: 60s
    networks:
      - webnet
      - dbnet
    depends_on:
      - db

  db:
    image: postgres:13
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.role == database
      restart_policy:
        condition: on-failure
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - dbnet
    secrets:
      - db_password

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints:
          - node.role == manager
    networks:
      - webnet

networks:
  webnet:
    driver: overlay
    attachable: true
  dbnet:
    driver: overlay
    internal: true

volumes:
  db_data:
    driver: local

configs:
  nginx_config:
    file: ./nginx.conf

secrets:
  ssl_cert:
    file: ./ssl/cert.pem
  ssl_key:
    file: ./ssl/key.pem
  db_password:
    external: true
```

### Stack Commands
```bash
# Deploy stack
docker stack deploy -c docker-stack.yml myapp

# Deploy with multiple compose files
docker stack deploy -c docker-stack.yml -c docker-stack.prod.yml myapp

# List stacks
docker stack ls

# List stack services
docker stack services myapp

# List stack tasks
docker stack ps myapp

# Remove stack
docker stack rm myapp
```

## Networking

### Network Management
```bash
# List networks
docker network ls

# Create overlay network
docker network create --driver overlay mynetwork

# Create overlay network with encryption
docker network create --driver overlay --opt encrypted mynetwork

# Create overlay network with custom subnet
docker network create \
  --driver overlay \
  --subnet 10.0.0.0/24 \
  --gateway 10.0.0.1 \
  mynetwork

# Attach service to network
docker service create --network mynetwork nginx

# Connect running service to network
docker service update --network-add mynetwork myservice

# Disconnect service from network
docker service update --network-rm mynetwork myservice

# Remove network
docker network rm mynetwork
```

### Network Types
```bash
# Ingress network (default for published ports)
# Automatically created, handles load balancing

# Overlay networks for service-to-service communication
docker network create --driver overlay backend

# Host network (bypass Docker networking)
docker service create --network host myservice

# None network (no networking)
docker service create --network none myservice
```

## Secrets Management

### Secret Operations
```bash
# Create secret from file
docker secret create my_secret ./secret.txt

# Create secret from stdin
echo "my-secret-value" | docker secret create my_secret -

# List secrets
docker secret ls

# Inspect secret
docker secret inspect my_secret

# Remove secret
docker secret rm my_secret

# Use secret in service
docker service create \
  --name myapp \
  --secret my_secret \
  myimage:latest

# Mount secret to custom location
docker service create \
  --name myapp \
  --secret source=my_secret,target=custom_secret \
  myimage:latest
```

### Secret Examples
```bash
# Database password secret
echo "super-secret-password" | docker secret create db_password -

# SSL certificate secret
docker secret create ssl_cert ./ssl/certificate.pem
docker secret create ssl_key ./ssl/private.key

# API key secret
docker secret create api_key ./api-key.txt

# Use multiple secrets in service
docker service create \
  --name webapp \
  --secret db_password \
  --secret api_key \
  --secret ssl_cert \
  --secret ssl_key \
  webapp:latest
```

## Config Management

### Configuration Operations
```bash
# Create config from file
docker config create nginx_conf ./nginx.conf

# Create config from stdin
cat nginx.conf | docker config create nginx_conf -

# List configs
docker config ls

# Inspect config
docker config inspect nginx_conf

# Remove config
docker config rm nginx_conf

# Use config in service
docker service create \
  --name web \
  --config source=nginx_conf,target=/etc/nginx/nginx.conf \
  nginx:alpine
```

## Health Checks and Rolling Updates

### Health Check Configuration
```yaml
# In docker-stack.yml
services:
  web:
    image: nginx:alpine
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s
    deploy:
      replicas: 3
```

### Rolling Update Strategies
```bash
# Update with zero downtime
docker service update \
  --update-parallelism 1 \
  --update-delay 30s \
  --update-failure-action rollback \
  --update-monitor 60s \
  --image myapp:v2.0 \
  myapp

# Blue-green deployment simulation
# Create new service
docker service create --name myapp-v2 myapp:v2.0

# Test new version
# Switch traffic (update load balancer config)

# Remove old service
docker service rm myapp-v1
docker service update --name myapp myapp-v2
```

## Load Balancing and Service Discovery

### Service Discovery
```bash
# Services can reach each other by service name
# Example: web service can connect to 'api' service
curl http://api:3000/users

# Use service discovery with custom networks
docker network create --driver overlay app-network
docker service create --name api --network app-network api:latest
docker service create --name web --network app-network web:latest
```

### Load Balancing
```bash
# Ingress load balancing (automatic for published ports)
docker service create --name web -p 80:80 --replicas 3 nginx

# Internal load balancing (VIP mode - default)
docker service create --name api --replicas 3 api:latest

# DNS round-robin load balancing
docker service create \
  --name api \
  --endpoint-mode dnsrr \
  --replicas 3 \
  api:latest
```

## Monitoring and Troubleshooting

### Monitoring Commands
```bash
# Monitor cluster status
docker node ls
docker service ls
docker stack ls

# Check service health
docker service ps <service-name>
docker service logs <service-name>

# Monitor resource usage
docker stats
docker system df
docker system events

# Debug service issues
docker service inspect --pretty <service-name>
docker service ps --no-trunc <service-name>
```

### Common Troubleshooting
```bash
# Check node connectivity
docker node inspect <node-id>

# Verify service constraints
docker service inspect <service-name> | grep -A 10 Placement

# Check network connectivity
docker exec <container-id> ping <service-name>
docker exec <container-id> nslookup <service-name>

# Inspect service tasks
docker inspect <task-id>

# Check logs from failed tasks
docker service logs --since 1h <service-name>
```

## High Availability Patterns

### Manager Node HA
```bash
# Recommended: 3 or 5 manager nodes for HA
# Odd numbers prevent split-brain scenarios

# Add manager nodes
docker swarm join --token <manager-token> <manager-ip>:2377

# Check manager status
docker node ls | grep Leader
docker node ls | grep Reachable
```

### Service HA Configuration
```yaml
version: '3.8'
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    deploy:
      replicas: 3
      placement:
        max_replicas_per_node: 1  # Spread across nodes
        constraints:
          - node.role == worker
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
        monitor: 60s
        max_failure_ratio: 0.3
      rollback_config:
        parallelism: 1
        delay: 5s
        failure_action: pause
        monitor: 60s
```

## Security Best Practices

### Cluster Security
```bash
# Enable auto-lock for manager nodes
docker swarm update --autolock=true

# Unlock swarm after manager restart
docker swarm unlock

# Rotate certificates
docker swarm ca --rotate

# Update CA certificate
docker swarm ca --ca-cert ca.pem --ca-key ca-key.pem

# Set certificate expiry
docker swarm update --cert-expiry 720h
```

### Runtime Security
```yaml
services:
  web:
    image: nginx:alpine
    deploy:
      replicas: 3
    # Security configurations
    read_only: true
    user: "nginx:nginx"
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    security_opt:
      - no-new-privileges:true
    tmpfs:
      - /tmp
      - /var/run
```

## Performance Tuning

### Resource Management
```yaml
services:
  web:
    image: nginx:alpine
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M
      placement:
        preferences:
          - spread: node.labels.datacenter
        max_replicas_per_node: 2
```

### Optimization Tips
```bash
# Use overlay networks efficiently
# Minimize cross-node network traffic
# Use placement constraints to co-locate related services

# Optimize image sizes
# Use multi-stage builds
# Use Alpine-based images where possible

# Monitor and adjust resource limits
docker stats
docker service inspect <service-name>
```

## Common Use Cases

### Web Application Stack
```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    configs:
      - source: nginx_config
        target: /etc/nginx/nginx.conf
    secrets:
      - ssl_cert
      - ssl_key
    deploy:
      replicas: 2
    networks:
      - frontend

  app:
    image: myapp:latest
    environment:
      - NODE_ENV=production
    deploy:
      replicas: 4
      placement:
        preferences:
          - spread: node.labels.zone
    networks:
      - frontend
      - backend

  db:
    image: postgres:13
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    volumes:
      - db_data:/var/lib/postgresql/data
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.storage == ssd
    networks:
      - backend
    secrets:
      - db_password

networks:
  frontend:
    driver: overlay
  backend:
    driver: overlay

volumes:
  db_data:
    driver: local

configs:
  nginx_config:
    file: ./nginx.conf

secrets:
  ssl_cert:
    file: ./ssl/cert.pem
  ssl_key:
    file: ./ssl/key.pem
  db_password:
    external: true
```

### Microservices Architecture
```yaml
version: '3.8'

services:
  api_gateway:
    image: traefik:v2.9
    command:
      - --api.insecure=true
      - --providers.docker=true
      - --providers.docker.swarmmode=true
      - --entrypoints.web.address=:80
    ports:
      - "80:80"
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    deploy:
      placement:
        constraints:
          - node.role == manager
    networks:
      - traefik-public

  user_service:
    image: user-service:latest
    deploy:
      replicas: 2
      labels:
        - traefik.enable=true
        - traefik.http.routers.user.rule=PathPrefix(`/api/users`)
        - traefik.http.services.user.loadbalancer.server.port=3000
    networks:
      - traefik-public
      - backend

  order_service:
    image: order-service:latest
    deploy:
      replicas: 3
      labels:
        - traefik.enable=true
        - traefik.http.routers.order.rule=PathPrefix(`/api/orders`)
        - traefik.http.services.order.loadbalancer.server.port=3000
    networks:
      - traefik-public
      - backend

networks:
  traefik-public:
    driver: overlay
    attachable: true
  backend:
    driver: overlay

```

## Useful Aliases

```bash
# Add to your shell configuration
alias dsls='docker service ls'
alias dsps='docker service ps'
alias dslogs='docker service logs'
alias dss='docker service scale'
alias dsu='docker service update'
alias dsi='docker service inspect'
alias dnls='docker node ls'
alias dstls='docker stack ls'
alias dsts='docker stack services'
alias dstp='docker stack ps'
```

## Quick Reference Commands

| Task | Command |
|------|---------|
| Initialize swarm | `docker swarm init` |
| Join worker | `docker swarm join --token <token> <ip>:2377` |
| Create service | `docker service create --name <name> <image>` |
| Scale service | `docker service scale <service>=<replicas>` |
| Update service | `docker service update --image <new-image> <service>` |
| Deploy stack | `docker stack deploy -c <compose-file> <stack>` |
| List services | `docker service ls` |
| Service logs | `docker service logs <service>` |
| List nodes | `docker node ls` |
| Remove service | `docker service rm <service>` |
