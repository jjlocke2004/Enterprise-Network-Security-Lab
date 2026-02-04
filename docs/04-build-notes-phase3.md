# Build Notes – Phase 3: Enterprise File Auditing and Server Hardening

This phase records the work done following Phase 2 in terms of Phase 2 work recorded in the AD structure and basic department shares, with a focus on cleanup of hostnames in servers, placement of OU, GPO baselines, file access auditing, and validation. It also captures a correction in which the IT department folder and user should have been included but was overlooked in Phase 2.

---

## Section A – Server Hostname Cleanup and OU Placement

### A.1 Rename File Server to FS01

Goal: Replace the random Windows hostname with a clean, enterprise-style name.

On the original file server:

```powershell
hostname    # shows WIN-3LMB8KE361E (example)

Rename-Computer -NewName "FS01" -DomainCredential CORP\Administrator -Restart
```

![Before changing hostname to FS01](../screenshots/build-notes-phase3-screenshots/Before_Changing_HostName_To_FS01.jpg)

![After changing hostname to FS01](../screenshots/build-notes-phase3-screenshots/After_Changing_HostName_to_FS01.jpg)

Repeat this for DC01 and the Windows Client as well.

### A.2 Move Servers into the Servers OU
All directory work was done on DC01 with the ActiveDirectory module.

```powershell
Import-Module ActiveDirectory

Move-ADObject "CN=DC01,CN=Computers,DC=corp,DC=local" `
  -TargetPath "OU=Servers,OU=Corp,DC=corp,DC=local"

Move-ADObject "CN=FS01,CN=Computers,DC=corp,DC=local" `
  -TargetPath "OU=Servers,OU=Corp,DC=corp,DC=local"
```
Verification:

```powershell
Get-ADComputer -Filter * -SearchBase "OU=Servers,OU=Corp,DC=corp,DC=local" |
  Select Name,DistinguishedName
```
![Verfitication_of_servers_in_server_OU](../screenshots/build-notes-phase3-screenshots/Verfitication_of_servers_in_server_OU.jpg)

### A.3 Create and Link “Server-Audit-Baseline” GPO
Goal: Centralize server audit settings and prepare for file-access and logon-event monitoring.

```powershell
Import-Module GroupPolicy

New-GPO   -Name "Server-Audit-Baseline"
New-GPLink -Name "Server-Audit-Baseline" -Target "OU=Servers,OU=Corp,DC=corp,DC=local"
```
![Linking Server-Audit-Baseline GPO to Servers OU](../screenshots/build-notes-phase3-screenshots/Setting_GPLink_to_OU.jpg)

Within the GPO, registry-based settings were configured to enforce advanced auditing:

```powershell
# Example: enable Audit Logon policy via registry in the GPO
Set-GPRegistryValue -Name "Server-Audit-Baseline" `
  -Key "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System\Audit" `
  -ValueName "AuditLogon" -Type DWord -Value 1

# Example: prevent legacy audit policy from overriding advanced audit policy
Set-GPRegistryValue -Name "Server-Audit-Baseline" `
  -Key "HKLM\System\CurrentControlSet\Control\Lsa" `
  -ValueName "SCENoApplyLegacyAuditPolicy" -Type DWord -Value 1
```
![Set-GPRegistryValue for AuditLogon (1)](../screenshots/build-notes-phase3-screenshots/Set-GPRegistryValue_AuditLogon.jpg)

![Set-GPRegistryValue for SCENoApplyLegacyAuditPolicy (1)](../screenshots/build-notes-phase3-screenshots/Set-GPRegistryValue_SCENoApplyLegacyAuditPolicy.jpg)


After configuring the GPO, a remote policy refresh was triggered on both servers:

```powershell
Invoke-GPUpdate -Computer "DC01" -Force
Invoke-GPUpdate -Computer "FS01" -Force
```
![Invoking GPUpdate on both servers](../screenshots/build-notes-phase3-screenshots/Invoke-GPUpdate_to_both_servers.jpg)
(Invoke-GPUpdate_to_both_servers)

## Section B – File Access Auditing on FS01
All file-auditing work in this section was done on FS01 using PowerShell.

### B.1 Create Missing IT Department Folder and User
A missed configuration from Phase 2 was corrected by adding the IT department folder and user.

On FS01, create the missing IT folder:

```powershell
New-Item -Path "C:\Dept\IT" -ItemType Directory -Force
```
![Creating IT department folder](../screenshots/build-notes-phase3-screenshots/Creating_IT_Folder.jpg)
(Creating_IT_Folder)

On DC01, create Ian IT (iit):

```powershell
Import-Module ActiveDirectory
$Password = Read-Host "Enter password for Ian IT" -AsSecureString

New-ADUser -Name "Ian IT" -SamAccountName "iit" -UserPrincipalName "iit@corp.local" `
  -Path "OU=IT,OU=Users,OU=Corp,DC=corp,DC=local" -AccountPassword $Password -Enabled $true
```
![Creating new IT user iit](../screenshots/build-notes-phase3-screenshots/Creating_New_User_iit.jpg)
(Creating_New_User_iit)

Grant read/write access for iit on the IT share and folder:

```powershell
Grant-SmbShareAccess -Name "IT" -AccountName "CORP\iit" -AccessRight Change -Force
icacls "C:\Dept\IT" /grant "CORP\iit:(OI)(CI)M"
```
![icacls output for IT folder including new user](../screenshots/build-notes-phase3-screenshots/Icacls_output_for_IT_folder_including_newuser.jpg)
(Icacls_output_for_IT_folder_including_newuser)

### B.2 Enable File System Auditing Policy
Goal: Ensure Windows generates 4663 object access events.

Initial HR-only configuration:

```powershell
auditpol /set /subcategory:"File System" /success:enable /failure:enable
```
![Enabling file system auditing for HR](../screenshots/build-notes-phase3-screenshots/enabling_file_system_auditng_HR.jpg)
(enabling_file_system_auditng_HR)

Extended to all department folders as final state:

```powershell
auditpol /get /subcategory:"File System" /success:enable /failure:enable
```

![Enabling file system auditing for all dept folders](../screenshots/build-notes-phase3-screenshots/Enabling_File_system_auditing_2.jpg)
(Enabling_File_system_auditing_2)

### B.3 Configure SACLs for Department Folders
```powershell
$folders = @(
  "C:\Dept\HR",
  "C:\Dept\IT",
  "C:\Dept\Finance",
  "C:\Dept\Sales"
)

$account = "CORP\Domain Users"

foreach ($path in $folders) {
    $acl = Get-Acl $path
    $auditRule = New-Object System.Security.AccessControl.FileSystemAuditRule(
        $account,
        "Read, Write",
        "ContainerInherit, ObjectInherit",
        "None",
        "Success"
    )
    $acl.AddAuditRule($auditRule)
    Set-Acl -Path $path -AclObject $acl
}
```
Verification:

```powershell
(Get-Acl "C:\Dept\HR").Audit
(Get-Acl "C:\Dept\IT").Audit
```
## Section C – Generating and Reviewing 4663 Events
### C.1 Generate Test Access from WIN10
On WIN10 as HR user hgreen:

```powershell
New-PSDrive -Name H -PSProvider FileSystem -Root "\\FS01\HR" -Persist
```
hgreen creates and edits an audit-test-txt.txt file in the HR share to trigger events.

![Creating audit-test-txt.txt as hgreen](../screenshots/build-notes-phase3-screenshots/Creating_audit_test_txt_as_hgreen.jpg)
(Creating_audit_test_txt_as_hgreen)

(Repeat similar steps for IT, Finance, and Sales users as needed.)

### C.2 Verify 4663 Events for Department Folders
On FS01, confirm 4663 events are recorded:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4663} -MaxEvents 20 |
  Format-List TimeCreated, Id, Message
```
Filter by HR path:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4663} -MaxEvents 100 |
  Where-Object { $_.Message -like '*C:\Dept\HR\*' } |
  Format-List TimeCreated, Message
```
![4663 event – HR user access proof](../screenshots/build-notes-phase3-screenshots/4663_Event_HR_User_Access_Proof.jpg)
(4663_Event_HR_User_Access_Proof)

You can repeat the same query with C:\Dept\IT, C:\Dept\Finance, and C:\Dept\Sales to confirm auditing for the other departments.

## Section D – Phase 3 Outcome
By the end of Phase 3:

The servers (DC01 and FS01) have clean hostnames and are under Servers OU that is linked to Server-Audit-Baseline GPO.

Registry based setting of GPO implements advanced auditing and block legacy audit policy overriding it, and GPUpdate is propagated to both servers.

The Phase 2 configuration was read and corrected by adding IT department folder, creating user Ian IT (iit) and allowing him to have a read/write access to the IT share.

The audits of files systems are turned on both at the policy and SACL level of all data in departments.

Domain user access, such as hgreen and iit, is now consistent and gives rise to 4663 Security log events indicating user access to which department files, which will later be central to the collection of logs and analysis by SIEM.
