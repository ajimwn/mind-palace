---
type: tool
room: blue
tool_name: "wireshark"
version: "v4.x"
platform: [Linux, Windows, macOS]
phase: [Detection, Investigation, Network Forensics]
tags: [blue/tool, blue/network-forensics, packet-analysis, pcap]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# Wireshark / tshark (Defensive Analysis)

> [!quote]
> *"You see, but you do not observe. The distinction is clear."*
> — *A Scandal in Bohemia*

**Room:** Blue — Instruments | **Role:** Packet Analysis & Network Forensics | **Platform:** `Linux, Windows, macOS`

---

## The Instrument *(What It Is)*

Wireshark and its CLI interface `tshark` are the standard instruments for dissecting network packet captures (PCAP). For defensive operations, they allow incident responders to identify beaconing behavior, reconstruct unencrypted data exfiltration, isolate malicious DNS requests, trace lateral movement over SMB, and decrypt TLS sessions using pre-master secrets.

- **Author / Origin:** Gerald Combs & the Wireshark Community
- **License:** GNU GPL v2
- **Install:** `sudo apt install wireshark tshark`

---

## The Method *(Core Usage)*

### Essential Display Filters

```wireshark
# === BASIC NETWORK FILTERS ===
ip.addr == 10.10.10.50          # Traffic to or from specific host
ip.src == 10.10.10.50           # Source host only
ip.dst == 10.10.10.50           # Destination host only
ip.addr == 192.168.1.0/24       # Subnet filter

# === SUSPICIOUS PROTOCOLS & EXPLOITATION ===
http.request.method == "POST"   # POST requests (potential webshell or exfil)
http.host contains "malicious"  # Suspicious domain in Host header
dns.flags.rcode == 3            # NXDOMAIN errors (DGA malware detection)
dns.qry.type == 16              # TXT records (DNS tunnelling / exfil)
arp.duplicate-address-frame     # ARP poisoning / duplicate IP detection
icmp.type == 8 && data.len > 64 # ICMP echo with oversized payload (ICMP tunnel)

# === CLEARTEXT CREDENTIAL EXPOSURE ===
ftp.request.command == "PASS"   # FTP password in cleartext
pop.request.command == "PASS"   # POP3 password in cleartext
imap.request contains "LOGIN"   # IMAP authentication
http.authbasic                  # HTTP Basic Auth credentials

# === BEACONING & C2 ===
tcp.flags.syn == 1 && tcp.flags.ack == 0  # Initial connection attempts
frame.len > 1400                # Large frames (exfiltration)
```

---

## The Cases *(Common Investigative Scenarios)*

### Case 1 — Command Line Stream Extraction (`tshark`)
```bash
# Follow TCP stream #0 in ASCII
tshark -r incident.pcap -q -z "follow,tcp,ascii,0"

# Extract all unique DNS queries from an infected host
tshark -r incident.pcap -Y "dns.flags.response == 0 and ip.src == 10.10.10.25" \
  -T fields -e dns.qry.name | sort -u
```

### Case 2 — Detecting Long DNS Labels (DNS Data Exfiltration)
```bash
# Identify anomalous high-entropy or long DNS query names (>50 chars)
tshark -r incident.pcap -Y "dns" -T fields -e dns.qry.name \
  | awk '{ if (length($1) > 50) print length($1), $1 }' | sort -rn
```

### Case 3 — Extracting Carved Files Transferred over HTTP
```bash
# Export all transmitted files from HTTP responses into /tmp/exported
tshark -r incident.pcap -Y "http.response" --export-objects "http,/tmp/exported"
```

---

## D3FEND / MITRE Mapping

| Framework | Reference | Capability |
|---|---|---|
| **D3FEND** | `d3f:NetworkTrafficAnalysis` | Network Traffic Analysis |
| **Counters ATT&CK** | `T1041`, `T1071`, `T1048` | Exfiltration Over C2, DNS Tunneling |

---

## Related Notes

- [[Blue/Instruments/tcpdump|tcpdump]]
- [[Blue/Instruments/networkminer|NetworkMiner]]
- [[Blue/Monographs/IR - Detect and Analyse|IR — Detect and Analyse]]

↩ [[Blue/Instruments/readme|Blue: Instruments]] · [[Blue/Blue|Blue]] · [[Mind Palace]]
