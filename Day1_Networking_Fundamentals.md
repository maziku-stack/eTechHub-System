# Day 1: Networking Fundamentals & Reconnaissance

## Training Objective
Students will master networking concepts, TCP/IP stack, and learn reconnaissance techniques used in security assessments.

---

## Module 1: TCP/IP Model Deep Dive

### 1.1 Network Layers
- **Application Layer**: HTTP, HTTPS, FTP, SSH, DNS, Telnet
- **Transport Layer**: TCP (reliable), UDP (fast), Ports (0-65535)
- **Internet Layer**: IP (IPv4, IPv6), ICMP, Routing
- **Link Layer**: MAC addresses, Ethernet, ARP

### 1.2 Common Ports You Must Know
```
21    - FTP (File Transfer)
22    - SSH (Secure Shell)
23    - Telnet (unencrypted remote access - VULNERABLE)
25    - SMTP (Email sending)
53    - DNS (Domain Name System)
80    - HTTP (Web traffic)
443   - HTTPS (Secure web)
3306  - MySQL Database
3389  - RDP (Remote Desktop)
5432  - PostgreSQL Database
8080  - Alternative HTTP
```

---

## Module 2: Reconnaissance Techniques

### 2.1 Passive Reconnaissance (Non-Intrusive)
Tasks that don't directly contact the target system.

**WHOIS Lookup - Domain Information**
```bash
whois example.com
# Returns: Registrant info, DNS servers, creation date
```

**DNS Enumeration**
```bash
nslookup example.com
dig example.com
dig example.com MX          # Mail servers
dig example.com NS          # Name servers
dig example.com ANY         # All records
```

**Google Dorking (OSINT)**
```
site:example.com            # All indexed pages
site:example.com filetype:pdf
inurl:admin
inurl:login
"example.com" password
```

### 2.2 Active Reconnaissance (Direct Contact)

**ICMP Ping - Check if Host is Alive**
```bash
ping example.com
ping -c 4 example.com  # Linux: 4 packets
```

**Traceroute - Find Path to Target**
```bash
tracert example.com    # Windows
traceroute example.com # Linux
```

**Port Scanning - Find Open Ports**
```bash
nmap example.com                    # Basic scan
nmap -p 1-1000 example.com         # Specific port range
nmap -p- example.com               # All ports
nmap -sV example.com               # Service version detection
nmap -O example.com                # OS detection
nmap -A example.com                # Aggressive scan (all above)
```

---

## Module 3: Python Network Tools (Hands-On)

### 3.1 Basic ICMP Ping Tool
```python
import socket
import os

def ping_host(host):
    """Simple ping implementation using ICMP"""
    try:
        # This works on Linux/Mac. Windows may need ICMP permission
        result = os.system(f"ping -c 1 {host}")
        if result == 0:
            print(f"[+] {host} is alive")
            return True
        else:
            print(f"[-] {host} is unreachable")
            return False
    except Exception as e:
        print(f"Error: {e}")
        return False

# Usage
ping_host("8.8.8.8")
ping_host("google.com")
```

### 3.2 Port Scanner
```python
import socket
import sys

def scan_port(host, port):
    """Scan a single port"""
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.settimeout(1)
    
    try:
        result = sock.connect_ex((host, port))
        if result == 0:
            print(f"[+] Port {port}: OPEN")
            return True
        else:
            print(f"[-] Port {port}: CLOSED")
            return False
    except socket.gaierror:
        print(f"[-] Hostname could not be resolved: {host}")
        return False
    except socket.error:
        print(f"[-] Could not connect to {host}")
        return False
    finally:
        sock.close()

def scan_ports(host, port_range):
    """Scan multiple ports"""
    print(f"\n[*] Scanning host: {host}")
    print(f"[*] Scanning ports: {port_range}\n")
    
    open_ports = []
    for port in port_range:
        if scan_port(host, port):
            open_ports.append(port)
    
    if open_ports:
        print(f"\n[+] Found {len(open_ports)} open ports: {open_ports}")
    else:
        print(f"\n[-] No open ports found")

# Usage
scan_ports("127.0.0.1", range(20, 100))
scan_ports("google.com", [80, 443, 8080])
```

### 3.3 DNS Resolver
```python
import socket

def resolve_hostname(hostname):
    """Resolve hostname to IP"""
    try:
        ip = socket.gethostbyname(hostname)
        print(f"[+] {hostname} -> {ip}")
        return ip
    except socket.gaierror as e:
        print(f"[-] Cannot resolve hostname: {e}")
        return None

def reverse_dns_lookup(ip):
    """Reverse DNS lookup"""
    try:
        hostname = socket.gethostbyaddr(ip)
        print(f"[+] {ip} -> {hostname[0]}")
        return hostname[0]
    except socket.herror as e:
        print(f"[-] Reverse DNS failed: {e}")
        return None

# Usage
resolve_hostname("google.com")
reverse_dns_lookup("8.8.8.8")
```

### 3.4 Banner Grabbing (Service Detection)
```python
import socket

def grab_banner(host, port):
    """Grab service banner from open port"""
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.settimeout(2)
    
    try:
        sock.connect((host, port))
        banner = sock.recv(1024).decode('utf-8', errors='ignore')
        print(f"[+] Banner from {host}:{port}")
        print(f"    {banner.strip()}")
        return banner
    except Exception as e:
        print(f"[-] Could not grab banner: {e}")
        return None
    finally:
        sock.close()

# Usage
grab_banner("google.com", 80)
grab_banner("8.8.8.8", 53)
```

---

## Module 4: Network Traffic Analysis

### 4.1 Understanding Packet Structure
- **MAC Address**: Physical address (Layer 2)
- **IP Address**: Logical address (Layer 3)
- **Port Number**: Process identifier (Layer 4)
- **Payload**: Actual data being transmitted (Layer 5+)

### 4.2 Common Protocols
| Protocol | Port | Purpose | Security |
|----------|------|---------|----------|
| HTTP | 80 | Web | None (Plaintext) |
| HTTPS | 443 | Secure Web | SSL/TLS (Encrypted) |
| FTP | 21 | File Transfer | None (Plaintext) |
| SSH | 22 | Secure Shell | Encrypted |
| Telnet | 23 | Remote Access | None (Plaintext) |
| SMTP | 25 | Email Send | None (Plaintext) |
| DNS | 53 | Domain Resolution | None (Plaintext) |

---

## Module 5: Lab Exercises

### Exercise 1: Scan Your Local Network
```python
import subprocess
import sys

def scan_local_network(network_range):
    """Scan local network for active hosts"""
    print(f"[*] Scanning network: {network_range}")
    
    for i in range(1, 255):
        ip = f"{network_range}.{i}"
        # Using ARP scan (faster, more reliable)
        if sys.platform == "win32":
            result = subprocess.run(['ping', '-n', '1', '-w', '500', ip],
                                  capture_output=True)
        else:
            result = subprocess.run(['ping', '-c', '1', '-W', '1', ip],
                                  capture_output=True)
        
        if result.returncode == 0:
            print(f"[+] Host found: {ip}")

# Usage
scan_local_network("192.168.1")
```

### Exercise 2: Create a Port Scanner with Threading
```python
import socket
import threading
from queue import Queue

def worker(host, port_queue):
    """Worker thread for scanning ports"""
    while True:
        port = port_queue.get()
        if port is None:
            break
        
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(0.5)
        
        try:
            result = sock.connect_ex((host, port))
            if result == 0:
                print(f"[+] Port {port}: OPEN")
        except:
            pass
        finally:
            sock.close()
        
        port_queue.task_done()

def threaded_scan(host, ports, threads=10):
    """Multi-threaded port scanner"""
    port_queue = Queue()
    
    # Add ports to queue
    for port in ports:
        port_queue.put(port)
    
    # Create worker threads
    thread_list = []
    for _ in range(threads):
        t = threading.Thread(target=worker, args=(host, port_queue))
        t.start()
        thread_list.append(t)
    
    # Wait for completion
    port_queue.join()
    
    # Stop workers
    for _ in range(threads):
        port_queue.put(None)
    for t in thread_list:
        t.join()

# Usage
threaded_scan("127.0.0.1", range(1, 1001), threads=20)
```

---

## Day 1 Summary

✓ Learned TCP/IP model and networking concepts
✓ Mastered reconnaissance techniques (passive & active)
✓ Built Python network tools
✓ Understood packet structure and protocols
✓ Can now identify targets and find vulnerabilities entry points

**Homework**: 
1. Map your local network topology
2. Create a network scanner script
3. Identify 5 running services on your PC

---

## Important Note on Ethics
⚠️ All techniques taught here are for AUTHORIZED security testing only:
- Only scan networks you own or have explicit permission to test
- Unauthorized network scanning is ILLEGAL
- Always get written permission before security testing
- These skills must be used responsibly for nation's cybersecurity
