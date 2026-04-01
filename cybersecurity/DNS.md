# DNS — Domain Name System

A study note covering how DNS works at the protocol level, the hierarchy of domain names, record types, resolution flow, and the direct relevance of DNS to cybersecurity — including real-world attacks where DNS was both the attack vector and the target.

---

## Overview

DNS (Domain Name System) is the internet's distributed directory service. It translates human-readable domain names into IP addresses that machines use to communicate.

Without DNS, every request to a website would require knowing its exact IP address. DNS handles this translation invisibly, billions of times per day.

```
Without DNS:
  You type: 142.250.64.100  → reaches Google
  You type: 1.1.1.1         → reaches Cloudflare

With DNS:
  You type: google.com      → DNS resolves → 142.250.64.100 → reaches Google
  You type: cloudflare.com  → DNS resolves → 1.1.1.1        → reaches Cloudflare
```

DNS operates on **port 53** — UDP for most queries (fast, stateless), TCP for large responses or zone transfers.

> DNS is one of the most abused protocols in cybersecurity. Because it is always allowed through firewalls and generates enormous volumes of legitimate traffic, attackers use it for data exfiltration, command-and-control, and lateral movement — often undetected.

---

## Key Concepts

| Term | Description |
|------|-------------|
| **Domain Name** | A human-readable address like `mail.google.com` |
| **IP Address** | The numeric address a domain resolves to — what machines actually use |
| **Resolver** | The DNS server your device queries first — usually your ISP or a public resolver |
| **Recursive Resolver** | A resolver that does the full lookup on your behalf, querying other servers as needed |
| **Root Name Server** | The top of the DNS hierarchy — knows where to find TLD servers |
| **TLD Server** | Authoritative for a top-level domain (`.com`, `.org`, `.br`) |
| **Authoritative Name Server** | The final authority for a specific domain — holds the actual records |
| **DNS Record** | An entry in a DNS zone that maps a name to data (IP, mail server, text, etc.) |
| **TTL (Time to Live)** | How long a DNS response is cached before being re-queried |
| **Zone** | A portion of the DNS namespace managed by a specific authoritative server |
| **Zone Transfer** | A mechanism to replicate DNS records between name servers (AXFR) |
| **DNS Cache** | A local store of recent DNS responses — reduces latency and query volume |

---

## The Domain Name Hierarchy

Domain names are read **right to left** — from the most general to the most specific.

```
          mail.accounts.google.com.
          │    │        │      │  └── Root (.)
          │    │        │      └───── TLD (.com)
          │    │        └──────────── Second-Level Domain (google)
          │    └───────────────────── Subdomain (accounts)
          └────────────────────────── Subdomain (mail)
```

Every fully qualified domain name (FQDN) ends in a dot — the root. It is usually omitted but always implied.

---

### Root Zone

The root (`.`) is the starting point of all DNS resolution. It is maintained by IANA and served by **13 root server clusters** (labeled A through M), distributed globally through anycast. These servers do not know the answer to any specific domain query — they only know where to find the TLD servers.

```
Root servers answer:
  "I don't know what google.com is, but I know who manages .com → go ask them."
```

---

### TLD — Top-Level Domain

The TLD is the rightmost label of a domain name. There are two main categories:

#### gTLD — Generic Top-Level Domain

Not tied to any country. Originally designed to categorize the type of organization.

| gTLD | Original Purpose |
|------|-----------------|
| `.com` | Commercial organizations |
| `.org` | Non-profit organizations |
| `.net` | Network infrastructure |
| `.edu` | Educational institutions (US) |
| `.gov` | US government entities |
| `.mil` | US military |
| `.int` | International organizations |

In 2012, ICANN opened the new gTLD program — allowing entities to apply for virtually any string as a TLD. This created thousands of new gTLDs like `.app`, `.shop`, `.bank`, `.cloud`, `.io`, `.dev`.

**Security note:** New gTLDs are frequently abused for phishing. A domain like `paypal.secure-login.com` looks suspicious, but `paypal.secure.login` (hypothetical) would confuse many users. Attackers register visually similar domains under new gTLDs to impersonate legitimate services.

#### ccTLD — Country Code Top-Level Domain

Two-letter TLDs assigned to specific countries or territories, based on ISO 3166-1.

| ccTLD | Country |
|-------|---------|
| `.br` | Brazil |
| `.uk` | United Kingdom |
| `.de` | Germany |
| `.ru` | Russia |
| `.cn` | China |
| `.io` | British Indian Ocean Territory (heavily used by tech companies) |
| `.tv` | Tuvalu (repurposed for streaming/media brands) |

Some ccTLDs have become culturally detached from their country and are used commercially (`.io`, `.tv`, `.ai` — Anguilla, not artificial intelligence). This creates confusion and is exploited in phishing campaigns targeting the tech industry.

---

### Second-Level Domain (SLD)

The label directly to the left of the TLD. This is what organizations register — it is the core identity of the domain.

```
google.com  →  "google" is the SLD
github.com  →  "github" is the SLD
gov.br      →  "gov" is the SLD under .br (Brazil's government structure adds an extra layer)
```

Registration of second-level domains is managed by **registrars** — accredited entities authorized to sell domain names under each TLD. Examples: GoDaddy, Namecheap, Registro.br.

---

### Subdomains

Everything to the left of the SLD is a subdomain. Subdomains are created by the domain owner — no registration required.

```
mail.google.com        →  mail is a subdomain of google.com
accounts.google.com    →  accounts is a subdomain of google.com
blog.github.io         →  blog is a subdomain of github.io
vpn.corp.example.com   →  vpn and corp are nested subdomains
```

Organizations use subdomains to separate services:

```
mail.company.com       →  Email
vpn.company.com        →  VPN gateway
portal.company.com     →  Internal portal
dev.company.com        →  Development environment
```

**Security note:** Subdomain enumeration is a core reconnaissance technique. Attackers scan for subdomains to find forgotten development environments, unpatched services, and shadow IT — infrastructure the security team may not know exists.

---

## How DNS Resolution Works (Step by Step)

When you type `www.example.com` into a browser, the following happens:

### Step 1 — Local Cache Check

```
Your OS checks:
  1. Browser DNS cache
  2. OS DNS cache
  3. /etc/hosts (or C:\Windows\System32\drivers\etc\hosts on Windows)

If found → resolution complete, no network query needed.
```

### Step 2 — Query Goes to Recursive Resolver

If not cached, your device sends a query to its configured DNS resolver — typically your ISP's resolver or a public one (8.8.8.8, 1.1.1.1).

```
Your device → Recursive Resolver: "What is the IP of www.example.com?"
```

The recursive resolver now does the heavy lifting on your behalf.

### Step 3 — Resolver Queries a Root Server

```
Recursive Resolver → Root Server: "What is www.example.com?"
Root Server        → "I don't know, but .com is managed by these TLD servers: [list]"
```

### Step 4 — Resolver Queries the TLD Server

```
Recursive Resolver → .com TLD Server: "What is www.example.com?"
TLD Server         → "I don't know the IP, but example.com is managed by ns1.example.com"
```

### Step 5 — Resolver Queries the Authoritative Name Server

```
Recursive Resolver → ns1.example.com: "What is www.example.com?"
Authoritative NS   → "www.example.com is 93.184.216.34"  ← Final answer
```

### Step 6 — Answer Returns to Client

```
Recursive Resolver → Your device: "www.example.com = 93.184.216.34"
                          ↑
                   Response cached for TTL duration
```

### Full Flow

```
Client
  │
  ├──► Local cache? ──Yes──► Done
  │
  No
  │
  ▼
Recursive Resolver
  │
  ├──► Root Server ──► "Ask .com TLD"
  │
  ├──► .com TLD Server ──► "Ask ns1.example.com"
  │
  ├──► Authoritative NS (ns1.example.com) ──► "93.184.216.34"
  │
  └──► Returns answer to client + caches with TTL
```

---

## DNS Record Types

DNS records define what a query for a domain should return. Each record has a **type**, a **name**, a **value**, and a **TTL**.

| Record | Full Name | Purpose |
|--------|-----------|---------|
| **A** | Address | Maps a hostname to an IPv4 address |
| **AAAA** | IPv6 Address | Maps a hostname to an IPv6 address |
| **CNAME** | Canonical Name | Alias — points one name to another name |
| **MX** | Mail Exchange | Specifies mail servers for the domain |
| **TXT** | Text | Arbitrary text — used for SPF, DKIM, DMARC, domain verification |
| **NS** | Name Server | Identifies the authoritative name servers for the domain |
| **PTR** | Pointer | Reverse DNS — maps an IP back to a hostname |
| **SOA** | Start of Authority | Metadata about the zone — serial number, refresh times, admin contact |
| **SRV** | Service | Specifies location of services (port + hostname) — used by SIP, XMPP |
| **CAA** | Certification Authority Authorization | Restricts which CAs can issue TLS certificates for the domain |

### Record Examples

```
; A record
www.example.com.    300    IN    A       93.184.216.34

; AAAA record
www.example.com.    300    IN    AAAA    2606:2800:220:1:248:1893:25c8:1946

; CNAME record — shop.example.com is an alias for example.myshopify.com
shop.example.com.   300    IN    CNAME   example.myshopify.com.

; MX records — priority 10 is preferred over priority 20
example.com.        3600   IN    MX   10 mail1.example.com.
example.com.        3600   IN    MX   20 mail2.example.com.

; TXT record — SPF policy
example.com.        3600   IN    TXT   "v=spf1 include:_spf.google.com ~all"

; NS records
example.com.        86400  IN    NS    ns1.example.com.
example.com.        86400  IN    NS    ns2.example.com.
```

---

## DNS in a Cybersecurity Context

### From an Attacker's Perspective

**Subdomain Enumeration**

Before attacking a target, attackers map all subdomains to discover forgotten services, dev environments, and shadow IT.

```bash
# Passive — query public sources without touching the target
subfinder -d example.com

# Active — brute-force subdomain names against the target's DNS
gobuster dns -d example.com -w /usr/share/wordlists/subdomains.txt

# Certificate transparency logs — finds subdomains from public TLS certs
curl "https://crt.sh/?q=%.example.com&output=json"
```

A single forgotten subdomain running an unpatched WordPress install has been the entry point for full corporate compromises.

---

**DNS Zone Transfer (AXFR)**

Zone transfers are designed for replication between primary and secondary name servers. When misconfigured, they allow anyone to request a full dump of the DNS zone — revealing every subdomain, internal hostname, and IP address the organization uses.

```bash
# Attempt zone transfer — should fail on properly configured servers
dig axfr example.com @ns1.example.com

# Successful output reveals the entire zone:
; <<>> DiG 9.18 <<>> axfr example.com
example.com.          86400  SOA   ns1.example.com. ...
www.example.com.      300    A     93.184.216.34
mail.example.com.     300    A     10.0.0.25
dev.example.com.      300    A     10.0.0.100         ← internal host exposed
vpn.example.com.      300    A     203.0.113.42        ← VPN gateway exposed
```

A successful AXFR against a real target is a critical finding. It collapses the reconnaissance phase entirely.

**Mitigation:** Restrict zone transfers to authorized secondary name servers only — deny all others.

---

**DNS Cache Poisoning**

An attacker sends forged DNS responses to a recursive resolver before the legitimate response arrives. If successful, the resolver caches the fake record — and every client using that resolver gets sent to the attacker's server.

```
Normal:
  Resolver → ns1.example.com: "What is bank.com?"
  ns1.example.com → "93.184.216.34" (real IP)

Poisoned:
  Attacker floods resolver with fake responses:
  "bank.com = 1.2.3.4 (attacker's server)"
  → Resolver caches attacker's answer
  → All clients are redirected to the fake server
```

Classic cache poisoning relied on predictable transaction IDs. The Kaminsky Attack (2008) demonstrated that the full 16-bit transaction ID space could be exhausted fast enough to reliably poison caches — affecting the entire internet.

**Mitigation:** DNSSEC — digitally signs DNS records so resolvers can verify authenticity.

---

**DNS Tunneling**

DNS is almost never blocked at the firewall level. Attackers exploit this by encoding data inside DNS queries and responses — creating a covert communication channel that bypasses network controls entirely.

```
Command-and-control over DNS:

Attacker controls: attacker.com (authoritative name server)

Compromised host → DNS query: cmd.aGVsbG8=.attacker.com
                              ↑
                   Base64-encoded command request

Attacker's NS     → DNS response: TXT "aGVsbG8gd29ybGQ="
                                  ↑
                   Base64-encoded response data

No HTTP, no TCP — just DNS queries and responses.
Completely invisible to most firewalls and basic monitoring.
```

```bash
# Tools used for DNS tunneling:
iodine     # Tunnels IPv4 over DNS
dnscat2    # Command-and-control over DNS
```

**Detection:** Unusually long subdomain labels, high query frequency to a single domain, entropy analysis of queried names, TXT record responses with encoded data.

---

**DNS Hijacking**

Instead of poisoning the cache, the attacker modifies where DNS queries go — either by compromising the resolver itself, the domain registrar account, or the authoritative name server.

```
Variants:

Local hijacking:   Malware on the victim's machine changes /etc/resolv.conf
                   or the router's DNS settings

Registrar hijack:  Attacker compromises registrar account →
                   changes NS records → all resolution goes to attacker's server

BGP + DNS:         Nation-state actors route traffic for a DNS provider through
                   their infrastructure, silently intercepting queries at scale
```

---

**Typosquatting and Lookalike Domains**

Attackers register domains that visually resemble legitimate ones to capture mistyped URLs or deceive users in phishing campaigns.

```
Legitimate:   paypal.com
Typosquats:   paypa1.com      (1 instead of l)
              paypall.com     (double l)
              paypal.com.login-secure.net  (subdomain abuse)
              рaypal.com      (Cyrillic р — looks identical in most fonts)
```

The last example is a **homograph attack** — using Unicode characters that are visually indistinguishable from ASCII. A user reading the URL cannot tell it is fake.

---

**NXDOMAIN-based Reconnaissance**

Querying non-existent subdomains returns NXDOMAIN responses. By observing which queries return NXDOMAIN vs valid records, attackers can passively map an organization's DNS structure without generating suspicious traffic on the target's servers.

---

### From a Defender's Perspective

**DNSSEC — DNS Security Extensions**

DNSSEC adds digital signatures to DNS records. A resolver with DNSSEC validation enabled can verify that a response was signed by the legitimate zone owner and has not been tampered with.

```
Without DNSSEC:
  Anyone can forge: "google.com = 1.2.3.4"
  Resolver has no way to verify authenticity

With DNSSEC:
  Response includes a cryptographic signature (RRSIG record)
  Resolver validates against the public key (DNSKEY record)
  Forged responses fail validation → discarded
```

DNSSEC does not encrypt DNS traffic — it only authenticates it. Queries and responses are still visible to network observers.

**DNS over HTTPS (DoH) and DNS over TLS (DoT)** solve the privacy problem by encrypting DNS traffic, preventing ISPs and on-path attackers from reading queries.

---

**SPF, DKIM, and DMARC — Email Authentication via DNS**

These three email security standards all rely on DNS TXT records.

```
SPF (Sender Policy Framework):
  Lists which servers are allowed to send email for the domain
  TXT "v=spf1 include:_spf.google.com ~all"

DKIM (DomainKeys Identified Mail):
  Adds a cryptographic signature to outgoing email
  The public key is published as a DNS TXT record — receivers verify the signature

DMARC (Domain-based Message Authentication, Reporting & Conformance):
  Tells receivers what to do with emails that fail SPF or DKIM
  TXT "v=DMARC1; p=reject; rua=mailto:dmarc@example.com"
```

Domains without DMARC (policy=reject) can be trivially spoofed for phishing. An attacker can send email that appears to come from `ceo@example.com` with no authentication barriers.

**Querying DNS records for reconnaissance:**

```bash
# Query specific record types
dig A example.com
dig MX example.com
dig TXT example.com
dig NS example.com

# Full record dump (when allowed)
dig axfr example.com @ns1.example.com

# Reverse DNS lookup
dig -x 93.184.216.34

# Check DNSSEC
dig +dnssec example.com

# Use a specific resolver
dig @1.1.1.1 example.com
```

---

## Real-World Breaches

### Sea Turtle / UNC1326 (2019)
A nation-state threat actor (attributed to Turkey) conducted a multi-year DNS hijacking campaign targeting registrars, DNS providers, and telecommunications companies across the Middle East, North Africa, and Europe. By gaining access to registrar accounts, they modified **NS records** of government and military domains — redirecting all DNS traffic for those domains through attacker-controlled servers. Victims' credentials were harvested at scale via man-in-the-middle interception of the intercepted traffic. The attack was entirely invisible to the victim organizations — their own domains were being served by attacker infrastructure.

### Krebs on Security / Brian Krebs DDoS via DNS Amplification (2016)
The Mirai botnet launched one of the largest DDoS attacks ever recorded against security journalist Brian Krebs's website, peaking at **620 Gbps**. Part of the attack used **DNS amplification** — a technique where attackers spoof the victim's IP and send small DNS queries to open resolvers, which respond with much larger answers sent to the victim. A 60-byte query can generate a 4,000-byte response — a 70x amplification factor. Open recursive resolvers are the enabler.

### Dyn DNS Attack (2016)
A Mirai-based DDoS attack targeted **Dyn**, a major DNS provider, taking down authoritative DNS for dozens of major services including Twitter, Netflix, Reddit, Airbnb, and GitHub — all simultaneously. Because Dyn handled DNS for those services, their domains became unresolvable. The attack highlighted a critical dependency: even if your own infrastructure is uncompromised, your DNS provider is part of your attack surface.

### SolarWinds DNS Exfiltration (SUNBURST, 2020)
The SUNBURST implant embedded in SolarWinds Orion used DNS as its primary C2 channel. Infected hosts encoded victim environment data inside subdomain labels of queries to `avsvmcloud.com` — a domain controlled by the attackers. The data exfiltrated this way included the organization's unique GUID, domain name, and security product information — allowing attackers to determine which targets were worth further exploitation. Queries looked like legitimate DNS traffic and bypassed security controls in thousands of organizations, including US government agencies.

> The pattern across these cases: DNS is trusted infrastructure. Attackers who control DNS — whether through registrar compromise, cache poisoning, or tunneling — operate below the visibility of most security tools. Protecting DNS is not optional.

---

## Common Misconfigurations

| Misconfiguration | Risk | Fix |
|-----------------|------|-----|
| Zone transfers allowed from any IP | Full zone enumeration by attackers | Restrict AXFR to authorized secondary NS only |
| No DNSSEC | Cache poisoning, forged responses | Enable DNSSEC on all public zones |
| Open recursive resolver | DNS amplification DDoS, unauthorized use | Restrict recursion to internal clients only |
| No DMARC / SPF / DKIM | Domain spoofing for phishing | Deploy all three email authentication records |
| DMARC policy = none | Phishing reports generated but mail not rejected | Move to p=quarantine, then p=reject |
| Long TTLs on all records | Slow incident response — poisoned records persist | Use short TTLs for critical records during incidents |
| Wildcard DNS records | Attacker-created subdomains resolve to real infrastructure | Avoid wildcards unless strictly required |
| Single DNS provider | Provider outage or DDoS = full resolution failure | Use multiple geographically distributed providers |
| Internal hostnames in public DNS | Internal network topology exposed | Keep internal DNS strictly internal |

---

## Best Practices

- Enable DNSSEC on all public zones — prevents cache poisoning and record forgery
- Restrict zone transfers — AXFR should only be permitted between authorized name servers
- Close open recursive resolvers — accept queries only from internal networks
- Deploy SPF, DKIM, and DMARC with a reject policy — prevents email spoofing
- Monitor DNS logs — high query rates to unusual domains, long subdomain labels, and TXT responses with encoded data are all red flags
- Use DNS over HTTPS (DoH) or DNS over TLS (DoT) for client queries — encrypts traffic against on-path observers
- Separate internal and external DNS — internal hostnames should never appear in public DNS
- Use CAA records — restrict which certificate authorities can issue TLS certificates for your domain
- Audit NS and MX records regularly — registrar account compromise can silently redirect all traffic
- Set low TTLs on critical records when changes are expected — reduces propagation time and exposure window during incidents

---

## Full Flow Diagram

```
User types: www.example.com
     │
     ▼
Browser cache? ──Yes──► Done
     │ No
     ▼
OS cache? ──Yes──► Done
     │ No
     ▼
/etc/hosts? ──Yes──► Done
     │ No
     ▼
Query sent to Recursive Resolver (e.g., 1.1.1.1)
     │
     ▼
Resolver cache? ──Yes──► Return cached answer
     │ No
     ▼
Resolver → Root Server: "www.example.com?"
           Root → "Ask .com TLD server at [IP]"
     │
     ▼
Resolver → .com TLD Server: "www.example.com?"
           TLD → "Ask ns1.example.com at [IP]"
     │
     ▼
Resolver → Authoritative NS (ns1.example.com): "www.example.com?"
           NS → "A record: 93.184.216.34, TTL 300"
     │
     ▼
Resolver caches response for 300 seconds
     │
     ▼
Returns 93.184.216.34 to client
     │
     ▼
Browser connects to 93.184.216.34
```

---

## What I Studied

- How DNS resolves domain names — the full recursive lookup chain
- The domain name hierarchy: root, TLD, SLD, subdomains
- The difference between gTLDs and ccTLDs, and how both are abused in phishing
- All major DNS record types: A, AAAA, CNAME, MX, TXT, NS, PTR, SOA, SRV, CAA
- DNS attack techniques: cache poisoning, zone transfer abuse, DNS tunneling, hijacking, typosquatting, amplification
- How DNSSEC authenticates DNS responses, and how DoH/DoT encrypt DNS traffic
- Email authentication via DNS: SPF, DKIM, and DMARC
- Real breaches caused by DNS infrastructure compromise
- Reconnaissance tools: subfinder, gobuster dns, dig, crt.sh

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
