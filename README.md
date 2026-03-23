# Ethical Hacking Lab Portfolio

A structured collection of hands-on penetration testing labs performed in an isolated virtual environment. Each report documents objectives, methodology, findings, and defensive remediation — written to demonstrate both offensive technique and security-minded thinking.

> **Disclaimer:** All labs are performed on intentionally vulnerable machines (Metasploitable, DVWA, VulnHub, etc.) in an air-gapped virtual environment. No real systems, networks, or individuals were targeted at any point.

---

## Lab Index

| # | Lab | Key Tools | Techniques | Difficulty |
|---|---|---|---|---|
| 01 | [Virtual Lab Environment Setup](labs/01-virtual-lab-setup/report.md) | VirtualBox, pfSense | Network segmentation, VM configuration | 🟢 Beginner |
| 02 | [vsftpd 2.3.4 Backdoor Exploitation](labs/02-vsftpd-backdoor/report.md) | netdiscover, netcat | Host discovery, backdoor exploitation, remote shell | 🟢 Beginner |

---

## Lab Environment

All labs run locally in VirtualBox using the following VM topology:

```
[ Internet ]
     |
[ pfSense Firewall ] ← air gap between host and lab
     |
[ Internal LAN ]
     |----[ Kali Linux ]       ← Attacker
     |----[ Metasploitable 2 ] ← Vulnerable Linux server target
     |----[ Ubuntu Desktop ]   ← Vulnerable desktop target (future labs)
```

**Host requirements:** Windows, Linux, or macOS with VirtualBox, 30GB+ free disk, 8GB+ RAM recommended.

---

## Report Structure

Each lab report follows a consistent format:

- **Objective** — what the lab set out to demonstrate
- **Environment** — machines, IPs, network config
- **Methodology** — step-by-step walkthrough with commands
- **Results** — findings and outcomes
- **Key Takeaways** — what was learned
- **Remediation** — how a defender would address the vulnerability
- **References** — CVEs, documentation, tools

---

## Resources & Learning Path

This portfolio follows the curriculum in **"The Art of Hacking"** and supplements it with additional labs from:

- [TryHackMe](https://tryhackme.com) — guided, beginner-friendly rooms
- [Hack The Box](https://www.hackthebox.com) — realistic machines (employer-recognized)
- [VulnHub](https://www.vulnhub.com) — offline downloadable vulnerable VMs

---

*Work in progress — new labs added as completed.*
