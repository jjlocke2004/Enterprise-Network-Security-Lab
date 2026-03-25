# Enterprise Network Security Lab

Virtual enterprise network security lab constructed with VMware Workstation to model a small windows and linux network with Active Directory, file services, Ubuntu Server and Kali VM in case study security tools VM. Vulnerabilities are scanned, hardened, and viewed on a segmented portion of the 192.168.10.0/24 network by scanning the environment.

**Status:** Phase 7 (Hardening and Re-Assessment) complete. Phase 6 (Nessus Essentials vulnerability assessment) full with unauthenticated and attempted authenticated scan. Phase 5 (RMF Asset Inventory and Risk Baseline) is finished. Phases 1-4 are fully documented (AD/ file services, cross platform auditing, Linux integration).

## Lab Goals

- Learn administration of windows enterprise (AD DS, file server, GPOs, NTFS/share permissions, Group Policy baselines).
- Design and record realistic access controls based on users and groups, and departmental shares in Windows and Linux.
- Use Linux servers with the Windows environment with Active Directory (realmd/SSSD) to manage identities.
- Centralize monitoring and SIEM integration - Enabling cross platform file auditing (Windows 4663 Security events + Linux auditd).
- Use such tools as Nessus and Kali to perform vulnerability assessment and consequently harden.
- Network and security events capture and explanation using Wireshark and windows logging.
- Prepare professional, GitHub published build notes and security documentation as a junior security analyst or sysadmin would do in the real world.

## High-Level Architecture

Every system is a VMware Workstation operating on the VMware Workstation on the 192.168.10.0/24 lab network.

- **DC01** – Windows server 2019/2022 domain controller (AD DS, DNS) of corp.local domain.
- **FS01** – Windows Server file server hosting departmental share (HR, IT, Finance, Sales) NTFS + share permissions, centralized file auditing and 4663 event logging.
- **WIN10** – Domain-joined Windows 10 Enterprise client, to be used in access testing, user workflow, cross-platform validation.
- **UBUNTU-SRV** – ubuntu server 22.04 boosted into the domain corp.local via realmd/SSSD, which gives Linux endpoint access to cross-platform file access (SMB mounts), auditd-based logging, and SIEM-ready logs forwarding.
- **SEC-TOOLS (Kali)** – VM Kali Linux security tools with the Nessus Essentials, Nmap, Wireshark, and other vulnerability scanning and network traffic capture tools.

**Planned additions:**
- Windows Security log ingestion and Linux syslog ingestion to a centralized logging / SIEM collection system (Graylog, Splunk, or ELK stack).
- More hardening was in line with CIS specifications and RMF-type of compliance documentation.
- Active Directory structure and Network topology diagrams.

## Phases and Documentation

Detailed build notes (power shell/ bash commands and screenshots) are stored in the directory docs/ and screenshots/ directory.

---

### Phase 1 – Core Network and Domain Setup

**Objective:** To have a functional corp.local domain setup using static IPs, tested connectivity and troubleshooting of firewall.

**Key accomplishments:**
- All VMs on 192.168.10.0/24 will be configured with a static IP (DC01: 192.168.10.10, FS01: 192.168.10.11, WIN10: 192.168.10.30, UBUNTU-SRV: 192.168.10.20, SEC-TOOLS: 192.168.10.50).
- Connected DC01 to new corp.local forest first domain controller with Install-ADDSForest PowerShell cmdlet.
- Installed built in DNS on DC01 to resolve domain names.
- Added FS01 and WIN10 to the domain with the help of add-computer PowerShell command and domain credentials.
- Allowed FS01 to respond to ICMP echo (ping) packets by enabling firewall rules of File and Printer Sharing to troubleshoot connectivity.
- Verified end to end connectivity between lab subnet (DC01 - FS01 - WIN10 - UBUNTU-SRV).

**Documentation:**
- docs/03-build-notes-phase1.md - Static IP setup commands, DC01 promotion, domain additions, firewall modifications, connection tests, and screenshots.

---

### Phase 2 – Virtual Network Architecture: AD Structure and File Services

**Purpose:** Construct an authenticated small-company directory hierarchy comprising of departmental OUs, users, groups, and secured file shares having least-privilege access.

**Key accomplishments:**
- Developed AD OU structure: OU=Corp, DC=corp, DC=local - substructure created for Servers, Workstations, and Users (IT, HR, Finance, Sales).
- Lexically created 5 departmental user accounts (jadmin, itech, hgreen, fsmith, sjones) and placed them at the right OU.
- Generated group-based access control security groups: HR Share RW, Finance Share RW, Sales Share RW, and IT Admins.
- Created departmental folders (C:\Dept\HR, C:\Dept\Finance, C:\Dept\Sales), and shares (SMB) (\\FS01\HR, etc.).
- NTFS and share permissions are configured and set by the least-privilege principle: each department group has full control over their own share; Domain Admins have full control; other departments denied.
- WIN10 verified access control: department users can read/write on their own shares and are not allowed to access other share files.

**Documentation:**
- docs/04-build-notes-phase2.md - OU creation, user/group management, create smb share, set NTFS ACL, access test result and screenshots.

---

### Phase 3 – Enterprise File Auditing and Server Hardening

**Aim:** Centralize the server audit settings and file access audit and set a baseline of security monitoring with 4663 object-access events.

**Key accomplishments:**
- Changed names of servers to enterprise standard names (e.g., WIN-3LMB8KE361E - FS01, WIN-DC01 - DC01).
- Transferred DC01 and FS01 to Servers OU via manipulation of AD object.
- Developed Server-Audit-Baseline GPO associated with OU=Servers with greater audit policy configurations (AuditLogon, SCENoApplyLegacyAuditPolicy).
- Enabled file system auditing policy in FS01 with auditpol commands of Read and Write access.
- Department folders (C:\Dept\*) had configured SACLs to audit file access by CORP\Domain Users.
- Made missing IT department folder and user (iit), finished IT share set up.
- 4663 Security log events: validated file access generated by WIN10 to the department shares and the corresponding 4663 events were captured.
- Authenticated audit recording is identical to user identity, access type, and folder path.

**Documentation:**
- docs/05-build-notes-phase3.md Hostname rename commands, OU moves, GPO creation and registry settings, auditpol commands, SACL PowerShell scripts, 4663 event queries, and screenshots.

---

### Phase 4 – Linux Integration and Cross-Platform Auditing

**Objective:** Install UBUNTU-SRV into corp.local domain and allow cross-platform access to files using SMB, and establish Linux auditing to supplement windows 4663 events.

**Key accomplishments:**
- Active Directory installed realmd/SSSD on UBUNTU-SRV.
- Logged into UBUNTU-SRV to the corp.local with domain credentials.
- Mounted FS01 departmental shares (HR, IT, Finance, Sales) of the Linux as domain users with SMB/CIFS.
- Ubuntu servers monitored with installed and configured auditd with file system watch rules in directories being monitored.
- Tuned the rsyslog to send the logs to a future SIEM (192.168.10.60:514) which would have centralized logs.
- Tested cross platform access: domain users access to windows shares on Linux without the occurrence of an audit event on both platforms.

**Documentation:**
- docs/06-build-notes-phase4.md - realm join and SSSD configuration, SMB mount examples, auditd installation and audit rules, rsyslog forwarding configuration, and realm join, mounts, and audit event screenshots.

---

### Phase 5 – RMF Asset Inventory and Risk Baseline

**Aim:** Identify and document all lab assets, identify system boundary and data classification, discover threats as well as define a baseline risk posture.

**Status:** Complete.

**Key accomplishments:**
- **System Boundary Definition:** Defined range of ENSL which comprises of all five VMs (DC01, FS01, WIN10, UBUNTU-SRV, SEC-TOOLS), VMware Workstation hypervisor and lab subnet 192.168.10.0/24; all external networks, backups and DR infrastructure is excluded.
- **Asset Inventory:** 5 VMs, 2 infrastructure assets, 25 AD objects (users, groups, computers, OUs), 7 network services and 6 data repositories with criticality levels and data sensitivity ratings have been cataloged.
- **Data Classification:** Introduced four levels of classification (Public, Internal, Sensitive, Restricted) and assigned all the lab data (credentials, HR/Finance/Sales shares, event logs) to the corresponding level and applied access control measures.
- **Threat Profile:** Found 5 types of threat actors (insider HR user, insider Finance user, external attacker, accidental misconfiguration, privilege escalation attack), and eight types of typical attack vectors (weak passwords, unpatched systems, legacy SMB, excessive permissions, credential theft, privilege escalation, lateral movement, unauthorized logons).
- **Risk Matrix:** Risk scoring methodology (Likelihood x Impact x Exploitability) has been developed (8+ risks) and rated as Critical (weak passwords: score 100, unpatched OS: score 80), Low (legacy SMB: score 12).
- **Mitigation Roadmap:** For every Critical/High risk, each of them was associated with mitigation that was planned in Phase 6 (vulnerability assessment) and Phase 7 (hardening).

**Documentation:**
- docs/10-asset-inventory-risk-baseline.md - The entire asset inventory table, data classification scheme, threat actor profiles, attack vectors analysis, risk matrix with scoring methodology and residual risk evaluation.

---

### Phase 6 – Vulnerability Assessment (Nessus + Kali)

**Task:** Install Nessus on the security tools VM, unauthenticated scans, authenticated scans, traffic capturing and hardening actions identification.

**Status:** Finished - vulnerability assessment is completed; three major findings were chosen to be remedied during Phase 7.

**Key accomplishments:**

#### 6.1 Nessus Installation
- CONFIG Nessus Essentials on SEC-TOOLS (Kali VM).
- License key of Registered Essentials and downloaded initial plugins.
- Authenticated web UI access at the address of 127.0.0.1:8834.
- Updates the plugins with sudo /opt/nessus/sbin/nessuscli update --plugins-only.

#### 6.2 Unauthenticated Scan
- **Scan settings:** Lab-Unauthenticated targeting 192.168.10.11 (FS01) and 192.168.10.20 (UBUNTU-SRV), no credentials.
- **Key findings identified:**
  1. **SMB Signing not required (FS01, Medium)** - Signing not required: It is possible to have an SMB session without a signature; mapped to the hardening action on FS01.
  2. **ICMP Timestamp Request Remote Date Disclosure (UBUNTU-SRV, Low)** - Information disclosure; equivalent to ufw hardening on UBUNTU-SRV.
  3. **SMB (Multiple Issues) (FS01, Info)** - SMB (Multiple Issues) sets justification of SMB hardening; SMBv1 assessment.
  4. **HTTP (Multiple Issues) (UBUNTU-SRV, Info)** - HTTP banner/service exposure; reported to be hardened later.
  5. **Open Port / Service Enumeration (FS01, Info)** - Open Port / Service Enumeration, Confirms reachability, service landscape; post-hardening base; Use for comparison.

#### 6.3 Authenticated Scan Attempt
- **Scan setup:** Lab-Authenticated-DC-FS with 192.168.10.10 (DC01) and 192.168.10.11 (FS01).
- **Account set up:** Went into AD and created account in domain `svc_scanner` and added it to local Users group on both of the hosts.
- **Firewall configuration:** [DC01 and FS01] Enabled WMI and Remote Administration firewall rule groups; winmgmt services and RemoteRegistry services started.
- **Findings:** Nessus scans were performed, and mentioned Auth: Fail. The Nessus Essentials failed to use credentialed checks comprehensively despite the attempts to configure the credentialing. Lab left the unauthenticated baseline findings and SMB-enumerable (e.g. share access via svc scanner) to guide hardening.
- **Auth attempt result:** (DC01, High) **Microsoft windows SMB Shares Unprivileged Access** - svc scanner were able to read SYSVOL/NETLOGON and other shares; they were mapped to ACL hardening on DC01.

#### 6.4 Selected Vulnerabilities and Planned Mitigations

Three vulnerabilities that were selected to be remedied, one of them per host:

| Host       | Nessus Finding                                        | Type                    | Planned Control                                             |
|-----------|--------------------------------------------------------|-------------------------|-------------------------------------------------------------|
| UBUNTU-SRV | ICMP Timestamp Request Remote Date Disclosure         | Information disclosure  | Enable ufw default deny incoming allow only SSH             |
| FS01       | SMB Signing not required                               | Configuration / SMB     | Enforce SMB signing on the server (Registry/PowerShell)     |
| DC01       | Microsoft Windows SMB Unprivileged Access Shares       | Access control / shares | Apply least-privilege ACLs to lab shares                    |

#### 6.5 Remediation Progress
- **UBUNTU-SRV ICMP Timestamp:** Default deny on incoming and only allow SSH by default by setting up the ufw and enabling it by starting with ufw status verbose.
- **FS01 SMB Signing:** Ready PowerShell registry keys to impose SMB signing; awaiting execution and post-hardening countercheck.
- **DC01 Share Access:** SYSVOL/NETLOGON unprotected shares (i.e., not to be restricted) identified as critical shares; least-privilege ACLs to be placed on lab-specific shares; awaiting implementation and post-hardening verification.

**Documentation:**
- docs/08-build-notes-phase6.md - Nessus installation, unauthenticated and authenticated scan configurations, top 5 findings with type tags, svc_scanner account setup, auth failure documentation, and screenshots.

---

### Phase 7 – Hardening and Re-Assessment

**Purpose:** Hardening actions following the Phase 5 risk matrix and Phase 6 Nessus results, and confirm with follow-up scans and record remediation measures towards each of the hosts selected.


**Status:** Complete - hardening controls implemented on UBUNTU-SRV, FS01, and DC01 and Nessus follow-up scans were reviewed.

**Key accomplishments:**

#### UBUNTU-SRV - ICMP Timestamp Request Remote Date Disclosure.
- Nessus confirmed that UBUNTU-SRV was responding to ICMP timestamp requests (information disclosure).
- Enabled and set up ufw to use a default deny posture on inbound traffic but permit needed services:
- sudo ufw default deny incoming.
- sudo ufw default allow outgoing.
- `sudo ufw allow ssh`
- `sudo ufw enable`
- Checked with the command verified ufw status verbose indicated that firewall was active with default deny on incoming and allow on outgoing and the SSH being an explicit allow rule.
- Re-scanned Nessus and ensured that the "ICMP Timestamp Request Remote Date Disclosure" plug-in did not flag UBUNTU-SRV anymore.

#### FS01 - SMB Signing Not Required
- Used authenticated Nessus results of Phase 6 to prioritize on FS01 (risk of SMB session hijacking/MITM), SMB Signing Not Required.
- Signing SMB using registry keys and a restart of the service:
- `reg add HKLM\SYSTEM\CurrentControlSet\Services LanmanServer\Parameters /v RequireSecuritySignature /t REG_DWORD/d 1 /f.`
- `reg add "HKLM\SYSTEM\CurrentControlSet\Services LanmanWorkstation\Parameters" /v RequiresecuritySignature/t REG_DWORD/d 1/f`
- net stop lanmanserver/net start lanmanserver.
- validated a configuration using command output and re-executed the authenticated DC/FS scan to ensure that the vulnerability of not having SMB Signing being not required was not true of FS01 anymore.

#### DC01 - SMB - Least-privilege access (lab share)
- Took the Phase 6 result of Microsoft Windows SMB Shares Unprivileged Access as an excuse to harden a lab data share rather than make changes to SYSVOL/NETLOGON (which should not be changed because it needs to be readable by AD and GPO).
- Shared out a specific lab with a virtual lab share that was created and named C:\DeptShare_Test and set it with extensive access at first in order to simulate a misconfiguration:
NTFS: icacls C:\DeptShare_Test /grant everyone (OI) (CI)M.
- Share: `New-SmbShare -Name 'DeptShare_Test' -Path C:DeptShare_Test' -FullAccess everyone'
- Share-level applied least-privilege hardening:
- Removed broad identities:
- Revoke-SmbShareAccess Name 'DeptShare_Test' AccountName everyone force
- Revoke-SmbShareAccess -Name DeptShare Test -accountname Authenticated Users -force
- Ignored only domain groups necessary:
- Grant-SmbShareAccess -Name "DeptShare_ Test" -AccountName CORP domain Admins -AccessRight Full -Force.
- Grant-SmbShareAccess -Name DeptShare -Test -AccountName CORPHRShareRW -AccessRight Full -Force.
- Checked with Get-SmbShareAccess -Name DeptShare_Test' that it was limited to the following groups.
- NTFS ACLs that match the least-privilege model:
- icacls C:DeptShareTest/ remove:g everyone.
- icacls C: DeptShare Test / grant:r CORP Domain admins:(OI)/CI /F
- icacls C:DeptShare_Test /grant:r CORP HR Share RW: (OI) (CI) F.
- Verified by running icacls C:DeptShare_Test that Everyone has been removed and only the domestic groups (as well as system/administrators) have been left.
- Re-executed the authenticated DC/FS scan and used the result, together with the ACL/SMB output, as a testimony to employing the least-privilege but leaving SYSVOL/NETLOGON to their safe default settings.

- `docs/09-build-notes-phase7.md` - Step-by-step instructions on how to harden UBUNTU-SRV (ufw configuration), FS01 (SMB signing registry changes and service restart), and DC01 (DeptShare_Test NTFS/share ACL changes) including before and after Nessus screenshots found in the sub-directory screenshots/build-notes-phase7-screenshots.

