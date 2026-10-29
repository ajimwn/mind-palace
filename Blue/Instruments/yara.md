---
type: tool
room: blue
tool_name: "yara"
version: "v4.3.x"
platform: [Linux, Windows, macOS]
phase: [Detection, Malware Analysis]
tags: [blue/tool, blue/malware, yara-rules, signature-matching]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# YARA

> [!quote]
> *"There is nothing more deceptive than an obvious fact."*
> — *The Boscombe Valley Mystery*

**Room:** Blue — Instruments | **Role:** Pattern Matching Swiss Army Knife for Malware Researchers | **Platform:** `Linux, Windows, macOS`

---

## The Instrument *(What It Is)*

`YARA` is a multi-platform tool aimed at helping malware researchers identify and classify malware samples. With rule-based descriptions composed of strings (text, hex, regex) and boolean conditions, YARA scans files, directories, and running process memory to find matching threat indicators.

- **Author / Origin:** Victor Alvarez (VirusTotal)
- **License:** BSD 3-Clause
- **Install:** `sudo apt install yara`

---

## The Method *(Core Usage)*

### Scanning Commands
```bash
# Scan a single file with a rule
yara /path/to/rules.yar suspicious.exe

# Scan a directory recursively
yara -r /path/to/rules/ /path/to/scan/

# Scan running process memory by PID
sudo yara /path/to/rules.yar <PID>

# Compile rules for speed before scanning
yarac rules.yar compiled.rules
yara -C compiled.rules /path/to/scan/
```

### Anatomy of a YARA Rule
```yara
rule Suspicious_Encoded_PowerShell {
    meta:
        description = "Detects base64 encoded PowerShell command execution"
        author = "Blue Team"
        date = "2026-10-30"
        reference = "T1059.001"
    strings:
        $enc1 = "-EncodedCommand" nocase
        $enc2 = "-enc " nocase
        $func1 = "FromBase64String" nocase
        $func2 = "IEX" nocase
        $magic = { 4D 5A } // MZ Header
    condition:
        $magic at 0 and ($enc1 or $enc2) and ($func1 or $func2)
}
```

---

## D3FEND / MITRE Mapping

| Framework | Reference | Capability |
|---|---|---|
| **D3FEND** | `d3f:FileAnalysis` | File Pattern Matching |
| **Counters ATT&CK** | `T1027`, `T1059.001`, `T1204` | Obfuscated Files, PowerShell |

---

## Related Notes

- [[Blue/Instruments/malware-analysis|Malware Analysis]]
- [[Blue/Instruments/volatility|Volatility 3]]

↩ [[Blue/Instruments/readme|Blue: Instruments]] · [[Blue/Blue|Blue]] · [[Mind Palace]]
