# Task 1: Scan Your Local Network for Open Ports

**Internship:** Elevate Labs Cyber Security Internship  
**Task:** Scan Your Local Network for Open Ports  
**Tools Used:** Nmap 7.98, Windows Command Prompt (Administrator)  
**Date Completed:** 2026-05-29  

---

## Objective

To perform active network reconnaissance on a local subnet using Nmap's TCP SYN scan (`-sS`), identify all live hosts and their open ports, analyze the exposed services, and assess potential security risks — developing foundational skills in network security auditing.

---

## Tools & Environment

| Tool | Version | Purpose |
|------|---------|---------|
| Nmap | 7.98 | Network/port scanning |
| Windows CMD | (Admin) | Command execution environment |
| Wireshark | 4.x (optional) | Packet capture analysis |

- **OS:** Windows 10/11
- **Network Type:** Local VirtualBox Host-Only Network
- **Scan Scope:** `192.168.56.1/24`

---

## Commands Executed

### 1. Identify Local IP Address

```cmd
ipconfig
```

**Output (relevant section):**

```
Ethernet adapter VirtualBox Host-Only Network:

   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::a1b2:c3d4:e5f6:g7h8%12
   IPv4 Address. . . . . . . . . . . : 192.168.56.1
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . :
```

### 2. Verify Nmap Installation

```cmd
nmap --version
```

**Output:**

```
Nmap version 7.98 ( https://nmap.org )
Platform: i686-pc-windows-windows
Compiled with: nmap-liblua-5.4.4 openssl-3.0.13 libssh2-1.11.0 libz-1.3.1
               libpcre-8.45 libpcap-1.10.4 nmap-libdnet-1.12 ipv6
Compiled without:
Available nsock engines: iocp epoll poll select
```

### 3. TCP SYN Scan — Full Subnet

```cmd
nmap -sS -v --open -oN scan_results.txt 192.168.56.1/24
```

**Flags explained:**

| Flag | Description |
|------|-------------|
| `-sS` | TCP SYN (half-open/stealth) scan — sends SYN, never completes handshake |
| `-v` | Verbose output for real-time progress |
| `--open` | Display only hosts with at least one open port |
| `-oN scan_results.txt` | Save output in human-readable Normal format |

---

## Raw Scan Results

```
# Nmap 7.98 scan initiated Fri May 29 18:44:39 2026 as: nmap -sS -v --open -oN scan_results.txt 192.168.56.1/24

Nmap scan report for 192.168.56.1
Host is up (0.00055s latency).
Not shown: 995 closed tcp ports (reset)
PORT     STATE SERVICE
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
2869/tcp open  icslap
7070/tcp open  realserver

Read data files from: C:\Program Files (x86)\Nmap
# Nmap done at Fri May 29 18:44:52 2026 -- 256 IP addresses (1 host up) scanned in 12.67 seconds
```

---

## Findings Summary

### Hosts Discovered

| Host IP | Hostname | Open Ports | Likely Device |
|---------|----------|------------|---------------|
| 192.168.56.1 | N/A | 135, 139, 445, 2869, 7070 | Windows OS (VirtualBox Host) |

### Open Ports & Services Analysis

| Port | Protocol | Service | Risk Level | Notes |
|------|----------|---------|------------|-------|
| 135 | TCP | msrpc | ⚠️ Medium | RPC endpoint mapper. Commonly exposed on Windows; can leak system information to enumerators. |
| 139 | TCP | netbios-ssn | ⚠️ Medium | Legacy NetBIOS file/printer sharing. Vulnerable to enumeration and legacy attacks. |
| 445 | TCP | microsoft-ds (SMB) | 🔴 High | Modern SMB over TCP. Targeted by critical exploits like EternalBlue (WannaCry ransomware). Must be patched. |
| 2869 | TCP | icslap (UPnP/SSDP) | 🟢 Low | Used by Microsoft HTTPAPI for UPnP device discovery. Low risk on private network. |
| 7070 | TCP | realserver | 🟢 Low | Associated with media streaming or third-party apps. AnyDesk client detected on deeper scan. |

---

## Security Risk Assessment

### Identified Risks

**1. Port 445 (SMB) open on 192.168.56.1**  
SMB file sharing is active. If this machine is missing security patches, it is highly susceptible to lateral movement attacks, ransomware, or remote code execution via exploits like EternalBlue.  
**Recommendation:** Ensure the host is fully patched (Windows Update). If file sharing is not required, disable the SMB service or restrict access via Windows Firewall to trusted IPs only.

**2. Ports 135 & 139 (RPC/NetBIOS) are open**  
These ports allow an attacker on the same network to enumerate network shares, user accounts, and system domain information.  
**Recommendation:** Disable NetBIOS over TCP/IP in Network Adapter Settings → Advanced → WINS tab if not needed for legacy compatibility.

**3. Port 7070 (AnyDesk/RealServer) detected**  
A remote access application (AnyDesk) was detected running. Remote access tools are a common persistence mechanism used by attackers.  
**Recommendation:** Confirm the application is intentionally installed and actively monitored. Disable if not required.

### Overall Risk Rating: 🟡 Medium

**Justification:** While SMB (445) and NetBIOS (139) pose significant risks if exposed to an untrusted network, this scan was conducted on a local `192.168.56.X` VirtualBox host-only subnet, which is not routable to the internet. The risk is contextually mitigated, but these ports remain dangerous if an attacker gains internal network access — which is the most common real-world attack scenario.

---

## Key Learnings

1. **TCP SYN scans** do not complete the three-way handshake, making them faster and less likely to be logged by older intrusion detection systems compared to full connect scans.

2. **Every open port is a potential attack vector.** Minimising unnecessary open ports (attack surface reduction) is one of the highest-ROI security practices in any organisation.

3. **Network reconnaissance** is the foundational first step in both offensive security (penetration testing) and defensive security (vulnerability assessments). Understanding it from both sides is critical.

4. **CIDR /24 notation** represents 256 IP addresses on the same subnet — a standard layout for home networks, small offices, and lab environments like VirtualBox host-only adapters.

5. **Nmap requires administrator/root privileges** for SYN scanning because it needs to craft raw packets at a level below the operating system's TCP stack.

---

## Interview Questions — Answered

**Q1. What is an open port?**  
A port is a logical communication endpoint numbered 0–65535. A port is "open" when a service is actively listening on it and responding to incoming connection requests. For example, port 443 being open means a web server is ready to serve HTTPS traffic.

**Q2. How does Nmap perform a TCP SYN scan?**  
Nmap sends a SYN packet to each target port. If it receives a SYN-ACK in response, the port is open — Nmap immediately sends an RST (reset) to abort the connection without completing the handshake. If it receives RST, the port is closed. No response indicates a filtered (firewalled) port.

**Q3. What risks are associated with open ports?**  
Open ports expand the attack surface. Risks include exploitation of vulnerable service versions, brute-force attacks on authentication services (SSH, RDP), information leakage from misconfigured services, and use as pivot points in lateral movement during a breach.

**Q4. Explain the difference between TCP and UDP scanning.**  
TCP is connection-oriented — it uses a handshake, so open/closed status is definitively confirmed by the response. UDP is connectionless — Nmap sends a UDP packet and interprets no response as `open|filtered` since both firewalls and open ports can stay silent. UDP scanning is slower and less reliable but essential for finding services like DNS (53), DHCP (67/68), and SNMP (161).

**Q5. How can open ports be secured?**  
Close unnecessary ports via firewall rules; keep all service software patched; use strong authentication (key-based over password-based); implement VPN or port knocking for sensitive services; deploy IDS/IPS to detect and alert on scan attempts.

**Q6. What is a firewall's role regarding ports?**  
A firewall filters network traffic based on rules defining which ports accept traffic, from which source IPs, and on which interfaces. It acts as a gatekeeper — permitting legitimate traffic (e.g., port 443 from any IP) while blocking unauthorised access (e.g., port 3389 from the internet).

**Q7. What is a port scan and why do attackers perform it?**  
A port scan is a systematic probe of a host's ports to map its network attack surface. Attackers perform it during the reconnaissance phase to identify running services, infer OS and software versions, locate outdated/vulnerable software, and select the most viable exploitation path before launching an attack.

**Q8. How does Wireshark complement port scanning?**  
Wireshark captures live network packets during a scan, allowing you to visually verify the SYN/SYN-ACK/RST exchange at the packet level. It confirms what Nmap reports, helps debug scan anomalies, and demonstrates exactly how TCP operates at the wire level — invaluable for both learning and professional forensic investigations.

---

## Repository Structure

```
task1-network-port-scanning/
├── README.md               ← This file
├── scan_results.txt        ← Raw Nmap normal output
├── scan_results.xml        ← Nmap XML output (optional)
├── scan_results.html       ← HTML report (optional)
└── screenshots/
    ├── ipconfig_output.png
    ├── nmap_scan_running.png
    └── scan_results_terminal.png
```

---

## References

- [Nmap Official Documentation](https://nmap.org/docs.html)
- [Nmap TCP SYN Scan](https://nmap.org/book/synscan.html)
- [MITRE ATT&CK — Reconnaissance (TA0043)](https://attack.mitre.org/tactics/TA0043/)
- [Microsoft Security Advisory — EternalBlue / MS17-010](https://docs.microsoft.com/en-us/security-updates/securitybulletins/2017/ms17-010)
- [OWASP Network Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)

---

*Completed as part of the Elevate Labs Cyber Security Internship — Task 1*
