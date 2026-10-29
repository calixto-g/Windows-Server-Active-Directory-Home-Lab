# Windows Server & Active Directory Home Lab

**Personal learning project | October 2026 | Windows Server 2025 · Windows 11 · Hyper-V**

## Overview

I built a small Windows domain lab on my personal desktop to get hands-on experience with Windows Server, Active Directory, DNS, and basic user support tasks. Before starting, I had no practical experience with Windows Server, Hyper-V, or Active Directory. I followed guided setup steps, learned what the settings were for, and tested the environment as I went.

The lab uses **two virtual machines**: a Windows Server domain controller (**DC01**) and a Windows 11 employee workstation (**CLIENT01**). I used the workstation to test domain authentication and a simple Group Policy restriction.

## Lab environment

| Component | Configuration / purpose |
| --- | --- |
| Host computer | Windows 11 Pro desktop running Microsoft Hyper-V |
| Server VM | **DC01** — Windows Server 2025 Standard Evaluation |
| Client VM | **CLIENT01** — Windows 11 Enterprise Evaluation |
| Active Directory domain | `callix.corp.test` |
| Hyper-V switch | `AD-Lab` — **Internal** virtual switch |
| Server IPv4 | `10.10.10.10/24` |
| Workstation IPv4 | `10.10.10.20/24` |
| Client DNS server | `10.10.10.10` (DC01) |
| Default gateway | None configured on the isolated lab network |
| Server roles | Active Directory Domain Services (AD DS) and DNS Server |

```text
Windows 11 Pro host — Microsoft Hyper-V
                   |
        AD-Lab internal switch
             10.10.10.0/24
                   |
          +--------+--------+
          |                 |
      DC01                 CLIENT01
  Windows Server 2025     Windows 11 Enterprise
  10.10.10.10             10.10.10.20
  AD DS + DNS             Domain-joined workstation
          |                 |
          +--- callix.corp.test ---+
```

I chose an **internal virtual switch** so the lab VMs could communicate with each other and the host without being directly bridged onto my home network. With no gateway configured, the lab was not set up for normal internet routing.

![Hyper-V showing DC01 and CLIENT01](screenshots/01-hyper-v-two-vms.png)

## What I built and tested

### 1. Created the virtual network and configured Windows Server

I used Hyper-V to create the VMs and an internal switch named `AD-Lab`. I attached DC01 to that switch, renamed the Windows Server machine to `DC01`, and manually configured its IPv4 settings.

**Why I did this:** Domain-joined computers need to reliably find the domain controller. A fixed address makes DC01 a consistent DNS and authentication endpoint for the lab.

| DC01 network setting | Value |
| --- | --- |
| IPv4 address | `10.10.10.10` |
| Subnet mask | `255.255.255.0` |
| Preferred DNS | `10.10.10.10` |
| Default gateway | Blank |

I checked the result using `ipconfig /all` and Server Manager.

![DC01 settings and ipconfig verification](screenshots/05-dc01-ipconfig-verification.png)

<details>
<summary><strong>Additional network setup screenshots</strong> — virtual switch, DC01 adapter, and IPv4 settings</summary>

**Created an internal Hyper-V virtual switch named AD-Lab.**

![AD-Lab internal virtual switch configuration](screenshots/02-internal-virtual-switch.png)

**Connected DC01's virtual network adapter to AD-Lab.**

![DC01 network adapter attached to AD-Lab](screenshots/03-dc01-network-adapter.png)

**Entered the static IPv4 and DNS values on DC01.**

![DC01 IPv4 properties showing static addressing and DNS](screenshots/04-dc01-ipv4-configuration.png)

</details>

### 2. Installed Active Directory and created the domain

On DC01, I installed the **Active Directory Domain Services** role, promoted the server to the first domain controller in a new forest, and configured the `callix.corp.test` domain. DNS Server was installed as part of the domain controller setup.

I ran the promotion wizard's prerequisite checks and verified DC01 appeared in **Active Directory Users and Computers → Domain Controllers** afterward.

![DC01 registered under Domain Controllers](screenshots/09-domain-controller-registration.png)

<details>
<summary><strong>Additional Active Directory installation screenshots</strong> — role, new forest, and prerequisites</summary>

**Installed the AD DS role on Windows Server 2025.**

![Active Directory Domain Services role installation](screenshots/06-ad-ds-role-installation.png)

**Selected a new forest for the lab domain.**

![Creating the callix.corp.test forest](screenshots/07-new-ad-forest.png)

**Checked the domain controller promotion prerequisites.**

![AD DS prerequisites passed](screenshots/08-ad-prerequisites-passed.png)

</details>

### 3. Practiced user and group administration

I created a `Company` organizational unit (OU) and added separate OUs for **IT Department**, **Human Resources**, **Sales**, and **Security Groups**. I then created fictional employee accounts and three department security groups:

| Example employee | AD account | Department / group |
| --- | --- | --- |
| James Wilson | `jwilson` | IT Department / `SG-IT` |
| Sarah Johnson | `sjohnson` | Human Resources / `SG-HR` |
| Michael Davis | `mdavis` | Sales / `SG-Sales` |

I practiced the account-management workflow: creating users, reviewing group membership, resetting a user's password, and disabling an account. These examples helped me understand the kinds of basic account requests a help desk technician may handle.

**What I learned:** An OU helps organize objects and apply policies; a security group collects accounts for assigning permissions. **Creating a group does not, by itself, grant access to a resource.**

![Department organizational units](screenshots/10-organizational-units.png)

![IT employee security group membership](screenshots/13-it-security-group-membership.png)

<details>
<summary><strong>Additional user administration screenshots</strong> — creating accounts, groups, and common support actions</summary>

**Created a fictional employee account in the IT Department OU.**

![New Active Directory employee user account](screenshots/11-domain-user-creation.png)

**Created departmental security groups.**

![IT HR and Sales security groups](screenshots/12-security-groups.png)

**Practiced resetting an employee's password.**

![Active Directory password reset dialog](screenshots/14-password-reset-dialog.png)

**Practiced disabling a fictional employee's account.**

![Account disable confirmation](screenshots/15-account-disable-confirmation.png)

</details>

### 4. Configured and tested the Windows 11 workstation

I installed Windows 11 Enterprise Evaluation in a second Hyper-V VM named CLIENT01. After its initial setup, I attached it to `AD-Lab` and configured:

- IP address: `10.10.10.20`
- Subnet mask: `255.255.255.0`
- DNS server: `10.10.10.10` (DC01)
- Default gateway: blank

From CLIENT01, I tested connectivity to DC01 using `ping 10.10.10.10` (**4 replies, 0% packet loss**). I also used PowerShell to confirm that the DNS server returned the expected domain A record and the Active Directory LDAP SRV record pointing to `dc01.callix.corp.test`.

![CLIENT01 resolving AD domain and domain controller service records](screenshots/18-client-domain-dns-and-srv.png)

<details>
<summary><strong>CLIENT01 connectivity evidence</strong> — static address and successful ping to DC01</summary>

![CLIENT01 IPv4 settings and four successful ping replies to DC01](screenshots/17-client-to-server-ping.png)

</details>

### 5. Joined CLIENT01 to the domain and tested employee sign-in

I joined CLIENT01 to `callix.corp.test` using the domain Administrator credentials. Windows displayed the **Welcome to the callix.corp.test domain** confirmation.

I then signed in to CLIENT01 with the fictional IT employee account `jwilson` and ran:

```cmd
whoami
```

The output was:

```text
callix\jwilson
```

This confirmed that the workstation accepted a **domain account**, rather than only a local user account.

![Successful CLIENT01 domain join](screenshots/19-client01-domain-join.png)

![Domain user authentication via whoami](screenshots/20-domain-user-whoami.png)

### 6. Created, tested, and reverted a Group Policy restriction

I created a GPO named **Sales - Restrict Control Panel**, linked it to the **Sales OU**, and enabled **Prohibit access to Control Panel and PC settings** under User Configuration.

I signed in to CLIENT01 as the Sales account `mdavis`, updated Group Policy, and checked the result. Windows blocked Control Panel access, and `gpresult /r /scope user` listed the Sales GPO under **Applied Group Policy Objects**.

After testing, I changed the restriction back to **Not Configured** to restore normal access.

**What I learned:** This policy targeted the Sales OU's **user accounts**. It was not applied just because somebody belonged to the `SG-Sales` security group.

![Control Panel restriction observed on CLIENT01](screenshots/23-sales-gpo-restriction-result.png)

![gpresult confirms Sales policy applied](screenshots/24-sales-gpo-gpresult.png)

<details>
<summary><strong>Additional Group Policy configuration screenshots</strong> — policy link and restriction setting</summary>

**Linked the GPO to the Sales OU.**

![Sales GPO link shown in Group Policy Management](screenshots/21-sales-gpo-link.png)

**Enabled the Control Panel and PC Settings restriction for testing.**

![Sales GPO setting enabled in Group Policy Management Editor](screenshots/22-sales-gpo-setting.png)

</details>

## Troubleshooting and lessons learned

| Issue | What I checked / learned | Result |
| --- | --- | --- |
| DNS lookups initially showed timeouts or an unknown DNS server name | Inspected DNS Manager, checked the Windows DNS service, and tested direct resolution instead of assuming the domain was broken | `Get-Service DNS` showed **Running**; `Resolve-DnsName` returned the domain's `10.10.10.10` A record |
| A `Resolve-DnsName` command was not recognized | I was running a **PowerShell** cmdlet in **Command Prompt** | Opened PowerShell and ran the command successfully |
| Needed to confirm client-to-server connectivity before domain joining | Checked the client's static IP, pinged DC01, and queried DNS A/SRV records | Ping replies, domain lookup, and domain-controller discovery succeeded |
| Regular domain user sign-in initially produced a Remote Desktop sign-in-rights error | Hyper-V's **Enhanced Session** uses Remote Desktop; the lab user did not need remote sign-in privileges | Used a basic VM console session to sign in as the domain employee |

The most useful lesson was to **verify each layer**: VM networking, IP configuration, DNS resolution, domain joining, and user sign-in. A working DNS service and record lookup gave me more confidence than relying on one `nslookup` error message.

![DNS service and A-record verification on DC01](screenshots/16-dns-service-and-a-record.png)

## Results and skills practiced

By the end of the lab, I had a working two-VM Windows domain environment where:

- DC01 hosted Active Directory Domain Services and DNS.
- CLIENT01 could reach and discover DC01 over the internal virtual network.
- I could create and organize domain users and security groups.
- A domain user could sign in successfully on a joined Windows 11 workstation.
- A user-scoped Group Policy could be applied and verified on CLIENT01.

**Introductory hands-on exposure:** Hyper-V, Windows Server 2025, Windows 11 deployment, IPv4 and DNS configuration, AD DS, OUs, users, security groups, domain joining, Group Policy, Command Prompt, PowerShell, and basic troubleshooting.

**Limitations:** This was a guided, single-domain-controller home lab using evaluation operating systems. It did **not** include enterprise-grade redundancy, production security hardening, backups, shared-folder permissions, cloud identity integration, or automated device management.

## Screenshot evidence

All **24 selected screenshots** are linked from this README using repository-relative paths. The most important verification screenshots are displayed directly in the sections above. Additional setup evidence can be expanded using each section's **Additional screenshots** menu.

Each screenshot is stored in the [`screenshots/`](screenshots/) directory. These are images from my own lab work, not stock screenshots.

