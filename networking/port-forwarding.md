# Port Forwarding

A study note covering how port forwarding works, its types, common use cases, and its direct relevance to cybersecurity — including real-world breaches caused by misconfigured or exposed ports.

---

## Overview

Port forwarding is a technique configured on routers and firewalls that redirects incoming traffic from a specific port on the public IP to a device inside the private network (local IP + port).

It exists because of **NAT (Network Address Translation)** — multiple devices share one public IP. From the outside, everything hits the same address. Port forwarding tells the router exactly where to send that traffic internally.

**Example:**

```
External request → 203.0.113.5:8080
Router rule      → forward to 192.168.1.50:80
Internal server  → receives the request on port 80
```

---

## Key Concepts

| Term | Description |
|------|-------------|
| **NAT** | Translates public IPs to private IPs across a router |
| **Public IP** | The address visible on the internet |
| **Private IP** | The internal address assigned to a device on the local network |
| **Port** | A logical channel used to distinguish types of traffic |
| **Forwarding rule** | The configuration that maps external port → internal IP:port |

---

## How It Works (Step by Step)

### Step 1 — Packet arrives at the router

```
Incoming packet:
  Destination IP:   203.0.113.5   (public IP)
  Destination Port: 8080
```

The router receives a packet from the internet destined for the public IP on port 8080.

### Step 2 — Router checks forwarding rules

```
Rule: IF port 8080 → send to 192.168.1.50:80
```

The router matches the destination port against its forwarding table. If a rule exists, it rewrites the packet header.

### Step 3 — Packet is rewritten and forwarded

```
Modified packet:
  Destination IP:   192.168.1.50  (private IP)
  Destination Port: 80
```

The packet is now sent to the internal device. From the server's perspective, it just received a normal local request.

### Step 4 — Response goes back through the router

The server replies to the router, which reverses the translation and sends the response back to the original external requester.

---

## Types of Port Forwarding

### Static (Manual)
A permanent rule set manually by the administrator. Always active.

```
External Port 22  →  192.168.1.10:22  (SSH server)
External Port 80  →  192.168.1.20:80  (Web server)
```

Used for servers that need to be reliably accessible from outside.

### Dynamic (UPnP)
Rules created automatically by applications — games, torrent clients, video calls. No manual configuration required.

**Security risk:** any application on the network can open a port without the administrator knowing. This is a common attack surface in home and small business networks.

### DMZ (Demilitarized Zone)
All external traffic is forwarded to a single device — regardless of port. That device is fully exposed to the internet.

```
All traffic → 192.168.1.99
```

Used for hosting servers that need unrestricted access, but it completely removes the NAT protection for that host. One compromised service = full exposure.

---

## Common Ports

| Port | Protocol | Service |
|------|----------|---------|
| 21 | TCP | FTP (File Transfer) |
| 22 | TCP | SSH (Secure Shell) |
| 80 | TCP | HTTP (Web) |
| 443 | TCP | HTTPS (Secure Web) |
| 3306 | TCP | MySQL (Database) |
| 3389 | TCP | RDP (Windows Remote Desktop) |
| 8080 | TCP | Alternative HTTP / Dev servers |

---

## Security Relevance

Port forwarding is one of the first things an attacker maps during reconnaissance. Every open port is a potential entry point — especially if the service behind it is outdated, misconfigured, or has no authentication.

Tools like **Nmap** are used in both offensive and defensive security to discover exactly which ports are exposed and what is running behind them.

```bash
nmap -sV 203.0.113.5
```

This command scans the target IP and attempts to identify services and versions on each open port — the same technique used in CTFs and real pentests.

---

## Real-World Breaches

### Capital One (2019)
A misconfigured web application firewall (WAF) was exploited via a Server-Side Request Forgery (SSRF) attack. The attacker used the exposed service to query AWS metadata, retrieve temporary credentials, and access over **100 million customer records**. The firewall was not blocking the request because the rule was incorrectly set — it was open when it should have been restricted.

### Target (2013)
Attackers entered Target's internal network through a third-party vendor's credentials. A misconfigured firewall allowed lateral movement from the vendor's segment into the payment systems network — which should have been isolated. The breach exposed **40 million credit card records**.

### Grafana Homelab (2025)
A DevOps engineer forwarded port 3000 to expose a Grafana monitoring dashboard. The instance was running an unpatched version with a known vulnerability (CVE-2023-38123) and had no authentication configured. An automated scanner found the port, exploited the vulnerability, deployed a reverse shell, and eventually stole GitHub tokens — resulting in malicious commits pushed to **three client repositories**.

> These cases share a common pattern: an exposed port + an unpatched or misconfigured service = full compromise. The attack surface begins at the forwarding rule.

---

## Full Flow Diagram

```
Internet
     │
     ▼
Public IP:PORT hits router
     │
     ▼
Does a forwarding rule exist? ──No──► Packet dropped or rejected
     │ Yes
     ▼
Router rewrites destination → Private IP:PORT
     │
     ▼
Internal device receives the packet
     │
     ▼
Response travels back through router → returned to external requester
```

---

## Known Risks

- UPnP allows applications to open ports silently — no admin approval required
- DMZ removes all NAT protection for the exposed host
- Forgotten rules leave ports open long after they are needed
- Services with default credentials behind forwarded ports are trivially exploited
- Out-of-range guesses still increment the tries counter

---

## Best Practices

- Only forward ports that are strictly necessary
- Disable UPnP on routers unless actively required
- Restrict forwarded ports to specific source IPs where possible
- Always run updated, authenticated services behind forwarded ports
- Audit forwarding rules regularly and remove anything unused
- Use a VPN instead of port forwarding for remote access where possible

---

## What I Studied

- How NAT works and why port forwarding exists
- The difference between static, dynamic, and DMZ forwarding
- How attackers use open ports as entry points during reconnaissance
- Real-world cases where port exposure led to full network compromise
- How Nmap maps exposed services during a pentest or CTF

---

## Requirements

- No code required — this is a networking concept study note
- Practical exercises done via [TryHackMe](https://tryhackme.com)

---

## Origin

Studied as part of the networking and security curriculum on [TryHackMe](https://tryhackme.com).

---

## Author

[pedromarinflach-cyber](https://github.com/pedromarinflach-cyber)
