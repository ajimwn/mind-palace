---
type: tool
room: blue
tool_name: "networkminer"
version: "v2.8"
platform: [Windows, Linux (via Mono)]
phase: [Network Forensics, Artifact Extraction]
tags: [blue/tool, blue/network-forensics, artifact-extraction, credentials, pcap]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# NetworkMiner

> [!quote]
> *"Observation with me is second nature."*
> — *A Study in Scarlet*

**Room:** Blue — Instruments | **Role:** Passive Network Forensic Analysis & Artifact Extraction | **Platform:** `Windows, Linux`

---

## The Instrument *(What It Is)*

`NetworkMiner` is an open-source Network Forensic Analysis Tool (NFAT). Unlike packet sniffers focused on raw packet analysis, NetworkMiner is host-centric: it parses PCAP files to automatically extract transferred files, images, emails, credentials, operating system fingerprints, and DNS sessions without executing any network traffic.

- **Author / Origin:** Erik Hjelmvik (Netresec)
- **License:** GNU GPL v2

---

## The Method *(Core Usage)*

Open a saved PCAP capture in NetworkMiner (`File > Open`) to browse organized tabs:

| Tab | Forensic Value |
|-----|----------------|
| **Hosts** | List of all endpoints, MAC addresses, open ports, and OS fingerprints |
| **Files** | Reconstructed files, documents, and payloads carved from unencrypted streams |
| **Credentials** | Cleartext usernames, passwords, cookies, and tokens (FTP, POP3, HTTP, etc.) |
| **DNS** | Resolving IPs and domain queries |
| **Parameters** | All extracted HTTP GET and POST parameters |
| **Messages** | Transferred emails and chat messages |

---

## D3FEND / MITRE Mapping

| Framework | Reference | Capability |
|---|---|---|
| **D3FEND** | `d3f:FileAnalysis` | Network Artifact Extraction |
| **Counters ATT&CK** | `T1040`, `T1041` | Sniffing, Exfiltration |

---

## Related Notes

- [[Blue/Instruments/wireshark|Wireshark / tshark]]
- [[Blue/Instruments/tcpdump|tcpdump]]

↩ [[Blue/Instruments/readme|Blue: Instruments]] · [[Blue/Blue|Blue]] · [[Mind Palace]]
