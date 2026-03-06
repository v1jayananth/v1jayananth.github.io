---
title: Threat Hunting
date: 2026-03-06 00:00:00 +0100
categories: [Threat Hunting]
tags: [threat hunting, threat intelligence, SOC]
series: ""
series_order: 
---
Threat hunting is a **proactive cybersecurity practice** where analysts actively search through networks, endpoints, and datasets to detect malicious activity that has **evaded existing automated security tools**. Unlike reactive security (waiting for alerts), threat hunters go looking for threats before damage is done.

Threat Hunting is **proactive** while Incident Response is **reactive**. 

Primary Goal is to **catch existing threats**: Attackers who have bypassed firewalls, EDR, SIEM, and other controls and are quietly dwelling in the network. Studies show the average dwell time of an attacker is weeks to months before detection. Threat hunting aims to slash that window dramatically.

The secondary benefit includes a reduced attack surface for future adversaries, in the context of existing infrastructure / tools. If technology is constantly evolving, then the attack surface keeps changing or increasing. So the best thing to do is to keep looking out for threats, and assessing the attack surface constantly. 

## General Threat Hunting Process

1. 🧠Form a Hypothesis
    - How an attacker *might* be operating in your environment. 
    - Use Threat Intelligence which could split into two types:
        - Unique Threat Intelligence includes Indicators of Compromise, since it is unique to your organization, and thus highly valuable. 
        - Threat Intelligence feeds allow learning from other organizations who can produce Threat Intelligence. Examples include MISP (A platform for sharing threat intel), Recorded Future, ReliaQuest, Digital Shadows. 
2. Collect & Organize Data
    - Logs. Logs from everything (network, endpoint, authentication, cloud).
    - The more quality, the better.
3. Investigate
    - Use analytical techniques to find patterns in the data, or anomalies. 
4. Identify & **Triage** findings
    - Separate true positives (real threats) from false positives. Escalate confirmed threats to incident response. **Document everything** — even dead ends provide value.
5. 🔄Feed Back Into Defenses
    - Patch or harden exposed systems/tools.
    - Create new SIEM detection rules.
    - Share threat intelligence internally and externally.


![General Threat Hunting Process]({{ "assets/img/threat_hunting_process.png" | relative_url }})


## What is MITRE ATT&CK Navigator?


[MITRE ATT&CK Navigator](https://mitre-attack.github.io/attack-navigator/) is a free, web-based tool built on top of the ATT&CK framework that lets you visually annotate, layer, and interact with the ATT&CK matrix. Think of it as a heatmap editor for attack techniques. This tool helps you gather relevant information from the ATT&CK framework, thus helping to identify the TTPs (Tactics, Techniques and Procedures) of an attacker based on real-life observations. 

## Why do we need Threat Hunting? 

- **Proactive** approach to finding malicious actors
- You may also stumble upon malicious threat actors in your network while threat hunting, and that would trigger **Incident Response**. 
- Primary goal of threat hunting is to minimize a threat actor's dwell time in your network, thus limiting the time they have, to act on their objectives. 
- Nation-state and sophisticated criminal actors use novel, low-and-slow techniques specifically designed not to trigger alerts. **Only human intuition** can find them.
- SOC analysts are overwhelmed by volume. Threat hunting complements reactive work by proactively going after the most dangerous threats.
- Security is a **continuous process**, and threat hunting allows for better detection mechanisms, and the cycle continues. 

> Hunters find what automation misses.
{: .prompt-info}

> [Repository of Known Malware Samples for research in malware analysis](https://github.com/ytisf/theZoo)
{: .prompt-info}
