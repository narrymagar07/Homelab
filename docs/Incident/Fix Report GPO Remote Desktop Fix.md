# Incident/Fix Report — GPO Remote Desktop Fix

**Date:** 21 May 2026
**Environment:** nareshlab.local (Hyper-V homelab — Windows 11 VM)
**Affected Machine:** PC-01
**Affected User:** john.smith (NARESHLAB\john.smith)
**Documented by:** Naresh Kumar Budha Magar

---

## 1. Problem Description

While setting up Enhanced Session on PC-01, the domain user
john.smith was unable to log in. The error message stated:

"To sign in remotely, you need the right to sign in through
Remote Desktop Services. Members of the Remote Desktop Users
group have this right."

The user was blocked from logging in entirely.

## 2. Root Cause

When Remote Desktop was enabled on PC-01, domain users were
not automatically granted access. Only local administrators
have Remote Desktop access by default. Additionally, the PC-01
computer object was sitting in the default "Computers"
container in Active Directory instead of the correct OU
(IT Solutions → Computers → Workstations), which meant any
GPO linked to the correct OU would not apply to it.

## 3. Fix Applied

Step 1 — Created a new GPO called "Allow Remote Desktop Users"
linked to IT Solutions → Computers OU in Group Policy
Management on DC01.

Step 2 — Edited the GPO: Computer Configuration → Policies →
Windows Settings → Security Settings → Restricted Groups.
Added "Remote Desktop Users" group with NARESHLAB\GG-IT
as a member.

Step 3 — Renamed PC-01 from its default hostname
(DESKTOP-6Q9RQED) to PC-01 using:
Rename-Computer -NewName "PC-01" -DomainCredential
"NARESHLAB\Administrator" -Restart

Step 4 — Moved the PC-01 computer object in ADUC from the
default Computers container to:
IT Solutions → Computers → Workstations

Step 5 — Ran gpupdate /force on PC-01 to apply the GPO
immediately.

## 4. Verification

Logged in successfully as NARESHLAB\john.smith on PC-01 via
Enhanced Session in Hyper-V Manager. No error message appeared.

## 5. Lesson Learned

When adding a new machine to the domain, always move the
computer object to the correct OU immediately after it joins.
GPOs only apply to objects in the linked OU — leaving a machine
in the default Computers container means it receives no custom
policies.

For future VM setups, plan Remote Desktop access requirements
before deployment and include the correct OU placement as part
of the standard build checklist.