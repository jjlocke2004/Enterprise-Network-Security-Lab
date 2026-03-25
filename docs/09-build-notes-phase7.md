# Phase 7 – Hardening and Re‑Assessment

For Phase 7 I chose three important findings in each of the Nessus assessments and applied host-level hardening controls and scanned the repaired to ensure the issues were resolved or addressed accordingly.

---

## 7.1 UBUNTU‑SRV – ICMP Timestamp Request Remote Date Disclosure

### 7.1.1 Original issue

Nessus had indicated that UBUNTU-SRV answered ICMP timestamp, providing system time data on the network, which may be useful in reconnaissance and time based attacks when left active.

![Before UFW rule change on UBUNTU-SRV](screenshots/build-notes-phase7-screenshots/before-ubuntu-srv-rule-change.png)

### 7.1.2 Control applied – host firewall hardening

On UBUNTU-SRV I have logged into the host firewall, ufw, and enabled and configured it to block unsolicited inbound traffic and implicitly block ICMP timestamp requests:

```bash
sudo ufw status verbose
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw enable
sudo ufw status verbose
```
Following these commands ufw appeared to be active with default policy of "deny (incoming), allow (outgoing) and SSH only.

![Verification of UFW rule change on UBUNTU-SRV](screenshots/build-notes-phase7-screenshots/verification-of-rule-change-ubuntu-srv.png)

### 7.1.3 Verification
I again scanned UBUNTU-SRV with the Nessus scan with the same scan policy and ensured that the "ICMP Timestamp Request Remote Date Disclosure" scan did not display the host as a victim any longer.

## 7.2 FS01 – SMB Signing Not Required
### 7.2.1 Original issue
Nessus marked FS01 as having the "SMB Signing Not Required" plugin, which means that the SMB traffic with the file server did not need to be digitally signed and was therefore susceptible to spoofing and man-in-the-middle attacks.
### 7.2.2 Control applied – enforce SMB signing on server
On FS01 I had SMB signing through the registry and restarted the server service:

```powershell
reg add "HKLM\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" /v RequireSecuritySignature /t REG_DWORD /d 1 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters" /v RequireSecuritySignature /t REG_DWORD /d 1 /f

net stop lanmanserver
net start lanmanserver
```
![FS01 SMB signing registry changes and service restart](screenshots/build-notes-phase7-screenshots/FS01-enable-SMB-signing-and-restart-LanmanServer.png)

### 7.2.3 Verification
Once SMB signing was enabled, I again started the authenticated Nessus scan against FS01 and noted that the SMB Signing Not Required finding was no longer applicable to the host.

![Authenticated DC/FS scan host view used for verification](screenshots/build-notes-phase7-screenshots/Nessus-lab-authenticated-DC-FS-host-vulnerability.png)

## 7.3 DC01 – SMB Shares Least‑Privilege Access (DeptShare_Test)
### 7.3.1 Context
The Nessus scan which was authenticated on the domain controller indicated that a low-privilege domain account (svc_scanner) was able to access default administrative shares ( SYSVOL and NETLOGON ). I did not modify the ACLs of these shares because they should not be overridden by the domain users when using Active Directory to log in and by Group Policy to create groups. I had instead shown the principle of least privilege on a dedicated lab data share, DeptShare_Test, which is hosted on DC01.
![DeptShare_Test combined NTFS and SMB share permissions before hardening](screenshots/build-notes-phase7-screenshots/DC01-DeptShare_Test-NTFS-and-SMB-share-permissions.png)

### 7.3.2 Broad access configuration (before)
First, I had specified DeptShare_Test using too permissive so that I could model a real world misconfiguration.

Created the data folder:

```powershell
New-Item -ItemType Directory -Path 'C:\DeptShare_Test'
```
Granted broad NTFS permissions:

```powershell
icacls C:\DeptShare_Test /grant "Everyone:(OI)(CI)M"
icacls C:\DeptShare_Test
```
Shared the folder over SMB and allowed broad share‑level access:

```powershell
New-SmbShare -Name 'DeptShare_Test' -Path 'C:\DeptShare_Test' -FullAccess 'Everyone'
Get-SmbShareAccess -Name 'DeptShare_Test'
```
At this stage, DeptShare_Test permitted Everyone and Authenticated Users Full access at the share level and added Everyone to the NTFS ACL, which does not correspond with least-privilege access control.

### 7.3.3 Hardening DeptShare_Test to least privilege
Share-level hardening
I limited the access to the share level to the groups that ought to be able to manage or use the share:
```powershell
Revoke-SmbShareAccess -Name 'DeptShare_Test' -AccountName 'Everyone' -Force
Revoke-SmbShareAccess -Name 'DeptShare_Test' -AccountName 'Authenticated Users' -Force

Grant-SmbShareAccess -Name 'DeptShare_Test' -AccountName 'CORP\Domain Admins' -AccessRight Full -Force
Grant-SmbShareAccess -Name 'DeptShare_Test' -AccountName 'CORP\HR_Share_RW' -AccessRight Full -Force

Get-SmbShareAccess -Name 'DeptShare_Test'
```
![DeptShare_Test share-level hardening commands and final access list](screenshots/build-notes-phase7-screenshots/DC01-DeptShare_Test-SMB-share-permissions-updated-DomainAdmins-and-HR_group.png)

Finally, there are final share permissions, which indicate only access of CORP Domain Admins and CORP HR Share RW, deleting access of Everyone and Authenticated Users.
NTFS hardening
Then I added NTFS permissions to the share-level model:

```powershell
icacls C:\DeptShare_Test /remove:g "Everyone"
icacls C:\DeptShare_Test /grant:r "CORP\Domain Admins:(OI)(CI)F"
icacls C:\DeptShare_Test /grant:r "CORP\HR_Share_RW:(OI)(CI)F"
icacls C:\DeptShare_Test
```
![DeptShare_Test final NTFS ACL after hardening](screenshots/build-notes-phase7-screenshots/DC01-DeptShare_Test-NTFS-permissions-domain-admins-and-HR_group.png)

The last ACL has the same access as CORP\Domain Admins and CORP\HR_Share_RW (as well as local system/administrators and creator owner), but does not include the everyone anymore.
### 7.3.4 Verification and rationale
I re-executed the authenticated Nessus scan against DC01 and FS01 after hardening DeptShare Test to ensure the environment.

![Authenticated DC/FS detailed vulnerability list used for verification](screenshots/build-notes-phase7-screenshots/Nessus-lab-authenticated-DC-FS-detailed-vulnerability-list.png)

The default permissions of SYSVOL and NETLOGON have been left deliberately since they are needed by domain logon and processing group policies. I did not change those critical shares but, to show a realistic least-privilege process on a data share, I used DeptShare_Test to illustrate the choices of broad access, risk identification, and narrowing to only the required groups and then verified through both configuration output and subsequent vulnerability scanning.
## 7.4 Scan baselines
To complete the picture, I also maintained unauthenticated scan baselines to compare it to the authenticated results as well as the hardening work done during this stage.

![Lab Unauthenticated scan – host summary](screenshots/build-notes-phase7-screenshots/Nessus-lab-unauthenticated-scan-results.png)

![Lab Unauthenticated scan – vulnerabilities view](screenshots/build-notes-phase7-screenshots/Nessus-lab-unauthenticated-vuln-list.png)

These baselines point out the extra visibility of the findings that can only be done once credentialed scanning is made possible and the background to the remediation effort performed during Phase 7.
