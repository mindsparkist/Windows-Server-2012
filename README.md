### 1. The DORA Process

When a client (like a fresh VM) boots up and is set to "Obtain an IP address automatically," it goes through a four-step handshake known as **DORA**.

* **Discover:** The client broadcasts a message to the entire network: *"I need an IP address! Is there a DHCP server out there?"*
* **Offer:** The DHCP server hears the request and broadcasts back: *"I have an IP (192.168.1.50) available for you. Do you want it?"*
* **Request:** The client responds: *"Yes, please! I would like to use 192.168.1.50."*
* **Acknowledgment (ACK):** The server sends the final confirmation: *"Great! You have it for the next 8 days. Here is your Subnet Mask and Gateway too."*

---

### 2. Steps to Install DHCP on Windows Server 2012 R2

Installing DHCP is a two-part process: installing the role and then "Authorizing" it in Active Directory so it can safely provide IPs without being blocked as a "Rogue" server.

#### **Step 1: Install the Role**

1. Open **Server Manager** > **Add Roles and Features**.
2. Select **DHCP Server** from the list.
3. Click through the wizard and select **Install**.

#### **Step 2: Post-Deployment Configuration**

1. Click the **Yellow Warning Flag** in Server Manager.
2. Click **Complete DHCP Configuration**.
3. Provide your **Domain Admin credentials** to "Authorize" the server in AD.

#### **Step 3: Create a Scope**

A "Scope" is the range of IPs the server is allowed to give out.

1. Open **DHCP Manager** (`dhcpmgmt.msc`).
2. Right-click **IPv4** > **New Scope**.
3. **Name:** Give it a name like "Lab-Network."
4. **IP Address Range:** Set your Start (e.g., `192.168.1.100`) and End (e.g., `192.168.1.200`).
5. **Exclusions:** Add any IPs in that range you want to keep for static servers.
6. **Lease Duration:** Default is 8 days (standard for office environments).
7. **Scope Options:** Add your **Default Gateway** (003) and **DNS Servers** (006).

---

### 3. Key Concepts in DHCP Management

| Term | DevOps Use Case |
| --- | --- |
| **Lease** | The amount of time a VM is allowed to keep an IP before it must ask for a renewal. |
| **Reservation** | A "fixed" IP assigned to a specific **MAC Address**. The VM uses DHCP, but it always gets the same IP (useful for printers or app servers). |
| **Exclusion** | Preventing the DHCP server from giving out specific IPs (like the ones you used for your Domain Controller). |
| **DHCP Relay** | Used if your clients are on a different subnet than your DHCP server; it "relays" the DORA broadcast to the server. |

---

Managing a DHCP server in Windows Server 2012 R2 is a core infrastructure task. It ensures that as your DevOps lab grows, IP address management remains automated and error-free.

---

### 1. Creating and Managing DHCP Scopes

A **Scope** is a range of IP addresses that the DHCP server can assign to clients on a specific subnet.

* **Creation:** Right-click **IPv4** in the DHCP Manager (`dhcpmgmt.msc`) and select **New Scope**. You define the start and end IP addresses, the subnet mask, and exclusions (IPs within the range that should not be given out).
* **Scope Options:** These are additional settings sent to the client during the "Acknowledgment" phase of the DORA process.
* **003 Router:** The Default Gateway IP.
* **006 DNS Servers:** The IPs of your Domain Controllers.
* **015 Domain Name:** Your Active Directory domain name (e.g., `corp.contoso.com`).


* **Management:** You can right-click an existing scope to change lease durations or deactivate it if you are performing network maintenance.

---

### 2. Creating DHCP Reservations

A **Reservation** ensures that a specific device (like a printer or a database server) always receives the same IP address from the DHCP server.

* **How it works:** It maps a specific **MAC Address** to a specific **IP Address**.
* **Steps:** 1.  Expand your Scope and right-click **Reservations**.
2.  Select **New Reservation**.
3.  Enter the Name, the desired IP, and the unique MAC address of the device.
* **Benefit:** Unlike a static IP, you can manage the gateway and DNS settings for that device centrally from the DHCP console.

---

### 3. DHCP Failover (Replication)

In Windows Server 2012 R2, "DHCP Failover" replaced the old "Split-Scope" method. It allows two DHCP servers to share lease information so that if one fails, the other can continue serving the network.

* **Modes:**
* **Load Balance:** Both servers active at the same time (usually 50/50).
* **Hot Standby:** One server is active; the other sits idle until a failure occurs.


* **Configuration:** Right-click a scope and select **Configure Failover**. You point it to your partner DC/DHCP server and set a **Shared Secret** (password) for the replication.

---

### 4. DHCP Database Location and Backup

The DHCP server stores its leases in a specialized database file.

* **Database Location:** `%SystemRoot%\System32\dhcp\` (The main file is `dhcp.mdb`).
* **Automatic Backup:** By default, Windows backs up the DHCP database every **60 minutes** to `%SystemRoot%\System32\dhcp\backup`.
* **Manual Backup:** In the DHCP console, right-click the server name and select **Backup**. This is vital before performing a migration or a major configuration change.

---

### 5. PowerShell: Import/Export DHCP Server

For a DevOps engineer, managing DHCP through the CLI is much more efficient, especially when moving settings between servers.

**Exporting the Configuration:**
This command exports all scopes, reservations, and settings to a single file.

```powershell
Export-DHCPServer -File "C:\Exports\DHCPConfig.xml" -Force

```

**Importing to a New Server:**
This allows you to stand up a replacement DHCP server in seconds.

```powershell
Import-DHCPServer -File "C:\Exports\DHCPConfig.xml" -BackupPath "C:\DHCPBackup\"

```

* **Note:** If the new server has a different IP than the old one, you may need to adjust your "DHCP Relay" settings on your routers.

---

### Summary Checklist

| Task | Tool | Why it matters |
| --- | --- | --- |
| **Scope Creation** | `dhcpmgmt.msc` | Defines the IP "pool" for your VMs. |
| **Reservation** | `dhcpmgmt.msc` | Keeps specific servers on the same IP. |
| **Failover** | Replication Wizard | High Availability (No downtime if a DC crashes). |
| **Migration** | `Export-DHCPServer` | Moves your lab configuration to a new host easily. |
