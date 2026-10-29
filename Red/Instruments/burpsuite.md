---
type: tool
room: red
tool_name: "burpsuite"
version: "Community / Pro"
platform: [Linux, Windows, macOS]
phase: [Reconnaissance, Exploitation]
tags: [red/tool, red/web, proxy, repeater, intruder]
date_added: 2026-10-30
last_reviewed: 2026-10-30
---

# Burp Suite

> [!quote]
> *"There is a wonderful clarity in the dissection of an intercepted transmission."*

**Room:** Red — Instruments | **Phase(s):** `Reconnaissance, Exploitation` | **Platform:** `Linux, Windows, macOS`

---

## The Instrument *(What It Is)*

Burp Suite by PortSwigger is the industry-standard HTTP/HTTPS proxy and integrated platform for performing security assessments of web applications and APIs. It intercepts, logs, modifies, and replays web communications between browser and server.

- **Author / Origin:** PortSwigger
- **License:** Proprietary (Community Edition free / Pro commercial)
- **Install:** Pre-installed on Kali Linux / `burpsuite`

---

## The Method *(Core Usage)*

### Essential Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+R` | Send currently viewed request to **Repeater** |
| `Ctrl+I` | Send currently viewed request to **Intruder** |
| `Ctrl+Shift+U` | URL-decode selected string |
| `Ctrl+U` | URL-encode selected string |
| `Ctrl+Shift+B` | Base64-decode selected string |
| `Ctrl+B` | Base64-encode selected string |
| `Ctrl+Space` | Send request in Repeater |

### Intruder Attack Types

| Attack Type | Use Case |
|-------------|----------|
| **Sniper** | Single payload list tested across one or multiple markers sequentially |
| **Battering Ram** | Single payload list inserted into ALL marker positions simultaneously |
| **Pitchfork** | Multiple payload lists tested in lockstep (Index 0 with Index 0, 1 with 1) |
| **Cluster Bomb** | Multiple payload lists tested in full Cartesian product (all combinations) |

---

## The Cases *(Common Use-Patterns)*

### Case 1 — Parameter Manipulation in Repeater
1. Configure browser proxy to `127.0.0.1:8080`.
2. Inspect target request in `Proxy > HTTP history`.
3. Press `Ctrl+R` to forward to Repeater.
4. Modify parameters (SQLi payloads, IDOR values, header forgery) and evaluate responses in real-time.

### Case 2 — IDOR / Rate-Limit Testing in Intruder
1. Select target parameter e.g. `user_id=§1§`.
2. Set Attack Type to **Sniper**.
3. Under `Payloads`, choose **Numbers** from `1` to `1000`.
4. Start attack and sort results by response byte length or status code.

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name |
|-------------|----------------|
| `T1190` | Exploit Public-Facing Application |
| `T1557` | Adversary-in-the-Middle |

---

## Related Notes

- [[Red/Instruments/sqlmap|sqlmap]]
- [[Red/Instruments/ffuf|ffuf]]
- [[Red/Monographs/TA0001 - Initial Access|TA0001 — Initial Access]]

↩ [[Red/Instruments/readme|Red: Instruments]] · [[Red/Red|Red]] · [[Mind Palace]]
