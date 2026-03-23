# Lab 01 — Virtual Lab Environment Setup

> **All activity performed on intentionally vulnerable machines in an isolated virtual lab environment. No real systems were targeted.**

| Field          | Details                                                                                |
| -------------- | -------------------------------------------------------------------------------------- |
| **Date**       | 2026-03-22                                                                             |
| **Source**     | *The Art of Hacking* — Chapter 1                                                       |
| **Platform**   | VirtualBox (local hypervisor)                                                          |
| **Difficulty** | Beginner                                                                               |
| **Tags**       | `virtualbox` `pfsense` `kali-linux` `metasploitable` `network-setup` `lab-environment` |

---

## Objective

Build a fully isolated, multi-machine virtual hacking lab that can safely host and attack intentionally vulnerable targets — without any exposure to real networks or systems. The lab should mirror a realistic internal network topology including a firewall, attacker machine, and victim machines.

---

## Environment Overview

| VM | OS | Role | Network |
|---|---|---|---|
| pfSense | FreeBSD 64-bit | Router / Firewall | Adapter 1: Bridged (WAN) · Adapter 2: Internal LAN |
| Kali Linux | Kali (Debian-based) | Attacker | Internal LAN |
| Metasploitable 2 | Ubuntu 64-bit | Vulnerable Target (Linux server) | Internal LAN |
| Ubuntu Desktop | Ubuntu 64-bit | Vulnerable Target (desktop) | Internal LAN |

The pfSense VM acts as the perimeter between the host machine's internet connection and the internal lab network. All other VMs communicate exclusively over the `Internal LAN` virtual network — they cannot reach the internet and cannot be reached from outside the lab.

### Network Diagram

```
[ Internet ]
     |
[ pfSense VM ]  ← Bridged adapter (WAN)
     |
[ Internal LAN (VirtualBox) ]
     |----[ Kali Linux ]       ← Attacker
     |----[ Metasploitable 2 ] ← Target: Linux server
     |----[ Ubuntu Desktop ]   ← Target: Desktop environment
```

---

## Setup Methodology

### 1. VirtualBox Installation

Downloaded and installed VirtualBox from `https://www.virtualbox.org`. Accepted default installation options. VirtualBox serves as the hypervisor for all virtual machines.

### 2. pfSense VM Configuration

- Downloaded pfSense 2.5.0 ISO (AMD64, DVD Image installer) from `https://www.pfsense.org/download/`
- Created a new VM in VirtualBox:
  - **Name:** pfSense | **Type:** BSD | **Version:** FreeBSD (64-bit)
  - **RAM:** 1024 MB | **Disk:** 5 GB VDI (dynamically allocated)
- Network adapters configured:
  - **Adapter 1:** Bridged Adapter → host wireless/ethernet NIC (WAN — internet access)
  - **Adapter 2:** Internal Network → named `Internal LAN` (LAN — connects to other VMs)
- Booted from ISO, accepted defaults, completed install, rebooted
- After reboot: powered off VM, removed ISO from Storage settings to prevent boot loop
- Booted again — pfSense console confirmed:
  - WAN (em0) → DHCP IP from host router
  - LAN (em1) → `192.168.100.1/24`

### 3. Metasploitable 2 VM Configuration

- Downloaded Metasploitable 2 from SourceForge: `https://sourceforge.net/projects/metasploitable/`
- Extracted the `.vmdk` disk image from the ZIP archive
- Created a new VM in VirtualBox:
  - **Name:** Metasploitable | **Type:** Linux | **Version:** Ubuntu (64-bit)
  - Used existing virtual disk: selected the `.vmdk` file (no new disk created)
  - **RAM:** default suggested amount
- Network: Adapter 1 → Internal Network → `Internal LAN`
- Booted VM — confirmed Metasploitable 2 splash screen appeared in terminal
- Login credentials: `msfadmin / msfadmin`

### 4. Kali Linux VM Configuration

- Downloaded Kali Linux VirtualBox image (OVA) from `https://www.offensive-security.com/kali-linux-vm-vmware-virtualbox-image-download/`
- Imported OVA into VirtualBox (File → Import Appliance)
- Network: Adapter 1 → Internal Network → `Internal LAN`
- Booted VM — confirmed Kali login screen
- Login credentials: `kali / kali`

### 5. Ubuntu Desktop VM Configuration

- Downloaded Ubuntu Desktop ISO from `https://ubuntu.com/download/desktop`
- Created a new VM:
  - **Name:** Ubuntu | **Type:** Linux | **Version:** Ubuntu (64-bit)
  - **RAM:** 2048 MB | **Disk:** 10 GB
- Network: Adapter 1 → Internal Network → `Internal LAN`
- Booted from ISO, selected language, clicked **Install Ubuntu**, completed setup
- Shut down after installation — not needed until a later chapter

---

## Results

All four virtual machines successfully provisioned and networked:

- pfSense firewall running and routing traffic between the WAN and Internal LAN
- Metasploitable 2 accessible from Kali over the internal network
- Kali Linux operational with full tool suite available
- Ubuntu Desktop installed and ready for future labs
- No VMs have direct internet access except through pfSense (which can be firewall-restricted)

---

## Key Takeaways

- **Network isolation is critical** — placing vulnerable VMs on an internal-only network prevents accidental exposure. pfSense acts as the gatekeeper so even if Kali or Metasploitable misbehave, they cannot reach real systems.
- **Bridged vs. Internal adapters** — pfSense needs a bridged adapter on its WAN side to reach the internet (for updates, etc.), but all targets should be Internal-only. Mixing these up would expose vulnerable machines to the real network.
- **Removing the ISO after install** — forgetting to remove the boot ISO causes pfSense to loop back to the installer. Good habit: detach the ISO from Storage settings immediately after the OS install completes.
- **Metasploitable 2 vs. 3** — version 2 was used here because it is simpler to set up (pre-built VMDK, no provisioning required). Version 3 is more realistic but requires Vagrant and more configuration overhead.

---

## Remediation / Defensive Notes

This lab intentionally creates a vulnerable environment. In a real scenario, every machine in this topology would need hardening:

- pfSense firewall rules should follow a default-deny policy, explicitly allowing only required traffic
- Metasploitable-style machines should never exist in production — the concept of an intentionally vulnerable server exists only for training
- Network segmentation (as implemented here with the Internal LAN) is a genuine best practice: isolating sensitive or legacy systems from the broader network limits blast radius if they are compromised

---

## References

- VirtualBox documentation: https://www.virtualbox.org/wiki/Documentation
- pfSense download: https://www.pfsense.org/download/
- Metasploitable 2: https://sourceforge.net/projects/metasploitable/
- Kali Linux VirtualBox images: https://www.kali.org/get-kali/#kali-virtual-machines
