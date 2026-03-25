# Switches and VLANs

A study note covering how switches work at the hardware level, how VLANs segment networks logically, trunking, inter-VLAN routing, and how both concepts appear in real-world attack scenarios and enterprise defense.

---

## Overview

A **switch** is a Layer 2 network device that connects devices within the same network and forwards frames based on MAC addresses. Unlike a hub — which broadcasts everything to everyone — a switch learns where each device is and sends traffic only to the intended destination.

A **VLAN (Virtual Local Area Network)** is a logical segmentation of a physical network. It allows you to divide a single physical switch into multiple isolated broadcast domains — as if they were separate physical networks — without needing separate hardware.

Together, switches and VLANs form the backbone of internal network design in every organization.

```
Without VLANs:
[ PC1 ]──[ PC2 ]──[ PC3 ]──[ Printer ]
All devices in the same broadcast domain.
If PC1 sends a broadcast, everyone receives it.

With VLANs:
[ PC1 ]──┐                [ PC3 ]──┐
[ PC2 ]──┤ VLAN 10 (IT)   [ Printer]┤ VLAN 20 (Finance)
         └──[ SWITCH ]────┘
PC1 cannot reach PC3 without going through a router.
```

> In a breach, flat networks — no VLANs, no segmentation — allow an attacker to move laterally from a single compromised device to every other device on the network. VLANs are one of the primary controls that stop this.

---

## Key Concepts

| Term | Description |
|------|-------------|
| **Frame** | A Layer 2 unit of data — the switch's basic unit of work |
| **MAC Address** | Hardware address burned into every NIC — used by switches to forward frames |
| **MAC Table / CAM Table** | The switch's internal database mapping MAC addresses to ports |
| **Broadcast Domain** | A group of devices that receive each other's broadcasts |
| **VLAN** | A logical grouping of ports into an isolated broadcast domain |
| **Access Port** | A switch port assigned to a single VLAN — used for end devices |
| **Trunk Port** | A port that carries traffic for multiple VLANs simultaneously — used between switches or to routers |
| **802.1Q** | The IEEE standard for VLAN tagging on trunk links |
| **Native VLAN** | The VLAN whose traffic travels untagged on a trunk link |
| **Inter-VLAN Routing** | The process of routing traffic between VLANs — requires a Layer 3 device |
| **SVI** | Switched Virtual Interface — a virtual interface on a Layer 3 switch used for inter-VLAN routing |

---

## How a Switch Works

### The CAM Table (MAC Address Table)

When a switch is first powered on, its CAM table is empty. It learns by observing traffic.

```
Step 1 — Frame arrives:
  Source MAC:      AA:BB:CC:DD:EE:01
  Source port:     Fa0/1
  Destination MAC: AA:BB:CC:DD:EE:02

Step 2 — Switch learns:
  "AA:BB:CC:DD:EE:01 is reachable through port Fa0/1"
  Entry added to CAM table.

Step 3 — Switch looks up destination:
  Is AA:BB:CC:DD:EE:02 in the CAM table?
  → Yes → Forward only to that port.
  → No  → Flood the frame to all ports except the source.
```

This flooding behavior when a destination is unknown is called **unicast flooding** — it looks similar to broadcast traffic and is abused by attackers (see MAC Flooding below).

### CAM Table Structure

```
Port    MAC Address          VLAN
Fa0/1   AA:BB:CC:DD:EE:01    10
Fa0/2   AA:BB:CC:DD:EE:02    10
Fa0/3   FF:AA:BB:CC:DD:03    20
Fa0/4   FF:AA:BB:CC:DD:04    20
```

Entries age out after a period of inactivity (default 300 seconds on Cisco switches). This keeps the table clean and prevents stale entries from persisting.

---

## VLANs — Logical Network Segmentation

### Why VLANs Exist

Without VLANs, every device plugged into a switch is in the same broadcast domain. Broadcasts from one device reach every other device. An attacker who compromises one machine can see all traffic, scan all hosts, and pivot freely.

VLANs solve this by creating isolated segments:

```
VLAN 10 — IT Department
VLAN 20 — Finance
VLAN 30 — Guest WiFi
VLAN 99 — Management (isolated, restricted access)
```

Devices in VLAN 10 cannot communicate with devices in VLAN 20 unless traffic is explicitly routed — and routing can be controlled, logged, and filtered.

### Access Ports

An access port belongs to exactly one VLAN. It connects end devices like PCs, printers, and phones. The device itself has no awareness of VLANs — it just sends normal frames.

```
Cisco IOS — Configuring an access port:

interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
```

### Trunk Ports

A trunk port carries traffic for multiple VLANs simultaneously. Used between switches, and between a switch and a router or Layer 3 switch.

To differentiate frames belonging to different VLANs on the same physical link, frames are **tagged** with the VLAN ID using the **802.1Q** standard.

```
Frame leaving VLAN 10 on a trunk:

[ Dest MAC | Src MAC | 802.1Q Tag (VLAN 10) | EtherType | Payload | FCS ]
                          ↑
                     4-byte tag inserted
                     TPID: 0x8100
                     VLAN ID: 10
```

```
Cisco IOS — Configuring a trunk port:

interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 10,20,30
```

### Native VLAN

On a trunk link, one VLAN is designated as the **native VLAN**. Frames belonging to the native VLAN travel **untagged**. By default on Cisco switches, this is VLAN 1.

This is a common misconfiguration risk — see the VLAN Hopping section below.

```
Cisco IOS — Changing the native VLAN:

interface GigabitEthernet0/1
 switchport trunk native vlan 999
```

Best practice: Set the native VLAN to an unused VLAN (like 999) and never assign any devices to it.

---

## VLAN Tagging — 802.1Q In Detail

The 802.1Q tag is inserted into the Ethernet frame between the source MAC address and the EtherType field.

```
Standard Ethernet Frame:
[ Dest MAC (6B) | Src MAC (6B) | EtherType (2B) | Payload | FCS ]

802.1Q Tagged Frame:
[ Dest MAC (6B) | Src MAC (6B) | TPID (2B) | TCI (2B) | EtherType (2B) | Payload | FCS ]
                                    ↑              ↑
                               0x8100           PCP (3 bits) | DEI (1 bit) | VID (12 bits)
                            (marks as 802.1Q)                                    ↑
                                                                         VLAN ID (0–4094)
```

The 12-bit VLAN ID field supports values from 0 to 4094 — giving a theoretical maximum of **4094 usable VLANs** per switch (0 and 4095 are reserved).

---

## Inter-VLAN Routing

VLANs are isolated by design. For traffic to cross between them, it must pass through a **Layer 3 device** — a router or a Layer 3 switch.

### Option 1 — Router on a Stick (ROAS)

A single physical link between the switch and a router, divided into logical sub-interfaces — one per VLAN.

```
                  [ Router ]
                      |
             GigabitEthernet0/0 (trunk)
              /         |        \
         .10            .20       .30
       VLAN 10        VLAN 20   VLAN 30
```

```
Cisco IOS — Configuring Router on a Stick:

interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
```

Devices in VLAN 10 use `192.168.10.1` as their default gateway. Traffic destined for VLAN 20 is sent to the router, which routes it back into the switch on the correct sub-interface.

**Weakness:** All inter-VLAN traffic passes through a single physical link. At scale, this becomes a bottleneck.

### Option 2 — Layer 3 Switch with SVIs

A Layer 3 switch handles routing internally using **Switched Virtual Interfaces (SVIs)** — virtual interfaces assigned to each VLAN.

```
Cisco IOS — Configuring SVIs on a Layer 3 Switch:

ip routing

interface Vlan10
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface Vlan20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
```

No external router needed. Inter-VLAN routing happens at wire speed inside the switch hardware. This is the standard approach in modern enterprise networks.

---

## VLANs and Subnets

VLANs and subnets are different concepts, but they map directly to each other in practice.

| VLAN | Name | Subnet | Gateway |
|------|------|--------|---------|
| VLAN 10 | IT | 192.168.10.0/24 | 192.168.10.1 |
| VLAN 20 | Finance | 192.168.20.0/24 | 192.168.20.1 |
| VLAN 30 | Guest | 192.168.30.0/24 | 192.168.30.1 |
| VLAN 99 | Management | 10.0.99.0/24 | 10.0.99.1 |

Each VLAN is its own broadcast domain and maps to its own subnet. A device in VLAN 10 gets an IP in the `192.168.10.0/24` range. It can only communicate outside that subnet through the gateway — which is the router or SVI.

---

## Switches and VLANs in a Cybersecurity Context

### From an Attacker's Perspective

**MAC Flooding**

The attacker floods the switch with frames containing thousands of fake MAC addresses, filling the CAM table. Once the table is full, the switch can no longer learn new entries and falls back to flooding all frames to all ports — turning the switch into a hub.

```
Attack tool: macof (from dsniff)

macof -i eth0

Generates ~150,000 MAC entries per minute.
Once CAM table overflows, traffic is flooded.
Attacker running Wireshark captures everything.
```

**Mitigation:** Port Security — limit the number of MAC addresses allowed per port.

```
Cisco IOS — Port Security:

interface FastEthernet0/1
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation shutdown
```

---

**VLAN Hopping — Switch Spoofing**

If an access port is left in auto-negotiation mode (the Cisco default), an attacker can send DTP (Dynamic Trunking Protocol) negotiation frames from their machine, causing the port to become a trunk. Once trunked, the attacker sends and receives tagged frames for any VLAN.

```
Normal:
  Attacker PC → Access Port (VLAN 10 only)

After Switch Spoofing:
  Attacker PC → Trunk Port (VLAN 10, 20, 30, 99 — all accessible)
```

```
Attack tool: yersinia

yersinia -G   # graphical interface
# Select DTP → Send DTP Desirable
# Port becomes trunk → attacker accesses all VLANs
```

**Mitigation:** Disable DTP on all access ports. Set ports explicitly.

```
Cisco IOS:

interface FastEthernet0/1
 switchport mode access
 switchport nonegotiate
```

---

**VLAN Hopping — Double Tagging**

This attack exploits the native VLAN. The attacker crafts a frame with **two 802.1Q tags**: the outer tag matching the native VLAN, and the inner tag set to the target VLAN.

```
Crafted frame:

[ Dest MAC | Src MAC | Tag: VLAN 1 (native) | Tag: VLAN 20 | Payload ]
                              ↑                      ↑
                    Stripped by first switch    Passed to VLAN 20
```

When the frame arrives at the first switch, it strips the outer native VLAN tag (as expected for untagged traffic) and forwards the frame on the trunk. The second switch sees the inner tag and delivers the frame to VLAN 20. The attack is **one-directional** — responses cannot return to the attacker — but it is still used for DoS and reconnaissance.

```
Attack tool: scapy

from scapy.all import *

frame = Ether() / Dot1Q(vlan=1) / Dot1Q(vlan=20) / IP(dst="192.168.20.50") / ICMP()
sendp(frame, iface="eth0")
```

**Mitigation:**
- Set native VLAN to an unused VLAN (not VLAN 1)
- Tag native VLAN traffic explicitly
- Never assign user devices to the native VLAN

```
Cisco IOS — Tag native VLAN traffic:

vlan dot1q tag native
```

---

**ARP Spoofing Within a VLAN**

Even inside a VLAN, ARP spoofing allows a man-in-the-middle attack. The attacker broadcasts false ARP replies, associating their MAC address with the gateway's IP. Traffic meant for the gateway is sent to the attacker instead.

```
Normal:
  PC (192.168.10.5) → ARP: "Who is 192.168.10.1?" → Gateway replies with its MAC

After ARP Spoofing:
  Attacker sends: "192.168.10.1 is at AA:BB:CC:DD:EE:FF (attacker's MAC)"
  PC updates its ARP cache — all traffic now goes to attacker
```

```
Attack tool: arpspoof

arpspoof -i eth0 -t 192.168.10.5 192.168.10.1
arpspoof -i eth0 -t 192.168.10.1 192.168.10.5
```

**Mitigation:** Dynamic ARP Inspection (DAI) — the switch validates ARP packets against a trusted DHCP binding table.

```
Cisco IOS:

ip dhcp snooping
ip dhcp snooping vlan 10,20,30

ip arp inspection vlan 10,20,30
```

---

### From a Defender's Perspective

**Network Segmentation**

The primary defensive value of VLANs is containment. If an attacker compromises a guest device on VLAN 30, they are isolated from finance systems on VLAN 20 and management infrastructure on VLAN 99 — assuming inter-VLAN routing rules are enforced.

```
Guest VLAN 30  →  Router  →  ACL: DENY all to VLAN 20, VLAN 99
                           →  PERMIT only to VLAN 30 gateway and internet
```

**Management VLAN Isolation**

All switch and router management interfaces (SSH, SNMP, web console) should sit on a dedicated management VLAN — accessible only from authorized workstations or jump hosts.

```
VLAN 99 — Management
  Access: Only from VLAN 10 (IT) via jump host
  SSH only — disable Telnet
  SNMP v3 — disable SNMPv1/v2
```

---

## Full Flow — Packet Crossing Two VLANs

```
PC-A (VLAN 10, 192.168.10.5) wants to reach PC-B (VLAN 20, 192.168.20.8)

Step 1: PC-A sends frame to its default gateway (192.168.10.1)
        Frame is tagged VLAN 10 on the trunk to the router/L3 switch

Step 2: Router/L3 switch receives frame on VLAN 10 interface
        Strips VLAN 10 tag, inspects IP header
        Routes to 192.168.20.0/24 via VLAN 20 SVI

Step 3: Router/L3 switch re-tags frame with VLAN 20
        Sends it back down the trunk to the switch

Step 4: Switch strips VLAN 20 tag, forwards frame out the access port to PC-B
        PC-B receives a normal Ethernet frame — unaware of VLANs entirely

──────────────────────────────────────────────────────────────────────────
PC-A → [VLAN 10 tag] → Trunk → [Route] → [VLAN 20 tag] → Trunk → PC-B
──────────────────────────────────────────────────────────────────────────
```

---

## Real-World Breaches

### Target (2013)
The breach that introduced the concept of VLAN segmentation failure to mainstream security awareness. Attackers entered Target's network through a **third-party HVAC vendor** with remote access. The vendor's systems were on the same flat network segment as the point-of-sale systems. No VLANs separated them. Attackers moved laterally from the HVAC environment directly to POS devices and installed malware that exfiltrated **40 million credit card numbers**. Had VLANs and inter-VLAN ACLs been in place, the vendor's access would have been contained to their segment with no path to POS infrastructure.

### Uber (2022)
An attacker used social engineering to obtain MFA credentials from an Uber contractor, then accessed the internal VPN. Once inside, the network had insufficient segmentation — the contractor's access level allowed lateral movement to internal tools including PAM (Privileged Access Management), AWS, and Google Workspace admin consoles. Proper VLAN segmentation and zero-trust network access would have limited what was reachable from that initial foothold.

### Florida Water Treatment Facility (2021)
An attacker remotely accessed the SCADA system controlling chemical levels at the Oldsmar water treatment plant and briefly increased sodium hydroxide to dangerous levels. The OT (Operational Technology) network was not properly segmented from the IT network — a Windows 7 machine with remote desktop exposed to the internet served as the entry point. A dedicated OT VLAN with strict ACLs and no direct internet exposure is the baseline control that should have prevented this entirely.

> In all three cases, the presence of switches and the absence of meaningful VLAN segmentation turned a limited compromise into a major incident. The attacker did not break through the segmentation — there was none to break through.

---

## Common Misconfigurations

| Misconfiguration | Risk | Fix |
|-----------------|------|-----|
| VLAN 1 as native VLAN | Double-tagging attack vector | Change native VLAN to unused VLAN (e.g., 999) |
| DTP auto-negotiation on access ports | Switch spoofing / VLAN hopping | `switchport nonegotiate` on all access ports |
| No port security | MAC flooding, rogue devices | Limit MAC addresses per port |
| Flat network, no VLANs | Full lateral movement after any breach | Implement VLAN segmentation per function/department |
| All VLANs allowed on all trunks | Unnecessary exposure | Prune trunks to only carry required VLANs |
| Management VLAN = user VLAN | Management interfaces exposed to users | Separate management into a dedicated VLAN |
| No DAI (Dynamic ARP Inspection) | ARP spoofing / MitM within VLAN | Enable DAI on user VLANs |

---

## Best Practices

- Assign every port explicitly — never leave ports in default VLAN 1
- Disable all unused switch ports and place them in an isolated unused VLAN
- Prune trunk links — only allow VLANs that actually need to traverse each trunk
- Set native VLAN to an unused VLAN and tag all traffic
- Disable DTP (`switchport nonegotiate`) on all non-trunk ports
- Apply inter-VLAN ACLs at the routing layer — VLANs segment, ACLs enforce what can cross
- Place management interfaces in a dedicated VLAN accessible only from authorized hosts
- Enable DHCP Snooping and Dynamic ARP Inspection on all user VLANs
- Document VLAN assignments and review them regularly — unused VLANs become attack surface

---

## How to Practice

| Platform / Tool | What to do |
|----------------|-----------|
| **Cisco Packet Tracer** | Build topologies with multiple VLANs, trunk links, and Router on a Stick — free and browser-based |
| **GNS3** | Emulate real Cisco IOS with full VLAN and inter-VLAN routing configs |
| **TryHackMe** | Rooms on network segmentation, switching, and lateral movement |
| **VirtualBox + pfSense** | Create isolated VM networks per VLAN and enforce routing between them |
| **Wireshark** | Capture trunk traffic and observe 802.1Q tags — filter with `vlan.id == 10` |
| **Scapy** | Craft double-tagged 802.1Q frames to understand the attack at the packet level |
| **macof / yersinia** | Practice MAC flooding and DTP-based VLAN hopping in an isolated lab |

---

## What I Studied

- How switches build and use the CAM table to forward frames
- The difference between access and trunk ports, and when to use each
- How 802.1Q tagging works at the byte level
- VLAN design for network segmentation — isolating departments, guest access, and management
- Inter-VLAN routing via Router on a Stick and Layer 3 SVIs
- Attack techniques: MAC flooding, switch spoofing, double-tagging, ARP spoofing
- Mitigations: port security, DAI, DHCP snooping, DTP disabling, native VLAN hardening
- Real breaches caused by flat networks and missing segmentation

---

## Requirements

- No code required — this is a networking and security concept study note
- Lab practice done via Cisco Packet Tracer and TryHackMe
- Attack techniques practiced in isolated VirtualBox lab environment

---

## Origin

Studied as part of the networking and security curriculum on [TryHackMe](https://tryhackme.com) and [Cisco Networking Academy](https://www.netacad.com).

---

## Author

[pedromarinflach-cyber](https://github.com/pedromarinflach-cyber)
