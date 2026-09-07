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

## 3. Tools Used

| Tool | Purpose |
|---|---|
| `curl` | HTTP/HTTPS requests, reconnaissance, authentication testing, session handling, and controlled file retrieval |
| `grep / sed` | Response parsing and evidence/hash normalization |
| `Python 3` | Parsing and analyzing the authorized database-backup evidence |
| `PyPDF2` | Authorized PDF decryption and text extraction |
| `pdf2john` | Extraction of PDF password hashes for offline assessment |
| `John the Ripper` | Offline password-strength assessment |
| `rockyou.txt` | Dictionary used for authorized password-strength testing |
| `file` | File-type verification |
| `md5sum` | Evidence integrity verification |
| `ls` | Evidence inventory and file verification |

---

# 4. Activities Performed

## 4.1 Reconnaissance & Attack Surface Mapping

I began the assessment by performing external reconnaissance against the Mediroza General Hospital web application. The objective was to identify publicly accessible resources, technologies, application directories, and potentially sensitive files.

Initial HTTP requests identified the web server and application technology. Further examination of `robots.txt`, publicly accessible directories, and application responses revealed several areas requiring additional investigation.

### Reconnaissance Findings

The reconnaissance phase identified:

- LiteSpeed web-server infrastructure
- PHP 8.2.33
- Mediroza CMS 1.4.2
- Publicly accessible application directories
- `/patient/`, `/staff/`, and `/old/` paths referenced through `robots.txt`
- Directory-indexing behavior
- A potentially sensitive database backup located within the web-accessible `/old/` directory
- Application endpoints associated with authentication and report retrieval

These observations provided the foundation for subsequent authorized security testing.

---

## 4.2 Publicly Exposed Database Backup

A directory listing was identified under the `/old/` path. The directory contained an unencrypted SQL database backup that was accessible without authentication.

### Exposed Information

The backup contained sensitive employee and shareholder information, including:

- Employee names
- Job titles and departments
- Corporate contact information
- National identification numbers
- Monthly salaries
- Shareholder ownership information
- Share counts and share classes

The exposed backup was approximately **6,346 bytes** and contained records for **30 employees and 10 shareholders**.

### Security Significance

A database backup should never be stored in a publicly accessible web directory. Because the file could be retrieved without authentication, an external attacker would not need to compromise the application before obtaining the information contained within it.

This represents a direct confidentiality failure and significantly increases the risk of identity theft, targeted social engineering, insider targeting, and corporate information exposure.

### Severity

🔴 **Critical**

---

## 4.3 Authentication & SQL Injection Testing

The patient authentication endpoint was tested for input-validation weaknesses.

Testing identified verbose SQL error messages when malformed input was supplied to the username parameter. The application response disclosed database-related SQL syntax information, confirming that user-controlled input was being incorporated into a database query without adequate parameterization.

Differential responses also demonstrated that the application distinguished between existing and nonexistent usernames.

Further authorized testing confirmed that the authentication mechanism could be bypassed through SQL injection, resulting in an authenticated application session.

### Security Significance

Authentication controls are intended to ensure that only legitimate users can establish authenticated sessions. A SQL injection vulnerability in the login process can undermine that control entirely.

The issue therefore represents a critical authentication and input-validation weakness.

### Severity

🔴 **Critical**

---

## 4.4 Unauthorized Access to the Patient Portal

Following the authentication weakness identified during testing, an authenticated application session was established.

The patient portal subsequently displayed multiple pathology reports associated with different patients rather than restricting the authenticated session to a single authorized patient's records.

The portal exposed direct report-download functionality associated with sequential identifiers.

### Security Significance

A patient-facing system should enforce server-side authorization checks to ensure that a user can access only records belonging to that user.

The observed behavior indicates a significant access-control weakness that could allow cross-patient access to confidential medical information.

### Severity

🔴 **Critical**

---

## 4.5 Insecure Direct Object Reference (IDOR)

The report-download functionality used sequential numeric identifiers to identify individual reports.

During authorized testing, multiple report objects were successfully retrieved through the compromised authenticated session. The behavior demonstrated that the application did not sufficiently enforce ownership checks between the authenticated user and the requested report object.

### Security Significance

Using sequential identifiers is not inherently a vulnerability; the critical issue is the absence of effective server-side authorization checks.

If authorization is not verified for each requested record, an authenticated user may potentially access another patient's medical records simply by changing an object identifier.

### Severity

🔴 **Critical**

---

## 4.6 PDF Protection & Offline Password Assessment

One of the retrieved pathology reports was protected using legacy PDF encryption.

An offline password-strength assessment was performed against the authorized copy of the PDF. The password was recovered rapidly using a common dictionary, demonstrating that the protection depended on a trivially guessable password.

After the password was recovered, the PDF was successfully decrypted and its contents were reviewed as part of the authorized assessment.

### Security Significance

Encryption provides limited protection when the encryption password is weak and easily recoverable.

The weakness is particularly significant because it was combined with the preceding unauthorized file-access issue. Once an encrypted document has been obtained, an easily guessable password can substantially reduce the effectiveness of the document's confidentiality control.

### Severity

🟠 **High**

---

## 4.8 Information Disclosure Through Headers and Meta Tags

The application disclosed several technical details through HTTP response headers and HTML metadata.

Observed information included:

- LiteSpeed server identification
- PHP 8.2.33
- Mediroza CMS 1.4.2
- Mediroza IT Department attribution

### Security Significance

Technology disclosure does not automatically constitute a vulnerability. However, detailed version information can assist attackers in researching platform-specific vulnerabilities and constructing more targeted attacks.

**Severity: 🟡 Medium**

---

## 4.9 Username Enumeration

The patient login endpoint returned different messages depending on whether the supplied username existed.

For example, the application differentiated between an incorrect password and a nonexistent username.

### Security Significance

Observable differences in authentication responses can allow attackers to build a list of valid accounts.

When combined with an authentication vulnerability, username enumeration can further reduce the effort required to compromise accounts.

**Severity: 🟡 Medium**

---

# 5. Risk Analysis / Impact

Based on the findings identified during the assessment, the following risks were documented.

| # | Risk / Finding | Evidence / Observation | Potential Impact | Risk Level |
|---|---|---|---|---|
| 1 | Publicly exposed database backup | SQL backup accessible through publicly indexed directory | Exposure of employee, salary, national-ID, and shareholder information | 🔴 Critical |
| 2 | SQL Injection in patient login | User input generated SQL errors and allowed authentication bypass | Authentication compromise and potential database exposure | 🔴 Critical |
| 3 | Broken access control | Patient portal exposed reports associated with multiple patients | Unauthorized disclosure of medical records | 🔴 Critical |
| 4 | IDOR in report download | Sequential report identifiers lacked adequate ownership enforcement | Cross-patient access to confidential reports | 🔴 Critical |
| 5 | Weak PDF encryption/password | Legacy encryption and trivially recoverable password | Reduced confidentiality of downloaded medical reports | 🟠 High |
| 6 | Directory listing | Sensitive application files and error log exposed | Application reconnaissance and sensitive-file discovery | 🟠 High |
| 7 | Technology/version disclosure | Server, PHP, CMS and organizational information exposed | Targeted exploitation and reconnaissance assistance | 🟡 Medium |
| 8 | Username enumeration | Different responses for valid and invalid usernames | Account discovery and targeted authentication attacks | 🟡 Medium |

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
