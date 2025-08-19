# Vault Cheat Sheet

## Table of Contents

1. [Basics](#basics)
2. [Vault Initialization & Unseal](#vault-initialization--unseal)
3. [Authentication](#authentication)
4. [Secrets Engines](#secrets-engines)
5. [Policies](#policies)
6. [Tokens](#tokens)
7. [CLI Commands](#cli-commands)
8. [Audit Logging](#audit-logging)
9. [KV Secrets Advanced](#kv-secrets-advanced)
10. [Transit Secrets Engine](#transit-secrets-engine)
11. [Database Secrets Engine](#database-secrets-engine)
12. [Vault Status & Monitoring](#vault-status--monitoring)
13. [Homelab Use Cases](#homelab-use-cases)

---

## Basics

```text
Vault → Centralized secrets & encryption
Supports:
- Dynamic secrets (databases, cloud creds)
- Encryption as a service
- Access control (policies)
- Audit logging
- Token-based & AppRole auth
```

---

## Vault Initialization & Unseal

```bash
# Initialize Vault
vault operator init

# Unseal Vault (multiple keys required)
vault operator unseal <unseal-key>

# Check Vault status
vault status
```

---

## Authentication

```bash
# Login using root token
vault login <root-token>

# Enable userpass auth
vault auth enable userpass

# Create user
vault write auth/userpass/users/<username> password=<password> policies=<policy>

# AppRole authentication
vault auth enable approle
vault write auth/approle/role/<role> policies=<policy>
vault read auth/approle/role/<role>/role-id
vault write -f auth/approle/role/<role>/secret-id
```

---

## Secrets Engines

```bash
# Enable KV v2
vault secrets enable -path=secret kv-v2

# Enable Transit engine for encryption
vault secrets enable transit

# Enable Database secrets engine
vault secrets enable database

# Enable AWS secrets engine
vault secrets enable -path=aws aws
```

---

## Policies

```bash
# Create policy from file
vault policy write <name> <policy-file.hcl>

# Example HCL policy
path "secret/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

# List policies
vault policy list
vault policy read <policy-name>
```

---

## Tokens

```bash
# Create token
vault token create -policy=<policy> -ttl=1h

# Lookup token
vault token lookup <token>

# Revoke token
vault token revoke <token>

# Create orphan token (not tied to parent)
vault token create -orphan -policy=<policy>
```

---

## CLI Commands

```bash
# Vault status
vault status

# List mounts
vault secrets list

# Enable auth method
vault auth enable <type>

# Disable auth method
vault auth disable <type>

# Enable secrets engine
vault secrets enable <type>

# Disable secrets engine
vault secrets disable <path>

# Write secret
vault kv put secret/<path> key=value

# Read secret
vault kv get secret/<path>

# Delete secret version
vault kv delete secret/<path>
```

---

## Audit Logging

```bash
# Enable audit device
vault audit enable file file_path=/var/log/vault_audit.log

# List audit devices
vault audit list

# Disable audit device
vault audit disable file
```

---

## KV Secrets Advanced

```bash
# List secrets in a path
vault kv list secret/

# Destroy a version permanently
vault kv destroy secret/myapp 2

# Restore deleted version
vault kv undelete secret/myapp 2
```

---

## Transit Secrets Engine

```bash
# Create encryption key
vault write -f transit/keys/my-key

# Encrypt data
vault write transit/encrypt/my-key plaintext=$(base64 <<< "mysecret")

# Decrypt data
vault write transit/decrypt/my-key ciphertext=<ciphertext>

# Rotate key
vault write -f transit/keys/my-key/rotate
```

---

## Database Secrets Engine

```bash
# Configure database connection
vault write database/config/mydb \
  plugin_name=mysql-database-plugin \
  connection_url="username:password@tcp(localhost:3306)/"

# Create dynamic role
vault write database/roles/my-role \
  db_name=mydb \
  creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}'; GRANT SELECT ON *.* TO '{{name}}'@'%';" \
  default_ttl="1h" max_ttl="24h"

# Generate dynamic credentials
vault read database/creds/my-role
```

---

## Vault Status & Monitoring

```bash
# Vault health check
vault status

# Check seal status
vault operator seal-status

# Monitor audit logs
tail -f /var/log/vault_audit.log
```

---

## Homelab Use Cases

```bash
# Store Home Lab API keys
vault kv put secret/homeassistant api_key="12345"

# Encrypt secrets for apps
vault write transit/encrypt/app-key plaintext=$(base64 <<< "supersecret")

# Generate temporary database credentials for MySQL/PostgreSQL
vault read database/creds/my-role

# Centralized secret storage for multi-server setups
vault kv put secret/nfs username="user" password="pass"
vault kv put secret/prometheus token="abcdef"
```

---

## Useful Command Summary

| Command                                    | Description                  |
| ------------------------------------------ | ---------------------------- |
| `vault operator init`                      | Initialize Vault             |
| `vault operator unseal <key>`              | Unseal Vault                 |
| `vault login <token>`                      | Login to Vault               |
| `vault auth enable <type>`                 | Enable authentication method |
| `vault secrets enable <type>`              | Enable secrets engine        |
| `vault kv put <path> key=value`            | Write KV secret              |
| `vault kv get <path>`                      | Read KV secret               |
| `vault kv delete <path>`                   | Delete KV secret version     |
| `vault policy write <name> <file>`         | Write policy                 |
| `vault token create -policy=<policy>`      | Create token                 |
| `vault token revoke <token>`               | Revoke token                 |
| `vault audit enable file file_path=<path>` | Enable audit logging         |
