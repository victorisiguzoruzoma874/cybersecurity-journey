# Nmap Recon Lab — Week 7 / Session 13

**Date:** 2026-09-03
**Source:** Cybersecurity Course — Week 7, Session 13 (Intro to Pentesting & Ethical Hacking)
**Category:** Reconnaissance & Enumeration
**Tools used:** Nmap 7.99, Kali Linux, VirtualBox
**Target:** Metasploitable 2 (192.168.56.102)

> Authorization: all scanning was performed only in my own authorized VirtualBox lab, against my own target VM, on an isolated host-only network.

---

## Objective
Discover a target host on the lab subnet, then enumerate its open ports, service versions, and operating system with Nmap — and document the findings.

## Environment
- Attacker: Kali Linux — 192.168.56.20 (host-only adapter eth1)
- Target: Metasploitable 2 — 192.168.56.102
- Network: VirtualBox Host-only, 192.168.56.0/24 (isolated lab)

---

## Part 1 — Host Discovery & Basic Scanning

### Q1. Host discovery (ping scan)
**Command:**
```bash
sudo nmap -sn 192.168.56.0/24
```
**Result:** 4 live hosts — 192.168.56.1 (VirtualBox gateway), 192.168.56.100 (VirtualBox DHCP server), 192.168.56.20 (my Kali box), and 192.168.56.102 (target).

**Answer — Target IP:** `192.168.56.102`
Identified by elimination: not my own Kali, not the gateway/DHCP infrastructure. Its MAC showed 'Oracle VirtualBox virtual NIC', confirming it is a VM.

### Q2. Default scan
**Command:**
```bash
nmap 192.168.56.102
```
**Answer — Ports scanned by default:** 1000. A bare `nmap` scans the 1000 most common TCP ports. The output confirms it: 977 closed + 23 open = 1000.

**Answer — Three open ports:** 21/tcp (ftp), 22/tcp (ssh), 80/tcp (http).

### Q3. Scanning specific ports
**Command:**
```bash
nmap -p 22,80,443 192.168.56.102
```
**Result:**
- 22/tcp — open — ssh
- 80/tcp — open — http
- 443/tcp — closed — https

443 is closed because the target serves plain HTTP, not HTTPS. 'closed' is a normal, informative result.

---

## Part 2 — Advanced Enumeration

### Q4. Service version detection
**Command:**
```bash
nmap -sV 192.168.56.102
```
**Answer:**
- Port 22 (SSH): OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
- Port 80 (HTTP): Apache httpd 2.2.8 ((Ubuntu) DAV/2)

Both are outdated versions — and that matters, because a known version number can be matched against public vulnerability databases to find working exploits.

### Q5. Operating system detection
**Command:**
```bash
sudo nmap -O 192.168.56.102
```
**Answer — OS:** Linux, kernel 2.6.X (Nmap details: Linux 2.6.9 – 2.6.33).

Note: `-O` is a capital letter O and requires sudo/root, because OS detection sends raw packets.

### Q6. Aggressive scan
**Command:**
```bash
sudo nmap -A 192.168.56.102
```
**What it does:** enables OS detection, version detection, default NSE scripts, and traceroute in one command. Notable script findings included `ftp-anon: Anonymous FTP login allowed` on port 21, and `smb-os-discovery` identifying the host as 'metasploitable'.

---

## Part 3 — Output & Theory

### Q7. Saving output to a file
**Command:**
```bash
nmap -oN scan_results.txt 192.168.56.102
```
`-oN` saves the results in Normal (human-readable) text format to the given filename. Verified with `ls` (file present) and `cat scan_results.txt` (contents readable).

### Q8. SYN scan vs Connect scan (theory)
By default, running Nmap with sudo (root) performs a **TCP SYN scan (`-sS`)**; as a normal user it performs a **TCP Connect scan (`-sT`)**.

- **SYN scan (`-sS`):** sends a SYN, receives SYN-ACK if the port is open, then immediately sends RST to abort *before* the 3-way handshake completes. Because no full connection is established, the target application usually does not log it. Requires root to craft the raw packets.
- **Connect scan (`-sT`):** completes the full TCP 3-way handshake (SYN → SYN-ACK → ACK) using the operating system normal connect call. The target records a real connection, so it is noisier and easier to detect. Does not require root.

**Why SYN is called a 'stealth' scan:** it never completes the handshake — it tears the half-open connection down with an RST as soon as it learns the port is open — so no session is established and the connection typically goes unlogged by the target service.

---

## What I Learned
- How to find an unknown target on a subnet with a ping sweep, and how to tell the target apart from my own host and from network infrastructure (by elimination and MAC vendor).
- The difference between scanning all common ports and specific ports, and that 'closed' is a normal, useful result.
- How version and OS detection turn 'a port is open' into actionable intelligence — old software maps to known vulnerabilities.
- What the aggressive scan bundles together, and how to save results for documentation.
- The difference between a SYN scan and a Connect scan, and why stealth matters.

## Challenges & How I Solved Them
- My Kali box was initially only on the NAT network (10.0.2.15) with no IP on the host-only lab network, so early scans were inconsistent. I gave Kali a lab address with `sudo ip addr add 192.168.56.20/24 dev eth1` and brought the interface up, which put it on 192.168.56.0/24.
- `sudo dhclient` was not available on this Kali build, so I assigned the address manually instead.
- I first ran OS detection with a lowercase `-o` (an output flag) instead of capital `-O`, which produced 'No targets were specified'. Switching to `-O` fixed it — a reminder that Nmap flags are case-sensitive.

## References
- Course: Week 7, Session 13 resources (Intro to Pentesting & Ethical Hacking)
- Nmap reference guide: https://nmap.org/book/man.html
