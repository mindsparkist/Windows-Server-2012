Here is a guide to getting Windows Server 2012 R2 up and running, tailored for someone heading into a DevOps mindset.

---

## 1. Pre-Installation Checklist

Before you begin, ensure your hardware (or Virtual Machine) meets these minimum requirements, I recommend over-provisioning slightly to handle containers or automation agents.

* **Processor:** 1.4 GHz 64-bit processor.
* **RAM:** 512 MB (Minimum), but **4 GB** is recommended for a smooth GUI experience.
* **Disk Space:** 32 GB (Minimum), but **60 GB+** is better for logs and tools.

---

## 2. The Installation Process

The installation is straightforward, but the **choice of version**.

1. **Boot from Media:** Boot your machine using the ISO or DVD.
2. **Select Language:** Choose your language, time, and keyboard preferences.
3. **Install Now:** Click the "Install Now" button.
4. **Pick Your Version:**
* **Server Core:** No GUI. Managed via command line/PowerShell."
* **Server with a GUI:** Standard desktop experience. Better for initial learning.


5. **Installation Type:** Choose **Custom: Install Windows only (advanced)** for a fresh install.
6. **Partitioning:** Select the drive where you want to install the OS and click "Next."

---

## 3. Post-Installation Configuration

Once the server reboots, you’ll be prompted to set an Administrator password. After logging in, follow these steps to prep the server.

### A. Initial Server Setup

Use the **Server Manager** dashboard that pops up automatically:

* **Computer Name:** Give it a meaningful name (e.g., `DEV-SRV-01`).
* **IP Address:** Assign a static IP. DevOps tools (like Jenkins or Ansible) need a consistent address to find your server.
* **Windows Updates:** Even though it's older, ensure all security patches are installed.

### B. Enabling Remote Management (WinRM)

In DevOps, we rarely "log in" to a desktop. We control servers via code. You must enable **WinRM** so tools like Ansible or Terraform can talk to this machine.

* Open PowerShell as Administrator and run:
`winrm quickconfig`

### C. Install the "DevOps Starter" Features

You will likely need these roles for your open-source projects:

1. **IIS (Web Server):** If you plan to host .NET apps.
2. **PowerShell Framework:** Ensure you are running at least PowerShell 5.1 for better compatibility with modern scripts.



---
### Roles and Features
Roles define what a server is (its primary identity or job), while Features are the specific tools or capabilities that help those roles function.

---

![Windows Server 2012 R2 Principles](https://github.com/mindsparkist/Windows-Server-2012/blob/01_-_Key_Windows_Server_2012_R2_Principles/image%20(1).png?raw=true)

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
