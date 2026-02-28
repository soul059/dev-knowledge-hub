# Chapter 3.1: Assembly Language Basics

## 📚 Chapter Overview

Assembly language is a low-level programming language that provides a symbolic representation of machine code. This chapter covers the fundamentals of 8085 assembly language programming, including syntax, directives, and program structure.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Understand assembly language syntax and format
- Use assembler directives correctly
- Write well-structured assembly programs
- Convert assembly code to machine code
- Debug and trace program execution

---

## 1. Assembly Language Fundamentals

### 1.1 What is Assembly Language?

```
ASSEMBLY LANGUAGE LEVELS:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  HIGH-LEVEL LANGUAGE (C, Python, Java)                          │
│  ─────────────────────────────────────                           │
│  sum = a + b;                                                   │
│      │                                                          │
│      ▼ Compiler                                                 │
│                                                                  │
│  ASSEMBLY LANGUAGE (Human-readable)                             │
│  ─────────────────────────────────────                           │
│  LDA A_ADDR      ; Load 'a'                                     │
│  MOV B, A        ; Save to B                                    │
│  LDA B_ADDR      ; Load 'b'                                     │
│  ADD B           ; Add a + b                                    │
│  STA SUM_ADDR    ; Store result                                 │
│      │                                                          │
│      ▼ Assembler                                                │
│                                                                  │
│  MACHINE CODE (Binary/Hex)                                      │
│  ─────────────────────────────────────                           │
│  3A 00 20        ; LDA 2000H                                    │
│  47              ; MOV B, A                                     │
│  3A 01 20        ; LDA 2001H                                    │
│  80              ; ADD B                                        │
│  32 02 20        ; STA 2002H                                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Advantages of Assembly Language

| Advantage | Description |
|-----------|-------------|
| **Speed** | Highly optimized, fast execution |
| **Size** | Compact code, minimal memory use |
| **Hardware Access** | Direct control over processor |
| **Efficiency** | No overhead from higher-level abstractions |
| **Timing Control** | Precise control over execution timing |

---

## 2. Statement Format

### 2.1 Assembly Statement Structure

```
ASSEMBLY STATEMENT FORMAT:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   [Label:]    Opcode    [Operand(s)]    [; Comment]            │
│                                                                  │
│   Optional    Required  Depends on      Optional                │
│                         instruction                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

FIELD DESCRIPTIONS:
───────────────────────────────────────────────────────────────────

LABEL (Optional):
• Symbolic name for memory address
• Must start with letter (A-Z)
• Can contain letters, digits, underscore
• Ends with colon (:)
• Examples: LOOP:, START:, DATA1:

OPCODE (Required):
• Mnemonic for instruction
• Represents operation to perform
• Examples: MOV, ADD, JMP, LDA

OPERAND(S) (Instruction-dependent):
• Data on which opcode operates
• Can be: register, data, address, label
• Multiple operands separated by comma
• Examples: A, B    45H    LOOP    2050H

COMMENT (Optional):
• Starts with semicolon (;)
• Ignored by assembler
• Documents code purpose
• Example: ; This adds two numbers


EXAMPLES:
───────────────────────────────────────────────────────────────────

START:  MVI A, 45H      ; Load 45H into accumulator
        ADD B           ; Add B to A
        STA 2050H       ; Store result at 2050H
LOOP:   DCR C           ; Decrement counter
        JNZ LOOP        ; Repeat if not zero
        HLT             ; Stop processor
```

### 2.2 Valid Label Names

```
LABEL NAMING RULES:

VALID LABELS:                    INVALID LABELS:
─────────────                    ──────────────
START:                           2START:   (starts with number)
LOOP1:                           #LOOP:    (special character)
DATA_1:                          LDA:      (reserved opcode)
MyLabel:                         MOV:      (reserved opcode)
COUNTER:                         A:        (register name)
X1:                              3CH:      (starts with number)

RESERVED WORDS (Cannot be labels):
─────────────────────────────────
• All opcodes: MOV, ADD, SUB, JMP, etc.
• Register names: A, B, C, D, E, H, L, M
• Directives: ORG, DB, DW, EQU, DS, END
```

---

## 3. Assembler Directives

### 3.1 ORG (Origin)

```
ORG DIRECTIVE:

Purpose: Sets the starting address for subsequent code/data
Syntax:  ORG address

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   ORG 0000H           ; Code starts at 0000H                    │
│   START: JMP MAIN     ; Reset vector                            │
│                                                                  │
│   ORG 0100H           ; Main program at 0100H                   │
│   MAIN:  MVI A, 00H                                             │
│          ...                                                     │
│                                                                  │
│   ORG 2000H           ; Data segment at 2000H                   │
│   DATA1: DB 45H                                                 │
│                                                                  │
│   MEMORY LAYOUT:                                                 │
│   ┌────────────────────────────────────────┐                    │
│   │  0000H   │ JMP MAIN (Reset)           │                    │
│   │  ...     │                             │                    │
│   │  0100H   │ Main program starts        │                    │
│   │  ...     │                             │                    │
│   │  2000H   │ Data storage               │                    │
│   └────────────────────────────────────────┘                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 DB (Define Byte)

```
DB DIRECTIVE:

Purpose: Reserves and initializes byte(s) in memory
Syntax:  [Label:] DB value1 [, value2, ...]

EXAMPLES:
───────────────────────────────────────────────────────────────────

DATA1:  DB 45H              ; Single byte: 45H

DATA2:  DB 12H, 34H, 56H    ; Three bytes: 12H, 34H, 56H

MSG:    DB 'H','E','L','L','O'  ; ASCII string (5 bytes)

TABLE:  DB 00H, 01H, 02H, 03H   ; Lookup table


MEMORY REPRESENTATION:
───────────────────────────────────────────────────────────────────

ORG 2000H
DATA1:  DB 45H
DATA2:  DB 12H, 34H, 56H
MSG:    DB 'HELLO'

Address │ Content │ Label
────────┼─────────┼─────────
2000H   │   45    │ DATA1
2001H   │   12    │ DATA2
2002H   │   34    │
2003H   │   56    │
2004H   │   48    │ MSG ('H')
2005H   │   45    │     ('E')
2006H   │   4C    │     ('L')
2007H   │   4C    │     ('L')
2008H   │   4F    │     ('O')
```

### 3.3 DW (Define Word)

```
DW DIRECTIVE:

Purpose: Reserves and initializes 16-bit word(s)
Syntax:  [Label:] DW value1 [, value2, ...]

EXAMPLES:
───────────────────────────────────────────────────────────────────

ADDR1:  DW 2050H            ; Single word: 2050H

ADDRS:  DW 1000H, 2000H, 3000H  ; Three 16-bit addresses

COUNTER: DW 0000H           ; 16-bit counter (initialized to 0)


MEMORY REPRESENTATION (Little Endian):
───────────────────────────────────────────────────────────────────

ORG 2000H
ADDR1:  DW 2050H
ADDRS:  DW 1000H, 2000H

Address │ Content │ Description
────────┼─────────┼─────────────────
2000H   │   50    │ Low byte of 2050H (ADDR1)
2001H   │   20    │ High byte of 2050H
2002H   │   00    │ Low byte of 1000H (ADDRS)
2003H   │   10    │ High byte of 1000H
2004H   │   00    │ Low byte of 2000H
2005H   │   20    │ High byte of 2000H

NOTE: 8085 uses LITTLE ENDIAN format
      (Low byte at lower address)
```

### 3.4 DS (Define Storage)

```
DS DIRECTIVE:

Purpose: Reserves uninitialized memory space
Syntax:  [Label:] DS size

EXAMPLES:
───────────────────────────────────────────────────────────────────

BUFFER: DS 50           ; Reserve 50 bytes
TEMP:   DS 2            ; Reserve 2 bytes (for 16-bit temp)
ARRAY:  DS 100          ; Reserve 100 bytes for array


USAGE EXAMPLE:
───────────────────────────────────────────────────────────────────

ORG 2000H
BUFFER: DS 10           ; 10-byte buffer (2000H-2009H)
COUNT:  DB 00H          ; Counter at 200AH
RESULT: DS 2            ; 2-byte result (200BH-200CH)

Memory map:
2000H-2009H: BUFFER (10 bytes, uninitialized)
200AH:       COUNT  (1 byte, initialized to 00H)
200BH-200CH: RESULT (2 bytes, uninitialized)
```

### 3.5 EQU (Equate)

```
EQU DIRECTIVE:

Purpose: Assigns a constant value to a symbol
Syntax:  Name EQU value

EXAMPLES:
───────────────────────────────────────────────────────────────────

; Define constants
COUNT   EQU 10          ; COUNT = 10
PORTA   EQU 80H         ; Port A address
PORTB   EQU 81H         ; Port B address
CR      EQU 0DH         ; Carriage return
LF      EQU 0AH         ; Line feed

; Usage in program
        MVI A, COUNT    ; Same as MVI A, 10
        OUT PORTA       ; Same as OUT 80H
        MVI B, CR       ; Same as MVI B, 0DH


BENEFITS:
───────────────────────────────────────────────────────────────────
• Makes code readable (COUNT vs 10)
• Easy to change values (change in one place)
• Self-documenting code
• No memory used (compile-time substitution)
```

### 3.6 END Directive

```
END DIRECTIVE:

Purpose: Marks the end of assembly source code
Syntax:  END [start_label]

EXAMPLES:
───────────────────────────────────────────────────────────────────

; Simple END
        ORG 0100H
START:  MVI A, 00H
        HLT
        END             ; End of source

; END with start address
        ORG 0000H
        JMP MAIN
        
        ORG 0100H
MAIN:   MVI A, 00H
        HLT
        END MAIN        ; End, execution starts at MAIN


NOTE: Code/data after END is ignored by assembler
```

---

## 4. Program Structure

### 4.1 Standard Program Template

```
8085 PROGRAM TEMPLATE:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   ; ─────────────────────────────────────────────────────────   │
│   ; Program: [Program Name]                                     │
│   ; Description: [What the program does]                        │
│   ; Author: [Your name]                                         │
│   ; Date: [Date]                                                │
│   ; ─────────────────────────────────────────────────────────   │
│                                                                  │
│   ; ==================== CONSTANTS ====================         │
│                                                                  │
│   COUNT   EQU 10              ; Loop count                      │
│   PORTA   EQU 80H             ; Output port                     │
│                                                                  │
│   ; ==================== RESET VECTOR =================         │
│                                                                  │
│           ORG 0000H                                             │
│           JMP MAIN            ; Jump to main program            │
│                                                                  │
│   ; ==================== INTERRUPT VECTORS ============         │
│                                                                  │
│           ORG 0024H           ; TRAP vector                     │
│           JMP TRAP_ISR                                          │
│                                                                  │
│           ORG 002CH           ; RST 5.5 vector                  │
│           JMP RST55_ISR                                         │
│                                                                  │
│   ; ==================== MAIN PROGRAM =================         │
│                                                                  │
│           ORG 0100H                                             │
│   MAIN:   LXI SP, 0FFFH       ; Initialize stack pointer       │
│                                                                  │
│           ; Main program code here                              │
│           ...                                                    │
│           ...                                                    │
│                                                                  │
│           HLT                 ; End of program                  │
│                                                                  │
│   ; ==================== SUBROUTINES ==================         │
│                                                                  │
│           ORG 0200H                                             │
│   DELAY:  ; Delay subroutine                                    │
│           ...                                                    │
│           RET                                                    │
│                                                                  │
│   ; ==================== ISRs =========================         │
│                                                                  │
│           ORG 0300H                                             │
│   TRAP_ISR:                                                      │
│           ...                                                    │
│           EI                                                     │
│           RET                                                    │
│                                                                  │
│   ; ==================== DATA SEGMENT =================         │
│                                                                  │
│           ORG 2000H                                             │
│   DATA1:  DB 00H                                                │
│   BUFFER: DS 50                                                 │
│                                                                  │
│           END MAIN                                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Memory Organization

```
TYPICAL 8085 MEMORY MAP:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Address Range    │ Usage                                      │
│   ─────────────────┼───────────────────────────────────────────  │
│   0000H - 00FFH    │ Reset and Interrupt Vectors               │
│                    │   0000H: RESET                             │
│                    │   0024H: TRAP                              │
│                    │   002CH: RST 5.5                           │
│                    │   0034H: RST 6.5                           │
│                    │   003CH: RST 7.5                           │
│   ─────────────────┼───────────────────────────────────────────  │
│   0100H - 1FFFH    │ Program Code (ROM/EPROM)                  │
│   ─────────────────┼───────────────────────────────────────────  │
│   2000H - 3FFFH    │ Data Storage (RAM)                        │
│   ─────────────────┼───────────────────────────────────────────  │
│   4000H - 7FFFH    │ I/O Memory (if memory-mapped)             │
│   ─────────────────┼───────────────────────────────────────────  │
│   8000H - FFFFH    │ Stack Area (grows downward)               │
│                    │   Stack Pointer typically at FFFFH        │
│                                                                  │
│   STACK GROWTH:                                                  │
│   ┌───────────────────┐ FFFFH ← SP (initial)                   │
│   │  Available Stack  │                                         │
│   │       Space       │                                         │
│   │         ↓         │ Stack grows DOWN                        │
│   │  (PUSH decrements │                                         │
│   │       SP)         │                                         │
│   └───────────────────┘                                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Hand Assembly

### 5.1 Converting Assembly to Machine Code

```
HAND ASSEMBLY PROCESS:

EXAMPLE PROGRAM:
───────────────────────────────────────────────────────────────────

        ORG 2000H
START:  MVI A, 25H      ; Load A with 25H
        MVI B, 30H      ; Load B with 30H
        ADD B           ; A = A + B
        STA 2050H       ; Store result
        HLT             ; Stop

STEP 1: Determine instruction opcodes and sizes
───────────────────────────────────────────────────────────────────
MVI A, data  → 3E, data     (2 bytes)
MVI B, data  → 06, data     (2 bytes)
ADD B        → 80           (1 byte)
STA addr     → 32, lo, hi   (3 bytes)
HLT          → 76           (1 byte)

STEP 2: Calculate addresses
───────────────────────────────────────────────────────────────────
Address │ Instruction     │ Bytes
────────┼─────────────────┼────────
2000H   │ MVI A, 25H      │ 2
2002H   │ MVI B, 30H      │ 2
2004H   │ ADD B           │ 1
2005H   │ STA 2050H       │ 3
2008H   │ HLT             │ 1

STEP 3: Generate machine code
───────────────────────────────────────────────────────────────────
Address │ Machine Code │ Assembly
────────┼──────────────┼──────────────────
2000H   │     3E       │ MVI A, 25H (opcode)
2001H   │     25       │           (data)
2002H   │     06       │ MVI B, 30H (opcode)
2003H   │     30       │           (data)
2004H   │     80       │ ADD B
2005H   │     32       │ STA 2050H (opcode)
2006H   │     50       │           (low addr)
2007H   │     20       │           (high addr)
2008H   │     76       │ HLT
```

### 5.2 Label Resolution

```
RESOLVING LABELS:

EXAMPLE WITH LABELS:
───────────────────────────────────────────────────────────────────

        ORG 2000H
        MVI C, 05H
LOOP:   DCR C
        JNZ LOOP
        HLT

PASS 1: Create symbol table (assign addresses to labels)
───────────────────────────────────────────────────────────────────
Symbol Table:
Label   │ Address
────────┼─────────
LOOP    │ 2002H

PASS 2: Generate code with resolved addresses
───────────────────────────────────────────────────────────────────
Address │ Bytes │ Code     │ Instruction
────────┼───────┼──────────┼─────────────────
2000H   │  2    │ 0E 05    │ MVI C, 05H
2002H   │  1    │ 0D       │ DCR C       ← LOOP
2003H   │  3    │ C2 02 20 │ JNZ LOOP (2002H)
2006H   │  1    │ 76       │ HLT

Note: JNZ LOOP → C2 02 20
      C2 = JNZ opcode
      02 = low byte of 2002H
      20 = high byte of 2002H
```

---

## 6. Programming Examples

### 6.1 Basic Arithmetic Program

```
PROGRAM: Add two numbers stored in memory

; ─────────────────────────────────────────────────────────────────
; Program: ADD_TWO
; Purpose: Add two 8-bit numbers from memory, store result
; Input:   NUM1 at 2000H, NUM2 at 2001H
; Output:  SUM at 2002H
; ─────────────────────────────────────────────────────────────────

        ORG 0100H
START:  LXI SP, 0FFFH       ; Initialize stack

        LDA 2000H           ; Load NUM1
        MOV B, A            ; Save to B
        LDA 2001H           ; Load NUM2
        ADD B               ; A = NUM1 + NUM2
        STA 2002H           ; Store sum
        
        HLT                 ; Stop

; Data segment
        ORG 2000H
NUM1:   DB 25H              ; First number
NUM2:   DB 30H              ; Second number
SUM:    DS 1                ; Result storage

        END START

; EXECUTION TRACE:
; ─────────────────────────────────────────────────────────────────
; A = 25H (after LDA 2000H)
; B = 25H (after MOV B, A)
; A = 30H (after LDA 2001H)
; A = 55H (after ADD B: 25H + 30H = 55H)
; Memory[2002H] = 55H (after STA 2002H)
```

### 6.2 Loop Counter Program

```
PROGRAM: Count down from N to 0

; ─────────────────────────────────────────────────────────────────
; Program: COUNTDOWN
; Purpose: Decrement counter from 10 to 0
; ─────────────────────────────────────────────────────────────────

COUNT   EQU 0AH             ; Start value (10)

        ORG 0100H
START:  MVI B, COUNT        ; B = 10

LOOP:   MOV A, B            ; A = current count
        OUT 80H             ; Display count (optional)
        DCR B               ; B = B - 1
        JNZ LOOP            ; If B ≠ 0, repeat
        
        HLT                 ; Stop when B = 0

        END START

; TRACE:
; ─────────────────────────────────────────────────────────────────
; Iteration │  B value  │ Z flag
; ──────────┼───────────┼─────────
;     -     │    0AH    │   -
;     1     │    09H    │   0
;     2     │    08H    │   0
;    ...    │    ...    │   0
;     9     │    01H    │   0
;    10     │    00H    │   1  ← Exit loop
```

---

## 📋 Summary Table

| Directive | Purpose | Syntax | Example |
|-----------|---------|--------|---------|
| ORG | Set origin address | ORG addr | ORG 2000H |
| DB | Define byte(s) | label: DB val | DATA: DB 45H |
| DW | Define word(s) | label: DW val | ADDR: DW 2050H |
| DS | Reserve space | label: DS size | BUF: DS 50 |
| EQU | Define constant | name EQU val | COUNT EQU 10 |
| END | End of program | END [label] | END START |

---

## ❓ Quick Revision Questions

1. **What are the four fields in an assembly language statement?**
   <details>
   <summary>Show Answer</summary>
   Label (optional, ends with colon), Opcode (required, the instruction mnemonic), Operand (depends on instruction), Comment (optional, starts with semicolon).
   </details>

2. **What is the difference between DB and DS directives?**
   <details>
   <summary>Show Answer</summary>
   DB (Define Byte) initializes memory with specific values. DS (Define Storage) only reserves memory space without initialization. Example: DB 45H stores 45H; DS 10 reserves 10 bytes.
   </details>

3. **Why is EQU directive useful in programming?**
   <details>
   <summary>Show Answer</summary>
   EQU makes code readable by using meaningful names instead of numbers. It allows easy modification (change value in one place). No memory is used - it's a compile-time substitution.
   </details>

4. **Convert MVI A, 45H to machine code.**
   <details>
   <summary>Show Answer</summary>
   MVI A opcode is 3EH. So machine code is: 3E 45 (2 bytes at consecutive addresses).
   </details>

5. **What does ORG 2000H do?**
   <details>
   <summary>Show Answer</summary>
   ORG 2000H sets the location counter to 2000H. All subsequent code and data will be placed starting from address 2000H until another ORG directive is encountered.
   </details>

6. **Why is stack pointer initialized at the beginning of main program?**
   <details>
   <summary>Show Answer</summary>
   Stack pointer must be initialized before using PUSH, POP, CALL, or RET instructions. It's typically set to top of RAM (e.g., FFFFH or 0FFFH). Without initialization, stack operations will fail or corrupt memory.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Unit 3 Index](README.md) | [Unit 3 Index](README.md) | [3.2 Data Transfer Programs](02-data-transfer-programs.md) |

---

*[← Back to Unit 3 Index](README.md) | [Next: Data Transfer Programs →](02-data-transfer-programs.md)*
