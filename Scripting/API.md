# **Random API Key Generator Cheat Sheet**

## **Table of Contents**

1. [Overview](#overview)
2. [Linux CLI Methods](#linux-cli-methods)

   * [OpenSSL](#openssl)
   * [/dev/urandom](#devurandom)
3. [Python](#python)

   * [Alphanumeric API Keys](#alphanumeric-api-keys)
   * [Base64 API Keys](#base64-api-keys)
4. [Node.js / JavaScript](#nodejs--javascript)
5. [PowerShell (Windows)](#powershell-windows)
6. [Best Practices](#best-practices)

---

## **Overview**

* API keys are used for **authentication and authorization**.
* Must be **random, long, and cryptographically secure**.
* Methods shown include **Linux CLI, Python, Node.js, and PowerShell**.

---

## **Linux CLI Methods**

### **OpenSSL**

```bash
# Generate 32-byte hex key
openssl rand -hex 32
# Example: 9f74e2c8d2f4a6b7e3c8d9a0f1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0

# Generate 32-byte base64 key
openssl rand -base64 32
# Example: jK8Jd1+T8v9xYpA3c2LQq5GZ2kP9rBvW0hQmR4cV6T0=
```

---

### **/dev/urandom**

```bash
# Generate 32-character alphanumeric key
head /dev/urandom | tr -dc A-Za-z0-9 | head -c 32 ; echo
# Example: aB3dE6fGh9JkLm1NoPqRsTuVwXyZ0123
```

---

## **Python**

### **Alphanumeric API Keys**

```python
import secrets
import string

alphabet = string.ascii_letters + string.digits
api_key = ''.join(secrets.choice(alphabet) for i in range(32))
print(api_key)
# Example: X9aB3kL8pQ2rS1dE6fGh0JkLmN5oPqRs
```

---

### **Base64 API Keys**

```python
import secrets
import base64

key_bytes = secrets.token_bytes(32)
api_key = base64.urlsafe_b64encode(key_bytes).decode()
print(api_key)
# Example: wK8Jd1+T8v9xYpA3c2LQq5GZ2kP9rBvW0hQmR4cV6T0=
```

---

## **Node.js / JavaScript**

```javascript
const crypto = require('crypto');

// Generate 32-byte hex key
const key = crypto.randomBytes(32).toString('hex');
console.log(key);

// Generate 32-byte base64 key
const keyBase64 = crypto.randomBytes(32).toString('base64');
console.log(keyBase64);
```

---

## **PowerShell (Windows)**

```powershell
# Generate 32-character alphanumeric key
Add-Type -AssemblyName System.Web
[System.Web.Security.Membership]::GeneratePassword(32,4)

# Using .NET RNGCryptoServiceProvider
$bytes = New-Object Byte[] 32
[System.Security.Cryptography.RNGCryptoServiceProvider]::Create().GetBytes($bytes)
[System.Convert]::ToBase64String($bytes)
```
