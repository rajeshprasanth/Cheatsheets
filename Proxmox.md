# **Proxmox Complete Cheat Sheet**

## **Table of Contents**

1. [Overview](#overview)
2. [VM Management (`qm`)](#vm-management-qm)

   * [Create & Configure VM](#create--configure-vm)
   * [Start, Stop, Reset](#start-stop-reset)
   * [Snapshots](#snapshots)
   * [Migrate & Clone](#migrate--clone)
   * [Resource Limits](#resource-limits)
3. [Container Management (`pct`)](#container-management-pct)

   * [Create & Configure Container](#create--configure-container)
   * [Start, Stop, Restart](#start-stop-restart)
   * [Snapshots & Backups](#snapshots--backups)
   * [Resource Limits](#resource-limits-1)
4. [Backup & Restore (`vzdump`)](#backup--restore-vzdump)
5. [Storage Management](#storage-management)
6. [Networking](#networking)
7. [Cluster & HA Management](#cluster--ha-management)
8. [Logs & Monitoring](#logs--monitoring)
9. [Advanced Tasks & Troubleshooting](#advanced-tasks--troubleshooting)
10. [Best Practices](#best-practices)

---

## **Overview**

* Proxmox VE supports **KVM VMs** and **LXC containers**.
* Main CLIs:

  * `qm` → VM management
  * `pct` → LXC container management
  * `vzdump` → Backup/restore
* Supports **clusters, high availability (HA), storage pools, snapshots, and advanced networking**.

---

## **VM Management (`qm`)**

### **Create & Configure VM**

```bash
qm create 100 --name MyVM --memory 4096 --cores 2 --net0 virtio,bridge=vmbr0 --ide2 local:cloudinit
qm set 100 --scsi0 local-lvm:32   # Add disk
qm set 100 --boot c --bootdisk scsi0
qm set 100 --agent 1               # Enable QEMU guest agent
```

---

### **Start, Stop, Reset**

```bash
qm start 100
qm shutdown 100
qm stop 100
qm reset 100
```

---

### **Snapshots**

```bash
qm listsnapshot 100
qm snapshot 100 snap1 --description "Before upgrade"
qm rollback 100 snap1
qm delsnapshot 100 snap1
```

---

### **Migrate & Clone**

```bash
qm migrate 100 proxmox-node2
qm clone 100 101 --name MyVM-Clone --full
```

---

### **Resource Limits**

```bash
qm set 100 --memory 8192 --cores 4 --cpu units=2000
qm set 100 --balloon 4096               # Enable dynamic memory ballooning
qm set 100 --numa 1                     # Enable NUMA
```

---

### **Advanced VM Config**

```bash
qm set 100 --virtio0 local-lvm:32      # Add extra disk
qm set 100 --ide0 iso/ubuntu.iso,media=cdrom
qm set 100 --hotplug disk,network,usb
qm set 100 --boot order=virtio0
```

---

## **Container Management (`pct`)**

### **Create & Configure Container**

```bash
pct create 200 local:vztmpl/debian-12-standard_12.0-1_amd64.tar.gz \
  --hostname MyCT --cores 2 --memory 2048 \
  --net0 name=eth0,bridge=vmbr0,ip=192.168.1.200/24,gw=192.168.1.1

pct set 200 --password secret123
pct set 200 --features nesting=1,keyctl=1
```

---

### **Start, Stop, Restart**

```bash
pct start 200
pct stop 200
pct restart 200
pct enter 200         # Shell into container
```

---

### **Snapshots & Backups**

```bash
pct listsnapshot 200
pct snapshot 200 snap1
pct rollback 200 snap1
vzdump 200 --storage local --mode snapshot --compress zstd
```

---

### **Resource Limits**

```bash
pct set 200 --memory 4096 --swap 1024
pct set 200 --cores 4
pct set 200 --cpuunits 2000
pct set 200 --rootfs local-lvm:10
```

---

## **Backup & Restore (`vzdump`)**

```bash
# Backup VM or CT
vzdump 100 --storage local --mode snapshot --compress zstd
vzdump 200 --storage local --mode snapshot --compress zstd

# Restore VM
qmrestore /var/lib/vz/dump/vzdump-qemu-100-20250820-0100.vma.zst 100

# Restore container
pct restore 200 /var/lib/vz/dump/vzdump-lxc-200-20250820-0100.tar.zst
```

---

## **Storage Management**

```bash
pvesm status
pvesm add lvmthin local-lvm --vgname pve --thinpool data
pvesm remove local-lvm
qm set 100 --scsi1 local-lvm:32
pct set 200 --rootfs local-lvm:10
```

---

## **Networking**

```bash
# List bridges
brctl show

# Add VM NIC
qm set 100 --net1 virtio,bridge=vmbr1

# Configure container network
pct set 200 --net0 name=eth0,bridge=vmbr0,ip=dhcp
```

---

## **Cluster & HA Management**

```bash
# View cluster status
pvecm status

# Join node to cluster
pvecm add <cluster-ip>

# Remove node from cluster
pvecm delnode <node-name>

# List HA resources
ha-manager status

# Enable HA for VM
ha-manager set <vmid> --max_restart 3 --state started

# Migrate HA VM
ha-manager migrate <vmid> <target-node>
```

---

## **Logs & Monitoring**

```bash
journalctl -f
pveperf
tail -f /var/log/pve/tasks/<task-id>.log
qm monitor 100   # QEMU monitor for VM
pct monitor 200  # LXC monitoring
```

---

## **Advanced Tasks & Troubleshooting**

```bash
# Check disk usage
df -h

# Check running VMs and CTs
qm list
pct list

# VM console
qm terminal 100
pct console 200

# Network test
ping 192.168.1.1
traceroute 192.168.1.1

# Check HA status
ha-manager status

# Repair cluster quorum
pvecm expected 3
```

---

## **Best Practices**

1. **Backup regularly** using `vzdump`.
2. Use **snapshots for testing**, not long-term.
3. Monitor **resource usage** to prevent overload.
4. Use **bridged networking** for external access.
5. Enable **nested virtualization** if needed.
6. Automate repetitive tasks with **CLI scripts**.
7. Maintain **cluster quorum and HA configurations**.


