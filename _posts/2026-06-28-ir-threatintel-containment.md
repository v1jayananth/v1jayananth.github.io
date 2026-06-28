---
title: Incident Response - Threat Intel & Containment
date: 2026-06-28 00:03:00 +0200
categories: [Incident Response]
tags: [incident response, containment, IOCs, threat intelligence]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

## Objective

> Learn what threat intelligence looks like, and some containment strategies used in the IR process.

---

Containment is a crucial phase in incidence response because the core aim is to minimize the damage caused by an incident and prevent further damage.

Threat intelligence, briefly, is the knowledge gained from collecting and analyzing intelligence about a threat actor. Intelligence such as IP addresses can be used to identify a specific threat actor or, for example, analyze their tactics, techniques, and procedures (TTPs).

## Pre-containment

Prevent an incident from having further impact. 

Gather evidence for containment (file hashes for example). 

---

## Containment Strategies

- **Complete Isolation**: Total isolation through network segmentation (even air gapped if possible). Pretty aggressive strategy. A strategy such as this may have drawbacks, since the adversary may also notice it, and rush to complete their action on objectives. They may also change their focus to a system that hasn't been noticed yet. 
- **Controlled Isolation**: Keep the system accessible to the adversary, and note their actions. This way, the incident response team can gather vital information and intelligence about the adversary. However, this isn't risk-free of course. Controlled isolation should be chosen after careful consideration. If enough is known about the adversary, then there would be no need for a step like this. 

---

## Threat Intelligence

Anything that can be attributed to a malicious threat actor. Can include: 

- File Hashes
- IP addresses
- Domains
- File names
- Patterns or techniques (TTPs)

### TTPs

- Tactics: High level strategies the threat actor use. **What are they trying to do? What are their objectives?** 
- Techniques: Specific tools employed. **How did they do it?** 
- Procedures: Attack chain used by the adversary. **What is the process of an attack by the threat actor?**

---

### Threat Intelligence Platforms

Aids in distributing threat intelligence. Allows for collaboration among defenders and ultimately more insight and intelligence. 

**OpenCTI** is a framework that allows for this sort of collaborative sharing of threat intelligence. 

It is also important to subscribe to threat intelligence feeds that will provide valuable information to predict future behaviour of threat actors. 

---

## Practical

In the practical task, we are given a packet capture and we have to find the IP address of the attacker, and file that was downloaded onto the victim machine. This can be done easily by typing `http` in the search bar to narrow the search to http packets, and we will find the relevant information there. 

![Image]({{ "assets/img/containment_1.png" | relative_url}})



---


