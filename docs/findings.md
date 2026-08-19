# Penetration Testing Findings

## Executive Summary

An authorized penetration test was conducted against the **KingsCollege virtual machine**, a deliberately vulnerable academic lab environment.

The assessment followed a structured workflow covering:

- Reconnaissance
- Service discovery
- SMB enumeration
- Web application enumeration
- Vulnerability assessment
- Controlled exploitation
- Password-security testing
- Privilege-impact validation
- Post-exploitation
- Remediation analysis

The assessment identified two primary critical attack paths:

1. **SQL Injection — Kings Module Directory**
2. **Webmin 1.984 — CVE-2022-0824 Remote Code Execution**

The combined weaknesses enabled unauthorized database access, disclosure of sensitive information, privileged system access, and ultimately **root-level compromise** of the laboratory target.

> **Scope:** All activity represented here was performed against an intentionally vulnerable virtual machine within an authorized academic environment.

---

# Findings Summary

| ID | Finding | Severity | Result |
| --- | --- | --- | --- |
| PT-01 | SQL Injection — Kings Module Directory | Critical | Exploited |
| PT-02 | Webmin 1.984 — CVE-2022-0824 | Critical | Exploited |
| PT-03 | Weak Password Policy | High | Validated |
| PT-04 | SMB User Enumeration | Medium | Validated |
| PT-05 | Excessive Administrative Exposure | High | Contributed to compromise |
| PT-06 | Publicly Accessible Application Resources | Medium | Validated |
| PT-07 | Post-Compromise Persistence Risk | High | Demonstrated in lab |

---

# Attack Surface Discovery

## Nmap Service Discovery

Initial reconnaissance identified several network services.

| Port | Service | Version / Context | Security Observation |
| ---: | --- | --- | --- |
| 21 | FTP | vsftpd 3.0.5 | Anonymous login disabled |
| 22 | SSH | Filtered | Direct SSH access unavailable |
| 80 | HTTP | Apache 2.4.58 / WordPress | Public web application |
| 139/445 | SMB | Samba 4 | User enumeration possible |
| 10000 | Webmin | 1.984 | Vulnerable administrative interface |
| 11235 | HTTP | Apache 2.4.58 | Kings Module Directory |

The reconnaissance stage therefore prioritized:

```text
SMB
Web applications
Webmin
```

for deeper investigation.

---

# PT-01 — SQL Injection

## Severity

**Critical**

## Affected Component

```text
Kings Module Directory
HTTP service — Port 11235
```

## Description

The Kings Module Directory contained a search function that accepted user-controlled input without sufficient sanitization.

Testing determined that the application was vulnerable to **SQL injection**, allowing attacker-controlled input to influence backend database queries.

This created a direct path from a public-facing application function to sensitive database information.

---

## Discovery

Web enumeration identified several interesting resources, including:

```text
/results.php
/index.php
/phpmyadmin/
/initial_sql_scripts/
```

The exposed SQL scripts were particularly useful because they disclosed information about the application's database structure.

This reduced the amount of uncertainty required to understand the backend application architecture.

---

## Validation

The suspected SQL injection vulnerability was investigated using:

```text
Burp Suite
sqlmap
```

Burp Suite supported inspection and manipulation of HTTP requests.

sqlmap was used in the authorized lab to validate the injection condition and investigate the accessible backend databases.

---

## Security Impact

Successful exploitation allowed unauthorized database access and extraction of information that should not have been available to an unauthenticated user.

The vulnerability demonstrated potential impact to:

```text
Confidentiality
     +
Integrity
     +
Authentication security
```

depending on the privileges of the application's database account.

---

## Attack Path

Conceptually:

```text
Attacker
   │
   ▼
Kings Module Directory
Port 11235
   │
   ▼
Unsanitized Input
   │
   ▼
SQL Injection
   │
   ▼
Backend Database
   │
   ▼
Unauthorized Data Access
```

---

## Root Cause

The primary issue was insecure handling of user-controlled application input.

Application data should never be directly incorporated into executable SQL statements without appropriate query parameterization.

---

## Remediation

Recommended controls include:

- Use parameterized queries / prepared statements
- Separate application data from SQL syntax
- Validate user-controlled input
- Apply least privilege to database accounts
- Remove unnecessary database-management exposure
- Restrict sensitive application directories
- Monitor abnormal database queries
- Consider application-layer traffic inspection
- Perform regular secure-code review and application security testing

---

# PT-02 — Webmin Remote Code Execution

## Severity

**Critical**

## Affected Component

```text
Webmin 1.984
TCP/10000
```

## Vulnerability

```text
CVE-2022-0824
```

## Description

Reconnaissance identified a Webmin administrative interface running version:

```text
1.984
```

Version analysis identified the installation as affected by **CVE-2022-0824**.

The vulnerability permits authenticated remote code execution through vulnerable Webmin functionality.

Because Webmin is an administrative service, successful exploitation can have significantly greater impact than compromise of a low-privilege application.

---

## Exploitation Context

The assessment used a public proof-of-concept associated with:

```text
Exploit-DB 50809
```

within the controlled lab environment.

Successful exploitation resulted in privileged command execution.

---

## Attack Path

The compromise path can be represented as:

```text
Reconnaissance
      │
      ▼
Webmin 1.984 Identified
      │
      ▼
Authentication Weakness
      │
      ▼
CVE-2022-0824
      │
      ▼
Remote Code Execution
      │
      ▼
Privileged Access
      │
      ▼
Root-Level Compromise
```

---

## Security Impact

Successful exploitation provided highly privileged access to the operating system.

This demonstrated potential impact including:

- Full host control
- Access to protected files
- Modification of system configuration
- Credential exposure
- Persistence
- Log manipulation
- Service manipulation
- Potential further compromise from the affected system

---

## Root Cause

Several weaknesses contributed to the attack path:

```text
Outdated administrative software
        +
Weak authentication controls
        +
Highly privileged management interface
        +
Insufficient administrative exposure restrictions
```

This is an example of how multiple individually manageable weaknesses can combine into a critical compromise path.

---

## Remediation

Recommended controls include:

- Upgrade Webmin to a supported patched release
- Establish formal patch-management procedures
- Restrict Webmin to trusted management networks
- Require VPN access for remote administration
- Enable MFA where supported
- Enforce strong password policy
- Implement account lockout controls
- Apply least privilege
- Monitor administrative authentication
- Monitor privileged configuration changes

---

# PT-03 — Weak Password Policy

## Severity

**High**

## Description

SMB enumeration exposed weaknesses in the system's password policy.

Observed policy weaknesses included:

```text
Password complexity: Disabled
Minimum password length: 5 characters
Account lockout: Not configured
```

These controls significantly reduce resistance to password guessing and cracking.

---

## Why This Matters

Weak password policy becomes substantially more dangerous when account names can also be enumerated.

Conceptually:

```text
User Enumeration
       +
Weak Password Policy
       +
No Account Lockout
       │
       ▼
Increased Authentication Attack Surface
```

The assessment found that weak authentication controls contributed to later access to the Webmin administrative interface.

---

## Password Analysis

Password-security testing included:

```text
John the Ripper
```

within the authorized lab.

The objective was to demonstrate the practical consequences of weak password selection and inadequate password-policy enforcement.

---

## Remediation

Recommended controls include:

- Increase minimum password length
- Require appropriate password complexity
- Block commonly used passwords
- Implement account lockout or rate limiting
- Require MFA for administrative accounts
- Monitor repeated authentication failures
- Use unique administrative credentials
- Review dormant and unnecessary accounts

---

# PT-04 — SMB User Enumeration

## Severity

**Medium**

## Affected Service

```text
SMB
TCP/139
TCP/445
Samba 4
```

## Description

SMB enumeration was performed using:

```text
enum4linux
```

The service disclosed local user-account information.

Six accounts were identified during the assessment.

For portfolio purposes, the specific usernames are not necessary to reproduce because the security issue is the ability to enumerate account identities, not the identities themselves.

---

## Security Impact

Account enumeration gives an attacker useful information for later authentication attacks.

Instead of guessing both:

```text
Username
+
Password
```

the attacker can focus only on obtaining or guessing the password for a known valid account.

---

## Attack Relationship

In this assessment:

```text
SMB Enumeration
      │
      ▼
Valid Accounts Identified
      │
      ▼
Weak Password Policy Identified
      │
      ▼
Authentication Attack Surface Increased
```

This demonstrates why seemingly low-impact information disclosure can become important when combined with other weaknesses.

---

## Remediation

Recommended controls include:

- Restrict SMB exposure
- Disable unnecessary SMB services
- Prevent unnecessary anonymous enumeration
- Limit SMB access to trusted networks
- Segment administrative services
- Enforce strong authentication
- Monitor SMB authentication activity
- Apply host firewall rules

---

# PT-05 — Administrative Interface Exposure

## Severity

**High**

## Description

The Webmin administrative interface was reachable through:

```text
TCP/10000
```

Administrative interfaces represent high-value attack surfaces because successful authentication or exploitation may provide extensive system control.

The risk was increased because the exposed Webmin version was outdated and vulnerable.

---

## Security Principle

Administrative services should generally follow:

```text
Internet / User Network
        │
        X
        │
Management Network
        │
        ▼
Administrative Interface
```

rather than being unnecessarily reachable from broad network segments.

---

## Remediation

Recommended controls include:

- Place administrative interfaces on dedicated management networks
- Restrict access using firewall rules
- Require VPN connectivity
- Use MFA
- Patch administrative software
- Centralize administrative authentication
- Log privileged access
- Alert on abnormal management activity

---

# PT-06 — Exposed Application Resources

## Severity

**Medium**

## Description

Directory enumeration identified publicly reachable resources associated with the Kings Module Directory.

Examples included:

```text
/results.php
/index.php
/phpmyadmin/
/initial_sql_scripts/
```

The exposed SQL-script directory provided information about the application's backend database structure.

---

## Security Impact

Information disclosure can reduce the effort required to attack an application.

For example:

```text
Exposed SQL Structure
        │
        ▼
Database Understanding
        │
        ▼
More Efficient SQL Injection Analysis
```

Information exposure therefore contributed to the wider attack path.

---

## Remediation

Recommended controls include:

- Remove development artefacts from production web roots
- Restrict database-management interfaces
- Disable directory listing
- Apply access controls to administrative resources
- Separate development and production environments
- Review web-root contents before deployment
- Perform periodic content-discovery testing

---

# PT-07 — Persistence Risk

## Severity

**High**

## Description

Following root-level compromise, the assessment demonstrated how an attacker could establish persistence.

A systemd service was used in the controlled lab to illustrate persistence across service/system restarts.

The purpose was to demonstrate post-compromise impact rather than to create long-term unauthorized access.

---

## Attack Path

```text
Root Compromise
      │
      ▼
Privileged System Modification
      │
      ▼
Persistence Mechanism
      │
      ▼
Access Survives Restart
```

---

## Defensive Significance

Obtaining initial access is only one part of an intrusion.

Defenders must also monitor for changes that indicate persistence, including:

- New services
- Modified startup configuration
- Scheduled tasks
- New privileged accounts
- Modified SSH configuration
- Unexpected binaries
- Unusual startup scripts

---

## Remediation

Recommended controls include:

- Monitor service creation
- Alert on privileged configuration changes
- Use EDR
- Baseline system services
- Centralize logs
- Audit startup mechanisms
- Restrict root-level access
- Perform file-integrity monitoring

---

# Post-Exploitation Impact

The assessment demonstrated that root-level compromise creates opportunities for extensive system manipulation.

Post-exploitation activities were limited to demonstrating security impact within the lab.

Areas examined included:

```text
Filesystem access
Privilege verification
Persistence
Log manipulation
Command history
Temporary exploitation artefacts
```

---

# Anti-Forensic Demonstration

Basic anti-forensic activity was demonstrated after compromise, including manipulation or clearing of local evidence sources.

This was performed to illustrate an important defensive lesson:

> Local logs alone cannot be relied upon after a host has been fully compromised.

An attacker with root privileges may be capable of altering local evidence.

---

# Defensive Response to Anti-Forensics

Organizations should therefore use controls such as:

- Centralized logging
- Remote SIEM ingestion
- EDR telemetry
- Immutable or protected logs
- Network telemetry
- Authentication monitoring
- File-integrity monitoring

This allows security teams to retain evidence even if local logs are manipulated.

---

# Combined Attack Chain

The overall assessment can be summarized as:

```text
                    Nmap
                      │
                      ▼
              Service Discovery
                      │
          ┌───────────┴───────────┐
          │                       │
          ▼                       ▼
        SMB                 Web Applications
          │                       │
          ▼                       ▼
 User Enumeration        Directory Enumeration
          │                       │
          ▼                       ▼
 Weak Passwords            SQL Injection
          │                       │
          │                       ▼
          │                Database Access
          │
          ▼
      Webmin Login
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
     Root Access
          │
          ▼
   Post-Exploitation
          │
     ┌────┴────┐
     ▼         ▼
Persistence  Impact Analysis
```

This is the most important lesson from the assessment:

> Serious compromise often results from a chain of weaknesses rather than one isolated vulnerability.

---

# Remediation Priorities

## Critical Priority

1. Patch or replace vulnerable Webmin installations.
2. Remediate SQL injection using parameterized database queries.
3. Restrict administrative interfaces to approved management networks.

## High Priority

4. Strengthen password policies.
5. Implement MFA for administrative access.
6. Apply least privilege to administrative and database accounts.
7. Remove unnecessary publicly accessible application resources.

## Medium Priority

8. Restrict SMB exposure and enumeration.
9. Introduce application-layer monitoring.
10. Segment management services from user networks.

## Monitoring & Detection

11. Centralize system and authentication logs.
12. Deploy endpoint detection and response.
13. Monitor privileged service creation.
14. Alert on repeated authentication failures.
15. Conduct recurring vulnerability assessments.

---

# Assessment Outcome

The penetration test successfully demonstrated how weaknesses across several security layers could be combined.

The assessment progressed from:

```text
Discovery
   ↓
Enumeration
   ↓
Weakness Identification
   ↓
Controlled Exploitation
   ↓
Privileged Access
   ↓
Post-Exploitation
   ↓
Remediation Analysis
```

The strongest technical findings were:

- SQL injection
- Outdated vulnerable Webmin
- Remote code execution
- Root-level compromise
- Weak password policy
- SMB account enumeration
- Excessive management exposure
- Persistence risk

---

# Important Active Directory Scope Note

Despite the repository name, this assessment should **not** be represented as a complete Active Directory compromise.

The preserved coursework supports:

```text
SMB enumeration
User enumeration
Password-security assessment
Internal service enumeration
Web exploitation
Administrative-service exploitation
Privilege escalation / root compromise
```

It does **not** provide evidence for claiming techniques such as:

```text
Kerberoasting
AS-REP Roasting
BloodHound / SharpHound
LDAP domain enumeration
DCSync
Pass-the-Hash
Pass-the-Ticket
Golden Ticket
NTDS.dit extraction
Domain Admin compromise
Domain Controller compromise
```

Those techniques should only be added to the repository if separately performed and evidenced in an authorized Active Directory lab.

---

## Related Documentation

- [Assessment Methodology](assessment-methodology.md)
