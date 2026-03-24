# Enterprise Virtualization & Shared Storage Infrastructure

## 1. Project Overview
This project demonstrates a hyper-converged infrastructure integrating VMware ESXi for compute, TrueNAS SCALE for iSCSI shared storage, and pfSense for network security. The core focus is validating High Availability (HA) and vMotion in a clustered environment.
---
## 2. Infrastructure & Network Segmentation
### 2.1 IP Addressing & Assignment
- pfSense Firewall (Gateway):

  - Management (VLAN 10): 192.168.10.1

  - iSCSI Storage (VLAN 20): 192.168.20.1

  - Servers/Guest (VLAN 30): 192.168.30.1

- VMware Infrastructure:

  - vCenter Server : 192.168.110.231

  - ESXi Host 01: 192.168.110.8

  - ESXi Host 02: 192.168.110.87

- Storage Node:

  - TrueNAS SCALE: 192.168.20.5

### 2.2 VLAN Planning
- VLAN 10 (Management): Administrative traffic for ESXi, vCenter, and pfSense GUI.

- VLAN 20 (iSCSI): Isolated, non-routed segment for block storage traffic.

- VLAN 30 (Servers): Production segment for guest VMs with dedicated DHCP and firewall rules.
---
## . Storage Architecture (TrueNAS SCALE)

### 3.1 iSCSI Shared Storage
- Protocol: iSCSI (Block-level).

- Implementation: Configured ZFS Zvols as dedicated LUNs for ESXi Datastores to enable cluster mobility.

### Backup & Snapshots
- Mechanism: Periodic ZFS Snapshots.

- Validation: Verified point-in-time recovery for critical virtual machine data.
---
##4. Compute & High Availability (vCenter)

###4.1 VMware vMotion
- Test: Successfully migrated running VMs between hosts with zero packet loss.

- Status: Verified and functional across the management network.

### 4.2 vSphere High Availability (HA)
- Configuration: Automated VM restart on healthy nodes during host failure.

- Persistence: Leverages shared storage for consistent VM state recovery.

---

## 5. Technical Troubleshooting: VLAN Tag Stripping
### 5.1 The Problem
Guest VMs on VLAN 30 could not obtain DHCP leases or ping the gateway (192.168.30.1), despite correct logical configuration.

###5.2 Root Cause Analysis
- Environment: Nested virtualization (ESXi inside VMware Workstation).

- Cause: The physical NIC driver on the Windows Host OS performs VLAN Tag Stripping, removing 802.1Q tags from frames before they reach the pfSense VM.

### 5.3 Verification Steps
- Firewall: Confirmed pfSense rules allow DHCP (UDP 67/68) and ICMP.

- Static IP: Manual assignment (192.168.30.50) confirmed that connectivity remains blocked by host-level tag removal.

- Conclusion: Network design is logically sound; limitation is purely environment-specific (Nested NIC limitation).

---

##6. Technologies & Tools Used
- Hypervisors: VMware ESXi (Cluster), VMware Workstation (Host).

- Orchestration: VMware vCenter Server.

- Storage: TrueNAS SCALE (iSCSI, ZFS).

- Networking: pfSense (802.1Q VLANs, DHCP, Firewall).

- Operating Systems: Ubuntu Server, Alpine Linux.
