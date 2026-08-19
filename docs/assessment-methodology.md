# Penetration Testing Assessment Methodology

## Overview

This project documents an authorized penetration-testing assessment performed in a controlled academic laboratory environment.

The assessment followed a structured workflow designed to identify exposed services, enumerate accessible resources, investigate vulnerabilities, validate selected findings through controlled exploitation, assess post-exploitation impact, and develop remediation recommendations.

> **Scope note:** Although this portfolio repository is titled `active-directory-penetration-testing`, the original assessment was broader than a dedicated Active Directory engagement. The preserved evidence primarily supports internal-network, SMB, web-application, service-enumeration, exploitation, credential-security, and post-exploitation testing. Unsupported Active Directory techniques are not presented as completed work.

---

# Assessment Workflow

The investigation followed the general sequence:

```text
Reconnaissance
      │
      ▼
Host & Service Discovery
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
Post-Exploitation Analysis
      │
      ▼
Risk Assessment
      │
      ▼
Remediation
```

---

# 1. Reconnaissance

The first stage focused on understanding the available attack surface.

Objectives included:

- Identifying reachable hosts
- Discovering exposed ports
- Determining running services
- Identifying service versions
- Establishing potential enumeration targets
- Prioritising services for deeper investigation

Nmap was used as a primary reconnaissance and service-discovery tool.

The purpose of this stage was not to exploit systems immediately, but to establish an evidence-based view of the environment before selecting further testing activities.

---

# 2. Service Enumeration

Following initial discovery, exposed services were investigated in greater detail.

Enumeration focused on obtaining information that could reveal:

- User accounts
- Network shares
- Service configuration
- Authentication exposure
- Web applications
- Administrative interfaces
- Potentially vulnerable software

The assessment included SMB-oriented enumeration using tools such as:

```text
enum4linux
```

This helped investigate information exposed through Windows/SMB-style network services.

---

# 3. SMB & Account Enumeration

SMB enumeration formed an important part of the internal-network assessment.

The objective was to determine whether network services exposed useful information about:

- Users
- Shares
- Authentication
- Host configuration
- Account naming
- Access permissions

Information discovered during enumeration was used to support later security analysis.

This stage demonstrates why excessive unauthenticated or weakly restricted service information can increase an organisation's attack surface.

---

# 4. Web Application Assessment

Web-facing services discovered during reconnaissance were assessed separately.

Testing included investigation of:

- Application inputs
- Parameters
- Authentication surfaces
- Potential injection vulnerabilities
- Administrative interfaces

Tools used in the assessment included:

```text
Burp Suite
sqlmap
```

Burp Suite supported inspection of web requests and application behaviour.

sqlmap was used where appropriate to investigate suspected SQL injection vulnerabilities in the controlled lab environment.

---

# 5. Vulnerability Assessment

Information collected during reconnaissance and enumeration was correlated with potential security weaknesses.

The objective was to distinguish between:

```text
Open service
     │
     ▼
Potential weakness
     │
     ▼
Validated vulnerability
```

rather than assuming that every exposed port represented an exploitable vulnerability.

Assessment areas included:

- Service exposure
- Software/version weaknesses
- Authentication weaknesses
- Web application vulnerabilities
- SMB information exposure
- Administrative interfaces
- Password security

---

# 6. Controlled Exploitation

Selected vulnerabilities were validated through controlled exploitation within the authorized laboratory environment.

The original assessment includes exploitation involving:

- SQL injection
- Vulnerable Webmin functionality
- Remote command/code execution
- Shell access
- Privilege-impact validation

The purpose of exploitation was to demonstrate security impact rather than simply report theoretical vulnerability information.

---

# 7. Shell Access

Netcat was among the tools used during exploitation and post-exploitation activities.

Shell access provides stronger evidence of vulnerability impact than vulnerability identification alone because it demonstrates that an attacker may be able to interact with the target operating system.

The assessment considered the difference between:

```text
Vulnerability identified
```

and:

```text
Vulnerability successfully used to obtain system access
```

---

# 8. Post-Exploitation

After obtaining access, the assessment moved into controlled post-exploitation analysis.

Objectives included determining:

- Current privilege level
- Accessible system resources
- Potential credential exposure
- Security impact of the compromise
- Whether higher privileges could be obtained

The assessment documented compromise reaching elevated/root-level access in the affected lab system.

Post-exploitation was limited to activities necessary to demonstrate the impact of the identified weaknesses.

---

# 9. Password & Credential Security

Password security was also assessed.

The original work used:

```text
John the Ripper
```

for password-related analysis.

The objective was to demonstrate the risk created by weak credentials and insufficient password controls.

Weak passwords can turn information obtained during enumeration into a practical compromise path.

---

# 10. Evidence-Based Findings

The assessment distinguishes between three levels of evidence.

## Observed

Information directly identified during reconnaissance or enumeration.

Examples:

- Open ports
- Services
- User information
- Network shares
- Web applications

## Validated

Security weaknesses confirmed through additional testing.

Examples:

- SQL injection
- Weak authentication controls
- Vulnerable services

## Exploited

Weaknesses successfully used within the authorized lab to demonstrate impact.

Examples include:

- Remote execution
- Shell access
- Elevated/root access

This distinction prevents theoretical vulnerabilities from being represented as successful compromises.

---

# 11. Risk Assessment

Findings were evaluated based on the potential consequences of exploitation.

Relevant impact areas include:

- Unauthorized access
- Credential compromise
- Data exposure
- Remote command execution
- Privilege escalation
- Administrative compromise
- Loss of confidentiality
- Loss of integrity
- Loss of system control

The technical severity of a vulnerability should therefore be considered alongside the access and business impact it enables.

---

# 12. Remediation Approach

The final stage translates technical findings into defensive recommendations.

Remediation themes include:

- Patch vulnerable services
- Remove unnecessary exposed services
- Restrict SMB access
- Apply least privilege
- Strengthen password policy
- Harden administrative interfaces
- Validate web application inputs
- Use parameterized database queries
- Restrict management access
- Monitor authentication activity
- Segment sensitive systems
- Maintain vulnerability-management processes

The objective is not only to demonstrate how compromise occurred, but also how the underlying weaknesses can be reduced or removed.

---

# Tools Used

| Tool | Assessment Role |
| --- | --- |
| Nmap | Host, port and service discovery |
| enum4linux | SMB and account enumeration |
| Burp Suite | Web request and application analysis |
| sqlmap | SQL injection assessment |
| Netcat | Network interaction and shell-related testing |
| John the Ripper | Password-security analysis |

Additional utilities may appear in the original evidence where relevant.

---

# Scope Boundaries

This repository does **not** claim completion of Active Directory attack techniques unless supported by preserved evidence.

For example, the project does not automatically claim:

- BloodHound/SharpHound enumeration
- Kerberoasting
- AS-REP roasting
- DCSync
- Golden Ticket attacks
- Pass-the-Hash
- Pass-the-Ticket
- NTDS extraction
- Domain Admin compromise
- Domain Controller compromise

Those are valid Active Directory penetration-testing techniques, but they should not be added to a portfolio simply because they are commonly associated with AD assessments.

The repository instead focuses on the techniques demonstrated by the original assessment.

---

# Ethical & Authorization Statement

All penetration-testing activity represented in this repository was performed as part of an authorized academic cybersecurity exercise in a controlled laboratory environment.

The material is provided for:

- Cybersecurity education
- Defensive security learning
- Technical portfolio demonstration
- Vulnerability-management understanding

No testing documented here should be interpreted as authorization to perform equivalent activity against systems without explicit permission.

---

# Key Skills Demonstrated

The assessment demonstrates practical experience with:

- Penetration-testing methodology
- Network reconnaissance
- Port scanning
- Service enumeration
- SMB enumeration
- User/account enumeration
- Vulnerability assessment
- Web application testing
- SQL injection assessment
- Exploitation validation
- Shell access
- Post-exploitation analysis
- Password-security testing
- Privilege-impact analysis
- Risk assessment
- Security remediation
- Technical reporting
