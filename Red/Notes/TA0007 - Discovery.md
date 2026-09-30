---
type: tactic-hub
room: red
tactic: "Discovery"
tactic_id: "TA0007"
tags: [red/discovery, mitre/TA0007]
date_updated: {{date}}
---

# TA0007 — Discovery

> [!quote]
> *"To a great mind, nothing is little."*
> — *A Study in Scarlet*

**MITRE Reference:** [TA0007](https://attack.mitre.org/tactics/TA0007/) | **Kill Chain Phase:** Post-exploitation

The adversary is trying to understand the environment they are inside. After initial access, the adversary is blind — they know nothing about the network topology, the user accounts, the running services, or the defences. Discovery is Holmes cataloguing a new crime scene: systematic, methodical, leaving nothing unexamined.

---

## Techniques in This Tactic

| Technique ID | Name | Platform |
|-------------|------|---------|
| [[T1087 - Account Discovery]] | Account Discovery | All |
| [[T1010 - Application Window Discovery]] | Application Window Discovery | Windows |
| [[T1217 - Browser Information Discovery]] | Browser Bookmarks / History | All |
| [[T1482 - Domain Trust Discovery]] | Domain Trust Discovery | Windows/AD |
| [[T1083 - File and Directory Discovery]] | File & Directory Discovery | All |
| [[T1046 - Network Service Discovery]] | Network Service Scanning | All |
| [[T1135 - Network Share Discovery]] | Network Share Discovery | Windows |
| [[T1040 - Network Sniffing]] | Network Sniffing | All |
| [[T1201 - Password Policy Discovery]] | Password Policy Discovery | All |
| [[T1057 - Process Discovery]] | Process Discovery | All |
| [[T1018 - Remote System Discovery]] | Remote System Discovery | All |
| [[T1082 - System Information Discovery]] | System Information Discovery | All |
| [[T1016 - System Network Configuration Discovery]] | Network Config Discovery | All |
| [[T1033 - System Owner User Discovery]] | User/Owner Discovery | All |
| [[T1007 - System Service Discovery]] | System Service Discovery | All |

---

## Standard Discovery Workflows

### Situational Awareness (First 5 Minutes)

```bash
# ===== LINUX =====
id && whoami && hostname
uname -a                        # OS + kernel
cat /etc/os-release
ifconfig || ip a                # Network interfaces
ip route                        # Routes
cat /etc/hosts                  # Host file entries
env                             # Environment variables (credentials!)
cat /proc/version
df -h                           # Disk mounts
ps auxf                         # Running processes

# ===== WINDOWS (CMD) =====
whoami /all
systeminfo
ipconfig /all
net user
net localgroup administrators
net group "Domain Admins" /domain
net view
arp -a
route print
netstat -anob
tasklist /svc
```

### Active Directory Discovery (T1087.002 / T1482)
```powershell
# Domain info
[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
nltest /domain_trusts
net group "Domain Admins" /domain
net group "Enterprise Admins" /domain

# PowerView (import first: . .\PowerView.ps1)
Get-Domain
Get-DomainController
Get-DomainUser | Select-Object name, samaccountname, description
Get-DomainUser -SPN                     # Kerberoastable accounts
Get-DomainUser -PreauthNotRequired      # ASREPRoastable accounts
Get-DomainGroup -AdminCount 1          # High-privilege groups
Get-DomainComputer | Select-Object dnshostname, operatingsystem
Get-DomainGPO | Select-Object displayname, gpcfilesyspath
Get-DomainTrust
Find-LocalAdminAccess                   # Hosts where current user is local admin
```

### Network Discovery (T1046 / T1018)
```bash
# Internal subnet sweep
nmap -sn <internal_cidr> -oG - | grep Up | awk '{print $2}' > live-hosts.txt

# Port scan live hosts
nmap -sT -p 22,80,135,139,443,445,3389,5985 <internal_cidr> -oA internal-scan

# SMB shares (T1135)
nxc smb live-hosts.txt -u <user> -p <pass> --shares
smbclient -L //<target>/ -U <user>%<pass>

# ARP table (hosts on same subnet, no scan needed)
arp -a
ip neigh show
```

### File & Credential Hunting (T1083 / T1552)
```bash
# Linux — find interesting files
find / -name "*.conf" -o -name "*.config" -o -name "*.env" -o -name "id_rsa" -o -name "*.pem" 2>/dev/null
find / -name "*.kdbx" -o -name "*.db" -o -name "*.sqlite" 2>/dev/null

# Windows — search for credentials
dir /s /b C:\*.config C:\*.xml C:\*.txt C:\*.ini 2>nul | findstr /i "pass cred secret key"
findstr /si "password" C:\*.txt C:\*.xml C:\*.ini C:\*.config

# Windows — look in common locations
type C:\Windows\Panther\unattend.xml
type C:\inetpub\wwwroot\web.config
type "C:\Program Files\<app>\settings.conf"
```

### Process & Service Discovery
```powershell
# Windows — interesting processes
Get-Process | Where-Object { $_.Name -notmatch "svchost|lsass|winlogon|System|smss" } | Select-Object Name, Id, Path

# Windows — services with non-standard paths
Get-WmiObject win32_service | Where-Object { $_.PathName -notlike "C:\Windows\*" } | Select-Object Name, PathName, State

# Linux — listening services
ss -tulpn
cat /etc/services
systemctl list-units --type=service --state=running
```

---

## OPSEC Considerations

> [!warning] Discovery Is Noisy
> - `Find-LocalAdminAccess` (PowerView) sends auth attempts to every host — generates many Event 4625s.
> - Nmap from inside the network is loud; prefer `netstat` and `arp` for stealthy enumeration first.
> - LDAP queries to AD are logged if AD auditing is enabled (Event 1644).
> - Avoid `net group "Domain Admins" /domain` repeatedly — it generates LDAP traffic.

---

## What Blue Sees

| Activity | Log Source | Event / Indicator |
|----------|-----------|-------------------|
| LDAP enumeration | DC (1644) | High-volume LDAP queries |
| Local admin check | Security (4625) | Failed auth to many hosts |
| Net commands | Security (4688) | `net.exe` with group/user/view flags |
| Nmap internal | Network | SYN sweeps from internal host |
| PowerView | Sysmon (1) | `powershell.exe` with LDAP module calls |

---

## Related Notes

- [[TA0004 - Privilege Escalation]] ← Discovery informs this
- [[TA0006 - Credential Access]] ← Discovery finds the credentials
- [[TA0008 - Lateral Movement]] ← Discovery maps the routes

↩ [[Red - Notes]] · [[Red]] · [[Mind Palace]]
