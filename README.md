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

- VLAN 10 (Management): Administrative traffic for ESXi hosts, vCenter Appliance, and pfSense WebGUI.

- VLAN 20 (iSCSI): Isolated, non-routed segment dedicated to high-speed block storage traffic.

- VLAN 30 (Servers): Production segment for guest VMs with dedicated DHCP pools and firewall policies.
  
---

## 3. Storage Architecture (TrueNAS SCALE)

### 3.1 iSCSI Shared Storage

- Protocol: iSCSI (Block-level).

- Implementation: Configured ZFS Zvols as dedicated LUNs for ESXi Datastores to enable cluster mobility.
![TrueNAS](https://github.com/abd-alhusseini/Project-4/raw/main/photo%20project%204/TrueNAS.png)

### 3.2 Backup & Snapshots

- Mechanism: Periodic ZFS Snapshots.

- Validation: Verified point-in-time recovery for critical virtual machine data.
  ![ZFS](https://github.com/abd-alhusseini/Project-4/raw/main/photo%20project%204/ZFS%20snapshots.png)

---
## 4. Compute & High Availability (vCenter)

### 4.1 VMware vMotion

- Test: Successfully performed live migration of running VMs between ESXi hosts with zero downtime.

- Status: Verified and functional across the dedicated management VMkernel.
  
 [▶ Watch the Live vMotion Demo](https://abd-alhusseini.github.io/Project-4/demo.html) 

### 4.2 vSphere High Availability (HA)

- Configuration: Automated VM restart on healthy nodes triggered by host failure simulation.
  
- Persistence: Leverages TrueNAS shared storage to maintain VM disk state during failover.
  
[▶ Watch the Live High Availability HA Demo](https://abd-alhusseini.github.io/Project-4/HA.html) 

---

## 5. Technical Troubleshooting: VLAN Tag Stripping

### 5.1 The Problem

Guest VMs on VLAN 30 were unable to obtain DHCP leases or ping the gateway (192.168.30.1), despite correct logical configuration in vCenter and pfSense.

![not obtain DHCP](https://github.com/abd-alhusseini/Project-4/raw/main/photo%20project%204/%D9%85%D9%88%20%D8%B1%D8%A7%D8%B6%D9%8A%20%D9%8A%D8%B3%D8%AD%D8%A8%20ip%20.png)

### 5.2 Root Cause Analysis

- Environment: Nested virtualization (ESXi nodes running inside VMware Workstation).

- Cause: The physical NIC driver on the Windows Host OS performs VLAN Tag Stripping. It removes the 802.1Q tags from frames leaving the virtual switch before they reach the pfSense VM.

- Configuration Audit: Verified that ESXi Port Groups were correctly tagged with VLAN ID 30, confirming the configuration was sound.

  ![Verification](https://github.com/abd-alhusseini/Project-4/raw/main/photo%20project%204/static%20ip%20vlan30.png)

### 5.3 Verification Steps

- Firewall Policy: Implemented a permissive "Any-to-Any" rule on the VLAN 30 interface to rule out ACL/Firewall blocking.
  
  ![Rules Vlan](https://github.com/abd-alhusseini/Project-4/raw/main/photo%20project%204/Rule%20Vlan30.png)

  - DHCP Scope Validation: Confirmed the DHCP service was active with the correct pool range (192.168.30.50 - 100).

    ![DHCP Service](https://github.com/abd-alhusseini/Project-4/raw/main/photo%20project%204/vlan30%20dhcp.png)

- Static IP Test: Manual IP assignment (192.168.30.51) failed to establish Layer 2 adjacency. Note: The successful ping seen in some tests was achieved by temporarily bypassing the nested host limitation to verify the logic.

![Static IP Test](https://github.com/abd-alhusseini/Project-4/raw/main/photo%20project%204/ping%20vlan30%20gateway.png)

---

## 6. Technologies & Tools Used

- Hypervisors: VMware ESXi (Cluster), VMware Workstation (Host).

- Orchestration: VMware vCenter Server.

- Storage: TrueNAS SCALE (iSCSI, ZFS).

- Networking: pfSense (802.1Q VLANs, DHCP, Firewall).

- Operating Systems: Ubuntu Server, Alpine Linux.
