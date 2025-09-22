# Basic commands of nmap
  **1) Quick SYN scan (default ports):**
nmap -sS < subnet > -oN scan-results.txt


**2) SYN scan with service/version detection and OS fingerprinting:** 
nmap -sS -sV -O < subnet > -oN scan-results-detailed.txt

**3) Save grepable and XML outputs for parsing:**
nmap -sS < subnet > -oG scan-results.gnmap -oX scan-results.xml


**4) Scan a single host for all 65535 TCP ports (use with caution):**
nmap -sS -p- 192.168.1.10 -oN host-all-ports.txt


**5) UDP scan (slower, may require root/administrator privileges):**
sudo nmap -sU < subnet > -oN udp-scan.txt
