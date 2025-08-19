# **Linux File Encryption & Decryption Cheat Sheet**

## **Table of Contents**

1. [Overview](#overview)
2. [GPG (GNU Privacy Guard)](#gpg-gnu-privacy-guard)

   * [Symmetric Encryption](#symmetric-encryption-password-only)
   * [Asymmetric Encryption](#asymmetric-encryption-public-private-keys)
   * [Signing & Verifying Files](#signing--verifying-files)
   * [Encrypt Multiple Files / Directories](#encrypt-multiple-files--directories)
   * [Advanced GPG Commands](#advanced-gpg-commands)
3. [OpenSSL](#openssl)

   * [Symmetric Encryption](#symmetric-encryption)
   * [Decrypting Files](#decrypting-files)
   * [Encrypt Multiple Files / Directories](#encrypt-multiple-files--directories-1)
   * [Advanced OpenSSL Commands](#advanced-openssl-commands)
4. [ZIP / Tar Encryption](#zip--tar-encryption)
5. [Automation & Batch Mode](#automation--batch-mode)
6. [Best Practices](#best-practices)
7. [Troubleshooting](#troubleshooting)

---

## **Overview**

* Encrypting files ensures confidentiality.
* Decrypting restores original content.
* Methods: **GPG (asymmetric & symmetric)**, **OpenSSL (symmetric)**, **Archive + encryption**.
* Useful for **personal, enterprise, or homelab use**.

---

## **GPG (GNU Privacy Guard)**

### **Symmetric Encryption (Password Only)**

```bash
# Encrypt a file
gpg -c secret.txt
# Output: secret.txt.gpg
# Prompts for password

# Decrypt the file
gpg secret.txt.gpg
# Prompts for password, restores secret.txt
```

---

### **Asymmetric Encryption (Public / Private Keys)**

```bash
# Encrypt for a recipient using their public key
gpg --encrypt --recipient alice@example.com secret.txt

# Decrypt using your private key
gpg --decrypt secret.txt.gpg > secret.txt
```

* Multiple recipients:

```bash
gpg --encrypt --recipient alice@example.com --recipient bob@example.com secret.txt
```

---

### **Signing & Verifying Files**

```bash
# Create detached signature
gpg --detach-sign secret.txt
# Output: secret.txt.sig

# Verify signature
gpg --verify secret.txt.sig secret.txt

# Sign & encrypt in one step
gpg --sign --encrypt --recipient alice@example.com secret.txt

# Decrypt & verify
gpg --decrypt secret.txt.gpg > secret.txt
```

---

### **Encrypt Multiple Files / Directories**

```bash
# Archive folder and encrypt
tar czf - myfolder/ | gpg -c -o myfolder.tar.gz.gpg

# Decrypt and extract
gpg -d myfolder.tar.gz.gpg | tar xzf -
```

---

### **Advanced GPG Commands**

```bash
# Generate key pair
gpg --full-generate-key

# List keys
gpg --list-keys
gpg --list-secret-keys

# Export public key
gpg --armor --export john.doe@example.com > john_pub.asc

# Export private key
gpg --armor --export-secret-keys john.doe@example.com > john_priv.asc

# Import key
gpg --import alice_pub.asc

# Batch mode (non-interactive)
gpg --batch --yes --passphrase "mypassword" -c secret.txt

# ASCII armored output
gpg --armor -c secret.txt
```

---

## **OpenSSL**

### **Symmetric Encryption**

```bash
# Encrypt with AES-256
openssl enc -aes-256-cbc -salt -in secret.txt -out secret.txt.enc
# Prompts for password
```

### **Decrypting Files**

```bash
# Decrypt file
openssl enc -d -aes-256-cbc -in secret.txt.enc -out secret.txt
# Prompts for password
```

---

### **Encrypt Multiple Files / Directories**

```bash
# Archive folder and encrypt
tar czf - myfolder/ | openssl enc -aes-256-cbc -salt -out myfolder.tar.gz.enc

# Decrypt and extract
openssl enc -d -aes-256-cbc -in myfolder.tar.gz.enc | tar xzf -
```

---

### **Advanced OpenSSL Commands**

```bash
# Use base64 ASCII output
openssl enc -aes-256-cbc -in secret.txt -out secret.txt.enc -a

# Encrypt with a specific password inline (not recommended for production)
openssl enc -aes-256-cbc -in secret.txt -out secret.txt.enc -k "mypassword"

# Check file digest
openssl dgst -sha256 secret.txt
```

---

## **ZIP / Tar Encryption**

```bash
# Encrypt ZIP file with password
zip -e secret.zip secret.txt

# Decrypt ZIP file
unzip secret.zip

# Tar with gzip and OpenSSL encryption
tar czf - secret_dir/ | openssl enc -aes-256-cbc -out secret_dir.tar.gz.enc
```

---

## **Automation & Batch Mode**

```bash
# GPG non-interactive encryption
gpg --batch --yes --passphrase "mypassword" -c secret.txt

# OpenSSL automated encryption
openssl enc -aes-256-cbc -in secret.txt -out secret.txt.enc -k "mypassword"
```

* Useful for scripts, backups, CI/CD pipelines.

---

## **Best Practices**

1. Use **strong keys**: RSA 4096 or ed25519 for GPG.
2. Prefer **AES-256** for symmetric encryption.
3. Keep private keys secure and backed up.
4. Protect passphrases and avoid storing them in plain text.
5. Use detached signatures for file verification.
6. Rotate keys periodically.

---

## **Troubleshooting**

```bash
# Check GPG key validity
gpg --check-sigs john.doe@example.com

# Debug decryption
gpg --verbose --decrypt secret.txt.gpg

# Verify OpenSSL file integrity
openssl dgst -sha256 secret.txt

# Decrypt errors with OpenSSL
# Ensure correct password and cipher (-aes-256-cbc)
```

