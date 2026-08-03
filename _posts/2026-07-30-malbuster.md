---
title: Malbuster - Challenge
date: 2026-07-30 00:00:00 +0200
categories: [Malware Analysis]
tags: [malware analysis, windows, binaries, pe, static analysis]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

## Objective

> There are a couple of malicious files to investigate. 

The four samples are: malbuster\_1, malbuster\_2, malbuster\_3, malbuster\_4

> We will be using a REMnux virtual machine to analyze the samples. 
{: .prompt-info}


---


## malbuster\_1

- `file malbuster_1` reveals that it is a **32-bit** PE executable (for Windows)
- MD5 hash: 4348da65e4aeae6472c7f97d6dd8ad8f
- On VirusTotal, the popular threat label is **trojan.zbot/smrl**. 
- Using CAPA, `capa`, we see that this binary has anti-VM techniques, to detect and possibly evade VM sandboxes. 
- `strings` reveals that this binary has the user agent string **Mozilla/4.0 ...**. 


## malbuster\_2

- `file malbuster_2` reveals that it is a **32-bit** PE executable, with possibly **.NET** assembly code. 
- MD5 hash: 1d7ebed1baece67a31ce0a17a0320cb2
- On VirusTotal, popular threat label is **trojan.msil/agenttesla**. 
    - Avira's signature is **HEUR/AGEN.1306860**
- Using CAPA, `capa`, you can find out the capabilities 
- Using `pecheck`, you can find out the PE information. 
    - For example, `_CorExeMain` function is imported, and it is imported from **mscoree.dll** DLL file. 
- You can find out the original filename, based on the **VS_VERSION_INFO** header, using `pecheck`
    - ![internal original name, vs_version_info]({{ "assets/img/malbuster_1.png" | relative_url}})
- `strings` command reveals that this binary contains the string **GodMode**. 


## malbuster\_3

- **32-bit** PE executable
- MD5 hash: 47ba62ce119f28a55f90243a4dd8d324
- On abuse.ch, that is, **MalwareBazaar** platform, this hash reveals this is TrickBot malware. 
- `capa` reveals that this binary can log keystrokes. 
    - **log keystrokes via application hook**
    - **log keystrokes via polling** 

## malbuster\_4

- **32-bit** PE executable, Possibly DLL file. 
- MD5 hash: 061057161259e3df7d12dccb363e56f9
- MalwareBazaar reveals that this is **ZLoader**'s signature. 
    - ![Zloader]({{ "assets/img/malbuster_2.png" | relative_url}})
- We can use `xxd` to find if there is a special message in the DOS_STUB of this malware
    - ![DOS stub]({{ "assets/img/malbuster_3.png" | relative_url}})
- This malware imports **ShellExecuteA** function. From `pecheck`, we can gather that this is from **shell32.dll**. 
- `capa` reveals some useful MITRE IDs 
    - ![capa mitre]({{ "assets/img/malbuster_4.png" | relative_url}})


---

> `strings -f ./* | grep <string>` can help with finding some key strings across all binaries. Useful in real life scenarios for quick triaging. 
{: .prompt-tip}

---


