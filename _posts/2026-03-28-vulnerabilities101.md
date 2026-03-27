---
title: Vulnerabilities 101
date: 2026-03-28 00:00:00 +0100
categories: [Penetration Testing]
tags: [penetration testing, vulnerability research, basics]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes on the relevant topic, but the credits for the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

---

> As a penetration tester, it is important to understand what vulnerabilities and **how to research and exploit them**. 
{: .prompt-info}

---

A vulnerability is a weakness or a flaw in the design, implementation or behaviour of a system. 

---

## **Main Categories of Vulnerabilities**
While there are thousands of unique vulnerabilities, they generally fall into several broad categories based on their nature:

* **Injection Flaws:** Occur when untrusted data is sent to an interpreter as part of a command or query (e.g., SQL Injection, Cross-Site Scripting or XSS, Command Injection). The attacker's hostile data tricks the interpreter into executing unintended commands.
* **Memory Management Errors:** Weaknesses in how a program handles memory allocation. Common in C/C++ applications (e.g., Buffer Overflows, Use-After-Free, Out-of-Bounds Write). These can often lead to arbitrary code execution.
* **Authentication & Authorization Failures:** Flaws that allow attackers to bypass login mechanisms, steal sessions, or elevate their privileges to access restricted data or administrative functions (e.g., Broken Access Control, Insecure Direct Object References).
* **Cryptographic Failures:** Incorrect use of encryption, relying on weak/outdated algorithms (like MD5 or SHA1), or improper key management, exposing sensitive data in transit or at rest.
* **Security Misconfigurations:** Insecure default settings, open cloud storage buckets, unpatched systems, or overly permissive firewall rules.
* **Logic Flaws:** Vulnerabilities where the code does exactly what it was programmed to do, but the underlying business logic is flawed, allowing for abuse (e.g., manipulating an e-commerce cart to change the price of an item).

---

## **Scoring Vulnerabilities: CVSS vs. VPR**
To prioritize patching, the industry uses scoring systems to quantify the severity of a vulnerability.


* **CVSS (Common Vulnerability Scoring System):** This is the industry-standard, open framework for rating the severity of a vulnerability on a scale from **0.0 to 10.0** (10 being the most critical). It calculates a score based on fixed metrics:
    * *Base Score:* Intrinsic qualities (How is it exploited? Does it require privileges? What is the impact on confidentiality, integrity, and availability?).
    * *Note:* The latest major version, CVSS v4.0 (released in late 2023), introduced better granular metrics for threat intelligence and environmental impact.
* **VPR (Vulnerability Priority Rating):** A proprietary dynamic scoring system developed by Tenable. While CVSS measures *static severity* (how bad the flaw is theoretically), VPR measures *dynamic risk* (how likely it is to be exploited *today*). It uses machine learning to analyze threat intelligence, exploit code availability, and dark web chatter to give a more **practical prioritization** score from 0.1 to 10.

> You would use VPR if you wanted to assess vulnerabilities based on the **risk it poses to an organization**. 
{: .prompt-info}

---

## **The Terminology: CVE vs. CWE**
These two acronyms are often confused, but they serve distinct purposes in vulnerability management.


* **CVE (Common Vulnerabilities and Exposures):** Think of this as the **specific diagnosis** for a specific patient. It is a unique identifier assigned to a *specific instance* of a vulnerability in a specific product. 
    * *Example:* CVE-2021-44228 is the specific identifier for the infamous "Log4Shell" vulnerability in the Apache Log4j library.
* **CWE (Common Weakness Enumeration):** Think of this as the **category of disease**. It is a community-developed dictionary of software hardware weakness types. 
    * *Example:* CWE-79 is the identifier for "Improper Neutralization of Input During Web Page Generation" (which is the root cause of Cross-Site Scripting). Log4Shell (the CVE) was categorized under CWE-502 (Deserialization of Untrusted Data).

---

## **Vulnerability Databases**
Once vulnerabilities are discovered and cataloged, they are stored in databases for researchers and defenders to analyze.

* **NVD (National Vulnerability Database):** Maintained by the U.S. government (NIST), this is the most comprehensive database. It synchronizes with the MITRE CVE list but adds immense value by providing CVSS scores, fix information, and CWE categorizations for almost every CVE.
* **Exploit-DB:** Maintained by Offensive Security, this is a highly popular database specifically for **Proof of Concept (PoC) exploit code**. While NVD tells you *about* the vulnerability, Exploit-DB often provides the actual scripts or methods an attacker might use to exploit it. It is heavily used by penetration testers and vulnerability researchers.

---

## **Additional Resources for Vulnerability Research**
If you are diving into vulnerability research or bug hunting, these resources are essential:

* **OWASP (Open Worldwide Application Security Project):** The absolute gold standard for web application security. Their "OWASP Top 10" list is required reading, and their testing guides provide deep technical methodologies for finding flaws.
* **MITRE ATT&CK Framework:** While CVEs focus on the vulnerabilities themselves, ATT&CK maps out the exact tactics and techniques adversaries use to exploit them during an active campaign.
* **PortSwigger Web Security Academy:** Created by the makers of Burp Suite, this is arguably the best free, interactive training platform for learning how web vulnerabilities work and how to exploit them.
* **Bug Bounty Platforms (HackerOne, Bugcrowd, Intigriti):** These platforms host public "hacktivity" or disclosed reports. Reading how top researchers string together minor bugs into massive critical exploits (often called "exploit chaining") is one of the best ways to learn modern vulnerability research.

---
