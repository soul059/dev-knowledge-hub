# Chapter 4.4: 8086 Addressing Modes

## 📚 Chapter Overview

Addressing modes determine how operands are specified in instructions. The 8086 has a rich set of addressing modes that provide flexibility in accessing registers, immediate data, and memory locations.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Identify all 8086 addressing modes
- Calculate effective addresses
- Choose appropriate addressing modes for different tasks
- Understand addressing mode encoding
- Apply addressing modes in programming

---

## 1. Addressing Mode Categories

### 1.1 Overview

```
8086 ADDRESSING MODES:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   REGISTER ADDRESSING                                           │
│   • Operand is in a register                                    │
│   • Example: MOV AX, BX                                         │
│                                                                  │
│   IMMEDIATE ADDRESSING                                          │
│   • Operand is part of instruction                              │
│   • Example: MOV AX, 1234H                                      │
│                                                                  │
│   MEMORY ADDRESSING                                              │
│   • Operand is in memory                                        │
│   • Multiple sub-modes for different access patterns            │
│                                                                  │
│      ├── Direct Addressing                                      │
│      │      MOV AX, [1234H]                                     │
│      │                                                          │
│      ├── Register Indirect                                      │
│      │      MOV AX, [BX]                                        │
│      │                                                          │
│      ├── Based Addressing                                       │
│      │      MOV AX, [BX+10H]                                    │
│      │                                                          │
│      ├── Indexed Addressing                                     │
│      │      MOV AX, [SI+10H]                                    │
│      │                                                          │
│      └── Based-Indexed Addressing                               │
│             MOV AX, [BX+SI]                                     │
│             MOV AX, [BX+SI+10H]                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Register Addressing

```
REGISTER ADDRESSING MODE:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Both operands are registers                                   │
│                                                                  │
│   SYNTAX:    instruction reg, reg                               │
│                                                                  │
│   EXAMPLES:                                                      │
│   MOV AX, BX      ; AX ← BX (16-bit)                            │
│   MOV AL, BL      ; AL ← BL (8-bit)                             │
│   ADD CX, DX      ; CX ← CX + DX                                │
│   XCHG AX, BX     ; Exchange AX and BX                          │
│                                                                  │
│   CHARACTERISTICS:                                               │
│   • Fastest addressing mode                                     │
│   • No memory access needed                                     │
│   • Smallest instruction size                                   │
│   • Both 8-bit and 16-bit registers allowed                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

REGISTER ENCODING:

┌────────────────────────────────────────────────────────────────┐
│                                                                 │
│   16-bit Registers:         8-bit Registers:                   │
│   ┌──────┬────────┐         ┌──────┬────────┐                  │
│   │ Code │Register│         │ Code │Register│                  │
│   ├──────┼────────┤         ├──────┼────────┤                  │
│   │ 000  │   AX   │         │ 000  │   AL   │                  │
│   │ 001  │   CX   │         │ 001  │   CL   │                  │
│   │ 010  │   DX   │         │ 010  │   DL   │                  │
│   │ 011  │   BX   │         │ 011  │   BL   │                  │
│   │ 100  │   SP   │         │ 100  │   AH   │                  │
│   │ 101  │   BP   │         │ 101  │   CH   │                  │
│   │ 110  │   SI   │         │ 110  │   DH   │                  │
│   │ 111  │   DI   │         │ 111  │   BH   │                  │
│   └──────┴────────┘         └──────┴────────┘                  │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

---

## 3. Immediate Addressing

```
IMMEDIATE ADDRESSING MODE:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Operand is a constant value embedded in instruction           │
│                                                                  │
│   SYNTAX:    instruction reg, immediate                         │
│              instruction mem, immediate                         │
│                                                                  │
│   EXAMPLES:                                                      │
│   MOV AX, 1234H     ; AX ← 1234H                                │
│   MOV AL, 56H       ; AL ← 56H                                  │
│   ADD BX, 100       ; BX ← BX + 100                             │
│   CMP CX, 0         ; Compare CX with 0                         │
│   MOV [2000H], 55H  ; Memory ← 55H                              │
│                                                                  │
│   CHARACTERISTICS:                                               │
│   • Data is part of instruction                                 │
│   • Cannot be destination                                       │
│   • Increases instruction size                                  │
│   • Good for constants and initialization                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

INSTRUCTION ENCODING EXAMPLE:

MOV AX, 1234H:
┌────────────────────────────────────────────────────────────────┐
│                                                                 │
│   B8 34 12                                                     │
│   ↑  ↑  ↑                                                      │
│   │  │  └── High byte of immediate (12H)                       │
│   │  └───── Low byte of immediate (34H)                        │
│   └──────── Opcode (MOV AX, imm16)                             │
│                                                                 │
│   Note: Little endian - low byte first in memory               │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

---

## 4. Direct Addressing

```
DIRECT ADDRESSING MODE:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Memory address is directly specified in instruction           │
│                                                                  │
│   SYNTAX:    instruction reg, [address]                         │
│              instruction [address], reg                         │
│                                                                  │
│   EXAMPLES:                                                      │
│   MOV AX, [1234H]     ; AX ← DS:[1234H]                         │
│   MOV [2000H], BX     ; DS:[2000H] ← BX                         │
│   ADD AL, [5000H]     ; AL ← AL + DS:[5000H]                    │
│                                                                  │
│   EFFECTIVE ADDRESS = Displacement (direct value)               │
│   PHYSICAL ADDRESS = DS × 16 + Displacement                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

VISUALIZATION:

    Instruction: MOV AX, [1234H]
    
    ┌──────────────────────┐
    │   MOV AX, [1234H]    │  Instruction in memory
    └──────────────────────┘
              │
              │  Address = 1234H (from instruction)
              ▼
    ┌─────────────────────────────────────────────────┐
    │                     MEMORY                       │
    │                                                 │
    │    Address    Content                           │
    │    1233H      xx                                │
    │    1234H      78  ←── Low byte of word         │
    │    1235H      56  ←── High byte of word        │
    │    1236H      xx                                │
    │                                                 │
    │    Result: AX = 5678H                          │
    └─────────────────────────────────────────────────┘

USING LABELS:

DATA_SEGMENT SEGMENT
    MYDATA DW 1234H     ; Define word at MYDATA
DATA_SEGMENT ENDS

CODE_SEGMENT SEGMENT
    MOV AX, MYDATA      ; Same as MOV AX, [offset of MYDATA]
    MOV AX, [MYDATA]    ; Explicit bracket notation
CODE_SEGMENT ENDS
```

---

## 5. Register Indirect Addressing

```
REGISTER INDIRECT ADDRESSING:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Address is in a register (BX, SI, DI, or BP)                  │
│                                                                  │
│   SYNTAX:    instruction reg, [base_reg]                        │
│                                                                  │
│   ALLOWED REGISTERS:                                             │
│   • [BX]  - Base register, default segment DS                   │
│   • [SI]  - Source index, default segment DS                    │
│   • [DI]  - Destination index, default segment DS               │
│   • [BP]  - Base pointer, default segment SS                    │
│                                                                  │
│   NOT ALLOWED: [AX], [CX], [DX], [SP]                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

EXAMPLES:

; Using BX
MOV BX, 1000H       ; BX = 1000H (pointer)
MOV AX, [BX]        ; AX ← word at DS:1000H

; Using SI
MOV SI, 2000H       ; SI = 2000H
MOV AL, [SI]        ; AL ← byte at DS:2000H

; Using DI  
MOV DI, 3000H       ; DI = 3000H
MOV [DI], CX        ; Word at DS:3000H ← CX

; Using BP (defaults to SS)
MOV BP, SP          ; BP = stack pointer
MOV AX, [BP]        ; AX ← word at SS:BP


VISUALIZATION:

    MOV AX, [BX]    where BX = 2000H
    
         BX
    ┌─────────┐
    │  2000H  │──────────────┐
    └─────────┘              │
                             ▼
    ┌───────────────────────────────────────────┐
    │               MEMORY                       │
    │    2000H: │ 34 │ 12 │                     │
    │           └────┴────┘                     │
    │            Low  High                      │
    │                                           │
    │    Result: AX = 1234H                    │
    └───────────────────────────────────────────┘
```

---

## 6. Based Addressing

```
BASED ADDRESSING MODE:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Address = Base Register + Displacement                        │
│                                                                  │
│   SYNTAX:    instruction reg, [base + disp]                     │
│              instruction reg, [base].disp                       │
│              instruction reg, disp[base]                        │
│                                                                  │
│   BASE REGISTERS: BX (default DS), BP (default SS)              │
│                                                                  │
│   EXAMPLES:                                                      │
│   MOV AX, [BX+10H]      ; AX ← DS:[BX+10H]                      │
│   MOV AX, [BP+4]        ; AX ← SS:[BP+4]                        │
│   MOV AX, 10H[BX]       ; Same as [BX+10H]                      │
│   MOV AX, [BX].FIELD    ; Structure field access                │
│                                                                  │
│   EFFECTIVE ADDRESS = Base Register + Displacement              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

USE CASE: STRUCTURE ACCESS

; Define a structure
STUDENT STRUC
    ID      DW ?        ; Offset 0
    AGE     DB ?        ; Offset 2
    GRADE   DB ?        ; Offset 3
    NAME    DB 20 DUP(?); Offset 4
STUDENT ENDS

; Access structure fields
MOV BX, OFFSET STUDENT1     ; BX points to structure
MOV AX, [BX+0]              ; AX = ID
MOV AL, [BX+2]              ; AL = AGE
MOV AL, [BX+3]              ; AL = GRADE


VISUALIZATION:

    MOV AX, [BX+10H]    where BX = 2000H
    
         BX
    ┌─────────┐
    │  2000H  │
    └─────────┘
         │
         + 10H (displacement)
         │
         = 2010H (effective address)
              │
              ▼
    ┌───────────────────────────────────────────┐
    │               MEMORY                       │
    │    2010H: │ 78 │ 56 │                     │
    │           └────┴────┘                     │
    │                                           │
    │    Result: AX = 5678H                    │
    └───────────────────────────────────────────┘
```

---

## 7. Indexed Addressing

```
INDEXED ADDRESSING MODE:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Address = Index Register + Displacement                       │
│                                                                  │
│   SYNTAX:    instruction reg, [index + disp]                    │
│              instruction reg, disp[index]                       │
│                                                                  │
│   INDEX REGISTERS: SI, DI (default segment DS)                  │
│                                                                  │
│   EXAMPLES:                                                      │
│   MOV AX, [SI+10H]      ; AX ← DS:[SI+10H]                      │
│   MOV AX, [DI+20H]      ; AX ← DS:[DI+20H]                      │
│   MOV AX, ARRAY[SI]     ; AX ← DS:[ARRAY+SI]                    │
│   MOV AX, TABLE[DI]     ; AX ← DS:[TABLE+DI]                    │
│                                                                  │
│   EFFECTIVE ADDRESS = Index Register + Displacement             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

USE CASE: ARRAY ACCESS

ARRAY DW 10, 20, 30, 40, 50    ; Array of 5 words

; Access ARRAY[2] (third element)
MOV SI, 4               ; Index = 2 × 2 (word size)
MOV AX, ARRAY[SI]       ; AX = 30

; Loop through array
MOV SI, 0               ; Start at index 0
MOV CX, 5               ; 5 elements
LOOP_START:
    MOV AX, ARRAY[SI]   ; Get element
    ; Process AX...
    ADD SI, 2           ; Next word (SI += 2)
    LOOP LOOP_START


VISUALIZATION:

    MOV AX, ARRAY[SI]   where SI = 4, ARRAY at 2000H
    
    ARRAY offset = 2000H
         SI
    ┌─────────┐
    │    4    │
    └─────────┘
         │
    2000H + 4 = 2004H
              │
              ▼
    ┌───────────────────────────────────────────────────┐
    │               MEMORY                               │
    │    2000H: │ 0A 00 │ 14 00 │ 1E 00 │ 28 00 │ ...  │
    │           └───────┴───────┴───────┴───────┘      │
    │           ARRAY[0] ARRAY[1] ARRAY[2] ARRAY[3]    │
    │             10       20       30       40        │
    │                       ↑                          │
    │                    SI=4 points here              │
    │                                                  │
    │    Result: AX = 001EH = 30                      │
    └───────────────────────────────────────────────────┘
```

---

## 8. Based-Indexed Addressing

### 8.1 Without Displacement

```
BASED-INDEXED ADDRESSING (No Displacement):

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Address = Base Register + Index Register                      │
│                                                                  │
│   SYNTAX:    instruction reg, [base + index]                    │
│                                                                  │
│   COMBINATIONS:                                                  │
│   • [BX + SI]  - Default segment DS                             │
│   • [BX + DI]  - Default segment DS                             │
│   • [BP + SI]  - Default segment SS                             │
│   • [BP + DI]  - Default segment SS                             │
│                                                                  │
│   NOT ALLOWED: [BX+BP], [SI+DI]                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

EXAMPLES:

MOV AX, [BX+SI]     ; AX ← DS:[BX+SI]
MOV AX, [BX+DI]     ; AX ← DS:[BX+DI]
MOV AX, [BP+SI]     ; AX ← SS:[BP+SI]
MOV AX, [BP+DI]     ; AX ← SS:[BP+DI]


USE CASE: 2D ARRAY ACCESS

; Access MATRIX[row][col]
; Row major order: element at row×COLS + col

MATRIX DW 100 DUP(?)    ; 10×10 matrix of words
COLS EQU 10

; Access MATRIX[3][5]
MOV BX, 3 * COLS * 2    ; BX = row × columns × 2 (word size)
MOV SI, 5 * 2           ; SI = column × 2
MOV AX, MATRIX[BX+SI]   ; AX = MATRIX[3][5]

; Or using based-indexed:
MOV AX, [BX+SI]         ; After setting up BX and SI
```

### 8.2 With Displacement

```
BASED-INDEXED WITH DISPLACEMENT:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Address = Base + Index + Displacement                         │
│                                                                  │
│   SYNTAX:    instruction reg, [base + index + disp]             │
│              instruction reg, disp[base + index]                │
│              instruction reg, disp[base][index]                 │
│                                                                  │
│   EXAMPLES:                                                      │
│   MOV AX, [BX+SI+10H]       ; AX ← DS:[BX+SI+10H]               │
│   MOV AX, TABLE[BX+SI]      ; AX ← DS:[TABLE+BX+SI]             │
│   MOV AX, [BP+DI+4]         ; AX ← SS:[BP+DI+4]                 │
│                                                                  │
│   This is the most flexible addressing mode                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

USE CASE: STRUCTURE ARRAY

EMPLOYEE STRUC
    ID      DW ?        ; Offset 0
    SALARY  DD ?        ; Offset 2
    NAME    DB 20 DUP(?); Offset 6
EMPLOYEE ENDS

EMPLOYEES EMPLOYEE 100 DUP(<>)  ; Array of 100 employees

; Access EMPLOYEES[i].SALARY
; Displacement = offset of SALARY in structure = 2
; BX = base of array
; SI = i × size of EMPLOYEE

MOV BX, OFFSET EMPLOYEES
MOV SI, 26 * 5          ; 5th employee (index 5, size 26)
MOV AX, [BX+SI+2]       ; Low word of SALARY
MOV DX, [BX+SI+4]       ; High word of SALARY


VISUALIZATION:

    MOV AX, [BX+SI+10H]   where BX=1000H, SI=0200H
    
         BX         SI        Disp
    ┌─────────┐┌─────────┐┌─────────┐
    │  1000H  ││  0200H  ││   10H   │
    └─────────┘└─────────┘└─────────┘
         │          │          │
         └────┬─────┴────┬─────┘
              │          │
              = 1210H (effective address)
                   │
                   ▼
    ┌───────────────────────────────────────────┐
    │               MEMORY                       │
    │    1210H: │ AB │ CD │                     │
    │           └────┴────┘                     │
    │                                           │
    │    Result: AX = CDABH                    │
    └───────────────────────────────────────────┘
```

---

## 9. Effective Address Summary

### 9.1 All Addressing Modes Table

```
EFFECTIVE ADDRESS CALCULATION:

┌────────────────────────────────────────────────────────────────────────────┐
│  Addressing Mode          │  Effective Address    │  Default  │  Example   │
│                           │  Formula              │  Segment  │            │
├───────────────────────────┼───────────────────────┼───────────┼────────────┤
│  Register                 │  N/A (register)       │  N/A      │ MOV AX,BX  │
│  Immediate                │  N/A (in instruction) │  N/A      │ MOV AX,5   │
├───────────────────────────┼───────────────────────┼───────────┼────────────┤
│  Direct                   │  Displacement         │  DS       │ [1234H]    │
│  Register Indirect        │  [BX|SI|DI]          │  DS       │ [BX]       │
│                           │  [BP]                 │  SS       │ [BP]       │
├───────────────────────────┼───────────────────────┼───────────┼────────────┤
│  Based                    │  [BX] + disp          │  DS       │ [BX+4]     │
│                           │  [BP] + disp          │  SS       │ [BP+4]     │
├───────────────────────────┼───────────────────────┼───────────┼────────────┤
│  Indexed                  │  [SI] + disp          │  DS       │ [SI+4]     │
│                           │  [DI] + disp          │  DS       │ [DI+4]     │
├───────────────────────────┼───────────────────────┼───────────┼────────────┤
│  Based-Indexed            │  [BX+SI]              │  DS       │ [BX+SI]    │
│                           │  [BX+DI]              │  DS       │ [BX+DI]    │
│                           │  [BP+SI]              │  SS       │ [BP+SI]    │
│                           │  [BP+DI]              │  SS       │ [BP+DI]    │
├───────────────────────────┼───────────────────────┼───────────┼────────────┤
│  Based-Indexed + Disp     │  [BX+SI] + disp       │  DS       │ [BX+SI+4]  │
│                           │  [BX+DI] + disp       │  DS       │ [BX+DI+4]  │
│                           │  [BP+SI] + disp       │  SS       │ [BP+SI+4]  │
│                           │  [BP+DI] + disp       │  SS       │ [BP+DI+4]  │
└────────────────────────────────────────────────────────────────────────────┘
```

### 9.2 Valid Combinations Diagram

```
VALID ADDRESSING MODE COMBINATIONS:

Effective Address = [Base] + [Index] + [Displacement]

                    ┌─────────────────────────────────────────────┐
                    │            EFFECTIVE ADDRESS                │
                    │                                             │
                    │   ┌─────────────────────────────────────┐   │
                    │   │         BASE (optional)             │   │
                    │   │    ┌────────┐    ┌────────┐         │   │
                    │   │    │   BX   │ OR │   BP   │         │   │
                    │   │    └────────┘    └────────┘         │   │
                    │   └──────────────┬──────────────────────┘   │
                    │                  │                          │
                    │                  +                          │
                    │                  │                          │
                    │   ┌──────────────┴──────────────────────┐   │
                    │   │         INDEX (optional)            │   │
                    │   │    ┌────────┐    ┌────────┐         │   │
                    │   │    │   SI   │ OR │   DI   │         │   │
                    │   │    └────────┘    └────────┘         │   │
                    │   └──────────────┬──────────────────────┘   │
                    │                  │                          │
                    │                  +                          │
                    │                  │                          │
                    │   ┌──────────────┴──────────────────────┐   │
                    │   │      DISPLACEMENT (optional)        │   │
                    │   │    ┌──────────────────────────┐     │   │
                    │   │    │  8-bit or 16-bit value   │     │   │
                    │   │    └──────────────────────────┘     │   │
                    │   └─────────────────────────────────────┘   │
                    │                                             │
                    └─────────────────────────────────────────────┘

RULES:
• At least one component must be present for memory access
• Cannot use [BX + BP] (two base registers)
• Cannot use [SI + DI] (two index registers)
• Can use any combination of one base + one index + displacement
```

---

## 📋 Summary Table

| Mode | Example | Effective Address | Use Case |
|------|---------|-------------------|----------|
| Register | MOV AX, BX | N/A | Fast data transfer |
| Immediate | MOV AX, 5 | N/A | Load constants |
| Direct | MOV AX, [1000H] | 1000H | Access fixed locations |
| Reg Indirect | MOV AX, [BX] | BX | Pointer dereference |
| Based | MOV AX, [BX+4] | BX+4 | Structure fields |
| Indexed | MOV AX, [SI+4] | SI+4 | Array elements |
| Based-Indexed | MOV AX, [BX+SI] | BX+SI | 2D arrays |
| Based-Indexed+Disp | MOV AX, [BX+SI+4] | BX+SI+4 | Structure arrays |

---

## ❓ Quick Revision Questions

1. **Which registers can be used for indirect addressing?**
   <details>
   <summary>Show Answer</summary>
   BX, SI, DI (default segment DS) and BP (default segment SS). AX, CX, DX, and SP cannot be used inside brackets for memory addressing.
   </details>

2. **Calculate EA for [BX+SI+10H] if BX=1000H, SI=0200H**
   <details>
   <summary>Show Answer</summary>
   EA = BX + SI + 10H = 1000H + 0200H + 10H = 1210H. Physical address would be DS×16 + 1210H.
   </details>

3. **What is the default segment for [BP+4]?**
   <details>
   <summary>Show Answer</summary>
   SS (Stack Segment). Any addressing mode involving BP defaults to SS because BP is designed for stack frame access. [BX], [SI], [DI] default to DS.
   </details>

4. **Why is [BX+BP] not a valid addressing mode?**
   <details>
   <summary>Show Answer</summary>
   BX and BP are both base registers. The 8086 allows only one base (BX or BP) and one index (SI or DI) in an address. You cannot combine two base registers or two index registers.
   </details>

5. **How would you access the 5th element of a word array at ARRAY?**
   <details>
   <summary>Show Answer</summary>
   MOV SI, 8 (5th element = index 4, ×2 for word size = 8), then MOV AX, ARRAY[SI]. Or use MOV AX, [ARRAY+8] for direct addressing.
   </details>

6. **What's the difference between MOV AX, BX and MOV AX, [BX]?**
   <details>
   <summary>Show Answer</summary>
   MOV AX, BX copies the contents of BX into AX (register addressing). MOV AX, [BX] uses BX as a pointer and copies the word at memory address DS:BX into AX (register indirect addressing).
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [4.3 Memory Segmentation](03-memory-segmentation.md) | [Unit 4 Index](README.md) | [4.5 Instruction Set](05-instruction-set.md) |

---

*[← Previous: Memory Segmentation](03-memory-segmentation.md) | [Next: Instruction Set →](05-instruction-set.md)*
