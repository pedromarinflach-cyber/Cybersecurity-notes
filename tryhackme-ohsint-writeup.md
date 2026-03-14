# 🔍 OSINT Investigation – OhSINT Challenge

> **Platform:** [TryHackMe – OhSINT](https://tryhackme.com/room/ohsint)  
> **Category:** OSINT  
> **Difficulty:** Easy  
> **Status:** ✅ Completed

---

## Overview

This write-up documents my methodology while solving the **OhSINT** challenge on TryHackMe — my first CTF write-up.

The challenge starts with a **single image**. From that one file, the goal is to uncover multiple pieces of information about a target individual using only publicly available data and open source tools.

This document focuses on **methodology and reasoning**, not answers — the point is to understand *how* to investigate, not to hand anyone a shortcut.

---

## Tools Used

| Category | Tool |
|---|---|
| Metadata Analysis | ExifTool |
| Search Engines | Google |
| Social Media | Twitter |
| Code Repositories | GitHub |
| Wireless Networks | WiGLE |
| Website Analysis | WordPress / Browser DevTools |

---

## Investigation Methodology

### Step 1 – Metadata Analysis

Every investigation starts with what you already have. Before searching for anything, I ran **ExifTool** on the provided image to extract embedded metadata.

```bash
exiftool image.jpg
```

Images captured by phones and cameras often silently embed:

- **GPS coordinates** — physical location of the photo
- **Author / copyright fields** — username or real name of the creator
- **Device information** — camera model, software used

The metadata yielded an author name that became the **first pivot point** of the investigation.

> **Lesson:** Never underestimate the data hiding inside a file. Metadata is invisible to the eye but trivially extractable with the right tool.

---

### Step 2 – Username Enumeration

The author field from the metadata revealed a **username**. This is extremely valuable in OSINT because people routinely reuse the same alias across dozens of platforms.

Search queries used:

```
"username"
"username" site:twitter.com
"username" site:github.com
"username" blog
```

This quickly surfaced multiple online accounts linked to the same identity.

> **Lesson:** A single username can be the thread that unravels an entire digital identity.

---

### Step 3 – Social Media Investigation

One of the discovered accounts was on **Twitter**. Social media profiles are dense with unintentional disclosures:

- Photos revealing physical locations
- Screenshots exposing device or network information
- Metadata embedded in uploaded images
- Personal habits and routines

A specific post contained information related to a **wireless access point**, including a **BSSID** — the hardware identifier that uniquely identifies a Wi-Fi router.

> **Lesson:** What someone posts casually can expose infrastructure details they never intended to make public.

---

### Step 4 – Wireless Network Investigation

The BSSID was queried against **WiGLE** — a large public database of wireless networks collected through wardriving and community submissions.

A BSSID lookup can reveal:

- The **SSID** (network name)
- The **approximate physical location** of the router
- Historical mapping data

This step connected a social media post directly to a real-world location.

> **Lesson:** Wireless infrastructure leaves a persistent, searchable footprint in public databases. A leaked BSSID can geolocate a target as precisely as GPS coordinates.

---

### Step 5 – GitHub Repository Investigation

OSINT on the username led to a **public GitHub repository**. Developers often treat public repos as purely technical spaces — but they can leak:

- Personal email addresses (in commits, config files)
- API keys or credentials accidentally pushed
- Internal infrastructure details in comments
- Developer habits and tooling preferences

I inspected README files, scripts, configuration files, and inline code comments carefully.

> **Lesson:** `git log` never lies. Even deleted files can persist in commit history. Public repositories are a goldmine for investigators — and a minefield for developers.

---

### Step 6 – Website Investigation

The trail eventually led to a **personal WordPress blog**. Websites contain layers of information beyond what's visible on screen.

Techniques used:

**Viewing page source:**
```
Right Click → View Page Source  (or Ctrl+U)
```

**What to look for:**
- HTML comments (`<!-- hidden notes -->`)
- Elements hidden with CSS (`display:none`, `visibility:hidden`, `color:white`)
- Unusual script tags or external resources
- Meta tags with personal information

In this case, information was concealed within the page structure in a way that was invisible to a casual visitor but immediately visible in the source code.

> **Lesson:** "Hidden" on screen does not mean hidden in the code. Always inspect the source.

---

## What Was Difficult

The **WiGLE step** was the least intuitive part of the investigation. The jump from "BSSID in a social media post" to "query a wardriving database" is not obvious without prior exposure to wireless OSINT. It required stepping back and thinking about *what a BSSID actually is* and what databases might track it — rather than just Googling the value directly.

This was a good reminder that OSINT requires knowing what kinds of data sources exist for different data types.

---

## Key Takeaways

| Finding | Real-World Implication |
|---|---|
| Image metadata revealed GPS + author | Photos shared publicly can expose location and identity |
| Username reused across platforms | A single alias can link all of a person's online presence |
| Social media post exposed BSSID | Casual posts can leak infrastructure details |
| WiGLE mapped the BSSID to a location | Wireless networks create a persistent, searchable footprint |
| GitHub repo contained leaked data | Public repositories are frequently audited by threat actors |
| Website hid data in HTML source | "Invisible" content is still accessible to anyone who looks |

---

## Skills Practiced

- Image metadata extraction with ExifTool
- Username enumeration and cross-platform pivoting
- Social media intelligence (SOCMINT)
- Wireless network OSINT via WiGLE
- Source code and repository inspection
- Website analysis and HTML source review

---

## Final Thoughts

OhSINT is a great introductory OSINT challenge because it mirrors how real investigations work: **one small clue leads to the next**, and the investigator's job is to recognize what each piece of data implies and where to pivot next.

The broader lesson is about **OPSEC** — operational security. Every account created, every photo uploaded, every post made is a potential data point. People who don't think about this actively leave a trail that is surprisingly easy to follow.

---

*Write-up by Pedro | [TryHackMe Profile](https://tryhackme.com/p/Pedr0Marin) | No challenge answers disclosed.*
