---
type: tool
room: red
tool_name: "wireshark"
version: "v4.x"
platform: [Linux, Windows, macOS]
phase: [Reconnaissance, Credential Access]
tags: [red/tool, red/sniffing, packet-analysis, pcap]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# wireshark / tshark

> [!quote]
> *"There is nothing more deceptive than an obvious fact."*
> — *The Boscombe Valley Mystery*

**Room:** Red — Instruments | **Phase(s):** `Reconnaissance, Credential Access` | **Platform:** `Linux, Windows, macOS`

---

## The Instrument *(What It Is)*

Wireshark is the world's foremost graphical network protocol analyzer, and `tshark` is its native terminal-based counterpart. Together, they provide deep packet inspection across thousands of protocols, reassembly of TCP streams, automated file carving, and display filtering essential for both offensive credential harvesting and post-exploitation traffic analysis.

- **Author / Origin:** Gerald Combs & the Wireshark Developer Community
- **License:** GNU GPL v2
- **Install:** `sudo apt install wireshark tshark`

---

## The Method *(Core Usage)*

### Basic Syntax
```bash
# GUI Wireshark
wireshark [capture_file.pcap]

# CLI tshark
tshark -i <interface> [options]
tshark -r <capture_file.pcap> -Y <display_filter> [options]
```

### Key Flags (`tshark`)

| Flag | Meaning |
|------|---------|
| `-i <iface>` | Capture interface |
| `-r <file>` | Read input PCAP file |
| `-Y <filter>` | Apply Wireshark display filter (evaluated post-dissection) |
| `-f <filter>` | Apply BPF capture filter (evaluated at socket level) |
| `-T fields` | Output selected fields rather than raw text packet lines |
| `-e <field>` | Field to extract (e.g. `-e ip.src -e http.host`) |
| `-w <file>` | Write captured packets to PCAP |
| `-q` | Quiet mode (useful when aggregating statistics) |
| `-z <stats>` | Calculate protocol statistics, endpoints, or conversations |

---

## The Cases *(Common Use-Patterns)*

### Case 1 — Live Credential Extraction via `tshark`
```bash
# Capture POP3 traffic live and dump decoded text
sudo tshark -i eth0 -f "port 110" -Y "pop" -T fields -e text

# Extract FTP usernames and passwords from offline capture
tshark -r capture.pcap -Y "ftp.request.command == USER or ftp.request.command == PASS" -T fields -e ftp.request.arg
```

### Case 2 — Web Credential & Cookie Harvesting
```bash
# Extract HTTP POST bodies containing submitted logins
tshark -r capture.pcap -Y "http.request.method == POST" -T fields -e ip.src -e http.host -e http.file_data

# Extract cookie strings and Authorization headers
tshark -r capture.pcap -Y "http.cookie or http.authorization" -T fields -e http.host -e http.cookie -e http.authorization
```

### Case 3 — DNS Exfiltration & Discovery
```bash
# Dump all queried DNS hostnames uniquely
tshark -r capture.pcap -Y "dns.flags.response == 0" -T fields -e dns.qry.name | sort -u
```

### Case 4 — Wireshark GUI Display Filters
Apply these directly in the Wireshark filter bar:
- `pop or imap` — Cleartext email authentication and messages
- `telnet` — Unencrypted interactive terminal sessions
- `http.authbasic` — Decodes Base64 HTTP basic authentication credentials
- `tcp.flags.syn == 1 and tcp.flags.ack == 0` — Initial connection requests (port scan detection)
- `data-text-lines contains "password"` — Raw ASCII search across reassembled data

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name |
|-------------|----------------|
| `T1040` | Network Sniffing |
| `T1071` | Application Layer Protocol |
| `T1557` | Adversary-in-the-Middle |

---

## Gotchas & Caveats

> [!warning] Common Mistakes
> - **Display Filter (`-Y`) vs Capture Filter (`-f`):** Capture filters use BPF syntax (e.g. `tcp port 80`), whereas display filters use Wireshark syntax (e.g. `http.request.method == "POST"`). Applying a display filter to `-f` will error out.
> - **Permissions:** Running GUI Wireshark as `root` is dangerous and discouraged. Add your user to the `wireshark` group (`sudo usermod -aG wireshark $USER`) or use dumpcap.

---

## OPSEC Notes

- Purely passive when sniffing. Emits no packets on the network interface.
- Large captures consume significant memory if kept in Wireshark RAM; use `tshark` stream processing for multi-gigabyte captures.

---

## Related Notes

- [[Red/Instruments/tcpdump|tcpdump]]
- [[Red/Instruments/bettercap|bettercap]]
- [[Blue/Instruments/readme|Blue: Instruments — Wireshark Filters]]
- [[Casebook/Labs/TryHackMe/THM - Protocols and Servers 2|THM — Protocols and Servers 2]]

↩ [[Red/Instruments/readme|Red: Instruments]] · [[Red/Red|Red]] · [[Mind Palace]]
