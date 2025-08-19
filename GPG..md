# **GPG (GNU Privacy Guard) Cheat Sheet**

## **Table of Contents**

1. [Overview](#overview)
2. [Installation](#installation)
3. [Key Management](#key-management)
4. [Encrypting & Decrypting Files](#encrypting--decrypting-files)
5. [Signing & Verifying](#signing--verifying)
6. [Key Servers](#key-servers)
7. [Revocation & Expiration](#revocation--expiration)
8. [Advanced Options](#advanced-options)
9. [Integration](#integration)
10. [Security Best Practices](#security-best-practices)
11. [Troubleshooting](#troubleshooting)

---

## **Overview**

GPG (GNU Privacy Guard) is used for:

* **Encrypting** files or messages
* **Signing** files or emails to prove authenticity
* Managing **public/private key pairs**
* Verifying integrity of files

It uses **asymmetric cryptography** with a public key for encryption and a private key for decryption.

---

## **Installation**

```bash
# Debian/Ubuntu
sudo apt update && sudo apt install gnupg

# RedHat/Fedora
sudo dnf install gnupg

# macOS
brew install gnupg
```

---

## **Key Management**

```bash
# Generate a new key pair (interactive)
gpg --full-generate-key

# List keys (public)
gpg --list-keys

# List secret/private keys
gpg --list-secret-keys

# Export public key
gpg --armor --export john.doe@example.com > john_pub.asc

# Export private key (careful!)
gpg --armor --export-secret-keys john.doe@example.com > john_priv.asc

# Import a key
gpg --import alice_pub.asc

# Delete a key (public)
gpg --delete-key john.doe@example.com

# Delete secret key
gpg --delete-secret-key john.doe@example.com
```

**Notes:**

* `--armor` produces ASCII-armored output suitable for emails or sharing
* Keys are identified by **name, email, or key ID**

---

## **Encrypting & Decrypting Files**

```bash
# Encrypt a file for a recipient
gpg --encrypt --recipient alice@example.com secret.txt

# Output: secret.txt.gpg

# Decrypt a file
gpg --decrypt secret.txt.gpg > secret.txt

# Encrypt for multiple recipients
gpg --encrypt --recipient alice@example.com --recipient bob@example.com secret.txt
```

---

## **Signing & Verifying**

```bash
# Create a detached signature
gpg --detach-sign file.txt
# Output: file.txt.sig

# Verify signature
gpg --verify file.txt.sig file.txt

# Sign & encrypt in one step
gpg --sign --encrypt --recipient alice@example.com file.txt

# Check signed message
gpg --decrypt signed_encrypted_file.gpg
```

**Notes:**

* Detached signatures are useful for verifying files without encrypting them
* `--clearsign` creates a signed text file readable without decryption

---

## **Key Servers**

```bash
# Send public key to keyserver
gpg --send-keys --keyserver keyserver.ubuntu.com <key-id>

# Receive a key from keyserver
gpg --recv-keys --keyserver keyserver.ubuntu.com <key-id>
```

**Tip:** Use a reliable keyserver such as `keys.openpgp.org` for better privacy.

---

## **Revocation & Expiration**

```bash
# Create a revocation certificate (useful if key is lost/compromised)
gpg --output revoke.asc --gen-revoke john.doe@example.com

# Set key expiration
gpg --edit-key john.doe@example.com
# then use 'expire' command inside GPG prompt

# Extend expiration
gpg --edit-key john.doe@example.com
# 'expire' → new duration → save
```

---

## **Advanced Options**

```bash
# Encrypt using symmetric cipher (no public key)
gpg -c file.txt
# Decrypt with password
gpg file.txt.gpg

# Use specific algorithm for encryption
gpg --cipher-algo AES256 -c file.txt

# List fingerprints
gpg --fingerprint john.doe@example.com

# Export keys with fingerprints
gpg --export --armor <fingerprint>
```

---

## **Integration**

* **Git commits**: `git commit -S -m "Signed commit"`
* **Email encryption**: Integrate with Thunderbird + Enigmail or Outlook GPG plugins
* **Automation**: Scripts can use `gpg --batch` for unattended encryption/decryption

---

## **Security Best Practices**

* Use strong keys: **ed25519 or RSA 4096**
* Keep private keys secure, preferably with **hardware token (YubiKey)**
* Use **passphrase protection**
* Regularly rotate keys
* Use detached signatures for public sharing
* Revoke old or compromised keys immediately

---

## **Troubleshooting**

```bash
# Verbose output
gpg --verbose --decrypt file.txt.gpg

# Check key validity
gpg --check-sigs john.doe@example.com

# Import errors
gpg --import <keyfile> --allow-secret-key-import

# Trust issues
gpg --edit-key john.doe@example.com
# then 'trust' → select level
```

