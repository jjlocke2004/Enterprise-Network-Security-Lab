# RMF Asset Inventory and Risk Baseline – Enterprise Network Security Lab

**Document Version:** 1.0  
**Date:** February 2026  
**System:** Enterprise Network Security Lab (ENSL)  
**Classification:** Internal Use

---

## 1. Executive Summary

This document defines the System Boundary, Asset Inventory, Data Classification, Threat Profile, and Risk Matrix for the Enterprise Network Security Lab (ENSL). The ENSL is a virtualized enterprise environment designed to simulate a small company's IT infrastructure (approximately 50–100 employees) with Windows and Linux systems, Active Directory, file services, and centralized auditing.

**Purpose:** To establish a baseline understanding of the system's assets, vulnerabilities, and risk posture in preparation for vulnerability assessment (Phase 5), hardening (Phase 6), and ongoing security monitoring.

---

## 2. System Boundary and Scope

### 2.1 Defined System Boundary

The ENSL system encompasses the following virtualized components running on **VMware Workstation** within the **192.168.10.0/24** network segment:

| Component | Included | Rationale |
|-----------|----------|-----------|
| DC01, FS01, WIN10, UBUNTU-SRV, SEC-TOOLS VMs | ✓ In Scope | Core lab infrastructure |
| VMware Workstation hypervisor | ✓ In Scope | Host platform for lab VMs |
| Physical lab network (192.168.10.0/24) | ✓ In Scope | Lab subnet connectivity |
| External networks / Internet | ✗ Out of Scope | Lab is air-gapped; no internet connectivity |
| Backup systems | ✗ Out of Scope | Not deployed in current phases |
| Disaster recovery infrastructure | ✗ Out of Scope | Not deployed in current phases |

### 2.2 Environment Type

- **Classification:** Isolated Test/Development Lab
- **Access:** Local physical lab at University at Albany, Albany, NY
- **Network Connectivity:** Air-gapped (no internet access; internal lab network only)
- **Regulatory Applicability:** N/A (non-production educational environment)

---

## 3. Asset Inventory

### 3.1 Hardware and Virtual Infrastructure

| Asset ID | Asset Name | Type | Specs | Location | Criticality | Owner |
|----------|-----------|------|-------|----------|------------|-------|
| HW-001 | VMware Workstation Host | Hypervisor | Windows/Linux desktop with 16+ GB RAM, 250+ GB storage | Lab | Critical | Student Lab |
| HW-002 | Lab Network Switch (VMnet1) | Virtual Switch | VMware network adapter | Lab | Critical | VMware |

### 3.2 Virtual Machine Assets

| Asset ID | Hostname | Role | IP Address | OS | vCPU | RAM | Disk | Criticality | Data Sensitivity |
|----------|----------|------|------------|----|----|-----|------|------------|------------------|
| VM-001 | DC01 | Domain Controller | 192.168.10.10 | Windows Server 2019+ | 2 | 4 GB | 60 GB | **Critical** | **High** |
| VM-002 | FS01 | File Server | 192.168.10.11 | Windows Server 2019+ | 2 | 4 GB | 100 GB | **Critical** | **High** |
| VM-003 | WIN10 | Workstation | 192.168.10.30 | Windows 10 Enterprise | 2 | 4 GB | 80 GB | **Medium** | **Medium** |
| VM-004 | UBUNTU-SRV | Linux Server | 192.168.10.20 | Ubuntu Server 22.04 | 2 | 4 GB | 60 GB | **Medium** | **Medium** |
| VM-005 | SEC-TOOLS | Security Tools | 192.168.10.50 | Kali Linux | 2 | 4 GB | 80 GB | **Low** | **Low** |

### 3.3 Active Directory Assets

| Asset ID | Object Type | Name | Location (OU) | Count | Criticality |
|----------|-------------|------|----------------|-------|------------|
| AD-001 | Forest | corp.local | – | 1 | Critical |
| AD-002 | Domain | corp.local | – | 1 | Critical |
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

| Asset ID | Service | Host(s) | Port | Protocol | Status | Criticality |
|----------|---------|---------|------|----------|--------|------------|
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
| DATA-001 | HR Share | `\\FS01\HR` (C:\Dept\HR) | HR files, documents, employee records | hgreen (HR) | None | High |
| DATA-002 | Finance Share | `\\FS01\Finance` (C:\Dept\Finance) | Financial data, budgets, reports | fsmith (Finance) | None | High |
| DATA-003 | Sales Share | `\\FS01\Sales` (C:\Dept\Sales) | Sales reports, client data, opportunities | sjones (Sales) | High | Medium |
| DATA-004 | IT Share | `\\FS01\IT` (C:\Dept\IT) | IT documentation, scripts, configs | iit (IT) | None | Medium |
| DATA-005 | Security Event Logs | DC01, FS01 (Security log) | Windows Security events (4663, logon, policy) | System | None | High |
| DATA-006 | Linux Audit Logs | UBUNTU-SRV (/var/log/audit/audit.log) | auditd file system events, access logs | System | None | Medium |

---

## 4. Data Classification

### 4.1 Classification Levels

| Level | Definition | Examples in Lab | Handling |
|-------|-----------|-----------------|----------|
| **Public** | No confidentiality impact if disclosed | Lab documentation, public screenshots | No encryption required |
| **Internal** | Should not be widely distributed; moderate confidentiality | IT documentation, general logs | Access restricted to IT team |
| **Sensitive** | High confidentiality; disclosure would impact business | HR employee records, financial data, credentials | Encrypted, access-controlled, audited |
| **Restricted** | Highest confidentiality; regulatory/compliance implications | Domain admin credentials, production backups | Strongest controls; minimal access |

### 4.2 Lab Data Classification Mapping

| Data Type | Classification | Owner | Access Control | Audit Logging |
|-----------|----------------|-------|-----------------|----------------|
| Domain credentials (CORP\Administrator) | **Restricted** | Lab Admin | IT Admins only | Yes (4663 + auditd) |
| HR employee records (names, roles, salaries implied) | **Sensitive** | HR Department | HR_Share_RW group | Yes (4663) |
| Finance reports and budgets | **Sensitive** | Finance Department | Finance_Share_RW group | Yes (4663) |
| Sales client data | **Sensitive** | Sales Department | Sales_Share_RW group | Yes (4663) |
| IT configurations and scripts | **Internal** | IT Department | IT_Admins + iit | Yes (4663) |
| User access logs (4663, auditd events) | **Internal** | IT Department | IT Admins | Yes (centralized) |
| Lab documentation and build notes | **Public** | Lab / GitHub | Public (GitHub repo) | No |

---

## 5. Threat Profile

### 5.1 Threat Actors and Motivations

| Threat Actor | Type | Motivation | Capability | Likelihood | Impact |
|--------------|------|-----------|-----------|-----------|--------|
| **Insider – Disgruntled HR User** | Internal | Data theft, sabotage | Medium (access to HR share) | Medium | High (HR data exposure) |
| **Insider – Finance User** | Internal | Fraud, unauthorized transfers | Medium (access to Finance data) | Medium | High (financial data loss) |
| **External – Network Attacker** | External | Reconnaissance, lateral movement | Medium (if air-gap compromised) | Low (air-gapped lab) | High (if exploited) |
| **Misconfigured User** | Accidental | Unintended data access | Low (permissions) | Medium | Medium (accidental disclosure) |
| **Privilege Escalation Attack** | Internal | Unauthorized admin access | Medium | Low–Medium | Critical (domain compromise) |

### 5.2 Common Attack Vectors in Scope

| Vector | Description | Current Mitigation | Residual Risk |
|--------|------------|-------------------|----------------|
| **Weak Passwords** | Short, guessable domain passwords | No password policy enforced | High |
| **Unpatched Systems** | Missing Windows/Linux patches | No patching baseline | High |
| **Legacy SMB (SMBv1)** | Insecure file sharing protocol | SMBv2/v3 enabled (default) | Low |
| **Excessive NTFS Permissions** | Overly permissive file shares | Least-privilege configured | Low |
| **Credential Theft** | Captured domain credentials | No MFA; plain credentials in logs | Medium |
| **Privilege Escalation** | Local privilege escalation to admin | Standard user accounts only | Medium |
| **Lateral Movement** | Attacker moves across systems | No network segmentation | High |
| **Unauthorized Logons** | Bruteforce or credential replay attacks | No account lockout policy audited | Medium |

---

## 6. Risk Assessment Matrix

### 6.1 Risk Scoring Methodology

**Risk Score = Likelihood × Impact × Exploitability**

- **Likelihood:** 1 (Low) to 5 (High) – How often threat occurs
- **Impact:** 1 (Low) to 5 (High) – Business/security consequence if realized
- **Exploitability:** 1 (Hard) to 5 (Easy) – How easy to exploit without mitigations

**Risk Level:**
- **Critical:** 60–125 (Immediate action required)
- **High:** 40–59 (Action required within 1–2 weeks)
- **Medium:** 20–39 (Plan action within 1 month)
- **Low:** 1–19 (Monitor; plan action if feasible)

### 6.2 Risk Matrix – Current State (Phase 4)

| Risk ID | Threat | Asset(s) Affected | Likelihood | Impact | Exploitability | Score | Level | Mitigation Status |
|---------|--------|------------------|-----------|--------|----------------|-------|-------|------------------|
| **R-001** | Weak password policy | All user accounts | 4 | 5 | 5 | **100** | **Critical** | Unaddressed (Phase 6) |
| **R-002** | Unpatched OS vulnerabilities | DC01, FS01, WIN10, UBUNTU-SRV | 4 | 5 | 4 | **80** | **Critical** | Unaddressed (Phase 6) |
| **R-003** | Unauthorized file access / data theft | HR, Finance, Sales shares | 3 | 5 | 3 | **45** | **High** | Partially mitigated (NTFS/share permissions, 4
