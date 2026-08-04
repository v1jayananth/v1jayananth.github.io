---
title: Basic Dynamic Analysis
date: 2026-08-04 00:00:00 +0200
categories: [Malware Analysis]
tags: [malware analysis, windows, pe executable, regshot, api logger, procmon]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

## Objective


> Learn how to analyze malware dynamically by running them in a Virtual Machine.

Malware can use techniques to hide its features from being analyzed. But while it can be extremely good at hiding its features from being **statically analyzed**, since the main purpose is to execute, the real nature will be revealed when executed, and we can analyze that too, **dynamically**. 

In this module, we will explore: 

- Sandboxing. 
- ProcMon, to monitor a process's activity (on Windows). 
- API logger.
- ProcExp to identify if a process is modified **maliciously**. 
- And finally, use Regshot to track registry changes made by malware. 

---

## Sandboxing

Ideally, we would want: 

1. An isolated machine, that is not connected to any other system, has controlled network access, and dedicated to malware analysis. The controlled network access must be carefully implemented, and if it is deemed not necessary for analyzing, then completely removed. 
2. Virtualization, to have the ability to save machine states and move between them. 
3. Monitoring tools to analyze malware while it is executing. 
4. File sharing mechanism to send analysis data out to us. This must be carefully implemented. 

There are even a wide range of online solutions, such as Any.run or Hybrid Analysis. Then there are dedicated sandbox solutions such as Cuckoo Sandbox. But ultimately, it can be any virtual machine, that has been hardened properly and cut off from other systems and networks that it can under no circumstances harm. Obviously, it should match the OS for which the binary is built for. 

---

## ProcMon

ProcMon can help analyze the execution of a PE executable well. To do so, we just have to ensure we clear all the logs, and capture at the right moment, the executable to be analyzed gets executed, so that we don't have unnecessary logs in the way. 

ProcMon has advanced filtering that can help narrow down quickly, and find relevant information. 

It also has the feature to display all parent-child process relationships among the captured information, forming a process tree, giving a complete insight into the processes running on the system. 

![procmon process tree]({{ "assets/img/bda_1.png" | relative_url}})

---

## API Logger

Windows provides APIs for performing a wide range of tasks. Often, a malware uses them too, and it would be useful to monitor the APIs a malware calls when executed. 

**API Logger** is a simple tool that provides basic information about APIs called by a process. 

![api logger]({{ "assets/img/bda_2.png" | relative_url}})



---

## Process Explorer

Process Explorer is a very powerful tool that can help us identify process hollowing and masquerading techniques.

Using the processes's properties, we can find out if the executable running is signed by a verified entity. 

![procexp]({{ "assets/img/bda_3.png" | relative_url}})

In the above image, clicking on "verify" leads to the message "No signature was present in the subject". This executable was therefore not signed by Microsoft and is **masquerading** as one. 

> If process hollowing was performed, then the verification would work, because the process is legitimate, but it has been hollowed out and replaced with malicious code. 
{: .prompt-danger}

> Under strings, you can view the strings from the Image, and the Memory. This can help detect Process Hollowing!
{: .prompt-tip}

Enabling the **lower pane**, allows insight into more details like the registry keys, mutexes, thread information etc. 

![lower pane procexp]({{ "assets/img/bda_4.png" | relative_url}})

---

## Regshot

Regshot is a tool that identifies any changes to the registry or file system selected. Regshot does this by taking snapshots of the registry and comparing them, before and after the execution of the malware. 

If we choose "Scan dir" option, we can scan for file changes in the file system. 

But if we just want a normal registry check, then we can start by clicking "1st shot", which will take a snapshot of the registry. After that is done, we will execute the malware, and take the "2nd shot", and then compare. 

![regshot first]({{ "assets/img/bda_5.png" | relative_url}})

After executing the malware, and taking the 2nd shot, we can choose **Compare**, to compare the two shots, and we can choose **Compare and Output**, to view what was changed. 

![regshot compare]({{ "assets/img/bda_6.png" | relative_url}})


---
