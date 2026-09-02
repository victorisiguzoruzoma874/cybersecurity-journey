# Nmap Recon Lab — Week 7 / Session 13

**Date:** 2026-09-02
**Source:** Cybersecurity Course — Week 7, Session 13 (Intro to Pentesting & Ethical Hacking)
**Category:** Reconnaissance & Enumeration
**Tools used:** Nmap
**Difficulty (my take):** Easy

> Authorization first: performed only in my authorized VirtualBox lab, against my own target VM.

---

## 1. Objective
Discover a target host on my lab subnet, then enumerate its open ports, services, versions and OS — and document the findings.

## 2. Environment
- Attacker: Kali Linux VM
- Target: VM at 192.168.56.x _(fill in your actual IP)_
- Scope: authorized lab only — my own VMs

## 3. Steps & Commands

### Step 1 — Host discovery (ping sweep)
```bash
sudo nmap -sn 192.168.56.0/24
```
**Result / what I observed:**
_(fill in the target IP you found)_

### Step 2 — Default scan
```bash
nmap <Target-IP>
```
**Result / what I observed:**
_(default scans the 1000 most common ports — list 3 open ports + their services)_

### Step 3 — Service version detection
```bash
nmap -sV <Target-IP>
```
**Result / what I observed:**
_(software versions on port 22 and port 80)_

### Step 4 — OS detection
```bash
sudo nmap -O <Target-IP>
```
**Result / what I observed:**
_(the OS Nmap guessed, and its accuracy)_

### Step 5 — Aggressive scan
```bash
sudo nmap -A <Target-IP>
```
**Result / what I observed:**
_(combined OS + version + scripts + traceroute output)_

### Step 6 — Save output to a file
```bash
nmap -oN scan_results.txt <Target-IP>
```

## 4. Findings

| Item | Value |
|------|-------|
| Target IP | |
| Open ports | |
| Services / versions | |
| Operating system | |

## 5. What I Learned
- 
- 
- 

## 6. Theory — SYN scan vs Connect scan
A SYN scan (`-sS`, the default when run with sudo) sends a SYN, receives SYN-ACK, then sends RST to abort before the 3-way handshake completes. Because no full connection is established, the target often does not log it — that is why it is called a stealth scan. A Connect scan (`-sT`, the default without root) completes the full handshake, so the target records a real connection and it is easier to detect.

## 7. References
- Course: Week 7, Session 13 resources
- Nmap reference guide: https://nmap.org/book/man.html
