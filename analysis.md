# Analysis of Nmap Scan Results


## Host: 192.168.1.1
- **Open ports**: 22/tcp (ssh), 80/tcp (http), 443/tcp (https)
- **Likely services**: OpenSSH (SSH), Apache/Nginx (HTTP/HTTPS)
- **Risk assessment**:
- SSH (22): If password authentication is enabled, risk of brute-force attacks. Ensure strong passwords or use key-based auth and disable root login.
- HTTP (80): May expose administrative pages or default pages. Check for sensitive endpoints and disable directory listing.
- HTTPS (443): Good to have encrypted traffic, but check certificate validity and server configuration to avoid TLS vulnerabilities.
- **Recommendations**:
- Use firewall rules to limit SSH to trusted IPs.
- Disable unused services.
- Keep server software updated.


## Host: 192.168.1.10
- **Open ports**: 139/tcp (netbios-ssn), 445/tcp (microsoft-ds)
- **Likely services**: SMB/CIFS (file sharing)
- **Risk assessment**:
- SMB exposed on local network can allow file share enumeration and, if misconfigured, data exfiltration or ransomware pivoting.
- **Recommendations**:
- Disable SMB if not required, or restrict access via firewall.
- Apply latest patches and disable SMBv1.
- Require strong credentials and limit share permissions.
