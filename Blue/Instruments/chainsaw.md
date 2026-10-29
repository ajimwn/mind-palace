---
type: tool
room: blue
tool_name: "chainsaw"
version: "v2.x"
platform: [Windows, Linux, macOS]
phase: [Detection, Host Forensics, Log Analysis]
tags: [blue/tool, blue/host-forensics, evtx, sigma, windows-logs]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# Chainsaw

> [!quote]
> *"Eliminate all other factors, and the one which remains must be the truth."*
> — *The Sign of the Four*

**Room:** Blue — Instruments | **Role:** Fast Windows EVTX Log Analysis with Sigma Rules | **Platform:** `Windows, Linux, macOS`

---

## The Instrument *(What It Is)*

`Chainsaw` provides rapid search, extraction, and Sigma rule evaluation across Windows Event Log files (`.evtx`). Written in Rust, it processes gigabytes of triage logs in seconds, matching adversary tradecraft against community Sigma rules, detecting event log tampering, and producing forensic timelines.

- **Author / Origin:** WithSecure Labs (Countercept)
- **License:** GNU GPL v3
- **Repository:** https://github.com/WithSecureLabs/chainsaw

---

## The Method *(Core Usage)*

### Basic Syntax
```bash
chainsaw hunt <log_directory> --rules <rules_dir> --mapping <mapping.yml> [options]
chainsaw search <pattern> <log_directory> [options]
chainsaw analyse --timeline <log_directory> [options]
```

---

## The Cases *(Common Use-Patterns)*

### Case 1 — Rapid Sigma Rule Threat Hunt
```bash
# Hunt through EVTX triage folder using Sigma rules and export CSV
chainsaw hunt /evidence/C/Windows/System32/winevt/Logs/ \
  --rules /opt/chainsaw/rules/sigma/ \
  --mapping /opt/chainsaw/mapping_files/sigma-event-logs-all.yml \
  --output hunt_results.csv --csv
```

### Case 2 — Keyword Search Across Forensic Dumps
```bash
# Search for keyword (e.g. Mimikatz, Rubeus, Cobalt Strike)
chainsaw search "mimikatz" /evidence/Logs/

# Search for specific sensitive Event ID (e.g. 4688 Process Creation)
chainsaw search --event-id 4688 /evidence/Logs/
```

### Case 3 — Time-Bounded Investigation
```bash
# Filter hunt to specific incident window
chainsaw hunt /evidence/Logs/ --rules ./rules/ --mapping ./mapping.yml \
  --from "2026-10-29T18:00:00" --to "2026-10-30T02:00:00"
```

---

## D3FEND / MITRE Mapping

| Framework | Reference | Capability |
|---|---|---|
| **D3FEND** | `d3f:LogAnalysis` | Operating System Event Analysis |
| **Counters ATT&CK** | `T1078`, `T1059`, `T1053` | Valid Accounts, Command Execution |

---

## Related Notes

- [[Blue/Instruments/log-hunting|log-hunting (grep/awk)]]
- [[Blue/Instruments/splunk|Splunk SPL Queries]]
- [[Blue/Monographs/IR - Detect and Analyse|IR — Detect and Analyse]]

↩ [[Blue/Instruments/readme|Blue: Instruments]] · [[Blue/Blue|Blue]] · [[Mind Palace]]
