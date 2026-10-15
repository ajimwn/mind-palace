---
type: monograph
room: blue
subject: "Snort IDS/IPS"
tags: [blue/detection, blue/snort, blue/ids]
date_updated: {{date}}
---

# Detection — Snort IDS/IPS

> [!quote]
> *"The game is afoot."* — Snort has been playing it since 1998.

**Version:** Snort 2.x / 3.x | **Phase:** Detect (NIST CSF) | **D3FEND:** Network Traffic Analysis

Snort is an open-source Network Intrusion Detection/Prevention System (NIDS/NIPS). It operates in three modes: **sniffer** (packet dump), **packet logger** (disk capture), and **NIDS/IPS** (rule-based detection with alerts and blocking).

---

## Mode Overview

```
┌─────────────────────────────────────────────────────────┐
│ Snort Modes                                             │
│                                                         │
│  Sniffer  ──► reads packets, prints to console          │
│  Logger   ──► reads packets, writes to disk             │
│  NIDS/IPS ──► reads packets, applies rules, alerts/drops│
└─────────────────────────────────────────────────────────┘
```

---

## Global Flags

| Flag | Description |
|------|-------------|
| `-V` / `--version` | Display Snort version |
| `-q` | Quiet — suppress banner and status reports |
| `-i eth0` | Use specific network interface |
| `-c /etc/snort/snort.conf` | Load configuration file |
| `-T` | Test configuration file (no live capture) |
| `-N` | Disable logging |
| `-D` | Run Snort as a background daemon |

---

## Sniffer Mode

Reads packets from the interface and prints to console. No rules applied.

```bash
# Verbose — show IP and TCP/UDP/ICMP headers
snort -v

# Show link-layer (Ethernet) headers
snort -e

# Show data payload (ASCII)
snort -d

# Show full packet in HEX + ASCII
snort -X

# Show everything (link-layer + payload in HEX)
snort -eX

# Sniff exactly N packets then stop
snort -v -n 10

# Sniff on specific interface
snort -v -i eth0
```

---

## Logger Mode

Writes captured packets to disk.

```bash
# Log to default path (/var/log/snort)
snort -v -l /var/log/snort

# Log to custom path
snort -v -l /home/username/Desktop

# Log in ASCII format (human-readable filenames)
snort -v -K ASCII -l /var/log/snort

# Read a previously captured Snort log file
snort -v -r snort.log

# Read only N packets from log
snort -v -r snort.log -n 10

# Filter log with Berkeley Packet Filters (BPF)
snort -v -r snort.log tcp
snort -v -r snort.log 'udp and port 53'
snort -v -r snort.log 'host 192.168.1.1'
```

> [!note] Default Log Path
> `/var/log/snort` — binary format by default. Use `-K ASCII` for human-readable output or `-r` to read binary files back.

---

## IDS/IPS Mode — With Config File

Applies rules from `snort.conf` and generates alerts.

```bash
# Load config, run on live interface
snort -c /etc/snort/snort.conf -i eth0

# Test config without capturing
snort -c /etc/snort/snort.conf -T

# Run with verbose output
snort -c /etc/snort/snort.conf -v

# Run in background (daemon)
snort -c /etc/snort/snort.conf -D

# Disable logging (alerts only to console)
snort -c /etc/snort/snort.conf -N
```

### Alert Modes

| Mode | Command | Output |
|------|---------|--------|
| None | `-A none` | No alert output |
| Console (brief) | `-A console` | Timestamp, rule message to terminal |
| Console (detailed) | `-A cmg` | Full header + payload to terminal |
| Fast (file) | `-A fast` | Single-line alert per match → `/var/log/snort/alert` |
| Full (file) | `-A full` | All packet details → `/var/log/snort/alert` |

```bash
# No alert output (useful for testing)
snort -c /etc/snort/snort.conf -v -A none

# Console — brief (best for live monitoring)
snort -c /etc/snort/snort.conf -v -A console

# Console — full packet detail + CMG format
snort -c /etc/snort/snort.conf -v -A cmg

# File — fast (lightweight, production-friendly)
snort -c /etc/snort/snort.conf -v -A fast -l /var/log/snort

# File — full packet detail
snort -c /etc/snort/snort.conf -v -A full -l /var/log/snort

# Use local rules file without full config
snort -c /etc/snort/rules/local.rules -v -A console
```

---

## PCAP Processing Mode

Run Snort against captured `.pcap` files — no live interface needed. Essential for offline analysis and CTF forensics.

```bash
# Process a single pcap
snort -c /etc/snort/snort.conf -q -r file.pcap -A console

# Process multiple named pcap files
snort -c /etc/snort/snort.conf -q \
  --pcap-list="file1.pcap file2.pcap" \
  -A console

# Process all pcaps in a directory
snort -c /etc/snort/snort.conf -q \
  --pcap-dir=/home/pcap-folder \
  -A console

# Show which pcap file is currently being processed
snort -c /etc/snort/snort.conf -q \
  --pcap-list="file1.pcap file2.pcap" \
  -A console --pcap-show
```

> [!tip] CTF Workflow
> ```bash
> # Quick triage — check all pcaps in a directory, print alerts to console
> snort -c /etc/snort/snort.conf -q --pcap-dir=./pcaps -A console --pcap-show 2>/dev/null
> ```

---

## Snort Rule Syntax

### Rule Structure

```
<Action> <Protocol> <SrcIP> <SrcPort> <Direction> <DstIP> <DstPort> (<Options>)
```

```
alert tcp $EXTERNAL_NET any -> $HOME_NET $HTTP_PORTS ( <options> )
│     │   │              │   │  │         │
│     │   │              │   │  │         └─ Destination ports (variable)
│     │   │              │   │  └─ Destination IP (variable)
│     │   │              │   └─ Direction: -> (one-way) or <> (bidirectional)
│     │   │              └─ Source port (any)
│     │   └─ Source IP (variable $EXTERNAL_NET)
│     └─ Protocol: tcp | udp | icmp | ip
└─ Action: alert | log | pass | drop | reject | sdrop
```

### Direction Operators
| Operator | Meaning |
|----------|---------|
| `->` | Source → Destination (unidirectional) |
| `<>` | Bidirectional match |

### Common Variables (defined in `snort.conf`)
| Variable | Typical Value |
|----------|--------------|
| `$HOME_NET` | Your internal network (e.g. `192.168.1.0/24`) |
| `$EXTERNAL_NET` | `!$HOME_NET` (everything not internal) |
| `$HTTP_PORTS` | `[80,8080,8443]` |
| `$SQL_PORTS` | `[3306,1433,5432]` |

---

## Rule Options

### General Options
| Option | Description | Example |
|--------|-------------|---------|
| `msg` | Alert message displayed | `msg:"Directory Traversal!";` |
| `sid` | Unique rule ID (>1000000 for custom) | `sid:1000001;` |
| `rev` | Rule revision number | `rev:1;` |
| `reference` | External reference (CVE, URL) | `reference:CVE,2021-44228;` |
| `priority` | Rule priority (1=highest) | `priority:1;` |

### Payload Detection Options
| Option | Description | Example |
|--------|-------------|---------|
| `content` | Exact byte/string match | `content:"passwd";` |
| `nocase` | Case-insensitive content match | `content:"GET"; nocase;` |
| `fast_pattern` | Prioritise this content for search engine | `fast_pattern;` |
| `offset` | Start content search at byte N | `offset:4;` |
| `depth` | Only search first N bytes | `depth:20;` |
| `distance` | Content must follow previous match by N bytes | `distance:0;` |
| `within` | Content must appear within N bytes of prior match | `within:10;` |
| `pcre` | Perl-compatible regex | `pcre:"/passwd/i";` |

### Non-Payload Detection Options
| Option | Description | Example |
|--------|-------------|---------|
| `flags` | TCP flag check | `flags:S;` (SYN) |
| `ttl` | Match IP TTL value | `ttl:128;` |
| `itype` | ICMP type | `itype:8;` (ICMP echo) |
| `dsize` | Payload size match | `dsize:>100;` |
| `fragbits` | IP fragmentation bits | `fragbits:M;` |

### Flow / Post-Detection Options
| Option | Description | Example |
|--------|-------------|---------|
| `flow` | TCP session direction | `flow:established,to_server;` |
| `session` | Extract session data | `session:all;` |
| `threshold` | Rate limit alerts | `threshold:type limit,track by_src,count 5,seconds 60;` |
| `detection_filter` | Alert only after N matches | `detection_filter:track by_src,count 10,seconds 5;` |

---

## Rule Examples

```bash
# Detect any ICMP ping
alert icmp any any -> $HOME_NET any (
  msg:"ICMP Ping Detected";
  itype:8;
  sid:1000001; rev:1;)

# Detect possible SQL injection in HTTP
alert tcp $EXTERNAL_NET any -> $HOME_NET $HTTP_PORTS (
  msg:"Possible SQL Injection";
  flow:established,to_server;
  content:"SELECT"; nocase; fast_pattern;
  content:"FROM"; nocase; distance:0;
  sid:1000002; rev:1;)

# Detect directory traversal
alert tcp $EXTERNAL_NET any -> $HOME_NET $HTTP_PORTS (
  msg:"Directory Traversal Attempt";
  flow:established;
  content:"HTTP"; fast_pattern;
  content:"|2E 2E 2F|"; content:"/.."; nocase;
  reference:CVE,XXX;
  sid:1000003; rev:1;)

# Detect nmap SYN scan (large number of SYN with no ACK)
alert tcp $EXTERNAL_NET any -> $HOME_NET any (
  msg:"Possible Nmap SYN Scan";
  flags:S,12;
  threshold:type threshold,track by_src,count 20,seconds 5;
  sid:1000004; rev:1;)

# Detect FTP brute force
alert tcp $EXTERNAL_NET any -> $HOME_NET 21 (
  msg:"Possible FTP Brute Force";
  flow:established,to_server;
  content:"PASS"; nocase;
  detection_filter:track by_src,count 10,seconds 30;
  sid:1000005; rev:1;)
```

---

## Testing Custom Rules

```bash
# Write your rule to a local rules file
echo 'alert icmp any any -> any any (msg:"ICMP Test"; sid:9999; rev:1;)' \
  >> /etc/snort/rules/local.rules

# Test rule syntax
snort -c /etc/snort/snort.conf -T

# Test against a pcap
snort -c /etc/snort/rules/local.rules -r test.pcap -A console -q

# Watch alerts live
tail -f /var/log/snort/alert
```

---

## Related Notes

- [[Blue - Instruments]] ← Full blue team tool cheat sheet
- [[Detection - Zeek]] ← Network analysis (complements Snort)
- [[IOC Hunting & Threat Intel]] ← What to write rules for

↩ [[Blue - Monographs]] · [[Blue]] · [[Mind Palace]]
