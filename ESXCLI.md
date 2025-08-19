# **VMware ESXi Cheat Sheet**

## **Table of Contents**

1. [Overview](#overview)
2. [ESXi CLI (`esxcli`)](#esxi-cli-esxcli)

   * [Host Info](#host-info)
   * [Network](#network)
   * [Storage](#storage)
   * [VM Management](#vm-management)
   * [Services](#services)
   * [Hardware](#hardware)
   * [Troubleshooting](#troubleshooting)
3. [govc CLI](#govc-cli)

   * [Setup](#setup)
   * [VM Operations](#vm-operations)
   * [Host & Datastore Info](#host--datastore-info)
   * [Automation & Scripting](#automation--scripting)
4. [Best Practices](#best-practices)

---

## **Overview**

* VMware ESXi is a **bare-metal hypervisor**.
* `esxcli` provides **host-level CLI management**.
* `govc` is a **vSphere automation CLI**.
* Useful for **VM management, networking, storage, monitoring, and scripting**.

---

## **ESXi CLI (`esxcli`)**

### **Host Info**

```bash
# Show ESXi version
esxcli system version get

# Show build number
vmware -v

# Host uptime
esxcli system uptime get

# Hardware summary
esxcli hardware summary get
```

---

### **Network**

```bash
# List physical NICs
esxcli network nic list

# Show NIC statistics
esxcli network nic stats get -n vmnic0

# Configure VMkernel IP
esxcli network ip interface ipv4 set -i vmk0 -I 10.0.0.10 -N 255.255.255.0 -t static

# Display VMkernel interfaces
esxcli network ip interface list

# Add VMkernel adapter
esxcli network ip interface add -i vmk1 -p vSwitch0

# Remove VMkernel adapter
esxcli network ip interface remove -i vmk1

# List open network connections
esxcli network connection list
```

---

### **Storage**

```bash
# List all datastores
esxcli storage filesystem list

# Mount a datastore
esxcli storage filesystem mount -l Datastore1

# Unmount a datastore
esxcli storage filesystem unmount -l Datastore1

# List storage devices
esxcli storage core device list

# Rescan HBA
esxcli storage core adapter rescan --all

# List iSCSI sessions
esxcli iscsi session list

# Check multipathing status
esxcli storage nmp device list
```

---

### **VM Management**

```bash
# List all VMs
vim-cmd vmsvc/getallvms

# Get VM power state
vim-cmd vmsvc/power.getstate <vmid>

# Power on VM
vim-cmd vmsvc/power.on <vmid>

# Power off VM
vim-cmd vmsvc/power.off <vmid>

# Reset VM
vim-cmd vmsvc/power.reset <vmid>

# Suspend VM
vim-cmd vmsvc/power.suspend <vmid>

# Get VM info
vim-cmd vmsvc/get.summary <vmid>

# List VM snapshots
vim-cmd vmsvc/snapshot.get <vmid>

# Create VM snapshot
vim-cmd vmsvc/snapshot.create <vmid> "Snapshot Name" "Description" 0 0

# Revert to snapshot
vim-cmd vmsvc/snapshot.revert <vmid> <snapshotid>
```

---

### **Services**

```bash
# List all services
esxcli system services list

# Start a service
esxcli system services start --service=hostd

# Stop a service
esxcli system services stop --service=hostd

# Restart management agents (hostd & vpxa)
services.sh restart

# Enable/disable autostart services
esxcli system startup set --enabled true --service=hostd
```

---

### **Hardware**

```bash
# CPU info
esxcli hardware cpu list

# Memory info
esxcli hardware memory get

# PCI devices
esxcli hardware pci list

# Storage controllers
esxcli storage core adapter list
```

---

### **Troubleshooting**

```bash
# Check system logs
tail -f /var/log/vmkernel.log
tail -f /var/log/hostd.log
tail -f /var/log/vpxa.log

# Monitor resource usage
esxtop

# Network ping test
vmkping <ip-address>

# Test DNS resolution
nslookup <hostname>
```

---

## **govc CLI**

### **Setup**

```bash
export GOVC_URL='https://vcenter.local/sdk'
export GOVC_USERNAME='administrator@vsphere.local'
export GOVC_PASSWORD='your_password'
export GOVC_INSECURE=1
```

---

### **VM Operations**

```bash
# List all VMs
govc ls /DC0/vm

# Power on a VM
govc vm.power -on "MyVM"

# Power off a VM
govc vm.power -off "MyVM"

# Create VM
govc vm.create -m 2048 -c 2 -g otherGuest64 -disk 20G MyNewVM

# Delete VM
govc vm.destroy "MyVM"

# Take a snapshot
govc snapshot.create -m "Snapshot Name" "MyVM"

# Revert to snapshot
govc snapshot.revert "MyVM" "Snapshot Name"

# Clone a VM
govc vm.clone -vm SourceVM -on=false TargetVM

# Relocate VM to another host or datastore
govc vm.move -vm "MyVM" -ds "Datastore2" -host "esxi01.local"
```

---

### **Host & Datastore Info**

```bash
# List ESXi hosts
govc host.info

# List datastores
govc datastore.info

# List networks
govc network.info

# List resource pools
govc pool.info
```

---

### **Automation & Scripting**

```bash
# List VMs with their power states
govc ls /DC0/vm | xargs -I{} govc vm.info {}

# Snapshot all VMs in a folder
for vm in $(govc ls /DC0/vm/folder); do
    govc snapshot.create "$vm" "AutoSnapshot"
done

# Bulk VM power operations
for vm in $(govc ls /DC0/vm/folder); do
    govc vm.power -off "$vm"
done
```

---

## **Best Practices**

1. Always **backup ESXi configurations** and vCenter.
2. Use `govc` for **automation and scripting**.
3. Monitor **NICs, datastores, and CPU/memory usage**.
4. Restart **management agents cautiously**.
5. Tag VMs consistently for **easier automation**.
6. Use **snapshots for temporary testing** but avoid long-term usage.
7. Ensure **security hardening** (SSH access, firewall, and roles).

