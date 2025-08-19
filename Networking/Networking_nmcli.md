# nmcli Command Cheat Sheet

## Basic Information Commands

### Show NetworkManager Status
```bash
nmcli general status
nmcli general hostname
nmcli general permissions
```

### Show Device Information
```bash
nmcli device status               # List all devices
nmcli device show                 # Show detailed device info
nmcli device show eth0            # Show specific device details
nmcli device wifi list            # List available WiFi networks
```

### Show Connection Information
```bash
nmcli connection show             # List all connections
nmcli connection show --active    # Show only active connections
nmcli connection show "MyWiFi"    # Show specific connection details
```

## Connection Management

### Create Connections
```bash
# Ethernet connection
nmcli connection add type ethernet con-name "MyEthernet" ifname eth0

# WiFi connection
nmcli connection add type wifi con-name "MyWiFi" ifname wlan0 ssid "NetworkName"

# WiFi with WPA2 password
nmcli connection add type wifi con-name "SecureWiFi" ifname wlan0 ssid "NetworkName" wifi-sec.key-mgmt wpa-psk wifi-sec.psk "password123"

# Static IP connection
nmcli connection add type ethernet con-name "Static" ifname eth0 ip4 192.168.1.100/24 gw4 192.168.1.1
```

### Modify Connections
```bash
# Set static IP
nmcli connection modify "MyConnection" ipv4.addresses 192.168.1.100/24
nmcli connection modify "MyConnection" ipv4.gateway 192.168.1.1
nmcli connection modify "MyConnection" ipv4.dns "8.8.8.8,8.8.4.4"
nmcli connection modify "MyConnection" ipv4.method manual

# Set DHCP
nmcli connection modify "MyConnection" ipv4.method auto

# Change WiFi password
nmcli connection modify "MyWiFi" wifi-sec.psk "newpassword"

# Set connection to auto-connect
nmcli connection modify "MyConnection" connection.autoconnect yes
```

### Activate/Deactivate Connections
```bash
nmcli connection up "MyConnection"      # Activate connection
nmcli connection down "MyConnection"    # Deactivate connection
nmcli device disconnect eth0            # Disconnect device
nmcli device connect eth0              # Connect device
```

### Delete Connections
```bash
nmcli connection delete "MyConnection"
```

## WiFi Management

### Connect to WiFi
```bash
# Connect to open network
nmcli device wifi connect "NetworkName"

# Connect with password
nmcli device wifi connect "NetworkName" password "mypassword"

# Connect to hidden network
nmcli device wifi connect "HiddenNetwork" password "mypassword" hidden yes
```

### WiFi Hotspot
```bash
# Create hotspot
nmcli device wifi hotspot ifname wlan0 ssid "MyHotspot" password "mypassword"

# Stop hotspot
nmcli connection down Hotspot
```

### WiFi Scanning
```bash
nmcli device wifi list              # List available networks
nmcli device wifi rescan            # Refresh available networks
nmcli -f SSID,SIGNAL device wifi list  # Show only SSID and signal strength
```

## Advanced Configuration

### DNS Configuration
```bash
# Set DNS servers
nmcli connection modify "MyConnection" ipv4.dns "8.8.8.8,1.1.1.1"

# Add additional DNS server
nmcli connection modify "MyConnection" +ipv4.dns "8.8.4.4"

# Remove DNS server
nmcli connection modify "MyConnection" -ipv4.dns "8.8.4.4"
```

### VLAN Configuration
```bash
# Create VLAN connection
nmcli connection add type vlan con-name "VLAN100" ifname eth0.100 dev eth0 id 100
```

### Bridge Configuration
```bash
# Create bridge
nmcli connection add type bridge con-name "MyBridge" ifname br0

# Add interface to bridge
nmcli connection add type ethernet con-name "BridgeSlave" ifname eth0 master br0
```

### Bond Configuration
```bash
# Create bond
nmcli connection add type bond con-name "MyBond" ifname bond0 mode active-backup

# Add slaves to bond
nmcli connection add type ethernet con-name "BondSlave1" ifname eth0 master bond0
nmcli connection add type ethernet con-name "BondSlave2" ifname eth1 master bond0
```

## Useful Options and Formatting

### Output Formatting
```bash
nmcli -t connection show              # Terse output (script-friendly)
nmcli -p connection show              # Pretty output
nmcli -f NAME,TYPE connection show    # Show specific fields only
nmcli --colors yes connection show    # Force colors
```

### Field Selection
```bash
# Common fields for connections
nmcli -f NAME,TYPE,DEVICE connection show

# Common fields for devices
nmcli -f DEVICE,TYPE,STATE device status

# Common fields for WiFi
nmcli -f SSID,MODE,CHAN,RATE,SIGNAL,SECURITY device wifi list
```

## Troubleshooting Commands

### Network Diagnostics
```bash
nmcli general logging              # Show logging info
nmcli networking on                # Enable networking
nmcli networking off               # Disable networking
nmcli networking connectivity     # Check connectivity
```

### Device Management
```bash
nmcli device set eth0 managed yes    # Set device as managed
nmcli device set eth0 managed no     # Set device as unmanaged
nmcli device reapply eth0            # Reapply connection settings
```

### Connection Reload
```bash
nmcli connection reload           # Reload all connections
nmcli connection reload "MyConnection"  # Reload specific connection
```

## Examples

### Complete WiFi Setup
```bash
# Scan for networks
nmcli device wifi list

# Connect to WPA2 network
nmcli device wifi connect "MyNetwork" password "mypassword"

# Verify connection
nmcli connection show --active
```

### Static IP Setup
```bash
# Create static IP connection
nmcli connection add type ethernet con-name "Static-Eth" ifname eth0 \
  ip4 192.168.1.100/24 gw4 192.168.1.1 ipv4.dns "8.8.8.8,8.8.4.4"

# Activate the connection
nmcli connection up "Static-Eth"
```

### Quick Network Status Check
```bash
nmcli device status && echo "---" && nmcli connection show --active
```

## Tips

- Use tab completion for connection names and commands
- Connection names with spaces need quotes: `"My Connection"`
- Use `nmcli -h` for help on any command
- Configuration files are stored in `/etc/NetworkManager/system-connections/`
- Use `nmcli connection clone` to duplicate existing connections
- Always test network changes with a backup connection method available
