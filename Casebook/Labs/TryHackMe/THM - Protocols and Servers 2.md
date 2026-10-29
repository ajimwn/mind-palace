---
type: casefile
case: "Protocols and Servers 2"
platform: "THM"
category: "Network Security"
difficulty: "Easy"
status: solved
date: 2026-10-30
tags: [casebook, thm, network-security, sniffing, mitm, tls, ssh, hydra, credentials]
---

# THM — Protocols and Servers 2

> [!quote]
> *"There is nothing more deceptive than an obvious fact."*
> — *The Boscombe Valley Mystery*

**Platform:** `THM` | **Room:** Protocols and Servers 2 | **Category:** `Network Security` | **Difficulty:** `Easy`
**Focus:** Cleartext Protocol Vulnerabilities · Network Sniffing · Man-in-the-Middle (MITM) · TLS · SSH · Password Attacks with Hydra

---

## The Mystery *(Room Overview)*

This laboratory investigates why cleartext protocols represent a critical risk to network security, how adversaries exploit them via sniffing and Man-in-the-Middle (MITM) positioning, and how cryptographic defences — specifically **Transport Layer Security (TLS)** and **Secure Shell (SSH)** — neutralise these vectors. Finally, it explores password attacks using **THC Hydra** against authentication services.

---

## Phase 1 — Sniffing Attacks (Task 1)

> *"You see, but you do not observe. The distinction is clear."*

### Concept & Mechanics
A **sniffing attack** captures network packets flowing across a transmission medium. When protocols communicate in cleartext, credentials, session tokens, and sensitive messages are exposed in transit.

**Prerequisites for Sniffing:**
- Network interface operating in promiscuous mode (or capturing on host endpoints).
- Root / Administrator privileges.
- Appropriate positioning: network tap, switch port mirroring (SPAN), ARP spoofing, or compromised host on the broadcast domain.

### Packet Sniffing Tools
- `tcpdump`: Lightweight CLI packet capture utility present on virtually all Unix-like platforms.
- `Wireshark`: Full-featured GUI protocol analyzer and packet dissector.
- `tshark`: CLI interface for the Wireshark dissection engine, ideal for scripting.
- `tcpflow` / `ngrep`: TCP stream reassembly and packet-level regex matching.

### Practical Capture — Cleartext Credentials over POP3
Target service: POP3 on TCP port 110.

```bash
# Capture POP3 traffic on interface, displaying packet contents in ASCII
sudo tcpdump port 110 -A
```

**Intercepted Packet Stream:**
```text
USER frank
+OK Password required.
PASS D2xc9CgD
```
*Result:* Cleartext username `frank` and password `D2xc9CgD` recovered directly from the wire.

### Key Tcpdump Commands & BPF Filters
```bash
# Capture Telnet traffic (Port 23)
sudo tcpdump port 23 -A

# Capture HTTP authentication or POST payloads (Port 80)
sudo tcpdump port 80 -A

# Capture FTP authentication traffic (Port 21)
sudo tcpdump port 21 -A

# Filter traffic to/from a specific target IP
sudo tcpdump host 10.20.30.148 -A

# Save capture to PCAP file for offline analysis
sudo tcpdump -i eth0 -w capture.pcap port 110 or port 23

# Read and inspect PCAP in ASCII
tcpdump -r capture.pcap -A
```

### Questions & Answers
> **Q: What do you need to add to the command `sudo tcpdump` to capture only Telnet traffic?**
> `port 23`

> **Q: What is the simplest display filter you can use with Wireshark to show only IMAP traffic?**
> `imap`

---

## Phase 2 — Man-in-the-Middle (MITM) Attacks (Task 2)

> *"When a client and server speak in the dark, the adversary provides the candle."*

### Attack Mechanics
In a MITM attack, the adversary positions themselves between two communicating parties (A and B), intercepting and optionally altering communications without either party's knowledge:

```text
Victim (A) ──────────► Attacker (E) ──────────► Server (B)
            "Auth Req"              "Auth Req"
```

### Positioning Techniques
| Vector | Scope | Mechanism |
|---|---|---|
| **ARP Cache Poisoning** | Local Layer 2 | Gratuitous ARP replies map target IP to attacker's MAC address |
| **DNS Spoofing** | Layer 3 / UDP 53 | Forged DNS responses resolve target domains to attacker-controlled IPs |
| **Rogue Access Points** | 802.11 Wireless | Evil Twin APs broadcast legitimate SSIDs to attract client connections |
| **BGP Hijacking** | WAN / Internet | Malicious Autonomous System announces false routing prefixes |

### MITM Frameworks
- **Bettercap:** Modern, actively maintained MITM Swiss-army knife supporting ARP/DNS spoofing, packet manipulation, and HTTP proxying. Can be invoked in **3 modes**:
  1. Interactive mode
  2. Eval mode (`-eval`)
  3. Caplet mode (automated script files)
- **Ettercap:** Legacy comprehensive suite for LAN MITM. Offers **3 interface modes**:
  1. Text-only (`-T`)
  2. Ncurses Curses GUI (`-C`)
  3. GTK Graphical GUI (`-G`)
- **mitmproxy:** Python-based SSL-capable HTTP/HTTPS interception proxy.

### Encrypted Traffic Interception Techniques
1. **SSL Stripping:** Downgrading HTTPS requests to HTTP transparently before the victim establishes TLS.
2. **Untrusted / Rogue CA Certificates:** Generating on-the-fly fake certificates using an attacker-installed or compromised root CA.

### Defences
- **HSTS (HTTP Strict Transport Security):** Forces browsers to communicate exclusively over HTTPS; preloaded lists prevent initial bootstrap stripping.
- **Certificate Transparency (CT):** Public, tamper-evident audit logs of issued TLS certificates.
- **Certificate Pinning:** Hardcoding expected public keys or certificates within client applications.

### Questions & Answers
> **Q: How many different interfaces does Ettercap offer?**
> `3`

> **Q: In how many ways can you invoke Bettercap?**
> `3`

---

## Phase 3 — Transport Layer Security (TLS) (Task 3)

> *"The seal on the envelope guarantees both confidentiality and authorial authenticity."*

### Architecture & Positioning
TLS operates at the Presentation / Session boundary, encapsulating application protocols within an encrypted and authenticated tunnel over TCP.

```text
┌──────────────────────────────────────┐
│ Application Layer (HTTP, FTP, SMTP)  │
├──────────────────────────────────────┤
│ ▶ TLS Layer (Crypto & Integrity)     │
├──────────────────────────────────────┤
│ Transport Layer (TCP)                │
└──────────────────────────────────────┘
```

### Protocol Port Upgrades
| Protocol | Cleartext Port | Encrypted Protocol | TLS Port |
|---|:---:|---|:---:|
| HTTP | 80 | HTTPS | 443 |
| FTP | 21 | FTPS (Implicit) | 990 |
| SMTP | 25 | SMTPS | 465 |
| POP3 | 110 | POP3S | 995 |
| IMAP | 143 | IMAPS | 993 |
| DNS | 53 | DoT (DNS over TLS) | 853 |
| DNS | 53 | DoH (DNS over HTTPS) | 443 |

### Implicit TLS vs STARTTLS
- **Implicit TLS:** Connection establishes encryption immediately upon connection to a designated port (e.g. 443, 993, 853). Eliminates downgrade attack surface.
- **STARTTLS:** Initiates an unencrypted session and requests opportunistic upgrade to TLS. Susceptible to plaintext injection or stripping attacks if strict enforcement is missing.

### Questions & Answers
> **Q: DNS can also be secured using TLS. What is the three-letter acronym of the DNS protocol that uses TLS?**
> `DoT`

---

## Phase 4 — Secure Shell (SSH) Operations (Task 4)

> *"The standard for secure remote system administration."*

SSH (TCP 22) provides secure channel establishment with three cryptographic guarantees:
1. **Authentication:** Proves server identity via host keys and client identity via keys or passwords.
2. **Confidentiality:** Symmetrical encryption of all session data.
3. **Integrity:** Message Authentication Codes (MACs) ensure undetected transmission tampering is impossible.

### Practical Investigation
Connecting to target machine `10.48.147.45` as user `mark` with password `XBtc49AB`:

```bash
# Connect and extract kernel release directly
ssh mark@10.48.147.45 "uname -r"
```
**Output:**
```text
5.15.0-119-generic
```

### Secure File Transfer via SCP
```bash
# Download book.txt from the remote target to local workstation
scp mark@10.48.147.45:/home/mark/book.txt ./book.txt
```
**Output observed:**
```text
book.txt                 100%  415KB 415.0KB/s   00:00
```
*Result:* Transfer size reported: **415** KB.

### SSH Key Pairs & Cryptographic Hygiene
```bash
# Generate modern, robust Ed25519 key pair
ssh-keygen -t ed25519 -C "admin@lab"

# Copy public key to target for passwordless authentication
ssh-copy-id -i ~/.ssh/id_ed25519.pub mark@10.48.147.45

# Hardening /etc/ssh/sshd_config:
PasswordAuthentication no
PermitRootLogin no
```

### Questions & Answers
> **Q: Use SSH to connect to 10.48.147.45 as mark with the password XBtc49AB. Using `uname -r`, find the Kernel release?**
> `5.15.0-119-generic`

> **Q: Use SSH to download the file `book.txt` from the remote system. How many KBs did scp display as download size?**
> `415`

---

## Phase 5 — Password Attacks with Hydra (Task 5)

> *"When the lock is secure, the adversary attacks the key."*

### Authentication Concepts
Authentication relies on three classic factors:
1. **Something you know:** Passwords, passphrases, PINs.
2. **Something you have:** Smart cards, TOTP hardware tokens, physical security keys (FIDO2).
3. **Something you are:** Biometrics (fingerprints, facial recognition).

### Attack Methodologies Against Passwords
- **Password Guessing / OSINT:** Targeted guesses using personal or contextual knowledge.
- **Dictionary Attacks:** Iterating through wordlists of known words and breach corpuses (e.g., `rockyou.txt`).
- **Brute Force:** Exhaustive mathematical permutation across a defined character set.
- **Password Spraying:** Submitting 1–2 common passwords against large numbers of accounts to avoid threshold lockouts.
- **Credential Stuffing:** Replaying leaked username:password combinations from external breaches across target services.

### THC Hydra Execution
Target: IMAP service running on `10.48.147.45`, username `lazie`, wordlist `/usr/share/wordlists/rockyou.txt`.

```bash
# Attack IMAP service for account 'lazie' using rockyou
hydra -l lazie -P /usr/share/wordlists/rockyou.txt 10.48.147.45 imap
```

**Hydra Console Output:**
```text
[DATA] attacking imap://10.48.147.45:143/
[143][imap] host: 10.48.147.45   login: lazie   password: butterfly
1 of 1 target successfully completed, 1 valid password found
```

### Questions & Answers
> **Q: We learned that one of the email accounts is lazie. What is the password used to access the IMAP service on 10.48.147.45?**
> `butterfly`

---

## The Attack Chain & MITRE ATT&CK Mapping

```text
Network Reconnaissance ──► Cleartext Sniffing / MITM ──► Credential Harvesting
        │                                                       │
        └─────────────────► Online Brute-Force (Hydra) ─────────┴──► Remote Access (SSH)
```

| Tactic | Technique ID | Technique Name | Context in Lab |
|---|---|---|---|
| **Collection / Credential Access** | `T1040` | Network Sniffing | Cleartext POP3/Telnet credential capture via tcpdump |
| **Credential Access** | `T1557` | Adversary-in-the-Middle | ARP/DNS poisoning concept via Bettercap & Ettercap |
| **Credential Access** | `T1110.001` | Password Guessing / Brute Force | Service dictionary attack using Hydra against IMAP |
| **Lateral Movement / Exec** | `T1021.004` | Remote Services: SSH | Authenticated administration & remote shell execution |

---

## Blue Team Countermeasures

| Attack Vector | Defensive Controls | Implementation |
|---|---|---|
| **Packet Sniffing** | Enforce End-to-End Encryption | Retire cleartext protocols (Telnet, POP3, IMAP, FTP, HTTP) in favour of SSH, POP3S, IMAPS, FTPS, HTTPS. |
| **ARP Poisoning** | Dynamic ARP Inspection (DAI) & 802.1X | Enable DHCP snooping and DAI on switches; deploy 802.1X port security. |
| **MITM / SSL Stripping** | HSTS & Certificate Pinning | Enable `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`. |
| **Brute Force / Hydra** | Rate Limiting & Account Lockout | Implement `fail2ban` for SSH/IMAP, exponential backoff, and MFA enforcement. |
| **Weak Credentials** | NIST SP 800-63B Compliance | Mandate length (≥12 chars), check against haveibeenpwned breach lists, eliminate arbitrary rotation. |

---

## Instruments Mastered

- [[Red/Instruments/readme#🦈 tcpdump|tcpdump]] — Packet capture, BPF filters, ASCII payload inspection.
- [[Red/Instruments/readme#🦈 Wireshark / tshark|Wireshark / tshark]] — Protocol dissection, IMAP/POP3 display filters.
- [[Red/Instruments/readme#🔑 hydra|THC Hydra]] — Multi-threaded service authentication brute-forcing.
- [[Red/Instruments/readme#🔐 ssh / scp / sftp|SSH & SCP]] — Secure command execution, host key verification, remote file retrieval.
- [[Red/Instruments/readme#⚡ bettercap / ettercap|Bettercap & Ettercap]] — Layer 2 MITM mechanics and interface configurations.

---

## References

- Room: [THM — Protocols and Servers 2](https://tryhackme.com/room/protocolsandservers2)
- [[Casebook/Casebook]] · [[Casebook/Labs/readme|Labs & Walkthroughs]]
- Related ATT&CK Monographs: [[Red/Monographs/TA0006 - Credential Access|TA0006 - Credential Access]], [[Red/Monographs/TA0043 - Reconnaissance|TA0043 - Reconnaissance]]
