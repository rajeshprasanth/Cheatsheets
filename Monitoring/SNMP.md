# SNMP Cheat Sheet

## Basic Commands

### Get & Walk

```bash
# Get single OID value
snmpget -v2c -c public <host> <OID>

# Walk through OID subtree
snmpwalk -v2c -c public <host> <OID>

# Bulk walk (faster, SNMPv2c+)
snmpbulkwalk -v2c -c public <host> <OID>

# Bulk get multiple OIDs at once
snmpbulkget -v2c -c public <host> <OID1> <OID2>
```

### Set Values

```bash
# Set SNMP object value
snmpset -v2c -c private <host> <OID> <TYPE> <VALUE>

# Example: set system location
snmpset -v2c -c private 192.168.1.1 sysLocation.0 s "Server Room"
```

### OID Translation

```bash
# Translate OID name to numeric
snmptranslate -On sysUpTime.0

# Translate numeric OID to name
snmptranslate -IR -On 1.3.6.1.2.1.1.5.0

# Show OID description
snmptranslate -Td sysUpTime.0

# Print entire OID tree
snmptranslate -Tp
```

---

## Quick Information

### System

```bash
# Get hostname
snmpget -v2c -c public <host> sysName.0

# Get uptime
snmpget -v2c -c public <host> sysUpTime.0

# Walk full system info
snmpwalk -v2c -c public <host> system

# Quick summary (uptime, load, interfaces)
snmpstatus -v2c -c public <host>
```

### Interfaces

```bash
# List all interfaces
snmpwalk -v2c -c public <host> ifDescr

# Show interface status (up/down)
snmpwalk -v2c -c public <host> ifOperStatus

# Show incoming traffic counters
snmpwalk -v2c -c public <host> ifInOctets

# Show outgoing traffic counters
snmpwalk -v2c -c public <host> ifOutOctets

# Pretty print interface table
snmptable -v2c -c public <host> ifTable
```

### Storage & Performance

```bash
# Show CPU load per core
snmpwalk -v2c -c public <host> hrProcessorLoad

# Show total memory
snmpwalk -v2c -c public <host> hrMemorySize

# Show storage descriptions
snmpwalk -v2c -c public <host> hrStorageDescr

# Show storage used
snmpwalk -v2c -c public <host> hrStorageUsed

# Disk usage summary (like df)
snmpdf -v2c -c public <host>
```

---

## Useful OIDs

| OID                       | Description                         |
| ------------------------- | ----------------------------------- |
| `.1.3.6.1.2.1.1.1.0`      | sysDescr → System description       |
| `.1.3.6.1.2.1.1.3.0`      | sysUpTime → Device uptime           |
| `.1.3.6.1.2.1.1.5.0`      | sysName → Hostname                  |
| `.1.3.6.1.2.1.1.6.0`      | sysLocation → Location              |
| `.1.3.6.1.2.1.2.2.1.2`    | ifDescr → Interface names           |
| `.1.3.6.1.2.1.2.2.1.8`    | ifOperStatus → Interface state      |
| `.1.3.6.1.2.1.2.2.1.10`   | ifInOctets → RX bytes               |
| `.1.3.6.1.2.1.2.2.1.16`   | ifOutOctets → TX bytes              |
| `.1.3.6.1.2.1.25.3.3.1.2` | hrProcessorLoad → CPU load per core |
| `.1.3.6.1.2.1.25.2.2.0`   | hrMemorySize → Total RAM            |
| `.1.3.6.1.2.1.25.2.3.1.6` | hrStorageUsed → Storage used        |

---

## SNMPv3

```bash
# SNMPv3 walk with authentication + encryption
snmpwalk -v3 -u <user> \
  -l authPriv -a SHA -A <authPass> \
  -x AES -X <privPass> \
  <host> system
```

* `-l` → Security level: `noAuthNoPriv`, `authNoPriv`, `authPriv`
* `-a` → Auth protocol: `MD5` or `SHA`
* `-x` → Encryption: `DES` or `AES`

---

## Troubleshooting

```bash
# Verbose/debug output
snmpwalk -v2c -c public -d <host> system

# Retry with timeout
snmpget -v2c -c public -r 3 -t 5 <host> sysName.0

# Only show values (not OIDs)
snmpwalk -v2c -c public -Oqv <host> sysName
```

---

## Homelab Use Cases

### Routers & Switches

```bash
# Monitor WAN traffic (bytes in/out on interface 1)
snmpget -v2c -c public <router-ip> ifInOctets.1
snmpget -v2c -c public <router-ip> ifOutOctets.1

# Check switch port status
snmpwalk -v2c -c public <switch-ip> ifOperStatus

# Identify switch interfaces
snmpwalk -v2c -c public <switch-ip> ifDescr
```

### Servers & VMs

```bash
# Get CPU load
snmpwalk -v2c -c public <server-ip> hrProcessorLoad

# Get total memory
snmpget -v2c -c public <server-ip> hrMemorySize.0

# Get storage usage
snmpwalk -v2c -c public <server-ip> hrStorageUsed
```

### UPS (Uninterruptible Power Supply)

```bash
# UPS battery charge
snmpget -v2c -c public <ups-ip> .1.3.6.1.2.1.33.1.2.4.0

# UPS battery runtime remaining
snmpget -v2c -c public <ups-ip> .1.3.6.1.2.1.33.1.2.3.0

# UPS input line voltage
snmpget -v2c -c public <ups-ip> .1.3.6.1.2.1.33.1.3.3.1.3.1
```

### Printers (if SNMP enabled)

```bash
# Printer model
snmpget -v2c -c public <printer-ip> sysDescr.0

# Printer status
snmpwalk -v2c -c public <printer-ip> .1.3.6.1.2.1.25.3.5.1.1

# Toner levels
snmpwalk -v2c -c public <printer-ip> .1.3.6.1.2.1.43.11.1.1.9
```
