# Analysis of Nmap Scan Results

This file contains the analysis of the Nmap scans performed on different hosts as per the screenshots.

---

## 1. Host: 192.168.1.1 (Local Router / Gateway)

**Scan Commands Used:**

* `nmap -sS 192.168.1.1 -oG scan-results.gnmap -oX scan-results.xml`
* `nmap -sS -p- 192.168.1.1 -oN host-all-ports.txt`

**Open / Filtered Ports:**

* 22/tcp — filtered (ssh)
* 23/tcp — filtered (telnet)
* 53/tcp — open (domain / DNS)
* 80/tcp — open (http)
* 139/tcp — open (netbios-ssn)
* 443/tcp — open (https)
* 445/tcp — open (microsoft-ds)
* 631/tcp — open (ipp)
* 2233/tcp — filtered (infocrypt)

**Risk Analysis:**

* **DNS (53):** If misconfigured, may allow DNS amplification attacks.
* **HTTP (80) & HTTPS (443):** Indicates a web admin interface or hosted service. Ensure TLS is correctly configured and admin panels are protected.
* **NetBIOS/SMB (139, 445):** High risk if exposed; SMB vulnerabilities have been used in ransomware (e.g., WannaCry). Should be restricted to internal trusted hosts.
* **IPP (631):** Internet Printing Protocol — may expose printer interfaces. Risk if exposed externally.
* **Filtered Ports (22, 23):** SSH and Telnet attempts are blocked but still reachable. Best to fully close/disable Telnet and secure SSH with key authentication.

**Recommendations:**

* Restrict DNS (53) and SMB (139/445) to trusted internal IPs only.
* Disable Telnet (23) entirely.
* Secure web admin (80/443) with strong credentials and HTTPS with valid TLS.
* Close unused services and enable firewall rules.

---

## 2. Host: [www.microsoft.com](http://www.microsoft.com) (23.36.5.119)

**Scan Command Used:**

* `nmap -sS www.microsoft.com -oN microsoftscanresult.txt`

**Open Ports:**

* 80/tcp — open (http)
* 443/tcp — open (https)

**Risk Analysis:**

* **HTTP (80):** Standard web traffic. Likely redirect to HTTPS. Risk minimal given Microsoft’s hardened infrastructure.
* **HTTPS (443):** Encrypted web traffic. TLS is expected and well configured by Microsoft.

**Recommendations:**

* No specific action needed (Microsoft servers are managed). This scan is only for educational purposes.

---

# Summary

* The local router (192.168.1.1) shows multiple open ports including DNS, HTTP, SMB, and IPP which may expose services unnecessarily. Hardening and restricting these services is highly recommended.
* External scan of [www.microsoft.com](http://www.microsoft.com) shows only web services (80/443) as expected for a large enterprise web server.

---

**Note:** Always ensure you scan only systems you own or have explicit permission to test. The local network scan is safe since it is your own network; scanning Microsoft was limited and benign but in general, avoid scanning third-party systems without authorization.

---

## 3. Detailed Recommendations (per-port + general hardening)

Below are specific, actionable recommendations for each observed port and general hardening steps you should apply to devices on your network.

### Per-port recommendations

* **22/tcp (SSH)**

  * Disable password authentication; use SSH keys only: set `PasswordAuthentication no` in `/etc/ssh/sshd_config`.
  * Disable root login: `PermitRootLogin no`.
  * Run fail2ban or equivalent to block brute-force attempts.
  * Restrict SSH access via firewall to trusted management IPs (example `ufw` rule):

    ```bash
    sudo ufw allow from 203.0.113.5 to any port 22 proto tcp
    sudo ufw deny 22/tcp
    ```
  * Use non-standard port only if you understand the trade-offs (security through obscurity is not a substitute for controls).

* **23/tcp (Telnet)**

  * **Disable Telnet completely.** Telnet transmits credentials in cleartext—replace with SSH.
  * Remove telnet server package (e.g., `sudo apt remove telnetd`) and close port in firewall.

* **53/tcp (DNS)**

  * Ensure the DNS service is not an open resolver. Configure it to answer only trusted clients or internal subnets.
  * Bind DNS to LAN interfaces only (avoid `0.0.0.0` on public-facing interfaces).
  * Apply rate-limiting and response-rate limiting to mitigate amplification attacks.
  * Example `bind/named` configuration snippet: `allow-query { 192.168.1.0/24; localhost; };`

* **80/tcp (HTTP)**

  * Redirect HTTP to HTTPS and disable serving sensitive content over HTTP.
  * Harden web applications: remove default pages, disable directory listing, enforce strong auth for admin paths.
  * Use a Web Application Firewall (WAF) if the device supports it and keep web server software patched.

* **443/tcp (HTTPS)**

  * Use strong TLS configurations (disable TLS 1.0/1.1, prefer TLS 1.2+/1.3). Test with SSL Labs or `openssl`.
  * Use valid certificates from a trusted CA and enable HSTS where appropriate.
  * Regularly scan web applications for common vulnerabilities (use `nikto`, `owasp-zap` on permitted targets).

* **139/tcp & 445/tcp (NetBIOS / SMB)**

  * Disable SMB if not required. If required:

    * Disable SMBv1.
    * Restrict access to trusted subnets only via firewall rules.
    * Enforce strong credentials and principle of least privilege on shares.
    * Keep SMB/CIFS implementation patched.
  * Example firewall rule (iptables):

    ```bash
    # allow SMB from LAN only
    iptables -A INPUT -p tcp -s 192.168.1.0/24 --dport 445 -j ACCEPT
    iptables -A INPUT -p tcp --dport 445 -j DROP
    ```

* **631/tcp (IPP)**

  * Disable printing services if unused, or restrict to internal hosts.
  * Require authentication for admin/print functions and update printer firmware.
  * If printer exposes a web UI, secure it behind HTTPS and strong credentials.

### General network hardening recommendations

1. **Firewall & Access Control**: Implement host-based or network firewall rules that allow only necessary services and restrict management interfaces to a small set of trusted IPs.
2. **Patch Management**: Keep firmware and software up to date for routers, NAS devices, printers, and servers.
3. **Least Privilege**: Limit user/share permissions; avoid giving broad access to network shares.
4. **Segmentation**: Use VLANs or guest network segmentation to isolate IoT/printer devices from critical systems.
5. **Strong Authentication**: Use key-based auth for SSH, strong passwords, MFA for admin portals when available.
6. **Logging & Monitoring**: Enable logging on devices and centralize logs if possible. Monitor for unusual connection attempts or multiple failed logins.
7. **Backup & Recovery**: Maintain regular backups and test recovery procedures in case of compromise (e.g., ransomware).
8. **Disable Unused Services**: Turn off any services that are not required for the device’s operation.
9. **Periodic Re-Scans**: After applying changes, re-run `nmap` scans to verify ports are closed/filtered as expected and document the before/after state in your repo.

