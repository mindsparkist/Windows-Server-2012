In your journey as a Business Solutions Analyst, understanding storage is critical because it dictates the performance and reliability of your virtual machines and databases. In a Windows Server 2012 R2 environment, you will encounter these four storage types, each serving a specific role in your infrastructure.

---

## 1. Local Storage (DAS)

**Direct Attached Storage (DAS)** refers to digital storage directly connected to the computer accessing it, rather than being accessed over a network.

* **Components:** Hard drives (HDD) or Solid State Drives (SSD) plugged directly into the server’s motherboard or RAID controller.
* **Pros:** Simplest to set up and offers the lowest latency since data doesn't travel over a network.
* **Cons:** Not easily shared between multiple servers, making features like **Live Migration** more complex (requiring "Shared-Nothing" setups).

---

## 2. RAID (Redundant Array of Independent Disks)

RAID is a technology that groups multiple local physical disks into one logical unit to provide **redundancy** (protection against disk failure) or **performance** (speed).

* **RAID 0 (Striping):** Splits data across disks. Fast, but if one disk fails, all data is lost.
* **RAID 1 (Mirroring):** Duplicates data on two disks. Very safe, but you only get 50% of your total capacity.
* **RAID 5 (Striping with Parity):** Requires at least 3 disks. Can survive one disk failure while providing good speed.
* **RAID 10:** A combination of 1 and 0. Offers the best performance and safety but requires at least 4 disks.

---

## 3. NAS vs. SAN

This is a core distinction for enterprise storage management. Both provide network-based storage, but they communicate differently.

### **NAS (Network Attached Storage)**

NAS is a **file-level** storage device. It is essentially a specialized computer dedicated to serving files over a network.

* **Protocols:** Uses **SMB** (Windows) or **NFS** (Linux).
* **Usage:** Think of it as a giant shared "C:" drive or a mapped network folder.
* **Connection:** Connects via standard Ethernet cables.

### **SAN (Storage Area Network)**

A SAN is a **block-level** storage network. It presents storage to the server as if it were a local physical hard drive.

* **Protocols:** Uses **iSCSI** or **Fibre Channel (vFC)**.
* **Usage:** Used for high-performance databases or **Failover Clusters** where multiple servers need "raw" access to the same disk.
* **Connection:** Often requires dedicated high-speed fiber networks or dedicated iSCSI VLANs.

---

## 4. Summary Comparison

| Feature | **NAS** | **SAN** |
| --- | --- | --- |
| **Data Type** | File-level (Files/Folders) | Block-level (LUNs/Disks) |
| **Appearance to OS** | Network Share / Mapped Drive | Local Physical Disk in Disk Management |
| **Speed** | Moderate (Ethernet speed) | High (Dedicated Fabric) |
| **Best For** | File sharing, Backups | Databases, VM clusters, High performance |
| **Complexity** | Low (Easy to manage) | High (Requires Zoning/LUN Masking) |

---
