# Phase 5 - RMF Asset Inventory and Risk Baseline

**Document Version:** 1.0  
**Date:** February 2026  
**Lab:** Enterprise Network Security Lab (ENSL)  
**Categorization:** Internal Use

## Executive Summary

This is a document that establishes the System Boundary, Asset Inventory, Data Classification, Threat Profile, and Risk Matrix of Enterprise Network Security Lab (ENSL). The ENSL is a virtualized business network environment that has an approximate of 50-100 employee sized IT infrastructure simulated on windows and linux operating systems, use of active directory, file services and centralized auditing.

**Purpose:** To set up the minimum level of knowledge regarding the assets and vulnerabilities of the system prior to vulnerability assessment (Phase 5), hardening (Phase 6), and continuous security monitoring.

---

## System Boundary and Scope

### 2.1 Defined System Boundary

ENSL system will involve the following service virtual machines that will be operating on VMware Workstation in the 192.168.10.0/24 network segment:

| Component | Included | Rationale |
|-----------|----------|-----------|
| DC01, FS01, WIN10, UBUNTU-SRV, SEC-TOOLS VMs | In Scope | Core lab infrastructure |
| VMware Workstation hypervisor | In Scope | Host platform of lab VMs |
| Physical lab network (192.168.10.0/24) | In Scope | Lab subnet connectivity |
| External networks / Internet | Out of Scope | Lab is air-gapped; there is no internet connectivity |
| Backup systems | Out of Scope | Not deployed in current phases |
| Disaster recovery infrastructure | Out of Scope | Not implemented at present stages |

### 2.2 Environment Type

- **Classification:** Isolated Test/Development Lab
- **Access:** Physical lab on campus of University at Albany, Albany, NY
- **Connection to the network:** Air-gapped (there is no connection to the network; internal lab network only)
- **Applicability of Regulations:** N/A (non-production educational setting)

---

## Asset Inventory

### 3.1 Hardware and Virtual Infrastructure

| Asset ID | Asset Name | Type | Specifications | Location | Criticality | Owner |
|----------|-----------|------|----------------|----------|------------|-------|
| HW-001 | VMware Workstation Host | Hypervisor | Windows/Linux desktop with 16+ GB RAM, 250+ GB storage | Lab | Critical | Student Lab |
| HW-002 | Lab Network Switch (VMnet1) | Virtual Switch | VMware network adapter | Lab | Critical | VMware |

### 3.2 Virtual Machine Assets

| Asset ID | Hostname | Role | IP Address | OS | vCPU | RAM | Disk | Criticality | Data Sensitivity |
|----------|----------|------|-----------|----|----|-----|------|------------|------------------|
| VM-001 | DC01 | Domain Controller | 192.168.10.10 | Windows Server 2019+ | 2 | 4 GB | 60 GB | Critical | High |
| VM-002 | FS01 | File Server | 192.168.10.11 | Windows Server 2019+ | 2 | 4 GB | 100 GB | Critical | High |
| VM-003 | WIN10 | Workstation | 192.168.10.30 | Windows Enterprise | 2 | 4 GB | 80 GB | Medium | Medium |
| VM-004 | UBUNTU-SRV | Linux Server | 192.168.10.20 | Ubuntu Server 22.04 | 2 | 4 GB | 60 GB | Medium | Medium |
| VM-005 | SEC-TOOLS | Security Tools | 192.168.10.50 | Kali Linux | 2 | 4 GB | 80 GB | Low | Low |

### 3.3 Active Directory Assets

| Asset ID | Object Type | Name | Location (OU) | Count | Criticality |
|----------|-----------|------|---------------|-------|------------|
| AD-001 | Forest | corp.local | - | 1 | Critical |
| AD-002 | Domain | corp.local | - | 1 | Critical |
| AD-003 | OU | Corp | DC=corp,DC=local | 1 | Critical |
| AD-004 | OU | Servers | OU=Corp,DC=corp,DC=local | 1 | Critical |
| AD-005 | OU | Workstations | OU=Corp,DC=corp,DC=local | 1 | Medium |
| AD-006 | OU | Users | OU=Corp,DC=corp,DC=local | 1 | Medium |
| AD-007 | OU | IT (Department) | OU=Users,OU=Corp,DC=corp,DC=local | 1 | High |
| AD-008 | OU | HR (Department) | OU=Users,OU=Corp,DC=corp,DC=local | 1 | High |
| AD-009 | OU | Finance (Department) | OU=Users,OU=Corp,DC=corp,DC=local | 1 | High |
| AD-010 | OU | Sales (Department) | OU=Users,OU=Corp,DC=corp,DC=local | 1 | High |
| AD-011 | User | Administrator (CORP\Administrator) | CN=Administrator,CN=Users,DC=corp,DC=local | 1 | Critical |
| AD-012 | User | jadmin (Jon Admin) | OU=IT,OU=Users,OU=Corp,DC=corp,DC=local | 1 | High |
| AD-013 | User | itech (Ivy Tech) | OU=IT,OU=Users,OU=Corp,DC=corp,DC=local | 1 | High |
| AD-014 | User | iit (Ian IT) | OU=IT,OU=Users,OU=Corp,DC=corp,DC=local | 1 | High |
| AD-015 | User | hgreen (Hannah Green) | OU=HR,OU=Users,OU=Corp,DC=corp,DC=local | 1 | Medium |
| AD-016 | User | fsmith (Frank Smith) | OU=Finance,OU=Users,OU=Corp,DC=corp,DC=local | 1 | Medium |
| AD-017 | User | sjones (Sara Jones) | OU=Sales,OU=Users,OU=Corp,DC=corp,DC=local | 1 | Medium |
| AD-018 | Security Group | IT_Admins | OU=IT,OU=Users,OU=Corp,DC=corp,DC=local | 1 | High |
| AD-019 | Security Group | HR_Share_RW | OU=HR,OU=Users,OU=Corp,DC=corp,DC=local | 1 | Medium |
| AD-020 | Security Group | Finance_Share_RW | OU=Finance,OU=Users,OU=Corp,DC=corp,DC=local | 1 | Medium |
| AD-021 | Security Group | Sales_Share_RW | OU=Sales,OU=Users,OU=Corp,DC=corp,DC=local | 1 | Medium |
| AD-022 | Computer | DC01 | OU=Servers,OU=Corp,DC=corp,DC=local | 1 | Critical |
| AD-023 | Computer | FS01 | OU=Servers,OU=Corp,DC=corp,DC=local | 1 | Critical |
| AD-024 | Computer | WIN10 | CN=Computers,DC=corp,DC=local | 1 | Medium |
| AD-025 | Computer | UBUNTU-SRV | CN=Computers,DC=corp,DC=local | 1 | Medium |

### 3.4 Network Services and Protocols

| Asset ID | Service | Host(s) | Port(s) | Protocol | Status | Criticality |
|----------|---------|---------|---------|----------|--------|------------|
| SVC-001 | Active Directory | DC01 | 389, 636 | LDAP/LDAPS | Enabled | Critical |
| SVC-002 | DNS | DC01 | 53 | UDP/TCP | Enabled | Critical |
| SVC-003 | Kerberos | DC01 | 88, 464 | UDP/TCP | Enabled | Critical |
| SVC-004 | SMB (File Sharing) | FS01 | 445 | TCP | Enabled | Critical |
| SVC-005 | SSH | UBUNTU-SRV | 22 | TCP | Enabled | Medium |
| SVC-006 | Windows Update | All Windows VMs | 443 | TCP | Enabled | Medium |
| SVC-007 | RDP (Remote Desktop) | WIN10 | 3389 | TCP | Enabled | Medium |

### 3.5 Data Repositories

| Asset ID | Name | Location | Data Type | Owner | Backup | Criticality |
|----------|------|----------|-----------|-------|--------|------------|
| DATA-001 | HR Share | \\FS01\HR (C:\Dept\HR) | HR files, documents, employee records | hgreen (HR) | None | High |
| DATA-002 | Finance Share | \\FS01\Finance (C:\Dept\Finance) | Financial data, budgets, reports | fsmith (Finance) | None | High |
| DATA-003 | Sales Share | \\FS01\Sales (C:\Dept\Sales) | Sales reports, client data, opportunities | sjones (Sales) | None | Medium |
| DATA-004 | IT Share | \\FS01\IT (C:\Dept\IT) | IT documentation, scripts, configs | iit (IT) | None | Medium |
| DATA-005 | Security Event Logs | DC01, FS01 (Security log) | Windows Security events (4663, logon, policy) | System | None | High |
| DATA-006 | Linux Audit Logs | UBUNTU-SRV (/var/log/audit/audit.log) | auditd file system events, access logs | System | None | Medium |

---

## Data Classification

### 4.1 Classification Levels

| Level | Definition | Examples in Lab | Handling |
|-------|-----------|-----------------|----------|
| Public | No confidentiality will be affected in case disclosed | Lab documentation, public screenshots | No encryption needed |
| Internal | It should not be very ubiquitous; moderate confidentiality | IT documentation, general logs | Access should be restricted to IT team |
| Sensitive | High confidentiality; would affect business | HR employee records, financial data, credentials | Encrypted, access-controlled, audited |
| Restricted | Maximum confidence; regulatory/compliance ramifications | Domain admin credentials, production backups | Strongest controls; least access |

### 4.2 Lab Data Classification Mapping

| Data Type | Classification | Owner | Access Control | Audit Logging |
|-----------|----------------|-------|-----------------|---------------|
| Domain credentials (CORP\Administrator) | Restricted | Lab Admin | IT Admins only | Yes (4663 + auditd) |
| HR employee records (names, positions, salaries implied) | Sensitive | HR Department | HR_Share_RW group | Yes (4663) |
| Finance reports and budgets | Sensitive | Finance Department | Finance_Share_RW group | Yes (4663) |
| Sales client data | Sensitive | Sales Department | Sales_Share_RW group | Yes (4663) |
| IT configurations and scripts | Internal | IT Department | IT_Admins + iit | Yes (4663) |
| User access logs (4663, auditd events) | Internal | IT Department | IT Admins | Yes (centralized) |
| Lab documentation and build notes | Public | Lab / GitHub | Public (GitHub repo) | No |

---

## Threat Profile

### 5.1 Threat Actor and Motivation of Threats

| Threat Actor | Type | Motivation | Capability | Likelihood | Impact |
|--------------|------|-----------|-----------|-----------|--------|
| Insider - Disgruntled HR User | Internal | Data theft, sabotage | Medium (access to HR share) | Medium | High (HR data exposure) |
| Insider - Finance User | Internal | Fraud, unauthorized transfers | Medium (access to Finance data) | Medium | High (loss of financial data) |
| External - Network Attacker | External | Reconnaissance, lateral movement | Medium (in case of air-gap attack) | Low (air-gapped lab) | High (in case of exploitation) |
| Misconfigured User | Accidental | Unintended data access | Low (permissions) | Medium | Medium (accidental disclosure) |
| Privilege Escalation Attack | Internal | Unauthorized admin access | Medium | Low-Medium | Critical (domain compromise) |

### 5.2 Familiar Attack Vectors in Scope

| Vector | Description | Current Mitigation | Residual Risk |
|--------|-----------|-------------------|---------------|
| Weak Passwords | Domain passwords are short and easy to guess/no password policy implemented | No password policy enforced | High |
| Unpatched Systems | Missing patches on Windows/Linux systems | No patch baseline | High |
| Legacy SMB (SMBv1) | Insecure file sharing protocol | SMBv2/v3 enabled (default) | Low |
| Excessive NTFS Permissions | File shares too open | Least-privilege configured | Low |
| Credential Theft | Domain credentials captured | No MFA; plain credentials logged on-site | Medium |
| Privilege Escalation | Local privilege escalation to admin | Standard user accounts only | Medium |
| Lateral Movement | Attacker moves between systems | No network segmentation | High |
| Unauthorized Logons | Bruteforce or credential replay attacks | No account lockout policy audited | Medium |

---

## Risk Assessment Matrix

### 6.1 Risk Scoring Methodology

**Risk Score = Likelihood x Impact x Exploitability**

- **Likelihood:** 1 (Low) to 5 (High) - Threat frequency
- **Impact:** 1 (Low) to 5 (High) - Business/security impact in case of realized
- **Exploitability:** 1 (Hard) to 5 (Easy) - How easily it can be exploited without protection

**Risk Level:**
- **Critical:** 60-125 (Immediate action needed)
- **High:** 40-59 (Action necessary in 1-2 weeks)
- **Medium:** 20-39 (Plan action in 1 month)
- **Low:** 1-19 (Monitor; plan action where possible)

### 6.2 Risk Matrix - Present Situation (Phase 5)

| Risk ID | Threat | Asset(s) Affected | Likelihood | Impact | Exploitability | Score | Level | Mitigation Status |
|---------|--------|-------------------|-----------|--------|-----------------|-------|-------|------------------|
| R-001 | Weak password policy | All user accounts | 4 | 5 | 5 | 100 | Critical | Unaddressed (Phase 7) |
| R-002 | Unpatched OS vulnerabilities | DC01, FS01, WIN10, UBUNTU-SRV | 4 | 5 | 4 | 80 | Critical | Unaddressed (Phase 6-7) |
| R-003 | Unauthorized file access/data theft | HR, Finance, Sales shares | 3 | 5 | 3 | 45 | High | Partially mitigated (NTFS/share permissions, 4663 audit) |
| R-004 | SMB signing not enforced | FS01 | 3 | 4 | 4 | 48 | High | Planned (Phase 7) |
| R-005 | Weak SSH configuration | UBUNTU-SRV | 2 | 4 | 3 | 24 | Medium | Planned (Phase 7) |
| R-006 | Credential theft (plain text logs) | All systems | 3 | 4 | 3 | 36 | Medium | Partially mitigated (access controls, audit) |
| R-007 | Privilege escalation (local) | WIN10, UBUNTU-SRV | 2 | 5
