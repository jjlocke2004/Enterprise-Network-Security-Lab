# Enterprise Network Security Lab

Virtual enterprise network security lab built with VMware Workstation to simulate a small Windows and Linux environment with Active Directory, file services, Ubuntu Server, and a Kali “security tools” VM. The environment is used to scan, harden, and observe vulnerabilities in a segmented 192.168.10.0/24 network.

> Status: In progress – core AD/file services are built, documentation for Phase 1–2 is underway, and additional security phases (Nessus, hardening, logging, RMF-style analysis) are being added.

---

## Lab Goals

- Practice **enterprise Windows administration** (AD DS, file server, GPOs, NTFS/share permissions).
- Build and document **realistic access controls** using users, groups, and departmental shares.
- Perform **vulnerability assessment and hardening** using tools like Nessus and Kali.
- Capture and explain **network and security events** with Wireshark and Windows logging.
- Produce professional, GitHub-hosted **build notes and security documentation** similar to what a junior security analyst or sysadmin would create on the job.

---

## High-Level Architecture

All systems run as VMs on VMware Workstation within the 192.168.10.0/24 lab network.

- **DC01** – Windows Server domain controller (AD DS, DNS) for `corp.local`.
- **FS01** – Windows file server hosting departmental shares with NTFS + share permissions.
- **WIN10** – Domain-joined Windows 10 client used for access testing and user workflows.
- **UBUNTU-SRV** – Ubuntu Server providing a Linux endpoint for interoperability and scanning.
- **SEC-TOOLS (Kali)** – Security tools VM (Nessus, Nmap, Wireshark, etc.) used for scanning and traffic capture.

Planned additions:

- Centralized logging / SIEM-style collection (e.g., forwarding Windows logs to a collector).
- Additional hardening aligned with CIS benchmarks and basic RMF-style documentation.

---

## Phases and Documentation

Detailed build notes with screenshots are stored in the `docs/` and `screenshots/` directories.

### Phase 1 – Core Network and Domain Setup

**Goal:** Stand up a working `corp.local` domain with static IPs and verified connectivity.

Key work:

- Static IP configuration for all VMs on 192.168.10.0/24 (DC01, FS01, WIN10, UBUNTU-SRV, SEC-TOOLS).
- Promote **DC01** to the first domain controller for `corp.local` and install integrated DNS.
- Join **FS01** and **WIN10** to the domain using PowerShell (`Add-Computer`).
- Troubleshoot ICMP echo (ping) failures by enabling the proper Windows Defender Firewall rule on FS01.

Documentation:

- `docs/03-build-notes-phase1.md`  
  - Static IP screenshots.  
  - Domain controller promotion commands.  
  - Domain join screenshots.  
  - Ping / firewall troubleshooting with before/after evidence.

### Phase 2 – Virtual Network Architecture: AD Structure and File Services

**Goal:** Make the lab look like a small company: departmental OUs, users, groups, and secured file shares.

Key work:

- Build **AD OU structure**: `Corp` → `Servers`, `Workstations`, `Users` → `IT`, `HR`, `Finance`, `Sales`.
- Create **domain users** (e.g., IT admins, HR/Finance/Sales users) under their department OUs.
- Create **security groups** (e.g., `HR_Share_RW`, `Finance_Share_RW`, `Sales_Share_RW`, `IT_Admins`) for group-based access control.
- On **FS01**, build departmental folders on `C:\Dept` and share them as `\\FS01\HR`, `\\FS01\Finance`, `\\FS01\Sales`.
- Configure **NTFS and share permissions** so each department’s group has access only to its own share, with Domain Admins retaining full control.
- Fix a **trust relationship issue** on WIN10 and validate DNS / domain connectivity.
- Verify access from WIN10 using HR/Finance/Sales test accounts (allowed vs. Access Denied behavior).

Documentation:

- `docs/04-build-notes-phase2.md` 
  - PowerShell snippets for OUs, users, groups.  
  - Screenshots of AD Users and Computers, group membership, and file server ACLs.  
  - Access test results from WIN10 showing least-privilege in action.

### Planned Phases – Security and Assessment

These are planned additions to turn the lab into a complete security project:

- **Phase 3 – Logging and Auditing**
  - Create and link GPOs to enable advanced audit policies (logon, object access, policy change).
  - Enable folder auditing on key shares (e.g., HR, Finance).
  - Generate sample events (file creates/deletes, logons) and capture Security log entries.

- **Phase 4 – Asset Inventory and Risk Baseline**
  - Document all lab assets in an inventory table (role, IP, OS, criticality, data types).
  - Define system boundary and data classification.
  - Create a simple risk matrix for common threats in this environment.

- **Phase 5 – Vulnerability Assessment (Nessus + Kali)**
  - Install and configure Nessus Essentials on **SEC-TOOLS**.
  - Run unauthenticated and authenticated scans against DC01, FS01, and UBUNTU-SRV.
  - Summarize top findings and map them to mitigation actions.
  - Capture Wireshark traces during scans and explain what the traffic shows.

- **Phase 6 – Hardening and Re-Assessment**
  - Apply selected hardening steps (e.g., password policy, disabling legacy protocols, OS patching).
  - Re-run Nessus scans and compare before/after results.
  - Document changes and residual risk.

- **Phase 7 – Final Security Assessment Report**
  - High-level summary of environment, threats, findings, and remediations.
  - Screenshots and metrics (e.g., vulnerability counts before vs. after).
  - Intended to look like a junior analyst’s internal report.

---

## Technologies and Skills Demonstrated

- **Operating Systems**
  - Windows Server (AD DS, DNS, file services)
  - Windows 10 Enterprise
  - Ubuntu Server (command-line administration)
  - Kali Linux (security tools VM)

- **Core Infrastructure**
  - Active Directory: forests, domains, OUs, users, groups
  - File server: SMB shares, NTFS and share permissions, group-based access control
  - DNS, static IP addressing, basic network troubleshooting (ping, firewall rules)

- **Security / Blue-Team Skills**
  - Windows auditing and event logs (Security log, object access, logon events)
  - Group-based least-privilege access design
  - Vulnerability scanning (Nessus Essentials – planned/ongoing)
  - Network traffic capture and analysis (Wireshark – planned/ongoing)
  - Basic hardening and verification via re-scans

- **Tooling and Documentation**
  - PowerShell for administration and automation
  - VMware Workstation for lab virtualization
  - Markdown-based technical documentation and diagrams hosted in GitHub

---

## Repository Layout (Planned)

- `docs/`
  - `03-build-notes-phase1.md` – Core lab network and domain setup.
  - `04-build-notes-phase2.md` – AD structure, users/groups, and file server shares.
  - `2x-...` – Future phases for logging, Nessus scans, hardening, and final report.
- `screenshots/`
  - `build-notes-phase1-screenshots/`
  - `build-notes-phase2-screenshots/`
- `diagrams/`
  - Logical network and AD diagrams (planned).
- Root files
  - `README.md` (this file)
  - Other helper docs as the lab grows.

---

## Future Work

- Complete and upload remaining phase documentation and screenshots.
- Add a concise “Executive Summary” document for non-technical readers (e.g., hiring managers).
- Optionally integrate a lightweight SIEM or log collection solution for centralized security monitoring.

---

If you have feedback or suggestions for improvements, feel free to open an issue or reach out.
