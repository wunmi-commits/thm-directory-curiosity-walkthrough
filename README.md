# TryHackMe: Directory Curiosity - Incident Response Write-up

## 📌 Scenario
An alert was triggered: *"A user came across a poor file index, and their curiosity led to problems."* The objective was to inspect the provided network capture (`directory-curiosity.pcap`) to confirm if the alert was a true positive and extract Indicators of Compromise (IOCs).

**Tools Used:** TShark, VirusTotal, Linux CLI (sha256sum, grep)

---

## 🛠️ Methodology & Investigation

### 1. Network Traffic Analysis (DNS)
To find the suspicious domain, I filtered the pcap for DNS responses and sorted them to find unique domains.
```bash
tshark -r directory-curiosity.pcap -Y "dns.flags.response == 1" -T fields -e dns.qry.name | sort -u
Finding: Identified the malicious domain jx2-bavuong[.]com. VirusTotal confirmed this domain is malicious.


2. HTTP Traffic Investigation
Next, I analyzed the HTTP requests sent to this domain.

Total HTTP Requests: 14

Server Info: Apache/2.2.11 (Win32) DAV/2 mod_ssl/2.2.11 OpenSSL/0.9.8i PHP/5.2.9

Resolved IP Address: 141[.]164[.]41[.]174

3. Analyzing the Directory Listing (TCP Stream)
The alert mentioned a "poor file index." I followed the first TCP stream in ASCII mode to view the web server's response.

bash
tshark -r directory-curiosity.pcap -z follow,tcp,ascii,0 -q
Findings:

The server exposed a directory listing containing 3 files.

The first file listed was 123[.]php.

4. Payload Extraction & Hashing
I exported all HTTP objects from the pcap to extract the downloaded files.

bash
tshark -r directory-curiosity.pcap --export-objects http,~/exports
Inside the exports directory, I found the downloaded executable: vlauto[.]exe.

I generated the SHA256 hash of the file for threat intelligence lookup:

bash
sha256sum vlauto.exe
SHA256: b4851333efaf399889456f78eac0fd532e9d8791b23a86a19402c1164aed20de

5. Threat Intelligence (VirusTotal)
I submitted the SHA256 hash to VirusTotal for analysis:

PEiD Packer: .NET executable

Lastline Sandbox Classification: MALWARE TROJAN

🎯 Conclusion & Indicators of Compromise (IOCs)
The alert was a true positive. The user accessed a compromised web directory, viewed a malicious PHP script, and ultimately downloaded a malicious .exe file classified as a Trojan.

IOC List:

Domain: jx2-bavuong[.]com

IP: 141[.]164[.]41[.]174

File Name: vlauto[.]exe

SHA256: b4851333efaf399889456f78eac0fd532e9d8791b23a86a19402c1164aed20de

Author: Wunmi Adam.
Project: Directory-curiosity-walkthrough-Tryhackme.
Focus: Cybersecurity | Detection engineer | Threat Analysis | Defensive Security.
