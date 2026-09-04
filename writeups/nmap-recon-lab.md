# ð¡ï¸ Week 7 â Network Scanning & Enumeration with Nmap

> A walkthrough of reconnaissance against the host-only VirtualBox lab. We find the target, map its ports, fingerprint its services and OS, and finish with the "everything at once" scan â with the answers to Q1âQ6 called out along the way.

[TOC]

---

## ð¯ First, who's actually on the network?

Before touching a single port, we need to know **what** to scan. A host-discovery sweep across the subnet turned up four addresses â but only one of them is our real target. Here's how to read the list:

| IP | What it is | Verdict |
|---|---|---|
| `192.168.56.20` | That's **you** â your Kali box (the `.20` we just assigned) | ð« Ignore |
| `192.168.56.1` | The VirtualBox **gateway** (MAC `0A:00:27:00:00:00` â the host-only network's router, not a VM) | ð« Infrastructure |
| `192.168.56.100` | Almost certainly VirtualBox's built-in **DHCP server** (it uses `.100` by default) | ð« Infrastructure |
| `192.168.56.102` | An **actual VM** (MAC `08:00:27:1D:C7:F6` = "Oracle VirtualBox virtual NIC") â not you, not the gateway | â **Target** |

---

## Q1 â Host discovery

```bash
sudo nmap -sn 192.168.56.0/24
```

:::success
**Q1 command:** `sudo nmap -sn 192.168.56.0/24`
**Q1 â target IP:** `192.168.56.102`
:::

![image](https://hackmd.io/_uploads/HJ2Z2uwufg.png)

---

## Q2 â How many ports, and which are open?

**"How many ports did Nmap scan by default?"** â **1,000.** Here's the neat way to prove it straight from your own output: it says `Not shown: 977 closed tcp ports`, and it lists `23 open ports`. **977 + 23 = 1,000.** A bare `nmap` always scans the 1,000 *most common* ports (not all 65,535) â and your result literally adds up to it.

**"List three open ports and their services"** â easy, pick any three:

| Port | Service |
|---|---|
| `21/tcp` | ftp |
| `22/tcp` | ssh |
| `80/tcp` | http |

:::success
**Ports scanned by default:** 1,000 (977 closed + 23 open)
**Three open ports:** 21/ftp, 22/ssh, 80/http
:::

---

## Q3 â Scanning specific ports

Now let's narrow the aim to just a few ports of interest:

```bash
nmap -p 22,80,443 192.168.56.102
```

![image](https://hackmd.io/_uploads/rk_80_DuGe.png)
![image](https://hackmd.io/_uploads/ByqPA_DOMg.png)

---

## Q4 â Version detection (the real prize)

There it is â the **version numbers**, which are the real prize of this step:

| Port | Service | Version |
|---|---|---|
| `22` (SSH) | OpenSSH | `4.7p1 Debian 8ubuntu1` (protocol 2.0) |
| `80` (HTTP) | Apache httpd | `2.2.8` ((Ubuntu) DAV/2) |

```bash
nmap -sV 192.168.56.102
```

:::success
**Command:** `nmap -sV 192.168.56.102`
**SSH (22):** OpenSSH 4.7p1 Debian 8ubuntu1
**HTTP (80):** Apache httpd 2.2.8 (Ubuntu, DAV/2)
:::

> ðµï¸ **Attacker's-eye view â why this matters:** both of those are *ancient* (OpenSSH 4.7 and Apache 2.2.8 are from around 2007â2008). Once you have an exact version, you search it in a vulnerability database and find known exploits. You can even spot notorious ones right in your list â **vsftpd 2.3.4** (port 21) and **UnrealIRCd** (port 6667) both shipped with famous backdoors. That's the payoff of version detection: it turns *"a port is open"* into *"here's exactly how I get in."* (We're not exploiting today â just noting why enumeration is the foundation of every attack.)

![image](https://hackmd.io/_uploads/B1m7gKD_Ml.png)

---

## Q5 â OS detection

That worked â capital `-O` did the trick. Here's your OS detection result:

- **Running:** Linux 2.6.X
- **OS details:** Linux 2.6.9 â 2.6.33

```bash
sudo nmap -O 192.168.56.102
```

:::success
**Command:** `sudo nmap -O 192.168.56.102`
**OS guess:** Linux, kernel 2.6.X (specifically the 2.6.9 â 2.6.33 range)
:::

> ðµï¸ **Attacker's read:** kernel 2.6 is really old (that era is late-2000s), so it's exposed to a pile of well-documented privilege-escalation exploits. The picture's now complete â you know it's an old Linux box running old services. That's a full fingerprint.

![image](https://hackmd.io/_uploads/HyFwXtwOGe.png)

---

## Q6 â The aggressive scan (everything at once)

This is the "everything at once" shortcut â it runs OS detection **+** version detection **+** default scripts **+** traceroute in a single command:

```bash
sudo nmap -A 192.168.56.102
```

:::warning
**Heads-up:** this one is slow and very chatty â it can take 1â2 minutes and produces a lot of output (script results under each port, etc.). That's expected. You don't need to paste the whole wall of text â the top portion through the port list, or a screenshot of the end, is enough to confirm it ran.
:::

Look at everything `-A` pulled that the earlier scans didn't:

- **Version detection** (like Q4) â every service named
- **OS detection** (like Q5) â Linux 2.6.9 â 2.6.33 again
- **Default scripts (NSE)** â the extra intel: `ftp-anon: Anonymous FTP login allowed` (port 21 lets *anyone* in!), SSH host keys, `smb-os-discovery` naming the box **metasploitable**, MySQL details, and more
- **Traceroute** â at the very bottom: 1 hop â `192.168.56.102`

:::success
**Command:** `sudo nmap -A 192.168.56.102`
**What it does:** OS detection + version detection + default scripts + traceroute, all in one â the richest single scan.
:::

> ð¡ Notice `ftp-anon: Anonymous FTP login allowed` â that's `-A` doing recon and finding a real weakness in one shot. That's exactly how offensive folks work fast.

![image](https://hackmd.io/_uploads/Hy3EIKPOMg.png)
![image](https://hackmd.io/_uploads/H1qSIKDuGe.png)
![image](https://hackmd.io/_uploads/rkcILKDOGg.png)
![image](https://hackmd.io/_uploads/SyKw8YD_Me.png)

---

## ð Answer key at a glance

| Q | Command | Answer |
|---|---|---|
| **Q1** | `sudo nmap -sn 192.168.56.0/24` | Target: `192.168.56.102` |
| **Q2** | *(default scan)* | 1,000 ports (977 closed + 23 open); open: 21/ftp, 22/ssh, 80/http |
| **Q3** | `nmap -p 22,80,443 192.168.56.102` | Scans only ports 22, 80, 443 |
| **Q4** | `nmap -sV 192.168.56.102` | SSH: OpenSSH 4.7p1 Â· HTTP: Apache 2.2.8 |
| **Q5** | `sudo nmap -O 192.168.56.102` | Linux, kernel 2.6.9 â 2.6.33 |
| **Q6** | `sudo nmap -A 192.168.56.102` | OS + version + scripts + traceroute in one |

---

> **Takeaway:** enumeration turns *"a port is open"* into *"here's an old service with a known way in."* Every step here â discovery â ports â versions â OS â the all-in-one `-A` â is just building a sharper picture of the target before anything gets touched.
