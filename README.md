🛡️blue-team-labs
> Defensive security lab write-ups 

---

## 📁labs

### 01 · Network Traffic Forensics — Phishing Attack Investigation
**Tools:** Wireshark · TShark · VirusTotal · HTTP Object Export  
**Context:** Swiss Cyber Institute — Network Analysis Module (March 2026)

End-to-end phishing PCAP investigation — from "data on Pastebin" backwards to initial intrusion.

- ✅ SMTP analysis — spoofed phishing email identified (loser@hotmail.com)
- ✅ Malicious PDF extracted from PCAP (26 kB) — 41/64 VT detection
- ✅ PDF forensics — embedded JavaScript, auto-execute /OpenAction, obfuscated shellcode
- ✅ CVEs confirmed: CVE-2009-0927 · CVE-2008-2992 (Adobe Reader)
- ✅ 1.2 MB PII exfiltration traced — names, SSNs, credit cards in plaintext HTTP
- ✅ 5-phase attack timeline reconstructed · 20-hour dwell time identified
- ✅ Zero Pastebin results = finding (exfiltration via direct HTTP upload, not Pastebin)

📄 **[Download Full Lab Report (PDF)](https://github.com/jaalso/cybersecurity-portfolio/raw/main/Phishing_Forensics_Writeup_protected.pdf)**  
> 🔒 Password protected — contact me via [LinkedIn](https://linkedin.com/in/jaalso)

---

### 02 · Home Network Security Audit
**Tools:** netdiscover · nmap · Hydra · curl  
**Context:** Live Zürich home network · February–March 2026

Full recon-to-findings audit on a real home network using professional pentest methodology.

- ✅ 10 live hosts via ARP sweep — MAC OUI vendor mapping
- ✅ Critical finding: unauthenticated admin panel on Bang & Olufsen BeoPlay A9 (~CHF 3,500 device)
- ✅ SSL certificate expired **1999** — 26 years ago
- ✅ UPnP enabled on Swisscom Internet-Box — silent port-opening attack vector
- ✅ Hydra false-positive analysis — 16 "found" passwords, all manually disproved
- ✅ IoT security comparison: Roomba (hardened) vs BeoPlay A9 (critical failures)

| Device | Open Ports | Auth | Verdict |
|---|---|---|---|
| Swisscom Internet-Box | 5 | TLS 1.3 + SPA | ⚠️ UPnP risk |
| iRobot Roomba | 1 (outbound MQTT) | N/A | ✅ Hardened |
| BeoPlay A9 (~CHF 3,500) | 5 | ❌ None | 🔴 Critical |

📄 **[Download Full Lab Report (PDF)](https://github.com/jaalso/cybersecurity-portfolio/raw/main/HomeNetworkAudit_Writeup_protected.pdf)**  
> 🔒 Password protected — contact me via [LinkedIn](https://linkedin.com/in/jaalso)

---

### 03 · Web Application Security — Certificate Analysis
**Tools:** nmap NSE · sslyze · openssl · telnet · csvlook  
**Context:** Swiss Cyber Institute — CSS Exam Challenge #07 + 2022 Challenge #8

Two-exercise TLS certificate assessment — port discovery, cipher analysis, PKI chain verification.

- ✅ Full port scan found 6 ports vs 2 with default -F scan
- ✅ 25 cipher suites in TLS 1.2 (vs Mozilla-recommended 7)
- ✅ All 5 trust stores rejected certificate — SAN mismatch
- ✅ CSV audit: 3 critical findings (revoked, SHA-1+RSA1024, self-signed) + 12 expired certs
- ✅ Intermediate CA REVOKED — invalidates every certificate it ever signed
- ✅ Manual CRL revocation chain verified: Root CA → Intermediate CA → End Entity

📄 **[Download Full Lab Report (PDF)](https://github.com/jaalso/cybersecurity-portfolio/raw/main/CertificateAnalysis_Writeup_protected.pdf)**  
> 🔒 Password protected — contact me via [LinkedIn](https://linkedin.com/in/jaalso)

---

### 04 · SIEM & Endpoint Detection (Wazuh)
**Tools:** Wazuh · Elastic Stack · Sysmon  
**Status:** 🔜 Write-up coming soon

- ✅ Wazuh v4.14.3 deployed on VirtualBox
- ✅ CIS Benchmark assessment (190 controls)
- ✅ MITRE ATT&CK-tagged threat detection (258 events)
- ✅ File Integrity Monitoring on /etc/
- ✅ CVE vulnerability scanning

---

### 05 · Email Security Gateway Lab
**Tools:** SPF · DKIM · DMARC · DNS  
**Status:** 🔜 Write-up coming soon

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
