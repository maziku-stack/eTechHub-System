# Day 4: Defensive Security & Network Hardening

## Training Objective
Learn how to defend against attacks, harden systems, and implement security best practices.

---

## Module 1: Defensive Mindset

### 1.1 Defense in Depth (Layered Security)

```
Internet
    |
    v
[Firewall] <- Block unauthorized access
    |
    v
[IDS/IPS] <- Detect & prevent intrusions
    |
    v
[Web Application Firewall] <- Application-level protection
    |
    v
[Server Hardening] <- Minimize attack surface
    |
    v
[Authentication] <- User verification
    |
    v
[Authorization] <- Permission control
    |
    v
[Encryption] <- Protect data in transit
    |
    v
[Monitoring/Logging] <- Detect incidents
```

### 1.2 Security Principles (CIA Triad)

**Confidentiality**: Only authorized users access data
**Integrity**: Data cannot be modified without detection
**Availability**: Systems are accessible when needed

---

## Module 2: Server Hardening

### 2.1 Linux Server Hardening Checklist

```python
def server_hardening_checklist():
    """Linux server security hardening steps"""
    
    checklist = {
        "System Updates": [
            "sudo apt-get update",
            "sudo apt-get upgrade",
            "sudo apt-get install unattended-upgrades",
        ],
        "User Management": [
            "Remove unnecessary user accounts",
            "Disable root login",
            "sudo passwd -l root",
            "Enforce strong password policy",
        ],
        "SSH Hardening": [
            "Edit /etc/ssh/sshd_config",
            "PermitRootLogin no",
            "PasswordAuthentication no  # Use keys only",
            "PubkeyAuthentication yes",
            "Port 2222  # Non-standard port",
            "sudo systemctl restart ssh",
        ],
        "Firewall Configuration": [
            "sudo apt-get install ufw",
            "sudo ufw enable",
            "sudo ufw default deny incoming",
            "sudo ufw default allow outgoing",
            "sudo ufw allow 22/tcp",
            "sudo ufw allow 80/tcp",
            "sudo ufw allow 443/tcp",
        ],
        "File Permissions": [
            "chmod 600 ~/.ssh/id_rsa",
            "chmod 644 ~/.ssh/id_rsa.pub",
            "chmod 700 ~/.ssh",
            "chmod 600 /etc/shadow",
            "chmod 755 /etc/passwd",
        ],
        "Security Monitoring": [
            "sudo apt-get install auditd",
            "sudo apt-get install fail2ban",
            "Configure centralized logging",
            "Monitor system logs regularly",
        ],
        "Disable Unnecessary Services": [
            "sudo systemctl disable telnet",
            "sudo systemctl disable ftp",
            "Uninstall unnecessary packages",
            "Close unnecessary ports",
        ],
    }
    
    print("[*] SERVER HARDENING CHECKLIST\n")
    for category, steps in checklist.items():
        print(f"[{category}]")
        for step in steps:
            print(f"  - {step}")
        print()

server_hardening_checklist()
```

### 2.2 SSH Key Management (Best Practice)

```python
def setup_ssh_keys_secure():
    """Proper SSH key setup"""
    
    print("""[+] SECURE SSH KEY SETUP

STEP 1: Generate strong SSH key on CLIENT
$ ssh-keygen -t ed25519 -C "user@domain.com"
  - Use Ed25519 (more secure than RSA)
  - Provide strong passphrase
  - Key stored in ~/.ssh/id_ed25519

STEP 2: Copy key to SERVER
$ ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server.com
  or manually:
$ cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys

STEP 3: Secure permissions on SERVER
$ chmod 700 ~/.ssh
$ chmod 600 ~/.ssh/authorized_keys

STEP 4: Disable password authentication (/etc/ssh/sshd_config)
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no

STEP 5: Restart SSH
$ sudo systemctl restart ssh

STEP 6: TEST (don't close original session yet!)
$ ssh -i ~/.ssh/id_ed25519 user@server.com

STEP 7: Verify no password prompt, then close

BENEFITS:
✓ Much harder to crack than passwords
✓ Can't be intercepted over network
✓ Each system can have own key
✓ Keys can be rotated
✓ Passphrases protect local key storage
    """)

setup_ssh_keys_secure()
```

### 2.3 File Integrity Monitoring

```python
import hashlib
import json
import os

class FileIntegrityMonitor:
    """Monitor files for unauthorized changes"""
    
    def __init__(self, config_file="fim_config.json"):
        self.config_file = config_file
        self.hashes = {}
    
    def calculate_hash(self, filepath):
        """Calculate file hash"""
        hasher = hashlib.sha256()
        with open(filepath, 'rb') as f:
            while chunk := f.read(8192):
                hasher.update(chunk)
        return hasher.hexdigest()
    
    def baseline(self, directory):
        """Create baseline of file hashes"""
        print(f"[*] Creating baseline for {directory}...")
        
        for root, dirs, files in os.walk(directory):
            for file in files:
                filepath = os.path.join(root, file)
                try:
                    file_hash = self.calculate_hash(filepath)
                    self.hashes[filepath] = file_hash
                    print(f"  [+] {filepath}")
                except Exception as e:
                    print(f"  [-] Error: {filepath} - {e}")
        
        self.save_baseline()
        print(f"[+] Baseline saved ({len(self.hashes)} files)")
    
    def save_baseline(self):
        """Save hashes to file"""
        with open(self.config_file, 'w') as f:
            json.dump(self.hashes, f, indent=2)
    
    def load_baseline(self):
        """Load previous hashes"""
        if os.path.exists(self.config_file):
            with open(self.config_file, 'r') as f:
                self.hashes = json.load(f)
    
    def verify(self, directory):
        """Verify files haven't changed"""
        self.load_baseline()
        print(f"[*] Verifying {directory}...")
        
        changes = []
        for filepath, old_hash in self.hashes.items():
            if not os.path.exists(filepath):
                print(f"[!] DELETED: {filepath}")
                changes.append(("deleted", filepath))
            else:
                new_hash = self.calculate_hash(filepath)
                if new_hash != old_hash:
                    print(f"[!] MODIFIED: {filepath}")
                    changes.append(("modified", filepath))
        
        # Check for new files
        for root, dirs, files in os.walk(directory):
            for file in files:
                filepath = os.path.join(root, file)
                if filepath not in self.hashes:
                    print(f"[!] NEW: {filepath}")
                    changes.append(("new", filepath))
        
        return changes

# Usage
# fim = FileIntegrityMonitor()
# fim.baseline("/etc")  # Create baseline
# fim.verify("/etc")    # Check for changes
```

---

## Module 3: Network Security

### 3.1 Firewall Rules (iptables/ufw)

```bash
# UFW (Uncomplicated Firewall) - Beginner Friendly

# Enable firewall
sudo ufw enable

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow specific services
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
sudo ufw allow 3306/tcp from 192.168.1.0/24  # MySQL from internal only

# Block specific ports
sudo ufw deny 23/tcp  # Block Telnet
sudo ufw deny 21/tcp  # Block FTP

# Delete rules
sudo ufw delete allow ssh
sudo ufw delete deny 23/tcp

# View firewall status
sudo ufw status
sudo ufw status verbose
```

### 3.2 Intrusion Detection System (IDS)

```python
class SimpleIDS:
    """Basic Intrusion Detection System"""
    
    def __init__(self):
        self.alert_threshold = 100  # Connection attempts per minute
        self.blocked_ips = set()
    
    def monitor_connections(self, log_file="/var/log/auth.log"):
        """Monitor authentication attempts"""
        print("[*] Monitoring authentication attempts...")
        
        failed_attempts = {}
        
        try:
            with open(log_file, 'r') as f:
                for line in f:
                    if "Failed password" in line or "Invalid user" in line:
                        # Extract IP address
                        import re
                        match = re.search(r'\d+\.\d+\.\d+\.\d+', line)
                        if match:
                            ip = match.group()
                            failed_attempts[ip] = failed_attempts.get(ip, 0) + 1
                            
                            # Alert on suspicious activity
                            if failed_attempts[ip] > 5:
                                print(f"[!] ALERT: {ip} - {failed_attempts[ip]} failed attempts")
                                
                                if failed_attempts[ip] > 10:
                                    print(f"[!] BLOCKING: {ip}")
                                    self.block_ip(ip)
        except Exception as e:
            print(f"[-] Error reading log: {e}")
    
    def block_ip(self, ip):
        """Block suspicious IP"""
        self.blocked_ips.add(ip)
        print(f"  [+] Added {ip} to blocklist")
        # In production: run: sudo ufw deny from {ip}
    
    def get_blocked_ips(self):
        """Return list of blocked IPs"""
        return self.blocked_ips

# Usage
# ids = SimpleIDS()
# ids.monitor_connections()
```

### 3.3 DDoS Mitigation

```python
def ddos_protection_strategies():
    """DDoS protection strategies"""
    
    strategies = {
        "Rate Limiting": {
            "description": "Limit requests per IP/second",
            "implementation": "Use WAF or application-level throttling",
            "tools": "nginx rate_limit, f2b, mod_ratelimit",
        },
        "Traffic Filtering": {
            "description": "Filter malicious traffic patterns",
            "implementation": "Firewall rules to drop suspicious packets",
            "tools": "iptables, pf, cloud WAF",
        },
        "CDN/DDoS Mitigation": {
            "description": "Use content delivery network",
            "implementation": "Cloudflare, Akamai, AWS Shield",
            "tools": "Third-party DDoS protection services",
        },
        "Black Hole Routing": {
            "description": "Route attack traffic to null route",
            "implementation": "Automated traffic diversion",
            "tools": "BGP blackholing",
        },
        "Geo-Blocking": {
            "description": "Block traffic from specific countries",
            "implementation": "GeoIP-based filtering",
            "tools": "MaxMind GeoIP, nginx geo module",
        },
    }
    
    print("[!] DDoS PROTECTION STRATEGIES\n")
    for strategy, details in strategies.items():
        print(f"{strategy}:")
        print(f"  Description: {details['description']}")
        print(f"  Implementation: {details['implementation']}")
        print(f"  Tools: {details['tools']}\n")

ddos_protection_strategies()
```

---

## Module 4: Application Security

### 4.1 Secure Coding Practices

```python
# VULNERABLE vs SECURE Examples

# 1. INPUT VALIDATION
# VULNERABLE
@app.route('/search/<query>')
def search(query):
    return render_template_string(f"<h1>Results: {query}</h1>")

# SECURE
@app.route('/search/<query>')
def search(query):
    if len(query) > 100:
        return "Query too long", 400
    safe_query = escape(query)
    return render_template_string(f"<h1>Results: {safe_query}</h1>")

# 2. AUTHENTICATION
# VULNERABLE
if username == stored_user and password == stored_pass:
    login_user()

# SECURE
if username == stored_user and check_password_hash(stored_hash, password):
    login_user()

# 3. SESSION MANAGEMENT
# VULNERABLE
session_id = username  # Predictable!

# SECURE
import secrets
session_id = secrets.token_urlsafe(32)  # Cryptographically random

# 4. ERROR HANDLING
# VULNERABLE
try:
    db.query(user_input)
except Exception as e:
    return f"Error: {e}"  # Reveals database info!

# SECURE
try:
    db.query(user_input)
except Exception as e:
    logger.error(f"Database error: {e}")  # Log internally
    return "An error occurred"  # Generic response
```

### 4.2 Security Headers

```python
from flask import Flask

app = Flask(__name__)

@app.after_request
def set_security_headers(response):
    """Add security headers to all responses"""
    
    # Prevent clickjacking
    response.headers['X-Frame-Options'] = 'SAMEORIGIN'
    
    # Prevent MIME type sniffing
    response.headers['X-Content-Type-Options'] = 'nosniff'
    
    # Enable XSS protection
    response.headers['X-XSS-Protection'] = '1; mode=block'
    
    # Content Security Policy
    response.headers['Content-Security-Policy'] = "default-src 'self'"
    
    # HSTS (enforce HTTPS)
    response.headers['Strict-Transport-Security'] = 'max-age=31536000; includeSubDomains'
    
    # Referrer Policy
    response.headers['Referrer-Policy'] = 'strict-origin-when-cross-origin'
    
    return response

@app.route('/')
def index():
    return "Secure response with headers"
```

---

## Module 5: Incident Response

### 5.1 Incident Response Flowchart

```
Incident Detected
    |
    v
[IMMEDIATE RESPONSE]
  - Isolate affected systems
  - Preserve evidence/logs
  - Activate incident team
    |
    v
[INVESTIGATION]
  - Determine incident scope
  - Identify entry point
  - Find affected assets
  - Gather forensic data
    |
    v
[CONTAINMENT]
  - Stop ongoing attack
  - Patch vulnerabilities
  - Reset compromised credentials
  - Update firewall rules
    |
    v
[ERADICATION]
  - Remove all malware
  - Close all backdoors
  - Rebuild systems if needed
  - Verify clean state
    |
    v
[RECOVERY]
  - Restore from backups
  - Monitor closely
  - Bring systems back online
  - Verify functionality
    |
    v
[POST-INCIDENT]
  - Write incident report
  - Update security policies
  - Conduct training
  - Implement improvements
```

### 5.2 Incident Response Kit

```python
def incident_response_checklist():
    """Emergency response checklist"""
    
    print("""[!] INCIDENT RESPONSE CHECKLIST

IMMEDIATE (First 30 minutes):
□ Declare incident
□ Activate incident response team
□ Set up war room/communication channel
□ Determine incident severity
□ Identify affected systems
□ Preserve system logs
□ Take system snapshots
□ Disconnect compromised systems (if needed)

INVESTIGATION (First 24 hours):
□ Analyze system logs
□ Check for malware/backdoors
□ Review network traffic
□ Identify entry point
□ Determine what data was accessed
□ Map lateral movement
□ Gather timeline of events
□ Document all findings

CONTAINMENT:
□ Patch vulnerable systems
□ Reset all passwords
□ Block attacker IPs
□ Remove malware
□ Close compromised accounts
□ Update firewall rules
□ Review access controls

ERADICATION:
□ Remove all malicious code
□ Verify systems are clean
□ Apply patches
□ Update configurations
□ Review security settings

RECOVERY:
□ Restore from clean backups
□ Rebuild affected systems
□ Restore data carefully
□ Monitor for re-infection
□ Verify system integrity

POST-INCIDENT:
□ Write detailed report
□ Calculate impact/damages
□ Update incident response plan
□ Implement preventative measures
□ Conduct security awareness training
□ Schedule security audit
    """)

incident_response_checklist()
```

---

## Module 6: Lab Exercise - Build a Secure Web App

```python
from flask import Flask, request, render_template_string, escape
from werkzeug.security import generate_password_hash, check_password_hash
import secrets
import re

app = Flask(__name__)
app.secret_key = secrets.token_hex(32)

# Simulated database
users_db = {}
sessions = {}

def validate_input(data, max_length=100):
    """Validate and sanitize user input"""
    if not data or len(data) > max_length:
        return None
    # Only alphanumeric and basic characters
    if not re.match(r'^[a-zA-Z0-9@._-]+$', data):
        return None
    return data

@app.route('/register', methods=['POST'])
def register():
    """Secure user registration"""
    username = validate_input(request.form.get('username', ''), 50)
    password = request.form.get('password', '')
    
    if not username or not password or len(password) < 8:
        return "Invalid credentials", 400
    
    if username in users_db:
        return "User exists", 400
    
    # Hash password
    password_hash = generate_password_hash(password, method='pbkdf2:sha256')
    users_db[username] = password_hash
    
    return "Registration successful", 201

@app.route('/login', methods=['POST'])
def login():
    """Secure login with rate limiting"""
    username = validate_input(request.form.get('username', ''), 50)
    password = request.form.get('password', '')
    
    if not username or username not in users_db:
        return "Invalid credentials", 401
    
    if not check_password_hash(users_db[username], password):
        return "Invalid credentials", 401
    
    # Create session
    session_id = secrets.token_urlsafe(32)
    sessions[session_id] = username
    
    return {"session": session_id}, 200

@app.after_request
def set_security_headers(response):
    """Add security headers"""
    response.headers['X-Frame-Options'] = 'SAMEORIGIN'
    response.headers['X-Content-Type-Options'] = 'nosniff'
    response.headers['X-XSS-Protection'] = '1; mode=block'
    return response

if __name__ == '__main__':
    app.run(ssl_context='adhoc')  # Force HTTPS
```

---

## Day 4 Summary

✓ Learned defense-in-depth principles
✓ Mastered server hardening techniques
✓ Understood network security
✓ Learned secure coding practices
✓ Can respond to security incidents

**Homework:**
1. Harden a Linux server completely
2. Create incident response plan for your organization
3. Review and update firewall rules
4. Implement security headers in a web app

---

## Key Defense Principles

🛡️ **Remember:**
- Assume you WILL be attacked
- Multiple layers of defense are needed
- Monitor everything
- Log everything
- Respond quickly
- Update continuously
- Train your team
- Defense is ONGOING, not one-time
