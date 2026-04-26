---
title: Introduction to Threat Emulation
date: 2026-04-26 00:00:00 +0100
categories: [Threat Emulation]
tags: [threat emulation, tryhackme, mitre att&ck]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

The challenge of assessments such as vulnerability assessments, penetration testing and even red teaming, is that, they do not represent real-world threats. They commonly identify **initial** access vectors that attackers **may** use. 

Another challenge is that, in these assessments, red and blue teams are not incentivized to share their intel and TTPs. 

**Threat Emulation** addresses these challenges and provides a holistic security evaluation. 

---

Mainly, threat emulation is an **intelligence-driven** impersonation of real-world attack scenarios and TTPs (Tactics, Techniques and Procedures) in a **controlled environment** to test, assess and improve an organizations' defense. The idea is to identify and mitigate security gaps before attackers exploit them. 

If there is red team engagement and it is unannounced to the rest of the defensive blue team, then this is also threat emulation. More specifically, a **blind** operation. It could also be **non-blind** where teams are aware of each other and can share knowledge, allowing for more planning and testing. 

---

## Threat Emulation vs. Red Teaming

These two concepts are related and often confused, but they serve distinct purposes in cybersecurity.

---

### Threat Emulation

Threat emulation (also called adversary emulation) is the practice of **replicating the specific tactics, techniques, and procedures (TTPs) of known threat actors** — usually based on threat intelligence. The goal is to test whether your defenses can detect and respond to a *particular* adversary.

#### Key characteristics:

- **Intelligence-driven** — modeled after real threat groups (e.g., APT29, Lazarus Group)
- **Structured and repeatable** — follows documented playbooks (often using frameworks like MITRE ATT&CK)
- **Narrow scope** — focuses on emulating specific known behaviors, not finding all possible weaknesses
- **Detection-focused** — often designed to validate that your SOC/SIEM/EDR actually catches what it should
- **Collaborative** — the security team may be aware it's happening (purple team exercises often use emulation)

---

### Red Teaming

Red teaming is a **broader, goal-oriented adversarial assessment** where a team of skilled attackers tries to achieve a specific objective (e.g., access sensitive data, compromise a domain controller) using *any means necessary* — not constrained to a specific threat actor's playbook.

#### Key characteristics:

- **Objective-driven** — the red team is given a goal, not a script
- **Creative and unconstrained** — attackers use novel techniques, social engineering, physical access, etc.
- **Broad scope** — targets people, processes, and technology holistically
- **Stealth-focused** — often fully covert; the blue team doesn't know it's happening
- **Finds unknown gaps** — surfaces vulnerabilities that aren't on any known threat actor's radar

---

### Side-by-Side Comparison

| | **Threat Emulation** | **Red Teaming** |
|---|---|---|
| **Basis** | Known threat actor TTPs | Attacker's creativity + objectives |
| **Scope** | Narrow, specific | Broad, holistic |
| **Blue team awareness** | Often aware (purple team) | Usually unaware |
| **Framework** | Heavily structured | Loosely structured |
| **Primary goal** | Validate detection/response | Find any path to the objective |
| **Output** | Detection coverage gaps | Exploitable attack paths |

---

### How They Relate

In practice, a red team **may use threat emulation as one technique within a broader engagement** — for example, emulating a ransomware group's initial access methods while also trying other creative approaches. Threat emulation is a *tool*; red teaming is an *exercise*.

> Threat simulation is the exercise of representing adversary functions through predefined and **automated** attack patterns. 
{: .prompt-info}

---

## Key Concepts 

![Pyramid of Pain]({{ "assets/img/pyramid_of_pain.png" | relative_url}})

Threat emulation lines up well with the Pyramid of Pain, the MITRE ATT&CK framework, and cyber threat intelligence. 

---

## Emulation Methodologies

Every adversary would follow a methodology and definitely have a workflow for their attacks. The idea of using comprehensive methodologies is to ensure we have a proper knowledge base when dealing with threats and better plan the emulation exercises.  

### MITRE ATT&CK Framework

- Visually represents attackers' techniques. 
- Showcases 14 tactics, with several adversary techniques listed for each of them. 
- An extension is the **MITRE ATT&CK Navigator**, which allows for better filtering and viewing a particular adversary's techniques and sub-techniques. 

[MITRE ATT&CK](https://attack.mitre.org/tactics/TA0001/) 

### Atomic Testing

- Library of emulation tests developed by Red Canary 
- The individual tests (atomics) are mapped to the MITRE ATT&CK framework. 
- The process is as follows: **Choose ATT&CK technique** -> **Choose test for that technique** -> **Execute test** -> **Analyze detections** -> **Make improvements to your defenses**. And the cycle repeats. 
- Atomic Red Team supports emulation from operating systems to cloud environments. 

### TIBER-EU Framework

- **Threat Intelligence-Based Ethical Red Teaming** is an European framework to deliver controlled, intelligence-led emulation testing on entities and an organizations' critical live production systems. 
- Follows three phases for end-to-end adversary testing:
    1. Preparation: scope, teams established. 
    2. Testing: Threat intelligence team will prepare a detailed report to showcase the threat areas. Red team will use that report to craft emulation tests. Blue team will look for the attacks. 
    3. Closure: After tests are run and defenses are measured, emulation team must consider reporting and remediation measures. 

### CTID Adversary Emulation Library

- **Center for Threat-Informed Defense**. 
- Operated by MITRE. 
- Promotes practices of **threat-informed** defense. 
- [Github link to library](https://github.com/center-for-threat-informed-defense/adversary_emulation_library)

---

## Emulation Process

1. Define Objectives
2. Research Adversary TTPs
    1. Information Gathering: gather as much about the threats as you can, avoid non-concerning adversaries. 
    2. Narrow to one adversary you would like to emulate
    3. Narrow down to the TTPs of the adversary you would like to emulate. 
    4. Construct TTP Outline (See below for sample)
        - ![Sample TTP Outline]({{ "assets/img/sample_ttp_outline.png" | relative_url}})
3. Plan the Threat Emulation Engagement
    - Proper planning to avoid disclosure issues, data loss or system downtimes. 
    - You can find emulation plans from CTID. 
    - Scope, Schedule, Objectives, rules of engagement, communication. 
4. Conduct the Emulation
    - Requires skilled professionals who can accurately and precisely replicate the tactics and techniques of target adversary and their TTPs. 
5. Conclusion and Reporting


---

## Task 1 (Carbon Spider)

> Search for carbon spider on MITRE ATT&CK, and use the information from that and the associated ATT&CK NAVIGATOR (which you can download or view from the same webpage) to learn more about the threat group. 
{: .prompt-tip}

1. Assuming you're emulating Carbon Spider, what will you use to achieve Execution Techniques?
    - **Windows Command Shell**
2. While emulating Carbon Spider, what would be used to achieve persistence?
    - **Scheduled Task**
3. There is a POS Malware that Carbon Spider has been known to use to harvest credit card data. What is it?
    - **Pillowmint**
    - > Using POS malware, including PILLOWMINT, the adversary harvested credit card track data and sold this data on criminal forums such as Joker’s Stash.
4. Carbon Spider's Ransomware is: **Darkside**. 

---

## Task 2 (Reaper, APT37)

> Same as with carbon spider, search for reaper. Also find the blog posts and threat write-ups and reports from the references, for more detailed information. 
{: .prompt-tip}

1. Assuming you're emulating Reaper, what will you use to achieve Initial Access Techniques?
    - **Drive-by Compromise**
2. Reaper's Command and Control plan utilises DOGCALL malware. Which platform would be used to deploy it?
    - **Dropbox**
3. If you were Reaper, which Defense Evasion technique would you use?
    - **Steganography**
4. Initial zero-day abused by Reaper was on? 
    - **Adobe Flash**

---

> Information is Power. The more information you can gather, the better you can defend against threats.
{: .prompt-info}


