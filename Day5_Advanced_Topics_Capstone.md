# Day 5: Advanced Topics & Capstone Project

## Training Objective
Master advanced cybersecurity concepts and apply all learned skills in a comprehensive capstone project.

---

## Module 1: Cryptography Fundamentals

### 1.1 Symmetric Encryption (Same key for encrypt/decrypt)

```python
from cryptography.fernet import Fernet
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.backends import default_backend
import os

class SymmetricEncryption:
    """Symmetric encryption examples"""
    
    @staticmethod
    def simple_encryption():
        """Use Fernet for simple, secure encryption"""
        # Generate key
        key = Fernet.generate_key()
        print(f"[+] Generated key: {key}")
        
        # Create cipher
        cipher = Fernet(key)
        
        # Encrypt data
        plaintext = b"Sensitive data that needs protection"
        ciphertext = cipher.encrypt(plaintext)
        print(f"[+] Encrypted: {ciphertext}")
        
        # Decrypt data
        decrypted = cipher.decrypt(ciphertext)
        print(f"[+] Decrypted: {decrypted}")
        
        return key
    
    @staticmethod
    def encrypt_file(filename, key):
        """Encrypt entire file"""
        cipher = Fernet(key)
        
        with open(filename, 'rb') as f:
            plaintext = f.read()
        
        ciphertext = cipher.encrypt(plaintext)
        
        with open(f"{filename}.encrypted", 'wb') as f:
            f.write(ciphertext)
        
        print(f"[+] File encrypted: {filename}.encrypted")

# Usage
# key = SymmetricEncryption.simple_encryption()
# SymmetricEncryption.encrypt_file("secret.txt", key)
```

### 1.2 Asymmetric Encryption (Public/Private Keys)

```python
from cryptography.hazmat.primitives.asymmetric import rsa, padding
from cryptography.hazmat.primitives import serialization, hashes
from cryptography.hazmat.backends import default_backend

class AsymmetricEncryption:
    """RSA public key encryption"""
    
    @staticmethod
    def generate_keys():
        """Generate RSA key pair"""
        # Generate private key
        private_key = rsa.generate_private_key(
            public_exponent=65537,
            key_size=2048,
            backend=default_backend()
        )
        
        # Extract public key
        public_key = private_key.public_key()
        
        # Save keys to files
        with open("private_key.pem", "wb") as f:
            f.write(private_key.private_bytes(
                encoding=serialization.Encoding.PEM,
                format=serialization.PrivateFormat.PKCS8,
                encryption_algorithm=serialization.NoEncryption()
            ))
        
        with open("public_key.pem", "wb") as f:
            f.write(public_key.public_bytes(
                encoding=serialization.Encoding.PEM,
                format=serialization.PublicFormat.SubjectPublicKeyInfo
            ))
        
        print("[+] Keys generated and saved")
        return private_key, public_key
    
    @staticmethod
    def encrypt_with_public_key(public_key, message):
        """Encrypt message with public key"""
        ciphertext = public_key.encrypt(
            message.encode(),
            padding.OAEP(
                mgf=padding.MGF1(algorithm=hashes.SHA256()),
                algorithm=hashes.SHA256(),
                label=None
            )
        )
        return ciphertext
    
    @staticmethod
    def decrypt_with_private_key(private_key, ciphertext):
        """Decrypt message with private key"""
        plaintext = private_key.decrypt(
            ciphertext,
            padding.OAEP(
                mgf=padding.MGF1(algorithm=hashes.SHA256()),
                algorithm=hashes.SHA256(),
                label=None
            )
        )
        return plaintext.decode()

# Usage
# private_key, public_key = AsymmetricEncryption.generate_keys()
# encrypted = AsymmetricEncryption.encrypt_with_public_key(public_key, "Secret message")
# decrypted = AsymmetricEncryption.decrypt_with_private_key(private_key, encrypted)
```

### 1.3 Hashing (One-way, for integrity/verification)

```python
import hashlib
import hmac

class Hashing:
    """Hash functions for data integrity"""
    
    @staticmethod
    def password_hashing(password):
        """Hash password securely"""
        # Using SHA-256 (for demo; use bcrypt in production)
        salt = os.urandom(32)
        pwdhash = hashlib.pbkdf2_hmac('sha256', password.encode(), salt, 100000)
        
        print(f"[+] Salt: {salt.hex()}")
        print(f"[+] Hash: {pwdhash.hex()}")
        
        return salt, pwdhash
    
    @staticmethod
    def verify_password(password, salt, stored_hash):
        """Verify password against stored hash"""
        pwdhash = hashlib.pbkdf2_hmac('sha256', password.encode(), salt, 100000)
        return hmac.compare_digest(pwdhash, stored_hash)
    
    @staticmethod
    def file_integrity(filename):
        """Calculate file hash for integrity verification"""
        hasher = hashlib.sha256()
        
        with open(filename, 'rb') as f:
            while chunk := f.read(8192):
                hasher.update(chunk)
        
        print(f"[+] SHA256: {hasher.hexdigest()}")
        return hasher.hexdigest()

# Usage
# salt, hash_val = Hashing.password_hashing("mypassword")
# Hashing.verify_password("mypassword", salt, hash_val)
# Hashing.file_integrity("document.pdf")
```

---

## Module 2: Advanced Network Attacks

### 2.1 DNS Spoofing Attack Concept

```python
def explain_dns_attack():
    """DNS attack explanation (educational)"""
    
    print("""[!] DNS SPOOFING ATTACK

NORMAL DNS RESOLUTION:
1. User: "What's the IP for google.com?"
2. DNS Server: "8.8.8.8"
3. User connects to 8.8.8.8

ATTACK SCENARIO:
1. Attacker intercepts DNS query
2. Attacker responds before legitimate DNS server
3. Responds with attacker's IP (fake.attacker.com)
4. User thinks attacker's site is google.com
5. User connects to attacker instead

PREVENTION:
- DNSSEC (cryptographic signing of DNS records)
- Use trusted DNS servers (8.8.8.8, 1.1.1.1)
- DNS query monitoring
- Update DNS cache regularly
    """)

# Prevention example
def implement_dnssec():
    """DNSSEC verification"""
    import subprocess
    
    print("""[+] DNSSEC VALIDATION:
    
    Use dig to verify DNSSEC:
    $ dig +dnssec google.com
    
    Look for ad flag (DNSSEC validated):
    ;; flags: qr rd ra ad;
    
    Or check online DNSSEC validators
    """)

explain_dns_attack()
```

### 2.2 Certificate-Based Attacks

```python
def explain_ssl_attacks():
    """SSL/TLS attack scenarios"""
    
    print("""[!] SSL/TLS ATTACKS

1. EXPIRED CERTIFICATE
   - Attacker uses old, stolen certificate
   - User browser may not check expiration
   - Defense: Always verify certificate dates

2. SELF-SIGNED CERTIFICATE
   - Attacker uses self-signed cert for google.com
   - Not signed by trusted CA
   - Defense: Modern browsers warn about this

3. CERTIFICATE PINNING
   - App/browser "pins" expected certificate
   - Any other cert is rejected
   - Prevents MITM attacks
   - Implementation: Hardcode public key hash

4. HEARTBLEED (CVE-2014-0160)
   - OpenSSL vulnerability leaked memory
   - Could extract private keys
   - Defense: Update OpenSSL immediately

PROTECTION:
✓ Use HTTPS everywhere
✓ Verify SSL certificates
✓ Use certificate pinning for critical apps
✓ Keep OpenSSL/TLS updated
✓ Use strong ciphers (TLS 1.2+)
✓ Monitor certificate expiration
    """)

explain_ssl_attacks()
```

### 2.3 Zero-Day Vulnerability

```python
def zero_day_explanation():
    """Understanding zero-day vulnerabilities"""
    
    print("""[!] ZERO-DAY VULNERABILITY LIFECYCLE

VULNERABILITY DISCOVERED:
- Attacker finds critical bug in software
- Vendor is UNAWARE of vulnerability
- NO PATCH AVAILABLE YET
- "Zero days" = days before public disclosure

EXPLOITATION WINDOW:
- Attacker exploits unpatched systems
- Vendor doesn't know about it
- Users have NO DEFENSE
- Very dangerous and valuable

TIMELINE EXAMPLE:
Day 0: Attacker discovers Windows privilege escalation
Days 0-90: Attacker uses it silently
Day 30: Security researcher discovers it
Day 45: Vendor releases emergency patch
Day 90: Public disclosure

DEFENSE STRATEGIES:
1. Assume you can't rely on vendor patches
2. Implement compensating controls:
   - Network segmentation
   - Endpoint monitoring
   - Firewall rules
   - Application whitelisting
3. Monitor for suspicious activity
4. Have incident response ready
5. Keep systems updated (patches other issues)
6. Use intrusion prevention systems

FAMOUS ZERO-DAYS:
- Stuxnet (2010): Nuclear facility attack
- EternalBlue (2017): WannaCry ransomware
- Log4j (2021): Critical logging library
    """)

zero_day_explanation()
```

---

## Module 3: Threat Intelligence & Threat Hunting

### 3.1 Threat Intelligence Framework

```python
def threat_intelligence_process():
    """Threat intelligence workflow"""
    
    print("""[+] THREAT INTELLIGENCE PROCESS

PHASE 1: COLLECTION
- Monitor security news
- Review vendor advisories
- Track hacker forums
- Analyze leaked databases
- Monitor dark web
- Follow security researchers
Sources: SecurityFocus, CVE Database, Twitter, OSINT

PHASE 2: ANALYSIS
- Categorize threats
- Identify targets
- Assess capabilities
- Evaluate motivations
- Determine techniques (MITRE ATT&CK)

PHASE 3: DISSEMINATION
- Create actionable intelligence
- Share with stakeholders
- Update detection rules
- Brief executive team
- Coordinate with vendors

PHASE 4: FEEDBACK
- Measure impact
- Refine intelligence
- Update indicators of compromise (IoCs)
- Adjust defensive posture

ACTIONABLE INTELLIGENCE EXAMPLE:
[+] Threat: APT-28 targeting energy sector
    Timeline: January 2024
    Technique: Spear-phishing with MACRO malware
    Indicators: 
        - C2 domain: badactor.ru
        - Email regex: *@energy*.com
        - File hash: abc123def456...
    Recommendation: Block domain, train on macro risks, monitor for Cobalt Strike
    """)

threat_intelligence_process()
```

### 3.2 Threat Hunting

```python
import re
from datetime import datetime

class ThreatHunter:
    """Proactive threat hunting"""
    
    def __init__(self, log_file):
        self.log_file = log_file
        self.suspicious_indicators = []
    
    def hunt_for_lateral_movement(self):
        """Hunt for signs of lateral movement"""
        print("[*] Hunting for lateral movement indicators...\n")
        
        indicators = {
            "SMB Activity": r"SMB.*445",
            "RDP Activity": r"RDP.*3389",
            "Kerberos Tickets": r"Kerberos.*TGT",
            "Pass the Hash": r"NTLM.*reuse",
            "Process Injection": r"CreateRemoteThread",
            "Service Installation": r"Service.*installed",
        }
        
        for indicator_name, pattern in indicators.items():
            print(f"  [*] Checking for: {indicator_name}")
            print(f"      Pattern: {pattern}\n")
    
    def hunt_for_data_exfiltration(self):
        """Hunt for data theft patterns"""
        print("[*] Hunting for data exfiltration indicators...\n")
        
        suspicious_patterns = [
            ("Large file transfer", r"(transfer|download).*[0-9]{7,} bytes"),
            ("Unusual protocols", r"(FTP|SFTP|DNS|ICMP).*large.*data"),
            ("Cloud upload", r"(dropbox|aws|azure|gdrive).*upload"),
            ("USB activity", r"USB.*mass.*storage"),
            ("Backup activity", r"backup.*unusual.*time"),
        ]
        
        for pattern_name, pattern in suspicious_patterns:
            print(f"  [*] {pattern_name}: {pattern}")
        print()
    
    def generate_hunting_report(self):
        """Generate threat hunting report"""
        report = f"""
[+] THREAT HUNTING REPORT
Date: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}

HYPOTHESIS: 
Attackers have compromised internal systems and are moving laterally

INDICATORS HUNTED:
✓ Lateral movement patterns
✓ Data exfiltration
✓ Command execution
✓ Persistence mechanisms
✓ Privilege escalation

FINDINGS:
[!] Suspicious SMB activity from 192.168.1.100
[!] Large data transfer to external IP 10.0.0.1
[!] Multiple failed RDP attempts
[!] Unusual process execution: powershell.exe with base64 args

RECOMMENDATIONS:
1. Isolate affected systems
2. Preserve logs and memory
3. Analyze process memory dumps
4. Check for backdoors
5. Reset credentials
        """
        return report

# Usage
# hunter = ThreatHunter("auth.log")
# hunter.hunt_for_lateral_movement()
# print(hunter.generate_hunting_report())
```

---

## Module 4: Compliance & Regulations

### 4.1 Common Compliance Frameworks

```python
def compliance_frameworks():
    """Security compliance standards"""
    
    print("""[!] COMPLIANCE & REGULATORY FRAMEWORKS

1. GDPR (General Data Protection Regulation) - EU
   - Applies to EU residents' data
   - Requirements:
     ✓ Data protection by design
     ✓ User consent for data processing
     ✓ Right to be forgotten
     ✓ Data breach notification (72 hours)
     ✓ Data Protection Officer required
   - Penalties: Up to 4% of annual revenue or €20M

2. HIPAA (Health Insurance Portability) - USA
   - Applies to healthcare providers
   - Requirements:
     ✓ Encryption of PHI (Protected Health Info)
     ✓ Access controls
     ✓ Audit logs
     ✓ Incident response plan
   - Penalties: Up to $1.5M per violation category

3. PCI DSS (Payment Card Industry) - Financial
   - Applies to credit card processing
   - Requirements:
     ✓ Network segmentation
     ✓ Encryption of cardholder data
     ✓ Regular vulnerability scanning
     ✓ Regular security testing
   - Penalties: Visa/MC fines, card processing suspension

4. SOC 2 (Service Organization Control)
   - Applies to SaaS/cloud providers
   - Audits for:
     ✓ Security
     ✓ Availability
     ✓ Processing Integrity
     ✓ Confidentiality
   - Type I: Point in time assessment
   - Type II: 6-month evaluation

5. ISO 27001 (Information Security Management)
   - International standard
   - Requirements:
     ✓ Information security policy
     ✓ Risk assessment
     ✓ Access control
     ✓ Cryptography
     ✓ Physical security
   - Certification available

6. NIST Cybersecurity Framework
   - US government standard
   - Five functions:
     ✓ Identify risks
     ✓ Protect systems
     ✓ Detect incidents
     ✓ Respond to incidents
     ✓ Recover from incidents
    """)

compliance_frameworks()
```

---

## Module 5: Capstone Project - Security Assessment Report

### 5.1 Project Overview

```
CAPSTONE PROJECT: Complete Security Assessment

SCENARIO:
You are hired to conduct a security assessment of a company's infrastructure.
You must:
1. Reconnaissance
2. Vulnerability scanning
3. Analysis and prioritization
4. Report generation
5. Remediation recommendations

DELIVERABLES:
- Executive summary
- Detailed findings with CVSS scores
- Risk matrix
- Remediation timeline
- Budget for improvements
```

### 5.2 Capstone Project Template

```python
class SecurityAssessment:
    """Complete security assessment framework"""
    
    def __init__(self, client_name, assessment_date):
        self.client_name = client_name
        self.assessment_date = assessment_date
        self.findings = []
        self.risk_summary = {"Critical": 0, "High": 0, "Medium": 0, "Low": 0}
    
    def add_finding(self, finding_dict):
        """Add vulnerability finding"""
        required_fields = [
            "title", "severity", "description", "cvss_score",
            "affected_systems", "business_impact", "recommendation"
        ]
        
        if all(field in finding_dict for field in required_fields):
            self.findings.append(finding_dict)
            severity = finding_dict["severity"]
            if severity in self.risk_summary:
                self.risk_summary[severity] += 1
            return True
        return False
    
    def generate_executive_summary(self):
        """Generate executive summary"""
        total_findings = len(self.findings)
        
        summary = f"""
╔══════════════════════════════════════════════════════════════╗
║         SECURITY ASSESSMENT EXECUTIVE SUMMARY              ║
╚══════════════════════════════════════════════════════════════╝

CLIENT: {self.client_name}
DATE: {self.assessment_date}
ASSESSMENT TYPE: Full Network Security Assessment

RISK SUMMARY:
  Critical: {self.risk_summary['Critical']} vulnerabilities
  High:     {self.risk_summary['High']} vulnerabilities
  Medium:   {self.risk_summary['Medium']} vulnerabilities
  Low:      {self.risk_summary['Low']} vulnerabilities
  ─────────────────────────────
  TOTAL:    {total_findings} vulnerabilities

OVERALL RISK LEVEL: {'CRITICAL' if self.risk_summary['Critical'] > 0 else 'HIGH' if self.risk_summary['High'] > 0 else 'MEDIUM'}

KEY FINDINGS:
{self._top_findings()}

REMEDIATION PRIORITY:
1. Immediately address all CRITICAL findings
2. Address HIGH findings within 30 days
3. Address MEDIUM findings within 90 days
4. Address LOW findings within 6 months

ESTIMATED REMEDIATION COST: $25,000 - $50,000
        """
        return summary
    
    def _top_findings(self):
        """Get top 5 findings"""
        sorted_findings = sorted(self.findings, 
                               key=lambda x: self._severity_value(x["severity"]),
                               reverse=True)
        
        output = ""
        for i, finding in enumerate(sorted_findings[:5], 1):
            output += f"\n{i}. {finding['title']} [{finding['severity']}]\n"
            output += f"   CVSS Score: {finding['cvss_score']}\n"
        
        return output
    
    def _severity_value(self, severity):
        """Convert severity to numeric value for sorting"""
        values = {"Critical": 4, "High": 3, "Medium": 2, "Low": 1}
        return values.get(severity, 0)
    
    def generate_detailed_report(self):
        """Generate full technical report"""
        report = self.generate_executive_summary()
        report += "\n\n" + "="*60 + "\n"
        report += "DETAILED FINDINGS\n"
        report += "="*60 + "\n"
        
        for i, finding in enumerate(self.findings, 1):
            report += f"""
┌─ FINDING {i}: {finding['title']} ─────────────────────────┐
│
│ SEVERITY: {finding['severity']}
│ CVSS SCORE: {finding['cvss_score']}/10
│
│ DESCRIPTION:
│ {finding['description']}
│
│ AFFECTED SYSTEMS:
│ {finding['affected_systems']}
│
│ BUSINESS IMPACT:
│ {finding['business_impact']}
│
│ RECOMMENDATION:
│ {finding['recommendation']}
│
└──────────────────────────────────────────────────────────────┘
            """
        
        return report
    
    def generate_risk_matrix(self):
        """Generate risk assessment matrix"""
        matrix = """
RISK MATRIX (Likelihood x Impact)

                 LIKELIHOOD
           Low    Medium    High    Critical
       ┌────────────────────────────────────┐
       │                                    │
   I   │  Low   Medium   High   Critical   │
   M   │                                    │
   P   │  Low    Low    Medium    High      │
   A   │                                    │
   C   │ Medium Medium   High   Critical   │
   T   │                                    │
       │  High   High   Critical Critical  │
       │                                    │
       └────────────────────────────────────┘

ACTION REQUIRED BY RISK LEVEL:
  Critical: Immediate remediation (24 hours)
  High:     Urgent remediation (1-2 weeks)
  Medium:   Plan remediation (1-3 months)
  Low:      Standard remediation (6 months)
        """
        return matrix

# USAGE EXAMPLE
def capstone_example():
    """Run capstone project example"""
    
    assessment = SecurityAssessment("ACME Corporation", "2024-01-15")
    
    # Add findings
    assessment.add_finding({
        "title": "SQL Injection in Login Form",
        "severity": "Critical",
        "description": "Login form vulnerable to SQL injection attacks",
        "cvss_score": 9.8,
        "affected_systems": "Web application (prod), Database backend",
        "business_impact": "Complete database compromise, user data theft",
        "recommendation": "Use parameterized queries, conduct code review"
    })
    
    assessment.add_finding({
        "title": "Weak SSH Configuration",
        "severity": "High",
        "description": "SSH allows password authentication and root login",
        "cvss_score": 8.2,
        "affected_systems": "All Linux servers (prod & dev)",
        "business_impact": "Unauthorized access, lateral movement",
        "recommendation": "Disable password auth, disable root login, use key-based auth"
    })
    
    assessment.add_finding({
        "title": "Missing Security Headers",
        "severity": "Medium",
        "description": "Web application missing XSS and clickjacking protections",
        "cvss_score": 5.3,
        "affected_systems": "Web application",
        "business_impact": "XSS attacks, account takeover",
        "recommendation": "Add CSP, X-Frame-Options, X-Content-Type-Options headers"
    })
    
    # Print reports
    print(assessment.generate_executive_summary())
    print(assessment.generate_risk_matrix())
    print(assessment.generate_detailed_report())

# Run example
# capstone_example()
```

---

## Day 5 Summary & Graduation

✅ **You've completed the 5-day intensive cybersecurity training!**

### Skills Acquired:
✓ Network reconnaissance and scanning
✓ Vulnerability identification and exploitation (ethical)
✓ Penetration testing methodology
✓ Defensive security practices
✓ Incident response procedures
✓ Cryptography fundamentals
✓ Security assessment reporting
✓ Compliance understanding

### Next Steps:

1. **Certifications:**
   - CompTIA Security+
   - Certified Ethical Hacker (CEH)
   - OSCP (Offensive Security Certified Professional)
   - GIAC Security Essentials (GSEC)

2. **Hands-On Practice:**
   - HackTheBox.com (practice labs)
   - TryHackMe.com (guided labs)
   - PicoCTF (capture the flag)
   - BugBounty.jp (real-world vulnerabilities)

3. **Career Path:**
   - Security Operations Center (SOC) Analyst
   - Penetration Tester
   - Security Engineer
   - Incident Response Specialist
   - Chief Information Security Officer (CISO)

4. **Continuous Learning:**
   - Follow security news daily
   - Join security communities
   - Participate in CTF competitions
   - Contribute to open-source security projects
   - Read security research papers

---

## Professional Code of Ethics

As a cybersecurity professional, you commit to:

🔐 **INTEGRITY**
- Use skills only for authorized purposes
- Protect client confidentiality
- Report findings honestly
- Admit limitations

🛡️ **RESPONSIBILITY**
- Follow laws and regulations
- Respect privacy and civil liberties
- Minimize collateral damage
- Disclose vulnerabilities responsibly

👥 **PROFESSIONALISM**
- Maintain high technical standards
- Share knowledge to improve industry
- Mentor junior professionals
- Continue education

⚖️ **ACCOUNTABILITY**
- Accept responsibility for actions
- Follow established guidelines
- Report ethical violations
- Support consequences

---

## National Cybersecurity Commitment

🇳🇦 **You are now equipped to:**
- Defend our nation's critical infrastructure
- Protect citizen data and privacy
- Secure governmental systems
- Respond to cyber threats
- Build resilient infrastructure

**Remember:** These powerful skills must always be used to PROTECT, DEFEND, and BUILD - never to attack, steal, or harm.

Your training is complete. Your responsibility begins now.

**Welcome to the cybersecurity profession. Use your power wisely. 🛡️**
