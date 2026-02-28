# Chapter 9.1: 80286 Processor

## 📚 Chapter Overview

This chapter covers the Intel 80286 processor, the first x86 processor with protected mode, introducing hardware-based memory protection and multi-tasking support.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Understand the 80286 architecture and improvements over 8086
- Explain protected mode operation
- Describe segment descriptors and privilege levels
- Understand the addressing scheme and memory management

---

## 1. 80286 Architecture Overview

### 1.1 Basic Specifications

```
80286 PROCESSOR SPECIFICATIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌────────────────────────────────────────────────────────────────┐
│                                                                │
│   Year Introduced:     1982                                    │
│   Transistor Count:    134,000                                 │
│   Process Technology:  1.5 µm                                  │
│   Clock Speed:         8 MHz, 10 MHz, 12.5 MHz (later 20 MHz)  │
│                                                                │
│   Data Bus Width:      16 bits                                 │
│   Address Bus Width:   24 bits                                 │
│   Physical Memory:     16 MB (2^24)                            │
│   Virtual Memory:      1 GB (2^30)                             │
│                                                                │
│   Operating Modes:     Real Mode, Protected Mode               │
│   Package:             68-pin PLCC or PGA                      │
│                                                                │
└────────────────────────────────────────────────────────────────┘

IMPROVEMENTS OVER 8086:
─────────────────────
✓ 24-bit addressing (1 MB → 16 MB physical memory)
✓ Protected mode for multi-tasking
✓ Hardware memory protection
✓ 4 privilege levels
✓ Virtual memory support (up to 1 GB)
✓ Faster execution (same clock = 2-3× faster)
✓ 68 pins vs 40 pins
```

### 1.2 Block Diagram

```
80286 BLOCK DIAGRAM
━━━━━━━━━━━━━━━━━━━

┌─────────────────────────────────────────────────────────────────────┐
│                            80286 CPU                                │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    Bus Unit (BU)                             │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌──────────────────┐     │   │
│  │  │ Instruction │  │   Address   │  │   Bus Control    │     │   │
│  │  │   Queue     │  │  Latches    │  │                  │     │   │
│  │  │  (6 bytes)  │  │             │  │                  │     │   │
│  │  └─────────────┘  └─────────────┘  └──────────────────┘     │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              ▲                                      │
│                              │                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                   Instruction Unit (IU)                      │   │
│  │  ┌─────────────────────────────────────────────────────┐    │   │
│  │  │              Instruction Decoder                     │    │   │
│  │  └─────────────────────────────────────────────────────┘    │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                   Execution Unit (EU)                        │   │
│  │  ┌───────────┐  ┌───────────┐  ┌──────────────────────┐     │   │
│  │  │ General   │  │   ALU     │  │  Control Unit        │     │   │
│  │  │ Registers │  │           │  │                      │     │   │
│  │  │ AX,BX,CX, │  │           │  │                      │     │   │
│  │  │ DX,SP,BP, │  │           │  │                      │     │   │
│  │  │ SI,DI     │  │           │  │                      │     │   │
│  │  └───────────┘  └───────────┘  └──────────────────────┘     │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                   Address Unit (AU)                          │   │
│  │  ┌───────────────┐  ┌────────────────────────────────┐      │   │
│  │  │ Segment       │  │   Protection Unit               │      │   │
│  │  │ Registers     │  │   - Descriptor Cache           │      │   │
│  │  │ CS,DS,ES,SS   │  │   - Limit/Access Checking      │      │   │
│  │  └───────────────┘  └────────────────────────────────┘      │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
                │                      │
                ▼                      ▼
        ┌───────────────┐      ┌───────────────┐
        │ 16-bit Data   │      │ 24-bit Address│
        │ Bus           │      │ Bus           │
        └───────────────┘      └───────────────┘
```

### 1.3 Register Set

```
80286 REGISTER SET
━━━━━━━━━━━━━━━━━━

GENERAL PURPOSE REGISTERS (Same as 8086):
────────────────────────────────────────

┌────────────────────────────────────────┐
│  AH        │  AL        │    AX       │ Accumulator
├────────────────────────────────────────┤
│  BH        │  BL        │    BX       │ Base
├────────────────────────────────────────┤
│  CH        │  CL        │    CX       │ Count
├────────────────────────────────────────┤
│  DH        │  DL        │    DX       │ Data
├────────────────────────────────────────┤
│            SP           │    SP       │ Stack Pointer
├────────────────────────────────────────┤
│            BP           │    BP       │ Base Pointer
├────────────────────────────────────────┤
│            SI           │    SI       │ Source Index
├────────────────────────────────────────┤
│            DI           │    DI       │ Destination Index
└────────────────────────────────────────┘

SEGMENT REGISTERS:
─────────────────

┌──────────────────────────────────────────────────────────────────┐
│  CS (16-bit) │ Code Segment Selector                             │
├──────────────────────────────────────────────────────────────────┤
│  DS (16-bit) │ Data Segment Selector                             │
├──────────────────────────────────────────────────────────────────┤
│  SS (16-bit) │ Stack Segment Selector                            │
├──────────────────────────────────────────────────────────────────┤
│  ES (16-bit) │ Extra Segment Selector                            │
└──────────────────────────────────────────────────────────────────┘

Each segment register has a hidden 48-bit descriptor cache!

SPECIAL REGISTERS:
─────────────────

┌──────────────────────────────────────────────────────────────────┐
│  IP (16-bit)  │ Instruction Pointer                              │
├──────────────────────────────────────────────────────────────────┤
│  FLAGS (16-bit)│ Status and Control Flags                        │
├──────────────────────────────────────────────────────────────────┤
│  MSW (16-bit) │ Machine Status Word (new in 286!)                │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Operating Modes

### 2.1 Real Mode

```
REAL MODE (8086 Compatible)
━━━━━━━━━━━━━━━━━━━━━━━━━━━

Characteristics:
───────────────
• Exactly like 8086 operation
• 1 MB address space (only A0-A19 used)
• No memory protection
• All code runs at privilege level 0
• Physical Address = Segment × 16 + Offset
• Default mode at power-up/reset

Addressing:
──────────

    Segment (16-bit)              Offset (16-bit)
    ┌──────────────────┐          ┌──────────────────┐
    │     XXXX         │          │      YYYY        │
    └──────────────────┘          └──────────────────┘
              │                            │
              │ × 16                       │
              │ (shift left 4)             │
              ▼                            ▼
    ┌──────────────────────┐    ┌──────────────────┐
    │    XXXX0             │ +  │      YYYY        │
    └──────────────────────┘    └──────────────────┘
              │                            │
              └──────────┬─────────────────┘
                         ▼
    ┌────────────────────────────────────────────┐
    │        20-bit Physical Address             │
    │              (1 MB max)                    │
    └────────────────────────────────────────────┘

Limitations:
───────────
• No multi-tasking protection
• Programs can corrupt each other
• No privilege levels
• Only 1 MB accessible (wastes 15 MB)
```

### 2.2 Protected Mode

```
PROTECTED MODE
━━━━━━━━━━━━━━

Key Features:
────────────
• Hardware memory protection
• 16 MB physical address space (24-bit)
• 1 GB virtual address space (30-bit)
• 4 privilege levels (rings 0-3)
• Segment descriptors define memory regions
• Suitable for multi-tasking OS

Entering Protected Mode:
──────────────────────

1. Disable interrupts (CLI)
2. Load GDT register (LGDT instruction)
3. Set PE (Protection Enable) bit in MSW
4. Jump to flush prefetch queue
5. Load segment registers with selectors
6. Enable interrupts (STI)

    ; Example: Switch to Protected Mode
    CLI                     ; Disable interrupts
    LGDT [GDT_Ptr]         ; Load GDT pointer
    MOV AX, MSW
    OR AX, 1               ; Set PE bit
    LMSW AX                ; Load MSW
    JMP FLUSH              ; Flush prefetch queue
FLUSH:
    MOV AX, DATA_SEL       ; Load data selector
    MOV DS, AX
    STI                    ; Enable interrupts

Exiting Protected Mode:
─────────────────────
• 80286 cannot exit protected mode in software!
• Requires CPU reset (keyboard controller reset)
• This limitation fixed in 80386
```

---

## 3. Memory Management

### 3.1 Segment Descriptors

```
SEGMENT DESCRIPTOR (8 bytes)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────────────────────────────────────────────────────────────────┐
│ Byte 7  │ Byte 6  │ Byte 5  │ Byte 4  │ Byte 3-2  │ Byte 1-0   │
├─────────┴─────────┼─────────┴─────────┼───────────┼────────────┤
│    Reserved       │   Access Rights   │Base(23:16)│Limit(15:0) │
│    (must be 0)    │                   │           │            │
└───────────────────┴───────────────────┴───────────┴────────────┘
│◄───── 16 bits ───►│◄───── 16 bits ───►│◄─8 bits ─►│◄─16 bits ─►│

Bytes 1-0: Segment Limit (0-15) - Size of segment - 1
Bytes 3-2: Base Address (0-15) - Lower 16 bits of base
Byte 4:    Base Address (16-23) - Upper 8 bits of base
Byte 5:    Access Rights byte
Bytes 6-7: Reserved for 80386 (must be 0 in 80286)

ACCESS RIGHTS BYTE:
─────────────────

Bit    7      6      5      4      3      2      1      0
    ┌──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┐
    │  P   │ DPL  │ DPL  │  S   │  E   │ED/C │ R/W  │  A   │
    └──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┘

P:    Present (1 = segment in memory)
DPL:  Descriptor Privilege Level (0-3)
S:    Segment type (1 = code/data, 0 = system)
E:    Executable (1 = code, 0 = data)
ED/C: Expand Down (data) / Conforming (code)
R/W:  Readable (code) / Writable (data)
A:    Accessed (set by CPU when accessed)

EXAMPLE DESCRIPTORS:
──────────────────

; Code Segment: Base=0x10000, Limit=0xFFFF, DPL=0
    DW 0FFFFh      ; Limit 15:0
    DW 0000h       ; Base 15:0
    DB 01h         ; Base 23:16
    DB 10011010b   ; P=1, DPL=00, S=1, E=1, C=0, R=1, A=0
    DW 0000h       ; Reserved

; Data Segment: Base=0x20000, Limit=0xFFFF, DPL=0
    DW 0FFFFh      ; Limit 15:0
    DW 0000h       ; Base 15:0
    DB 02h         ; Base 23:16
    DB 10010010b   ; P=1, DPL=00, S=1, E=0, ED=0, W=1, A=0
    DW 0000h       ; Reserved
```

### 3.2 Segment Selectors

```
SEGMENT SELECTOR FORMAT (16 bits)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Bit 15                                    3    2    1    0
┌──────────────────────────────────────┬─────┬─────┬─────┐
│             Index (13 bits)          │ TI  │ RPL │ RPL │
│        (Selects descriptor)          │     │     │     │
└──────────────────────────────────────┴─────┴─────┴─────┘

Index: Descriptor table index (0-8191)
TI:    Table Indicator (0 = GDT, 1 = LDT)
RPL:   Requested Privilege Level (0-3)

EXAMPLE SELECTORS:
─────────────────

Selector 0008h:
    Index = 1 (second descriptor in table)
    TI = 0 (GDT)
    RPL = 0 (Ring 0)

Selector 0023h:
    Index = 4 (fifth descriptor)
    TI = 0 (GDT)
    RPL = 3 (Ring 3)

DESCRIPTOR TABLES:
─────────────────

GDT (Global Descriptor Table):
• One per system
• Contains descriptors for all segments
• GDTR register points to GDT

LDT (Local Descriptor Table):
• One per task
• Task-specific segment descriptors
• LDTR register contains selector for current LDT

IDT (Interrupt Descriptor Table):
• Contains interrupt/trap gates
• IDTR register points to IDT
```

### 3.3 Address Translation

```
PROTECTED MODE ADDRESS TRANSLATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Logical Address → Physical Address

┌────────────────┬────────────────────────────────────────────┐
│  Selector      │              Offset                        │
│  (16 bits)     │             (16 bits)                      │
└───────┬────────┴──────────────────────────────────────────┬─┘
        │                                                    │
        ▼                                                    │
┌───────────────────────────────────────────────────┐       │
│          Descriptor Table (GDT/LDT)               │       │
│  ┌─────────────────────────────────────────────┐  │       │
│  │ Descriptor: Base Address (24-bit)           │  │       │
│  │             Limit (16-bit)                  │  │       │
│  │             Access Rights                   │  │       │
│  └─────────────────────────────────────────────┘  │       │
└───────────────────┬───────────────────────────────┘       │
                    │                                        │
                    ▼                                        │
        ┌───────────────────────┐                           │
        │  Base (24 bits)       │ + ◄───────────────────────┘
        └───────────────────────┘   Offset (after limit check)
                    │
                    ▼
        ┌───────────────────────────────────────────────┐
        │         24-bit Physical Address               │
        │              (0 - 16 MB)                      │
        └───────────────────────────────────────────────┘

PROTECTION CHECKS:
─────────────────
1. Segment Present (P=1)
2. Offset within Limit
3. Access type allowed (read/write/execute)
4. Privilege level sufficient (CPL ≤ DPL for data)
5. Segment type matches operation

If any check fails → Exception (Trap)
```

---

## 4. Privilege Levels

```
80286 PRIVILEGE LEVELS (PROTECTION RINGS)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

                    Ring 0 (Kernel)
                    Most Privileged
                    ┌───────────┐
                    │  OS Core  │
                    │  Drivers  │
                    ├───────────┤
                Ring 1 │         │
                    │  OS       │
                    │ Services  │
                    ├───────────┤
                Ring 2 │         │
                    │ Protected │
                    │ Utilities │
                    ├───────────┤
                Ring 3 │         │  Least Privileged
                    │  User     │
                    │  Apps     │
                    └───────────┘

PRIVILEGE TERMS:
───────────────
CPL: Current Privilege Level (in CS selector's RPL)
DPL: Descriptor Privilege Level (in segment descriptor)
RPL: Requested Privilege Level (in segment selector)

ACCESS RULES:
────────────

For Data Segments:
    Access allowed if: max(CPL, RPL) ≤ DPL
    (Can access same or less privileged data)

For Code Segments (call/jump):
    Direct transfer: CPL must equal DPL
    Through gate: CPL ≥ Gate DPL, target DPL ≤ CPL

TYPICAL USAGE:
─────────────
Ring 0: Operating System Kernel
Ring 1: Device Drivers (rarely used, often merged with Ring 0)
Ring 2: System Services (rarely used)
Ring 3: User Applications
```

---

## 5. System Registers

### 5.1 Machine Status Word (MSW)

```
MSW - MACHINE STATUS WORD
━━━━━━━━━━━━━━━━━━━━━━━━━

Bit 15                                    4    3    2    1    0
┌──────────────────────────────────────┬─────┬─────┬─────┬─────┐
│              Reserved                │ ET  │ TS  │ EM  │ PE  │
└──────────────────────────────────────┴─────┴─────┴─────┴─────┘

PE (bit 0): Protection Enable
    0 = Real Mode
    1 = Protected Mode
    (Cannot be cleared by software in 80286!)

MP (bit 1): Monitor Processor extension (coprocessor)
    Works with TS for WAIT instruction

EM (bit 2): Emulate coprocessor
    1 = Generate exception on ESC instructions

TS (bit 3): Task Switched
    Set on task switch, cleared by coprocessor instruction

Instructions:
    LMSW reg16    ; Load MSW
    SMSW reg16    ; Store MSW
```

### 5.2 Descriptor Table Registers

```
DESCRIPTOR TABLE REGISTERS
━━━━━━━━━━━━━━━━━━━━━━━━━━

GDTR (Global Descriptor Table Register) - 48 bits:
┌───────────────────────────────────────┬───────────────────┐
│        Base Address (24 bits)         │   Limit (16 bits) │
└───────────────────────────────────────┴───────────────────┘

IDTR (Interrupt Descriptor Table Register) - 48 bits:
┌───────────────────────────────────────┬───────────────────┐
│        Base Address (24 bits)         │   Limit (16 bits) │
└───────────────────────────────────────┴───────────────────┘

LDTR (Local Descriptor Table Register) - 16 bits:
┌───────────────────────────────────────┐
│      Selector (16 bits)               │
└───────────────────────────────────────┘
(Plus hidden 48-bit descriptor cache)

TR (Task Register) - 16 bits:
┌───────────────────────────────────────┐
│      Selector (16 bits)               │
└───────────────────────────────────────┘
(Plus hidden 48-bit descriptor cache for TSS)

Instructions:
    LGDT mem48    ; Load GDTR
    SGDT mem48    ; Store GDTR
    LIDT mem48    ; Load IDTR
    SIDT mem48    ; Store IDTR
    LLDT reg16    ; Load LDTR
    SLDT reg16    ; Store LDTR
    LTR reg16     ; Load Task Register
    STR reg16     ; Store Task Register
```

---

## 6. New Instructions

```
80286 NEW INSTRUCTIONS
━━━━━━━━━━━━━━━━━━━━━━

PROTECTED MODE INSTRUCTIONS:
───────────────────────────

LGDT mem48      ; Load Global Descriptor Table Register
SGDT mem48      ; Store GDTR
LIDT mem48      ; Load Interrupt Descriptor Table Register
SIDT mem48      ; Store IDTR
LLDT r/m16      ; Load Local Descriptor Table Register
SLDT r/m16      ; Store LDTR
LTR r/m16       ; Load Task Register
STR r/m16       ; Store Task Register
LMSW r/m16      ; Load Machine Status Word
SMSW r/m16      ; Store MSW

PROTECTION INSTRUCTIONS:
───────────────────────

LAR r16,r/m16   ; Load Access Rights
LSL r16,r/m16   ; Load Segment Limit
ARPL r/m16,r16  ; Adjust RPL (for OS use)
VERR r/m16      ; Verify segment for Read
VERW r/m16      ; Verify segment for Write

PRIVILEGED INSTRUCTIONS (Ring 0 only):
─────────────────────────────────────

CLTS            ; Clear Task Switched flag
HLT             ; Halt processor
LGDT, LIDT, LLDT, LTR, LMSW   ; All load operations

NEW GENERAL INSTRUCTIONS:
────────────────────────

PUSHA           ; Push all general registers
POPA            ; Pop all general registers
BOUND r16,m32   ; Check array bounds
ENTER imm16,imm8; Build stack frame
LEAVE           ; Destroy stack frame
INS/INSB/INSW   ; Input string from port
OUTS/OUTSB/OUTSW; Output string to port
```

---

## 📋 Summary Table

| Feature | 8086 | 80286 |
|---------|------|-------|
| Address Bus | 20 bits | 24 bits |
| Physical Memory | 1 MB | 16 MB |
| Virtual Memory | No | 1 GB |
| Protection | No | Yes |
| Privilege Levels | 1 | 4 |
| Segment Limit | 64 KB | 64 KB |
| Modes | Real | Real + Protected |

---

## ❓ Quick Revision Questions

1. **How do you switch from Real Mode to Protected Mode?**
   <details>
   <summary>Show Answer</summary>
   1. Disable interrupts (CLI), 2. Load GDT with LGDT, 3. Set PE bit in MSW using LMSW, 4. Far jump to flush prefetch queue, 5. Load segment registers with valid selectors.
   </details>

2. **What is the purpose of segment descriptors?**
   <details>
   <summary>Show Answer</summary>
   Descriptors define segment properties: base address (where it starts), limit (size), access rights (read/write/execute), and privilege level. The CPU uses them to translate logical to physical addresses and enforce protection.
   </details>

3. **Why can't 80286 return to Real Mode by software?**
   <details>
   <summary>Show Answer</summary>
   The PE (Protection Enable) bit in MSW cannot be cleared by software. A CPU reset is required. This was a design limitation fixed in 80386 with a clear mechanism.
   </details>

4. **What is the difference between GDT and LDT?**
   <details>
   <summary>Show Answer</summary>
   GDT is global (one per system) containing shared segments like kernel code. LDT is local (one per task) containing task-specific segments. This allows memory isolation between tasks.
   </details>

5. **How does privilege checking work for data access?**
   <details>
   <summary>Show Answer</summary>
   Access is allowed if max(CPL, RPL) ≤ DPL. CPL is current privilege, RPL is requested privilege in selector, DPL is descriptor privilege. Lower numbers = higher privilege. Ring 0 can access Ring 3 data.
   </details>

6. **What are the new instructions added in 80286 for stack frames?**
   <details>
   <summary>Show Answer</summary>
   ENTER imm16,imm8 creates a stack frame (push BP, copy SP to BP, allocate local space). LEAVE reverses it (mov SP,BP; pop BP). These optimize high-level language function prologue/epilogue.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Unit 9 Index](README.md) | [Unit 9 Index](README.md) | [9.2 80386 Processor](02-80386-processor.md) |

---

*[← Back to Unit 9 Index](README.md) | [Next: 80386 Processor →](02-80386-processor.md)*
