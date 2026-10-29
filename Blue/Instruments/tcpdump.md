---
type: tool
room: blue
tool_name: "tcpdump"
version: "v4.99.x"
platform: [Linux]
phase: [Detection, Network Forensics]
tags: [blue/tool, blue/network-forensics, packet-capture]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# tcpdump (Defensive Capture)

> [!quote]
> *"It has long been an axiom of mine that the little things are infinitely the most important."*
> — *A Case of Identity*

**Room:** Blue — Instruments | **Role:** Live Network Traffic Capture & Headless Recording | **Platform:** `Linux`

---

## The Instrument *(What It Is)*

In defensive incident response, `tcpdump` is the primary command-line tool used on servers, perimeter gateways, and monitoring taps to record live network evidence, capture suspicious lateral connections, or implement rotating circular PCAP buffers during active incidents.

- **Author / Origin:** Van Jacobson, Craig Leres, Steven McCanne
- **License:** BSD 3-Clause
- **Install:** `sudo apt install tcpdump`

---

## The Method *(Core Usage)*

### Rotating Circular Capture Buffer
To record live traffic without running out of disk space during an investigation:

```bash
# Capture full packets on eth0 into rotating 50MB files, retaining max 10 files
sudo tcpdump -i eth0 -s 0 -W 10 -C 50 -w /var/log/incident_%Y%m%d_%H%M%S.pcap
```

### Essential Triage Flags

| Flag | Meaning |
|------|---------|
| `-i <iface>` | Capture interface |
| `-nn` | Do not resolve IP addresses or port names (essential for speed) |
| `-s 0` | Snap length 0 (capture complete packet payload) |
| `-C <MB>` | File size limit before rotating to next file |
| `-W <num>` | Maximum number of rotating files to keep |
| `-G <sec>` | Rotate file every N seconds |
| `-r <file>` | Read captured PCAP for quick terminal triage |

---

## The Cases *(Common Use-Patterns)*

### Case 1 — Triage Specific Suspicious Connection
```bash
# Inspect traffic on non-standard port 4444 with ASCII decoding
sudo tcpdump -i eth0 -nnvA "tcp port 4444"

# Capture all traffic excluding management SSH traffic
sudo tcpdump -i eth0 -nn "not port 22" -w /tmp/filtered_traffic.pcap
```

### Case 2 — Detecting Lateral Movement / SMB Traffic
```bash
# Monitor internal SMB/RPC activity from a compromised host
sudo tcpdump -i eth0 -nn "host 10.10.10.50 and (port 445 or port 139 or port 135)"
```

---

## D3FEND / MITRE Mapping

| Framework | Reference | Capability |
|---|---|---|
| **D3FEND** | `d3f:NetworkTrafficAnalysis` | Network Traffic Analysis |
| **Counters ATT&CK** | `T1071`, `T1021`, `T1048` | Command and Control, Lateral Movement |

---

## Related Notes

- [[Blue/Instruments/wireshark|Wireshark / tshark]]
- [[Blue/Instruments/networkminer|NetworkMiner]]

↩ [[Blue/Instruments/readme|Blue: Instruments]] · [[Blue/Blue|Blue]] · [[Mind Palace]]
