In Windows Server 2012 R2, you have three primary "interface levels" or configuration options. Unlike previous versions, you can actually switch between these levels without reinstalling the entire OS (unless you've removed the binary files for space).

### The Three Configuration Levels

| Configuration Level | Description | Primary Management Tools |
| --- | --- | --- |
| **Server Core** | The "headless" version. No desktop, no Start menu, and no Explorer. It is the most secure and resource-efficient. | PowerShell, Command Prompt, `sconfig.cmd`, Remote Management. |
| **Minimal Server Interface** | A "middle ground" new to 2012. It removes the heavy GUI elements (Internet Explorer, Windows Explorer) but keeps **Server Manager** and the **MMC**. | Server Manager, MMC consoles, and local GUI tools (minus web/file browsers). |
| **Server with a GUI** | The "Full" installation. Includes the complete Windows desktop experience, including the Start screen, IE, and all graphical tools. | Full Desktop Interface, Server Manager, and all standard GUI tools. |

---

### Switching Levels

In 2012 R2, you can use PowerShell to jump between these. For example, to move from Core to the Full GUI, you would run:
`Install-WindowsFeature Server-Gui-Mgmt-Infra, Server-Gui-Shell -Restart`

Choosing between **Server Core** and a **Full Installation** (Server with a GUI) is a strategic decision that depends on your technical comfort, hardware resources, and specific role requirements.

In Windows Server 2012 R2, you have the unique flexibility to switch between these modes using PowerShell without reinstalling the OS.

---

### Comparison Table: Core vs. Full

| Feature | **Server Core** | **Full Installation (GUI)** |
| --- | --- | --- |
| **Primary Interface** | Command Line / PowerShell | Full Windows Desktop |
| **Disk Footprint** | Small (~4 GB smaller) | Large |
| **RAM Usage** | Low (approx. 180 MB at idle) | Higher (approx. 310 MB at idle) |
| **Security** | **High** (Reduced attack surface) | **Lower** (More code = more risk) |
| **Maintenance** | Fewer updates & fewer reboots | Frequent updates for GUI components |
| **Best For...** | Active Directory, DHCP, DNS, Hyper-V | App compatibility, New Admins |

---

### When to Use Each

#### **Use Server Core When:**

* **Security is a Priority:** You are building critical infrastructure like a **Domain Controller** or **DNS Server**.
* **Scaling in the Cloud/VMs:** You want to pack as many Virtual Machines as possible onto a single physical host.
* **DevOps Automation:** You are managing servers via tools like **Ansible**, **Chef**, or **Puppet**. It forces you to write code rather than click buttons.
* **Low Maintenance:** You want to minimize the number of times you have to reboot for Windows Updates.

#### **Use Full Installation (GUI) When:**

* **App Compatibility:** You are running software that *requires* a visual interface to install or function (e.g., some older SQL Server versions or 3rd party apps).
* **Initial Troubleshooting:** You are still learning the environment and need the visual tools to diagnose complex networking or disk issues.
* **Legacy Management:** Your team is not yet comfortable with PowerShell and needs the familiar "Start" menu and MMC consoles.

---

To convert a GUI server to Core in 2012 R2, run this in PowerShell:
`Uninstall-WindowsFeature Server-Gui-Mgmt-Infra, Server-Gui-Shell -Restart`

---
Managing servers in a workgroup (non-domain) environment requires a few extra security "handshake" steps because you don't have Active Directory to handle the trust between machines.

---

### Step 1: Prepare the Remote Server (The one you want to manage)

You must ensure the remote server is ready to listen for management commands.

1. **Enable Remote Management:** Open an elevated PowerShell prompt and run:
`Configure-SMRemoting.exe -enable`
2. **Verify WinRM:** Ensure the WinRM service is running:
`winrm quickconfig` (Type 'y' if prompted).
3. **Check Firewall:** Ensure **Windows Remote Management (HTTP-In)** is allowed through the firewall.

---

### Step 2: Configure the Local Machine (The one running Server Manager)

Your local machine won't trust a non-domain server by default. You must explicitly add the remote server to your **TrustedHosts** list.

1. **Add to TrustedHosts:** In PowerShell (as Administrator), run:
`Set-Item WSMan:\localhost\Client\TrustedHosts -Value "RemoteServerNameOrIP" -Force`
*Note: Use `-Concatenate` if you already have other servers in that list.*
2. **DNS/Hosts File:** If you aren't using a DNS server, add the remote server's IP and name to your local `C:\Windows\System32\drivers\etc\hosts` file so your machine can resolve the name.

---

### Step 3: Add the Server in Server Manager

1. Open **Server Manager**.
2. Click **Manage** > **Add Servers**.
3. Go to the **DNS** tab, type the name of your server, and click the arrow to move it to the **Selected** list. Click **OK**.
4. **Crucial Step (Credentials):** The server will likely show a "Kerberos" or "Access Denied" error because it's trying to use your local login.
* Right-click the server in the **All Servers** list.
* Select **Manage As...**
* Enter the credentials in this format: `RemoteServerName\Administrator` or `Workgroup\Administrator` and the password.



---

### Summary Checklist for Workgroup Management

| Action | Purpose |
| --- | --- |
| **TrustedHosts** | Tells the local PC it's safe to talk to the remote PC without a domain. |
| **WinRM Quickconfig** | Opens the communication "ports" on the remote server. |
| **Manage As** | Passes the local administrator account of the *remote* machine. |


---
In Windows Server 2012 R2, the structure of the installation media has changed significantly compared to older versions like Windows XP or Server 2003. Here is the breakdown of why you don't see that folder and how to execute your imaging plan.

### 1. Where is the i386 folder?

**It doesn't exist anymore.** The `i386` folder was used in 32-bit (x86) legacy versions of Windows. Since **Windows Server 2012 R2 is strictly 64-bit**, that folder has been replaced by the **`sources`** folder.

* All your critical installation files, including the `.wim` files you are looking for, are now located in `X:\sources\`.

---


The Steps of creating a "Golden Image" or "Master Image" process. Here is the step-by-step to go from a fresh install to a reusable duplicate:

#### **Step A: The Build (Install & Customize)**

1. Perform a clean installation of Windows Server 2012 R2.
2. Install all required open-source tools (Git, Docker, etc.) and Windows Updates.
3. **Do not join a domain.** Keep the machine in a workgroup to avoid baking domain-specific SIDs into your image.

#### **Step B: Sysprep (The Generalization)**

Sysprep removes unique identifiers (like the Computer Name and SID) so the image can be duplicated safely.

1. Open Command Prompt as Admin and navigate to: `C:\Windows\System32\Sysprep`.
2. Run: `sysprep.exe /generalize /oobe /shutdown`
* **/generalize:** Removes system-specific data.
* **/oobe:** Forces the next user to see the "Out of Box Experience" (setup wizard).
* **/shutdown:** Crucial—you must capture the image while the OS is "cold" (off).



#### **Step C: Capture (Creating the .WIM)**

Once the machine is off, you need to boot into a **WinPE (Windows Preinstallation Environment)** or use a Windows installation disk to access the command line (Shift + F10 at the setup screen).

Use the **DISM** tool to capture your `install.wim`:

```powershell
dism /Capture-Image /ImageFile:D:\CapturedImage\install.wim /CaptureDir:C:\ /Name:"MyDevOpsImage"

```

* **CaptureDir:** This is your Windows partition (usually `C:\`).
* **ImageFile:** Where you want to save the new file (a USB or network drive).

---

### 3. Difference between Boot.wim and Install.wim

* **boot.wim:** This is the "Engine." It boots the computer and starts the installation environment (WinPE). You rarely need to modify this unless you are adding network drivers for a specific server model.
* **install.wim:** This is the "Payload." It contains the actual Windows OS and your custom DevOps tools. This is the file you just created with DISM.

---

### PXE Boot

Since you are doing open-source work, instead of manually replacing `install.wim` on a USB stick every time, look into **Windows Deployment Services (WDS)** or **MDT (Microsoft Deployment Toolkit)**. These tools allow you to PXE boot (boot over the network) and choose which image to install from a menu.

When you are preparing your open-source DevOps lab, choosing the right installation method for Windows Server 2012 R2 is all about speed and repeatability. Since the `i386` folder no longer exists, you will be working primarily with the `\sources` folder and `.wim` files across all these methods.

### 1. CD / DVD (Physical Media)

This is the "old school" method. You burn the ISO to a physical disk and boot the server from the optical drive.

* **Pros:** Reliable for older hardware.
* **Cons:** Slowest transfer speeds; disks can get scratched or lost.

### 2. USB (Bootable Flash Drive)

The most common method for physical hardware today. You use tools like **Rufus** or the **Windows USB/DVD Download Tool** to "burn" the ISO onto a thumb drive.

* **Pros:** Fast installation; portable.
* **Cons:** Requires physical access to the server; some older BIOS versions struggle to boot from USB.

### 3. VM from ISO (Virtualization)

In a DevOps environment (using Hyper-V, VMware, or VirtualBox), you simply "mount" the ISO file as a virtual DVD drive.

* **Pros:** Instant setup; no physical media needed.
* **Cons:** Requires a host OS already running a hypervisor.

### 4. Network Share (PXE / WDS)

This is the professional DevOps choice. You place the installation files on a **Windows Deployment Services (WDS)** server (Multicast Possible). The target server boots from its network card (PXE boot) and pulls the files over the LAN.

* **Pros:** Install on dozens of servers simultaneously; no physical media.
* **Cons:** Requires a complex initial setup (DHCP, DNS, and WDS server).

### 5. VHD (Native Boot to VHD)

This is a "hidden gem" for testing. You can install Windows directly into a `.vhd` or `.vhdx` file stored on an existing hard drive. You then tell the computer’s bootloader to boot from that file instead of a physical partition.

* **Pros:** Allows "Dual Booting" without re-partitioning your hard drive; easy to delete the whole OS by just deleting one file.
* **Cons:** Slightly more complex to set up using the Command Prompt during installation.

---

### Summary Comparison Table

| Method | Speed | Best For... | Key Tool |
| --- | --- | --- | --- |
| **CD/DVD** | Slow | Legacy Hardware | Optical Drive |
| **USB** | Fast | Single Physical Server | Rufus |
| **VM/ISO** | Instant | Lab Testing / DevOps | Hyper-V / ESXi |
| **Network** | Variable | Bulk Deployments | WDS / PXE |
| **VHD** | Fast | Multi-boot / Testing | Diskpart (Shift+F10) |

**Since you are doing DevOps, would you like the specific "Diskpart" commands to perform a "Native Boot to VHD" installation?**

---

[Windows Server 2012 Step-by-Step Installation](https://www.youtube.com/watch?v=ScSJMfG5R1Y)
This video provides a practical, visual guide to the standard installation process, which serves as the foundation for all the different media types mentioned above.

Installing Windows Server 2012 R2 to a **VHD (Virtual Hard Disk)** is a professional technique often called "Native Boot." This allows you to run the OS from a single file on your physical hard drive without needing a virtual machine or a separate partition.

Since you are setting up an open-source DevOps project, this is perfect for testing different "Golden Images" without messing up your main system.

### Step 1: Launch the Command Prompt

1. Boot from your Windows Server 2012 R2 installation media (USB or ISO).
2. On the first setup screen (Language selection), press **SHIFT + F10**.
3. A black Command Prompt window will appear.

### Step 2: Create the VHD using Diskpart

You need to create a virtual disk file on your physical drive to act as the "hard drive" for your installation.

```cmd
diskpart
list disk
select disk 0  (Ensure this is your primary physical drive)
list partition (Check where you have space; usually C: or D:)

```

Now, create the VHD file. In this example, we'll create a 50GB file on the D: drive:

```cmd
create vdisk file="D:\Win2012.vhd" maximum=51200 type=expandable
attach vdisk
exit

```

* **maximum=51200**: Sets the size to 50GB (50 x 1024).
* **type=expandable**: The file starts small and grows as you add data (saves space).

### Step 3: Proceed with Installation

1. Close the Command Prompt and return to the Setup Wizard.
2. Click **Install Now** and accept the license terms.
3. Choose **Custom: Install Windows only (advanced)**.
4. You will now see a new, unallocated drive in the list (the 50GB VHD you just created).
5. **Important:** You might see a warning saying "Windows cannot be installed to this disk." **Ignore it.**
6. Select that drive and click **Next**. Windows will begin installing normally into that file.

---

### Why use VHD for DevOps?

* **Portability:** You can copy this `.vhd` file to another computer, attach it, and boot from it instantly.
* **Snapshotting:** Before you run a "risky" script or install a new open-source tool, you can simply make a copy of the `.vhd` file. If something breaks, just replace the file with your backup.
* **Dual Boot:** You can keep your existing Windows OS and this Server OS on the same physical drive without the risk of overlapping `\Windows` folders.

---

**Would you like the PowerShell commands to add this VHD to your boot menu so you can choose it every time you restart?**

---

[Installing Windows Server 2012 R2 on a VHD](https://www.youtube.com/watch?v=Bq22mkZJLhQ)

This video provides a practical look at how to manage and mount virtual hard disks within the Windows Server 2012 R2 environment.

In a corporate or DevOps environment, a server isn't "ready" just because the installation finished. To meet production standards, you must follow a rigorous post-installation checklist to ensure security, manageability, and stability.

Here are the essential steps for Windows Server 2012 R2 in a professional setting.

---

## 1. Initial Security & Identity

Before connecting to the broader corporate network, establish the server's identity.

* **Set a Strong Administrator Password:** Ensure it meets corporate complexity policies.
* **Rename the Server:** Use the corporate naming convention (e.g., `HYD-PROD-SQL01`).
* *Path: Server Manager > Local Server > Computer Name.*


* **Configure Static IP:** Servers should never rely on DHCP. Assign a static IP, Subnet Mask, Gateway, and Corporate DNS servers.
* **Disable NIC Teaming (if not needed):** If you have multiple NICs, configure Teaming for redundancy now, before adding roles.

---

## 2. Remote Management Setup (The DevOps Essential)

Modern corporate environments are managed remotely. You must "open the doors" for management tools.

* **Enable Remote Desktop (RDP):** Allow connections only from computers running Remote Desktop with Network Level Authentication (NLA) for security.
* **Configure WinRM (Windows Remote Management):** This is vital for PowerShell Remoting and DevOps tools like Ansible.
* Run: `winrm quickconfig` in PowerShell.


* **Enable Remote Management in Server Manager:** Ensure "Remote Management" is set to **Enabled**.

---

## 3. Patching and Updates

A fresh install of 2012 R2 is missing over a decade of security patches.

* **Run Windows Update:** Install all "Important" and "Critical" updates.
* **Check for Driver Updates:** Ensure chipset, storage controller, and network drivers are the latest versions provided by the hardware vendor (Dell, HP, etc.).
* **Configure WSUS (Optional):** If your company uses Windows Server Update Services, point the server to the internal update server via Group Policy or Registry.

---

## 4. Storage & File System Optimization

* **Initialize Secondary Disks:** Use `Disk Management` to bring data drives online, initialize them as **GPT**, and format them with **NTFS** (or **ReFS** if using Storage Spaces).
* **Label Volumes:** Clearly name drives (e.g., `DATA_01`, `LOGS_01`) so monitoring tools report them correctly.
* **Set Page File:** For corporate stability, manually set the Page File size (typically 1.5x RAM or as per application requirements) on a drive other than `C:` if possible.

---

## 5. Enterprise Hardening (The "Security First" Step)

* **Disable the Guest Account:** Ensure it is disabled (usually is by default).
* **Configure Windows Firewall:** Only open necessary ports. For a Web Server, open 80/443; for a Domain Controller, open 53, 88, 135, etc.
* **Install Antivirus/Endpoint Detection (EDR):** Deploy the corporate standard security agent (e.g., CrowdStrike, Defender, or Symantec).
* **Join the Domain:** Once networking is stable, join the server to the Active Directory domain and move it to the correct **Organizational Unit (OU)** to receive Group Policies.

---

## 6. Corporate Monitoring & Backup

A server is not "production-ready" if it isn't being watched.

* **Install SNMP or WMI Providers:** For monitoring tools like SolarWinds, Nagios, or Zabbix.
* **Install Backup Agents:** Ensure the server is added to the corporate backup schedule (Veeam, Commvault, etc.).
* **Time Synchronization:** Ensure the server is syncing time with the Domain Controller or a reliable NTP source (crucial for Kerberos authentication).

---

### Post-Installation Summary Table

| Category | Action | Why? |
| --- | --- | --- |
| **Identity** | Rename & Static IP | Network consistency. |
| **Access** | Enable RDP & WinRM | Headless management. |
| **Security** | Windows Updates & EDR | Mitigation of vulnerabilities. |
| **Storage** | Format Data Drives | Separation of OS and Data. |
| **Compliance** | Domain Join | Group Policy enforcement. |

---

**Would you like a PowerShell script to automate these post-installation steps (Naming, Static IP, and WinRM) for your project?**

NIC Teaming, also known as **Load Balancing and Failover (LBFO)** in Windows Server 2012 R2, allows you to group multiple physical network adapters into a single logical "team." For your open-source DevOps lab, this is a core concept for achieving high availability.

### Pros of NIC Teaming

* **High Availability (Failover):** If one network card or cable fails, traffic automatically reroutes to the remaining healthy NICs without dropping the connection.
* **Bandwidth Aggregation:** You can combine multiple 1Gbps links into a larger logical pipe (e.g., two NICs = 2Gbps). Note that a single file transfer usually won't exceed the speed of one NIC; the benefit is seen when *multiple* clients connect simultaneously.
* **Switch Diversity:** In "Switch Independent" mode, you can connect each NIC to a different physical switch. This protects you if an entire switch loses power or crashes.
* **VLAN Support:** You can use a single NIC team to handle traffic for multiple VLANs, which is essential for segmenting management, production, and backup traffic in a corporate network.
* **Native Support:** Since it is built into Windows Server 2012 R2, you no longer need buggy third-party drivers from vendors like Broadcom or Intel to create teams.

---

### Cons of NIC Teaming

* **Increased Complexity:** Troubleshooting network issues becomes harder because you have to determine if the problem is with the physical NIC, the virtual team interface, or the switch configuration.
* **Hardware Waste:** In "Active-Standby" configurations, you are paying for hardware (cables, ports, and NICs) that sits idle and does nothing unless a failure occurs.
* **Performance Overhead:** The CPU has to do a small amount of extra work to calculate how to "hash" and distribute traffic across the team members.
* **Incompatibility with iSCSI:** Microsoft explicitly recommends **not** using NIC Teaming for iSCSI storage traffic. Instead, you should use **MPIO** (Multi-Path I/O), which is designed specifically for storage redundancy.
* **Configuration Mismatches:** If you use "Switch Dependent" modes (like LACP) and the network admin configures the switch incorrectly, the entire server can lose connectivity.

---

### Summary Table: When to use which mode?

| Teaming Mode | Best For... | Switch Requirement |
| --- | --- | --- |
| **Switch Independent** | General DevOps/Web servers | Any basic switch (no config needed). |
| **LACP (Link Aggregation)** | High-performance backbone | Managed switch with LACP enabled. |
| **Static Teaming** | Legacy enterprise switches | Manual "Port Channel" config on switch. |

**Would you like the PowerShell command to create a NIC Team with specific load-balancing settings for your server?**

---

[NIC Teaming on Windows Server 2012](https://www.youtube.com/watch?v=m7exmRXRI9Y)

This video explains the benefits of combining network cards for better performance and redundancy, while also covering common pitfalls you might encounter.

Upgrading from Windows Server 2012 or 2012 R2 is a critical step for corporate environments, especially since these versions reached their End of Life in late 2023. You have two main paths: **In-place upgrades** (keeping everything as-is) or **Fresh installations** (migrating data to a clean slate).

---

### 1. Upgrade Requirements

Before you begin, your server must meet these minimum hardware and software standards for modern releases (2019/2022).

* **Processor:** 1.4 GHz 64-bit processor (32-bit is not supported).
* **RAM:** 2 GB minimum (800 MB for VMs during setup).
* **Disk Space:** 32 GB minimum (Corporate installs usually require **60 GB+** to account for the `Windows.old` folder).
* **Current OS:** To upgrade to 2012 R2, you must be on 2012 or 2008 R2 SP1. To move to 2019/2022, you typically need to "hop" through 2016 first.
* **Security:** TPM 2.0 and Secure Boot are required for Server 2022.

---

### 2. When to Upgrade vs. When NOT to

| Use **In-Place Upgrade** When... | Use **Fresh Installation** When... |
| --- | --- |
| You have complex software with "lost" install keys. | You want a "clean slate" free of legacy bugs/junk. |
| You need to minimize downtime and manual setup. | The current OS has performance issues or corruption. |
| The server has fairly standard roles (DNS, IIS). | You are moving from physical hardware to a VM. |
| You have a verified snapshot/backup ready. | You are jumping more than 2 versions ahead. |

---

### 3. Upgrade Paths: Standard vs. Datacenter

Microsoft allows you to move "up" or "across" editions, but never "down."

* **Standard  Standard:** Supported.
* **Standard  Datacenter:** Supported (often used when you want to unlock unlimited virtualization).
* **Datacenter  Datacenter:** Supported.
* **Datacenter  Standard:** **NOT Supported.** You would have to perform a fresh installation.

> **Note:** If you are using an **Evaluation** version, you must convert it to a **Retail** version using `DISM` before you can perform a standard in-place upgrade.

---

### 4. Fresh OS Installation (Clean Install)

This is the "gold standard" for DevOps. Instead of upgrading the old OS, you:

1. **Build** a new VM with a fresh copy of Windows Server 2022.
2. **Install** your open-source DevOps tools (Git, Jenkins, etc.).
3. **Migrate** only the necessary data from the old 2012 server.
4. **Decommission** the old server once the new one is stable.

---

### 5. Essential Upgrade Checklist

1. **Back up everything:** Take a VM snapshot or a full system image.
2. **Disable NIC Teaming:** Always break the team before an in-place upgrade; re-enable it after.
3. **Check disk space:** Ensure you have at least 40 GB free on the `C:` drive.
4. **Uninstall incompatible apps:** Old antivirus or monitoring agents often break upgrades.
5. **Run `setup.exe`:** Launch the installer from within the running OS (do not boot from the ISO for in-place upgrades).

**Would you like me to walk you through the specific PowerShell commands for a "two-hop" upgrade from 2012 to 2022?**

---

[Upgrade Windows Server 2012 to 2012 R2](https://www.youtube.com/watch?v=oEOVG48mSd4)

This video demonstrates the specific steps and prerequisites needed to perform an incremental in-place upgrade between these two versions safely.
