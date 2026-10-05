---
type: section
room: blue
tags: [blue/instruments]
---

# Blue: Instruments

> [!quote]
> *"It is a capital mistake to theorize before one has data."*

Analysis tools, commands, and filters for detection, investigation, and response. Every tool documented here includes the most-used commands so you can work without googling syntax mid-incident.

---

## Inventory

| Category | Tools |
|----------|-------|
| Packet Analysis | Wireshark, tshark, tcpdump, NetworkMiner |
| Log Analysis | Chainsaw, Hayabusa, grep/awk patterns |
| Memory Forensics | Volatility3 |
| Endpoint | Sysinternals (autoruns, procexp, procmon), velociraptor |
| SIEM Queries | Splunk SPL, Elastic KQL |
| Threat Intel | VirusTotal CLI, MISP, shodan |
| Malware Analysis | yara, capa, strings, exiftool |
| IOC Sources | See [[IOC Hunting & Threat Intel]] |

---

## 🦈 Wireshark / tshark

**Purpose:** Packet capture analysis — the most essential blue team instrument.

### Wireshark Display Filters (most used)

```wireshark
# === BASIC FILTERS ===
ip.addr == 192.168.1.1          # Filter by IP (src or dst)
ip.src == 192.168.1.1           # Source IP only
ip.dst == 10.10.10.5            # Destination IP only
ip.addr == 192.168.1.0/24       # Entire subnet

tcp.port == 443                 # TCP port (src or dst)
tcp.dport == 80                 # Destination port only
tcp.sport == 4444               # Source port only
udp.port == 53                  # UDP port

# === PROTOCOL FILTERS ===
http                            # All HTTP
https or ssl or tls             # TLS/HTTPS
dns                             # DNS only
ftp or ftp-data                 # FTP
smb or smb2                     # SMB traffic
icmp                            # ICMP (ping, tunnel detection)
arp                             # ARP (poisoning detection)
dhcp or bootp                   # DHCP

# === HTTP SPECIFIC ===
http.request                    # HTTP requests only
http.response                   # HTTP responses only
http.request.method == "POST"   # POST requests
http.request.method == "GET"    # GET requests
http.response.code == 200       # Successful responses
http.response.code == 404       # Not found
http.response.code >= 400       # All errors
http.host contains "evil"       # Host header contains keyword
http.request.uri contains ".php" # PHP endpoints

# === DNS ===
dns.qry.name contains "domain"  # DNS query name
dns.flags.rcode == 3            # NXDOMAIN (name not found)
dns.qry.type == 1               # A records
dns.qry.type == 28              # AAAA records
dns.qry.type == 16              # TXT records

# === TLS / CERTIFICATES ===
tls.handshake.type == 1         # ClientHello
tls.handshake.extensions_server_name contains "evil"  # SNI
ssl.handshake.ciphersuite == 0xc02b  # Specific cipher

# === TCP FLAGS ===
tcp.flags.syn == 1 && tcp.flags.ack == 0  # SYN (port scan)
tcp.flags.rst == 1              # RST (reset — closed ports)
tcp.flags == 0x002              # SYN only
tcp.flags == 0x012              # SYN-ACK

# === CREDENTIAL / CLEARTEXT ===
ftp.request.command == "PASS"   # FTP password in cleartext
pop.request.command == "PASS"   # POP3 password
imap.request contains "LOGIN"   # IMAP login
http.authheader                 # HTTP Basic auth

# === LARGE TRANSFERS / EXFIL ===
frame.len > 1400                # Large frames
tcp.len > 1000                  # Large TCP payload
http.content_length > 100000    # Large HTTP body

# === C2 DETECTION ===
# Look for periodic beaconing (same dst IP at regular intervals)
# Use Statistics > Conversations to see top talkers

# === COMBINE FILTERS ===
ip.src == 10.10.10.5 && tcp.dport == 443          # Specific host HTTPS
http.request.method == "POST" && http.host != "known-host.com"
dns.flags.rcode == 3 && !dns.qry.name contains "microsoft"  # Unexpected NXDOMAIN
```

### Wireshark Tips
```
Statistics > Protocol Hierarchy    → See all protocols in capture
Statistics > Conversations         → Top talkers (IP/TCP/UDP)
Statistics > Endpoints             → All unique IPs
Statistics > IO Graphs             → Traffic over time (spot beaconing)
Analyze > Follow TCP/UDP Stream    → Reconstruct a full session
File > Export Objects > HTTP       → Extract files from capture
```

### tshark (Command Line)

```bash
# Live capture on interface
tshark -i eth0

# Read a pcap file
tshark -r capture.pcap

# Apply display filter
tshark -r capture.pcap -Y "http.request.method == POST"

# Extract specific fields
tshark -r capture.pcap -Y "http.request" -T fields -e ip.src -e ip.dst -e http.host -e http.request.uri

# Extract DNS queries
tshark -r capture.pcap -Y "dns" -T fields -e dns.qry.name | sort | uniq -c | sort -rn

# Follow TCP stream (stream index from Wireshark)
tshark -r capture.pcap -q -z "follow,tcp,ascii,0"

# List all HTTP files
tshark -r capture.pcap -Y "http.response" --export-objects "http,/tmp/exported"

# Statistics: top talkers
tshark -r capture.pcap -q -z conv,ip

# Detect long DNS query labels (DNS exfil)
tshark -r capture.pcap -Y "dns" -T fields -e dns.qry.name | awk '{ if (length($1) > 50) print length($1), $1 }' | sort -rn

# Beaconing detection: connections per minute to same host
tshark -r capture.pcap -Y "tcp.flags.syn==1 && tcp.flags.ack==0" -T fields -e ip.dst -e frame.time_epoch | awk '{print $1, int($2/60)}' | sort | uniq -c | sort -rn
```

### tcpdump (Live Capture / Server Without GUI)

```bash
# Capture on interface, write to file
tcpdump -i eth0 -w capture.pcap

# Capture with filter
tcpdump -i eth0 "tcp port 80" -w http.pcap
tcpdump -i eth0 "not port 22" -w notssh.pcap   # Exclude SSH

# Read saved capture
tcpdump -r capture.pcap

# Capture to rotating files (10MB each, keep 5)
tcpdump -i eth0 -G 60 -W 5 -w capture_%Y%m%d_%H%M%S.pcap

# Key flags
# -n  no DNS resolution  -nn  no port name resolution
# -v  verbose  -vv  more verbose  -X  hex+ASCII  -A  ASCII only
tcpdump -i eth0 -nnvA "tcp port 4444"
```

---

## 🔗 Chainsaw (Windows EVTX)

**Purpose:** Hunt through Windows event logs at speed using Sigma rules.

```bash
# Install
wget https://github.com/WithSecureLabs/chainsaw/releases/latest/download/chainsaw_x86_64-unknown-linux-gnu.tar.gz

# Hunt with Sigma rules (best method)
chainsaw hunt /path/to/evtx/ \
  --rules /opt/chainsaw/rules/sigma/ \
  --mapping /opt/chainsaw/mapping_files/sigma-event-logs-all.yml \
  --output chainsaw-hunt.csv --csv

# Search for a keyword/string in all logs
chainsaw search "mimikatz" /path/to/evtx/
chainsaw search -t "Event[System/EventID=4624]" /path/to/evtx/

# Search for a specific event ID
chainsaw search --event-id 4688 /path/to/evtx/

# Generate timeline of all events
chainsaw analyse --timeline /path/to/evtx/ --output timeline.csv --csv

# Filter by time range
chainsaw hunt /path/to/evtx/ --rules ./rules/ --mapping ./mapping.yml \
  --from "2024-01-01T00:00:00" --to "2024-01-31T23:59:59"
```

---

## 🧠 Volatility 3 (Memory Forensics)

**Purpose:** Analyse Windows/Linux memory dumps for malware, credentials, network state.

```bash
# Determine OS (auto-detect)
python3 vol.py -f memory.dmp windows.info
python3 vol.py -f memory.dmp linux.info

# === PROCESS ANALYSIS ===
python3 vol.py -f memory.dmp windows.pslist         # Process list
python3 vol.py -f memory.dmp windows.pstree         # Process tree (see parents)
python3 vol.py -f memory.dmp windows.cmdline        # Command lines (key for hunting)
python3 vol.py -f memory.dmp windows.dlllist        # DLLs loaded per process
python3 vol.py -f memory.dmp windows.dlllist --pid 1234  # Specific process

# === NETWORK ===
python3 vol.py -f memory.dmp windows.netscan        # Network connections (incl. closed)
python3 vol.py -f memory.dmp windows.netstat        # Active only

# === CREDENTIAL HUNTING ===
python3 vol.py -f memory.dmp windows.hashdump       # SAM hashes
python3 vol.py -f memory.dmp windows.lsadump        # LSA secrets
# Use pypykatz for LSASS dump offline:
pypykatz lsa minidump lsass.dmp

# === MALWARE DETECTION ===
python3 vol.py -f memory.dmp windows.malfind        # Injected code (VAD anomalies)
python3 vol.py -f memory.dmp windows.hollowprocesses # Process hollowing

# === FILES / REGISTRY ===
python3 vol.py -f memory.dmp windows.filescan       # Open file handles
python3 vol.py -f memory.dmp windows.dumpfiles --physaddr 0x<addr>  # Extract file
python3 vol.py -f memory.dmp windows.registry.hivelist  # Registry hives
python3 vol.py -f memory.dmp windows.registry.printkey --key "Software\Microsoft\Windows\CurrentVersion\Run"

# === LINUX ===
python3 vol.py -f memory.dmp linux.pslist
python3 vol.py -f memory.dmp linux.bash            # Bash history from memory
python3 vol.py -f memory.dmp linux.netstat
```

---

## 🔎 grep / awk Patterns for Log Hunting

```bash
# === LINUX AUTH LOGS ===
# Failed SSH logins (brute force detection)
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn

# Successful logins from unusual IPs
grep "Accepted " /var/log/auth.log | awk '{print $9, $11}' | sort | uniq

# Sudo usage
grep "sudo:" /var/log/auth.log | grep -v "session\|opened\|closed"

# New user created
grep "useradd\|adduser\|new user" /var/log/auth.log

# === APACHE / NGINX ===
# Top IPs by request count
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -rn | head -20

# 4xx / 5xx errors (scanning / exploitation)
grep -E '" [45][0-9]{2} ' /var/log/apache2/access.log | awk '{print $7, $9}' | sort | uniq -c | sort -rn

# Webshell access (POST to .php)
grep "POST.*\.php" /var/log/nginx/access.log

# SQLi patterns in URLs
grep -iE "(union|select|insert|drop|exec|script)" /var/log/apache2/access.log

# === WINDOWS POWERSHELL ===
# Encoded PowerShell commands in logs
grep -r "EncodedCommand\|-enc " /path/to/logs/

# === GENERIC ===
# Count occurrences of pattern
grep -c "Failed" /var/log/auth.log

# Lines matching multiple patterns
grep -E "pattern1|pattern2" logfile

# Print surrounding context
grep -A 2 -B 2 "pattern" logfile

# Recursive search
grep -r "password" /var/www/html/ --include="*.php"
```

---

## 📊 Splunk SPL — Essential Queries

```splunk
# === AUTHENTICATION HUNTING ===
index=windows EventCode=4625
| stats count by src_ip, user, ComputerName
| where count > 10
| sort -count

index=windows EventCode=4624
| eval logon_type=case(Logon_Type==2,"Interactive",Logon_Type==3,"Network",Logon_Type==10,"Remote Interactive")
| table _time, ComputerName, user, logon_type, src_ip

# === PROCESS EXECUTION ===
index=windows (EventCode=4688 OR source="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=1)
| table _time, ComputerName, user, CommandLine
| where match(CommandLine, "(?i)(powershell|cmd|wscript|cscript|mshta|certutil|rundll32)")

# Encoded PowerShell detection
index=windows EventCode=4688
| where match(CommandLine, "(?i)(-enc|-EncodedCommand|frombase64)")
| table _time, host, user, CommandLine

# === LATERAL MOVEMENT ===
index=windows EventCode=4624 Logon_Type=3
| stats count by src_ip, user, ComputerName
| sort -count

# PsExec pattern
index=windows EventCode=7045
| where match(ServiceFileName, "(?i)(PSEXE|ADMIN\$|psexesvc)")
| table _time, ComputerName, ServiceName, ServiceFileName

# === SCHEDULED TASKS ===
index=windows EventCode=4698
| table _time, ComputerName, SubjectUserName, TaskName, TaskContent
| where NOT match(TaskName, "(?i)(microsoft|windows|google|adobe)")

# === NETWORK / DNS ===
index=network sourcetype=dns
| stats count by query
| where len(query) > 50   | sort -count

# Top outbound connections by bytes
index=network
| stats sum(bytes_out) as total_bytes by dest_ip
| sort -total_bytes
| head 20

# === BASELINE DEVIATION ===
index=windows EventCode=4688
| timechart span=1h count by host
| anomalydetection action=annotate
```

---

## 🔬 YARA Scanning

```bash
# Scan a single file
yara /path/to/rules.yar suspicious.exe

# Scan directory recursively
yara -r /path/to/rules/ /path/to/scan/

# Use rule categories from community rules
yara -r /opt/yara-rules/malware/ /tmp/

# Compile rules for speed
yarac rules.yar compiled.rules
yara -C compiled.rules target/

# Download community rules
git clone https://github.com/Yara-Rules/rules /opt/yara-rules
git clone https://github.com/Neo23x0/signature-base /opt/signature-base
```

**Write a simple YARA rule:**
```yara
rule Suspicious_PowerShell_Encoded {
    meta:
        description = "Detects encoded PowerShell in files"
        author = "Blue Team"
    strings:
        $enc1 = "-EncodedCommand" nocase
        $enc2 = "FromBase64String" nocase
        $enc3 = "IEX" nocase
    condition:
        2 of them
}
```

---

## 🧪 Malware Analysis Quick Commands

```bash
# Identify file type
file suspicious.exe
exiftool suspicious.exe

# Extract printable strings
strings suspicious.exe | grep -E "(http|https|\.exe|\.dll|cmd|power)"
strings -el suspicious.exe   # Unicode strings (Windows binaries)

# Compute hashes
sha256sum suspicious.exe && md5sum suspicious.exe

# PE header info
objdump -x suspicious.exe | head -50
readelf -h suspicious.elf

# Dynamic library imports
objdump -p suspicious.exe | grep "DLL"

# FLOSS — deobfuscated strings
floss suspicious.exe

# Capa — capability detection
capa suspicious.exe

# Check with YARA
yara /opt/signature-base/yara/ suspicious.exe
```

---

## 🖧 NetworkMiner

```
# Open PCAP in NetworkMiner → GUI
# Key tabs:
Hosts      → All IPs with OS fingerprint
Files      → Files extracted from capture
Credentials → Cleartext credentials found
DNS        → DNS queries and responses
Parameters → HTTP GET/POST parameters
```

---

## IOC Lookup Quick Commands

```bash
# VirusTotal hash check (CLI)
vt file <sha256>

# MalwareBazaar — no key needed
curl -s https://mb-api.abuse.ch/api/v1/ -d "query=get_info&hash=<sha256>" -X POST | jq '.data[0].tags, .data[0].vendor_intel'

# AbuseIPDB
curl -G https://api.abuseipdb.com/api/v2/check --data-urlencode "ipAddress=<ip>" -H "Key: <key>" | jq '.data.abuseConfidenceScore'

# Shodan
shodan host <ip>
shodan search "ssl:<cert_serial>"

# urlscan.io — search for domain
curl "https://urlscan.io/api/v1/search/?q=domain:<domain>&size=10" | jq '.results[].page.url'
```

---

## Related Notes

- [[IOC Hunting & Threat Intel]] ← Full IOC source list
- [[IR - Detect and Analyse]] ← Detection workflows
- [[IR - Contain]] ← What to do after detection

↩ [[Blue]] · [[Mind Palace]]