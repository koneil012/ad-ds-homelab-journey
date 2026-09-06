# 01 — VMware Networking Setup

Before touching Active Directory at all, the first job was getting three VMs (Windows Server 2025, Windows 11 Pro, Windows 11 Enterprise) able to talk to each other **and** reach the internet — inside VMware Workstation.

Coming from Hyper-V, where a "virtual switch" is a single straightforward concept, VMware's networking model took a bit of adjusting to.

## The goal

Two logically separate use cases:

1. VMs need to talk to each other (for AD DS, DNS, domain joins, etc.)
2. VMs also need internet access (Windows Updates, downloading tools)

## Setting it up: Virtual Network Editor

VMware Workstation's equivalent of a virtual switch lives in **Edit > Virtual Network Editor**. Each network there is a "VMnet."

I created two custom networks:

- **VMnet2** — Host-only, subnet `20.20.20.0/24` (isolated, no internet — kept for testing pure VM-to-VM isolation)
- **VMnet3** — NAT, subnet `30.30.30.0/24`, gateway `30.30.30.2` (VM-to-VM communication **and** internet access)

In the end, VMnet3 (NAT) turned out to be the one doing all the real work, since it satisfies both requirements (inter-VM + internet) at once — VMnet2 mostly stayed as a "what would isolation look like" reference point.

**Gotcha #1:** VMware doesn't always auto-assign a subnet to a custom network — you have to explicitly set the subnet IP and mask in the Virtual Network Editor, or the network won't route or NAT traffic properly.

## Assigning static IPs

The domain controller (`DC1`) needed a static IP so DNS records and client configuration would stay stable: `30.30.30.10`.

**Gotcha #2 — the "no internet after setting a static IP" scare:**
Right after setting DC1's static IP, internet access broke — but `ping 8.8.8.8` still worked fine. That distinction mattered:

- `ping <raw IP>` bypasses DNS entirely — it only tests routing.
- Browsing, `ping google.com`, and Windows' own connectivity check all depend on **DNS resolution**.

The actual cause: DNS was set to `127.0.0.1` (pointing at itself) before the AD DS/DNS server role was even installed — so there was no DNS server actually listening on that address yet. The fix was temporarily pointing DNS at a public resolver (`8.8.8.8` / `1.1.1.1`) until the DNS role was live, then switching back to `127.0.0.1` afterward.

*Initial theory (wrong, but worth noting):* I first assumed the DHCP scope's starting range being too high (`.128`–`.254`) was somehow conflicting with the static IP. It wasn't — a static IP outside a DHCP pool doesn't cause conflicts on its own. The DHCP range change and the actual fix (DNS) just happened close together, which made it look related at first. Good reminder to isolate variables one at a time instead of changing two things and assuming the last change was the fix.

## Migrating DHCP off VMware and onto the domain controller

Once AD DS was in and DHCP was going to run on Windows Server instead, I had to retire VMware's built-in DHCP for VMnet3 — but not immediately.

**Gotcha #3 — sequencing matters:**
Disabling VMware's DHCP *before* the Windows Server DHCP scope was configured, activated, **and** authorized would have left a window where no DHCP server was answering client requests at all — risking VMs falling back to self-assigned APIPA addresses.

Correct order that actually worked:

1. Finish configuring the new DHCP scope on the domain controller (start/end range, exclusions, gateway, DNS option)
2. **Activate** the scope
3. **Authorize** the DHCP server in Active Directory (a domain-joined DHCP server won't respond to clients until authorized — shows as a red vs. green icon in the DHCP console)
4. Test with one client: `ipconfig /release` → `ipconfig /renew` → confirm it gets an address from the new pool, confirm gateway/DNS/internet all still work
5. Only *then* disable VMware's local DHCP service on VMnet3

Also had to remember to **exclude** the addresses already used statically (gateway `30.30.30.2`, DC1 `30.30.30.10`) from the new DHCP scope's range — otherwise DHCP could eventually hand one of those addresses out to a VM and create an IP conflict.

## Key takeaways

- `ping <IP>` succeeding while browsing fails is a strong signal the problem is DNS, not routing — worth remembering as a general diagnostic pattern, not just an AD-specific one.
- When swapping out infrastructure services (like DHCP), sequence the change so the new service is confirmed working *before* retiring the old one — never a same-step swap.
- Don't assume the last change made was the fix just because the symptom went away — verify the actual mechanism, since two things changing close together can be misleading (this cost some hours by way of the DHCP-range red herring).
