# AD DS Home Lab Journey

A running log of building an Active Directory Domain Services lab from scratch in VMware Workstation — documenting what broke, why, and how it got fixed along the way.

## About this lab

- **Hypervisor:** VMware Workstation
- **VMs:** Windows Server 2025 (domain controller), Windows 11 Pro, Windows 11 Enterprise
- **Domain:** `oneil.local`
- **Goal:** Learn AD DS, DNS, DHCP, and Group Policy fundamentals through hands-on setup and real troubleshooting, not just following a checklist.

## Why document this

Most write-ups on AD DS setup show the "happy path" — click here, click there, done. This log intentionally keeps the mistakes and confusing moments in, because those are the parts that actually teach something (and the parts I'd otherwise forget once the setup feels routine).

## Entries

| # | Entry | Topics covered |
|---|-------|-----------------|
| 01 | [VMware Networking Setup](01-vmware-networking-setup.md) | Custom VMnets, host-only vs NAT, subnet/gateway config, DHCP range gotchas |
| 02 | [AD DS Learning Journal](02-ad-ds-learning-journal.md) | AD DS concepts, FSMO roles, Sites and Services, pfSense setup |
| 03 | DNS & DHCP Integration *(coming soon)* | DNS troubleshooting, secure dynamic updates, migrating DHCP off VMware onto the DC |
| 04 | Domain-Joining Clients *(coming soon)* | Client DNS config, domain join process, first domain login |
| 05 | Users, OUs & Group Policy *(coming soon)* | Structuring the directory, first GPOs |

## Environment reference

| VM | OS | Role | Static IP |
|----|-----|------|-----------|
| DC1 | Windows Server 2025 | Domain Controller / DNS / DHCP | 30.30.30.10 |
| — | Windows 11 Pro | Domain client | DHCP-assigned |
| — | Windows 11 Enterprise | Domain client | DHCP-assigned |

**Network:** VMnet3 (NAT) — `30.30.30.0/24`, gateway `30.30.30.2`
