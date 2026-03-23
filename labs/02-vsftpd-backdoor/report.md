# Lab 02 — Exploiting the vsftpd 2.3.4 Backdoor

> **All activity performed on an intentionally vulnerable Metasploitable 2 VM in an isolated virtual lab environment. No real systems were targeted.**

| Field | Details |
|---|---|
| **Date** | 2026-03-22 |
| **Source** | *The Art of Hacking* — Chapter 1 |
| **Target** | Metasploitable 2 (VirtualBox VM, Internal LAN) |
| **Attacker** | Kali Linux (VirtualBox VM, Internal LAN) |
| **Difficulty** | Beginner |
| **CVE** | CVE-2011-2523 |
| **Tags** | `ftp` `backdoor` `netcat` `netdiscover` `vsftpd` `metasploitable` `CVE-2011-2523` `remote-code-execution` |

---

## Objective

Exploit a known backdoor vulnerability in vsftpd 2.3.4 to gain unauthorized root-level shell access to the Metasploitable 2 target machine. This lab demonstrates the full attack lifecycle at a basic level: host discovery → service identification → exploitation → shell access → remediation analysis.

---

## Environment

| Machine | IP Address | Role |
|---|---|---|
| Kali Linux | 192.168.100.x (DHCP) | Attacker |
| pfSense | 192.168.100.1 | Router / Gateway |
| Metasploitable 2 | 192.168.100.101 | Target |

> Note: IP addresses will vary. The pfSense router always occupies `.1`; Metasploitable will be the next address assigned by DHCP.

---

## Background: The vsftpd 2.3.4 Backdoor

In July 2011, it was discovered that the source code of **vsftpd (Very Secure FTP Daemon) version 2.3.4** had been compromised by an attacker who inserted a backdoor directly into the official distribution package. This is a supply chain attack — the legitimate software project was infiltrated and a malicious modification was distributed to anyone who downloaded that version.

**How the backdoor works:**

When an FTP client connects and sends a username that ends with the string `:)` (a smiley face), the backdoor triggers. It opens a **bind shell on port 6200** — a shell process that listens for incoming TCP connections. Anyone who then connects to port 6200 on that machine is given a **root-level command prompt** with no password required.

This is notable because:
- It requires no authentication bypass — the backdoor itself is the access mechanism
- It grants root access immediately
- It was present in the official source tarball on the project's website

Metasploitable 2 ships with this vulnerable version of vsftpd intentionally installed for training purposes.

---

## Methodology

### Step 1 — Host Discovery with `netdiscover`

Before attacking a target, we need to identify its IP address on the network. The `netdiscover` tool performs ARP-based host discovery, scanning the local subnet for active machines.

On the Kali Linux terminal:

```bash
sudo netdiscover
```

`netdiscover` scanned the Internal LAN subnet and returned the following active hosts:

```
IP              At MAC Address     Count  Len  MAC Vendor / Hostname
192.168.100.1   08:00:27:3b:8f:ed  1      60   PCS Systemtechnik GmbH
192.168.100.101 08:00:27:fe:31:e6  1      60   PCS Systemtechnik GmbH
```

- `192.168.100.1` → pfSense router (gateway)
- `192.168.100.101` → Metasploitable 2 target

The MAC vendor "PCS Systemtechnik GmbH" is VirtualBox's default vendor string — both addresses belong to virtual machines as expected.

### Step 2 — Verify Target Reachability (Web Interface)

Opened the Kali Linux web browser and navigated to `http://192.168.100.101`. The Metasploitable 2 web interface loaded, confirming:
- The target is live and reachable
- Web services are running (TWiki, phpMyAdmin, Mutillidae, DVWA, WebDAV all listed)
- Both VMs are correctly connected to the Internal LAN

### Step 3 — Trigger the Backdoor via FTP

Connected to the Metasploitable machine's FTP server on port 21 using Netcat (`nc`), a general-purpose TCP/UDP networking tool:

```bash
nc 192.168.100.101 21
```

Once connected, the FTP service banner appeared. Sent the backdoor trigger credentials:

```
user Hacker:)
pass invalid
```

The username `Hacker:)` ends with `:)`, which triggers the backdoor. The password is irrelevant — the FTP server does not need to authenticate the user for the backdoor to activate. After sending these lines, the terminal appeared to hang with no response. This is expected — the backdoor has opened a shell on **port 6200** in the background.

### Step 4 — Connect to the Backdoor Shell

Opened a **new terminal window** on Kali and connected to port 6200 on the target:

```bash
nc -v 192.168.100.101 6200
```

The terminal connected successfully and appeared unresponsive — it was waiting for input. Typed:

```bash
ls
```

A directory listing of the Metasploitable machine's filesystem returned in the Kali terminal, confirming remote command execution.

### Step 5 — Confirm Privilege Level

```bash
whoami
```

**Output:** `root`

Full root access confirmed. To further verify control:

```bash
reboot
```

The Metasploitable virtual machine rebooted — demonstrating that commands entered on the Kali attacker machine executed with full administrative privileges on the target.

---

## Results

| Goal | Outcome |
|---|---|
| Identify target IP | ✅ `192.168.100.101` via `netdiscover` |
| Trigger vsftpd backdoor | ✅ Username ending in `:)` on port 21 |
| Obtain shell on port 6200 | ✅ Connected via `nc -v 192.168.100.101 6200` |
| Confirm root access | ✅ `whoami` returned `root` |
| Execute arbitrary commands | ✅ `reboot` successfully rebooted target |

---

## Key Takeaways

- **Supply chain attacks are devastating** — this vulnerability wasn't a bug in the code, it was a malicious modification to a trusted software package. Defenders must verify the integrity of software they download (checksums, signatures) and not assume that "the official source" is always safe.
- **Netcat is a fundamental tool** — `nc` requires no exploit framework; it's a raw TCP connection tool. Understanding how to use it for both connecting to services and catching shells is an essential baseline skill.
- **Backdoors bypass authentication entirely** — there was no credential to brute force, no service to scan for weaknesses. The attacker's only job was knowing the trigger existed. This underscores why keeping software updated and from verified sources matters.
- **Port 6200 is the giveaway** — in a real incident, a defender monitoring for unexpected listening ports (`netstat -tuln`, endpoint detection) would see port 6200 appear suddenly after an FTP connection. This kind of anomaly detection is how defenders catch active intrusions.
- **Root shells are game over** — once an attacker has a root shell, they can do anything: read all files, install persistent backdoors, pivot to other machines, delete data. Containment and minimizing privilege are key defensive principles.

---

## Remediation

**How would a defender fix this?**

1. **Update vsftpd** — Newer versions of vsftpd have removed this backdoor. The fix is straightforward: upgrade to any version released after the compromise was discovered (post-July 2011). On a Debian/Ubuntu system: `sudo apt-get install --only-upgrade vsftpd`

2. **Verify software integrity** — Before deploying any software, verify its cryptographic hash (SHA256 or similar) against the hash published by the official project. A compromised package will have a different hash. Use package managers that perform this verification automatically.

3. **Network segmentation** — As demonstrated in Lab 01, the Metasploitable machine was placed on an isolated internal network. In a real environment, FTP servers should not be directly accessible from untrusted networks without firewall rules limiting source IPs.

4. **Disable unnecessary services** — If FTP is not needed, it should not be running. The principle of least functionality means reducing attack surface by disabling services that aren't required.

5. **Monitor for unexpected listening ports** — A port suddenly appearing (like 6200) after an FTP connection is an Indicator of Compromise (IoC). SIEM rules or endpoint detection tools should alert on new listening ports on servers.

---

## References

- CVE-2011-2523: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2011-2523
- vsftpd project: https://security.appspot.com/vsftpd.html
- Metasploitable 2 documentation: https://docs.rapid7.com/metasploit/metasploitable-2/
- Netcat man page: `man nc`
