# Chapter 7.4: 8051 Registers and PSW

## 📚 Chapter Overview

This chapter provides detailed coverage of all 8051 registers including the Accumulator, B register, Data Pointer, Stack Pointer, and the Program Status Word (PSW) with its flag bits. Understanding registers is essential for efficient assembly programming.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Understand the function of each 8051 register
- Explain all PSW flag bits and their usage
- Use register banks effectively
- Apply register operations in programming

---

## 1. CPU Registers Overview

### 1.1 8051 Register Set

```
8051 PROGRAMMER'S MODEL
━━━━━━━━━━━━━━━━━━━━━━━

Main Registers:
┌───────────────────────────────────────────────────────────────┐
│                                                               │
│  ┌─────────────────────────────────────────────┐              │
│  │  ACCUMULATOR (ACC or A)        8-bit  E0H   │  ◄── Primary │
│  └─────────────────────────────────────────────┘              │
│                                                               │
│  ┌─────────────────────────────────────────────┐              │
│  │  B REGISTER                    8-bit  F0H   │  ◄── MUL/DIV │
│  └─────────────────────────────────────────────┘              │
│                                                               │
│  ┌─────────────────────────────────────────────┐              │
│  │  PROGRAM STATUS WORD (PSW)     8-bit  D0H   │  ◄── Flags   │
│  └─────────────────────────────────────────────┘              │
│                                                               │
│  ┌─────────────────────────────────────────────┐              │
│  │  STACK POINTER (SP)            8-bit  81H   │  ◄── Stack   │
│  └─────────────────────────────────────────────┘              │
│                                                               │
│  ┌──────────────────────┬──────────────────────┐              │
│  │  DPH (Data Ptr High) │  DPL (Data Ptr Low)  │  ◄── 16-bit  │
│  │      8-bit  83H      │      8-bit  82H      │     DPTR     │
│  └──────────────────────┴──────────────────────┘              │
│                                                               │
│  ┌─────────────────────────────────────────────┐              │
│  │  PROGRAM COUNTER (PC)          16-bit       │  ◄── Address │
│  └─────────────────────────────────────────────┘   (not SFR)  │
│                                                               │
│  Register Banks (R0-R7):                                      │
│  ┌────┬────┬────┬────┬────┬────┬────┬────┐                   │
│  │ R0 │ R1 │ R2 │ R3 │ R4 │ R5 │ R6 │ R7 │ ◄── 4 banks      │
│  └────┴────┴────┴────┴────┴────┴────┴────┘    in RAM        │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

---

## 2. Accumulator (ACC)

### 2.1 Accumulator Details

```
ACCUMULATOR (ACC) - Address E0H
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

The ACC is the most important register in the 8051.

Bit Structure:
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│ D7  │ D6  │ D5  │ D4  │ D3  │ D2  │ D1  │ D0  │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
  E7H   E6H   E5H   E4H   E3H   E2H   E1H   E0H  ◄── Bit addresses

Features:
┌────────────────────────────────────────────────────────────────┐
│ • Primary register for arithmetic and logic operations         │
│ • All arithmetic results go here (ADD, SUBB, MUL, DIV)        │
│ • Boolean operations with Carry flag                           │
│ • Data transfer hub                                            │
│ • Bit-addressable (ACC.0 through ACC.7)                       │
│ • Can be referred to as A or ACC in instructions              │
│ • Reset value: 00H                                             │
└────────────────────────────────────────────────────────────────┘

Common Operations:
──────────────────
    MOV A, #55H       ; Load immediate value
    MOV A, R0         ; Copy from register
    MOV A, 30H        ; Copy from direct address
    MOV A, @R0        ; Copy from indirect address
    
    ADD A, R1         ; Add R1 to A
    SUBB A, #10H      ; Subtract with borrow
    ANL A, B          ; AND with B register
    ORL A, P1         ; OR with Port 1
    XRL A, #0FFH      ; XOR (complement)
    
    RL A              ; Rotate left
    RR A              ; Rotate right
    RLC A             ; Rotate left through Carry
    RRC A             ; Rotate right through Carry
    SWAP A            ; Swap nibbles
    
    CLR A             ; Clear accumulator (A = 0)
    CPL A             ; Complement (1s complement)
```

### 2.2 A vs ACC Notation

```
A vs ACC USAGE
━━━━━━━━━━━━━━

Both refer to the same register, but usage differs:

┌────────────────────────────────────────────────────────────────┐
│  Instruction Type      │  Use A      │  Use ACC              │
├────────────────────────┼─────────────┼───────────────────────┤
│  Most instructions     │  MOV A, #5  │  -                    │
│  Data movement         │  ADD A, R0  │  -                    │
│  PUSH/POP              │  -          │  PUSH ACC, POP ACC    │
│  Direct addressing     │  -          │  MOV 30H, ACC         │
│  Bit operations        │  -          │  JB ACC.0, label      │
└────────────────────────┴─────────────┴───────────────────────┘

Examples:
─────────
    MOV A, #55H        ; Use A
    PUSH ACC           ; Use ACC (required for stack)
    POP ACC            ; Use ACC
    JB ACC.7, NEGATIVE ; Test MSB of accumulator
    MOV 40H, ACC       ; Store A to address 40H (ACC form)
    MOV B, A           ; A is fine here too
```

---

## 3. B Register

### 3.1 B Register Details

```
B REGISTER - Address F0H
━━━━━━━━━━━━━━━━━━━━━━━━

Bit Structure:
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│ D7  │ D6  │ D5  │ D4  │ D3  │ D2  │ D1  │ D0  │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
  F7H   F6H   F5H   F4H   F3H   F2H   F1H   F0H  ◄── Bit addresses

Features:
┌────────────────────────────────────────────────────────────────┐
│ • Used as second operand in MUL and DIV                       │
│ • Can be used as general-purpose register                     │
│ • Bit-addressable (B.0 through B.7)                           │
│ • Reset value: 00H                                             │
└────────────────────────────────────────────────────────────────┘

MULTIPLICATION (MUL AB):
──────────────────────
    A × B = 16-bit product
    
    Result:  B (High byte) : A (Low byte)
    
    Example:
    MOV A, #25H       ; A = 25H (37 decimal)
    MOV B, #65H       ; B = 65H (101 decimal)
    MUL AB            ; 37 × 101 = 3737 = 0E99H
                      ; B = 0EH (high byte)
                      ; A = 99H (low byte)
    
    Flags:
    • OV = 1 if product > FFH (result needs B)
    • CY = 0 (always cleared)

DIVISION (DIV AB):
────────────────
    A ÷ B = Quotient (A) and Remainder (B)
    
    Example:
    MOV A, #0FBH      ; A = FBH (251 decimal)
    MOV B, #12H       ; B = 12H (18 decimal)
    DIV AB            ; 251 ÷ 18 = 13 remainder 17
                      ; A = 0DH (quotient = 13)
                      ; B = 11H (remainder = 17)
    
    Flags:
    • OV = 1 if B = 0 (divide by zero)
    • CY = 0 (always cleared)
```

---

## 4. Program Status Word (PSW)

### 4.1 PSW Structure

```
PROGRAM STATUS WORD (PSW) - Address D0H
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│ CY  │ AC  │ F0  │ RS1 │ RS0 │ OV  │  -  │  P  │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
  D7    D6    D5    D4    D3    D2    D1    D0
  
  Bit  Symbol  Name                    Function
  ═══  ══════  ══════════════════════  ═══════════════════════════
  D7    CY     Carry Flag              Carry out from bit 7
  D6    AC     Auxiliary Carry         Carry out from bit 3
  D5    F0     Flag 0                  User-defined flag
  D4    RS1    Register Bank Select 1  \
  D3    RS0    Register Bank Select 0  / Select active bank
  D2    OV     Overflow Flag           Overflow in signed operation
  D1    -      Reserved                (Sometimes F1 in variants)
  D0    P      Parity Flag             Parity of accumulator

Reset Value: 00H
```

### 4.2 Flag Bit Details

```
CARRY FLAG (CY) - D7H
━━━━━━━━━━━━━━━━━━━━━

Set when:
• Carry out of bit 7 in addition
• Borrow into bit 7 in subtraction
• Shift/rotate operations

    Example: ADD
    ──────────
       A = 0F5H (11110101)
     + R0 = 0BH (00001011)
     ─────────────────────
       A = 100H = 00H with CY=1

Instructions affecting CY:
    ADD, ADDC, SUBB         - Based on result
    RLC, RRC                - Shifts through carry
    SETB C / CLR C          - Direct manipulation
    MUL AB, DIV AB          - Cleared
    ANL C, bit / ORL C, bit - Boolean operations


AUXILIARY CARRY (AC) - D6H
━━━━━━━━━━━━━━━━━━━━━━━━━━

Set when carry out of bit 3 to bit 4 (lower nibble overflow)
Used for BCD (Binary Coded Decimal) operations with DA A

    Example:
       A = 2EH (0010 1110)
     + R0 = 74H (0111 0100)
     ────────────────────
       A = A2H (1010 0010)
    
    Lower nibble: E + 4 = 12 (carry from bit 3)
    AC = 1


FLAG 0 (F0) - D5H
━━━━━━━━━━━━━━━━

• User-definable flag
• Can be set, cleared, or tested by program
• No automatic effect on CPU

    Example:
    SETB F0          ; Set flag
    JB F0, LABEL     ; Jump if F0 is set
    CLR F0           ; Clear flag


REGISTER BANK SELECT (RS1:RS0) - D4H:D3H
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────┬─────┬──────┬───────────────┐
│ RS1 │ RS0 │ Bank │ RAM Address   │
├─────┼─────┼──────┼───────────────┤
│  0  │  0  │  0   │ 00H - 07H     │ ◄── Default after reset
│  0  │  1  │  1   │ 08H - 0FH     │
│  1  │  0  │  2   │ 10H - 17H     │
│  1  │  1  │  3   │ 18H - 1FH     │
└─────┴─────┴──────┴───────────────┘

Bank Switching:
    ; Switch to Bank 1
    SETB RS0         ; RS1=0, RS0=1
    CLR RS1
    ; or simply:
    MOV PSW, #08H    ; Direct load
    
    ; Switch to Bank 3
    SETB RS0
    SETB RS1
    ; or:
    MOV PSW, #18H


OVERFLOW FLAG (OV) - D2H
━━━━━━━━━━━━━━━━━━━━━━━━

Set when signed arithmetic overflows (result > +127 or < -128)

Formula: OV = C7 XOR C6
         (Carry into bit 7) XOR (Carry out of bit 7)

    Example (Signed Overflow):
    ─────────────────────────
       A = +80H (+128 in unsigned, but -128 in signed!)
       No, let's use: A = 7FH (+127)
     + R0 = 01H (+1)
     ────────────────────
       A = 80H (-128 in signed!)  ◄── Should be +128, but 8-bit signed max is +127
       OV = 1 (overflow occurred)
       CY = 0 (no unsigned overflow)

    Unsigned: No problem (80H = 128)
    Signed: WRONG! +127 + 1 ≠ -128

Also set by:
    MUL AB: OV = 1 if product > 255
    DIV AB: OV = 1 if B = 0 (divide by zero)


PARITY FLAG (P) - D0H
━━━━━━━━━━━━━━━━━━━━━

Set if accumulator has ODD number of 1s

    A = 53H = 01010011 (5 ones) → P = 1 (odd)
    A = 55H = 01010101 (4 ones) → P = 0 (even)

• Automatically updated after any instruction that modifies A
• Used for parity checking in serial communication
• Cannot be set/cleared directly
```

### 4.3 PSW Summary Diagram

```
PSW BIT OPERATIONS
━━━━━━━━━━━━━━━━━━

┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  CY (D7) ───► Carry/Borrow in arithmetic                       │
│              Shift bit in rotate instructions                  │
│              Boolean result in bit operations                  │
│                                                                 │
│  AC (D6) ───► BCD arithmetic adjustment                        │
│              Indicates carry between nibbles                   │
│                                                                 │
│  F0 (D5) ───► General purpose user flag                        │
│              No CPU function - for programmer use              │
│                                                                 │
│  RS1:RS0 ──► Register bank selection                           │
│  (D4:D3)    Determines which R0-R7 bank is active              │
│                                                                 │
│  OV (D2) ───► Signed overflow detection                        │
│              MUL result > 255                                  │
│              DIV by zero                                       │
│                                                                 │
│  -  (D1) ───► Reserved (sometimes F1)                          │
│                                                                 │
│  P  (D0) ───► Accumulator parity                               │
│              Read-only, auto-updated                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Data Pointer (DPTR)

### 5.1 DPTR Details

```
DATA POINTER (DPTR) - 16-bit Register
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

DPTR is the only 16-bit register accessible to the programmer
(PC is 16-bit but not directly accessible)

Structure:
┌────────────────────┬────────────────────┐
│    DPH (83H)       │    DPL (82H)       │
│    High Byte       │    Low Byte        │
├────────────────────┴────────────────────┤
│           16-bit DPTR (0000H-FFFFH)     │
└─────────────────────────────────────────┘

Features:
┌────────────────────────────────────────────────────────────────┐
│ • 16-bit register for addressing external memory              │
│ • Can address full 64KB external code or data space           │
│ • Used with MOVX for external RAM                             │
│ • Used with MOVC for look-up tables in ROM                    │
│ • Can be loaded as 16-bit or as individual bytes              │
│ • Reset value: 0000H                                          │
└────────────────────────────────────────────────────────────────┘

DPTR Operations:
────────────────
    ; 16-bit load
    MOV DPTR, #1234H     ; DPH=12H, DPL=34H
    
    ; Individual byte access
    MOV DPL, #00H        ; Set low byte
    MOV DPH, #20H        ; Set high byte = DPTR=2000H
    
    ; Increment (16-bit)
    INC DPTR             ; DPTR = DPTR + 1
                         ; Note: No DEC DPTR instruction!
    
    ; External RAM access
    MOV DPTR, #8000H
    MOV A, #55H
    MOVX @DPTR, A        ; Write 55H to external address 8000H
    MOVX A, @DPTR        ; Read from external address 8000H
    
    ; Look-up table access
    MOV DPTR, #TABLE
    MOV A, #3            ; Offset
    MOVC A, @A+DPTR      ; Read TABLE[3]
```

---

## 6. Stack Pointer (SP)

### 6.1 SP Details

```
STACK POINTER (SP) - Address 81H
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Features:
┌────────────────────────────────────────────────────────────────┐
│ • 8-bit register pointing to top of stack                     │
│ • Stack is in internal RAM only                               │
│ • Reset value: 07H                                            │
│ • Stack grows UPWARD (increasing address)                     │
│ • PUSH: SP+1, then store                                      │
│ • POP: Read, then SP-1                                        │
│ • CALL: PC pushed (2 bytes)                                   │
│ • RET: PC popped                                              │
└────────────────────────────────────────────────────────────────┘

STACK OPERATIONS:
────────────────

    PUSH Operation:
    ┌─────────────────────────────────┐
    │ SP = SP + 1                     │
    │ [SP] = data                     │
    └─────────────────────────────────┘
    
    POP Operation:
    ┌─────────────────────────────────┐
    │ data = [SP]                     │
    │ SP = SP - 1                     │
    └─────────────────────────────────┘

    CALL Operation:
    ┌─────────────────────────────────┐
    │ SP = SP + 1                     │
    │ [SP] = PC low byte              │
    │ SP = SP + 1                     │
    │ [SP] = PC high byte             │
    │ PC = subroutine address         │
    └─────────────────────────────────┘

    RET Operation:
    ┌─────────────────────────────────┐
    │ PC high = [SP]                  │
    │ SP = SP - 1                     │
    │ PC low = [SP]                   │
    │ SP = SP - 1                     │
    └─────────────────────────────────┘

Stack Initialization Example:
────────────────────────────
    ; Avoid conflict with register banks
    MOV SP, #2FH         ; Stack starts above bit-addressable area
                         ; First PUSH goes to 30H
    
    ; or for more stack space:
    MOV SP, #5FH         ; Leaves 00H-5FH for variables
                         ; Stack uses 60H-7FH (32 bytes)

STACK VISUALIZATION:
───────────────────

    MOV SP, #2FH
    MOV A, #11H
    MOV B, #22H
    PUSH ACC
    PUSH B
    
    Before PUSH:          After PUSH ACC:       After PUSH B:
    
    7FH ┌─────┐           7FH ┌─────┐           7FH ┌─────┐
        │     │               │     │               │     │
    31H │     │           31H │     │           31H │ 22H │◄─SP=31H
    30H │     │           30H │ 11H │◄─SP=30H   30H │ 11H │
    2FH │     │◄─SP=2FH   2FH │     │           2FH │     │
        └─────┘               └─────┘               └─────┘
```

---

## 7. Program Counter (PC)

```
PROGRAM COUNTER (PC) - 16-bit
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

The PC is NOT an SFR - it cannot be read/written directly

Features:
┌────────────────────────────────────────────────────────────────┐
│ • 16-bit counter holding address of next instruction          │
│ • Automatically incremented after fetching each byte          │
│ • Modified by: LJMP, AJMP, SJMP, JMP @A+DPTR                  │
│ • Saved/restored by: LCALL, ACALL, RET, RETI                  │
│ • Range: 0000H to FFFFH (64KB code space)                     │
│ • Reset value: 0000H                                          │
└────────────────────────────────────────────────────────────────┘

PC MODIFICATIONS:
────────────────

    LJMP addr16      ; PC = 16-bit address (anywhere in 64KB)
    AJMP addr11      ; PC = (PC+2)[15:11] + addr11 (2KB page)
    SJMP rel         ; PC = PC + 2 + rel (-128 to +127)
    JMP @A+DPTR      ; PC = A + DPTR (indirect jump)
    
    LCALL addr16     ; Push PC, then PC = addr16
    ACALL addr11     ; Push PC, then PC = page address
    RET              ; Pop PC (return from subroutine)
    RETI             ; Pop PC and restore interrupt enable

Reading PC (indirectly):
───────────────────────
    ; The only way to get PC value is with MOVC
    MOVC A, @A+PC    ; Uses PC as base address
    
    ; Example: Self-locating code
        MOV A, #0          ; Offset = 0
        MOVC A, @A+PC      ; Read next byte
    HERE:
        DB 55H             ; Data byte
        ; A now contains 55H
        ; (But PC was PC of MOVC + 1)
```

---

## 8. Register Banks (R0-R7)

```
REGISTER BANKS
━━━━━━━━━━━━━━

Four banks of 8 registers each (R0 through R7)

RAM Layout:
┌───────────────────────────────────────────────────────────────┐
│ Address │ Bank 0 │ Bank 1 │ Bank 2 │ Bank 3                  │
├─────────┼────────┼────────┼────────┼────────                  │
│   00H   │   R0   │   -    │   -    │   -                      │
│   01H   │   R1   │   -    │   -    │   -                      │
│   02H   │   R2   │   -    │   -    │   -                      │
│   03H   │   R3   │   -    │   -    │   -                      │
│   04H   │   R4   │   -    │   -    │   -                      │
│   05H   │   R5   │   -    │   -    │   -                      │
│   06H   │   R6   │   -    │   -    │   -                      │
│   07H   │   R7   │   -    │   -    │   -                      │
├─────────┼────────┼────────┼────────┼────────                  │
│   08H   │   -    │   R0   │   -    │   -                      │
│   09H   │   -    │   R1   │   -    │   -                      │
│   ...   │   ...  │   ...  │   ...  │   ...                    │
│   0FH   │   -    │   R7   │   -    │   -                      │
├─────────┼────────┼────────┼────────┼────────                  │
│   10H   │   -    │   -    │   R0   │   -                      │
│   ...   │   ...  │   ...  │   ...  │   ...                    │
│   17H   │   -    │   -    │   R7   │   -                      │
├─────────┼────────┼────────┼────────┼────────                  │
│   18H   │   -    │   -    │   -    │   R0                     │
│   ...   │   ...  │   ...  │   ...  │   ...                    │
│   1FH   │   -    │   -    │   -    │   R7                     │
└───────────────────────────────────────────────────────────────┘

Bank Selection:
    ; Select Bank 0 (default)
    CLR RS0
    CLR RS1
    
    ; Select Bank 1
    SETB RS0
    CLR RS1
    
    ; Select Bank 2
    CLR RS0
    SETB RS1
    
    ; Select Bank 3
    SETB RS0
    SETB RS1

Register Operations:
    MOV R0, #55H      ; R0 = 55H (in active bank)
    MOV A, R3         ; A = R3
    ADD A, R7         ; A = A + R7
    INC R2            ; R2 = R2 + 1
    DEC R5            ; R5 = R5 - 1
    MOV R1, A         ; R1 = A

Indirect Addressing (R0 and R1 only):
    MOV R0, #30H      ; R0 = pointer to address 30H
    MOV @R0, #55H     ; [30H] = 55H
    MOV A, @R0        ; A = [30H]
    
    MOV R1, #40H
    MOV @R1, A        ; [40H] = A

Usage in Interrupts:
    ; Use different banks for main program and interrupts
    ; to avoid saving/restoring registers
    
    ; Main program uses Bank 0
    CLR RS0
    CLR RS1
    
    ; Timer ISR uses Bank 1
    TIMER_ISR:
        PUSH PSW          ; Save current bank
        SETB RS0          ; Switch to Bank 1
        ; ... ISR code using R0-R7 ...
        POP PSW           ; Restore original bank
        RETI
```

---

## 📋 Summary Table

| Register | Address | Size | Reset Value | Function |
|----------|---------|------|-------------|----------|
| ACC (A) | E0H | 8-bit | 00H | Main working register |
| B | F0H | 8-bit | 00H | MUL/DIV, general purpose |
| PSW | D0H | 8-bit | 00H | Status flags |
| SP | 81H | 8-bit | 07H | Stack pointer |
| DPL | 82H | 8-bit | 00H | Data pointer low |
| DPH | 83H | 8-bit | 00H | Data pointer high |
| PC | N/A | 16-bit | 0000H | Program counter |
| R0-R7 | 00-1FH | 8-bit | Undefined | General registers |

---

## ❓ Quick Revision Questions

1. **What is the difference between A and ACC in 8051 assembly?**
   <details>
   <summary>Show Answer</summary>
   Both refer to the same Accumulator register at E0H. Use 'A' for most instructions (MOV A, ADD A), but use 'ACC' for PUSH/POP operations (PUSH ACC, POP ACC) and for bit addressing (JB ACC.7). The assembler requires 'ACC' when the register must be specified as a direct address.
   </details>

2. **What are the PSW flags and their bit positions?**
   <details>
   <summary>Show Answer</summary>
   CY(D7)-Carry, AC(D6)-Auxiliary Carry, F0(D5)-User Flag, RS1(D4)-Bank Select 1, RS0(D3)-Bank Select 0, OV(D2)-Overflow, Reserved(D1), P(D0)-Parity. PSW is at address D0H and is bit-addressable.
   </details>

3. **How do you select Register Bank 2?**
   <details>
   <summary>Show Answer</summary>
   Set RS1=1 and RS0=0 in PSW: CLR RS0, SETB RS1, or MOV PSW, #10H. This makes R0-R7 refer to addresses 10H-17H in internal RAM.
   </details>

4. **What happens when you execute MUL AB with A=25H and B=65H?**
   <details>
   <summary>Show Answer</summary>
   25H × 65H = 37 × 101 = 3737 decimal = 0E99H. After MUL AB: A=99H (low byte), B=0EH (high byte), OV=1 (product > 255), CY=0 (always cleared by MUL).
   </details>

5. **Why is the reset value of SP = 07H considered problematic?**
   <details>
   <summary>Show Answer</summary>
   SP=07H means the first PUSH stores at address 08H, which is Bank 1's R0. If you switch to Bank 1, the stack will corrupt registers. Solution: Initialize SP to 2FH or higher to use general-purpose RAM (30H-7FH) for the stack.
   </details>

6. **What is the Parity flag and how is it updated?**
   <details>
   <summary>Show Answer</summary>
   The P flag (D0 of PSW) is set to 1 if the Accumulator contains an odd number of 1-bits, or 0 if even. It's automatically updated after any instruction that modifies ACC and cannot be directly set or cleared by the programmer.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [7.3 Memory Organization](03-memory-organization.md) | [Unit 7 Index](README.md) | [7.5 I/O Ports](05-io-ports.md) |

---

*[← Previous: Memory Organization](03-memory-organization.md) | [Next: I/O Ports →](05-io-ports.md)*
