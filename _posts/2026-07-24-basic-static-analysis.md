---
title: Basic Static Analysis
date: 2026-07-24 00:00:00 +0200
categories: [Malware Analysis]
tags: [malware analysis, windows, static analysis, binaries, pe]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

## Objective

> Learn basic malware analysis techniques, that do not require running the malware. 

---

## Lab setup

It is always beneficial to use **virtualization** when dealing with malware, because of its destructive nature. You do not want any of your real systems or networks to get affected. Often, the **snapshots** feature available in various virtualization softwares come in extremely handy, as you can revert to and from a state quickly. You could even perform snapshots between extremely small changes when analyzing malware, to understand what it is really doing in the background, and if there are some changes that can be noticed in the machine states. 

There are also specific virtual machines that can help with malware analysis: 

1. **Flare VM**: A windows-based VM well-suited for malware analysis, when dealing with malware that affects windows machines. It was created by Mandiant (previously FireEye). 
2. **REMnux**: Reverse Engineering Malware Linux. A linux-based malware analysis distribution created by Lenny Zeltser in 2010. Since it based on linux, everything is open source. While it can help with performing static analysis even for windows binaries, it cannot be used to perform dynamic analysis on windows binaries. 

---

## Strings

Basic string information in any binary can be extremely useful. Often, you could even get a complete picture of what the malware is doing. 

On windows, `string.exe`, and on linux, `strings` can help with this. 

On FlareVM, `string.exe` should be available, but other tools such as `pestudio` should also be available, which also show all the strings identified. 

Malware authors are aware of this, and that's why they deploy techniques to **obfuscate** strings in their malware. To counteract against this, Mandiant launched **FLOSS** (FireEye Labs Obfuscated String Solver), which uses several techniques to deobfuscate and extract strings that can not be found by a normal string search. 

On FlareVM, with FLOSS installed, you can use `floss` to run the command, and use `floss -h` to view all available options and parameters. You may want to use `floss --no-static-strings <binary>` command, to avoid viewing static strings, and informing floss to only display obfuscated strings that can be decoded. 

---

## Fingerprinting malware

What method can be used to accurately identify a file? A filename can't be used. All of the file's properties, including its contents, must be used to create a fingerprint that uniquely identifies that file. **Hashing** is useful here. The most common hash algorithms are: `sha256sum` or `sha512sum`. 

In `pestudio`, you will also find **imphash** or "Import Hash". The imphash is a hash of function calls/libraries that a malware sample imports. This imphash helps identify samples from the same threat groups or malware that performs similar activities. 

> The same imports in the same order will have the same imphash across different malware samples. 
{: .prompt-info}

### Fuzzy hashes

Another way to identify similar malware is through fuzzy hashes. This is also called a **Context Triggered Piecewise Hash (CTPH)**. This hash is calculated by dividing a file into pieces and calculating hash of the different pieces. 

A utility to use for this would be `ssdeep`. 

![ssdeep]({{ "assets/img/BSA1.png" | relative_url}})

---

## Signature-based detection

Signatures are a way to identify if a file has a particular type of content. This signature can be considered as a pattern that might be found inside a file. A sequence of bytes usually. 

Yara rules are a type of signature-based rule. Yara can identify information based on binary and text patterns, such as hex characters, strings etc within a file. It can be configured quite extensively, and shared with others too. Yara is open-source and community driven. 

Another open source tool is Capa, developed by Mandiant. This tool helps identify the capabilities found in a PE file specifically. 

![capa]({{ "assets/img/BSA2.png" | relative_url}})

Capa with the `-v` flag can also show specific matches along with their address information, allowing for further analysis. 

![capa 2]({{ "assets/img/BSA3.png" | relative_url}})


---

