# **DNS Cheat Sheet**

## **Table of Contents**

1. [Overview](#overview)
2. [Basic DNS Queries](#basic-dns-queries)

   * [dig](#dig)
   * [nslookup](#nslookup)
   * [host](#host)
3. [Zone Files & Records](#zone-files--records)

   * [A, AAAA, CNAME, MX, TXT, PTR](#a-aaaa-cname-mx-txt-ptr)
   * [SOA & NS Records](#soa--ns-records)
4. [BIND 9 Configuration](#bind-9-configuration)

   * [named.conf Structure](#namedconf-structure)
   * [Zone Configuration](#zone-configuration)
   * [Options & ACLs](#options--acls)
5. [BIND 9 Commands](#bind-9-commands)

   * [Start, Stop, Reload, Check](#start-stop-reload-check)
   * [Statistics & Debugging](#statistics--debugging)
6. [Troubleshooting](#troubleshooting)
7. [Advanced DNS Tools](#advanced-dns-tools)
8. [Best Practices](#best-practices)

---

## **Overview**

* DNS (Domain Name System) resolves **hostnames to IP addresses** and vice versa.
* Common tools: `dig`, `nslookup`, `host`.
* BIND 9 is a widely used **authoritative and recursive DNS server**.
* Homelab setup can include **internal zones (`internal.das`)** and **infrastructure zones (`rphomelab.in`)**.

---

## **Basic DNS Queries**

### **dig**

```bash
# Query A record
dig example.com

# Query specific record type
dig example.com MX

# Query using specific DNS server
dig @8.8.8.8 example.com

# Query reverse PTR record
dig -x 192.168.1.1

# Short answer
dig +short example.com

# Trace resolution path
dig +trace example.com
```

---

### **nslookup**

```bash
# Interactive mode
nslookup
> server 8.8.8.8
> example.com
> set type=MX
> exit

# Non-interactive
nslookup example.com 8.8.8.8
```

---

### **host**

```bash
# Simple query
host example.com

# Query specific type
host -t MX example.com

# Reverse lookup
host 192.168.1.1
```

---

## **Zone Files & Records**

### **A, AAAA, CNAME, MX, TXT, PTR**

```text
@       IN      A       10.0.0.1      ; IPv4
@       IN      AAAA    fd00::1       ; IPv6
www     IN      CNAME   hostname      ; Alias
@       IN      MX 10   mail          ; Mail exchanger
@       IN      TXT     "v=spf1 include:_spf.example.com ~all" ; SPF
1.0.0.10.in-addr.arpa. IN PTR hostname.internal.das ; Reverse PTR
```

---

### **SOA & NS Records**

```text
@       IN      SOA     ns1.example.com. admin.example.com. (
                        2025082001 ; Serial
                        3600       ; Refresh
                        1800       ; Retry
                        604800     ; Expire
                        86400 )    ; Minimum TTL
@       IN      NS      ns1.example.com.
@       IN      NS      ns2.example.com.
```

---

## **BIND 9 Configuration**

### **named.conf Structure**

```text
options {
    directory "/var/named";
    recursion yes;
    allow-query { any; };
    listen-on { any; };
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

---

### **Zone Configuration**

```text
zone "rphomelab.in" {
    type master;
    file "/var/named/rphomelab.in.db";
    allow-update { none; };
};

zone "internal.das" {
    type master;
    file "/var/named/internal.das.db";
    allow-update { none; };
};
```

---

### **Options & ACLs**

```text
acl "trusted" { 10.0.0.0/24; 192.168.1.0/24; };
allow-query { trusted; };
allow-transfer { 192.168.1.2; };  # Secondary DNS
```

---

## **BIND 9 Commands**

### **Start, Stop, Reload, Check**

```bash
# Start BIND
systemctl start named

# Stop BIND
systemctl stop named

# Reload zones without restarting
rndc reload

# Reload specific zone
rndc reload rphomelab.in

# Check zone file syntax
named-checkzone rphomelab.in /var/named/rphomelab.in.db

# Check named.conf syntax
named-checkconf
```

---

### **Statistics & Debugging**

```bash
# Query server statistics
rndc stats

# Enable query logging
logging {
    channel query_log {
        file "/var/log/query.log";
        severity info;
    };
    category queries { query_log; };
};

# Run named in foreground for debugging
named -g -d 3
```

---

## **Troubleshooting**

```bash
# Check if BIND is listening
ss -tulnp | grep named

# Test DNS resolution
dig @localhost example.com

# Test recursive query
dig @localhost www.google.com

# Flush cache (if caching enabled)
rndc flush
```

---

## **Advanced DNS Tools**

```bash
# dnssec-keygen for DNSSEC
dnssec-keygen -a RSASHA256 -b 2048 -n ZONE rphomelab.in

# dnssec-signzone
dnssec-signzone -o rphomelab.in rphomelab.in.db

# drill (alternative to dig)
drill example.com @8.8.8.8
```

---

## **Best Practices**

1. Maintain **separate zones** for internal and external networks.
2. Use **incrementing serial numbers** in SOA after each change.
3. Enable **DNSSEC** for security if exposed externally.
4. Keep **ACLs restrictive** for zone transfers.
5. Regularly **check syntax** using `named-checkconf` and `named-checkzone`.
6. Monitor query logs for unusual activity.

