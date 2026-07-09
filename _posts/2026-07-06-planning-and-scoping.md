---
title: Planning and Scoping in Penetration Testing
date: 2026-07-06 00:00:00 +0200
categories: [Penetration Testing]
tags: [penetration testing, planning, scoping, rules of engagement]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

## Objective

> Discover planning and scoping in penetration testing.

Securing a company or organization is not a one-time event. It is a continuous process of building defenses and **testing** them. 

The challenge is that testing the security of computer systems is extremely complex. The attack surface can span web applications, internal networks, cloud infrastructure, APIs and mobile applications. Each of these requires specialized skills. 

For this reason, organizations rely on **penetration testers**, **authorized** individuals who simulate real cyber attacks to find vulnerabilities and weaknesses in the systems **before** malicious actors do. 

---

## What is a penetration test? 

A penetration test is a comprehensive security assessment in which authorized professionals simulate real cyber attacks against an organization's systems, networks, or applications. The goal is to discover security vulnerabilities and demonstrate their potential impact before a malicious actor can exploit them. Two words in that definition deserve special attention: **authorized** and **simulate**. The testers have explicit permission to attack, and they do so in a controlled manner designed to evaluate risk without causing lasting harm.

The result of a penetration test is a report with **actionable recommendations** for improving the organizations' security posture. 

A penetration test follows a structured process. Often, split into seven phases: 

1. Pre-engagement: Define scope, rules and legal agreements. 
    - Proper legal agreements protect both parties from activities that would otherwise be illegal. 
2. Intelligence Gathering
3. Threat Modeling: Identifying most likely attack scenarios based on gathered intelligence. 
4. Vulnerability Analysis: Systematically discovering weaknesses in the target systems. 
5. Exploitation: Gain unauthorized access via the vulnerabilities. 
6. Post Exploitation: Determine value of compromised systems and assets. 
7. Reporting

> Notice how most of the phases are planning, analysis and reporting. This is often 80-90% of the work. The "hacking" is just a small aspect, and how well it goes, depends on how well the analysis and planning went. 
{: .prompt-info}


> A penetration test and vulnerability assessment are complementary, but distinct. A penetration test involves **active exploitation** of the vulnerabilities. 
{: .prompt-info}

---

## Types and Approaches

- Network-based
- Web application-based
- Wireless network-based
- Mobile application-baed
- Physical

Approaches are of three categories: 

1. Known environment (White-box): The tester is given comprehensive details about the environment. Maximizes vulnerability discovery. 
2. Partially known environment (Gray-box): The tester receives partial knowledge. Balances realism with efficiency. 
3. Unknown environment (Black-box): Tester is given minimal information needed to define the scope. Simulates external attacker with no insider knowledge. 

> A penetration test has a defined scope, and typically lasts for a few weeks, while a **red team engagement** is organization-wide and lasts for weeks to months. Critically, they operate in stealth. 
{: .prompt-info}

---

## Defining scope

The scope of a penetration test defines the boundaries of the engagement. It answers two fundamental questions: what systems, networks, and applications are you authorized to test, and what is explicitly off limits?

The tester needs to know which assets are on the **allowlist** (in-scope targets you are authorized to test) and which are on the **blocklist** (out-of-scope assets you must not touch).

A well-defined scope covers the following: 

- Target systems and networks: IP ranges, domain and subdomain lists, network subnets, etc. 
- Applications and services: API endpoints, URLs, mobile applications. 
- Wireless networks: SSIDs, access point locations, wireless protocols to be assessed. 
- Cloud infrastructure: Cloud accounts, regions, IAM configurations, provider information with respect to AWS, Azure or GCP. 
- User accounts and roles: Test accounts for testing, including the privilege levels for each account. 

The scope also specifies **out-of-scope** assets, that are not meant to be touched. 

> Scope can be too broad or too narrow and result in lower quality. Defining a balanced scope is a collaborative process between the penetration testing team and the client. 
{: .prompt-warning}

> It is also important to ensure that the scope does not change during the engagement, without formal changes that are known to the client and the team. 
{: .prompt-warning}

---

## Core Legal Documents 

1. NDA: Non-Disclosure Agreement. Ensures both parties protect confidential information throughout the engagement. 
2. MSA: Master Service Agreement. Overarching legal relationship between client and testing firm. 
3. SOW: Statement of Work. **Engagement-specific** document. Specifies what will be tested and how. 
4. Authorization Letter: **Get-out-of-jail** letter. Signed by someone of high authority in the client's firm.

---

## Rules of Engagement

In a penetration test, the **Rules of Engagement (RoE)** are the formal agreements that define the operational boundaries of the engagement: what can be tested, when testing can occur, how the parties communicate, and what happens when something goes wrong.

> Sometimes, critical findings during the engagement need to be informed immediately. This is where **Escalation Procedures** come into play. They fall into three categories: Informational, Urgent and Critical. 
{: .prompt-info}

### Sample RoE

| RoE Component | Detail |
| ---- | ---- |
| Engagement window | 	March 15, 2026 to March 28, 2026| 
| Testing hours	| Exploitation: 10:00 PM - 1:30 AM weekdays, unrestricted weekends. Passive recon: unrestricted. | 
| Primary contact (client)	| CISO, BrightCart, encrypted email | 
| Emergency contact (client)	| VP of Engineering, mobile phone | 
| Status reporting	| Daily email summary by 9:00 AM to primary contact | 
| Critical finding escalation	| Immediate phone call to emergency contact | 
| Permitted activities	| Port scanning, vulnerability scanning, exploitation, privilege escalation, lateral movement within scope | 
| Prohibited activities	| DoS/DDoS, social engineering, physical testing, data exfiltration beyond proof-of-concept | 
| Data handling	| Evidence encrypted with AES-256; retained for 30 days post-report; secure deletion confirmed in writing | 
| Final report due	| 10 business days after testing concludes | 

---
