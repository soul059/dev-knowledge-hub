# Chapter 3.2: Data Transfer Programs

## 📚 Chapter Overview

Data transfer is fundamental to all programs. This chapter covers moving data between registers, memory, and I/O ports using various addressing modes and techniques.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Transfer data between registers
- Move data to/from memory
- Perform block data transfers
- Use stack for data storage
- Exchange data between locations

---

## 1. Register to Register Transfer

### 1.1 MOV Instruction

```
MOV INSTRUCTION:

Syntax: MOV D, S    (Move Source to Destination)

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   MOV transfers 8-bit data between registers                   │
│                                                                  │
│   Source (S) and Destination (D) can be:                       │
│   A, B, C, D, E, H, L, or M (memory via HL)                    │
│                                                                  │
│   IMPORTANT:                                                     │
│   • Source is NOT modified                                      │
│   • Destination gets copy of source                             │
│   • Flags are NOT affected                                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

EXAMPLES:
───────────────────────────────────────────────────────────────────

; Copy A to B
MOV B, A        ; B ← A, A unchanged

; Copy contents between register pairs
MOV B, H        ; B ← H
MOV C, L        ; C ← L
; Now BC = HL

; Chain copy
MOV B, A        ; B ← A
MOV C, B        ; C ← B (which is A)
MOV D, C        ; D ← C
; Now A, B, C, D all have same value
```

### 1.2 Data Exchange

```
EXCHANGE OPERATIONS:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   XCHG - Exchange HL and DE (1 byte, 4 T-states)               │
│   ───────────────────────────────────────────────                │
│                                                                  │
│   Before:   HL = 1234H,  DE = 5678H                            │
│   After:    HL = 5678H,  DE = 1234H                            │
│                                                                  │
│   Single instruction exchange! No temp register needed.         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

MANUAL EXCHANGE (using temp register):
───────────────────────────────────────────────────────────────────

; Exchange A and B using C as temp
MOV C, A        ; C = A (save A)
MOV A, B        ; A = B
MOV B, C        ; B = saved A
; Result: A and B exchanged, C modified

; Exchange without temp (using XOR trick):
XRA B           ; A = A XOR B
XRA A           ; Wait, this clears A! 
                ; Better to use temp register for 8085

XCHG USAGE EXAMPLE:
───────────────────────────────────────────────────────────────────

; Using two pointers: HL for source, DE for destination
; Need to switch roles

LXI H, 2000H    ; HL = source address
LXI D, 3000H    ; DE = dest address

; ... do some work ...

XCHG            ; Now HL = 3000H, DE = 2000H
                ; HL is now dest, DE is source
```

---

## 2. Immediate Data Loading

### 2.1 MVI Instruction

```
MVI INSTRUCTION:

Syntax: MVI r, data    (Move Immediate to register)
        MVI M, data    (Move Immediate to memory via HL)

EXAMPLES:
───────────────────────────────────────────────────────────────────

; Load registers with immediate values
MVI A, 00H      ; A = 00H (clear A)
MVI B, FFH      ; B = FFH
MVI C, 55H      ; C = 55H (bit pattern 01010101)

; Initialize all registers
MVI A, 00H
MVI B, 00H
MVI C, 00H
MVI D, 00H
MVI E, 00H
MVI H, 00H
MVI L, 00H

; Load ASCII character
MVI A, 'A'      ; A = 41H (ASCII for 'A')
MVI B, 30H      ; B = 30H (ASCII for '0')

; Initialize memory via HL
LXI H, 2000H    ; HL = 2000H (pointer)
MVI M, 45H      ; Memory[2000H] = 45H
```

### 2.2 LXI Instruction

```
LXI INSTRUCTION:

Syntax: LXI rp, data16    (Load Immediate 16-bit)

Register pairs: B (BC), D (DE), H (HL), SP

EXAMPLES:
───────────────────────────────────────────────────────────────────

; Initialize register pairs
LXI B, 1234H    ; B = 12H, C = 34H (BC = 1234H)
LXI D, 2000H    ; D = 20H, E = 00H (DE = 2000H)
LXI H, 3050H    ; H = 30H, L = 50H (HL = 3050H)
LXI SP, 0FFFH   ; SP = 0FFFH (Stack pointer)

MEMORY LOADING ORDER (Little Endian):
───────────────────────────────────────────────────────────────────

LXI H, 2050H is stored as:
Address │ Content
────────┼─────────
PC      │   21    (Opcode for LXI H)
PC+1    │   50    (Low byte first!)
PC+2    │   20    (High byte second)

Fetched in order: H=20H, L=50H → HL = 2050H
```

---

## 3. Memory to Register Transfer

### 3.1 Direct Addressing

```
DIRECT MEMORY ACCESS:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   LDA addr    Load Accumulator Direct                           │
│   STA addr    Store Accumulator Direct                          │
│   LHLD addr   Load HL Direct                                    │
│   SHLD addr   Store HL Direct                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

EXAMPLES:
───────────────────────────────────────────────────────────────────

; Load single byte
LDA 2000H       ; A = Memory[2000H]

; Store single byte
STA 2050H       ; Memory[2050H] = A

; Load 16-bit value
LHLD 2000H      ; L = Memory[2000H]
                ; H = Memory[2001H]

; Store 16-bit value
SHLD 3000H      ; Memory[3000H] = L
                ; Memory[3001H] = H


PROGRAM: Copy data from one location to another
───────────────────────────────────────────────────────────────────

; Copy byte from 2000H to 3000H
        LDA 2000H       ; Read source
        STA 3000H       ; Write to destination

; Copy 16-bit word from 2000H to 3000H
        LHLD 2000H      ; Read 16-bit source
        SHLD 3000H      ; Write 16-bit destination
```

### 3.2 Indirect Addressing

```
INDIRECT MEMORY ACCESS (via pointer registers):

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Using HL as pointer:                                          │
│   ─────────────────────                                          │
│   MOV A, M    A ← Memory[HL]                                    │
│   MOV M, A    Memory[HL] ← A                                    │
│   MOV r, M    r ← Memory[HL]                                    │
│   MOV M, r    Memory[HL] ← r                                    │
│                                                                  │
│   Using BC or DE as pointer:                                    │
│   ────────────────────────────                                   │
│   LDAX B      A ← Memory[BC]                                    │
│   LDAX D      A ← Memory[DE]                                    │
│   STAX B      Memory[BC] ← A                                    │
│   STAX D      Memory[DE] ← A                                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

EXAMPLE - Array Access:
───────────────────────────────────────────────────────────────────

; Access array elements at 2000H-2004H
        LXI H, 2000H    ; HL points to array[0]
        MOV A, M        ; A = array[0]
        INX H           ; HL = 2001H (array[1])
        MOV B, M        ; B = array[1]
        INX H           ; HL = 2002H (array[2])
        MOV C, M        ; C = array[2]
        
; Result: A=array[0], B=array[1], C=array[2]


EXAMPLE - Two Pointers:
───────────────────────────────────────────────────────────────────

; Source at 2000H, Destination at 3000H
        LXI H, 2000H    ; Source pointer
        LXI D, 3000H    ; Destination pointer
        
        MOV A, M        ; Read from source (HL)
        STAX D          ; Write to destination (DE)
        
        INX H           ; Next source
        INX D           ; Next destination
```

---

## 4. Block Data Transfer

### 4.1 Byte-by-Byte Transfer

```
PROGRAM: Block Transfer (10 bytes from 2000H to 3000H)

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   ALGORITHM:                                                     │
│   1. Initialize source pointer (HL)                             │
│   2. Initialize destination pointer (DE)                        │
│   3. Initialize counter                                          │
│   4. Read byte from source                                       │
│   5. Write byte to destination                                   │
│   6. Increment pointers                                          │
│   7. Decrement counter                                           │
│   8. If counter ≠ 0, repeat from step 4                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

CODE:
───────────────────────────────────────────────────────────────────

        ORG 0100H
        
        LXI H, 2000H    ; Source address
        LXI D, 3000H    ; Destination address
        MVI C, 0AH      ; Counter = 10
        
LOOP:   MOV A, M        ; A = source byte
        STAX D          ; destination = A
        INX H           ; source++
        INX D           ; destination++
        DCR C           ; counter--
        JNZ LOOP        ; repeat if C ≠ 0
        
        HLT

; TRACE (first 3 iterations):
; ─────────────────────────────────────────────────────────────────
; Iter │  HL   │  DE   │  C  │ A (copied)
; ─────┼───────┼───────┼─────┼────────────
;   0  │ 2000H │ 3000H │ 0AH │    -
;   1  │ 2001H │ 3001H │ 09H │ M[2000]
;   2  │ 2002H │ 3002H │ 08H │ M[2001]
;   3  │ 2003H │ 3003H │ 07H │ M[2002]
```

### 4.2 Block Transfer with Count from Memory

```
PROGRAM: Transfer N bytes (count stored in memory)

        ORG 0100H
        
; Addresses
SRC     EQU 2000H       ; Source start
DST     EQU 3000H       ; Destination start
CNT     EQU 4000H       ; Count location
        
        LDA CNT         ; Get count
        MOV C, A        ; C = count
        
        ORA A           ; Check if count is 0
        JZ DONE         ; If zero, nothing to copy
        
        LXI H, SRC      ; Source pointer
        LXI D, DST      ; Destination pointer
        
LOOP:   MOV A, M        ; Read source
        STAX D          ; Write destination
        INX H           ; Next source
        INX D           ; Next destination
        DCR C           ; Decrement count
        JNZ LOOP        ; Repeat if not done
        
DONE:   HLT
```

### 4.3 Block Transfer Upward vs Downward

```
UPWARD vs DOWNWARD COPY:

PROBLEM: Overlapping blocks

Source: 2000H-200FH (16 bytes)
Dest:   2008H-2017H (overlaps with source)

┌───────────────────────────────────────────────────────────────┐
│                                                                │
│   UPWARD COPY (Forward):                                      │
│   ─────────────────────                                        │
│   Memory: │...│S0│S1│S2│S3│S4│S5│S6│S7│...│                  │
│   Index:    2000 01 02 03 04 05 06 07 08                      │
│                                     └─Dest starts here         │
│                                                                │
│   If copying forward:                                          │
│   M[2008] = M[2000] ← OK                                      │
│   M[2009] = M[2001] ← OK                                      │
│   ...                                                          │
│   But source data at 2008+ already overwritten!               │
│   DATA CORRUPTED!                                              │
│                                                                │
│   SOLUTION: Copy BACKWARD (from end to start)                 │
│   ─────────────────────────────────────────────                │
│   Start from highest address:                                  │
│   M[2017] = M[200F] ← Safe                                    │
│   M[2016] = M[200E] ← Safe                                    │
│   ...down to...                                                │
│   M[2008] = M[2000] ← Safe                                    │
│                                                                │
└───────────────────────────────────────────────────────────────┘

BACKWARD COPY CODE:
───────────────────────────────────────────────────────────────────

; Copy 16 bytes from 2000H to 2008H (overlapping)
; Copy backward to avoid corruption

        LXI H, 200FH    ; Source end
        LXI D, 2017H    ; Destination end
        MVI C, 10H      ; Count = 16
        
LOOP:   MOV A, M        ; Read from source
        STAX D          ; Write to destination
        DCX H           ; Previous source
        DCX D           ; Previous destination
        DCR C           ; Decrement count
        JNZ LOOP
        
        HLT
```

---

## 5. Stack Operations

### 5.1 PUSH and POP

```
STACK OPERATIONS:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   PUSH rp    Push register pair to stack                       │
│   POP rp     Pop from stack to register pair                   │
│                                                                  │
│   Register pairs: B (BC), D (DE), H (HL), PSW (A + Flags)      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

PUSH OPERATION:
───────────────────────────────────────────────────────────────────

Before PUSH B (BC = 1234H, SP = FFF0H):

    Stack Memory:              After:
    ┌─────┬───────┐           ┌─────┬───────┐
    │FFEF │       │           │FFEF │       │
    │FFF0 │       │ ← SP      │FFEE │  34   │ ← SP
    │FFF1 │       │           │FFEF │  12   │
    └─────┴───────┘           │FFF0 │       │
                              └─────┴───────┘
    
    SP = SP - 1; M[SP] = B (12H)
    SP = SP - 1; M[SP] = C (34H)
    New SP = FFEEH


POP OPERATION:
───────────────────────────────────────────────────────────────────

Before POP D (SP = FFEEH):

    Stack Memory:              After:
    ┌─────┬───────┐           ┌─────┬───────┐
    │FFEE │  34   │ ← SP      │FFEE │  34   │
    │FFEF │  12   │           │FFEF │  12   │
    │FFF0 │       │           │FFF0 │       │ ← SP
    └─────┴───────┘           └─────┴───────┘
    
    E = M[SP]; SP = SP + 1  (E = 34H)
    D = M[SP]; SP = SP + 1  (D = 12H)
    New SP = FFF0H, DE = 1234H
```

### 5.2 Saving and Restoring Context

```
PROGRAM: Save all registers, modify, restore

        ORG 0100H
        
        LXI SP, 0FFFH   ; Initialize stack
        
        ; Load some values
        LXI B, 1234H
        LXI D, 5678H
        LXI H, 9ABCH
        MVI A, 0FFH
        
        ; Save context
        PUSH PSW        ; Save A and Flags
        PUSH B          ; Save BC
        PUSH D          ; Save DE
        PUSH H          ; Save HL
        
        ; Stack now contains:
        ; FFEF: L (BCH)   ← SP
        ; FFF0: H (9AH)
        ; FFF1: E (78H)
        ; FFF2: D (56H)
        ; FFF3: C (34H)
        ; FFF4: B (12H)
        ; FFF5: Flags
        ; FFF6: A (FFH)
        
        ; Modify all registers freely
        MVI A, 00H
        LXI B, 0000H
        LXI D, 0000H
        LXI H, 0000H
        
        ; Restore context (reverse order!)
        POP H           ; Restore HL
        POP D           ; Restore DE
        POP B           ; Restore BC
        POP PSW         ; Restore A and Flags
        
        ; All registers restored to saved values
        HLT

; IMPORTANT: Pop in REVERSE order of Push!
```

### 5.3 XTHL Instruction

```
XTHL - Exchange Top of Stack with HL:

Before XTHL (SP = FFEEH, HL = 1234H):
Stack: M[FFEE] = 78H, M[FFEF] = 56H

    Stack:                    Registers:
    ┌─────┬───────┐          ┌───────────────┐
    │FFEE │  78   │ ← SP     │ H=12  L=34    │
    │FFEF │  56   │          └───────────────┘
    └─────┴───────┘

After XTHL:
    Stack:                    Registers:
    ┌─────┬───────┐          ┌───────────────┐
    │FFEE │  34   │ ← SP     │ H=56  L=78    │
    │FFEF │  12   │          └───────────────┘
    └─────┴───────┘

L ↔ M[SP], H ↔ M[SP+1]

USE CASE: Access return address on stack
```

---

## 6. I/O Operations

### 6.1 IN and OUT Instructions

```
I/O INSTRUCTIONS:

IN port     ; A ← Input port
OUT port    ; Output port ← A

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Port address: 8-bit (00H - FFH)                               │
│   Data width: 8-bit                                             │
│   Only accumulator used for I/O                                 │
│                                                                  │
│   Port address appears on:                                       │
│   • A0-A7 (lower address bus)                                   │
│   • A8-A15 (upper address bus) - duplicate!                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

EXAMPLES:
───────────────────────────────────────────────────────────────────

; Read from input port 80H
IN 80H          ; A = data from port 80H

; Write to output port 81H
MVI A, 55H      ; Data to send
OUT 81H         ; Send to port 81H

; Echo: Read input, send to output
LOOP:
    IN 80H      ; Read from input port
    OUT 81H     ; Write to output port
    JMP LOOP    ; Repeat forever


PROGRAM: Toggle LED on port 01H
───────────────────────────────────────────────────────────────────

        MVI A, 00H      ; Initial state: all OFF
        
LOOP:   OUT 01H         ; Send to LEDs
        CMA             ; Toggle all bits (00→FF→00...)
        CALL DELAY      ; Wait
        JMP LOOP        ; Repeat
        
DELAY:  MVI B, FFH      ; Outer loop
DLY1:   MVI C, FFH      ; Inner loop
DLY2:   DCR C
        JNZ DLY2
        DCR B
        JNZ DLY1
        RET
```

---

## 7. Complete Programs

### 7.1 Swap Two Memory Locations

```
PROGRAM: Swap contents of 2000H and 2001H

        ORG 0100H
        
        LDA 2000H       ; A = first value
        MOV B, A        ; Save in B
        LDA 2001H       ; A = second value
        STA 2000H       ; Store at first location
        MOV A, B        ; Retrieve first value
        STA 2001H       ; Store at second location
        
        HLT

; Before: M[2000H]=25H, M[2001H]=50H
; After:  M[2000H]=50H, M[2001H]=25H
```

### 7.2 Fill Memory Block with Value

```
PROGRAM: Fill 50 bytes starting at 3000H with AAH

        ORG 0100H
        
        LXI H, 3000H    ; Start address
        MVI C, 32H      ; Count = 50 decimal
        MVI A, 0AAH     ; Fill value
        
FILL:   MOV M, A        ; Store at [HL]
        INX H           ; Next location
        DCR C           ; Decrement count
        JNZ FILL        ; Repeat if not done
        
        HLT

; Result: M[3000H] through M[3031H] all contain AAH
```

### 7.3 Copy String with Null Terminator

```
PROGRAM: Copy null-terminated string

        ORG 0100H
        
        LXI H, 2000H    ; Source string
        LXI D, 3000H    ; Destination
        
COPY:   MOV A, M        ; Get character
        STAX D          ; Store at destination
        ORA A           ; Check if null (00H)
        JZ DONE         ; If null, we're done
        INX H           ; Next source
        INX D           ; Next destination
        JMP COPY        ; Continue
        
DONE:   HLT

; Data segment
        ORG 2000H
STRING: DB 'HELLO', 00H  ; Source: "HELLO\0"

        ORG 3000H
DEST:   DS 20            ; Destination buffer
```

---

## 📋 Summary Table

| Instruction | Operation | Bytes | T-States | Flags |
|-------------|-----------|-------|----------|-------|
| MOV r1, r2 | r1 ← r2 | 1 | 4 | None |
| MOV r, M | r ← M[HL] | 1 | 7 | None |
| MOV M, r | M[HL] ← r | 1 | 7 | None |
| MVI r, data | r ← data | 2 | 7 | None |
| LXI rp, data16 | rp ← data16 | 3 | 10 | None |
| LDA addr | A ← M[addr] | 3 | 13 | None |
| STA addr | M[addr] ← A | 3 | 13 | None |
| LDAX rp | A ← M[rp] | 1 | 7 | None |
| STAX rp | M[rp] ← A | 1 | 7 | None |
| XCHG | HL ↔ DE | 1 | 4 | None |
| PUSH rp | Stack ← rp | 1 | 12 | None |
| POP rp | rp ← Stack | 1 | 10 | PSW only |
| IN port | A ← port | 2 | 10 | None |
| OUT port | port ← A | 2 | 10 | None |

---

## ❓ Quick Revision Questions

1. **What is the difference between MOV and MVI?**
   <details>
   <summary>Show Answer</summary>
   MOV transfers data between registers (register to register), e.g., MOV A, B copies B to A. MVI loads immediate data into a register, e.g., MVI A, 45H loads the value 45H directly into A.
   </details>

2. **Why do we need LDAX/STAX when we have MOV with M?**
   <details>
   <summary>Show Answer</summary>
   MOV A, M uses HL as pointer. LDAX B/LDAX D use BC/DE as pointers. This allows using multiple pointers simultaneously for operations like copying data from one block to another without constantly modifying HL.
   </details>

3. **Write code to exchange contents of 2000H and 2001H.**
   <details>
   <summary>Show Answer</summary>
   LDA 2000H / MOV B, A / LDA 2001H / STA 2000H / MOV A, B / STA 2001H. Uses B as temporary storage to swap the values.
   </details>

4. **In what order must POP instructions appear relative to PUSH?**
   <details>
   <summary>Show Answer</summary>
   POP must be in reverse order of PUSH. If you PUSH B, PUSH D, PUSH H, you must POP H, POP D, POP B. Stack is LIFO (Last In, First Out).
   </details>

5. **What happens to SP after PUSH B?**
   <details>
   <summary>Show Answer</summary>
   SP decreases by 2. First, SP decrements and B is stored, then SP decrements again and C is stored. Example: If SP=FFF0H before PUSH B, SP=FFEEH after.
   </details>

6. **How would you copy 100 bytes from 2000H to 3000H?**
   <details>
   <summary>Show Answer</summary>
   LXI H, 2000H / LXI D, 3000H / MVI C, 64H / LOOP: MOV A, M / STAX D / INX H / INX D / DCR C / JNZ LOOP / HLT. Uses HL as source pointer, DE as destination, C as counter (64H = 100).
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [3.1 Assembly Language Basics](01-assembly-language-basics.md) | [Unit 3 Index](README.md) | [3.3 Arithmetic Operations](03-arithmetic-operations.md) |

---

*[← Previous: Assembly Language Basics](01-assembly-language-basics.md) | [Next: Arithmetic Operations →](03-arithmetic-operations.md)*
