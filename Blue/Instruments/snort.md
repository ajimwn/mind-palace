---
type: tool
room: blue
tool_name: "snort"
version: "Snort 2 / 3"
platform: [Linux]
phase: [Detection, Network Intrusion Detection]
tags: [blue/tool, blue/nids, snort, rules, signature-detection]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# Snort (NIDS)

> [!quote]
> *"To a great mind, nothing is little."*
> — *A Study in Scarlet*

**Room:** Blue — Instruments | **Role:** Network Intrusion Detection & Prevention System (NIDS/NIPS) | **Platform:** `Linux`

---

## The Instrument *(What It Is)*

`Snort` is an open-source Network Intrusion Detection System (NIDS) and packet logger. It monitors network interfaces in real time, matching packet headers and payloads against rule sets to alert on port scans, exploit attempts, malware command and control traffic, and unauthorized cleartext protocols.

- **Author / Origin:** Martin Roesch / Cisco Talos
- **License:** GNU GPL v2
- **Install:** `sudo apt install snort`

---

## The Method *(Core Usage)*

### Execution Modes
```bash
# 1. Packet Sniffer mode
sudo snort -v -i eth0

# 2. Packet Logger mode (record to directory)
sudo snort -dev -l /var/log/snort/ -i eth0

# 3. NIDS Mode with rules configuration
sudo snort -A console -q -c /etc/snort/snort.conf -i eth0

# 4. Test rules against a recorded PCAP file
sudo snort -A console -q -c /etc/snort/snort.conf -r incident.pcap
```

### Anatomy of a Snort Rule
```snort
# Action Protocol Source_IP Source_Port Direction Dest_IP Dest_Port (Options)
alert tcp any any -> 10.10.10.0/24 22 (msg:"SSH Brute Force Attempt"; flags:S; threshold:type threshold, track by_src, count 5, seconds 60; sid:1000001; rev:1;)
```

---

## D3FEND / MITRE Mapping

| Framework | Reference | Capability |
|---|---|---|
| **D3FEND** | `d3f:NetworkTrafficFiltering` | Network Signature Analysis |
| **Counters ATT&CK** | `T1110`, `T1040`, `T1071` | Brute Force, Sniffing, C2 |

---

## Related Notes

- [[Blue/Monographs/Detection - Snort IDS|Detection — Snort IDS]]
- [[Blue/Instruments/tcpdump|tcpdump]]
- [[Blue/Instruments/wireshark|Wireshark / tshark]]

↩ [[Blue/Instruments/readme|Blue: Instruments]] · [[Blue/Blue|Blue]] · [[Mind Palace]]
