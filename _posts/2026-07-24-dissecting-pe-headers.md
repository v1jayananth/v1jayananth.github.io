---
title: Dissecting PE headers
date: 2026-07-24 00:00:00 +0200
categories: [Malware Analysis]
tags: [malware analysis, windows, binaries, pe]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

## Objective


> Learn about Portable Executable files and how their headers work.

---

PE headers can be analyzed using tools such as **pe-tree** (available on linux based distributions such as **REMnux**) since viewing the raw hex bytes would be too tedious. 

![PE header]({{ "assets/img/PEheader1.png" | relative_url}})


PE headers include: 

1. DOS Header / IMAGE_DOS_HEADER
2. DOS Stub
3. NT Headers / IMAGE_NT_HEADERS
    - File Header
    - Optional Header / OPTIONAL_HEADER
    - PE Signature

and more. We will discuss most of them in this room. 

> All PE headers are of data type **struct**. That is, they combine several different types of data in a single variable. And more importantly, it is user-defined. 
{: .prompt-info}


---

## IMAGE_DOS_HEADER 

The IMAGE_DOS_HEADER consists of the **first 64 bytes** of the PE file. The first two bytes start with **4D 5A** in hex. They translate to **MZ** in ASCII. These are the initials of **Mark Zbikowski**, who created the MS-DOS file format. The **MZ** bytes identify the Portable Executable format. These are the **Magic Bytes** that indicate to the OS that this is a Windows Portable Executable format file. 

> The `e_lfanew` address at the end of the header, indicates the start of the IMAGE_NT_HEADERS
{: .prompt-info}


---

## DOS Stub

The DOS Stub follows the IMAGE_DOS_HEADER, and it contains the following message: "!This program cannot be run in DOS mode". This can be easily viewed in hex editors or any program to analyze bytes of the PE file. 

This stub only runs if the PE file is **incompatible** with the system it is being run on. For example, if we run this specific PE file in MS DOS, it will exit after showing this message. 

---

## IMAGE_NT_HEADERS

This header contains most of the vital information related to the PE file. 

It contains four sub-headers: 

1. NT_HEADERS
2. IMAGE_SECTION_HEADER
3. IMAGE_IMPORT_DESCRIPTOR
4. IMAGE_RESOURCE_DIRECTORY

### NT_HEADERS

Using `e_lfanew` to find the starting address of IMAGE_NT_HEADERS, we would find that `50 45 00 00` as the bytes at this address, or characters `PE` in ASCII. This is the **PE signature**. 

NT_HEADERS also contains **FILE_HEADER** and **OPTIONAL_HEADER**. 


#### FILE_HEADER

Contains information about 

- the architecture for which this PE file was written for
- the number of sections (where code, variables and other resources are stored)
- time and date of binary compilation
- the size of optional header
- And characteristics of the binary such as LINE_NUMS_STRIPPED (line number information is removed from the binary) and RELOCS_STRIPPED (relocation information removed out of the binary). 

![file header]({{ "assets/img/PEheader2.png" | relative_url}})

#### OPTIONAL_HEADER

Starts right after the end of FILE_HEADER, and contains information about the PE file such as AddressOfEntryPoint, ImageBase, CheckSum information, Linker versions and more. 

![optional header]({{ "assets/img/PEheader3.png" | relative_url}})

Some important fields in this header include: 

- Magic: Indicates whether the PE file is 32-bit or 64-bit application. `0x010B` indicates 32-bit and `0x020B` indicates 64-bit. 
- AddressOfEntryPoint: Address from where Windows will begin execution of this binary. This is an offset of the ImageBase. 
- BaseOfCode and BaseOfData indicate the base addresses of the code and data section respectively. 
- ImageBase: Preferred loading address of the PE file in memory. 
- Subsystem: Can be - Windows Native, GUI, CUI or some other subsystem. 
- Data Directory: Contains import and export information of the PE file. 

---

### IMAGE_SECTION_HEADER

Every PE file needs some code, some data, to run. These are stored in the IMAGE_SECTION_HEADER. 

- `.text`: Section that contains the executable code for the application. Contains EXECUTE, and READ permissions, but not WRITE permissions. 
- `.data`: Section that contains the initalized data of the application. It has READ/WRITE permissions but doesn't have EXECUTE permissions, to avoid allowing user-supplied data to be executed. 
- `.rdata/.idata`: Section that often contains the import information of the PE file. 
- `.ndata`: Section that contains uninitalized data. 
- `.reloc`: Section that contains relocation information of the PE file. 
- `.rsrc`: Section that contains icons, images and other resources required for the application.  

---

### IMAGE_IMPORT_DESCRIPTOR

The IMAGE_IMPORT_DESCRIPTOR structure contains information about the different Windows APIs that the PE file loads when executed. This information is handy in identifying the potential activity that a PE file might perform. 

![imports]({{ "assets/img/PEheader4.png" | relative_url}})

---

## Packing and identifying packed executables

As we have seen, a PE file's information can be easily viewed using a Hex editor or a tool like `pe-tree` (there are countless tools to view PE file information). Attackers do not want this. How do they evade it and ensure their malicious code cannot be reverse engineered so easily? They use something called **packers**. 

A packer is a tool to obfuscate the data in a PE file so that it can't be read without unpacking it. It adds a layer of obfuscation to avoid reverse engineering. This makes static analysis **useless**. The only way to learn what happens with the PE file, is to let it run and use dynamic analysis to identify when it gets unpacked and how. 

How can we identify packers? 

Well, **IMAGE_SECTION_HEADER** can help us a lot here. The different sections will have a field titled **Entropy**. It can have values from 0 to 8. The default value for a normal application should be near 6. If it has a higher entropy than 6, and more than 7 specifically, then it can be fairly assumed that the executable is packed. The different sections will have their own entropy, but the executable as a whole will also have an overall entropy. We can use a tool such as `pecheck` in linux to view this information, even though `pe-tree` will provide it as well. 

> Entropy represents the level of randomness. Packed executables are obfuscated and therefore more random than a normal application. 
{: .prompt-info}

> Another key information to identify packed executables or sections, would be to look at their permissions. They would usually have READ, WRITE and EXECUTE permissions, because often all three would be required to successfully pack and unpack. 
{: .prompt-info}

![entropy]({{ "assets/img/PEheader5.png" | relative_url}})


![entropy 2]({{ "assets/img/PEheader6.png" | relative_url}})

---

> PEstudio can also help identifying malicious libraries imported by a library, as shown by the image below. 
{: .prompt-info}

![blacklisted libraries]({{ "assets/img/PEheader7.png" | relative_url}})

---
