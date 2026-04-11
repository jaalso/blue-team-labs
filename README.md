🛡️blue-team-labs
> Defensive security lab write-ups 

---

## 📁labs

###  Network Traffic Forensics - Phishing Attack Investigation
Scenario: Corporate phishing incident — customer PII exfiltrated to external server
Investigated a real-world-style incident starting from a known outcome ("customer data on Pastebin")
and traced it backwards through a PCAP file to identify the initial intrusion vector, compromised users,
malware delivery chain, and data exfiltration method.
- **Tools:** Wireshark · TShark · VirusTotal · HTTP Object Export
- ✅ SMTP analysis :spoofed phishing email identified (loser@hotmail.com)
- ✅ Malicious PDF extracted from PCAP (26 kB) — 41/64 VT detection
- ✅ PDF forensics — embedded JavaScript, auto-execute /OpenAction, obfuscated shellcode
- ✅ CVEs confirmed: CVE-2009-0927 · CVE-2008-2992 (Adobe Reader)
- ✅ 1.2 MB PII exfiltration traced — names, SSNs, credit cards in plaintext HTTP
- ✅ 5-phase attack timeline reconstructed · 20-hour dwell time identified
- ✅ Zero Pastebin results = finding (exfiltration via direct HTTP upload, not Pastebin)

Key Finding: Two internal victims compromised. Full names, SSNs, credit card numbers and CVVs
exfiltrated in plaintext ~20 hours post-infection. 

CVEs: CVE-2009-0927 (Adobe Reader JS buffer overflow) · CVE-2008-2992 (util.printf stack overflow)
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
