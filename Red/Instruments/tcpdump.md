---
type: tool
room: red
tool_name: "tcpdump"
version: "v4.99.x"
platform: [Linux, macOS]
phase: [Reconnaissance, Credential Access]
tags: [red/tool, red/sniffing, red/recon, packet-capture]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# tcpdump

> [!quote]
> *"Observation with me is second nature."*
> — *A Study in Scarlet*

**Room:** Red — Instruments | **Phase(s):** `Reconnaissance, Credential Access` | **Platform:** `Linux, macOS`

---

## The Instrument *(What It Is)*

`tcpdump` is the premier command-line packet sniffer and network traffic analyzer for Unix-like systems. Powered by `libpcap`, it enables operators to capture, filter, decode, and store Layer 2 through Layer 7 traffic directly from network interfaces or recorded PCAP files.

- **Author / Origin:** Van Jacobson, Craig Leres, Steven McCanne (LBNL)
- **License:** BSD 3-Clause
- **Install:** `sudo apt install tcpdump`

---

## The Method *(Core Usage)*

### Basic Syntax
```bash
sudo tcpdump [options] [BPF_filter]
```

### Key Flags

| Flag | Meaning |
|------|---------|
| `-i <iface>` | Capture on specified interface (e.g. `eth0`, `any`) |
| `-A` | Print packet payload in ASCII (essential for reading cleartext credentials) |
| `-X` / `-XX` | Print payload in hex and ASCII (including link-layer header with `-XX`) |
| `-w <file>` | Write captured packets raw to a PCAP file |
| `-r <file>` | Read and dissect packets from a saved PCAP file |
| `-n` | Do not resolve IP addresses to domain names |
| `-nn` | Do not resolve host addresses or port numbers |
| `-v` / `-vv` / `-vvv` | Increase verbosity of protocol decoders |
| `-s <len>` | Set snap length in bytes (`-s 0` captures full packet) |
| `-c <count>` | Terminate capture after processing `count` packets |
| `-e` | Print link-level header (MAC addresses) |

---

## The Cases *(Common Use-Patterns)*

### Case 1 — Sniffing Cleartext Credentials
```bash
# Sniff POP3 credentials on port 110 (reveals USER and PASS)
sudo tcpdump -i eth0 port 110 -A

# Sniff Telnet credentials on port 23
sudo tcpdump -i eth0 port 23 -A

# Sniff HTTP POST logins and basic authentication
sudo tcpdump -i eth0 'port 80 and tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x504f5354' -A
```

### Case 2 — Filtering by Target Host and Protocol
```bash
# Capture all traffic to or from 10.10.10.50 omitting DNS resolution
sudo tcpdump -i eth0 host 10.10.10.50 -nn

# Capture ICMP traffic only (ping sweep or covert ICMP exfiltration)
sudo tcpdump -i eth0 icmp
```

### Case 3 — PCAP Recording & Offline Analysis
```bash
# Capture full packets on eth0 and dump directly to disk
sudo tcpdump -i eth0 -s 0 -w /tmp/session_capture.pcap

# Read offline PCAP filtering for DNS queries in ASCII
tcpdump -r /tmp/session_capture.pcap port 53 -A
```

### Case 4 — Stealth & SYN Packet Monitoring
```bash
# Detect incoming SYN packets across all interfaces
sudo tcpdump -i any 'tcp[tcpflags] & (tcp-syn) != 0 and tcp[tcpflags] & (tcp-ack) == 0' -nn
```

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name |
|-------------|----------------|
| `T1040` | Network Sniffing |
| `T1071.001` | Application Layer Protocol: Web Protocols |
| `T1557` | Adversary-in-the-Middle |

---

## Gotchas & Caveats

> [!warning] Common Mistakes
> - **Truncated Packets:** Older versions default to a 68- or 96-byte snap length, which cuts off packet payloads. Always use `-s 0` or modern defaults to retain entire payloads.
> - **Buffer Overflows:** High-speed capture on busy gigabit interfaces without packet filtering drops packets (`packets dropped by kernel`). Use specific BPF filters or buffer tuning (`-B`).
> - **Privileges:** Raw socket creation requires `root` / `sudo` or `CAP_NET_RAW` capabilities.

---

## OPSEC Notes

- **Promiscuous Mode Detection:** Putting an interface into promiscuous mode may broadcast an OS-level kernel log (`device eth0 entered promiscuous mode`) which host-based agents (e.g. OSSEC, auditd) report.
- **Traffic Signatures:** `tcpdump` is purely passive on the network; it transmits zero packets. It leaves no network footprint.

---

## Related Notes

- [[Red/Instruments/wireshark|wireshark / tshark]]
- [[Red/Instruments/bettercap|bettercap]]
- [[Red/Monographs/TA0006 - Credential Access|TA0006 — Credential Access]]
- [[Casebook/Labs/TryHackMe/THM - Protocols and Servers 2|THM — Protocols and Servers 2]]

↩ [[Red/Instruments/readme|Red: Instruments]] · [[Red/Red|Red]] · [[Mind Palace]]
