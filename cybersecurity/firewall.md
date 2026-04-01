# Firewall

A study note covering what firewalls are, how they work at the packet level, the difference between stateful and stateless inspection, types of firewalls, and their direct role in cybersecurity — including real-world breaches caused by misconfiguration.

---

## Overview

A firewall is a network security system that monitors and controls incoming and outgoing traffic based on a defined set of rules. It sits between a trusted internal network and an untrusted external one — like the internet — and decides what gets through and what gets blocked.

Think of it as a gatekeeper: every packet that arrives is checked against the rules. If it matches an allowed rule, it passes. If not, it gets dropped.

```
Internet → [ Firewall ] → Internal Network
              ↑
         Rules Engine:
         Allow / Block / Drop
```

> According to Gartner, **95% of all firewall breaches are caused by misconfiguration — not software flaws**. The firewall was there. It just wasn't set up correctly.

---

## Key Concepts

| Term | Description |
|------|-------------|
| **Packet** | A unit of data transmitted over a network |
| **Rule / Policy** | A condition that defines what traffic is allowed or blocked |
| **Inbound traffic** | Traffic coming from outside into the network |
| **Outbound traffic** | Traffic leaving the internal network |
| **Port** | A logical channel that identifies a type of service (ex: 80 = HTTP) |
| **Protocol** | The communication standard used (TCP, UDP, ICMP) |
| **Default deny** | Blocking all traffic unless explicitly allowed — the safest baseline |

---

## Stateless vs Stateful Firewalls

This is one of the most important distinctions in firewall architecture — and a frequent topic in cybersecurity exams and CTFs.

### Stateless Firewall

Evaluates each packet **independently**, with no memory of previous packets. It only checks:

- Source IP
- Destination IP
- Source port
- Destination port
- Protocol (TCP/UDP)

```
Packet arrives → Check rules → Allow or Block
(no context, no memory of past packets)
```

**Example rule:**
```
Allow TCP from ANY to 192.168.1.10 on port 80
Block ALL other traffic
```

**Weakness:** An attacker can craft packets that match the rules individually but are part of a malicious session. The firewall has no way to detect this because it does not track connections.

---

### Stateful Firewall

Tracks the **state of active connections** using a state table. It understands whether a packet belongs to an established session or is trying to initiate a new unauthorized one.

```
Client sends SYN → Firewall records connection state
Server replies SYN-ACK → Firewall verifies it matches a known session
Client sends ACK → Connection established and tracked
```

If a packet arrives that does not match any known session, it is dropped — even if it looks legitimate on the surface.

**Example:**
```
Outbound request allowed → 192.168.1.5 → google.com:443
Response from google.com → allowed automatically (matches session)
Unsolicited packet from unknown IP → blocked (no matching session)
```

**Stateless vs Stateful — Side by Side:**

| Feature | Stateless | Stateful |
|--------|-----------|----------|
| Tracks connections | No | Yes |
| Context-aware | No | Yes |
| Performance | Faster | Slightly slower |
| Security | Lower | Higher |
| Use case | Simple filtering, high-speed routing | Enterprise networks, perimeter defense |

---

## Types of Firewalls

### Packet Filtering Firewall
The simplest type. Stateless. Checks headers only — IP, port, protocol. No deep inspection.

### Stateful Inspection Firewall
Tracks the full state of connections. Standard in modern enterprise networks.

### Application Layer Firewall (Layer 7 / WAF)
Inspects the content of packets, not just headers. Can detect and block attacks like SQL injection, XSS, and malicious HTTP requests.

```
Normal request:  GET /index.html HTTP/1.1
Malicious:       GET /index.php?id=1' OR '1'='1  ← SQLi attempt
WAF blocks it before it reaches the server
```

### Next-Generation Firewall (NGFW)
Combines stateful inspection with deep packet inspection (DPI), intrusion prevention (IPS), application awareness, and threat intelligence. Used in modern enterprise and cloud environments.

### Software Firewall
Runs on the host machine itself (ex: Windows Firewall, UFW on Linux). Protects that specific device.

### Hardware Firewall
A dedicated physical device positioned at the network perimeter. Protects the entire network behind it.

---

## How a Firewall Processes Traffic (Step by Step)

### Step 1 — Packet arrives

```
Incoming packet:
  Source IP:        203.0.113.45
  Destination IP:   192.168.1.10
  Destination Port: 22 (SSH)
  Protocol:         TCP
```

### Step 2 — Rule matching

The firewall checks the packet against its ruleset from top to bottom. The first matching rule wins.

```
Rule 1: Allow TCP from 10.0.0.5 to 192.168.1.10 port 22   → no match
Rule 2: Block TCP from ANY to 192.168.1.10 port 22          → MATCH → DROP
```

### Step 3 — Action applied

```
Packet from 203.0.113.45 to port 22 → DROPPED
No response sent back to the source
```

### Step 4 — Logging

Most firewalls log blocked packets. This log is used in forensic investigation and incident response to trace attacker activity.

---

## Firewall in a Cybersecurity Context

Firewalls are the first line of defense — but also the first thing an attacker probes.

**From an attacker's perspective:**
- Scan for open ports to find exposed services
- Look for overly permissive rules
- Attempt firewall evasion (fragmentation, protocol abuse, tunneling)
- Target misconfigured rules to bypass filtering

**From a defender's perspective:**
- Define strict inbound and outbound rules
- Apply default deny — block everything not explicitly allowed
- Monitor firewall logs for unusual patterns
- Segment the network so a breach in one zone cannot spread freely

**Tools used in security testing:**
```bash
nmap -sS 192.168.1.1        # SYN scan to detect open ports
nmap -sA 192.168.1.1        # ACK scan to map firewall rules
nmap -f  192.168.1.1        # Fragmented packets to evade detection
```

---

## Real-World Breaches

### Capital One (2019)
A misconfigured **Web Application Firewall (WAF)** on AWS failed to block a Server-Side Request Forgery (SSRF) attack. The attacker used the exposed metadata endpoint to retrieve temporary cloud credentials, then accessed S3 buckets containing over **100 million customer records**. The firewall was in place — the rule was simply wrong.

### Mother of All Breaches — MOAB (2024)
A dataset of **26 billion records** compiled from thousands of past breaches was exposed publicly. The organization responsible, Leak-Lookup, confirmed the exposure was caused directly by a **firewall misconfiguration** that left the data accessible without authentication.

### Marquis Fintech (2026)
A financial technology provider suffered a ransomware attack traced back to exposed firewall configurations and backup data tied to **legacy SonicWall systems**. The misconfiguration had gone undetected for months. Attackers did not exploit a zero-day — they used what was already accessible: config files, insufficient monitoring, and perimeter controls no longer being actively reviewed.

> All three cases share the same pattern: the firewall existed. The breach happened because of how it was configured — not because it was missing.

---

## Full Flow Diagram

```
Packet arrives at network perimeter
     │
     ▼
Firewall checks ruleset (top to bottom)
     │
     ├── Match found → Apply action (Allow / Block / Drop)
     │
     └── No match → Apply default policy (usually Block)
                         │
                         ▼
                  [Stateful only]
                  Is this part of a known session?
                  ├── Yes → Allow
                  └── No  → Drop
```

---

## Known Risks

- Default rules are often too permissive — vendors ship firewalls "open" to avoid breaking connectivity
- Rules accumulate over time and are rarely audited or removed
- Legacy firewall systems may not support modern threat detection
- A single overly broad rule can expose the entire network
- Firewalls do not protect against threats originating from inside the network

---

## Best Practices

- Apply **default deny** — block everything and allow only what is needed
- Review and audit rules regularly — remove anything unused
- Never rely on a single firewall layer — use network segmentation and host-based firewalls together
- Enable logging and monitor blocked traffic for signs of reconnaissance
- Use a WAF for any web-facing application
- Keep firmware updated — attackers exploit known vulnerabilities in outdated firewall software

---

## How to Practice

| Platform | What to do |
|----------|-----------|
| **TryHackMe** | Rooms on network security, firewalls, and Nmap |
| **Hack The Box** | Machines where firewall evasion is required to progress |
| **VirtualBox / pfSense** | Set up a real firewall VM and define your own rules |
| **Nmap** | Practice scanning to understand what firewalls expose and hide |
| **Wireshark** | Analyze traffic before and after firewall rules to see what changes |

---

## What I Studied

- The difference between stateless and stateful firewall inspection
- How firewalls process packets and apply rules
- Types of firewalls and when each is used
- How attackers probe and evade firewalls
- Real cases where misconfiguration led to major breaches
- Practical scanning techniques with Nmap to test firewall exposure

---

## Requirements

- No code required — this is a networking and security concept study note
- Practical exercises done via [TryHackMe](https://tryhackme.com)

---

## Origin

Studied as part of the networking and security curriculum on [TryHackMe](https://tryhackme.com).

---

## Author

[pedromarinflach-cyber](https://github.com/pedromarinflach-cyber)
