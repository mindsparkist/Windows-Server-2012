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

