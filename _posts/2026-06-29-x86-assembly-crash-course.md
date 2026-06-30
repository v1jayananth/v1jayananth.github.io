---
title: x86 Assembly Crash Course
date: 2026-06-29 00:00:00 +0200
categories: [Malware Analysis]
tags: [malware analysis, x86, 32-bit systems, assembly, reverse engineering]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

## Objective

> A crash course in x86 assembly to enable us in malware reverse engineering.

When learning malware reverse engineering, knowing the basics of assembly language is essential. This is because when we get a malware sample to analyze, it is most likely a compiled binary. To understand this compiled binary, we would have to **decompile** it, which would translate 1's and 0's back to the original source code, except that a lot of the original information is lost (such as variable names, function names, etc.). **Decompilers** essentially guess how the source code might have been structured to result in that specific binary. 

As a result, the **most reliable code** we do have, is **its assembly code** (obtained by using a **disassembler**),  which is not exactly 1s and 0s, but rather what a specific CPU requires. This is more reliable because assembly code is essentially just **human-readable** representation of the 1s and 0s running on the CPU. **A direct 1:1 mapping**. 

---

## Opcodes and Operands

Machine bytes are just 1s and 0s. To make it understandable, we can group 4 binary bits to 1 hex digit. For example, the number **10100101**: 

- **1010**, first 4 digits, can be converted to **A** in hex (equivalent to 10 in decimal). 
- **0101**, next 4 digits, can be converted to **5** in hex

**Opcodes** are numbers that correspond to instructions performed by the CPU. It translates those opcodes to make them human readable. 

For example: 

- `040000:    b8 5f 00 00 00    mov eax, 0x5f`
- Instruction: `mov`, Operands: `eax` and `0x5f`. `0x5f` is moved into eax. 
- `040000:` corresponds to the address where this instruction is located. 
- `b8` refers to the opcode of the instruction `mov eax`, 
- `5F 00 00 00` indicates the other operand. This is in **little endian notion**, where the order of bytes are reversed. The least significant byte (LSB) is written first, and the most significant byte (MSB) is written last. 
- Here, `0x5f` is an **Immediate operand**, `eax` is a **register operand**, and something like `[eax]`, where the **address of eax** is used, would be a **memory operand**. 

---

## General Instructions

1. `MOV` instruction
    - `mov destination, source`
    - `mov ebx, eax`, would move value in eax to ebx
    - `mov eax, [0x5fc53e]`, would move value stored in the given memory location to eax
    - `mov eax, [ebp+4]` would calculate `ebp+4`, take the value at the memory location to move it to eax. 
    - `mov [ebx], [eax]` will not be allowed. Memory to memory data movement is not allowed. 
2. `LEA` instruction
    - **Load Effective Address**, moves the **address of source into the destination**
    - `lea destination, source`
    - `lea eax, [ebp+4]` would load the memory address `ebp+4` into eax. The value at `ebp+4` is of no concern in this case. 
    - `lea eax, ebp+4` would be a syntax error. 
3. `NOP` instruction
    - **No operation**
    - Used for consuming CPU cycles while waiting for another operation. 
    - Used by malware authors when directing execution to malicious shellcode. Sometimes, the authors wouldn't know the exact addresses, and using `nop`, especially many of them, they can jump to a location where they have placed a `nop`, and their malicious shellcode after the `nop`s would execute reliably. 
4. Shift instructions
    - `shr destination, count` and `shl destination, count`. The bits in the destination operand are shifted **right or left** by count operand. 
    - If we have `00000010` in eax, and shift it left once, it becomes `00000100`. This is essentially multiplying it by two. The `0` that was pushed out, will be stored in the `CF` or **Carry Flag**. 
    - **Shifting left** each bit is **multiplying by 2**. 
    - **Shifting right** each bit is **dividing by 2**. 
5. Rotate instructions
    - Similar to shift, but the bits are **rotated back to the other end of the register**. 
    - `ror` and `rol`, similar to `shr` and `shl`. 

---

## Flags

In x86, CPU uses several flags to indicate the outcome of operations and conditions. These flags are stored as bits in a special register known as the **flags register** or **EFLAGS** register. 

| Flag | Short Form | Description |
| --- | ---- | ---- | 
| Carry | CF | Set when a carry-out or borrow is required from the most significant bit in an arithmetic operation. Also used for bit-wise shifting operations. |
| Parity | PF | Set if the least significant byte of the result contains an even number of 1 bits.|
| Auxiliary | AF | Set if a carry-out or borrow is required from bit 3 to bit 4 in an arithmetic operation (BCD arithmetic).|
| Zero | ZF | Set if the result of the operation is zero.|
| Sign | SF | Set if the result of the operation is negative (i.e., the most significant bit is 1).|
| Overflow | OF | Set if there's a signed arithmetic overflow (e.g., adding two positive numbers and getting a negative result or vice versa).|
| Direction | DF | Determines the direction for string processing instructions. If DF=0, the string is processed forward; if DF=1, the string is processed backward.|
| Interrupt Enable | IF | If set (1), it enables maskable hardware interrupts. If cleared (0), interrupts are disabled.|

---

## Arithmetic Operations

1. `ADD`
    - `add destination, value`
    - Value is added to destination and **result is stored in destination**.
2. `SUB`
    - `sub destination, value`
    - Value is subtracted from destination and result is stored in destination. 
    - If destination is smaller than the value, the CARRY flag is set, because the operation needs to borrow (when dealing with unsigned values. Signed values involve the sign flag, the SF is checked against the OF)
    - If destination is greater than value, result is non-zero, flags are not used (when dealing with unsigned values. Signed values involve checking the SF and OF again, both must be equal)
3. `MUL`
    - `mul value`
    - Uses **eax** register
    - multiplies value with eax, and stores result in `edx:eax` as a 64-bit value. Sometimes, value may be higher than 32 bits, and that's why two registers are required. Lower 32 bits in eax, and higher 32 bits in edx. 
    - **In 64-bit systems, rax is used, which is a 64-bit register by itself**. In those systems, `rdx:rax` is used as a 128-bit value. 
4. `DIV`
    - `div value`
    - Works the opposite way of multiplication. **result in eax** and **remainder in edx**, for 64-bit values. 
5. `INC`
    - `inc eax`
    - the operand register is incremented by 1, i.e. its value is raised by 1 and stored there. 
6. `DEC`
    - `dec eax`
    - decrements eax by 1. 

---

## Logical Operations

1. `AND`
    - This operation performs a **bitwise AND** on the operands. In other words, this operation is applied on every bit of each operand against each other. 
    - Returns 1 if both inputs are 1, otherwise 0. 
    - `and al, 0x7c`
    - Suppose `al` has value `11111100` and `0x7c` is `01111100`. The above operation would result in: `01111100`
2. `OR`
    - Returns 1 if atleast one input is 1. Otherwise 0. 
    - With the same instruction and values above, but with `or`, the result would be: `11111100`
3. `NOT`
    - `not al`
    - Simple **inverts** the operand bits, replacing 1s with 0s and vice versa. 
4. `XOR`
    - **Exclusive OR**
    - Similar to OR, but **only one of the inputs must be a 1, for the result to be a 1**. In other words, **the inputs have to be different for the result to be 1. If the inputs are the same, the result is 0**. 
    - With the same instruction and values (`and al, 0x7c`), the result would be: `10000000`
    - XORing a register with itself, would result in 0s, which would be stored in that register itself. Useful operation to zero a register. More optimized than MOV. 

---

## Conditionals

The CPU uses conditional instructions to determine if two values are equal to, less than or greater than each other. 

1. `TEST` instruction
    - `test destination, source`
    - Performs **bitwise AND**, but instead of storing the result, it sets the **Zero Flag (ZF)** if the result is 0. 
    - This instruction is often used to check if an operand has a NULL value (`if val == 0`), by testing the operand against itself (only if the operand has all bits equal to 0, does the result equal 0). 
    - `test` is used rather than comparing (`cmp`) directly with 0 in assembly, because it takes fewer bytes to use the test instruction. 
2. `CMP` instruction
    - `cmp destination, source`
    - Compares two operands and sets the Zero Flag (ZF) or the Carry Flag (CF)
    - Similar to subtract instruction, but operands are not modified.
    - ZF is set if both operands are equal. 
    - CF is set **if source is greater than destination**. 
    - ZF and CF are cleared if destination is greater than source operand. 

---

## Branching

Most programs have branching operations, and are not a linear program (one instruction after another). Some conditions are checked, and different paths are taken based on the data. 

In assembly, the relevant instructions are as follows: 

- `JMP` instruction
    - `jmp location`
    - the program flow would jump to the given memory address after the above instruction. 
- Conditional jumps. There aren't actual `if` statements in assembly, but the same functionality is implemented using conditional jumps, which take the jump if certain flags are set or unset. 
    - `jz`: Jump if ZF=1
    - `jnz`: Jump if ZF=0
    - `je`: Jump if equal (often used after CMP instruction)
    - `jne`: Jump if not equal (often used after CMP instruction)
    - `jg`: Jump if destination greater than source operand (Performs signed comparison and often used after CMP instruction)
    - `jl`: Jump if destination less than source operand (Performs signed comparison and often used after CMP instruction)
    - `jge`: Jump if greater than or equal to. Similar to `jg`
    - `jle`: Jump if less than or equal to. Similar to `jl`
    - `ja`: Jump if above. Similar to `jg`, but unsigned comparison. 
    - `jb`: Jump if below. Similar to `jl`, but unsigned comparison. 
    - `jae`: Jump if above or equal to. Similar to `ja`
    - `jbe`: Jump if below or equal to. Similar to `jb`

---

## Stack and Function calls

The stack involves `push` and `pop` instructions to move things into and out of the stack. They work as follows: 

- `push source`
    - Pushes source operand onto the stack. Value of operand is stored at memory address pointed by the stack pointer (ESP in 32-bit systems and RSP in 64-bit systems). 
    - Becomes new top of the stack. 
    - Stack pointer is then decremented to allow next value or instruction to be placed onto the stack (stack grows downwards. So when value is decremented, it is actually growing). 
    - `pusha`: Push all words. Pushes all 16-bit general purpose registers such as **AX, BX, CX, DX, SI, DI, SP, BP**.
    - `pushad`: Push all double words. Pushes all 32-bit general purpose registers such as **EAX, EBX, ECX, EDX, ESI, EDI, ESP, EBP**. 
    - **The above two instructions could be signs of someone manually injecting assembly instructions to save the registers values. Often the case with shellcode.** 
- `pop destination`
    - Takes the value from the top of the stack, stores it in destination. 
    - The stack pointer is incremented (shrinks) because this value is popped out of the stack, and hence not within the stack anymore. 
    - `popa` and `popad` work the same as `pusha` and `pushad`, except that it pops values sequentially from the stack onto the general purpose registers in the following order: **DI, SI, BP, BX, DX, CX, AX** or **EDI, ESI, EBP, EBX, EDX, ECX, EAX**
- `call location`
    - Used in assembly language for performing a function call. 
    - Depending on the convention of the function call, arguments are placed on the stack or in the registers. The function prologue prepares the stack by adjusting EBP and ESP, and pushing return address on the stack to return back to the original flow after the function call is over. 


---

