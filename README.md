# Enterprise Network Security Lab

Virtual enterprise network security lab built with VMware Workstation to simulate a small Windows and Linux environment with Active Directory, file services, Ubuntu Server, and a Kali "security tools" VM. The environment is used to scan, harden, and observe vulnerabilities in a segmented 192.168.10.0/24 network.

> **Status:** Phase 4 complete – core AD/file services built with file auditing (4663 events), server hardening via GPO baselines, and Linux integration with cross-platform authentication and auditing. Documentation for Phases 1–4 is complete. Phase 5 (RMF Asset Inventory and Risk Baseline) and Phase 6 (Nessus + Kali Vulnerability Assessment) are being planned.

## Lab Goals

- Practice **enterprise Windows administration** (AD DS, file server, GPOs, NTFS/share permissions, Group Policy baselines).
- Build and document **realistic access controls** using users, groups, and departmental shares across Windows and Linux platforms.
- Integrate **Linux servers into Windows environments** using Active Directory (realmd/SSSD) for unified identity management.
- Enable **cross-platform file auditing** (Windows 4663 Security events + Linux auditd) for centralized monitoring and SIEM integration.
- Perform **vulnerability assessment and hardening** using tools like Nessus and Kali.
- Capture and explain **network and security events** with Wireshark and Windows logging.
- Produce professional, GitHub-hosted **build notes and security documentation** similar to what a junior security analyst or sysadmin would create on the job.

## High-Level Architecture

All systems run as VMs on VMware Workstation within the 192.168.10.0/24 lab network.

- **DC01** – Windows Server 2019/2022 domain controller (AD DS, DNS) for `corp.local` domain.
- **FS01** – Windows Server file server hosting departmental shares (HR, IT, Finance, Sales) with NTFS + share permissions, centralized file auditing, and 4663 event logging.
- **WIN10** – Domain-joined Windows 10 Enterprise client used for access testing, user workflows, and cross-platform validation.
- **UBUNTU-SRV** – Ubuntu Server 22.04 integrated into `corp.local` domain via realmd/SSSD, providing Linux endpoint for cross-platform file access (SMB mounts), auditd-based logging, and SIEM-ready log forwarding.
- **SEC-TOOLS (Kali)** – Kali Linux security tools VM (Nessus, Nmap, Wireshark, etc.) used for vulnerability scanning and network traffic capture.

**Planned additions:**

- Centralized logging / SIEM collection (Graylog, Splunk, or ELK stack) for ingesting Windows Security logs and Linux syslog.
- Additional hardening aligned with CIS benchmarks and RMF-style compliance documentation.
- Network topology and Active Directory structure diagrams.

## Phases and Documentation

Detailed build notes with PowerShell/Bash commands and screenshots are stored in the `docs/` and `screenshots/` directories.

### Phase 1 – Core Network and Domain Setup

**Goal:** Stand up a working `corp.local` domain with static IPs, verified connectivity, and firewall troubleshooting.

**Key accomplishments:**

- Static IP configuration for all VMs on 192.168.10.0/24 (DC01: 192.168.10.10, FS01: 192.168.10.11, WIN10: 192.168.10.30, UBUNTU-SRV: 192.168.10.20, SEC-TOOLS: 192.168.10.50).
- Promoted **DC01** to first domain controller in new `corp.local` forest using `Install-ADDSForest` PowerShell cmdlet.
- Installed integrated DNS on DC01 for domain name resolution.
- Joined **FS01** and **WIN10** to the domain using `Add-Computer` PowerShell cmdlet with domain credentials.
- Enabled ICMP echo (ping) on FS01 by activating "File and Printer Sharing" firewall rules to resolve connectivity troubleshooting.
- Validated end-to-end connectivity across lab subnet (DC01 ↔ FS01 ↔ WIN10 ↔ UBUNTU-SRV).

**Documentation:**
- `docs/03-build-notes-phase1.md`
  - Static IP configuration commands (PowerShell and netsh examples).
  - DC01 promotion commands and forest creation.
  - Domain join commands for FS01 and WIN10.
  - Ping test results before/after firewall rule changes.
  - Screenshots of successful domain joins and connectivity verification.

### Phase 2 – Virtual Network Architecture: AD Structure and File Services

**Goal:** Build a realistic small-company directory structure with departmental OUs, users, groups, and secured file shares with least-privilege access.

**Key accomplishments:**

- Created **AD OU hierarchy**: 
  - Root: `OU=Corp,DC=corp,DC=local`
  - Structure: `Servers`, `Workstations`, `Users` → `IT`, `HR`, `Finance`, `Sales`

- Created **domain user accounts** (5 total):
  - **IT:** jadmin (Jon Admin), itech (Ivy Tech)
  - **HR:** hgreen (Hannah Green)
  - **Finance:** fsmith (Frank Smith)
  - **Sales:** sjones (Sara Jones)

- Created **security groups** for group-based access control:
  - `HR_Share_RW`, `Finance_Share_RW`, `Sales_Share_RW` (department access)
  - `IT_Admins` (for IT staff with elevated privileges)

- On **FS01**, created departmental folders and SMB shares:
  - `C:\Dept\HR`, `C:\Dept\Finance`, `C:\Dept\Sales` (initially)
  - Shared as `\\FS01\HR`, `\\FS01\Finance`, `\\FS01\Sales`

- Configured **NTFS and share permissions**:
  - Each department group has full control of their own share.
  - Domain Admins retain full control across all shares.
  - Other departments denied access (least-privilege principle).

- Verified access from WIN10:
  - HR users (hgreen) can read/write to HR share, denied on Finance/Sales.
  - Finance users (fsmith) can read/write to Finance share, denied on HR/Sales.
  - Cross-checked access control across departments.

**Documentation:**
- `docs/04-build-notes-phase2.md`
  - PowerShell snippets for OU creation using `New-ADOrganizationalUnit`.
  - User creation commands with `New-ADUser` and password management.
  - Security group creation and membership management (`New-ADGroup`, `Add-ADGroupMember`).
  - SMB share creation commands using `New-SmbShare`.
  - NTFS ACL configuration using `icacls` with inheritance disabled and explicit permissions.
  - Screenshots of AD Users and Computers showing OU tree, users, and group memberships.
  - Access test results showing allowed/denied behavior from WIN10 for each department.

### Phase 3 – Enterprise File Auditing and Server Hardening

**Goal:** Centralize server audit settings, enable file access auditing, and establish a baseline for security monitoring with 4663 object-access events.

**Key accomplishments:**

- **Server hostname cleanup**: Renamed generic computer names to enterprise-standard names:
  - Original: `WIN-3LMB8KE361E` (example) → `FS01`
  - Original: `WIN-DC01` (example) → `DC01`

- **Moved servers into Servers OU**:
  - `CN=DC01,CN=Computers,DC=corp,DC=local` → `OU=Servers,OU=Corp,DC=corp,DC=local`
  - `CN=FS01,CN=Computers,DC=corp,DC=local` → `OU=Servers,OU=Corp,DC=corp,DC=local`
  - Verified movement using `Get-ADComputer -Filter * -SearchBase "OU=Servers,OU=Corp,DC=corp,DC=local"`

- **Created Server-Audit-Baseline GPO**:
  - New GPO linked to `OU=Servers,OU=Corp,DC=corp,DC=local`
  - Registry-based settings to enforce advanced audit policies:
    - `HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System\Audit\AuditLogon` = 1
    - `HKLM\System\CurrentControlSet\Control\Lsa\SCENoApplyLegacyAuditPolicy` = 1 (prevents legacy audit policy override)
  - Applied GPO changes to both DC01 and FS01 using `Invoke-GPUpdate -Computer -Force`

- **Enabled file system auditing policy**:
  - Ran `auditpol /set /subcategory:"File System" /success:enable /failure:enable` on FS01
  - Confirmed with `auditpol /get /subcategory:"File System"`

- **Configured SACLs on department folders**:
  - Added audit rules to `C:\Dept\HR`, `C:\Dept\IT`, `C:\Dept\Finance`, `C:\Dept\Sales`
  - Configured to audit Read and Write access for `CORP\Domain Users`
  - Used PowerShell `New-Object System.Security.AccessControl.FileSystemAuditRule` to create audit rules
  - Applied with `Set-Acl` cmdlet

- **Corrected Phase 2 oversight**: 
  - Created missing IT department folder: `C:\Dept\IT`
  - Created missing IT user: `iit` (Ian IT) in `OU=IT,OU=Users,OU=Corp,DC=corp,DC=local`
  - Added SMB share and NTFS permissions for IT user to access IT folder

- **Validated 4663 Security log events**:
  - Generated file access from WIN10 (as hgreen, iit, etc.) to department shares
  - Captured 4663 events using `Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4663}`
  - Filtered by department path (e.g., `C:\Dept\HR\*`) to verify auditing per department
  - Confirmed timestamps, user identities, and access types in event logs

**Documentation:**
- `docs/05-build-notes-phase3.md`
  - Server hostname rename commands using `Rename-Computer`
  - AD object move commands using `Move-ADObject`
  - GPO creation and linking using `New-GPO` and `New-GPLink`
  - Registry-based GPO settings using `Set-GPRegistryValue`
  - File system audit policy commands using `auditpol`
  - SACL configuration PowerShell scripts
  - 4663 event query examples and filtered results
  - Screenshots of hostname changes, OU verification, GPO settings, and 4663 event logs

### Phase 4 – Linux Integration and Cross‑Platform Auditing

**Goal:** Integrate UBUNTU-SRV into `corp.local` domain, enable cross-platform file access, and establish Linux auditing to complement Windows 4663 events.

**Key work:**

- Install **realmd/SSSD** on UBUNTU-SRV for Active Directory integration.
- Join **UBUNTU-SRV to corp.local** using domain credentials (CORP\Administrator).
- Mount FS01 departmental shares (HR, IT, Finance, Sales) from Linux using SMB/CIFS as domain users.
- Install and configure **auditd** on UBUNTU-SRV with file system watch rules for `/srv/shared`.
- Configure **rsyslog** to forward logs to a future SIEM (192.168.10.60:514) for centralized collection.
- Validate cross-platform access: domain users accessing Windows shares from Linux with matching audit events on both platforms.

**Planned Documentation:**
- `docs/06-build-notes-phase4.md`
  - Realm join and SSSD configuration commands.
  - SMB mount examples and cross-platform access verification.
  - auditd installation and audit rule creation.
  - rsyslog forwarding configuration (SIEM-ready).
  - Screenshots of successful realm join, SMB mounts, and Linux audit events matching Windows 4663 access.

## Planned Phases – Security and Assessment

### Phase 5 – RMF Asset Inventory and Risk Baseline

  - Document all lab assets in an inventory table (role, IP, OS, criticality, data types).
  - Define system boundary and data classification.
  - Create a simple risk matrix for common threats in this environment.
    
### Phase 6 – Vulnerability Assessment (Nessus + Kali)

  - Install and configure Nessus Essentials on SEC-TOOLS (Kali VM).

  - Run unauthenticated scans against DC01 (DNS/AD services), FS01 (SMB), UBUNTU-SRV (SSH/services).

  - Run authenticated scans using domain credentials to assess configuration weaknesses.

  - Capture network traffic during scans using Wireshark (tcpdump on Linux side).

  - Document top findings (missing patches, weak ciphers, open ports, credential exposure).

  - Map findings to mitigation actions and remediation steps.

### Phase 7 – Hardening and Re-Assessment

  - Apply hardening measures based on Phase 5 findings:

  - Update Windows Server patching policy

  - Disable legacy SMB versions (SMBv1)

  - Enable SMB signing and encryption

  - Apply Windows Defender exclusions for monitoring tools

  - Harden SSH configuration on UBUNTU-SRV

  - Enable SELinux or AppArmor on Linux

  - Re-run Nessus scans post-hardening

  - Compare before/after vulnerability counts and severity scores

  - Document residual risk and accepted exceptions

### Phase 8 – Final Security Assessment Report

- Executive summary of lab environment, architecture, and business context

- Threat modeling and risk assessment specific to this environment

- Detailed findings from Phase 5 (pre-hardening) Nessus scan


