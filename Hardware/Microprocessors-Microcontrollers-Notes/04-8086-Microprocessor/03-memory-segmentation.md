# Chapter 4.3: Memory Segmentation

## 📚 Chapter Overview

The 8086 uses a unique segmented memory architecture to address 1 MB of memory using 16-bit registers. This chapter explains the segmentation concept, physical address calculation, and segment organization.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Explain why segmentation is needed in 8086
- Calculate physical addresses from segment:offset
- Understand overlapping segments
- Describe memory organization and types
- Work with segment:offset notation

---

## 1. Why Segmentation?

### 1.1 The Addressing Problem

```
THE 8086 ADDRESSING CHALLENGE:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   PROBLEM:                                                       │
│   • 8086 has 20-bit address bus (A0-A19)                        │
│   • Can address 2²⁰ = 1,048,576 bytes = 1 MB                    │
│   • But all registers are only 16-bit                           │
│   • 16-bit register can only hold 2¹⁶ = 65,536 = 64 KB         │
│                                                                  │
│   SOLUTION: SEGMENTATION                                         │
│   • Divide memory into 64 KB segments                           │
│   • Use two 16-bit registers to form 20-bit address             │
│   • Segment Register: Base address (shifted)                    │
│   • Offset Register: Location within segment                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

COMPARISON:

┌─────────────────────────┬─────────────────────────────────────────┐
│   Linear Addressing     │   Segmented Addressing                  │
├─────────────────────────┼─────────────────────────────────────────┤
│                         │                                         │
│   20-bit address        │   16-bit Segment + 16-bit Offset       │
│       │                 │       │              │                  │
│       ▼                 │       ▼              ▼                  │
│   ┌───────┐             │   ┌───────┐    ┌───────┐               │
│   │1234567│             │   │ 1234H │ :  │ 5678H │               │
│   └───────┘             │   └───────┘    └───────┘               │
│       │                 │       │              │                  │
│   Needs 20-bit          │   Both fit in 16-bit registers        │
│   register!             │   ✓                                    │
│                         │                                         │
└─────────────────────────┴─────────────────────────────────────────┘
```

### 1.2 Segment Concept

```
MEMORY SEGMENTATION MODEL:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│                     1 MB MEMORY SPACE                           │
│                                                                  │
│    FFFFFH ┌─────────────────────────────────────────────┐       │
│           │                                             │       │
│           │                                             │       │
│           │          (Available Memory)                 │       │
│           │                                             │       │
│           │                                             │       │
│    A0000H ├─────────────────────────────────────────────┤       │
│           │ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ │       │
│           │ ▓        SEGMENT (64 KB)                 ▓ │       │
│           │ ▓   Can be anywhere in 1 MB space        ▓ │       │
│           │ ▓   Base = Segment Reg × 16              ▓ │       │
│    90000H │ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ │       │
│           │              ↑                             │       │
│           │        Segment starts at                   │       │
│           │        paragraph boundary                  │       │
│           │        (address divisible by 16)           │       │
│           │                                             │       │
│    00000H └─────────────────────────────────────────────┘       │
│                                                                  │
│   SEGMENT PROPERTIES:                                           │
│   • Maximum size: 64 KB (0000H to FFFFH offset)                │
│   • Must start on paragraph boundary (address mod 16 = 0)      │
│   • Segments can overlap                                        │
│   • Multiple segments can be active simultaneously             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Physical Address Calculation

### 2.1 The Formula

```
PHYSICAL ADDRESS CALCULATION:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Physical Address = (Segment × 16) + Offset                    │
│                                                                  │
│   OR equivalently:                                               │
│                                                                  │
│   Physical Address = (Segment × 10H) + Offset                   │
│                                                                  │
│   OR using shift:                                                │
│                                                                  │
│   Physical Address = (Segment << 4) + Offset                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

VISUAL REPRESENTATION:

    Segment Register:      XXXX H (16 bits)
                              ↓
    Shift left 4 bits:    XXXX0 H (20 bits)
                              +
    Offset Register:       YYYY H (16 bits)
                              =
    Physical Address:     ZZZZZ H (20 bits)


EXAMPLE 1: CS:IP = 1234H:5678H
───────────────────────────────────────────────────────────────────

    Segment (CS):    1234 H
    Shift left 4:   12340 H
                        +
    Offset (IP):     5678 H
                        =
    Physical:       179B8 H

    Calculation:
    12340H + 5678H = 179B8H ✓


EXAMPLE 2: DS:BX = 2000H:0100H
───────────────────────────────────────────────────────────────────

    Segment (DS):    2000 H
    Shift left 4:   20000 H
                        +
    Offset (BX):     0100 H
                        =
    Physical:       20100 H
```

### 2.2 Address Calculation Hardware

```
8086 ADDRESS ADDER (in BIU):

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   ┌────────────────┐                                            │
│   │ Segment Reg    │  16 bits                                   │
│   │  (CS/DS/SS/ES) │────────────┐                               │
│   └────────────────┘            │                               │
│                                 ▼                               │
│                          ┌────────────┐                         │
│                          │  × 16      │  (Shift left 4)         │
│                          │ (or << 4)  │                         │
│                          └────────────┘                         │
│                                 │  20 bits                      │
│                                 ▼                               │
│                          ┌────────────┐                         │
│   ┌────────────────┐     │            │                         │
│   │ Offset Reg     │────►│   ADDER    │──────► 20-bit Physical  │
│   │(IP/BX/SI/DI/SP)│     │   (Σ)      │        Address          │
│   └────────────────┘     └────────────┘                         │
│        16 bits                                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

This calculation happens automatically for every memory access!
```

### 2.3 Practice Examples

```
ADDRESS CALCULATION PRACTICE:

┌────────────────────┬──────────────────────────────────────────┐
│   Segment:Offset   │   Physical Address Calculation           │
├────────────────────┼──────────────────────────────────────────┤
│   1000H:0000H      │   10000H + 0000H = 10000H                │
│   1000H:FFFFH      │   10000H + FFFFH = 1FFFFH                │
│   0000H:1234H      │   00000H + 1234H = 01234H                │
│   FFFFH:0000H      │   FFFF0H + 0000H = FFFF0H                │
│   FFFFH:000FH      │   FFFF0H + 000FH = FFFFFH (max address)  │
│   FFFFH:0010H      │   FFFF0H + 0010H = 100000H (wraps!)      │
│   2345H:6789H      │   23450H + 6789H = 29BD9H                │
│   0050H:0100H      │   00500H + 0100H = 00600H                │
└────────────────────┴──────────────────────────────────────────┘

WRAP-AROUND (High Memory Area):
───────────────────────────────────────────────────────────────────

If Physical Address > FFFFFH, it wraps around:
FFFFH:0010H → FFFF0H + 0010H = 100000H → 00000H (wraps to 0)

This is the basis of the famous "A20 gate" issue in PC history!
```

---

## 3. Overlapping Segments

### 3.1 Same Physical Address, Different Segments

```
OVERLAPPING SEGMENTS CONCEPT:

A single physical address can be represented by MANY segment:offset pairs!

EXAMPLE: Physical address 12345H
───────────────────────────────────────────────────────────────────

All of these are equivalent:
• 1234H:0005H → 12340H + 0005H = 12345H ✓
• 1230H:0045H → 12300H + 0045H = 12345H ✓
• 1200H:0345H → 12000H + 0345H = 12345H ✓
• 1000H:2345H → 10000H + 2345H = 12345H ✓
• 0000H:12345H → Invalid! (offset > FFFFH)
• 1234H:0005H → 12340H + 0005H = 12345H ✓


VISUALIZATION:
───────────────────────────────────────────────────────────────────

Memory:     │     │     │     │     │     │     │
       ─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────
            │     │  ★  │     │     │     │     │
            │     │  ↑  │     │     │     │     │
                  12345H

Segment 1234H: [─────|─────|★────|─────|─────|─────]
               0000H      0005H

Segment 1230H: [─────|─────|─|★──|─────|─────|─────]
               0000H        0045H

Segment 1200H: [─────|─────|───|──★─|─────|─────]
               0000H            0345H

All three segments contain the same physical location!
```

### 3.2 Segment Arrangement Strategies

```
SEGMENT ARRANGEMENT OPTIONS:

1. NON-OVERLAPPING SEGMENTS (Separate regions):
───────────────────────────────────────────────────────────────────

    ┌────────────────────────────────────────────────────────────┐
    │                                                            │
    │    ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐    │
    │    │  CODE    │ │  DATA    │ │  STACK   │ │  EXTRA   │    │
    │    │   (CS)   │ │   (DS)   │ │   (SS)   │ │   (ES)   │    │
    │    └──────────┘ └──────────┘ └──────────┘ └──────────┘    │
    │    0000H      64KB       128KB      192KB       256KB      │
    │                                                            │
    │    Each segment gets full 64 KB, no overlap                │
    │                                                            │
    └────────────────────────────────────────────────────────────┘


2. FULLY OVERLAPPING SEGMENTS (Small programs):
───────────────────────────────────────────────────────────────────

    ┌────────────────────────────────────────────────────────────┐
    │                                                            │
    │    CS = DS = SS = ES = 1000H                               │
    │                                                            │
    │    ┌────────────────────────────────────────────┐          │
    │    │ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓│          │
    │    │   Code, Data, Stack all in same 64KB       │          │
    │    │ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓│          │
    │    └────────────────────────────────────────────┘          │
    │    10000H                                      1FFFFH      │
    │                                                            │
    │    .COM programs use this model                            │
    │                                                            │
    └────────────────────────────────────────────────────────────┘


3. PARTIALLY OVERLAPPING SEGMENTS:
───────────────────────────────────────────────────────────────────

    ┌────────────────────────────────────────────────────────────┐
    │                                                            │
    │         CS=1000H      DS=1800H      SS=2000H              │
    │            │             │             │                   │
    │            ▼             ▼             ▼                   │
    │    ┌──────────────────────────────────────────────────┐   │
    │    │░░░░░░░░░░░░░░░░░│                                │   │
    │    │    CODE         │▓▓▓▓▓▓▓▓▓▓▓▓▓│                  │   │
    │    │                 │    DATA     │████████████████│ │   │
    │    │                 │             │     STACK      │ │   │
    │    └──────────────────────────────────────────────────┘   │
    │                                                            │
    │    Segments overlap, saving memory for small sections     │
    │                                                            │
    └────────────────────────────────────────────────────────────┘
```

---

## 4. Segment Types and Usage

### 4.1 Default Segment Associations

```
DEFAULT SEGMENT USAGE:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   INSTRUCTION FETCH:                                            │
│   • Segment: CS (Code Segment)                                  │
│   • Offset: IP (Instruction Pointer)                            │
│   • Physical = CS × 16 + IP                                     │
│                                                                  │
│   STACK OPERATIONS:                                              │
│   • Segment: SS (Stack Segment)                                 │
│   • Offset: SP (Stack Pointer) or BP (Base Pointer)             │
│   • Physical = SS × 16 + SP/BP                                  │
│   • PUSH, POP, CALL, RET, INT use SS:SP                        │
│                                                                  │
│   DATA ACCESS:                                                   │
│   • Segment: DS (Data Segment) - default                        │
│   • Offset: BX, SI, DI, or direct address                       │
│   • Physical = DS × 16 + offset                                 │
│                                                                  │
│   STRING DESTINATION:                                            │
│   • Segment: ES (Extra Segment)                                 │
│   • Offset: DI (Destination Index)                              │
│   • Physical = ES × 16 + DI                                     │
│   • MOVS, STOS, CMPS, SCAS destinations                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

SEGMENT ASSOCIATION TABLE:

┌─────────────────────┬──────────────┬────────────────────────────┐
│   Offset Register   │   Default    │   Alternate (Override)     │
│                     │   Segment    │                            │
├─────────────────────┼──────────────┼────────────────────────────┤
│   IP                │   CS         │   None (always CS)         │
│   SP                │   SS         │   None (always SS)         │
│   BP                │   SS         │   CS, DS, ES               │
│   BX                │   DS         │   CS, SS, ES               │
│   SI                │   DS         │   CS, SS, ES               │
│   DI                │   DS         │   CS, SS, ES               │
│   DI (string dest)  │   ES         │   None (always ES)         │
└─────────────────────┴──────────────┴────────────────────────────┘
```

### 4.2 Segment Override Examples

```
SEGMENT OVERRIDE SYNTAX:

; Default segments
MOV AX, [BX]        ; DS:BX
MOV AX, [BP]        ; SS:BP
MOV AX, [SI]        ; DS:SI

; With override
MOV AX, ES:[BX]     ; ES:BX (override DS with ES)
MOV AX, CS:[SI]     ; CS:SI (read from code segment)
MOV AX, SS:[BX]     ; SS:BX (read from stack segment)
MOV AX, DS:[BP]     ; DS:BP (override SS with DS)


OVERRIDE EXAMPLE PROGRAM:
───────────────────────────────────────────────────────────────────

; Copy data from code segment to data segment
; (Self-modifying code or reading embedded data)

CODE_SEGMENT SEGMENT
    JMP START
    DATA_IN_CODE DB 'Hello', 0    ; Data embedded in code
    
START:
    MOV AX, CODE_SEGMENT
    MOV DS, AX                    ; Set up DS = CS
    
    ; Alternative: use segment override
    MOV SI, OFFSET DATA_IN_CODE
    MOV AL, CS:[SI]               ; Read from CS, not DS
    
CODE_SEGMENT ENDS
```

---

## 5. Memory Map

### 5.1 8086 Memory Organization

```
8086 MEMORY MAP (1 MB):

     Physical
     Address
     
     FFFFFH  ┌───────────────────────────────────────────────┐
             │                                               │
     F0000H  │         ROM BIOS (64 KB)                     │
             │         System startup code                  │
             │                                               │
     E0000H  ├───────────────────────────────────────────────┤
             │                                               │
             │         ROM Extensions                        │
             │         Video BIOS, etc.                      │
             │                                               │
     C0000H  ├───────────────────────────────────────────────┤
             │                                               │
     B8000H  │         Video Memory (Text)                  │
             │         CGA/EGA/VGA Text Mode                │
             │                                               │
     B0000H  ├───────────────────────────────────────────────┤
             │                                               │
     A0000H  │         Video Memory (Graphics)              │
             │         EGA/VGA Graphics Mode                │
             │                                               │
     9FFFFH  ├───────────────────────────────────────────────┤
             │                                               │
             │                                               │
             │         Conventional Memory                   │
             │         (640 KB for user programs)            │
             │                                               │
             │         • Application programs               │
             │         • DOS                                │
             │         • Device drivers                     │
             │                                               │
     00500H  ├───────────────────────────────────────────────┤
             │         BIOS Data Area (256 bytes)           │
     00400H  ├───────────────────────────────────────────────┤
             │                                               │
             │         Interrupt Vector Table               │
             │         256 vectors × 4 bytes = 1 KB         │
             │         (CS:IP pairs for ISRs)               │
             │                                               │
     00000H  └───────────────────────────────────────────────┘
```

### 5.2 Interrupt Vector Table

```
INTERRUPT VECTOR TABLE (IVT):

Located at 00000H - 003FFH (first 1 KB of memory)

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Address      Vector    Content                                 │
│   ─────────────────────────────────────                         │
│   003FCH-003FFH  255     IP, CS for INT 255                    │
│       ...         ...                                           │
│   00014H-00017H   5      IP, CS for INT 5 (Print Screen)       │
│   00010H-00013H   4      IP, CS for INT 4 (Overflow)           │
│   0000CH-0000FH   3      IP, CS for INT 3 (Breakpoint)         │
│   00008H-0000BH   2      IP, CS for INT 2 (NMI)                │
│   00004H-00007H   1      IP, CS for INT 1 (Single Step)        │
│   00000H-00003H   0      IP, CS for INT 0 (Divide by Zero)     │
│                                                                  │
│   EACH VECTOR = 4 BYTES:                                        │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Offset (IP)   │  Segment (CS)   │                      │   │
│   │   Low  │ High  │   Low   │ High  │                      │   │
│   │ Byte 0│ Byte 1│ Byte 2  │ Byte 3│                      │   │
│   └───────┴───────┴─────────┴───────┘                       │   │
│   Little endian: Low byte first                              │   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

EXAMPLE: INT 0 Vector at 00000H
───────────────────────────────────────────────────────────────────

If ISR for divide-by-zero is at 1234H:5678H:
  00000H: 78H  (IP low)
  00001H: 56H  (IP high)
  00002H: 34H  (CS low)
  00003H: 12H  (CS high)

Reading: IP = 5678H, CS = 1234H
```

---

## 6. Logical vs Physical Addressing

### 6.1 Address Types

```
ADDRESS TYPES IN 8086:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   LOGICAL ADDRESS (Segment:Offset):                             │
│   • Format: XXXX:YYYY                                           │
│   • Two 16-bit values                                           │
│   • What programmer writes                                      │
│   • Example: 1234H:5678H                                        │
│                                                                  │
│   PHYSICAL ADDRESS (Linear):                                    │
│   • Format: ZZZZZ (20-bit)                                      │
│   • Actual memory location                                      │
│   • What appears on address bus                                 │
│   • Example: 179B8H                                             │
│                                                                  │
│   EFFECTIVE ADDRESS (Offset only):                              │
│   • Format: YYYY (16-bit)                                       │
│   • Calculated from addressing mode                             │
│   • Combined with segment for physical                          │
│   • Example: [BX+SI+10H] → BX + SI + 10H                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

CONVERSION FLOW:

Addressing Mode → Effective Address → + Segment → Physical Address
     [BX+SI]    →      1234H       → + DS×16   →    21234H
```

### 6.2 Advantages and Disadvantages

```
SEGMENTED MEMORY: PROS AND CONS

ADVANTAGES:
───────────────────────────────────────────────────────────────────

1. Access to 1 MB with 16-bit registers
   • Without segmentation: max 64 KB
   • With segmentation: 1 MB addressable

2. Memory protection (in theory)
   • Segments can be isolated
   • Code/data separation

3. Relocatable programs
   • Program can run at different addresses
   • Just change segment registers
   • Offset within segment stays same

4. Logical organization
   • Separate code, data, stack
   • Cleaner program structure


DISADVANTAGES:
───────────────────────────────────────────────────────────────────

1. Complex addressing
   • Two values needed for each address
   • Mental overhead for programmer

2. 64 KB segment limit
   • Single data structure limited to 64 KB
   • "Near" vs "Far" pointer issues

3. Overlapping causes confusion
   • Same location, multiple addresses
   • Debugging difficulty

4. Performance overhead
   • Segment register loads take time
   • Far calls/jumps slower than near

5. Limited protection
   • No hardware-enforced boundaries
   • Any code can access any segment
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Physical Address Formula | Segment × 16 + Offset |
| Segment Size | Max 64 KB (0000H - FFFFH) |
| Address Space | 1 MB (00000H - FFFFFH) |
| Paragraph | 16 bytes (segment must start on paragraph boundary) |
| CS:IP | Code location (instruction fetch) |
| DS:offset | Data access (default) |
| SS:SP/BP | Stack access |
| ES:DI | String destination |

---

## ❓ Quick Revision Questions

1. **Calculate physical address for 2000H:3000H**
   <details>
   <summary>Show Answer</summary>
   Physical = 2000H × 16 + 3000H = 20000H + 3000H = 23000H
   </details>

2. **Why can't 8086 directly address 1 MB with a single register?**
   <details>
   <summary>Show Answer</summary>
   All 8086 registers are 16-bit, which can only hold values 0-65535 (64 KB range). To address 1 MB (2²⁰ bytes), you need 20 bits. Segmentation combines two 16-bit values to create a 20-bit address.
   </details>

3. **Can two different segment:offset pairs point to the same physical address?**
   <details>
   <summary>Show Answer</summary>
   Yes! For example, 1234H:0005H and 1230H:0045H both equal 12345H. There are many segment:offset combinations that produce the same physical address due to the overlapping nature of segmentation.
   </details>

4. **What is the maximum physical address in 8086?**
   <details>
   <summary>Show Answer</summary>
   FFFFFH (1,048,575 or 1 MB - 1 byte). This is achieved with segment:offset like FFFFH:000FH = FFFF0H + 000FH = FFFFFH. Actually, FFFFH:FFFFH would give 10FFEFH which wraps around.
   </details>

5. **What is a paragraph in 8086 memory?**
   <details>
   <summary>Show Answer</summary>
   A paragraph is 16 bytes. Segments must start on paragraph boundaries (addresses divisible by 16, i.e., ending in 0H). This is because the segment register is shifted left by 4 bits (×16).
   </details>

6. **Why is [BP] associated with SS while [BX] is associated with DS?**
   <details>
   <summary>Show Answer</summary>
   BP (Base Pointer) is designed for accessing stack frames - parameters and local variables. Since stack is in SS, BP defaults to SS. BX is for general data access, which is typically in DS. This design matches common programming patterns.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [4.2 Register Organization](02-register-organization.md) | [Unit 4 Index](README.md) | [4.4 Addressing Modes](04-addressing-modes.md) |

---

*[← Previous: Register Organization](02-register-organization.md) | [Next: Addressing Modes →](04-addressing-modes.md)*
