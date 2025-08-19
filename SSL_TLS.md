# SSL & TLS Cheat Sheet

## Table of Contents

1. [Basics](#basics)
2. [TLS Versions & Ciphers](#tls-versions--ciphers)
3. [OpenSSL Commands](#openssl-commands)
4. [Certificates](#certificates)
5. [CSR & Key Generation](#csr--key-generation)
6. [Certificate Signing & Management](#certificate-signing--management)
7. [Verify & Test SSL/TLS](#verify--test-ssltls)
8. [Server Configurations](#server-configurations)
9. [Troubleshooting & Debugging](#troubleshooting--debugging)
10. [Homelab / Use Cases](#homelab--use-cases)

---

## Basics

```text
# SSL / TLS overview
SSL (Secure Sockets Layer) → deprecated
TLS (Transport Layer Security) → modern protocol for encrypted communication

# Common use
HTTPS, SMTP over TLS, FTPS, VPN (OpenVPN, WireGuard)
```

---

## TLS Versions & Ciphers

```text
# TLS Versions
TLS 1.0 - Obsolete
TLS 1.1 - Obsolete
TLS 1.2 - Widely used
TLS 1.3 - Modern, recommended

# Cipher types
AES-GCM, AES-CBC, ChaCha20-Poly1305, RSA, ECDHE, DHE
```

---

## OpenSSL Commands

```bash
# Show OpenSSL version
openssl version
openssl version -a

# Connect to server and show TLS handshake
openssl s_client -connect example.com:443

# Show supported protocols/ciphers
openssl s_client -connect example.com:443 -tls1_2
openssl ciphers -v
```

---

## Certificates

```bash
# View certificate details
openssl x509 -in cert.pem -text -noout

# View certificate in DER format
openssl x509 -inform DER -in cert.der -text -noout

# Check certificate fingerprint
openssl x509 -in cert.pem -fingerprint -noout

# Convert certificate formats
openssl x509 -in cert.pem -outform DER -out cert.der
openssl x509 -in cert.der -outform PEM -out cert.pem
```

---

## CSR & Key Generation

```bash
# Generate private key (RSA 2048)
openssl genpkey -algorithm RSA -out server.key -pkeyopt rsa_keygen_bits:2048

# Generate private key (EC)
openssl ecparam -genkey -name prime256v1 -noout -out server.key

# Generate CSR (Certificate Signing Request)
openssl req -new -key server.key -out server.csr

# Generate self-signed certificate (valid 1 year)
openssl req -x509 -nodes -days 365 -key server.key -out server.crt
```

---

## Certificate Signing & Management

```bash
# Sign CSR with CA (internal)
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365

# View certificate chain
openssl verify -CAfile ca.crt server.crt

# Check expiration
openssl x509 -enddate -noout -in server.crt

# Combine key + cert for some apps
cat server.key server.crt > server.pem
```

---

## Verify & Test SSL/TLS

```bash
# Test HTTPS connection
openssl s_client -connect example.com:443 -showcerts

# Check expiration
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates

# Check ciphers
nmap --script ssl-enum-ciphers -p 443 example.com

# Test TLS versions
openssl s_client -connect example.com:443 -tls1_2
openssl s_client -connect example.com:443 -tls1_3
```

---

## Server Configurations

```text
# Apache
SSLEngine on
SSLCertificateFile /etc/ssl/certs/server.crt
SSLCertificateKeyFile /etc/ssl/private/server.key
SSLCertificateChainFile /etc/ssl/certs/ca-bundle.crt
SSLProtocol all -SSLv2 -SSLv3
SSLCipherSuite HIGH:!aNULL:!MD5

# Nginx
ssl_certificate /etc/ssl/certs/server.crt;
ssl_certificate_key /etc/ssl/private/server.key;
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers 'HIGH:!aNULL:!MD5';
ssl_prefer_server_ciphers on;
```

---

## Troubleshooting & Debugging

```bash
# Show handshake & cipher
openssl s_client -connect example.com:443 -tls1_2

# Test certificate chain
openssl verify -CAfile ca-bundle.crt server.crt

# Check certificate expiration
openssl x509 -in server.crt -noout -dates

# Decode certificate
openssl x509 -in server.crt -text -noout
```

---

## Homelab / Use Cases

```bash
# Self-signed cert for home lab web server
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout home.key -out home.crt

# Test TLS 1.3 connectivity on local web server
openssl s_client -connect 10.0.0.10:443 -tls1_3

# Generate wildcard certificate for internal lab domain
openssl req -new -nodes -key lab.key -out lab.csr -subj "/CN=*.rphomelab.in"
openssl x509 -req -in lab.csr -signkey lab.key -out lab.crt -days 365
```

---

## Useful Command Summary

| Command                                                 | Description                      |
| ------------------------------------------------------- | -------------------------------- |
| `openssl s_client -connect host:443`                    | Test TLS handshake               |
| `openssl x509 -in cert.pem -text -noout`                | Show certificate details         |
| `openssl req -new -key server.key -out server.csr`      | Generate CSR                     |
| `openssl req -x509 -nodes -days 365 -key key -out cert` | Generate self-signed cert        |
| `openssl verify -CAfile ca.crt server.crt`              | Verify certificate chain         |
| `openssl x509 -enddate -noout -in cert.pem`             | Check certificate expiration     |
| `openssl ciphers -v`                                    | List supported ciphers           |
| `nmap --script ssl-enum-ciphers -p 443 host`            | Enumerate TLS versions & ciphers |
| `cat key cert > pem`                                    | Combine key + cert for apps      |
| `openssl genpkey -algorithm RSA -out key`               | Generate private key             |

