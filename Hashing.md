# **Linux File Integrity Cheat Sheet**

## **Table of Contents**

1. [Overview](#overview)
2. [Hashing Files](#hashing-files)

   * [MD5](#md5)
   * [SHA Family](#sha-family)
   * [Other Hashes](#other-hashes)
3. [Checksums](#checksums)

   * [Generate Checksum Files](#generate-checksum-files)
   * [Verify Checksums](#verify-checksums)
4. [File Comparison](#file-comparison)

   * [diff](#diff)
   * [cmp](#cmp)
   * [rsync --checksum](#rsync--checksum)
5. [Filesystem Integrity Tools](#filesystem-integrity-tools)

   * [AIDE](#aide)
   * [Tripwire](#tripwire)
   * [Samhain](#samhain)
6. [Audit & Monitoring](#audit--monitoring)

   * [Auditd](#auditd)
   * [Inotify](#inotify)
7. [Best Practices](#best-practices)
8. [Troubleshooting](#troubleshooting)

---

## **Overview**

* File integrity ensures that files have not been **tampered with or corrupted**.
* Methods include **hashing, checksums, filesystem monitoring, and audit tools**.
* Common in **security-sensitive environments, backups, and DevOps pipelines**.

---

## **Hashing Files**

### **MD5**

```bash
# Generate MD5 hash
md5sum file.txt

# Example output
# d41d8cd98f00b204e9800998ecf8427e  file.txt
```

* Quick but **not secure** against collisions.
* Useful for simple file corruption detection.

---

### **SHA Family**

```bash
# SHA-1
sha1sum file.txt

# SHA-256
sha256sum file.txt

# SHA-512
sha512sum file.txt
```

* **SHA-256 or SHA-512** recommended for security-sensitive integrity checks.

---

### **Other Hashes**

```bash
# BLAKE2 checksum (faster, secure)
b2sum file.txt

# CRC32 checksum
cksum file.txt
```

---

## **Checksums**

### **Generate Checksum Files**

```bash
# Generate SHA256 checksums for all files in directory
sha256sum * > checksums.sha256
```

### **Verify Checksums**

```bash
# Verify checksums
sha256sum -c checksums.sha256

# Output:
# file.txt: OK
# file2.txt: FAILED
```

* Ensures files match previously stored hash values.

---

## **File Comparison**

### **diff**

```bash
# Compare two text files
diff file1.txt file2.txt

# Ignore whitespace differences
diff -w file1.txt file2.txt
```

### **cmp**

```bash
# Compare two files byte by byte
cmp file1.bin file2.bin

# Output: first differing byte
```

### **rsync --checksum**

```bash
# Synchronize directories using checksums
rsync -avc /src/ /dest/
```

* Forces rsync to use **file checksums** instead of modification times.

---

## **Filesystem Integrity Tools**

### **AIDE (Advanced Intrusion Detection Environment)**

```bash
# Initialize database
aide --init
mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Check integrity
aide --check
```

* Monitors files, directories, permissions, and hashes.

---

### **Tripwire**

```bash
# Initialize Tripwire database
tripwire --init

# Check for changes
tripwire --check
```

* Used for **enterprise-level file integrity monitoring**.

---

### **Samhain**

* Monitors **file changes, rootkits, and log tampering**.
* Typically configured via `/etc/samhain/samhainrc`.

---

## **Audit & Monitoring**

### **Auditd**

```bash
# Install
sudo apt install auditd

# Monitor file access
auditctl -w /etc/passwd -p wa -k passwd_changes

# View logs
ausearch -f /etc/passwd
```

### **Inotify**

```bash
# Monitor file or directory in real-time
inotifywait -m /var/log/

# With specific events
inotifywait -m -e modify,create,delete /home/john/data
```

---

## **Best Practices**

1. Use **SHA-256 or SHA-512** for secure hashing.
2. Keep **checksums in a separate secure location**.
3. Schedule **AIDE / Tripwire** checks regularly (cron jobs).
4. Monitor critical system files (`/etc/passwd`, `/etc/shadow`, `/bin`).
5. Combine **hashing, auditing, and monitoring tools** for comprehensive integrity.

---

## **Troubleshooting**

```bash
# Compare current and previous checksum
sha256sum -c old_checksums.sha256

# Check why AIDE reports changes
aide --check | less

# Ensure auditd service is running
systemctl status auditd

# Real-time monitoring not triggering?
# Check inotify watch limits
cat /proc/sys/fs/inotify/max_user_watches
```
