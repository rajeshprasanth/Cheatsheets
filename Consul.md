# Consul Cheat Sheet

## Table of Contents

1. [Basics](#basics)
2. [Installation & Agent](#installation--agent)
3. [CLI Commands](#cli-commands)
4. [Service Registration](#service-registration)
5. [Key/Value Store](#keyvalue-store)
6. [Health Checks](#health-checks)
7. [ACLs & Security](#acls--security)
8. [Multi-Datacenter Setup](#multi-datacenter-setup)
9. [Consul UI & API](#consul-ui--api)
10. [Homelab Use Cases](#homelab-use-cases)

---

## Basics

```text
Consul → Service discovery & configuration
Features:
- Service registry & health monitoring
- Key/Value store
- DNS and HTTP API interface
- Multi-datacenter support
- ACLs & token-based security
- Integration with Vault
```

---

## Installation & Agent

```bash
# Start Consul in dev mode (single node)
consul agent -dev

# Start server agent
consul agent -server -bootstrap-expect=1 -data-dir=/tmp/consul -node=<node-name> -bind=<IP>

# Start client agent
consul agent -data-dir=/tmp/consul -node=<node-name> -bind=<IP> -retry-join=<server-IP>

# Check agent info
consul info
```

---

## CLI Commands

```bash
# List cluster members
consul members

# Show Raft peers
consul operator raft list-peers

# Validate configuration file
consul validate /path/to/config.json

# Monitor agent status
consul monitor
```

---

## Service Registration

```bash
# Register service via CLI
consul services register -name=my-service -port=8080

# Register service using JSON
cat <<EOF > web.json
{
  "service": {
    "name": "web",
    "tags": ["http"],
    "port": 80,
    "check": {
      "http": "http://localhost:80/health",
      "interval": "10s"
    }
  }
}
EOF
consul services register web.json

# List registered services
consul catalog services

# Deregister service
consul services deregister <service-id>

# Query service via DNS
dig @127.0.0.1 -p 8600 web.service.consul
```

---

## Key/Value Store

```bash
# Write key/value
consul kv put myapp/config/db "mysql://user:pass@localhost/db"

# Read key/value
consul kv get myapp/config/db

# Delete key
consul kv delete myapp/config/db

# List keys recursively
consul kv list myapp/config/
```

---

## Health Checks

```bash
# HTTP health check for service
consul services register -name=web -port=80 -check='http://localhost:80/health'

# TCP health check
consul services register -name=redis -port=6379 -check='tcp://localhost:6379'

# Node health
consul health node <node-name>

# Service health
consul health service <service-name>
```

---

## ACLs & Security

```bash
# Enable ACL system
consul acl bootstrap

# List tokens
consul acl token list

# Create policy
consul acl policy create -name "read-only" -rules @policy.hcl

# Create token for policy
consul acl token create -description "App Token" -policy-name "read-only"

# Apply token to API/CLI
CONSUL_HTTP_TOKEN=<token> consul kv put myapp/key "value"
```

---

## Multi-Datacenter Setup

```bash
# Join a datacenter
consul join <other-dc-node-ip>

# List datacenters
consul catalog datacenters

# Query cross-datacenter service
consul catalog services -datacenter=<dc-name>
```

---

## Consul UI & API

```text
# UI access
http://<consul-agent-ip>:8500

# Features:
- Nodes & services
- KV store browser
- Health status dashboard
- ACL & token management
- Raft leader visualization

# HTTP API example
curl http://127.0.0.1:8500/v1/catalog/services
curl http://127.0.0.1:8500/v1/kv/myapp/config
```

---

## Homelab Use Cases

```text
# Service discovery for homelab
- Automatically register Home Assistant, Prometheus, Grafana services
- Monitor service health
- Dynamic configuration storage in KV store
- Secure ACL-based access to secrets & configuration
- Multi-datacenter emulation (prod vs dev homelab)
```

---

## Useful Command Summary

| Command                                | Description              |
| -------------------------------------- | ------------------------ |
| `consul agent -dev`                    | Start dev agent          |
| `consul agent -server ...`             | Start server agent       |
| `consul agent -client ...`             | Start client agent       |
| `consul members`                       | List cluster members     |
| `consul catalog services`              | List services            |
| `consul services register <file>`      | Register a service       |
| `consul services deregister <service>` | Deregister service       |
| `consul kv put <key> <value>`          | Write KV pair            |
| `consul kv get <key>`                  | Read KV pair             |
| `consul kv delete <key>`               | Delete KV pair           |
| `consul health node <node>`            | Check node health        |
| `consul health service <service>`      | Check service health     |
| `consul acl bootstrap`                 | Enable ACL system        |
| `consul acl token create ...`          | Create token with policy |
