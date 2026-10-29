---
type: tool
room: blue
tool_name: "volatility"
version: "Volatility 3"
platform: [Linux, Windows, macOS]
phase: [Host Forensics, Memory Analysis, Malware Detection]
tags: [blue/tool, blue/memory-forensics, malfind, process-injection]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# Volatility 3

> [!quote]
> *"What one man can invent another can discover."*
> — *The Adventure of the Dancing Men*

**Room:** Blue — Instruments | **Role:** Memory Forensics & RAM Analysis | **Platform:** `Linux, Windows, macOS`

---

## The Instrument *(What It Is)*

`Volatility 3` is the world's most widely used framework for volatile memory forensics. It analyzes physical RAM dumps (`.raw`, `.dmp`, `.vmem`) extracted from compromised endpoints, reconstructing running processes, loaded DLLs, memory injections, network connections, credentials, and kernel rootkits.

- **Author / Origin:** The Volatility Foundation
- **License:** Volatility Software License (VSL)
- **Install:** `pip3 install volatility3`

---

## The Method *(Core Usage)*

### Basic Syntax
```bash
python3 vol.py -f <memory_image> <os_plugin> [options]
```

---

## The Cases *(Common Investigative Plugins)*

### Case 1 — Identifying OS & System Baseline
```bash
# Auto-detect Windows / Linux architecture and kernel build
python3 vol.py -f memory.dmp windows.info
python3 vol.py -f memory.dmp linux.info
```

### Case 2 — Process Execution & Hierarchy
```bash
# Process tree (identify rogue parent-child relationships e.g. Word -> cmd.exe)
python3 vol.py -f memory.dmp windows.pstree

# Extract command-line parameters (find encoded PowerShell, downloads, execution paths)
python3 vol.py -f memory.dmp windows.cmdline

# List loaded DLLs for specific suspicious PID
python3 vol.py -f memory.dmp windows.dlllist --pid 4820
```

### Case 3 — Network Connections
```bash
# Scan memory for open and closed TCP/UDP sockets
python3 vol.py -f memory.dmp windows.netscan
```

### Case 4 — Malware Injection & Hooking
```bash
# Detect memory injections, unmapped code execution, and hollowed processes
python3 vol.py -f memory.dmp windows.malfind
python3 vol.py -f memory.dmp windows.hollowprocesses
```

### Case 5 — Credential Extraction from RAM
```bash
# Dump SAM hashes and LSA secrets
python3 vol.py -f memory.dmp windows.hashdump
python3 vol.py -f memory.dmp windows.lsadump

# Offline LSASS dump analysis with pypykatz:
pypykatz lsa minidump lsass.dmp
```

---

## D3FEND / MITRE Mapping

| Framework | Reference | Capability |
|---|---|---|
| **D3FEND** | `d3f:MemoryAnalysis` | Memory Analysis |
| **Counters ATT&CK** | `T1055`, `T1003.001`, `T1059` | Process Injection, OS Credential Dumping: LSASS |

---

## Related Notes

- [[Blue/Instruments/malware-analysis|Malware Analysis]]
- [[Blue/Monographs/IR - Detect and Analyse|IR — Detect and Analyse]]

↩ [[Blue/Instruments/readme|Blue: Instruments]] · [[Blue/Blue|Blue]] · [[Mind Palace]]
