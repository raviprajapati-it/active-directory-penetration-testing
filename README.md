# Internal Penetration Testing — SMB Enumeration, SQL Injection & Webmin RCE

Authorized penetration-testing lab documenting **network reconnaissance, SMB enumeration, web application assessment, SQL injection, credential-security weaknesses, Webmin remote code execution, root compromise, and post-exploitation analysis**.

> **Repository naming note:** The repository is named `active-directory-penetration-testing` for portfolio consistency, but the original assessment was broader than a dedicated Active Directory engagement. The preserved evidence primarily supports internal-network, SMB, web-application, administrative-service, exploitation, and post-exploitation testing.

---

## Project Overview

The assessment was conducted against the **KingsCollege virtual machine**, an intentionally vulnerable academic lab environment.

The testing followed a structured penetration-testing lifecycle:

```text
Reconnaissance
      │
      ▼
Service Discovery
      │
      ▼
Enumeration
      │
      ▼
Vulnerability Assessment
      │
      ▼
Controlled Exploitation
      │
      ▼
Privilege Validation
      │
      ▼
Post-Exploitation
      │
      ▼
Remediation
```

The strongest attack path combined several weaknesses rather than relying on one isolated vulnerability.

---

# Assessment Highlights

| Area | Result |
| --- | --- |
| Network reconnaissance | Six exposed services identified |
| SMB enumeration | User-account and password-policy information exposed |
| Web application assessment | Kings Module Directory identified as a critical attack surface |
| SQL injection | Confirmed and exploited |
| Database access | Multiple backend databases enumerated |
| Webmin assessment | Version 1.984 identified |
| CVE validation | CVE-2022-0824 successfully exploited |
| Remote shell | Reverse shell established |
| Privilege impact | Root-level access confirmed |
| Post-exploitation | Persistence and anti-forensic risk demonstrated |
| Remediation | Technical recommendations documented |

---

# Attack Surface Discovery

Initial scanning identified services including:

```text
21      FTP
22      SSH
80      HTTP
139/445 SMB
10000   Webmin
11235   HTTP
```

The highest-value attack surfaces became:

```text
SMB
Kings Module Directory
Webmin
```

Not every exposed service was exploitable, which is an important distinction in penetration testing.

---

# SMB Enumeration

SMB enumeration was performed using:

```text
enum4linux
```

The assessment identified multiple valid local user accounts and weak authentication policy controls.

Observed weaknesses included:

```text
Password complexity: Disabled
Minimum password length: 5 characters
Account lockout: Not configured
```

This materially increased the authentication attack surface.

Conceptually:

```text
SMB Enumeration
      │
      ▼
Valid Accounts
      │
      +
Weak Password Policy
      │
      ▼
Higher Authentication Risk
```

---

# Kings Module Directory

A second web application was discovered on:

```text
TCP/11235
```

The application exposed resources including:

```text
/results.php
/index.php
/phpmyadmin/
/initial_sql_scripts/
```

The exposed SQL-related resources provided useful information about the backend data structure.

---

# SQL Injection

The `/results.php` search functionality was found to process user input insecurely.

Testing was performed using:

```text
Burp Suite
sqlmap
```

The vulnerability was confirmed as:

```text
Time-based blind SQL injection
UNION-based SQL injection
```

Successful exploitation enabled unauthorized database enumeration and extraction.

---

## Database Impact

The assessment identified databases including:

```text
db_backend
kings_wordpress
```

This demonstrated how a vulnerability in one application could expose multiple databases hosted within the same environment.

Conceptually:

```text
Kings Module Directory
        │
        ▼
SQL Injection
        │
        ▼
Database Server
      /     \
     /       \
db_backend  kings_wordpress
```

---

# Webmin Remote Code Execution

Nmap identified:

```text
Webmin 1.984
TCP/10000
```

The installed release matched:

```text
CVE-2022-0824
```

an authenticated remote-code-execution vulnerability.

The weak password controls discovered during SMB enumeration reduced the effectiveness of the authentication boundary.

---

## Exploitation Path

```text
User Enumeration
      │
      ▼
Weak Password Policy
      │
      ▼
Webmin Authentication
      │
      ▼
Webmin 1.984
      │
      ▼
CVE-2022-0824
      │
      ▼
Remote Code Execution
      │
      ▼
Root Shell
```

A public proof-of-concept associated with:

```text
Exploit-DB 50809
```

was used in the authorized lab to validate the vulnerability.

---

# Root-Level Compromise

Successful exploitation produced a reverse shell running with root privileges.

Root access demonstrated impact including:

- Full filesystem access
- Service manipulation
- Configuration changes
- Access to application data
- Persistence capability
- Log manipulation
- Administrative control of the operating system

This changed the finding from:

```text
Vulnerable service
```

to:

```text
Complete host compromise
```

---

# Post-Exploitation

Following root compromise, limited post-exploitation activity was performed to demonstrate security impact.

The assessment included:

- Filesystem exploration
- Privilege verification
- Persistence
- Basic anti-forensic activity
- Cleanup of temporary artifacts

A systemd service was used in the lab to demonstrate persistence across restarts.

---

# Evidence

## 01 — Nmap Service Discovery

![Nmap service discovery](evidence/screenshots/01-nmap-service-discovery.png)

## 02 — SMB Enumeration

![SMB enumeration](evidence/screenshots/02-smb-enumeration.png)

## 03 — Kings Module Directory

![Kings Module Directory](evidence/screenshots/03-kings-module-directory.png)

## 04 — SQL Injection Validation

![SQL injection validation](evidence/screenshots/04-sql-injection-validation.png)

## 05 — Database Enumeration

![Database enumeration](evidence/screenshots/05-database-enumeration.png)

## 06 — Webmin Version & Authentication

![Webmin version and login](evidence/screenshots/06-webmin-version-and-login.png)

## 07 — Webmin RCE / Root Shell

![Webmin RCE root shell](evidence/screenshots/07-webmin-rce-root-shell.png)

## 08 — Root Access Validation

![Root access validation](evidence/screenshots/08-root-access-validation.png)

## 09 — Post-Exploitation Persistence

![Post-exploitation persistence](evidence/screenshots/09-post-exploitation-persistence.png)

---

# Findings Summary

| ID | Finding | Severity | Status |
| --- | --- | --- | --- |
| PT-01 | SQL Injection — Kings Module Directory | Critical | Exploited |
| PT-02 | Webmin 1.984 / CVE-2022-0824 | Critical | Exploited |
| PT-03 | Weak Password Policy | High | Validated |
| PT-04 | SMB User Enumeration | Medium | Validated |
| PT-05 | Excessive Administrative Exposure | High | Contributed to compromise |
| PT-06 | Publicly Accessible Application Resources | Medium | Validated |
| PT-07 | Persistence Risk | High | Demonstrated in lab |

---

# Key Remediation Priorities

## Critical

- Patch or replace vulnerable Webmin installations
- Remediate SQL injection using parameterized queries
- Restrict administrative services to approved management networks

## High

- Strengthen password policy
- Introduce MFA for administrative interfaces
- Apply least privilege
- Reduce unnecessary administrative exposure
- Remove sensitive application-development resources from web roots

## Medium

- Restrict SMB enumeration
- Segment management services
- Introduce application-layer inspection
- Improve centralized logging

---

# Defensive Lessons

Several broader security lessons emerge from the assessment.

## 1. Small weaknesses can combine into a critical attack chain

```text
User enumeration
+
Weak passwords
+
Outdated admin software
=
Root compromise
```

## 2. Information disclosure matters

Exposed usernames, SQL structures, and application resources reduced the effort required for later exploitation.

## 3. Administrative interfaces require stronger protection

Management services should be protected through:

- MFA
- Network restrictions
- VPN access
- Strong authentication
- Patch management
- Monitoring

## 4. Local logs are not enough after full compromise

A root-level attacker may alter local evidence.

Defensive monitoring should therefore include:

```text
SIEM
EDR
Centralized Logging
Network Telemetry
File Integrity Monitoring
```

---

# Tools Used

| Tool | Purpose |
| --- | --- |
| Nmap | Host, port and service discovery |
| enum4linux | SMB and user enumeration |
| Burp Suite | HTTP inspection and request manipulation |
| sqlmap | SQL injection validation and database enumeration |
| Netcat | Reverse-shell communication |
| John the Ripper | Password-security testing |
| 7-Zip | Archive extraction |
| Exploit-DB PoC 50809 | CVE-2022-0824 validation |

---

# Repository Structure

```text
active-directory-penetration-testing/
├── README.md
├── LICENSE
│
├── docs/
│   ├── assessment-methodology.md
│   └── findings.md
│
└── evidence/
    ├── README.md
    └── screenshots/
        ├── 01-nmap-service-discovery.png
        ├── 02-smb-enumeration.png
        ├── 03-kings-module-directory.png
        ├── 04-sql-injection-validation.png
        ├── 05-database-enumeration.png
        ├── 06-webmin-version-and-login.png
        ├── 07-webmin-rce-root-shell.png
        ├── 08-root-access-validation.png
        └── 09-post-exploitation-persistence.png
```

---

# Documentation

| Resource | Description |
| --- | --- |
| [Assessment Methodology](docs/assessment-methodology.md) | Penetration-testing workflow and scope |
| [Findings & Remediation](docs/findings.md) | Detailed technical findings |
| [Evidence Walkthrough](evidence/README.md) | Evidence chain from reconnaissance to post-exploitation |

---

# Skills Demonstrated

This project demonstrates practical experience with:

- Penetration-testing methodology
- Network reconnaissance
- Port scanning
- Service enumeration
- SMB enumeration
- Account enumeration
- Password-policy assessment
- Web application testing
- Burp Suite
- SQL injection
- sqlmap
- Database enumeration
- Vulnerability validation
- CVE analysis
- Remote code execution
- Reverse shells
- Privilege validation
- Root compromise
- Post-exploitation
- Persistence concepts
- Security remediation
- Technical reporting

---

# Active Directory Scope Note

The repository name is retained for portfolio consistency, but the original assessment was **not a complete Active Directory domain compromise**.

The evidence supports:

```text
SMB enumeration
User enumeration
Password-policy analysis
Internal service assessment
Web application exploitation
Administrative-service exploitation
Root compromise
Post-exploitation
```

It does not support claims of:

```text
BloodHound / SharpHound
Kerberoasting
AS-REP Roasting
LDAP domain enumeration
Pass-the-Hash
Pass-the-Ticket
DCSync
Golden Ticket
NTDS.dit extraction
Domain Admin compromise
Domain Controller compromise
```

Those techniques should only be added if separately performed and evidenced in an authorized AD lab.

---

# Ethical & Authorization Statement

All testing represented in this repository was performed against an intentionally vulnerable virtual machine as part of an authorized academic cybersecurity assessment.

No external or production systems were targeted.

This repository is intended for:

- Defensive security education
- Cybersecurity portfolio demonstration
- Vulnerability-management learning
- Penetration-testing methodology review

It does not provide authorization to test third-party systems.

---

## Author

**Ravi Prajapati**

Cybersecurity | Enterprise IT | Network Security | Security Operations

[LinkedIn](https://www.linkedin.com/in/ravi-prajapati-it) · [GitHub](https://github.com/raviprajapati-it)
