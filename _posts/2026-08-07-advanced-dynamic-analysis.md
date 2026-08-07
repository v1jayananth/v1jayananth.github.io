---
title: Advanced Dynamic Analysis - Debugging Overview
date: 2026-08-07 00:00:00 +0200
categories: [Malware Analysis]
tags: [malware analysis, debugging, dynamic analysis, code execution, evasion]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

## Objective


> Learn more advanced techniques of dynamic malware analysis. 

To defeat evasion techniques of malware, a malware analyst desires **more control** over malware execution. 

---

## Evasion of Basic Dynamic Analysis

Generally, malware authors try to code their malware to identify if it ever runs in a controlled analysis environment. This is done by: 

1. Identification of VMs: Checks for registry keys or device drivers (`VBoxMouse` or `VBoxKeyboard`) that indicate a virtual environment. If identified, malware takes a **different execution path** that is probably not malicious to fool the analyst. 
2. Timing attacks: Malware will often try to **time out automated analysis systems**. While there are newer mitigations to prevent this kind of behaviour, malware can also identify those mitigations by performing targeted timing checks to see if the time is being manipulated. 
3. Traces of user activity: Malware tries to observe the activity in the system. If there are next to no traces, it can be easy to figure out that it is within a controlled environment. These traces could include mouse activity, keyboard activity, browser history, system uptime etc. 
4. Identification of analysis tools: Malware could ask for running processes, in Windows for example via `Process32First` or `Process32Next`. If there are any processes running, associated with malware analysis tools, malware could take a benign execution path. 

---

## Debugging Overview

Debuggers are a widely used tool to identify and fix bugs in a program. Interactive debugging is an essential part of advanced malware analysis, because it allows stepping through the instructions, and being able to identify the state of the system in a controlled manner during the execution of the malware. 

It provides the control the malware analyst desires over malware execution. 

There are three levels of debuggers: 

1. Source-Level Debuggers: Works at the source code level. 
2. Assembly-Level Debuggers: Assembly level, after code compilation. **This is often the case with malware analysis**. This level allows insight into CPU register values and debuggee's memory. The debugger attaches to the program to be debugged. 
3. Kernel-Level Debuggers: Kernel Level, a level deeper and offering much more control. For this level, **two systems are required**. One system to debug the code running on the other system. This is done because, if the kernel stops during a breakpoint, the whole system will stop. 

---

## Debugging malware in Windows: Overview

Best popular options: **x32dbg** and **x64dbg**, same software but meant for 32-bit and 64-bit architectures respectively. 

![x32dbg interface]({{ "assets/img/ada_1.png" | relative_url}})
_x32dbg interface_

Breakpoints are points where the execution of the program is paused for the analyst to analyze the registers and memory. A breakpoint on a specific instruction can be enabled by clicking the dot in front of that instruction in the CPU tab.

The Memory Map shows the memory layout of the program, revealing which parts are readable, writeable and executable. 

![x32dbg memory map]({{ "assets/img/ada_2.png" | relative_url}})
_x32dbg memory map_

> The CPU tab is the main tab that shows the disassembly view (all assembly instructions). 
{: .prompt-info}

> If a process opens a file or another process, the Handles tab will contain information about it. 
{: .prompt-info}

---

## Debugging in Practice

When we open a file in the debugger, a blank cmd prompt might be present along with it. This is because the process started, but we paused it in the debugger by opening it there. 

![x32dbg command prompt]({{ "assets/img/ada_3.png" | relative_url}})

As we can see, at the top left of the debugger window, there are a couple of options that are general to debugging: "Reset", "Stop", "Next", "Pause", "Step into" and "Step over" if you view from the left (except for the first icon). 

x32dbg offers the option to set **automatic breakpoints**, and this can be viewed by going to **Options** -> **Preferences**. 

![x32dbg automatic breakpoints]({{ "assets/img/ada_4.png" | relative_url}})
_x32dbg automatic breakpoints_

> TLS callbacks are often used as an **Anti-reverse engineering technique**. Better to **Step into** the instruction. 
{: .prompt-warning}

![x32dbg conditional instructions]({{ "assets/img/ada_5.png" | relative_url}})
_x32dbg conditional instructions_

Since the **ZF (Zero flag is set)** (on the right pane), the jump is **not taken**. We will step into the call instruction that follows it. 

The function that we step into, seems to have important API calls such as **CreateToolhelp32Snapshot** (used for enumerating running processes), **LoadLibraryA**, **GetProcAddress**, and **SuspendThread**. This TLS callback will ensure the program will freeze if we continue with execution. This is a **code evasion path**. To skip, we must **unset the ZF call** before the `jne` instruction, and ensure the jump **is taken**. 

![x32dbg jump taken]({{ "assets/img/ada_6.png" | relative_url}})
_Restarting the debugger, we ensure ZF is set 0, and then step into, which ensures the EIP moves `pop ebp` instruction._

> We did this once, by changing the value in the debugger. Running the program again, we will see that it goes down the unwanted execution path. In order to make this change **Permanent**, we must **patch** it. 
{: .prompt-info}

We can do this by **Editing the assembly, and changing the instruction to a `je`**, or we can **Fill with NOPs** (No operations), that don't do anything, and go to our preferred instruction. 

![x32dbg fill with NOPs]({{ "assets/img/ada_7.png" | relative_url}})

Once done, go to File -> **Patch File**. 

![x32dbg nops]({{ "assets/img/ada_8.png" | relative_url}})

---

> This is just an overview into dynamic analysis through debugging. This is a very interesting yet complex field with too much knowledge to compress and share here. Feel free to explore more on the internet, there is an endless list of resources!
{: .prompt-info}


---
