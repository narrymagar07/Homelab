# Phase 1 — Host Setup

**Author:** Naresh Kumar Budha Magar
**Date:** 11 May 2026
**Lab:** nareshlab.local (Hyper-V homelab)
**Status:** Complete

---

## 1. Objective

Prepare the physical host machine to run Hyper-V virtualisation and create
an isolated lab network for future VMs (DC01, FS01, CLIENT01, CLIENT02,
HELPDESK01).

End state for this phase:
- Hyper-V role installed and operational
- Internal virtual switch created for the lab network
- Host has a static IP on the lab side
- Folder structure ready for VM files and ISOs

---

## 2. Host specifications

| Component        | Detail                                        |
|------------------|-----------------------------------------------|
| Model            | HP EliteDesk 800 G6 SFF                       |
| CPU              | Intel i7-10700 (8 cores / 16 threads)         |
| RAM              | 32 GB DDR4 3200 MHz|
| Storage          | 1 TB SATA SSD                                 |
| GPU              | Intel UHD Graphics 630 (integrated)           |
| Host OS          | Windows 11 Pro             |
| Firmware         | UEFI, Secure Boot enabled, TPM 2.0 enabled    |
| Virtualisation   | VT-x enabled in BIOS                          |

---

## 3. Network design

| Network              | Subnet              | Purpose                              |
|----------------------|---------------------|--------------------------------------|
| Home Wi-Fi           | 192.168.0.0/24      | Host internet access (DHCP)          |
| LAB-Switch (Internal)| 192.168.50.0/24     | Isolated lab network for VMs         |

**Reasoning:** Lab subnet kept separate from home network to prevent any
future lab DHCP server (DC01) from handing out IPs to home devices —
i.e. avoid a rogue DHCP situation. Internal switch type chosen so host
and VMs can communicate, but VMs cannot reach the home LAN or internet
directly. Internet access for VMs will be handled later via NAT or by
temporarily attaching to the Default Switch.

---

## 4. Steps performed

### 4.1 Renamed host

```powershell
Rename-Computer -NewName "Naresh" -Restart
```

> **Note for future:** "Naresh" is not a professional host name. Plan to
> rename to `LAB-HOST` before adding the host to any documentation
> shared externally (e.g. GitHub portfolio screenshots).

### 4.2 Enabled Hyper-V

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

Reboot required and completed.

### 4.3 Created internal virtual switch

```powershell
New-VMSwitch -Name "LAB-Switch" -SwitchType Internal
```

Verified with:

```powershell
Get-VMSwitch
```

Output showed both `Default Switch` (built-in NAT, left in place for
later internet access to VMs) and `LAB-Switch` (Internal, the new lab
network).

### 4.4 Assigned static IP to host's lab-side adapter

Found the adapter's `ifIndex`:

```powershell
Get-NetAdapter -Name "vEthernet (LAB-Switch)"
```

Result: `ifIndex = 50`, status `Up`.

Assigned IP:

```powershell
New-NetIPAddress -InterfaceIndex 50 -IPAddress 192.168.50.1 -PrefixLength 24
```

No gateway and no DNS configured on this adapter — it is internal only.
The Wi-Fi adapter continues to provide the host's internet via DHCP.

### 4.5 Created folder structure

```powershell
New-Item -Path "C:\Lab\VMs"  -ItemType Directory
New-Item -Path "C:\Lab\ISOs" -ItemType Directory
```

---

## 5. Verification

| Check                          | Command                                              | Result          |
|--------------------------------|------------------------------------------------------|-----------------|
| Hostname                       | `hostname`                                           | `Naresh`        |
| Hyper-V installed              | `Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V` | `Enabled` |
| Virtual switch present         | `Get-VMSwitch`                                       | `LAB-Switch` (Internal) |
| Host lab IP                    | `Get-NetIPAddress -InterfaceAlias "vEthernet (LAB-Switch)"` | `192.168.50.1/24` |
| Host internet access           | `ping 8.8.8.8`                                       | _to be tested_  |

---

## 6. Issues encountered

None during this phase.

---

## 7. Open items / follow-up

- [ ] Rename host from `Naresh` to `LAB-HOST` before external sharing
- [ ] Test internet connectivity from host (`ping 8.8.8.8`)
- [ ] Download Windows Server 2025 evaluation ISO into `C:\Lab\ISOs`
- [ ] Download Windows 11 Pro ISO into `C:\Lab\ISOs`
- [ ] Plan: add a second SSD by month 3 for VM storage separation

---

## 8. Next phase

**Phase 2 — Build DC01** (Windows Server 2025, AD DS + DNS + DHCP).
See `Phase-2-DC01-Build.md` (to be created).