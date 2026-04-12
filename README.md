🛡️blue-team-labs
> Defensive security lab write-ups 
All labs conducted in authorised environments — Swiss Cyber Institute lab targets, own infrastructure, and training platforms.
No unauthorised systems were accessed. All work complies with Swiss law and ethical hacking standards.
---

## 📁 Labs

| # | Lab | Tools | Status |
|---|---|---|---|
| 01 | Network Traffic Forensics — Phishing Attack Investigation | Wireshark · TShark · VirusTotal | ✅ Complete |
| 02 | Home Network Security Audit | netdiscover · nmap · Hydra · curl | ✅ Complete |
| 03 | Web Application Security — Certificate Analysis | nmap NSE · sslyze · openssl · telnet | ✅ Complete |
| 04 | SIEM & Endpoint Detection (Wazuh) | Wazuh v4.14.3 · OpenSearch (internal) · systemctl · SSH | ✅ Complete |
| 05 | Email Security Gateway — Proxmox Mail Gateway | Docker · Postfix · PMG · swaks · Thunderbird | 🔧 In Progress |

---

### 01 · Network Traffic Forensics - Phishing Attack Investigation
Scenario: SCI Network Analysis Module -> Corporate phishing incident customer PII exfiltrated to external server
Investigated a real-world-style incident starting from a known outcome ("customer data on Pastebin")
and traced it backwards through a PCAP file to identify the initial intrusion vector, compromised users,
malware delivery chain, and data exfiltration method.
<br>**Tools:** Wireshark · TShark · VirusTotal · HTTP Object Export
- ✅ SMTP analysis :spoofed phishing email identified (xxxx@hotmail.com)
- ✅ Malicious PDF extracted from PCAP (26 kB) — 41/64 VT detection
- ✅ PDF forensics — embedded JavaScript, auto-execute /OpenAction, obfuscated shellcode
- ✅ CVEs confirmed: CVE-2009-0927 · CVE-2008-2992 (Adobe Reader)
- ✅ 1.2 MB PII exfiltration traced — names, SSNs, credit cards in plaintext HTTP
- ✅ 5-phase attack timeline reconstructed · 20-hour dwell time identified
- ✅ Zero Pastebin results = finding (exfiltration via direct HTTP upload, not Pastebin)

**Wireshark filters and chosen options**
```bash
Phase 1 — Initial Triage
# Protocol hierarchy — bird's-eye view of traffic
# Wireshark: Statistics → Protocol Hierarchy
# Network participants — find all IPs
# Wireshark: Statistics → Conversations → IPv4 tab
# Filter out TCP noise to see UDP/other protocols
#! top
Phase 2 — SMTP Analysis (Phishing Email)
# Filter for SMTP traffic
#smtp
# Follow TCP stream to read full email exchange
# Right-click any SMTP packet → Follow → TCP Stream
# Filter for victim + attacker traffic
#ip.addr == 173.255.XXX.XX || ip.addr == 10.10X.XXX.XX
Phase 3 — HTTP Analysis (Malware Delivery)
# Find who clicked the phishing link
#http
# Filter traffic to malware delivery server
#ip.addr == 14.X.X.XX
# Extract malicious PDF from PCAP
# Wireshark: File → Export Objects → HTTP
# Save: pfqa.php (application/pdf, 26 kB)
Phase 4 — PDF Forensics
# Check file hash
#sha256sum xXXx.php
# Inspect PDF structure in text editor (NEVER open in Adobe Reader)
# cat xXx.php | grep -i "javascript\|openaction\|xxx\|xxx"
# Submit hash to VirusTotal
# https://virustotal.com → search SHA256 $VALUE
Phase 5 — Exfiltration Tracing
# Follow TCP stream on exfiltration connection
# Right-click packet 402 → Follow → TCP Stream
```
Key Finding: Two internal victims compromised. Full names, SSNs, credit card numbers and CVVs
exfiltrated in plaintext ~20 hours post-infection. 

CVEs:
- CVE-2009-0927 (Adobe Reader JS buffer overflow)
- CVE-2008-2992 (util.printf stack overflow)
> 📄 **[Download Full Lab Report (PDF)](https://github.com/jaalso/cybersecurity-portfolio/raw/main/Phishing_Forensics_Writeup_protected.pdf)**
<br>🔒 Password protected — contact me via [LinkedIn](https://linkedin.com/in/jaalso) to request access

---

### 02 · Home Network Security Audit
Scenario: Performed a full recon-to-findings audit on a real home network using the same methodology used in
professional penetration tests — ARP sweep, service enumeration, credential testing, and manual verification.
<br>**Tools:** netdiscover · nmap · Hydra · curl
- ✅ 10 live hosts via ARP sweep — MAC OUI vendor mapping
- ✅ Critical finding: unauthenticated admin panel on Bang & Olufsen Speaker
- ✅ SSL certificate expired **1999** — 26 years ago
- ✅ UPnP enabled on ISP — silent port-opening attack vector
- ✅ Hydra false-positive analysis — 16 "found" passwords, all manually disproved
- ✅ IoT security comparison: Roomba (hardened) vs Speaker BP A9 (critical failures)

**Commands**
```bash
Host Discovery
# sudo netdiscover -r 19X.XXX.X.X/XX -i eth0
# MAC OUI lookup — identifies device vendor from first 3 bytes
Service Enumeration
# gateway scan
# sudo nmap -A -sV 19X.XXX.X.X
# Full port scan on device (default scan showed all filtered)
# sudo nmap -p 0-65535 -Pn --min-rate 500 19X.XXX.X.XXX
SSL certificate details
# sudo nmap --script ssl-cert -Pn 19X.XXX.X.XXX
Manual Probing (curl)
# curl -v http://19X.XXX.X.XXX
Credential Testing (Hydra)
# Brute force 
# hydra -l admin -P /usr/share/wordlists/rockyou.txt \
#       19X.XXX.X.XXX $PROTOCOL-get /
```

**Asset discovery — 10 hosts identified**
<br><img width="335" height="25" alt="image" src="https://github.com/user-attachments/assets/dd25c244-83f6-4742-a8f7-c88e9747109f" />
<br><img width="553" height="255" alt="image" src="https://github.com/user-attachments/assets/6d4cda6f-b5bc-4c0e-8ece-1498657555d2" />


**Speaker — open ports + expired 1999 SSL cert**
<br><img width="486" height="101" alt="image" src="https://github.com/user-attachments/assets/af662602-c415-4325-8c09-144ddf5b696b" />
<br><img width="468" height="161" alt="image" src="https://github.com/user-attachments/assets/2ead66d0-d467-4ec7-8ba2-4993016ab4eb" />


**Unauthenticated admin panel — direct browser access**
<br><img width="404" height="69" alt="image" src="https://github.com/user-attachments/assets/89f716f6-18d8-4b6e-ba40-82a7f99e7a73" />

Key Finding:A device at 19X.XXX.X.XXX exposed a fully unauthenticated admin panel (GoXXX WebServer) with direct access to network settings and firmware
update capability. SSL certificate expired in 1999.
<br>Notable lesson: Hydra reported 16 valid passwords against the router — all false positives caused by
the router returning identical 301 redirects for every request.  

> 📄 **[Download Full Lab Report (PDF)](https://github.com/jaalso/cybersecurity-portfolio/raw/main/HomeNetworkAudit_Writeup_protected.pdf)**
<br>🔒 Password protected — contact me via [LinkedIn](https://linkedin.com/in/jaalso) to request access

---

### 03 · Web Application Security — Certificate Analysis
Scenario: Based on a CSS Exam Challenge: Two-exercise certificate security assessment lab combining port discovery, TLS certificate mapping, in-depth cipher analysis, CSV certificate database auditing, and manual PKI chain revocation verification.
<br>**Tools:** nmap NSE · sslyze · openssl · telnet · csvlook  
- ✅ Full port scan found 6 ports vs 2 with default -F scan
- ✅ 25 cipher suites in TLS 1.2 (vs Mozilla-recommended 7)
- ✅ All 5 trust stores rejected certificate — SAN mismatch
- ✅ CSV audit: 3 critical findings (revoked, SHA-1+RSA1024, self-signed) + 12 expired certs
- ✅ Intermediate CA REVOKED — invalidates every certificate it ever signed
- ✅ Manual CRL revocation chain verified: Root CA → Intermediate CA → End Entity

**Commands**
```bash
Port Discovery And Certificate Mapping
# nmap -p 0-65535 -T4 $HOST
# Extract TLS certificates from all open ports
# nmap --script ssl-cert -p 4443,8080,8081,9990,21021,30443 -Pn $HOST
Deep TLS Analysis
# Deep TLS analysis — cipher suites, trust stores, OCSP
# sslyze $HOST:443
# Port scan with SSL certificate extraction + service version
# nmap -sV -p 443,8443,8080,8888,9443 --script ssl-cert $HOST
CSV Certificate Audit
# Parse CSV for readability
# csvlook -d ';' $CERTIFICATEREPORTNAME.csv
# Single command — catches three critical failure categories
# csvlook -d ';' $CERTIFICATEREPORTNAME.csv | grep -iE "revoked|sha1|1024"
CRL Chain Verification
# Find CRL distribution point URLs
# openssl x509 -in $CERTIFICATE_CA.pem -text -noout | grep -A 3 "CRL Distribution"
# openssl x509 -in $CERTIFICATE.pem -text -noout | grep -A 3 "CRL Distribution"
# Verify end entity certificate (full chain)
# openssl verify -crl_check_all -CAfile $CERTIFICATE_CA.pem \
#                -untrusted $CERTIFICATE_CA.pem \
#                -CRLfile $CERTIFICATE_CA.crl.pem \
#                -CRLfile issuing_ca.crl.pem $CERTIFICATE.pem
```


**Discover All Open Ports - Identify Ports with TLS Certificates**
<br><img width="620" height="164" alt="image" src="https://github.com/user-attachments/assets/2a20f310-fda4-49ae-9936-46b899150dc1" />
<br><img width="549" height="50" alt="image" src="https://github.com/user-attachments/assets/7a897ca7-4848-405f-8c8e-eaa46591882b" />


**Finding Certificate with Subject or Specific Protocol**
<br><img width="220" height="69" alt="image" src="https://github.com/user-attachments/assets/18b5b787-c88f-4f49-8a73-32ef4168a187" />
<br><img width="499" height="127" alt="image" src="https://github.com/user-attachments/assets/49ed8f29-3bee-4aa4-91a5-326004f6ae85" />
<br><img width="552" height="70" alt="image" src="https://github.com/user-attachments/assets/fd06e633-5ccb-40c7-a950-d752fc733bfd" />
<br><img width="333" height="93" alt="image" src="https://github.com/user-attachments/assets/a3390396-7345-4332-aed6-4efd052d8628" />
<br><img width="398" height="160" alt="image" src="https://github.com/user-attachments/assets/4210d4c0-4838-418f-8e01-1ac643dff9a0" />


**In-Depth TLS Analysis — direct browser access - Auditing a Certificate Database for Insecure Certificates**
<br><img width="514" height="295" alt="image" src="https://github.com/user-attachments/assets/72a5af8b-2ccc-4a16-a60b-dcd33ccbfc96" />
<br><img width="589" height="73" alt="image" src="https://github.com/user-attachments/assets/99912577-94f3-4304-9e41-13008883da2d" />
<br><img width="637" height="87" alt="image" src="https://github.com/user-attachments/assets/793e8326-be2a-43af-aa1e-d75d29a9a84c" />
<br>Revocation/OCSP OCSP Stapling: NOT SUPPORTED 
<br><img width="611" height="39" alt="image" src="https://github.com/user-attachments/assets/99be1370-d617-4a8b-bb34-1c226aafd0c1" />
<br><img width="600" height="898" alt="image" src="https://github.com/user-attachments/assets/0acd73f3-1b33-4673-9126-1e179469a247" />

** CRL Revocation Verification of a Certificate Chain**
<br><img width="510" height="240" alt="image" src="https://github.com/user-attachments/assets/bfe8fb38-187d-4e49-a82b-414dc35ca81a" />
<br><img width="584" height="82" alt="image" src="https://github.com/user-attachments/assets/1f4ea08a-8af3-4d99-a53e-d01de59339d5" />
<br><img width="603" height="157" alt="image" src="https://github.com/user-attachments/assets/22047420-c621-4bab-8de2-eea34e97a41f" />
<br><img width="518" height="91" alt="image" src="https://github.com/user-attachments/assets/700297b2-06cd-447d-bc8a-6ff36c92e067" />

> 📄 **[Download Full Lab Report (PDF)](https://github.com/jaalso/cybersecurity-portfolio/raw/main/CertificateAnalysis_Writeup_protected.pdf)**
<br>🔒 Password protected — contact me via [LinkedIn](https://linkedin.com/in/jaalso) to request access

---

### 04 · SIEM & Endpoint Detection — Wazuh Home Lab
Scenario: From SCI Basic Hardening Module, I created a single-agent SIEM home lab deploying Wazuh as an open-source SIEM/XDR/EDR platform.
Covers the full deployment lifecycle — server setup, service recovery, agent enrollment, and live monitoring across 5 security domains mapped to class topics.
<br>**Tools:** VirtualBox · SSH · systemctl · wget · dpkg · nmap
- ✅ Wazuh OVA deployed on VirtualBox — server + Kali Linux agent on host-only network
- ✅ Service recovery — diagnosed wazuh-manager startup timeout (Java indexer RAM contention)
- ✅ Kali Linux enrolled as monitored endpoint — active within 30 seconds of agent start
- ✅ CIS Benchmark automated assessment — 190 controls · 45% score · cross-mapped to ISO 27001 · NIST · PCI-DSS
- ✅ FIM alert triggered — /etc/test-confidential.txt creation detected · 7,958 files monitored
- ✅ 258 events captured — PAM sessions · sudo to ROOT · host-based anomaly detection
- ✅ MITRE ATT&CK correlation — Rule 5402 (T1548 Privilege Escalation) · Rule 5501/5502 (T1078 Valid Accounts)

**Some Commands Used**
```bash
Phase 1 Server Deployment And Verification
# Confirm Wazuh OVA received correct IPs after VirtualBox import
# ip a
# eth0: 19X.1XX.XX.XXX (host-only — lab network)
# eth1: 10.X.X.XX (NAT — internet access)
# Access dashboard
# https://19X.XXX.XX.XXX → admin / admin
Phase 2 Service Recovery (Startup Timeout Fix)
# SSH into Wazuh server
#ssh wazuh-user@19X.XXX.XX.XXX
# Fix — start manager manually after indexer fully initialises
#sudo systemctl start wazuh-manager
#sudo systemctl restart wazuh-dashboard
# Verify all three services running
# sudo systemctl status wazuh-manager wazuh-indexer wazuh-dashboard
Phase 3 Kali Agent Enrollment
# Downloaded and installed Wazuh agent with pre-configured environment variables
# Enabled and started agent
# sudo systemctl daemon-reload
# sudo systemctl enable wazuh-agent        # starts automatically on boot
# sudo systemctl start wazuh-agent
Phase 4 File Integrity Monitoring (FIM)
# Created a test file in monitored /etc/ directory — simulates sensitive data
# Wrote content using echo  | $WORD $LOCATION
# Restarted agent to force immediate FIM scan (default runs on schedule)
#sudo systemctl restart wazuh-agent
# Verified in dashboard: Threat Hunting → search "test-confidential"
# Rule fired: "File added to the system"
Phase 5 Threat Detection (Recon Simulation)
# nmap -sS 12X.X.X.XX                       # triggers T1046 Network Service Discovery alert
# View in dashboard: Threat Hunting → filter by MITRE T1046
# Check what Wazuh sees from Kali's normal operation
# Rule 5402 = sudo to ROOT (T1548 Privilege Escalation)
# Rule 5501/5502 = PAM login sessions (T1078 Valid Accounts)
```


**Lab Results**
<br><img width="600" height="1079" alt="image" src="https://github.com/user-attachments/assets/16f57394-d948-4238-a943-e2dc45c2bbfb" />
<br><img width="600" height="931" alt="image" src="https://github.com/user-attachments/assets/f5790914-01a2-4025-bf23-4d27f3f69c24" />
<br><img width="600" height="911" alt="image" src="https://github.com/user-attachments/assets/f2c7e972-5ec0-4811-9790-9ae6960fa1e0" />
| Metric | Result |
|---|---|
| Total events captured | 258 |
| CIS Benchmark score | 45% (83/190 controls passed) |
| Files monitored (FIM) | 7,958 |
| High CVEs identified | 1 |
| MITRE ATT&CK techniques tagged | T1046 · T1548 · T1078 · T1074 |

**Key Findings**
- Service dependency management — wazuh-indexer RAM contention caused manager startup timeout on every boot. Fix: manual start sequence or ExecStartPre=/bin/sleep 30 in systemd unit. Same class of problem that causes production SIEM outages.
- 45% CIS score is the baseline — expected for a default Kali installation (pentesting distro, not hardened workstation). Each of 99 failing controls has a specific remediation command cross-mapped to ISO 27001, NIST SP 800-53, and PCI-DSS simultaneously.
- Kali's own tools look suspicious — running standard Kali tools while the agent was active triggered Defense Evasion (4 alerts), Privilege Escalation (4 alerts), and Initial Access (2 alerts). This is exactly how enterprise SOCs detect red team activity.

**Next Steps (Planned)**

| Session | Exercise | Goal |
|---|---|---|
| A | `nmap -sS 1XX.X.X.XX` → capture T1046 in Threat Hunting | Recon detection validation |
| B | Hydra brute-force against Wazuh SSH → Rule 5712 spike | Brute force pattern recognition |
| C | Deploy Windows 10 VM agent → compare CIS score vs Kali 45% | Multi-platform SIEM |
| D | Apply CIS remediation → verify score improvement | Full hardening loop |

> 📄 **[Download Full Lab Report (PDF)](https://github.com/jaalso/cybersecurity-portfolio/raw/main/wazuh_lab_report_protected.pdf)**
<br>🔒 Password protected — contact me via [LinkedIn](https://linkedin.com/in/jaalso) to request access

---

### 05 · Email Security Gateway — Proxmox Mail Gateway
Scenario: Complete enterprise-grade email security gateway deployed from scratch — three VMs, eight phases, five phishing attack simulations, and full detection/blocking demonstration. Direct defensive counterpart to the GoPhish red team lab.
<br>**Tools:** Proxmox Mail Gateway 9.0 · Postfix · Dovecot · Docker · SpamAssassin · ClamAV · swaks · Thunderbird · analyze.py
<br>**Architecture:**
| Component | Role | Software |
|---|---|---|
| Kali Linux VM | Attacker / Tester | swaks · analyze.py · EICAR |
| PMG VM | Email Gateway / Filter | Proxmox Mail Gateway 9.0 |
| Postfix + Dovecot | Mail Server | Docker mailserver container |
| Windows 10 VM | Victim / Client | Thunderbird IMAP |
- ✅ PMG 9.0 installed — ClamAV (6.6M signatures) · SpamAssassin · Quarantine rules configured
- ✅ Docker mail server — Postfix + Dovecot + OpenDKIM + OpenDMARC in single container
- ✅ Custom forensic tool analyze.py developed — 7-section .eml analysis (headers · SPF/DKIM/DMARC · MIME · URLs)
- ✅ DMARC p=reject blocked CEO fraud spoof — rejected before SpamAssassin even ran
- ✅ Sender blocklist rule — $SENDEREAMAIL blocked: pmg-smtp-filter: block mail to $VICTIMEAMAIL
- ✅ SpamAssassin threshold 5/5 — phishing email (Test 5) scored maximum
- ✅ ClamAV virus infrastructure confirmed — EICAR test file sent as attachment
- ✅ Thunderbird on Windows 10 — victim perspective demonstrated via IMAP :143
- ✅ PMG Tracking Center — full audit trail per email (SA score · rule applied · relay path · disposition)
- ✅ Trust zone finding documented — ALL_TRUSTED(-1) adjustment for internal traffic bypasses content scoring

**Some Commands Used**
```bash
Phase 1 Docker Mail Server Deployment
# Install Docker
# sudo apt install docker.io docker-compose -y
# docker --version
# docker-compose --version
# Start mailserver container (docker-compose.yml pre-configured)
# docker-compose up -d
# victim mailbox
# docker exec -it mailserver setup email add $VICTIMEAMAIL $PASSWORD
Phase 2 Email Forensics 
# Run custom 7-section forensic analysis on any .eml file
# python3 analyze.py test.eml
# Works on course samples too
# python3 analyze.py Testmail_1.eml
# python3 analyze.py karcher_spam.eml
Phase 3 Attack Simulation (5 Tests)
# Test 1 — Basic delivery
# swaks --to $VICTIMEAMAIL --from $ATTACKEREAMAIL\
#       --server 19X.XXX.XX.XX --port 25 --helo $ATTACKERHOST
Phase 4 PMG Configuration
# Access PMG Web UI
# PMG settings configured via Web UI:
# Verify mail flow through PMG
Phase 5 PMG Advanced Rules
# Test blocklist rule (configured in PMG Web UI)
# Generate EICAR test file for ClamAV testing
# Check PMG Tracking Center for SA scores and rule verdicts
```
**MITRE ATT&CK Mapping**
| Technique | ID | Tool | Phase |
|---|---|---|---|
| Phishing | T1566 | swaks | Phase 4 — all 5 tests |
| Spearphishing Link | T1566.002 | swaks + fake URL | Phase 4 — Test 5 |
| Masquerading | T1036 | $SENDEREMAIL@lab.local From: | Phase 4 — Test 5 |
| Email Hiding Rules | T1564.008 | Missing Message-ID | Phase 4 — Test 2 |
| Impersonation | T1656 | $SENDEREMAIL@DOMAIN.COM spoof | Phase 4 — Test 3 |
| Defense: Email Filtering | M1031 | PMG + SpamAssassin | Phase 6/8 |
| Defense: DMARC | M1054 | DMARC p=reject (COMPANY) | Phase 4 — Test 3 |

**Defensive Countermeasures**
| Attack Vector | Defense | Tool | Effectiveness |
|---|---|---|---|
| From: spoofing | DMARC p=reject | DNS + PMG enforcement | 🔴 Very High — blocks at gateway |
| Known malicious sender | Blocklist rule | PMG Mail Filter | 🔴 High — immediate block |
| Spam/phishing content | SpamAssassin scoring | PMG + SA rules | 🟡 High — scores 5/5 |
| Malware attachment | ClamAV scanning | PMG + ClamAV | 🔴 High — 6.6M signatures |
| Urgency manipulation | Subject filter rule | PMG What Objects | 🟡 Medium — keyword match |
| Missing Message-ID | Header analysis | analyze.py / SOC triage | 🔴 High — forensic indicator |
| Credential capture | MFA enforcement | Microsoft Authenticator | 🔴 Very High — neutralizes capture |

**Key Findings** 
<br>**DMARC** p=reject is the only control that stops spoofing at the gateway — proven in Test 3 where $SENDEREMAIL@DOMAIN.COM was rejected before SpamAssassin ran. Domains with p=none are invisible to the gateway
<br>**Trust zone misconfiguration** is a real risk — ALL_TRUSTED(-1) SpamAssassin adjustment for internal senders reduces phishing scores from 5/5 to 0/5. Overly broad trusted network ranges bypass content inspection — a production misconfiguration SOC analysts must audit
<br>**PMG Tracking Center** = Mimecast Message Center — same workflow: sender IP · SA score breakdown · rule applied · relay path · delivery status. Skills transfer directly to enterprise platforms
<br>**Message-ID** is a forensic fingerprint — its absence reveals non-standard sending tools; its domain reveals true sending infrastructure regardless of the From: header

> 📄 **[Download Full Lab Report (PDF)](https://github.com/jaalso/cybersecurity-portfolio/raw/main/email_security_gateway_report_protected.pdf)**
<br>🔒 Password protected — contact me via [LinkedIn](https://linkedin.com/in/jaalso) to request access

---

## 🧰 Tools Used

| Category | Tools |
|---|---|
| Traffic Analysis | Wireshark · TShark · NetworkMiner |
| Certificate Analysis | nmap NSE · sslyze · sslscan · openssl · telnet · csvlook |
| Network Scanning | nmap · netdiscover · curl · Hydra |
| Email Forensics | emlAnalyzer · CyberChef · MXToolbox |
| SIEM & Monitoring | Wazuh v4.14.3 · OpenSearch (internal) |
| Deployment | wget · dpkg · systemctl · SSH |
| Platform | Kali Linux · VirtualBox · .r.vuln.land (authorised) |

---

## ⚖️ Legal & Ethical Notice

All defensive security activities were conducted on:
- Authorised lab environments provided by Swiss Cyber Institute
- Own personal infrastructure (home network audit — authorised)
- Training platforms (TryHackMe, HackTheBox, CyberDefenders)

No unauthorised systems were accessed. All work complies with Swiss law.
