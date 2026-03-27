# VPN — Virtual Private Network

A study note covering how VPNs work at the protocol level, tunneling and encryption, types of VPN, common protocols, and their direct relevance to cybersecurity — including real-world cases where VPNs were both the defense and the attack vector.

---

## Overview

A VPN (Virtual Private Network) is a technology that creates an encrypted tunnel between two points over a public network — typically the internet. Everything that travels through that tunnel is encrypted and encapsulated, making it unreadable to anyone intercepting the traffic in between.

VPNs serve two primary purposes:

1. **Privacy** — hide the user's real IP and encrypt traffic from ISPs, surveillance, or attackers on the same network
2. **Remote access** — allow a device in a different location to behave as if it were physically connected to a private network

```
Without VPN:
  Your Device → [Internet — visible, readable] → Destination

With VPN:
  Your Device → [Encrypted Tunnel] → VPN Server → [Internet] → Destination
                      ↑
              No one between you and the
              VPN server can read this traffic
```

> A VPN does not make you anonymous. It shifts trust — instead of trusting your ISP or local network, you are trusting your VPN provider. If the provider logs your traffic, that log exists.

---

## Key Concepts

| Term | Description |
|------|-------------|
| **Tunnel** | An encrypted connection that encapsulates traffic between two endpoints |
| **Encapsulation** | Wrapping a packet inside another packet — the original packet becomes the payload |
| **Encryption** | Transforming data so it is unreadable without the correct key |
| **Authentication** | Verifying the identity of both ends of a VPN connection |
| **VPN Client** | The device or software initiating the VPN connection |
| **VPN Server / Gateway** | The endpoint receiving the connection and routing traffic |
| **Split Tunneling** | Routing only some traffic through the VPN — the rest goes directly to the internet |
| **Kill Switch** | A feature that cuts internet access entirely if the VPN connection drops, preventing accidental exposure |
| **No-log policy** | A provider claim that they do not store records of user activity — varies in reliability |

---

## How a VPN Tunnel Works (Step by Step)

### Step 1 — Handshake and Authentication

The client and server establish a secure channel. Both sides verify each other's identity — typically using certificates, pre-shared keys, or username/password — and negotiate the encryption algorithm to use.

```
Client → Server: "I want to connect. Here are my credentials and supported algorithms."
Server → Client: "Verified. We will use AES-256 + SHA-256. Here is the session key."
```

### Step 2 — Tunnel Established

Once authenticated, the encrypted tunnel is active. Every packet the client sends is:

1. Encrypted using the agreed algorithm
2. Encapsulated inside a new packet addressed to the VPN server

```
Original packet:
  Src: 192.168.1.5   Dst: 142.250.64.100 (Google)   Payload: HTTP request

After encapsulation:
  Src: 192.168.1.5   Dst: 203.0.113.99 (VPN Server)
  Payload: [ENCRYPTED — original packet is inside, unreadable]
```

### Step 3 — VPN Server Decrypts and Forwards

The VPN server receives the encapsulated packet, decrypts it, and forwards the original packet to its actual destination. From Google's perspective, the request came from the VPN server's IP — not the client's.

### Step 4 — Response Travels Back

Google responds to the VPN server. The server encrypts the response, encapsulates it, and sends it back to the client through the tunnel.

```
Full flow:

Client → [Encrypted Tunnel] → VPN Server → Google
                                    ↑
                          Google sees this IP
                          (not the client's)
```

---

## VPN Types

### Remote Access VPN

The most common type. A single user connects to a remote network — typically a corporate network — from any location. The device receives an IP address from the remote network and behaves as if it were physically present there.

```
Home PC → [VPN Tunnel] → Corporate Network (192.168.10.0/24)
                              ↑
                  Home PC gets assigned 192.168.10.45
                  Can access internal servers, printers, shares
```

Used by: remote workers, administrators accessing internal infrastructure.

### Site-to-Site VPN

Connects two entire networks together — typically two office locations or a branch to headquarters. No individual client software required — the VPN is configured at the router/gateway level.

```
Office A (10.0.1.0/24) ←→ [VPN Tunnel] ←→ Office B (10.0.2.0/24)

Devices in Office A can reach devices in Office B
as if they were on the same local network.
```

Used by: enterprises connecting geographically separated locations.

### Client-to-Site VPN

A variation of remote access — multiple individual clients connect to a central site. Common in corporate environments where employees work remotely.

### VPN as a Privacy Service

Consumer VPNs (NordVPN, Mullvad, ProtonVPN) route all user traffic through the provider's servers. The goal is to mask the user's IP from websites and prevent local network surveillance.

**Important distinction:** this type of VPN trusts the provider completely. If the provider keeps logs, those logs can be subpoenaed, hacked, or sold.

---

## VPN Protocols

### IPSec (Internet Protocol Security)

One of the most widely used VPN protocols in enterprise environments. Operates at Layer 3 (network layer) and can encrypt entire IP packets.

IPSec has two modes:

**Transport Mode** — encrypts only the payload of the packet. The IP header remains visible.
```
[ IP Header (visible) | IPSec Header | Encrypted Payload ]
```

**Tunnel Mode** — encrypts the entire original packet and wraps it in a new IP header. The most common mode for VPNs.
```
[ New IP Header | IPSec Header | Encrypted Original Packet ]
```

IPSec uses two sub-protocols:
- **AH (Authentication Header)** — provides integrity and authentication, but no encryption
- **ESP (Encapsulating Security Payload)** — provides encryption, integrity, and authentication

In practice, ESP in tunnel mode is what most IPSec VPNs use.

**Common pairings:** IPSec is often paired with IKEv2 (Internet Key Exchange) for the handshake and key negotiation phase, forming **IKEv2/IPSec** — considered one of the most secure and stable VPN configurations available.

---

### PPTP (Point-to-Point Tunneling Protocol)

One of the oldest VPN protocols, developed by Microsoft in the 1990s. Encapsulates PPP (Point-to-Point Protocol) frames inside GRE (Generic Routing Encapsulation) packets.

**PPTP is considered broken.** Its authentication mechanism (MS-CHAPv2) has known cryptographic weaknesses that allow offline brute-force attacks. It should not be used for anything requiring real security.

```
Why it still exists:
- Very fast — minimal encryption overhead
- Supported natively on almost every OS
- Still used in environments where speed > security (streaming geo-bypass)

Why it should not be trusted:
- MS-CHAPv2 can be cracked
- No forward secrecy
- NSA and similar agencies have documented capability to decrypt PPTP traffic
```

---

### L2TP/IPSec (Layer 2 Tunneling Protocol)

L2TP alone provides no encryption — it only creates the tunnel. IPSec is paired with it to provide the encryption layer. Together they form a widely supported and reasonably secure combination.

```
L2TP:   Creates the tunnel, encapsulates PPP frames
IPSec:  Encrypts everything L2TP sends through the tunnel
```

**Weakness:** Double encapsulation (L2TP inside IPSec) makes it slower than alternatives. Also, L2TP uses UDP port 500 and 4500 — ports that are commonly blocked or flagged by firewalls, making it unreliable on restrictive networks.

---

### OpenVPN

An open-source VPN protocol using SSL/TLS for encryption. Highly configurable, auditable, and considered very secure when properly set up.

```
Key properties:
- Uses TLS — the same encryption standard as HTTPS
- Runs over TCP or UDP (configurable)
- Can run on port 443 — making it very difficult to block (looks like HTTPS)
- Open source — publicly audited
```

**Weakness:** Slower than WireGuard. More complex to configure. Client software required.

---

### WireGuard

The newest major VPN protocol. Designed to be simpler, faster, and more auditable than IPSec or OpenVPN.

```
Code comparison:
  IPSec:    ~400,000 lines of code
  OpenVPN:  ~70,000 lines of code
  WireGuard: ~4,000 lines of code
```

Fewer lines of code = smaller attack surface = easier to audit for vulnerabilities.

WireGuard uses modern cryptography: ChaCha20 for encryption, Curve25519 for key exchange, Poly1305 for authentication. It operates at the kernel level on Linux, giving it a significant speed advantage.

**Weakness:** Does not hide that you are using a VPN (no obfuscation). Assigns static IP addresses by default — which has privacy implications if the server logs connection times alongside IPs.

---

### PPP (Point-to-Point Protocol)

PPP is not a VPN protocol by itself — it is a data link layer protocol used to establish direct connections between two nodes. It handles authentication, compression, and error detection over a direct link.

In the VPN context, PPP is the foundation that PPTP and L2TP both encapsulate. When you connect via PPTP or L2TP, a PPP session is established first, then that session is wrapped inside the tunneling protocol.

```
L2TP/PPTP VPN structure:

[ Outer IP Header ] → [ Tunnel Header (GRE or L2TP) ] → [ PPP Frame ] → [ Original Payload ]
                                                               ↑
                                                   Authentication, compression,
                                                   and framing happen here
```

---

## Protocol Comparison

| Protocol | Security | Speed | Firewall Bypass | Use Case |
|----------|----------|-------|-----------------|----------|
| PPTP | ❌ Broken | ✅ Fast | ✅ Easy | Legacy only — do not use |
| L2TP/IPSec | ✅ Good | ⚠️ Moderate | ⚠️ Sometimes blocked | Compatibility |
| IPSec/IKEv2 | ✅ Very Good | ✅ Fast | ⚠️ Sometimes blocked | Enterprise, mobile |
| OpenVPN | ✅ Very Good | ⚠️ Moderate | ✅ Port 443 bypass | Privacy, flexibility |
| WireGuard | ✅ Excellent | ✅ Fastest | ⚠️ No obfuscation | Modern deployments |

---

## Split Tunneling

By default, a VPN routes all traffic through the tunnel. Split tunneling changes this — only specific traffic goes through the VPN, the rest goes directly to the internet.

```
Split tunneling OFF (default):
  All traffic → VPN tunnel → VPN server → destination

Split tunneling ON:
  Corporate traffic (10.0.0.0/8) → VPN tunnel → corporate network
  Everything else → direct internet connection
```

**Security risk:** if a device is connected to a corporate VPN with split tunneling enabled and that device is compromised, the attacker has a direct path into the corporate network — bypassing the VPN's intended perimeter.

---

## VPN in a Cybersecurity Context

### From a Defender's Perspective

VPNs are the standard method for giving remote employees access to internal infrastructure without exposing services directly to the internet. Instead of forwarding port 3389 (RDP) publicly, a company runs a VPN — employees connect to the VPN, then access RDP internally.

```
Bad approach:
  Internet → port 3389 open → RDP server (exposed)

Good approach:
  Internet → VPN gateway → internal network → RDP server (never exposed)
```

VPNs also protect traffic on untrusted networks. An employee on a hotel Wi-Fi is on a network controlled by an unknown party. A VPN encrypts all traffic before it ever touches that network.

### From an Attacker's Perspective

VPN credentials are high-value targets. A compromised VPN account gives an attacker a legitimate authenticated entry point into the internal network — with the same access as the legitimate user.

**Common attack vectors against VPNs:**

- **Credential stuffing** — using leaked username/password pairs from other breaches to log into VPN portals
- **Exploiting unpatched VPN appliances** — VPN gateways are internet-facing and frequently have CVEs
- **MFA fatigue / push bombing** — spamming authentication push notifications until the user approves
- **Rogue VPN apps** — fake VPN clients that harvest credentials or route traffic through attacker infrastructure

```bash
# VPN gateway fingerprinting with Nmap
nmap -sV -p 500,1194,1723,4500 203.0.113.5

# Common VPN ports:
# 500/udp   — IKE (IPSec)
# 1194/udp  — OpenVPN (default)
# 1723/tcp  — PPTP
# 4500/udp  — IPSec NAT traversal
```

---

## Real-World Breaches

### Ivanti VPN (2024)
Two critical zero-day vulnerabilities (CVE-2024-21887 and CVE-2023-46805) in Ivanti Connect Secure VPN appliances were actively exploited before patches were available. Attackers used the flaws to bypass authentication entirely and execute arbitrary commands on the gateway. Thousands of organizations were affected — including government agencies. The entry point was the VPN itself, not the network it was protecting.

### Pulse Secure VPN (2021)
A critical vulnerability (CVE-2019-11510) in Pulse Secure VPN — disclosed and patched in 2019 — was still being actively exploited in 2021 by ransomware groups and nation-state actors. Organizations that had not applied the patch two years later had their VPN credentials stolen, internal networks accessed, and in several cases ransomware deployed. The vulnerability allowed unauthenticated file read — attackers pulled plaintext credentials directly from the device.

### Colonial Pipeline (2021)
The ransomware attack that shut down fuel supply to the US East Coast for days originated through a **compromised VPN account**. The account was no longer in active use but had not been deactivated. Its credentials appeared in a batch of leaked passwords from a separate breach. No MFA was enabled on that VPN profile. One unused, unprotected account was the entry point for an attack that caused a national emergency.

> The pattern across all three: the VPN was the perimeter. Attackers did not go around it — they went through it. Unpatched appliances, stolen credentials, and missing MFA are the three most common causes.

---

## Known Risks

- VPN appliances are internet-facing and frequently targeted — unpatched gateways are a major attack surface
- Stolen VPN credentials give attackers a legitimate, authenticated entry point
- No MFA on VPN access removes the only barrier after credentials are compromised
- Free VPN providers frequently monetize user traffic — the product is the data
- Split tunneling can expose internal networks if the connected device is compromised
- A VPN obscures traffic from the local network but not from the VPN provider itself
- PPTP should be considered broken — any system still using it is vulnerable

---

## Best Practices

- Always enable MFA on VPN access — credentials alone are not sufficient
- Patch VPN appliances immediately — they are internet-facing and actively targeted
- Disable and remove unused VPN accounts — the Colonial Pipeline breach started with a dormant account
- Use modern protocols — WireGuard or IKEv2/IPSec — avoid PPTP entirely
- Enable a kill switch to prevent accidental traffic exposure if the tunnel drops
- Monitor VPN logs for unusual login times, locations, or access patterns
- Apply least privilege to VPN access — a user who needs access to one system should not reach the entire network
- Use split tunneling carefully — if enabled, ensure endpoint security on all connected devices

---

## Full Flow Diagram

```
Client initiates connection
     │
     ▼
Handshake — identity verified, encryption negotiated
     │
     ▼
Tunnel established
     │
     ▼
Client sends packet → encrypted + encapsulated → sent to VPN server
     │
     ▼
VPN server decrypts → forwards original packet to destination
     │
     ▼
Response returns to VPN server → encrypted → sent back through tunnel
     │
     ▼
Client decrypts → receives response
     │
     ▼
Destination never saw the client's real IP
Local network never saw the contents of the traffic
```

---

## What I Studied

- How VPN tunneling and encapsulation work at the packet level
- The difference between remote access, site-to-site, and privacy VPNs
- VPN protocols: PPP, PPTP, L2TP/IPSec, IPSec/IKEv2, OpenVPN, WireGuard
- Why PPTP is broken and should not be used
- Split tunneling and its security implications
- How attackers target VPN infrastructure — credential stuffing, unpatched appliances, MFA bypass
- Real breaches caused by VPN misconfiguration and neglected credentials

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
