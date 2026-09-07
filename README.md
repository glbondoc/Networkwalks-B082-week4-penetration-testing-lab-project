# Penetration Testing Report
Mediroza General Hospital — External Web Application Assessment

Cybersecurity | Networkwalks

| Field | Details |
|---|---|
| **Pentester Name** | Genrei L. Bondoc |
| **Program/Batch** | B082 |
| **Date** | 07 September 2026 |
| **Modules Completed** | <br> Web Penetration Testing |
| **Client/Target** | Mediroza General Hospital — `https://medirozahospital.com` |
| **Permission Secured from Client?** | Yes |
| **Engagement Type** | Web Penetration Test |
| **Testing Period** | Single session — approximately 1.5 hours |
| **Overall Risk Rating** | 🔴 **CRITICAL** |

---

# 1. Executive Summary

<p>A penetration test was conducted against the external web presence of Mediroza General Hospital (medirozahospital.com). The assessment simulated an unauthenticated remote attacker with no prior knowledge of the environment.  </p>

<p>The engagement resulted in complete compromise of the patient records system and the disclosure of confidential corporate and employee data. Multiple severe vulnerabilities were identified across the application, and every stage of the intended attack path — reconnaissance, authentication bypass, unauthorized access, and data exfiltration — was successfully executed.  </p>

---

<table>
    <thead>
        <tr>
            <th>#</th>
            <th>Milestone</th>
            <th>Status</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1</td>
            <td>External reconnaissance &amp; attack surface mapping</td>
            <td>✅ Achieved</td>
        </tr>
        <tr>
            <td>2</td>
            <td>Discovery of publicly exposed database backup</td>
            <td>✅ Achieved</td>
        </tr>
        <tr>
            <td>3</td>
            <td>Authentication bypass via SQL Injection</td>
            <td>✅ Achieved</td>
        </tr>
        <tr>
            <td>4</td>
            <td>Unauthorized access to restricted patient portal</td>
            <td>✅ Achieved</td>
        </tr>
        <tr>
            <td>5</td>
            <td>Exfiltration of 3 confidential pathology reports (PDF)</td>
            <td>✅ Achieved</td>
        </tr>
        <tr>
            <td>6</td>
            <td>Offline cracking of PDF encryption password</td>
            <td>✅ Achieved</td>
        </tr>
        <tr>
            <td>7</td>
            <td>Full decryption &amp; disclosure of patient medical record</td>
            <td>✅ Achieved</td>
        </tr>
        <tr>
            <td>8</td>
            <td>Disclosure of all 30 employee salaries + national IDs</td>
            <td>✅ Achieved</td>
        </tr>
        <tr>
            <td>9</td>
            <td>Disclosure of complete shareholder registry</td>
            <td>✅ Achieved</td>
        </tr>
    </tbody>
</table>

**The combination of findings exposes Mediroza to:**
<ol>
  <li><strong>Regulatory &amp; Legal Exposure (POPIA/HIPAA-equivalent breach)</strong> — Protected Health Information (PHI) of at least 3 patients was accessed without authorization. Under South Africa's Protection of Personal Information Act (POPIA) and the National Health Act, this constitutes a notifiable breach with potential penalties.</li>

  <li><strong>Insider Tension &amp; HR Crisis</strong> — Publication of the complete payroll (R 1,998,000/month, 30 employees) and national IDs would cause significant internal unrest and is exploitable for social engineering.</li>

  <li><strong>Corporate Espionage Material</strong> — The shareholder registry (10 shareholders, 1,000,000 shares) reveals the hospital's ownership structure, insider stakes, and preferential share arrangements — valuable to competitors and hostile actors.</li>

  <li><strong>Complete Trust Erosion</strong> — A hospital's core business depends on patient trust. Public knowledge that lab results, medical conditions, and staff compensation are trivially obtainable would be commercially devastating.</li>
</ol>

**The attack chain required no credentials, no advanced tooling, and only standard utilities (curl, grep, sed, john, python3). Any motivated party — including automated scanners and commodity threat actors could replicate these results.**

---
# 2 — Scope and Methodology 
## 2.1 Scope
<table>
  <thead>
    <tr>
      <th>Item</th>
      <th>Detail</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>In-Scope Target</strong></td>
      <td> https://medirozahospital.com (all paths)</td>
    </tr>
    <tr>
      <td><strong>Out of Scope</strong></td>
      <td>Physical security, social engineering, staff interviews, denial-of-service, and other domains/subdomains owned by the organization</td>
    </tr>
    <tr>
      <td><strong>Authorization Level</strong></td>
      <td>Full exploitation permitted up to and including data exfiltration (per engagement rules of engagement)</td>
    </tr>
    <tr>
      <td><strong>Timing</strong></td>
      <td>All testing performed in a single daytime session; destructive testing and DoS were not attempted</td>
    </tr>
  </tbody>
</table>

## 2.2 Limitations Encountered
<table> <thead> <tr> <th>Limitation</th> <th>Impact</th> <th>Workaround</th> </tr> </thead> <tbody> <tr> <td> Rate limiting / WAF-style 403 responses on the patient login after rapid UNION SELECT testing </td> <td> Temporarily blocked SQLi column enumeration </td> <td> Slowed request cadence; pivoted to comment-termination payload (<code>admin'-- -</code>) which succeeded immediately </td> </tr> <tr> <td> Command environment shell pattern filter blocked certain piped command constructs </td> <td> Minor — required command splitting </td> <td> Split complex commands into sequential one-shot executions </td> </tr> <tr> <td> No shell/command-execution testing attempted (out of engagement focus) </td> <td> Potential additional vulnerabilities may exist in server-side file handling (download.php path traversal, LFI) — untested </td> <td> Flagged as recommended follow-up testing in §5 </td> </tr> <tr> <td> Staff login endpoint not brute-forced </td> <td> Staff-side credential strength unverified </td> <td> Flagged as recommended follow-up testing </td> </tr> </tbody> </table> <p> <strong>Note:</strong> All 403 responses observed were from LiteSpeed's rate-limiting, not a true WAF — the application itself performs no input filtering whatsoever. </p>

## 2.3 Methodology
**Testing followed a structured black-box methodology aligned with OWASP Web Security Testing Guide (OWASP WSTG v4.2):**

<ol>
  <li><strong>Reconnaissance (WSTG-INFO)</strong> — HTTP banner analysis, technology fingerprinting (LiteSpeed, PHP 8.2.33, Mediroza CMS 1.4.2), robots.txt/sitemap.xml review, endpoint discovery.</li>
  <li><strong>Configuration &amp; Deployment Management (WSTG-CONF)</strong> — Directory listing enumeration, exposed file verification.</li>
  <li><strong>Identity &amp; Authentication Testing (WSTG-ATHN/IDNT)</strong> — Credential injection, username enumeration via differential responses, authentication bypass.</li>
  <li><strong>Input Validation Testing (WSTG-INPV)</strong> — SQL injection (authentication-based), UNION-based exploration, comment-termination payloads.</li>
  <li><strong>Business Logic Testing (WSTG-BUSL)</strong> — Session authorization behavior on portal.php / download.php.</li>
  <li><strong>Post-Exploitation / Data Analysis</strong> — Offline PDF password recovery, PHI extraction, SQL dump parsing and aggregation.</li>
</ol>

---
# M1 
## 3. Tools Used

| Tool | Purpose |
|---|---|
| Kali Linux & Windows | Operating systems used for reconnaissance activities |
| WHOIS | Find domain registration details (owner, dates, name servers) |
| whatweb | Fingerprint web technologies (server, CMS, plugins, IP) |
| nslookup | Resolve the domain name to its IP address using DNS |
| curl -s -i -L  | To retrieve and inspect the website’s HTTP response headers and follow redirects to identify server information, security headers, and the final response. |
| wafw00f | Detect whether a Web Application Firewall protects the site |
| dnsrecon | Enumerate all DNS records (NS, MX, SPF, TXT, SRV) |
| nmap | Scan the local subnet to find live hosts and IP addresses |

---

# 4. Activities Performed

## 4.1 Reconnaissance & Attack Surface Mapping

I conducted reconnaissance on the networkwalks.com domain using six Kali Linux tools: WHOIS, WhatWeb, Nslookup, Curl, Wafw00f, and DNSRecon. Each tool provided a different set of information that helped build an overall profile of the target.
<ul>
 <li><strong>WHOIS</strong> — I examined the publicly available domain registration information. The domain is registered through NameCheap, Inc., was created on 2026-08-14, and is scheduled to expire on 2027-08-14. Its most recent update was recorded on 2026-08-14. The domain uses DNS1.NAMECHEAPHOSTING.COM and DNS2.NAMECHEAPHOSTING.COM as its name servers, indicating the use of Namecheap Hosting infrastructure. DNSSEC is currently unsigned, and the domain has the <code>clientTransferProhibited</code> status enabled. No registrant personal information was disclosed in the WHOIS output provided.</li>
<img width="630" height="500" alt="whois" src="https://github.com/user-attachments/assets/142ec5e7-2f88-4f9b-b3b3-6f114da46fb7" />

<li><strong>WhatWeb</strong> — I identified the technologies and server information exposed by the website. The HTTP endpoint redirects to HTTPS using a <strong>301 Moved Permanently</strong> response. Both the HTTP and HTTPS endpoints were identified as running on <strong>LiteSpeed</strong> and hosted at IP address <strong>199.188.201.16</strong>. The HTTPS endpoint returned a <strong>403 Forbidden</strong> response, indicating that access was restricted during scanning. The scan also identified HTML5 content and the uncommon <code>x-turbo-charged-by</code> response header.</li>
<img width="612" height="275" alt="whatweb" src="https://github.com/user-attachments/assets/f03f2c66-9071-468e-9803-6739951e0c53" />

  <li><strong>Nslookup</strong> — I used the tool to resolve <code>medirozahospital.com</code> to its corresponding IP address. The domain resolved to <strong>199.188.201.16</strong>. The DNS query was performed through the local DNS server <strong>192.168.93.2</strong>, which returned a non-authoritative response.</li>
  <img width="282" height="139" alt="nslookup" src="https://github.com/user-attachments/assets/e5d12840-5e0e-4695-a892-19f602a3f3b7" />

  <li><strong>Curl (-s -i -L)</strong> — I examined the website's HTTP response headers and followed any redirects. The server returned an <strong>HTTP/1.1 200 OK</strong> response and identified itself as <strong>OpenResty/1.31.1.1</strong>. The response used <strong>text/html</strong> content with a length of 11,975 bytes and included <code>Cache-Control</code> directives preventing caching. The response also contained a <code>cf-edge-cache: no-cache</code> header. The returned page displayed a <strong>"One moment, please..."</strong> verification screen that automatically reloads after five seconds, indicating that an automated request verification or anti-bot mechanism was present.</li>
  <img width="621" height="522" alt="curl -s -i -L (1)" src="https://github.com/user-attachments/assets/8002169f-6f40-425b-a186-9c4d2c4e35b8" />
  <img width="681" height="532" alt="curl -s -i -L (2)" src="https://github.com/user-attachments/assets/54361421-703c-434d-8024-694b051071ae" />
  <img width="625" height="496" alt="curl -s -i -L (3)" src="https://github.com/user-attachments/assets/b9eede16-a048-4699-a5fd-0cc9e9d9fdfd" />

  <li><strong>Wafw00f</strong> — I checked whether the website was protected by a Web Application Firewall (WAF). The results indicated that the site is behind <strong>LiteSpeed Technologies</strong>, which provides built-in WAF capabilities and supports ModSecurity-based rule sets for filtering potentially malicious web requests. :contentReference[oaicite:0]{index=0}</li>
  <img width="760" height="345" alt="wafw00f" src="https://github.com/user-attachments/assets/1dd6bfcf-fd55-455f-9f01-6dcf135479b3" />

  <li><strong>DNSRecon</strong> — I performed DNS record enumeration to identify publicly available DNS and hosting information. The results revealed SOA and NS records using <strong>dns1.namecheaphosting.com</strong> and <strong>dns2.namecheaphosting.com</strong>. The domain's A record resolved to <strong>199.188.201.16</strong>, while three MX records were identified under the <strong>jellyfish.systems</strong> hosting infrastructure. The enumeration also identified an SPF record authorizing the domain's A and MX records along with specific IP addresses and <code>spf.web-hosting.com</code>, as well as a DMARC record configured with <code>p=none</code>. Additional SRV records were discovered for CalDAV, CardDAV, and cPanel email autodiscovery services. DNSSEC enumeration returned no answer, and a total of <strong>12 DNS records</strong> were identified.</li>
  <img width="1144" height="523" alt="dnsrecon -d" src="https://github.com/user-attachments/assets/ef5cc852-0752-4521-9f76-0688f2424443" />

  <li><strong>Nmap (-sV -T5)</strong> — I performed service and version detection using Nmap with aggressive timing. The scan identified several publicly accessible TCP services, including <strong>FTP (Pure-FTPd)</strong> on port 21, <strong>SMTP (Exim 4.99.5)</strong> on ports 25 and 465, <strong>DNS (BIND)</strong> on port 53, <strong>HTTP/HAProxy</strong> on port 80, <strong>POP3 (Dovecot)</strong> on ports 110 and 995, <strong>IMAP (Dovecot)</strong> on ports 143 and 993, and <strong>HAProxy HTTPS proxy</strong> on port 8080. The results indicate that the host exposes multiple web, email, DNS, and file-transfer services that should be reviewed and restricted to only those required for the organization's operations.</li>
</ul>
<img width="753" height="467" alt="nmap -sV" src="https://github.com/user-attachments/assets/3b4515e4-3125-46cf-b2b1-da20a2682425" />

### Reconnaissance Findings

The reconnaissance phase was conducted against `medirozahospital.com` using seven Kali Linux tools: WHOIS, WhatWeb, Nslookup, Curl, Wafw00f, DNSRecon, and Nmap. Each tool provided different information that helped establish an overall technical profile of the target.

The reconnaissance identified:

- **Domain Registration:** The domain is registered through NameCheap, Inc., with Namecheap Hosting name servers and DNSSEC reported as unsigned.
- **Target Infrastructure:** `medirozahospital.com` resolved to **199.188.201.16**, with the DNS infrastructure associated with Namecheap Hosting.
- **Web Server Technologies:** WhatWeb identified **LiteSpeed** infrastructure, while Curl returned an **OpenResty/1.31.1.1** server response and a browser-verification page.
- **Web Protection:** Wafw00f identified **LiteSpeed Technologies** as the detected WAF technology.
- **DNS Information:** DNSRecon identified SOA, NS, A, MX, TXT, DMARC, and SRV records, including email, CalDAV, CardDAV, and cPanel autodiscovery services.
- **Email Infrastructure:** Multiple MX records associated with `jellyfish.systems` were identified, along with SPF and DMARC records.
- **Network Services:** Nmap identified publicly accessible services including **FTP, SMTP, DNS, HTTP/HAProxy, POP3, IMAP, and SSL/TLS-enabled email services** across multiple TCP ports.
- **Service Versions:** Service detection identified Pure-FTPd, Exim 4.99.5, Dovecot, BIND, and HAProxy 2.0.0+.
- **Security-Relevant Observations:** Multiple externally accessible services and infrastructure details were identified, providing useful information for subsequent authorized vulnerability assessment and configuration review.

These reconnaissance results established the target's domain, DNS, web, email, and network-service profile and provided the foundation for the subsequent phases of authorized security testing.

---
# 5. Findings
## Finding 1: SQL Injection in Patient Portal Authentication                                                                                                                
Location: POST /patient/login.php, parameter username Class: CWE-89 (SQL Injection) Severity Rating: 🔴 CRITICAL                                                        
Description                                                                                                                                                             
The patient login concatenates user input directly into a MySQL query without any parameterization, escaping, or prepared statements. Verbose error messages echo raw mysqli_query() SQL syntax errors back to unauthenticated users, confirming the injection point and leaking database structure.                                                                                                                                                                                                             

<img width="372" height="404" alt="login php sql injection" src="https://github.com/user-attachments/assets/01650d22-a016-4e62-a35e-1fba294f529c" />

**can be access with username: admin'-- -**
**password: anything**

<img width="1366" height="582" alt="3pdfs" src="https://github.com/user-attachments/assets/05df1a12-0a3e-410e-8b8c-5ef0a1558c8e" />

## Finding 2: Unauthorized Access to Restricted Patient Portal (Broken Access Control)                                                                                     
Location: GET /patient/portal.php (post-authentication) Class: CWE-287 (Improper Authentication), CWE-862 (Missing Authorization) Severity Rating: 🔴 CRITICAL          Description                                                                                                                                                              
Following the SQL injection bypass, a valid PHP session (PHPSESSID) was issued. The portal then listed all lab reports available to the compromised account with direct download links.  

<img width="1366" height="582" alt="Broken Access Control" src="https://github.com/user-attachments/assets/cba0d148-3e41-4ed5-899f-3503de8b149c" />

**Note: The admin account appears to be an administrative/principal account whose report listing includes multiple patients — a horizontal/vertical authorization weakness in itself. A true patient account should see only its own reports.**

## Finding 3: Insecure Direct Object Reference (IDOR) on Report Download                                                                                                  
Location: GET /patient/download.php?id=<n> Class: CWE-639 (Authorization Bypass Through User-Controlled Key), CWE-22 (Path Traversal — suspected, untested) Severity Rating: 🔴 CRITICAL (confirmed)      
untested potential for arbitrary file read                                                                                                                                Description                                                                                                                                                               
Report downloads are keyed by a sequential integer id (1, 2, 3). With a hijacked session, all three reports were retrieved. Sequential IDs mean any patient's report is one URL edit away for any authenticated session. The ?file= parameter also appeared in testing (returned 302 unauthenticated) — path traversal/LFI on this endpoint is a likely follow-up finding. 
**command: curl -s -b /tmp/cookies.txt "https://medirozahospital.com/patient/portal.php"**

<img width="926" height="735" alt="IDOR" src="https://github.com/user-attachments/assets/c894a4ff-b8e4-430c-9ded-ee907949fda0" />

## Finding 4: Weak PDF Encryption — Password Recoverable Offline 

One of the retrieved pathology reports was protected using legacy PDF encryption. Severity Rating: 🔴 CRITICAL

An offline password-strength assessment was performed against the authorized copy of the PDF. The password was recovered rapidly using a common dictionary, demonstrating that the protection depended on a trivially guessable password.

After the password was recovered, the PDF was successfully decrypted and its contents were reviewed as part of the authorized assessment.

### Security Significance

Encryption provides limited protection when the encryption password is weak and easily recoverable. 

The weakness is particularly significant because it was combined with the preceding unauthorized file-access issue. Once an encrypted document has been obtained, an easily guessable password can substantially reduce the effectiveness of the document's confidentiality control.

## Finding 5: Directory Listing on Sensitive Application Folders                                                                                                          
Location: /patient/, /old/, /_autoindex/ Class: CWE-548 (Exposure of Information Through Directory Listing), CWE-538 Severity Rating: 🟠 HIGH                             Description                                                                                                                                                               Multiple directories serve full autoindex listings, exposing application architecture, file inventory, and an active PHP error_log (242 KB) at /patient/error_log (contents access-blocked at time of test but its presence confirms ongoing error logging — a reconnaissance goldmine). 

<img width="1772" height="895" alt="Directory Listing on Sensitive Application Folders" src="https://github.com/user-attachments/assets/d26078e0-0249-4b90-bcb2-d639c90b4101" />

## Finding 6: Information Disclosure via Response Headers & Meta Tags                                                                                                   
Location: All responses Class: CWE-200 (Exposure of Sensitive Information) Severity Rating: 🟡 MEDIUM                                                                    Description                                                                                                                                                               
The application gratuitously fingerprints itself, aiding targeted exploitation.   

**Command: curl -s -i -L medirozahospital.com --max-time 30**
<img width="621" height="522" alt="Information Disclosure via Response Headers   Meta Tags" src="https://github.com/user-attachments/assets/0f473fcc-db38-4cbf-8425-a3698e32ef0d" />

## Finding 7: Username Enumeration via Differential Error Messages                                                                                                         
Location: POST /patient/login.php Class: CWE-204 (Observable Response Discrepancy) Severity Rating: 🟡 MEDIUM                                                           Description                                                                                                                                                             
"Username not found" vs. "Incorrect password" responses allow an attacker to confirm valid account names without any credentials, then focus password attacks. Confirmed admin exists on the patient portal (questionable design — an administrative account on a patient-facing system).

<img width="1876" height="907" alt="Username Enumeration via Differential Error Messages" src="https://github.com/user-attachments/assets/ab10214d-d534-4dcf-b532-cc769a725cb9" />

---
# M2
## 6. Practical Modules
| | |
|---|---|
| **Target files** | `patient_report_1.pdf`, `patient_report_2.pdf`, `patient_report_3.pdf` (password-protected) |
| **Module 1** | Password Cracking with NetworkWalks' online Hash Calculator & Password Cracker |
| **Module 2** | Password Cracking with NetworkWalks' online Hash Calculator & John the Ripper |
| **Attack type** | Dictionary attack |

**Cracked passwords:**

| File | Password |
|---|---|
| `patient_report_1.pdf` | `123456` |
| `patient_report_2.pdf` | `password` |
| `patient_report_3.pdf` | `!@#$%^&` |

<img width="848" height="239" alt="795252112_1135045865850419_1901597943272309923_n" src="https://github.com/user-attachments/assets/7d861189-9ee1-4e04-a863-545544745909" />
<img width="875" height="528" alt="797624517_1414435040622012_5465545025558426440_n" src="https://github.com/user-attachments/assets/08a99dc5-9ea6-462b-831e-c1d59f5fd74b" />
<img width="887" height="497" alt="797752886_1416563100404735_5124007249446300839_n" src="https://github.com/user-attachments/assets/1e68c065-396a-4ef1-bc23-28b735edde92" />
<img width="876" height="581" alt="796568020_2554561831688686_1699458589611673404_n" src="https://github.com/user-attachments/assets/ec598c57-ae40-44a3-aa55-cf8d0835b237" />

the first two pdf is so easy to cracked the password with the use of networkwalks hashed and input that in networkwalks password cracker.**

<img width="1329" height="721" alt="pdf3 hash" src="https://github.com/user-attachments/assets/ca1c6e1a-0a2a-4818-b0da-9bfd10cbb678" />

1st step is to get the hash of the pdf

<img width="1141" height="76" alt="pdf3hash to file" src="https://github.com/user-attachments/assets/e05f6c01-bc79-493e-964f-a6846e2d1992" />

2nd step is to put the hash into a file and with the hash value: 4294967292 replace it with -4
formula: 2^32 is equals to 4294967296-4294967292= 4
command: echo "$pdf$2*3*128*-4*1*32*3261393066326130336634386337323631306164373264323130316137616538*32*5090fa0a5dba99cb97c9d140cd23119428bf4e5e4e758a4164004e56fffa0108*32*58e03d692cf37b50b0b5eaa189fcbd372260a949c8992ad7b44fd13e2b40c1f8" > report3_hash_signed.txt

<img width="1141" height="76" alt="pdf3hash to file" src="https://github.com/user-attachments/assets/bd2fef7d-30ab-4325-acdd-1f9e8100c9aa" />

last step is to use john the ripper: john --format=PDF report3_hash_signed.txt :

<img width="765" height="240" alt="pdf3 password" src="https://github.com/user-attachments/assets/2055b32c-7374-4a6f-9bf5-8b9e11e33dab" />

**Unlocked PDF's**
<img width="1003" height="726" alt="pdf1 unlocked" src="https://github.com/user-attachments/assets/b7932aeb-421d-442e-9a53-6bf3d94733cd" />
<img width="944" height="691" alt="pdf2 unlocked" src="https://github.com/user-attachments/assets/9b18720a-9b28-4eba-8178-d590b0376a32" />
<img width="944" height="691" alt="pdf2 unlocked" src="https://github.com/user-attachments/assets/9f35c88b-6276-4706-aa9f-f7bacf573457" />

---
# M3
## Finding 8: Publicly Exposed Database Backup (Directory Listing + Sensitive File in Web Root)                                                                            
Location: https://medirozahospital.com/old/mediroza_db_backup_2019.sql Class: CWE-538 (File & Directory Information Exposure), CWE-200 (Exposure of Sensitive Information) Severity Rating: 🔴 CRITICAL
Description                                                                                                                                                               
The /old/ directory has directory listing (autoindex) enabled and contains an unencrypted 2019 HR database backup (6,346 bytes), freely downloadable without any authentication. The directory is even advertised in robots.txt (Disallow: /old/), serving as a roadmap for any attacker. 

<img width="801" height="527" alt="Publicly Exposed Database Backup" src="https://github.com/user-attachments/assets/933466fd-2385-4b6c-8052-6c87665b4fc7" />

<img width="801" height="527" alt="Publicly Exposed Database Backup" src="https://github.com/user-attachments/assets/51b0c433-6f9c-49c3-bb09-6c29a39b1edf" />

<img width="802" height="515" alt="Publicly Exposed Database Backup3" src="https://github.com/user-attachments/assets/fffa210f-6a9b-461f-a2f2-8d4c77daf17b" />

<img width="802" height="515" alt="Publicly Exposed Database Backup3" src="https://github.com/user-attachments/assets/ee8af681-1b34-41a9-b4ff-59bd67483bb1" />

# 6. Recommendations &amp; Remediation</h1>

## Finding 1 — SQL Injection in Patient Portal Authentication (CRITICAL)</h2>
<ul>
  <li>Rewrite all database queries using <strong>parameterized queries / prepared statements</strong> (e.g., PDO with bound parameters or <code>mysqli_stmt</code>). Never concatenate user input into SQL strings.</li>
  <li>Apply strict server-side input validation on the <code>username</code> and <code>password</code> fields (allow-list expected character sets; reject SQL metacharacters where not needed).</li>
  <li>Disable verbose database error output in production (<code>display_errors = Off</code> in <code>php.ini</code>); log errors server-side only, never echo raw <code>mysqli_query()</code> errors to the client.</li>
  <li>Run a full source-code audit of every form and endpoint in Mediroza CMS 1.4.2 for the same unparameterized-query pattern — this is rarely an isolated instance.</li>
  <li>Deploy a properly configured WAF/ModSecurity rule set tuned for SQLi signatures as a compensating control, not a substitute for code-level fixes.</li>
</ul>

## Finding 2 — Unauthorized Access to Restricted Patient Portal / Broken Access Control (CRITICAL)</h2>
<ul>
  <li>Enforce <strong>server-side authorization checks</strong> on every portal request — validate that the authenticated session's role and patient ID match the records being requested, not just that a session exists.</li>
  <li>Remove or strictly scope the administrative account so it cannot be reached through the patient-facing login; separate staff/admin authentication onto a distinct, more hardened endpoint (ideally with MFA).</li>
  <li>Implement role-based access control (RBAC) so a "patient" role can only ever query its own record set, enforced at the query layer (e.g., <code>WHERE patient_id = :session_patient_id</code>), not just hidden in the UI.</li>
  <li>Add centralized session validation middleware rather than relying on per-page ad hoc checks.</li>
</ul>

## Finding 3 — IDOR on Report Download (CRITICAL)</h2>
<ul>
  <li>Replace sequential integer <code>id</code> values with <strong>non-guessable, per-user opaque identifiers</strong> (UUIDs) or, better, server-side ownership checks that verify the requested report belongs to the requesting session before serving it.</li>
  <li>Add authorization middleware to <code>download.php</code> that rejects any request where <code>report.patient_id != session.patient_id</code>.</li>
  <li>Investigate and close the untested <code>?file=</code> parameter immediately — treat it as a suspected path traversal / LFI vector until proven otherwise; sanitize and canonicalize any file path input, and serve files from a locked-down directory outside the web root using an internal file reference rather than a user-supplied name/path.</li>
  <li>Log and alert on sequential/out-of-range ID access attempts as a detection control.</li>
</ul>

## Finding 4 — Weak PDF Encryption / Recoverable Password (CRITICAL)</h2>
<ul>
  <li>Move away from static, human-chosen PDF passwords for PHI documents. Use strong, randomly generated, per-document passwords (16+ characters) issued through a secure delivery channel, or better, eliminate password-only protection in favor of authenticated, access-controlled document delivery (e.g., signed, expiring download links tied to a verified session).</li>
  <li>Enforce a password policy that bans dictionary words (<code>123456</code>, <code>password</code>, common patterns) anywhere passwords are generated or accepted, including for internal document protection.</li>
  <li>Consider AES-256 PDF encryption (modern <code>qpdf</code>/Acrobat standard) rather than legacy RC4/40–128-bit schemes, and disable legacy encryption support in whatever tool generates these reports.</li>
  <li>Treat this as a compensating control only — the root fix is closing Finding 3 so encrypted files are never reachable by unauthorized users in the first place.</li>
</ul>

## Finding 5 — Directory Listing on Sensitive Application Folders (HIGH)</h2>
<ul>
  <li>Disable directory autoindexing globally in the LiteSpeed/Apache config (<code>Options -Indexes</code>) and specifically for <code>/patient/</code>, <code>/old/</code>, <code>/_autoindex/</code>.</li>
  <li>Remove the <code>/old/</code> directory and its contents from the web root entirely — legacy backups should never be stored inside a publicly reachable path; move to secured, non-web-accessible storage with access logging.</li>
  <li>Relocate or restrict access to <code>error_log</code> files outside the web root; rotate and purge logs regularly, and ensure logs never capture sensitive query parameters or credentials.</li>
  <li>Run a full inventory of the web root for other stale/legacy files (<code>.bak</code>, <code>.old</code>, <code>.sql</code>, <code>.zip</code>) and remove or relocate them.</li>
</ul>

## Finding 6 — Information Disclosure via Response Headers &amp; Meta Tags (MEDIUM)</h2>
<ul>
  <li>Strip or generalize identifying headers (<code>X-Powered-By</code>, <code>x-turbo-charged-by</code>, server version banners) at the web server/reverse-proxy level.</li>
  <li>Remove CMS/version-revealing meta tags and generator tags from HTML source.</li>
  <li>Adopt a standard secure-headers baseline: <code>Content-Security-Policy</code>, <code>X-Content-Type-Options: nosniff</code>, <code>X-Frame-Options</code>, <code>Referrer-Policy</code>, <code>Strict-Transport-Security</code>.</li>
</ul>

## Finding 7 — Username Enumeration via Differential Error Messages (MEDIUM)</h2>
<ul>
  <li>Return a single, generic authentication error (e.g., "Invalid username or password") for both invalid-username and invalid-password cases.</li>
  <li>Apply consistent response timing/behavior regardless of whether the username exists, to prevent timing-based enumeration.</li>
  <li>Implement account lockout / progressive throttling and CAPTCHA after repeated failed login attempts from the same source.</li>
  <li>Reassess whether an administrative account should exist on a patient-facing login at all (see Finding 2).</li>
</ul>

## Finding 8 — Publicly Exposed Database Backup (CRITICAL)</h2>
<ul>
  <li>Immediately remove <code>mediroza_db_backup_2019.sql</code> and any other database dumps from the web-accessible file system.</li>
  <li>Never rely on <code>robots.txt Disallow</code> as an access control — it is advisory only for well-behaved crawlers and effectively signposts sensitive paths to attackers. Enforce actual authentication/network-level restrictions on any path that must remain reachable.</li>
  <li>Establish a backup handling policy: backups are encrypted at rest, stored outside the web root (or off-host, e.g., in access-controlled cloud storage), and access is logged and reviewed.</li>
  <li>Since this backup contained HR/employee data, treat it as a confirmed breach of employee PII — begin incident response and notification procedures in parallel with technical remediation.</li>
</ul>

---

# 👤 Author

**Genrei L. Bondoc**

**Cybersecurity / Ethical Hacking Intern**

**Networkwalks — Batch B082**

**Report End — CONFIDENTIAL**

**Prepared by the authorized penetration-testing team — 07 September 2026**

**This document contains confidential vulnerability and security-assessment information. Distribution should be restricted to authorized client stakeholders, security personnel, and remediation owners.**
