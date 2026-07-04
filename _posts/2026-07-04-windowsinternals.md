---
title: Windows Internals for Malware Analysis
date: 2026-07-04 00:00:00 +0200
categories: [Malware Analysis]
tags: [windows, malware analysis, dll, exe, pe, threads, procmon, die]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

## Objective


> Learn and understand the fundamentals of how Windows operates at its core, for the broader purpose of malware analysis for windows. 

---

## Processes

A process is core to any operating system. It represents the execution of a program. It is an **instance** of the program. Thus, a program can contain one or more processes. A process has many components such as its own virtual address space, executable code, open handles to system objects, security context, process identifier, environment variables, and more. 

All processes have atleast one thread of execution. 

![Process in Memory]({{ "assets/img/windowsinternals_3.png" | relative_url}})

In windows, processes can be easily observed using tools such as **Process Explorer** and **ProcMon**. An example of ProcMon is shown below

![ProcMon Filter]({{ "assets/img/windowsinternals_2.png" | relative_url}})

![ProcMon Process Properties]({{ "assets/img/windowsinternals_1.png" | relative_url}})


### Threads

A thread is an executable unit employed by a process. More simply, a thread could be defined as a unit controlling the execution of a process. 

In terms of security, threads are commonly targeted, as it could be manipulated for code execution. 

Threads share the same details and resources as their parent process, such as code, global variables etc. They also have their unique values such as 

- Stack (holding data relevant to the thread) 
- Thread Local Storage (Pointers for allocating storage to unique data environment)
- Stack Argument (Unique value assigned to each thread) 
- Context Structure (Holds machine register values maintained by the kernel)

A thread has a thread ID, and a stack argument for itself. In the below example, 6584 would be the stack argument, while 5908 is the thread id. 

![Thread]({{ "assets/img/windowsinternals_4.png" | relative_url}})

---

## Virtual Memory

Allows internal components to interact with memory as if it was physical memory. The best way to understand would be with the example of developers of applications not having to worry about actual physical memory addresses in their program. The system would take care of it. The processes and threads would think the range of addresses they have is all for themselves, and that they are the only unit of execution in the system. This way, process isolation is achieved. The **Memory Management Unit** would abstract and handle all of this. 

On a **32-bit** system, the *theoretical* **maximum virtual address space** is **4 GB** (2^32). The upper half would be allocated for the OS, and the lower half for the user space, for applications to run. 

On **64-bit** systems, the limit is **256 TB**. 

> `increaseUserVa` or **Address Windowing Extension (AWE)** can be used to reallocate user process address space, if an application requires a larger address space. 
{: .prompt-info}

> On ProcMon, for a process, you can find it's base address by searching for **Image Base** value in the details. 
{: .prompt-tip}

---

## Dynamic Link Libraries

DLLs are libraries that contain code and data that can be used by more than one program at the same time. 

When a DLL is loaded as a function in a program, the DLL is assigned as a dependency. Attackers can target this dependency rather than the applications directly. 

DLLs are very similar to PE files. They just have a slight internal syntax modification. 

DLLs can be loaded in a program in two ways: 

1. Load-time dynamic linking: **Explicit** calls to the DLL functions are made from the application. This can be achieved only by providing a header (*.h*) and import library (*.lib*) file. The functions in the header file can be called directly. 
2. Run-time dynamic linking: A separate function (**LoadLibrary** or **LoadLibraryEx**) is used to the load the DLL at run time. Once loaded, `GetProcAddress` (Get Process Address) needs to be used to identify the exported DLL function to call. 

Threat actors will often use Option 2. 

---

## Portable Executable (PE) Format 

This format defines information about the executable and stored data. 

The PE and COFF (Common Object File Format) files make up the PE format. 

PE data is mostly seen in the hex dump of an executable file (Starting with an **MZ...**, which are the magic bytes, or header that defines the file format as `.exe`). This is followed shortly by the **DOS Stub**, which prints the message "This program cannot be run in DOS mode". 

The PE File Header provides PE header information of the binary. Starts with **PE..** (when viewing hex dump). 

> PE information can be viewed easily using **Detect It Easy** on Windows (examples shown below). 
{: .prompt-tip}

![DIE 1]({{ "assets/img/windowsinternals_5.png" | relative_url}})

![DIE 2]({{ "assets/img/windowsinternals_6.png" | relative_url}})

---

