# Windows Server 2025 Lab — Active Directory Domain Services

## 🎯 Objective

Deploy my first domain controller running **Windows Server 2025** using Hyper-V.

The objective of this lab is to practice:

* preparing a Windows Server system;
* installing Active Directory Domain Services;
* creating a domain;
* DNS;
* SYSVOL and NETLOGON;
* understanding the main components of a domain controller.

---

## 🏗️ Architecture

| Component  | Configuration           |
| ---------- | ----------------------- |
| Hypervisor | Hyper-V on Windows 11   |
| Server     | `LAB-DC01`              |
| OS         | Windows Server 2025     |
| Role       | Domain Controller + DNS |
| Domain     | `Homelab.local`         |
| NetBIOS    | `HOMELAB`               |
| Network    | Internal vSwitch        |

The **Internal vSwitch** allows the virtual machines in the lab to communicate with each other and with the Hyper-V host without being directly connected to the main physical network.

I temporarily used an **External vSwitch** to activate Windows and install server updates.

---

## 🖥️ Server Preparation

VM configuration:

* Generation 2;
* 4 GB of RAM;
* dynamically expanding VHDX disk;
* Windows Server 2025.

Before installing Active Directory:

* renamed the server to `LAB-DC01`;
* configured a static IPv4 address;
* configured the DNS server to point to itself because `LAB-DC01` also hosts the DNS service for the domain.

A domain controller should maintain a stable IP address.

Active Directory relies heavily on DNS to locate domain controllers and the different services available within the domain.

---

## 🧩 AD DS Installation

From **Server Manager**:

`Manage → Add Roles and Features → Active Directory Domain Services`

Installing the **Active Directory Domain Services** role adds the components required for AD DS.

Once the role is installed, the server must then be **promoted to a domain controller**.

---

## 🏢 Domain Creation

Because this is the first domain controller in my lab, I selected:

`Add a new forest`

Domain:

`Homelab.local`

NetBIOS name:

`HOMELAB`

This operation creates a new Active Directory forest as well as the first domain within that forest.

---

## 🌳 Functional Levels

Selected configuration:

* Forest Functional Level: **Windows Server 2025**
* Domain Functional Level: **Windows Server 2025**

Functional Levels determine, among other things, which Active Directory features are available and which versions of Windows Server can be used as domain controllers.

They do not determine which versions of Windows can be used by domain clients or member servers.

I selected **Windows Server 2025** because my environment is new and all future domain controllers in the lab will use Windows Server 2025.

Therefore, I do not need to maintain compatibility with older versions of Windows Server.

Other options:

* DNS Server: enabled;
* Global Catalog: enabled;
* RODC: not used, because the first domain controller of a new domain cannot be a Read-Only Domain Controller.

A **DSRM** password is also configured.

DSRM (*Directory Services Restore Mode*) can be used for certain Active Directory recovery or maintenance operations.

After the prerequisite checks are completed, the server is promoted to a domain controller and then restarted.

---

## 💾 NTDS.dit

The Active Directory database is stored by default in:

`C:\Windows\NTDS\ntds.dit`

It contains, among other things:

* users;
* groups;
* computers;
* Organizational Units;
* domain information.

Each domain controller has its own copy of the Active Directory database.

`ntds.dit` is therefore one of the essential components of a domain controller.

---

## 📂 SYSVOL and NETLOGON

Active Directory also creates the following directory:

`C:\Windows\SYSVOL`

SYSVOL contains, among other things, the files required by **Group Policy Objects (GPOs)** as well as certain scripts used within the domain.

After the server is promoted, two important network shares are available:

* `SYSVOL`
* `NETLOGON`

### SYSVOL

From my domain controller, the share can be accessed using:

`\\LAB-DC01\SYSVOL`

SYSVOL contains files that domain computers and users need to access in order to apply certain Active Directory configurations.

For example, when a GPO is applied to a computer or user, part of its configuration is retrieved from SYSVOL.

These files must be available on the domain controllers so that the same policies can be used throughout the domain.

### NETLOGON

The share can be accessed using:

`\\LAB-DC01\NETLOGON`

NETLOGON is used, among other things, to make **logon scripts** available to domain users.

For example, a script can be executed automatically when a user signs in.

The NETLOGON share exposes, among other things, the `scripts` folder located within the SYSVOL directory structure.

SYSVOL and NETLOGON are created automatically when the server is promoted to a domain controller.

---

## ✅ Verification

After the installation, I verified:

* the presence of the `Homelab.local` domain;
* the Active Directory Domain Services role;
* the DNS role;
* the domain DNS zone;
* the `\\LAB-DC01\SYSVOL` share;
* the `\\LAB-DC01\NETLOGON` share.

---

## 📌 Key Takeaways

This lab helped me understand that:

* a domain controller should use a stable IP address;
* DNS is essential to the operation of Active Directory;
* installing the AD DS role is not enough: the server must then be promoted to a domain controller;
* the first domain controller in a new forest also creates the first domain;
* Functional Levels determine, among other things, which AD DS features are available and which versions of Windows Server can be used as domain controllers;
* `ntds.dit` contains the Active Directory database;
* SYSVOL contains, among other things, the files required by GPOs;
* NETLOGON is used, among other things, to provide logon scripts;
* NETLOGON is linked to the SYSVOL directory structure;
* DNS, `ntds.dit`, SYSVOL, and NETLOGON are important components to understand when deploying a domain controller.

---

## 📚 Sources

* Microsoft Learn — Active Directory Domain Services
* Microsoft Learn — Windows Server 2025
* *Windows Server 2025 Administration* course — Kevin Brown
