---
type: tool
room: red
tool_name: "{{tool_name}}"
version: ""
platform: [Linux, Windows, macOS]
phase: [Reconnaissance, Scanning, Exploitation, Privilege Escalation, Lateral Movement, Persistence, Exfiltration]
tags: [red/tool, red/{{phase_slug}}]
date_added: {{date}}
last_reviewed: {{date}}
---

# {{tool_name}}

> [!quote]
> *"It is a capital mistake to theorize before one has data."*
> — *A Scandal in Bohemia*

**Room:** Red — Instruments | **Phase(s):** `{{phase}}` | **Platform:** `{{platform}}`

---

## The Instrument *(What It Is)*

Brief description of the tool: what problem it solves, why it's in the kit.

- **Author / Origin:**
- **License:**
- **Install:** `apt install {{tool_name}}` / `pip install {{tool_name}}` / GitHub link

---

## The Method *(Core Usage)*

### Basic Syntax
```bash
{{tool_name}} [options] <target>
```

### Key Flags

| Flag | Meaning |
|------|---------|
| `-f` | description |
| `-o` | output file |
| `-v` | verbose |

---

## The Cases *(Common Use-Patterns)*

### Case 1 — Typical Engagement Use
```bash
# Descriptive comment explaining the scenario
{{tool_name}} --flag value target
```

### Case 2 — Stealth / OPSEC-aware Variant
```bash
# Slower, quieter, avoids common detections
{{tool_name}} --slow --flag value target
```

### Case 3 — Output Parsing / Piping
```bash
# Chain with grep/awk/jq for actionable output
{{tool_name}} --flag target | grep "pattern" | tee output.txt
```

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name |
|-------------|----------------|
| T1XXX | Technique name |

---

## Gotchas & Caveats

> [!warning] Common Mistakes
> - Pitfall one — and how to avoid it
> - Pitfall two — rate limiting, detection, etc.

---

## OPSEC Notes

- Artefacts left on target system (if any)
- Network signatures / common IDS rules
- How blue teams detect this tool's traffic

---

## Related Notes

- [[]] ← Parent phase note
- [[]] ← Similar tool
- [[]] ← TTP that uses this tool

↩ [[Red - Instruments]] · [[Red]] · [[Mind Palace]]
