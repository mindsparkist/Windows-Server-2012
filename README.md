Active Directory (AD) is essentially the "phonebook" or "identity engine" for a Windows network. It is a centralized database that stores information about every user, computer, and printer in your organization.

For your open-source DevOps project, understanding AD is vital because it handles the **AAA** of security: **Authentication** (who are you?), **Authorization** (what can you do?), and **Accounting** (what did you do?).

---

### 1. Workgroup vs. Domain Join

Every Windows computer starts in a **Workgroup**. To scale a corporate environment, you must move it to a **Domain**.

* **Workgroup (The "Neighbor" Model):**
* Every computer manages its own **SAM (Security Accounts Manager)** database.
* If you have 10 servers and want to change your password, you have to log into each one individually and change it 10 times.
* There is no central control; machines are "peers."


* **Domain Join (The "Centralized" Model):**
* The computer trusts the **Domain Controller (DC)**.
* You log in with a single identity. The local computer sends your credentials to the DC via **Kerberos** to verify them.



---

### 2. Accessing AD: The LDAP Protocol

Active Directory is built on the **LDAP (Lightweight Directory Access Protocol)**.

* Because AD follows LDAP standards, it is "open-source friendly."
* You can connect your Linux servers, Jenkins instances, or Git servers to Active Directory using LDAP queries to let your users log in with their corporate credentials.

---

### 3. OU vs. Container

When you open **Active Directory Users and Computers (ADUC)**, you will see two types of "folders." While they look similar, they function very differently.

#### **Containers (The "Generic" Folders)**

* **Examples:** `Users`, `Computers`, `Builtin`.
* **Purpose:** These are default folders created by Windows.
* **Limitation:** You **cannot** link a Group Policy Object (GPO) to a container. This is why you should move your servers out of the default `Computers` container and into an OU.

#### **OU (Organizational Units)**

* **Purpose:** Logical folders created by the admin to represent departments (e.g., `Engineering`, `Sales`) or locations (e.g., `Noida`, `Hyderabad`).
* **The Power of OU:** 1.  **GPO Linking:** You can apply specific settings to an OU.
2.  **Delegation:** You can give a specific user (like a Lead Developer) permission to reset passwords *only* within their own OU, without making them a full Domain Admin.

| Feature | Container | Organizational Unit (OU) |
| --- | --- | --- |
| **Icon** | Plain Folder icon | Folder with a small "book" icon |
| **GPO Support** | No | **Yes** |
| **Delegation** | Limited | Full Support |
| **Creator** | System (Default) | Administrator |

---

### 4. Summary for DevOps

In a DevOps lab, your workflow will usually look like this:

1. **Spin up** a fresh VM (Workgroup).
2. **Join Domain** via PowerShell (`Add-Computer -DomainName "yourdomain.me"`).
3. **Move** the computer object from the `Computers` container to a specific `DevOps-Servers` **OU**.
4. **Wait** for Group Policy to automatically configure the firewall and tools on that server.

In Windows Server 2012 R2, Active Directory (AD) is designed as a hierarchical structure. Think of it like a biological classification: individuals live in families, families live in groves, and groves form a forest.

---

### 1. The Hierarchy: Domains, Trees, and Forests

* **Domain:** The smallest logical unit. It is a boundary for security and administration. Users in a domain share the same database and policies.
* **Tree:** A collection of domains that share a **contiguous name space**.
* *Example:* `devops.me` (Parent) and `lab.devops.me` (Child) form a tree.


* **Forest:** The highest level of container. A forest can contain multiple trees that do **not** share a name space but share a common schema (the rules of the database) and a Global Catalog.
* *Example:* `company-a.com` and `company-b.com` can be in the same forest.



---

### 2. Trusts: The "Handshake" Between Domains

Trusts allow users in one domain to access resources in another.

* **Automatic Trust:** When you create a child domain within a tree, a **two-way, transitive trust** is created automatically. The parent trusts the child, and the child trusts the parent.
* **Transitive Trust:** If Domain A trusts Domain B, and Domain B trusts Domain C, then **Domain A trusts Domain C**. It "flows" through the links.
* **Manual (Shortcut/External) Trust:** Created by an admin. You use this to connect two different forests or to create a "shortcut" between two distant branches of a tree to speed up authentication.

---

### 3. Functional Levels: The "Version" of your AD

Functional levels determine which Active Directory features are available in your environment.

| Level | Purpose |
| --- | --- |
| **Domain Functional Level** | Limits the features available within a **specific domain**. All DCs in that domain must run that version of Windows Server or higher. |
| **Forest Functional Level** | Limits features across the **entire forest**. All domains in the forest must be at this level. |

#### **Raising Levels: Pros and Cons**

* **Pros:** Unlocks new features (e.g., Active Directory Recycle Bin, enhanced Kerberos security, and improved replication).
* **Cons:** **Irreversible.** Once you raise the level to 2012 R2, you can never add a Windows Server 2008 R2 Domain Controller to that network again.

> **DevOps Tip:** You can have a 2012 R2 Server running in a 2008 Functional Level (this is common during migrations), but you won't get the new 2012 features until you "Raise" the level.

---

### 4. Steps to Migrate a Domain

In a corporate environment, you rarely "upgrade" an old server. Instead, you perform a **Swing Migration**:

1. **Prepare:** Ensure your existing 2008/2012 forest is healthy (`dcdiag`).
2. **Introduce:** Install a new Windows Server (e.g., 2019/2022) and join it to the existing domain.
3. **Promote:** Use Server Manager to promote the new server to a **Domain Controller**.
4. **Transfer FSMO Roles:** Move the 5 "Flexible Single Master Operation" roles (the "brains" of the domain) to the new server.
5. **Demote:** Run the wizard to remove the DC role from the old 2012 R2 server.
6. **Raise Levels:** Once all old servers are gone, raise the Domain and Forest Functional Levels.

---

### 5. Summary Table

| Term | Analogy | Key Fact |
| --- | --- | --- |
| **Domain** | A Family | Boundary for GPOs and security. |
| **Tree** | Extended Family | Shared name (e.g., sub-domains). |
| **Forest** | The Entire Ecosystem | Shared Schema; absolute security boundary. |
| **Trust** | A Passport | Allows cross-domain access. |
| **Functional Level** | Software Version | Limits the "oldest" server allowed in the mix. |

**Would you like me to list the 5 FSMO roles and explain which one is the most critical for your DevOps lab to stay online?**

---

[Active Directory Infrastructure - Domains, Trees, and Forests](https://www.youtube.com/watch?v=BDbPcGGTYmw)

This video explains the logical structure of AD, focusing on how trees and forests interact and the importance of functional levels during an upgrade.

In Windows Server 2012 R2, the process of becoming a Domain Controller (DC) is a two-step dance: installing the "bits" (the role) and then "promoting" the server.

---

### 1. Promoting a DC: GUI Step-by-Step

In modern Windows Server versions, the old `dcpromo` command is deprecated. If you run it, it will simply tell you to use **Server Manager**.

1. **Add the Role:** Open **Server Manager** > **Add Roles and Features** > Select **Active Directory Domain Services**. Click through and **Install**.
2. **The Promotion:** Once installed, a yellow warning flag appears in Server Manager. Click **Promote this server to a domain controller**.
3. **Deployment Configuration:** * Choose **Add a new forest** (if this is your first server) or **Add a domain controller to an existing domain** (if you are adding a backup/secondary DC).
4. **Domain Controller Options:** Here you select your Functional Levels and ensure **DNS** and **Global Catalog** are checked. Set your **DSRM Password** (keep this safe!).
5. **Paths:** This is where you define the Database, Logs, and SYSVOL.
6. **Prerequisites Check:** The wizard will verify your IP (should be static!) and DNS. Click **Install**. The server will reboot automatically.

---

### 2. Essential Definitions

* **DNS (Domain Name System):** The locator service. It tells computers where the DC is. Without it, you cannot log in.
* **Global Catalog (GC):** A searchable index of **every object in the forest**. It allows a user in one domain to find a printer in another domain within the same forest. At least one DC must be a GC for logins to work.
* **RODC (Read-Only Domain Controller):** A DC used in physically insecure locations (like a branch office). It holds a copy of the AD database but cannot be edited. If someone steals the server, they can't change passwords.
* **DSRM (Directory Services Restore Mode):** A "Safe Mode" for Active Directory. If the AD database gets corrupted, you boot into DSRM using the password you set during installation to perform repairs.

---

### 3. Default Storage Locations

During the wizard, you will see these default paths. In production, admins often move these to a separate drive (like `D:`) for better performance and recovery.

* **Database Folder:** `C:\Windows\NTDS` (Contains `ntds.dit`).
* **Log Files:** `C:\Windows\NTDS` (Transaction logs for the database).
* **SYSVOL:** `C:\Windows\SYSVOL` (Contains GPOs and login scripts; replicated to all DCs).

---

### 4. Adding a DC to an Existing Domain

When you add a second DC for redundancy (highly recommended for your lab):

1. **Point DNS to the first DC:** Your new server must be able to resolve the domain name.
2. **Join the Domain:** Join the server as a member server first.
3. **Run Promotion Wizard:** Choose **"Add a domain controller to an existing domain."**
4. **Replication:** The new DC will pull a copy of the database and SYSVOL from the existing DC over the network.

---

### 5. Advanced Deployment: IFM (Install From Media)

If you are adding a DC in a location with a very slow internet connection, you don't want to sync 10GB of data over the wire. You use **IFM**.

**Using `ntdsutil` to create IFM:**

1. On an **existing** healthy DC, open Command Prompt (Admin).
2. Type `ntdsutil`.
3. Type `activate instance ntds`.
4. Type `ifm`.
5. Type `create full C:\IFM_Data`.
6. **The Result:** Windows creates a "snapshot" of the AD database, registry, and SYSVOL in that folder.
7. **The Benefit:** Copy that folder to a USB drive, take it to the new server, and during the Promotion Wizard, choose **"Install from Media."** The server will only sync the *changes* since the snapshot was taken, saving massive bandwidth.

---

### 6. Restoring and Repopulating

If a DC fails and you need to restore it:

1. **Non-Authoritative Restore:** The default. You restore the VM/Server, and it asks the other DCs, "What did I miss?" and repopulates itself.
2. **Authoritative Restore:** You tell the forest, "My backup is the correct version, everyone else update to match me." (Used if someone accidentally deleted an entire OU and you need it back).




