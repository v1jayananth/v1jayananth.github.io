---
title: Understanding Vulnerability Databases
date: 2026-04-28 00:00:00 +0100
categories: [Vulnerabilities]
tags: [vulnerability research, fundamentals]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

Vulnerability Databases provide a **standardized**, **centralized**, source of information regarding vulnerabilities. This helps security professionals and teams by:

- Avoiding time-consuming duplicate research by referencing known issues.
- Awareness of possible patches and remediations. 
- Consistent naming across tools and teams. 
- Provides **reliable** reference data.

With proper standardization, these databases enable efficient vulnerability management, risk assessment, and remediation planning. 

---

## Key Terms

1. CVE (Common Vulnerabilities and Exposures): This is the **Vulnerability Identifier**, which allows **unique identification** across tools, teams and organizations. 
    - Each ID starts with `CVE-` followed by the year (when it was assigned or made public), followed by a number (sequence number, can be arbitrarily long). Example: `CVE-2021-44228`, which is the log4shell vulnerability. 
2. CVSS (Common Vulnerability Scoring System): This indicates the **Severity**, how critical the vulnerability is. 
    - Score ranges from `1-10`, 10 indicating that it is extremely easy to exploit and highly impactful when exploited. 
3. CPE (Common Platform Enumeration): Identifies **Affected Software**. Identifies the product and the specific version to narrow down to the right specifics. 
4. CWE (Common Weakness Enumeration): Grouping vulnerabilities by their **root cause**. For example, `CWE-89` is SQL Injection. Helps understand the vulnerability landscape for different types of vulnerabilities, giving more insight for security professionals. 
5. CVE Numbering Authorities (CNA): Organizations that are authorized to assign CVEs. CNAs help scaling CVE program by allowing vendors and organizations to research and report. 

> Other metadata like descriptions, remediation information are also included as reference. 
{: .prompt-info}

> Severity vs Risk: Severity describes **technical impact**. Risk considers how that vulnerability affects a **specific environment**. 
{: .prompt-info}

## CVE List

List of CVEs, maintained by MITRE. So that known security vulnerabilities are known to the public in a easy to access format. 

Official Website: [CVE List](https://www.cve.org)

---

## National Vulnerability Database (NVD)

It builds upon the CVE list by **enriching** CVE entries, with additional analysis.

[NVD Search](https://nvd.nist.gov/vuln/search)

### NIST NVD Regarding Lower Severity Vulnerabilities

NIST NVD has announced, starting April 15, that it will only analyze and provide additional details (enrich) them, for vulnerabilities that affect the U.S. federal government, software that is deemed critical, or part of CISA's Known Exploited Vulnerabilities (KEV) Catalog. 

The reason for this decision is due to the large number of submissions, which the organization just cannot handle anymore. 

It will still add CVEs to the list, provided by CNAs, but **it won't provide an additional severity score for them**. 

[Bleeping Computer Post](https://www.bleepingcomputer.com/news/security/nist-to-stop-rating-non-priority-flaws-due-to-volume-increase/)

---
