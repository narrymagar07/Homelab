# Phase 3: Client VM Deployment and Help Desk Management Setup

## Overview

In this phase, multiple Windows client virtual machines were created and configured to simulate a small business Active Directory environment within the `nareshlab.local` domain.

The main objective of this phase was to:

- Deploy domain-connected user workstations
- Configure centralized authentication using Active Directory
- Create a dedicated Help Desk support workstation
- Install and test RSAT administration tools
- Configure remote support access using Group Policy
- Test domain login and workstation management

---

# Lab Infrastructure Overview

| VM | Role | IP Address | Subnet | Gateway | DNS |
| --- | --- | --- | --- | --- | --- |
| DC01 | Domain Controller, DNS, DHCP | 192.168.50.10 | 255.255.255.0 | 192.168.50.1 | 192.168.50.10 (itself) |
| FS-01 | File Server | 192.168.50.20 | 255.255.255.0 | 192.168.50.1 | 192.168.50.10 |
| PC-01 | End-user workstation | 192.168.50.100 | 255.255.255.0 | 192.168.50.1 | 192.168.50.10 |
| PC-02 | End-user workstation | 192.168.50.101 | 255.255.255.0 | 192.168.50.1 | 192.168.50.10 |
| HELPDESK-01 | Support technician workstation | 192.168.50.102 | 255.255.255.0 | 192.168.50.1 | 192.168.50.10 |

---

# Virtual Machine Specifications

| VM Name | Specifications |
| --- | --- |
| PC-01 | 4 GB RAM, 60 GB SSD, Windows 11 Pro, Generation 2, Internal Lab Switch |
| PC-02 | 4 GB RAM, 60 GB SSD, Windows 11 Pro, Generation 2, Internal Lab Switch |
| HELPDESK-01 | 4 GB RAM, 60 GB SSD, Windows 11 Pro, Generation 2, Internal Lab Switch |
| FS-01 | 3 GB RAM, 60 GB SSD, Windows Server 2025, Generation 2, Internal Lab Switch |

---

# Domain Configuration

All systems were configured within the following Active Directory domain:

**Domain:**

```
nareshlab.local
```

**DHCP Range:**

```
192.168.50.100 – 192.168.50.200
```

**Virtual Switch:**

```
LAB-Switch
```

(Internal)

Client systems were successfully joined to the domain and authenticated using Active Directory user credentials.

DNS settings for all machines were configured to use the Domain Controller (`192.168.50.10`) to ensure proper domain name resolution and Active Directory communication.

---

# Help Desk Workstation Setup

The `HELPDESK-01` virtual machine was configured as a centralized support and administration workstation.

Remote Server Administration Tools (RSAT) were installed to allow server and Active Directory management without directly logging into the domain controller.

Installed management tools included:

- Active Directory Users and Computers (ADUC)
- DNS Manager
- DHCP Manager
- Group Policy Management Console (GPMC)

These tools were used to:

- Manage users and security groups
- Create and organize Organizational Units (OUs)
- Configure DNS records
- Monitor DHCP scopes and leases
- Create and manage Group Policy Objects (GPOs)

---

# User Login and Domain Testing

Domain login testing was performed on both client workstations:

- USER-01
- USER-02

Successful testing included:

- Domain authentication
- User credential validation
- Network communication with the domain controller
- Administrative access from HELPDESK-01
- Basic workstation management using RSAT tools

---

# Remote Desktop Configuration and Troubleshooting

Remote support functionality was configured to simulate real-world IT support scenarios.

Initially, users were unable to log in remotely because Remote Desktop access permissions had not yet been configured.

## Issue Identified

Remote login attempts failed due to missing Remote Desktop authorization and policy configuration on client machines.

## Resolution

The issue had two root causes:

1. The computer object for PC-01 had not been moved from the
default "Computers" container to the correct OU
(IT Solutions → Computers → Workstations). GPOs linked to
the IT Solutions OU do not apply to objects in the default
Computers container.
2. Domain users were not members of the local "Remote Desktop
Users" group on the workstation.

A GPO named "Allow Remote Desktop Users" was created and linked
to IT Solutions → Computers. Under Computer Configuration →
Policies → Windows Settings → Security Settings → Restricted
Groups, the NARESHLAB\GG-IT group was added as a member of
"Remote Desktop Users."

PC-01 was then moved to the correct OU in ADUC, and
gpupdate /force was run on the workstation to apply the policy
immediately. Login as john.smith was confirmed successful.

---

# Current Limitations

The following components were successfully completed:

- VM deployment
- Domain join configuration
- User login testing
- RSAT administration setup
- Basic remote support configuration

However, advanced testing has not yet been completed, including:

- Edge-case login scenarios
- Advanced security restrictions
- Concurrent remote sessions
- Network failure simulations
- Failed login auditing
- User permission conflict testing

Further validation and troubleshooting scenarios will be implemented in future phases.

---

# Planned Next Phase

The next phase of the homelab project will focus on File Server deployment and access management.

Planned tasks include:

- Shared folder configuration
- NTFS permission management
- Department-based folder access
- Security group-based permissions
- Network drive mapping
- File access testing between users

---

# Recommended Screenshots

The following screenshots will be added in future updates:

- Hyper-V VM overview
- Successful domain join
- Active Directory Users and Computers
- DNS Manager configuration
- DHCP scope configuration
- Group Policy Management Console
- Remote Desktop login testing
- Help Desk workstation management tools

---