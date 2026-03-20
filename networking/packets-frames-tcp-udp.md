# Packets, Frames, TCP and UDP

This project explains how data travels through a network, focusing on packets, frames, and the main transport protocols: TCP and UDP.  
The goal is to understand how communication actually works — and why this matters in cybersecurity.

---

## Basic Idea

When data is sent over the internet, it is not transmitted as a single block.  
Instead, it is divided into smaller units that travel through the network and are later reassembled at the destination.

These units are:

- **Packets** — network layer
- **Frames** — data link layer

---

## Packets

A packet is a small unit of data used at the **network layer**.

It contains:

- The actual data (payload)
- Source IP address
- Destination IP address

Packets are responsible for delivering data between devices **across different networks**.

---

## Frames

A frame is used at the **data link layer**, which operates within a local network (same network segment).

It wraps the packet and adds:

- MAC address (physical address of the device)
- Error detection information

In simple terms:

| Unit   | Scope                        |
|--------|------------------------------|
| Packet | Communication between networks |
| Frame  | Communication within a local network |

---

## How Data Travels (Simplified)

1. Data is created (message, request, file, etc.)
2. It is divided into packets
3. Each packet is encapsulated inside a frame
4. Frames are transmitted across the network
5. At the destination, the data is reassembled

---

## TCP vs UDP

These are **transport layer protocols** that define how data is transmitted between devices.

### TCP — Transmission Control Protocol

TCP is **reliable** and **connection-oriented**.

Key characteristics:

- Ensures data arrives correctly and in order
- Reorders packets if necessary
- Retransmits lost packets
- Requires a connection before sending data (Three-Way Handshake)

Common use cases:

- Web browsing (HTTP/HTTPS)
- Authentication systems
- File transfers

### UDP — User Datagram Protocol

UDP is **faster** and **connectionless**.

Key characteristics:

- No guarantee of delivery
- No packet reordering
- No retransmission of lost packets
- Lower latency

Common use cases:

- Online games
- Video/audio streaming
- Voice communication (VoIP)

### Comparison

| Feature        | TCP              | UDP               |
|----------------|------------------|-------------------|
| Reliability    | High             | Low               |
| Speed          | Slower           | Faster            |
| Connection     | Yes              | No                |
| Use cases      | Web, login, files | Games, streaming, VoIP |

---

## Why This Matters in Cybersecurity

Understanding packets and protocols is essential because:

- Attackers can **manipulate or forge packets**
- Many attacks rely on TCP behavior — such as **SYN flood attacks**
- UDP is frequently used in **amplification attacks (DDoS)**
- Packet analysis is a core skill for detecting malicious traffic

### Real-World Examples

- **DDoS via UDP amplification** — attacker sends small requests that generate massive responses toward the target
- **TCP SYN flood** — attacker sends many SYN packets without completing the handshake, exhausting server resources
- **Packet sniffing** — capturing unencrypted traffic to extract sensitive data

---

## Practical Usage

This knowledge is directly applied in:

- Network traffic analysis using tools like **Wireshark**
- Detecting suspicious or anomalous behavior in a network
- Understanding how network-based attacks are executed
- Designing and configuring more secure systems

---

## What I Learned

Studying this topic made it clear that network communication is structured and layered — it is not just about sending data, but about how data is organized, transmitted, and controlled at every step.

This is a fundamental foundation for anyone working in cybersecurity.

---

## Next Steps

- Study the OSI model in depth
- Practice packet analysis with Wireshark
- Learn about common network-based attacks in detail
- Explore tools like `nmap`, `tcpdump`, and Wireshark hands-on

---

*Part of my cybersecurity learning path — [github.com/pedromarinflach-cyber](https://github.com/pedromarinflach-cyber)*
