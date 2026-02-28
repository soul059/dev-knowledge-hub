# Chapter 4.5: 8086 Instruction Set Overview

## 📚 Chapter Overview

The 8086 has a rich instruction set with over 100 instructions organized into categories. This chapter provides an overview of the major instruction groups with examples.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Classify 8086 instructions into categories
- Understand data transfer instructions
- Use arithmetic and logical instructions
- Work with string and control transfer instructions
- Apply processor control instructions

---

## 1. Instruction Categories

```
8086 INSTRUCTION SET CLASSIFICATION:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   1. DATA TRANSFER INSTRUCTIONS                                 │
│      MOV, PUSH, POP, XCHG, IN, OUT, LEA, LDS, LES, etc.        │
│                                                                  │
│   2. ARITHMETIC INSTRUCTIONS                                     │
│      ADD, SUB, MUL, DIV, INC, DEC, CMP, NEG, etc.              │
│                                                                  │
│   3. LOGICAL INSTRUCTIONS                                        │
│      AND, OR, XOR, NOT, TEST, SHL, SHR, ROL, ROR, etc.         │
│                                                                  │
│   4. STRING INSTRUCTIONS                                         │
│      MOVS, CMPS, SCAS, LODS, STOS, REP, etc.                   │
│                                                                  │
│   5. CONTROL TRANSFER INSTRUCTIONS                              │
│      JMP, CALL, RET, Jcc (conditional), LOOP, INT, IRET        │
│                                                                  │
│   6. PROCESSOR CONTROL INSTRUCTIONS                             │
│      CLC, STC, CLI, STI, CLD, STD, HLT, NOP, etc.              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Data Transfer Instructions

### 2.1 MOV Instruction

```
MOV - Move Data:

SYNTAX:    MOV destination, source

VALID COMBINATIONS:
┌────────────────────────────────────────────────────────────────┐
│   MOV reg, reg       ; Register to register                   │
│   MOV reg, mem       ; Memory to register                     │
│   MOV mem, reg       ; Register to memory                     │
│   MOV reg, imm       ; Immediate to register                  │
│   MOV mem, imm       ; Immediate to memory                    │
│   MOV segreg, reg16  ; Register to segment register           │
│   MOV segreg, mem16  ; Memory to segment register             │
│   MOV reg16, segreg  ; Segment register to register           │
│   MOV mem16, segreg  ; Segment register to memory             │
└────────────────────────────────────────────────────────────────┘

INVALID:
• MOV mem, mem         ; Cannot move memory to memory directly
• MOV segreg, imm      ; Cannot load segment with immediate
• MOV CS, anything     ; Cannot load CS directly

EXAMPLES:
MOV AX, BX            ; AX ← BX
MOV CL, 25            ; CL ← 25
MOV [1000H], AX       ; Memory ← AX
MOV DS, AX            ; DS ← AX (after loading AX)
```

### 2.2 Stack Instructions

```
PUSH and POP:

PUSH - Push onto stack:
┌────────────────────────────────────────────────────────────────┐
│   PUSH reg16         ; Push 16-bit register                   │
│   PUSH mem16         ; Push 16-bit memory                     │
│   PUSH segreg        ; Push segment register                  │
│                                                                │
│   Operation: SP ← SP - 2, then [SS:SP] ← operand             │
└────────────────────────────────────────────────────────────────┘

POP - Pop from stack:
┌────────────────────────────────────────────────────────────────┐
│   POP reg16          ; Pop to 16-bit register                 │
│   POP mem16          ; Pop to 16-bit memory                   │
│   POP segreg         ; Pop to segment register (except CS)    │
│                                                                │
│   Operation: operand ← [SS:SP], then SP ← SP + 2             │
└────────────────────────────────────────────────────────────────┘

PUSHF / POPF:
PUSHF                  ; Push FLAGS register
POPF                   ; Pop to FLAGS register

EXAMPLES:
PUSH AX               ; Save AX on stack
PUSH DS               ; Save DS on stack
POP DS                ; Restore DS
POP AX                ; Restore AX (LIFO order!)
```

### 2.3 Exchange and I/O

```
XCHG - Exchange:
XCHG AX, BX           ; Swap AX and BX
XCHG AL, [BX]         ; Swap AL with memory

IN/OUT - Input/Output:
IN AL, 60H            ; Read byte from port 60H
IN AX, 60H            ; Read word from port 60H
IN AL, DX             ; Read from port in DX
OUT 61H, AL           ; Write AL to port 61H
OUT DX, AX            ; Write AX to port in DX

LEA - Load Effective Address:
LEA BX, [SI+10H]      ; BX ← SI + 10H (address, not contents)
; Equivalent to: MOV BX, SI / ADD BX, 10H (but faster)

LDS/LES - Load Pointer:
LDS SI, [2000H]       ; SI ← [2000H], DS ← [2002H]
LES DI, [2000H]       ; DI ← [2000H], ES ← [2002H]
; Loads both offset and segment from memory
```

---

## 3. Arithmetic Instructions

### 3.1 Addition and Subtraction

```
ADD/ADC/SUB/SBB:

ADD dest, source      ; dest ← dest + source
ADC dest, source      ; dest ← dest + source + CF
SUB dest, source      ; dest ← dest - source
SBB dest, source      ; dest ← dest - source - CF

EXAMPLES:
ADD AX, BX            ; AX ← AX + BX
ADD AL, 10            ; AL ← AL + 10
ADD [BX], CX          ; Memory ← Memory + CX

; 32-bit addition (DX:AX + CX:BX)
ADD AX, BX            ; Add low words
ADC DX, CX            ; Add high words with carry


INC/DEC:
INC AX                ; AX ← AX + 1
DEC BX                ; BX ← BX - 1
INC BYTE PTR [BX]     ; Increment byte at [BX]
; Note: INC/DEC don't affect Carry Flag


NEG - Two's Complement:
NEG AX                ; AX ← 0 - AX (negate)
; Sets CF=1 unless operand was 0
```

### 3.2 Multiplication

```
MUL - Unsigned Multiply:
┌────────────────────────────────────────────────────────────────┐
│   MUL reg/mem8       ; AX ← AL × operand                      │
│   MUL reg/mem16      ; DX:AX ← AX × operand                   │
│                                                                │
│   Flags: CF=OF=1 if result doesn't fit in AL/AX              │
└────────────────────────────────────────────────────────────────┘

IMUL - Signed Multiply:
┌────────────────────────────────────────────────────────────────┐
│   IMUL reg/mem8      ; AX ← AL × operand (signed)             │
│   IMUL reg/mem16     ; DX:AX ← AX × operand (signed)          │
└────────────────────────────────────────────────────────────────┘

EXAMPLES:
; 8-bit multiply
MOV AL, 10
MOV BL, 20
MUL BL                ; AX = 200 (00C8H)

; 16-bit multiply
MOV AX, 1000H
MOV BX, 10H
MUL BX                ; DX:AX = 10000H (DX=0001H, AX=0000H)
```

### 3.3 Division

```
DIV - Unsigned Divide:
┌────────────────────────────────────────────────────────────────┐
│   DIV reg/mem8       ; AL ← AX / operand, AH ← remainder      │
│   DIV reg/mem16      ; AX ← DX:AX / operand, DX ← remainder   │
│                                                                │
│   Division by zero causes INT 0                               │
└────────────────────────────────────────────────────────────────┘

IDIV - Signed Divide:
┌────────────────────────────────────────────────────────────────┐
│   IDIV reg/mem8      ; AL ← AX / operand (signed), AH ← rem   │
│   IDIV reg/mem16     ; AX ← DX:AX / operand (signed), DX ← rem│
└────────────────────────────────────────────────────────────────┘

SIGN EXTENSION (for signed division):
CBW                   ; AL → AX (sign extend byte to word)
CWD                   ; AX → DX:AX (sign extend word to dword)

EXAMPLES:
; Unsigned 8-bit divide
MOV AX, 100           ; Dividend = 100
MOV BL, 7             ; Divisor = 7
DIV BL                ; AL = 14 (quotient), AH = 2 (remainder)

; Signed 16-bit divide
MOV AX, -100          ; Dividend
CWD                   ; Sign extend to DX:AX
MOV BX, 7
IDIV BX               ; AX = -14, DX = -2
```

### 3.4 BCD Arithmetic

```
BCD (Binary Coded Decimal) Adjustments:

DAA - Decimal Adjust After Addition (packed BCD):
; Adjusts AL after adding two packed BCD numbers
MOV AL, 25H           ; 25 in packed BCD
ADD AL, 18H           ; 25 + 18 = 3DH (wrong)
DAA                   ; AL = 43H (correct: 43)

DAS - Decimal Adjust After Subtraction (packed BCD):
; Adjusts AL after subtracting packed BCD numbers
MOV AL, 43H           ; 43 in packed BCD
SUB AL, 18H           ; 43 - 18 = 2BH (wrong)
DAS                   ; AL = 25H (correct: 25)

AAA - ASCII Adjust After Addition (unpacked BCD):
; Adjusts AL after adding unpacked BCD/ASCII digits

AAS - ASCII Adjust After Subtraction

AAM - ASCII Adjust After Multiplication:
; AH ← AL / 10, AL ← AL mod 10
MOV AL, 35            ; Product of 5 × 7
AAM                   ; AH = 3, AL = 5

AAD - ASCII Adjust Before Division:
; AL ← AH × 10 + AL, AH ← 0
MOV AX, 0305H         ; Unpacked 35
AAD                   ; AX = 0023H (35 decimal)
```

---

## 4. Logical Instructions

### 4.1 AND, OR, XOR, NOT

```
LOGICAL OPERATIONS:

AND dest, source      ; dest ← dest AND source
OR dest, source       ; dest ← dest OR source
XOR dest, source      ; dest ← dest XOR source
NOT dest              ; dest ← NOT dest (1's complement)
TEST dest, source     ; dest AND source (flags only, no store)

EXAMPLES:
AND AL, 0FH           ; Mask upper nibble (keep lower)
OR AL, 80H            ; Set bit 7
XOR AX, AX            ; Clear AX (fast zero)
NOT BX                ; Invert all bits
TEST AL, 01H          ; Test bit 0 (ZF=1 if bit is 0)


COMMON BIT OPERATIONS:

; Set bit n
OR AL, (1 SHL n)      ; OR AL, 08H sets bit 3

; Clear bit n
AND AL, NOT (1 SHL n) ; AND AL, F7H clears bit 3

; Toggle bit n
XOR AL, (1 SHL n)     ; XOR AL, 08H toggles bit 3

; Test bit n
TEST AL, (1 SHL n)    ; ZF=1 if bit n is 0
JNZ bit_is_set
```

### 4.2 Shift Instructions

```
SHIFT OPERATIONS:

SHL/SAL dest, count   ; Shift Left (Logical = Arithmetic)
SHR dest, count       ; Shift Right Logical (fill with 0)
SAR dest, count       ; Shift Right Arithmetic (fill with sign)

Count can be:
• 1 (immediate)
• CL (for variable count)

┌────────────────────────────────────────────────────────────────┐
│   SHL/SAL (Shift Left):                                        │
│                                                                 │
│      ┌─┬─┬─┬─┬─┬─┬─┬─┐                                         │
│   ◄──│ │ │ │ │ │ │ │ │◄── 0                                   │
│      └─┴─┴─┴─┴─┴─┴─┴─┘                                         │
│   CF                                                           │
│                                                                 │
│   SHL AL, 1         ; AL ← AL × 2                             │
│   SHL AX, CL        ; AX ← AX × 2^CL                          │
└────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────┐
│   SHR (Shift Right Logical):                                   │
│                                                                 │
│      ┌─┬─┬─┬─┬─┬─┬─┬─┐                                         │
│   0 ──►│ │ │ │ │ │ │ │ │──►                                    │
│      └─┴─┴─┴─┴─┴─┴─┴─┘   CF                                   │
│                                                                 │
│   SHR AL, 1         ; AL ← AL / 2 (unsigned)                  │
└────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────┐
│   SAR (Shift Right Arithmetic):                                │
│                                                                 │
│     ┌─┐ ┌─┬─┬─┬─┬─┬─┬─┬─┐                                     │
│     │S├─►│ │ │ │ │ │ │ │ │──►                                  │
│     └─┘ └─┴─┴─┴─┴─┴─┴─┴─┘   CF                                │
│     Sign bit preserved                                         │
│                                                                 │
│   SAR AL, 1         ; AL ← AL / 2 (signed)                    │
└────────────────────────────────────────────────────────────────┘
```

### 4.3 Rotate Instructions

```
ROTATE OPERATIONS:

ROL dest, count       ; Rotate Left
ROR dest, count       ; Rotate Right
RCL dest, count       ; Rotate Left through Carry
RCR dest, count       ; Rotate Right through Carry

┌────────────────────────────────────────────────────────────────┐
│   ROL (Rotate Left):                                           │
│                                                                 │
│      ┌──◄───────────────────────────┐                         │
│      │  ┌─┬─┬─┬─┬─┬─┬─┬─┐          │                         │
│      └──│ │ │ │ │ │ │ │ │──────────┘                         │
│         └─┴─┴─┴─┴─┴─┴─┴─┘                                     │
│          │                                                     │
│          └──► CF (copy of rotated bit)                        │
│                                                                 │
│   ROL AL, 1         ; Rotate left once                        │
└────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────┐
│   RCL (Rotate Left through Carry):                            │
│                                                                 │
│      ┌──◄──────────────────────────────┐                      │
│      │        ┌─┬─┬─┬─┬─┬─┬─┬─┐       │                      │
│      │  CF◄───│ │ │ │ │ │ │ │ │◄──────┘                      │
│      │        └─┴─┴─┴─┴─┴─┴─┴─┘                               │
│      └──────────────────────────────────►                      │
│                                                                 │
│   RCL AL, 1         ; 9-bit rotate including CF               │
└────────────────────────────────────────────────────────────────┘

EXAMPLES:
ROL AX, 1             ; Rotate AX left once
ROR BL, CL            ; Rotate BL right by CL times
RCL DX, 1             ; Include carry in rotation
```

---

## 5. String Instructions

```
STRING INSTRUCTIONS:

┌────────────────────────────────────────────────────────────────┐
│   Instruction  │  Operation                                    │
├────────────────┼──────────────────────────────────────────────┤
│   MOVSB/MOVSW  │  [ES:DI] ← [DS:SI], update SI, DI           │
│   CMPSB/CMPSW  │  Compare [DS:SI] with [ES:DI], update SI,DI │
│   SCASB/SCASW  │  Compare AL/AX with [ES:DI], update DI      │
│   LODSB/LODSW  │  AL/AX ← [DS:SI], update SI                 │
│   STOSB/STOSW  │  [ES:DI] ← AL/AX, update DI                 │
└────────────────┴──────────────────────────────────────────────┘

DIRECTION FLAG:
CLD                   ; Clear DF - auto-increment (forward)
STD                   ; Set DF - auto-decrement (backward)

REP PREFIXES:
REP                   ; Repeat while CX ≠ 0
REPE/REPZ             ; Repeat while equal (ZF=1) and CX ≠ 0
REPNE/REPNZ           ; Repeat while not equal (ZF=0) and CX ≠ 0


EXAMPLES:

; Copy 100 bytes from SOURCE to DEST
CLD                   ; Forward direction
MOV SI, OFFSET SOURCE
MOV DI, OFFSET DEST
MOV CX, 100
REP MOVSB             ; Copy CX bytes

; Find byte in string
MOV DI, OFFSET STRING
MOV CX, LENGTH STRING
MOV AL, 'A'           ; Byte to find
CLD
REPNE SCASB           ; Scan until AL found or CX=0
JE FOUND              ; ZF=1 if found

; Compare two strings
MOV SI, OFFSET STR1
MOV DI, OFFSET STR2
MOV CX, 10
CLD
REPE CMPSB            ; Compare until mismatch or CX=0
JE EQUAL              ; ZF=1 if all bytes matched
```

---

## 6. Control Transfer Instructions

### 6.1 Unconditional Jumps

```
JMP - Unconditional Jump:

JMP SHORT label       ; -128 to +127 bytes (2 bytes)
JMP NEAR label        ; Same segment, any offset (3 bytes)
JMP FAR label         ; Different segment (5 bytes)
JMP reg16             ; Jump to address in register
JMP mem16             ; Jump to address in memory
JMP mem32             ; Far jump to seg:off in memory

EXAMPLES:
JMP NEXT              ; Jump to label NEXT
JMP AX                ; Jump to address in AX
JMP DWORD PTR [BX]    ; Far jump using pointer at [BX]
```

### 6.2 Conditional Jumps

```
CONDITIONAL JUMPS (Jcc):

After CMP or TEST, jump based on flags:

┌───────────────────────────────────────────────────────────────────────────┐
│   UNSIGNED COMPARISONS (after CMP):                                       │
│   JA/JNBE    Jump if Above (CF=0 AND ZF=0)                               │
│   JAE/JNB    Jump if Above or Equal (CF=0)                               │
│   JB/JNAE/JC Jump if Below (CF=1)                                        │
│   JBE/JNA    Jump if Below or Equal (CF=1 OR ZF=1)                       │
├───────────────────────────────────────────────────────────────────────────┤
│   SIGNED COMPARISONS (after CMP):                                         │
│   JG/JNLE    Jump if Greater (ZF=0 AND SF=OF)                            │
│   JGE/JNL    Jump if Greater or Equal (SF=OF)                            │
│   JL/JNGE    Jump if Less (SF≠OF)                                        │
│   JLE/JNG    Jump if Less or Equal (ZF=1 OR SF≠OF)                       │
├───────────────────────────────────────────────────────────────────────────┤
│   FLAG-BASED:                                                              │
│   JE/JZ      Jump if Equal/Zero (ZF=1)                                   │
│   JNE/JNZ    Jump if Not Equal/Not Zero (ZF=0)                           │
│   JC         Jump if Carry (CF=1)                                         │
│   JNC        Jump if No Carry (CF=0)                                      │
│   JO         Jump if Overflow (OF=1)                                      │
│   JNO        Jump if No Overflow (OF=0)                                   │
│   JS         Jump if Sign (SF=1, negative)                               │
│   JNS        Jump if No Sign (SF=0, positive)                            │
│   JP/JPE     Jump if Parity Even (PF=1)                                  │
│   JNP/JPO    Jump if Parity Odd (PF=0)                                   │
└───────────────────────────────────────────────────────────────────────────┘

EXAMPLES:
CMP AX, BX
JE EQUAL              ; Jump if AX = BX
JA ABOVE              ; Jump if AX > BX (unsigned)
JG GREATER            ; Jump if AX > BX (signed)
```

### 6.3 Loops

```
LOOP INSTRUCTIONS:

LOOP label            ; CX--, jump if CX ≠ 0
LOOPE/LOOPZ label     ; CX--, jump if CX ≠ 0 AND ZF = 1
LOOPNE/LOOPNZ label   ; CX--, jump if CX ≠ 0 AND ZF = 0

JCXZ label            ; Jump if CX = 0

EXAMPLES:
; Count down loop
MOV CX, 10
LOOP_START:
    ; Loop body
    LOOP LOOP_START   ; Decrement CX, loop until CX=0

; Search with early exit
MOV CX, 100
MOV DI, OFFSET ARRAY
MOV AL, TARGET
SEARCH:
    SCASB             ; Compare AL with [ES:DI]
    LOOPNE SEARCH     ; Continue if not found and CX ≠ 0
    JE FOUND
    JMP NOT_FOUND
```

### 6.4 Procedures

```
CALL and RET:

CALL label            ; Push IP, jump to procedure
CALL FAR label        ; Push CS:IP, far jump
RET                   ; Pop IP, return
RET n                 ; Pop IP, then SP ← SP + n
RETF                  ; Pop IP and CS, far return
RETF n                ; Far return, then SP ← SP + n

EXAMPLE:
; Procedure call
CALL MY_PROC          ; Save return address, call procedure
; Continues here after RET

MY_PROC PROC
    PUSH AX           ; Save registers
    PUSH BX
    ; ... procedure body ...
    POP BX            ; Restore registers
    POP AX
    RET               ; Return to caller
MY_PROC ENDP
```

### 6.5 Interrupts

```
INTERRUPT INSTRUCTIONS:

INT n                 ; Software interrupt (n = 0-255)
INTO                  ; Interrupt on Overflow (INT 4 if OF=1)
IRET                  ; Return from interrupt

INT OPERATION:
1. Push FLAGS
2. Clear IF and TF
3. Push CS
4. Push IP
5. Load CS:IP from IVT[n × 4]

IRET OPERATION:
1. Pop IP
2. Pop CS
3. Pop FLAGS

EXAMPLES:
INT 21H               ; DOS function call
INT 10H               ; BIOS video services
INT 3                 ; Breakpoint (debugging)
```

---

## 7. Processor Control

```
PROCESSOR CONTROL INSTRUCTIONS:

FLAG MANIPULATION:
CLC                   ; Clear Carry (CF = 0)
STC                   ; Set Carry (CF = 1)
CMC                   ; Complement Carry (CF = NOT CF)
CLD                   ; Clear Direction (DF = 0)
STD                   ; Set Direction (DF = 1)
CLI                   ; Clear Interrupt (IF = 0)
STI                   ; Set Interrupt (IF = 1)

OTHER:
HLT                   ; Halt until interrupt
NOP                   ; No operation (delay)
WAIT                  ; Wait for coprocessor
LOCK                  ; Lock bus for next instruction
ESC                   ; Escape to coprocessor

EXAMPLES:
CLI                   ; Disable interrupts
; Critical section
STI                   ; Re-enable interrupts

CLD                   ; Forward string operations
REP MOVSB             ; Copy forward
```

---

## 📋 Summary Table

| Category | Key Instructions | Purpose |
|----------|-----------------|---------|
| Data Transfer | MOV, PUSH, POP, XCHG, LEA | Move data between locations |
| Arithmetic | ADD, SUB, MUL, DIV, INC, DEC | Mathematical operations |
| Logical | AND, OR, XOR, NOT, TEST | Bit manipulation |
| Shift/Rotate | SHL, SHR, SAR, ROL, ROR | Bit shifting |
| String | MOVS, CMPS, SCAS, LODS, STOS | Block operations |
| Control | JMP, Jcc, LOOP, CALL, RET | Program flow |
| Interrupt | INT, IRET | System calls |
| Processor | CLI, STI, HLT, NOP | CPU control |

---

## ❓ Quick Revision Questions

1. **What's the difference between MUL and IMUL?**
   <details>
   <summary>Show Answer</summary>
   MUL performs unsigned multiplication, IMUL performs signed multiplication. Both use AL/AX as implicit operand and store result in AX (8-bit) or DX:AX (16-bit). The difference is in how they interpret the operands and set flags.
   </details>

2. **How do you clear a register to zero quickly?**
   <details>
   <summary>Show Answer</summary>
   XOR AX, AX is the fastest method. It's smaller (2 bytes) and faster than MOV AX, 0 (3 bytes). It also clears CF and sets ZF, PF appropriately.
   </details>

3. **What's the difference between SHR and SAR?**
   <details>
   <summary>Show Answer</summary>
   SHR (Shift Right Logical) fills vacated bits with 0, used for unsigned division by 2. SAR (Shift Arithmetic Right) fills with the sign bit, preserving the sign for signed division by 2.
   </details>

4. **What registers are used by string instructions?**
   <details>
   <summary>Show Answer</summary>
   SI for source (DS:SI), DI for destination (ES:DI), CX for count with REP prefix. Direction Flag (DF) controls auto-increment (CLD) or auto-decrement (STD).
   </details>

5. **How does LOOP differ from a CMP/JNZ combination?**
   <details>
   <summary>Show Answer</summary>
   LOOP automatically decrements CX and jumps if CX ≠ 0, all in one instruction. CMP/JNZ requires separate decrement, compare, and jump. LOOP doesn't affect flags except for the jump condition.
   </details>

6. **What happens when INT instruction is executed?**
   <details>
   <summary>Show Answer</summary>
   FLAGS are pushed, IF and TF are cleared, CS and IP are pushed, then CS:IP is loaded from the Interrupt Vector Table at address n×4 where n is the interrupt number.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [4.4 Addressing Modes](04-addressing-modes.md) | [Unit 4 Index](README.md) | [4.6 Min/Max Mode](06-min-max-mode.md) |

---

*[← Previous: Addressing Modes](04-addressing-modes.md) | [Next: Minimum and Maximum Mode →](06-min-max-mode.md)*
