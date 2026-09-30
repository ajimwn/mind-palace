---
type: detection-note
room: blue
d3fend_technique: "{{d3fend_id}}"
d3fend_name: "{{d3fend_name}}"
mitre_attack_counters: ["T{{id}}", "T{{id}}"]
platform: [Windows, Linux, Network]
tags: [blue/detect, blue/{{category_slug}}]
date_added: {{date}}
last_reviewed: {{date}}
status: draft
---

# {{d3fend_name}}

> [!quote]
> *"The world is full of obvious things which nobody by any chance ever observes."*
> — *The Hound of the Baskervilles*

**D3FEND:** `{{d3fend_id}}` | **NIST Phase:** Detect & Analyse | **Counters ATT&CK:** `{{attack_tactic}}`

---

## The Theory *(What & Why)*

What this defensive technique achieves and why it's worth implementing. What adversary behaviour does it surface?

**D3FEND Reference:** [{{d3fend_id}}](https://d3fend.mitre.org/technique/d3f:{{d3fend_slug}}/)
**ATT&CK Counters:** [T{{id}}](https://attack.mitre.org/techniques/T{{id}}/)

---

## Detection Logic

### Windows Event IDs to Watch

| Event ID | Source | Trigger Condition |
|----------|--------|-------------------|
| `XXXX` | Security | Condition description |
| `X (Sysmon)` | Sysmon | Condition description |

### Sigma Rule (YAML)

```yaml
title: Descriptive Title of the Detection
id: # generate at https://uuidgenerator.net/
status: experimental
description: >
  What this rule detects and why.
references:
  - https://attack.mitre.org/techniques/T{{id}}/
author: Your Name
date: {{date}}
tags:
  - attack.{{tactic_slug}}
  - attack.t{{id}}
logsource:
  category: process_creation
  product: windows
detection:
  selection:
    EventID: 4688
    CommandLine|contains:
      - 'keyword1'
      - 'keyword2'
  condition: selection
falsepositives:
  - List known legitimate uses that could trigger this
level: high
```

### Elastic KQL / Splunk SPL

```
# Elastic KQL
event.code: "4688" AND process.command_line: (*keyword1* OR *keyword2*)

# Splunk SPL
index=windows EventCode=4688 CommandLine IN ("*keyword1*","*keyword2*")
| table _time, host, user, CommandLine
| sort -_time
```

---

## Tooling

| Tool | Role | Command |
|------|------|---------|
| Tool name | What it does here | `command --flag` |

---

## False Positive Guidance

> [!note] Expected Legitimate Activity
> Document what legitimate processes may trigger this rule and how to distinguish them.

- Legitimate use case 1 → filter with `AND NOT process.parent.name: "known_parent.exe"`
- Legitimate use case 2 → whitelist specific paths

---

## Tuning Notes

- How to reduce noise in your specific environment
- Thresholds to set (e.g. >5 events in 60s)
- Exclusions to add for known-good systems

---

## Response Actions

When this alert fires:

1. **Triage:** Confirm it is not a false positive using context (parent process, user, time)
2. **Pivot:** Look for related events in the same timeframe (lateral movement, new accounts)
3. **Contain:** If confirmed malicious → follow [[IR - Contain]]
4. **Escalate:** If scope is unclear → engage IR lead

---

## Related Notes

- [[IR - Detect and Analyse]] ← Parent workflow
- [[]] ← ATT&CK technique being countered
- [[]] ← Related detection

↩ [[Blue - Notes]] · [[Blue]] · [[Mind Palace]]
