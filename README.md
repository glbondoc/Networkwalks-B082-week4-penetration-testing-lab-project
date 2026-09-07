# Penetration Testing Report
Mediroza General Hospital — External Web Application Assessment

Cybersecurity | Networkwalks

| Field | Details |
|---|---|
| **Pentester Name** | Genrei L. Bondoc |
| **Program/Batch** | B082 |
| **Date** | 06 September 2026 |
| **Modules Completed** | <br> Web Penetration Testing |
| **Client/Target** | Mediroza General Hospital — `https://medirozahospital.com` |
| **Permission Secured from Client?** | Yes |
| **Engagement Type** | External Black-Box Web Application Penetration Test |
| **Testing Period** | Single session — approximately 1.5 hours |
| **Overall Risk Rating** | 🔴 **CRITICAL** |
| **Phases Covered** | Phase 1: Reconnaissance & Footprinting <br> Phase 2: Web Application Scanning & Discovery <br> Phase 3: Authentication & Authorization Testing <br> Phase 4: Vulnerability Exploitation & Data Exposure Assessment |
---

## 1. Liability Disclaimer

<p>All penetration-testing activities documented in this report were conducted only within the authorized scope of the engagement and in accordance with the applicable rules of engagement. The assessment was performed for authorized security testing and educational purposes. No denial-of-service or destructive testing was performed. Sensitive evidence obtained during testing was handled according to the engagement's data-handling requirements.</p>

<p>The information contained in this report is confidential and is intended only for authorized client stakeholders and remediation personnel. Vulnerability details, evidence, and testing procedures must not be used against systems without explicit authorization. Unauthorized access, testing, disclosure, or misuse of security information may result in legal, financial, employment, or other consequences.</p>

---

## 2. Introduction

This report documents an external black-box penetration test conducted against the web presence of Mediroza General Hospital (`medirozahospital.com`). The assessment simulated an unauthenticated external attacker with no prior knowledge of the organization's internal environment.

The assessment covered reconnaissance, attack-surface mapping, authentication testing, input-validation testing, authorization testing, file-access testing, and controlled post-exploitation analysis. The objective was to determine whether publicly accessible weaknesses could be chained together to obtain unauthorized access to protected application resources and sensitive information.

The assessment identified multiple critical security weaknesses, including a publicly accessible database backup, SQL injection in the patient authentication endpoint, broken access controls, insecure direct object references, weak PDF protection, directory listing, information disclosure, and username enumeration.

The testing demonstrated that several of these weaknesses could be combined into an attack path resulting in unauthorized access to confidential patient reports and exposure of sensitive employee and corporate information.

All testing was performed during a single authorized session. Destructive testing and denial-of-service activities were excluded from the engagement.

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
### 4.1 Footprinting & Reconnaissance
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

## 4.2 Authentication & SQL Injection Testing

The patient authentication endpoint was tested for input-validation weaknesses.

Testing identified verbose SQL error messages when malformed input was supplied to the username parameter. The application response disclosed database-related SQL syntax information, confirming that user-controlled input was being incorporated into a database query without adequate parameterization.

Differential responses also demonstrated that the application distinguished between existing and nonexistent usernames.

Further authorized testing confirmed that the authentication mechanism could be bypassed through SQL injection, resulting in an authenticated application session.

### Security Significance

Authentication controls are intended to ensure that only legitimate users can establish authenticated sessions. A SQL injection vulnerability in the login process can undermine that control entirely.

The issue therefore represents a critical authentication and input-validation weakness.
<img width="372" height="404" alt="login php sql injection" src="https://github.com/user-attachments/assets/01650d22-a016-4e62-a35e-1fba294f529c" />

**can be access with username: admin'-- -**
**password: anything**

### Severity

🔴 **Critical**

---

## 4.3 Unauthorized Access to the Patient Portal

Following the authentication weakness identified during testing, an authenticated application session was established.

The patient portal subsequently displayed multiple pathology reports associated with different patients rather than restricting the authenticated session to a single authorized patient's records.

The portal exposed direct report-download functionality associated with sequential identifiers.

### Security Significance

A patient-facing system should enforce server-side authorization checks to ensure that a user can access only records belonging to that user.

The observed behavior indicates a significant access-control weakness that could allow cross-patient access to confidential medical information.

### Severity

🔴 **Critical**

---

## 4.4 PDF Protection & Offline Password Assessment

One of the retrieved pathology reports was protected using legacy PDF encryption.

An offline password-strength assessment was performed against the authorized copy of the PDF. The password was recovered rapidly using a common dictionary, demonstrating that the protection depended on a trivially guessable password.

After the password was recovered, the PDF was successfully decrypted and its contents were reviewed as part of the authorized assessment.

### Security Significance

Encryption provides limited protection when the encryption password is weak and easily recoverable.

The weakness is particularly significant because it was combined with the preceding unauthorized file-access issue. Once an encrypted document has been obtained, an easily guessable password can substantially reduce the effectiveness of the document's confidentiality control.

### Severity

🟠 **High**


## Overall Risk Assessment

The overall security posture identified during this assessment is rated **🔴 CRITICAL**.

The most significant concern is not any individual finding in isolation, but the ability to combine several weaknesses into a practical attack path.

The assessment demonstrated a progression from:

**Reconnaissance → Sensitive File Discovery → Authentication Weakness → Unauthorized Portal Access → Report Access → Weak Document Protection**

This chain resulted in unauthorized exposure of sensitive healthcare, employee, and corporate information.

The findings should therefore be treated as an **incident-level security concern** rather than isolated configuration issues.

---

# 6. Recommendations

## 6.1 Immediate Remediation

- Remove the Exposed Database Backup
- Remove the SQL backup and any other sensitive backups from all web-accessible directories. Assume that publicly accessible information may already have been copied.
- Disable directory indexing
- Disable autoindexing for /old/, /patient/, /_autoindex/, and other application directories.
- Secure the authentication process
- Replace dynamically constructed SQL queries with parameterized queries or prepared statements.
- Review and invalidate potentially compromised sessions
- Invalidate active sessions and review authentication logs for suspicious activity.
- Protect application logs
- Move server-side logs outside the web root and ensure that log files cannot be directly accessed through HTTP.
- Begin a formal breach-impact assessment
- Because healthcare and employee information was exposed during testing, the organization should conduct an appropriate legal, privacy, and incident-response assessment.
## 6.2 Short-Term Remediation
- Implement server-side authorization checks
- Every report request should verify that the authenticated user is authorized to access the requested record.
- Separate patient and administrative authentication
- Administrative accounts should not use the same authentication pathway as ordinary patient accounts.
- Use generic authentication error messages
- Replace differentiated login responses with a consistent message such as:
- Invalid username or password.
- Improve document encryption
- Use modern PDF encryption and strong, randomly generated passwords where password-protected document delivery is required.
- Remove unnecessary technology fingerprinting
- Avoid exposing unnecessary server, PHP, CMS, and organizational metadata.
- Implement authentication rate limiting
- Add account/IP-based rate limiting and temporary lockout controls to reduce automated authentication attacks.
Review robots.txt
- Do not rely on robots.txt to protect sensitive directories. Sensitive resources must be protected through authentication and authorization controls.
## 6.3 Medium-Term Security Improvements
- Establish a Secure Software Development Lifecycle
- Introduce security-focused code reviews, SAST, DAST, dependency scanning, and security testing before production deployment.
- Apply least privilege to database accounts
- The web application's database account should have only the privileges required for normal application operation.
- Implement data-classification controls
- Employee, medical, identification, and shareholder information should be classified as sensitive and prohibited from web-accessible storage.
- Centralize security logging and monitoring
- Monitor authentication failures, unusual report requests, sensitive-file requests, SQL errors, and directory enumeration.
- Develop an incident-response procedure
- Establish procedures for identifying, containing, investigating, and reporting potential exposure of patient and employee information.
- Perform a complete remediation retest
- After remediation, retest all eight findings and verify that the identified attack paths can no longer be reproduced.
# 7. Conclusion
- During this external penetration-testing assessment of Mediroza General Hospital, multiple security weaknesses were identified across the organization's publicly accessible web application.
- The assessment began with reconnaissance and attack-surface discovery, which revealed publicly accessible application directories and a sensitive database backup. Subsequent security testing identified weaknesses in authentication, input validation, authorization, object-level access control, document protection, directory configuration, and information disclosure.
The most serious findings were the publicly accessible database backup, SQL injection in the patient login, broken access control, and IDOR in the report-download functionality. These weaknesses could be chained together to obtain unauthorized access to confidential patient records.
- The assessment also demonstrated that legacy document encryption and weak passwords could further reduce the effectiveness of confidentiality controls after protected files had been obtained.
- Overall, the findings demonstrate that security controls must be implemented as a complete defense-in-depth system. Authentication alone is insufficient when SQL injection can bypass it, and authentication is insufficient when authorization checks do not restrict access to individual records.
- The highest priority should therefore be given to removing publicly exposed sensitive files, fixing SQL injection, implementing strict server-side authorization, disabling directory indexing, protecting logs, and conducting an appropriate incident-response and privacy assessment.
- Most importantly, all penetration-testing activities must remain within an explicitly authorized scope. The testing documented in this report was conducted under the defined engagement rules and was intended to identify weaknesses so that appropriate corrective measures could be implemented.

# 8. Evidence Collected
## 8.1 Reconnaissance Evidence

### Task 1 — HTTP/HTTPS Reconnaissance

curl -s -i http://medirozahospital.com
curl -s -i -L https://medirozahospital.com

Observed: HTTP redirection, LiteSpeed infrastructure, PHP and CMS information.

### Task 2 — robots.txt Review

curl -s -i https://medirozahospital.com/robots.txt

Observed: References to sensitive application paths including /patient/, /staff/, and /old/.

## 8.2 Public Database Backup Evidence

### Task 3 — Directory Enumeration

curl -s -i https://medirozahospital.com/old/

Observed:

Index of /old/
mediroza_db_backup_2019.sql

The database backup contained employee and shareholder information and was accessible without authentication.

## 8.3 Authentication Testing Evidence

### Task 4 — Patient Authentication Testing

The patient login endpoint was tested using controlled input-validation cases.

Observed: SQL-related error disclosure and differential authentication responses.

The testing subsequently demonstrated that the authentication mechanism could be bypassed, resulting in an authenticated session.

## 8.4 Patient Portal Evidence

### Task 5 — Authenticated Portal Review

Following successful authentication testing, the patient portal was accessed using the authorized test session.

Observed: Multiple pathology reports associated with different patients were presented through the portal.

## 8.5 Report Download Evidence

### Task 6 — Object-Level Authorization Testing

The report-download functionality was assessed to determine whether access controls were enforced for individual report objects.

Observed: Multiple report objects could be retrieved through the authenticated test session, demonstrating insufficient object-level authorization.

## 8.6 PDF Security Evidence

### Task 7 — PDF Password Assessment

The authorized encrypted PDF sample was analyzed offline to assess the strength of its password protection.

Observed: The password was recovered rapidly using a common dictionary, demonstrating inadequate password strength.

The recovered password was used only for the authorized assessment and validation of the document-protection finding.

## 8.7 Directory Listing Evidence

### Task 8 — Sensitive Directory Review

The following paths were reviewed:

/patient/
/old/
/_autoindex/

Observed: Application files, report functionality, login/logout resources, and a server-side error log were exposed through directory indexing.

## 9. Attack Path Summary

The principal attack path identified during the assessment can be summarized as follows:

External Reconnaissance
        ↓
Public Directory Discovery
        ↓
Sensitive Database Backup Exposure
        ↓
Authentication Endpoint Testing
        ↓
SQL Injection / Authentication Bypass
        ↓
Authenticated Patient Portal Access
        ↓
Insufficient Authorization
        ↓
Cross-Patient Report Access
        ↓
Protected PDF Retrieval
        ↓
Offline Password-Strength Assessment
        ↓
Confidential Medical Information Exposure

This chain demonstrates how individually addressable weaknesses can combine to create a significantly greater overall security impact.

## 10. Evidence Inventory
Artifact	Purpose	Status
Database backup	Evidence for Finding 1	Retained securely under engagement controls
Pathology Report 1	Evidence for report-access testing	Retained securely
Pathology Report 2	Evidence for report-access testing	Retained securely
Pathology Report 3	Evidence for PDF-security testing	Retained securely
Decrypted test document	Evidence for Finding 5	Retained securely and access restricted
PDF password hash	Evidence for password-strength assessment	Retained securely
Session evidence	Evidence of authentication behavior	Session invalidated after testing
HTTP responses	Reconnaissance and vulnerability evidence	Retained as assessment evidence

All sensitive evidence should remain within the authorized evidence store and should be securely destroyed according to the engagement's data-retention and client sign-off requirements.

## 11. Note on Sensitive Information

For privacy and security reasons, sensitive patient, employee, identification, salary, authentication-session, and corporate information should be redacted from publicly distributed copies of this report.

The full technical evidence should be restricted to authorized client stakeholders, security personnel, legal/privacy personnel, and designated remediation owners.

# 👤 Author

### Genrei L. Bondoc
### Cybersecurity / Ethical Hacking Intern
### Networkwalks — Batch B082
### Report End — CONFIDENTIAL

### Prepared by the authorized penetration-testing team — 06 September 2026
### This document contains confidential vulnerability and security-assessment information. Distribution should be restricted to authorized client stakeholders, security personnel, and remediation owners.
