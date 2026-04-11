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
| 04 | SIEM & Endpoint Detection (Wazuh) | Wazuh · Elastic Stack · Sysmon | 🔜 Coming soon |
| 05 | Email Security Gateway | SPF · DKIM · DMARC · Proxmox | 🔜 Coming soon |

## 📁labs

###  Network Traffic Forensics - Phishing Attack Investigation
Scenario: SCI Network Analysis Module -> Corporate phishing incident customer PII exfiltrated to external server
Investigated a real-world-style incident starting from a known outcome ("customer data on Pastebin")
and traced it backwards through a PCAP file to identify the initial intrusion vector, compromised users,
malware delivery chain, and data exfiltration method.
- **Tools:** Wireshark · TShark · VirusTotal · HTTP Object Export
- ✅ SMTP analysis :spoofed phishing email identified (xxxx@hotmail.com)
- ✅ Malicious PDF extracted from PCAP (26 kB) — 41/64 VT detection
- ✅ PDF forensics — embedded JavaScript, auto-execute /OpenAction, obfuscated shellcode
- ✅ CVEs confirmed: CVE-2009-0927 · CVE-2008-2992 (Adobe Reader)
- ✅ 1.2 MB PII exfiltration traced — names, SSNs, credit cards in plaintext HTTP
- ✅ 5-phase attack timeline reconstructed · 20-hour dwell time identified
- ✅ Zero Pastebin results = finding (exfiltration via direct HTTP upload, not Pastebin)

**Wireshark filters and chosen options**
```bash
# Phase 1 — Initial Triage
# Protocol hierarchy — bird's-eye view of traffic
# Wireshark: Statistics → Protocol Hierarchy
# Network participants — find all IPs
# Wireshark: Statistics → Conversations → IPv4 tab
# Filter out TCP noise to see UDP/other protocols
#! top
#Phase 2 — SMTP Analysis (Phishing Email)
# Filter for SMTP traffic
#smtp
# Follow TCP stream to read full email exchange
# Right-click any SMTP packet → Follow → TCP Stream
# Filter for victim + attacker traffic
#ip.addr == 173.255.XXX.XX || ip.addr == 10.10X.XXX.XX
#Phase 3 — HTTP Analysis (Malware Delivery)
# Find who clicked the phishing link
#http
# Filter traffic to malware delivery server
#ip.addr == 14.X.X.XX
# Extract malicious PDF from PCAP
# Wireshark: File → Export Objects → HTTP
# Save: pfqa.php (application/pdf, 26 kB)
#Phase 4 — PDF Forensics
# Check file hash
#sha256sum xXXx.php
# Inspect PDF structure in text editor (NEVER open in Adobe Reader)
# cat pfqa.php | grep -i "javascript\|openaction\|eval\|stream"
# Submit hash to VirusTotal
# https://virustotal.com → search SHA256 $VALUE
# Phase 5 — Exfiltration Tracing
# Follow TCP stream on exfiltration connection
# Right-click packet 402 → Follow → TCP Stream
# Seq 1 → 1,210,906 = ~1.2 MB uploaded to 55.XX.XX.XX
```
Key Finding: Two internal victims compromised. Full names, SSNs, credit card numbers and CVVs
exfiltrated in plaintext ~20 hours post-infection. 

CVEs:
- CVE-2009-0927 (Adobe Reader JS buffer overflow)
- CVE-2008-2992 (util.printf stack overflow)
> 📄 **[Download Full Lab Report (PDF)](https://github.com/jaalso/cybersecurity-portfolio/raw/main/Phishing_Forensics_Writeup_protected.pdf)**
<br>🔒 Password protected — contact me via [LinkedIn](https://linkedin.com/in/jaalso) to request access













---

### 03 · Home Network Security Audit
Performed a full recon-to-findings audit on a real home network using the same methodology used in
professional penetration tests — ARP sweep, service enumeration, credential testing, and manual verification.
- **Tools:** netdiscover · nmap · Hydra · curl
- Target:  192.168.X.X/24 home gateway.
- ✅ 10 live hosts via ARP sweep — MAC OUI vendor mapping
- ✅ Critical finding: unauthenticated admin panel on Bang & Olufsen Speaker
- ✅ SSL certificate expired **1999** — 26 years ago
- ✅ UPnP enabled on Swisscom Internet-Box — silent port-opening attack vector
- ✅ Hydra false-positive analysis — 16 "found" passwords, all manually disproved
- ✅ IoT security comparison: Roomba (hardened) vs Speaker BP A9 (critical failures)

Key Finding: Bang & Olufsen Speaker at 192.168.1.XXX exposed a fully
unauthenticated admin panel (GoAhead WebServer) with direct access to network settings and firmware
update capability. SSL certificate expired in 1999.
Notable lesson: Hydra reported 16 valid passwords against the router — all false positives caused by
the router returning identical 301 redirects for every request. Confirmed that automated tool output
must always be manually verified before reporting findings.

**Asset discovery — 10 hosts identified**
<br><img width="335" height="25" alt="image" src="https://github.com/user-attachments/assets/dd25c244-83f6-4742-a8f7-c88e9747109f" />
<br><img width="553" height="255" alt="image" src="https://github.com/user-attachments/assets/6d4cda6f-b5bc-4c0e-8ece-1498657555d2" />


**Speaker — open ports + expired 1999 SSL cert**
<br><img width="486" height="101" alt="image" src="https://github.com/user-attachments/assets/af662602-c415-4325-8c09-144ddf5b696b" />
<br><img width="468" height="161" alt="image" src="https://github.com/user-attachments/assets/2ead66d0-d467-4ec7-8ba2-4993016ab4eb" />


**Unauthenticated admin panel — direct browser access**
<br><img width="404" height="69" alt="image" src="https://github.com/user-attachments/assets/89f716f6-18d8-4b6e-ba40-82a7f99e7a73" />

> 📄 **[Download Full Lab Report (PDF)](https://github.com/jaalso/cybersecurity-portfolio/raw/main/HomeNetworkAudit_Writeup_protected.pdf)**
<br>🔒 Password protected — contact me via [LinkedIn](https://linkedin.com/in/jaalso) to request access

---

### 03 · Web Application Security — Certificate Analysis
**Tools:** nmap NSE · sslyze · openssl · telnet · csvlook  
**Context:** Based on a CSS Exam Challenge: Two-exercise certificate security assessment lab combining port discovery, TLS certificate mapping, in-depth cipher analysis, CSV certificate database auditing, and manual PKI chain revocation verification.
Target: Authorised .r.vuln.land lab targets · local certificate chain files
- ✅ Full port scan found 6 ports vs 2 with default -F scan
- ✅ 25 cipher suites in TLS 1.2 (vs Mozilla-recommended 7)
- ✅ All 5 trust stores rejected certificate — SAN mismatch
- ✅ CSV audit: 3 critical findings (revoked, SHA-1+RSA1024, self-signed) + 12 expired certs
- ✅ Intermediate CA REVOKED — invalidates every certificate it ever signed
- ✅ Manual CRL revocation chain verified: Root CA → Intermediate CA → End Entity

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
<br><img width="1379" height="898" alt="image" src="https://github.com/user-attachments/assets/0acd73f3-1b33-4673-9126-1e179469a247" />

** CRL Revocation Verification of a Certificate Chain**
<br><img width="510" height="240" alt="image" src="https://github.com/user-attachments/assets/bfe8fb38-187d-4e49-a82b-414dc35ca81a" />
<br><img width="584" height="82" alt="image" src="https://github.com/user-attachments/assets/1f4ea08a-8af3-4d99-a53e-d01de59339d5" />
<br><img width="603" height="157" alt="image" src="https://github.com/user-attachments/assets/22047420-c621-4bab-8de2-eea34e97a41f" />
<br><img width="518" height="91" alt="image" src="https://github.com/user-attachments/assets/700297b2-06cd-447d-bc8a-6ff36c92e067" />



📄 **[Download Full Lab Report (PDF)](https://github.com/jaalso/cybersecurity-portfolio/raw/main/CertificateAnalysis_Writeup_protected.pdf)**  
> 🔒 Password protected — contact me via [LinkedIn](https://linkedin.com/in/jaalso)
---


## 🧰 Tools Used

| Category | Tools |
|---|---|
| Traffic Analysis | Wireshark · TShark · NetworkMiner |
| Certificate Analysis | nmap NSE · sslyze · openssl · telnet |
| SIEM & Monitoring | Wazuh · Elastic Stack · Sysmon |
| Email Forensics | emlAnalyzer · CyberChef · MXToolbox |
| Platform | Kali Linux · VirtualBox · .r.vuln.land |

---

## ⚖️ Legal & Ethical Notice

All defensive security activities were conducted on:
- Authorised lab environments provided by Swiss Cyber Institute
- Own personal infrastructure (home network audit — authorised)
- Training platforms (TryHackMe, HackTheBox, CyberDefenders)

No unauthorised systems were accessed. All work complies with Swiss law.
