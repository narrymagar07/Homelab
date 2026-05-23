---

# Phase 4: File Server Deployment and Access Control Configuration

## Overview

In this phase, a dedicated file server was deployed to provide department-based shared folder access within the `nareshlab.local` domain.

The file server was configured with shared folders for Sales, Finance, and IT. Access was controlled using Active Directory security groups and NTFS permissions.

---

## Server Configuration

| Component | Details |
| --- | --- |
| Server Name | FS-01 |
| Operating System | Windows Server 2025 |
| RAM | 3 GB |
| Disk | 60 GB |
| Network | LAB-Switch |
| IP Address | 192.168.50.20 |
| Domain | nareshlab.local |

---

## File Server Role Installation

The File Server role was installed using PowerShell:

```powershell
Install-WindowsFeature -Name FS-FileServer -IncludeManagementTools
```

---

## Folder Structure

The following folders were created under:

```
C:\Shares\
```

| Folder | Intended Access |
| --- | --- |
| Sales | GG-Sales |
| Finance | GG-Finance |
| IT | GG-IT |

---

## Share Permission Configuration

The SMB shares were created using the following PowerShell commands:

```powershell
New-SmbShare -Name "Sales" -Path "C:\Shares\Sales" -FullAccess "NARESHLAB\GG-Sales"

New-SmbShare -Name "Finance" -Path "C:\Shares\Finance" -FullAccess "NARESHLAB\GG-Finance"

New-SmbShare -Name "IT" -Path "C:\Shares\IT" -FullAccess "NARESHLAB\GG-IT"
```

---

## NTFS Permission Configuration

NTFS inheritance was removed from each department folder. Each department security group was granted Full Control only to its own folder. Domain Admins were also granted Full Control for administration.

### Sales Folder

```powershell
icacls "C:\Shares\Sales" /inheritance:r
icacls "C:\Shares\Sales" /grant "NARESHLAB\GG-Sales:(OI)(CI)F"
icacls "C:\Shares\Sales" /grant "NARESHLAB\Domain Admins:(OI)(CI)F"
```

### Finance Folder

```powershell
icacls "C:\Shares\Finance" /inheritance:r
icacls "C:\Shares\Finance" /grant "NARESHLAB\GG-Finance:(OI)(CI)F"
icacls "C:\Shares\Finance" /grant "NARESHLAB\Domain Admins:(OI)(CI)F"
```

### IT Folder

```powershell
icacls "C:\Shares\IT" /inheritance:r
icacls "C:\Shares\IT" /grant "NARESHLAB\GG-IT:(OI)(CI)F"
icacls "C:\Shares\IT" /grant "NARESHLAB\Domain Admins:(OI)(CI)F"
```

---

## Issue Encountered

While using ADUC from `HELPDESK-01`, an attempt was made to move the `FS-01` computer object.

The following error occurred:

```
Access Denied
```

---

## Root Cause

The account being used, `naresh.kumar`, was a standard domain user account.

This account did not have permission to move computer objects in Active Directory.

---

## Resolution

ADUC was reopened using:

```
Run as different user
```

The following administrator account was used:

```
NARESHLAB\Administrator
```

After opening ADUC with administrator privileges, the `FS-01` computer object was successfully moved.

---

## Verification Testing

Testing was completed from `PC-01`.

Logged in as:

```
john.smith
```

Access tests:

| Network Path | Result |
| --- | --- |
| `\\FS-01\Sales` | Access successful |
| `\\FS-01\Finance` | Access denied |

The test confirmed that folder access was being controlled correctly based on group membership.

---

## Key Concepts Covered

### Share Permissions and NTFS Permissions

Share permissions control access when users connect over the network.

NTFS permissions control access at the file system level.

When both permission types apply, the most restrictive permission is effective.

### Permission Strategy Used

In this setup, access was controlled using both SMB share permissions and NTFS permissions.

Each department group was granted access only to its own shared folder.

---

## Notes for Future Improvement

In a future phase, this setup can be improved by:

- Mapping department drives using Group Policy
- Enabling Access-Based Enumeration
- Testing access with multiple users from different groups
- Creating read-only and modify-level permission scenarios
- Adding file access auditing
- Testing backup and restore procedures