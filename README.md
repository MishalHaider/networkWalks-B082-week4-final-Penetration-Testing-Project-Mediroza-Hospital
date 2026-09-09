# networkWalks-B082-week4-final-Penetration-Testing-Project-Mediroza-Hospital
Final Penetration Testing Project-Mediroza Hospital


# NETWORKWALKS | CONFIDENTIAL TRAINING ENGAGEMENT
# Mediroza General Hospital — Black-Box Penetration Test Report

| | |
|---|---|
| **Program** | Networkwalks Penetration Testing Training Program |
| **Batch** | B082 — Week 4 |
| **Prepared By** | Mishal |
| **Client** | Mediroza General Hospital |
| **Target** | https://medirozahospital.com |
| **Engagement Type** | Full Black-Box Penetration Test |
| **Duration** | 3 Days |
| **Status** | Final — All Milestones Completed |

> **Disclaimer:** This assessment was carried out in a controlled training environment against a system for which explicit written authorisation was granted. None of the techniques documented here were, or should be, applied to any system without equivalent written permission from the owner. This repository is shared for educational and portfolio purposes only. Exact payloads, credentials, and step-by-step exploitation details have intentionally been left out, since this exercise may still be worked through by other trainees.

---

## 1. Overview

This started as a black-box assignment with a login page and a "good luck." No credentials, no source code, no internal docs — just a target URL and three days on the clock.

What began as a simple recon exercise (`whois`, `nslookup`, `theHarvester`) turned into a full compromise chain: a login form that trusted user input a little too much, a portal that handed out other people's medical records without checking who was asking, PDF passwords weak enough to fall to a common-password list in minutes, and a forgotten backup folder sitting wide open on the same server — quietly holding the hospital's entire payroll and shareholder register.

None of it needed anything exotic. Every step used standard, well-known techniques. That's the part worth sitting with: this wasn't a zero-day. It was a chain of small, ordinary oversights that added up to a critical, real-world breach of patient and corporate data.

## 2. Scope and Rules of Engagement

| Item | Detail |
|---|---|
| Target | https://medirozahospital.com |
| Engagement type | Full black-box test, no credentials or internal documentation provided |
| Rules | Testing limited to the target domain only. No social engineering. No denial-of-service testing. No testing outside the agreed scope. |
| Authorisation | Written authorisation provided prior to testing |

## 3. Milestones

| # | Milestone | Objective | Status |
|---|---|---|---|
| M1 | Initial Access | Gain access to the patient portal and retrieve confidential lab reports | ✅ Complete |
| M2 | Data Extraction | Recover the contents of the protected documents obtained in M1 | ✅ Complete |
| M3 | Critical Exposure | Identify further server-side data exposure affecting staff and ownership records | ✅ Complete |
| M4 | Reporting | Produce a professional penetration testing report for the client | ✅ Complete |

## 4. Methodology and Tools

The assessment followed a standard black-box methodology: reconnaissance, manual testing of application logic, exploitation, and post-exploitation analysis of everything recovered along the way.

- `whois`, `nslookup`, `theHarvester` — passive and semi-passive reconnaissance
- `Gobuster` — directory and file enumeration, including legacy/backup paths
- `Burp Suite` (Repeater / Intruder) — manual request tampering and authentication testing
- Manual SQL injection testing — input validation and query-logic analysis
- `qpdf` — encryption inspection and offline PDF password recovery
- PowerShell scripting — automated wordlist-based password testing against recovered documents
- Curated wordlists, including context-derived candidates built from data found *during* the engagement itself

## 5. Findings Summary

| ID | Finding | Risk |
|---|---|---|
| F1 | Authentication bypass in the patient portal login (SQL injection) | Critical |
| F2 | Broken access control — cross-patient data exposure | High |
| F3 | Weak protection on distributed patient documents | High |
| F4 | Unauthenticated directory listing exposing a legacy backup | Critical |
| F5 | Disclosure of confidential HR records | Critical |
| F6 | Disclosure of confidential shareholder / ownership data | Critical |

### F1: Authentication Bypass, Patient Portal (Critical)
The login form didn't validate input before dropping it straight into a backend query. Confirmed with a raw SQL syntax error reflected back in the response, then bypassed outright — a crafted username value returned a 302 redirect and a live, authenticated session with no real password ever supplied. A WAF was in play and blocked the obvious payloads; it just wasn't blocking all of them.

*Figure 1: Authenticated session obtained via authentication bypass. Submitted value redacted.*

### F2: Broken Access Control, Cross-Patient Data Exposure (High)
Once inside, the portal didn't ask "whose records am I allowed to show this session?" — it just showed everyone's. Multiple unrelated patients' lab reports, listed side by side, no ownership check in sight.

*Figure 2: Portal returning reports for multiple unrelated patients. Identifying details redacted.*

### F3: Weak Protection on Distributed Patient Documents (High)
The reports themselves were password-protected — a reasonable idea, undone by weak execution. All three cracked offline in minutes using nothing more exotic than common-password lists and a short PowerShell loop around `qpdf`.

*Figure 3: Offline password recovery in progress. Recovered values withheld here.*

### F4: Directory Listing Enabled, Exposed Database Backup (Critical)
Away from the login form entirely, a legacy `/old/` directory turned out to be browsable, and inside it sat a full, unauthenticated database backup — no login, no bypass, no cleverness required. Just a forgotten folder nobody cleaned up.

*Figure 4: Unauthenticated directory listing exposing a legacy database backup. Filename redacted.*

### F5 & F6: Confidential HR & Shareholder Data Exposure (Critical)
That backup wasn't a small find. It contained a complete staff table — names, national ID numbers, salaries, contact details, for the whole workforce — and a full shareholder register with ownership percentages and share classes. This is the most severe finding of the engagement, and it existed completely independently of everything else that was found.

## 6. Attack Chain (High-Level)

```
Patient login form
        |  Authentication logic flaw (SQL injection)
        v
Login bypassed -> access to patient portal
        |  No server-side authorisation check
        v
Multiple patients' confidential documents retrieved
        |  Weak document protection defeated offline
        v
Document contents recovered

Independently:
Legacy /old/ directory -> directory listing enabled
        |  Unauthenticated download
        v
Internal backup file retrieved
        |
        v
Staff records and shareholder register exposed
```

## 7. Recommendations

- Use parameterised queries for all database interactions; never build queries by concatenating user input.
- Enforce server-side authorisation on every record request, scoped to the authenticated user's own identity.
- Apply strong, unique, randomly generated protection to any document containing personal or medical information.
- Disable directory listing on the webserver and ensure backup files are never stored inside the public web root.
- Review breach notification obligations given the exposure of national identification numbers and health-related information.

Full detail, business impact analysis, and remediation guidance for every finding is available in the complete client report.

## 8. Repository Contents

Raw recovered documents, the database backup, and unredacted staff or shareholder data are withheld from this public repository and were provided to the client and instructor separately, in line with responsible handling of sensitive data.

## 9. Skills Demonstrated

`Web Application Security` `Authentication Testing` `Access Control Testing`
`Offline Password & Hash Recovery` `Server Enumeration` `PowerShell Scripting`
`Professional Security Reporting` `Risk Assessment` `Technical Documentation`

---

**Prepared by Mishal**
Networkwalks Penetration Testing Training Program — Batch B082, Week 4

Target: `https://medirozahospital.com`
This project was completed as part of the Networkwalks penetration testing training program.
