# Chapter 7.6: 8051 Addressing Modes

## 📚 Chapter Overview

This chapter covers all addressing modes available in the 8051 microcontroller. Understanding addressing modes is essential for efficient assembly programming as they determine how the CPU accesses operands for instructions.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Understand all 8051 addressing modes
- Select appropriate addressing mode for different operations
- Write efficient code using optimal addressing
- Access different memory spaces correctly

---

## 1. Addressing Modes Overview

### 1.1 Summary of 8051 Addressing Modes

```
8051 ADDRESSING MODES
━━━━━━━━━━━━━━━━━━━━━

┌─────────────────────┬────────────────────────────────────────────┐
│ Addressing Mode     │ Example                                    │
├─────────────────────┼────────────────────────────────────────────┤
│ Immediate           │ MOV A, #55H                                │
│ Register            │ MOV A, R0                                  │
│ Direct              │ MOV A, 30H                                 │
│ Register Indirect   │ MOV A, @R0                                 │
│ Indexed             │ MOVC A, @A+DPTR                           │
│ Bit                 │ SETB P1.0                                  │
│ Relative            │ SJMP label                                 │
│ Absolute            │ AJMP addr11                                │
│ Long                │ LJMP addr16                                │
└─────────────────────┴────────────────────────────────────────────┘

ADDRESSING MODE USAGE BY MEMORY SPACE:
────────────────────────────────────────

┌─────────────────────┬──────────────────────────────────────────┐
│ Memory Space        │ Addressing Modes Available               │
├─────────────────────┼──────────────────────────────────────────┤
│ Internal RAM 00-7FH │ Direct, Register, Register Indirect      │
│ Internal RAM 80-FFH │ Register Indirect only (8052 upper RAM)  │
│ SFRs (80-FFH)       │ Direct only                              │
│ External RAM        │ Register Indirect (MOVX)                 │
│ Program Memory      │ Indexed (MOVC)                           │
│ Bit Space           │ Bit addressing                           │
└─────────────────────┴──────────────────────────────────────────┘
```

---

## 2. Immediate Addressing

### 2.1 Immediate Addressing Explained

```
IMMEDIATE ADDRESSING MODE
━━━━━━━━━━━━━━━━━━━━━━━━━

The operand is a CONSTANT value embedded in the instruction.
Preceded by '#' symbol.

Format: MOV destination, #data

    OPCODE    IMMEDIATE DATA
    ┌────────┬────────────┐
    │  MOV   │  #55H      │
    └────────┴────────────┘
         │         │
         │         └─► Constant value (in instruction)
         └───────────► Operation

Examples:
─────────
    MOV A, #25H        ; Load A with constant 25H (37 decimal)
    MOV R3, #62H       ; Load R3 with 62H
    MOV DPTR, #1234H   ; Load DPTR with 16-bit constant
    MOV 30H, #55H      ; Load address 30H with 55H
    MOV @R0, #0AAH     ; Load [R0] with AAH
    
    ADD A, #10H        ; Add 10H to A
    SUBB A, #05H       ; Subtract 5 (with borrow)
    ANL A, #0FH        ; Mask upper nibble
    ORL P1, #80H       ; Set bit 7 of P1
    XRL A, #0FFH       ; Complement A (same as CPL A)

Key Points:
┌────────────────────────────────────────────────────────────────┐
│ • # symbol indicates immediate data                           │
│ • 8-bit immediate for most instructions                       │
│ • 16-bit immediate only for MOV DPTR, #data16                │
│ • Cannot use immediate as destination                         │
│ • Instruction length: 2 bytes (opcode + data)                │
│               or 3 bytes for 16-bit data                      │
└────────────────────────────────────────────────────────────────┘

INCORRECT USAGE:
    MOV #55H, A        ; WRONG! Immediate cannot be destination
    MOV R0, R1         ; WRONG! No register-to-register MOV
```

---

## 3. Register Addressing

### 3.1 Register Addressing Explained

```
REGISTER ADDRESSING MODE
━━━━━━━━━━━━━━━━━━━━━━━━

The operand is one of the registers R0-R7 (in current bank).
Register number encoded in opcode itself.

    OPCODE WITH REGISTER
    ┌───────────────────┐
    │  MOV A, Rn        │  (n = 0-7, encoded in opcode)
    └───────────────────┘

Examples:
─────────
    MOV A, R0          ; A = R0
    MOV R5, A          ; R5 = A
    ADD A, R3          ; A = A + R3
    SUBB A, R7         ; A = A - R7 - CY
    XCH A, R2          ; Exchange A with R2
    INC R4             ; R4 = R4 + 1
    DEC R6             ; R6 = R6 - 1
    DJNZ R1, label     ; Decrement R1, jump if not zero

Register Bank Selection:
───────────────────────

    PSW Bits: RS1 RS0
    
    RS1=0, RS0=0 → Bank 0 (R0-R7 = 00H-07H)  ◄── Default
    RS1=0, RS0=1 → Bank 1 (R0-R7 = 08H-0FH)
    RS1=1, RS0=0 → Bank 2 (R0-R7 = 10H-17H)
    RS1=1, RS0=1 → Bank 3 (R0-R7 = 18H-1FH)

Key Points:
┌────────────────────────────────────────────────────────────────┐
│ • Only R0-R7 can be used (not ACC or B directly)             │
│ • Register number encoded in opcode (3 bits)                  │
│ • Most compact instruction format (1 byte)                    │
│ • Actual RAM address depends on selected bank                 │
│ • A is implied in many instructions (ADD A, Rn)              │
└────────────────────────────────────────────────────────────────┘

IMPORTANT: MOV Rn, Rm is NOT valid!
    MOV R0, R1    ; WRONG! No direct reg-to-reg transfer
    
    ; Must use A as intermediate:
    MOV A, R1     ; A = R1
    MOV R0, A     ; R0 = A
```

---

## 4. Direct Addressing

### 4.1 Direct Addressing Explained

```
DIRECT ADDRESSING MODE
━━━━━━━━━━━━━━━━━━━━━━

The operand's ADDRESS is given directly in the instruction.
Accesses Internal RAM (00-7FH) or SFRs (80-FFH).

    OPCODE    DIRECT ADDRESS
    ┌────────┬────────────┐
    │  MOV   │    30H     │
    └────────┴────────────┘
         │         │
         │         └─► RAM address (not a value)
         └───────────► Operation

Examples:
─────────
    ; Internal RAM access
    MOV A, 30H         ; A = [30H] (contents of address 30H)
    MOV 40H, A         ; [40H] = A
    MOV 50H, 30H       ; [50H] = [30H] (direct-to-direct)
    
    ; SFR access
    MOV A, P1          ; A = Port 1 latch (P1 = 90H)
    MOV P2, #55H       ; P2 = 55H
    MOV TH0, #0FFH     ; Timer 0 High = FFH
    MOV A, PSW         ; A = Program Status Word
    
    ; Arithmetic with direct
    ADD A, 30H         ; A = A + [30H]
    ANL 30H, A         ; [30H] = [30H] AND A
    ORL P1, #80H       ; P1 = P1 OR 80H
    INC 30H            ; [30H] = [30H] + 1

ADDRESS RANGES:
───────────────

    00H - 7FH:  Internal RAM (direct and indirect)
    80H - FFH:  SFRs only! (not upper RAM)
    
    ┌────────────────────────────────────────────────────────┐
    │  MOV A, 30H   ; Reads from RAM address 30H             │
    │  MOV A, 90H   ; Reads P1 (SFR), NOT RAM address 90H!  │
    │  MOV A, ACC   ; Same as MOV A, 0E0H (ACC = E0H)       │
    └────────────────────────────────────────────────────────┘

Key Points:
┌────────────────────────────────────────────────────────────────┐
│ • 8-bit address specified in instruction                      │
│ • No # symbol (contrast with immediate)                       │
│ • 00-7FH → Internal RAM                                       │
│ • 80-FFH → SFRs (NOT upper RAM in 8052)                      │
│ • Instruction length: 2-3 bytes                               │
│ • Allows direct-to-direct transfers                           │
└────────────────────────────────────────────────────────────────┘

COMPARING IMMEDIATE VS DIRECT:
───────────────────────────────
    MOV A, #30H        ; A = 30H (the value 48 decimal)
    MOV A, 30H         ; A = contents of address 30H
    
    #30H = the number 48
    30H  = the contents of RAM address 48
```

---

## 5. Register Indirect Addressing

### 5.1 Register Indirect Addressing Explained

```
REGISTER INDIRECT ADDRESSING MODE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

The register contains the ADDRESS of the operand.
Uses @R0 or @R1 only (R2-R7 cannot be used indirectly).

    OPCODE    REGISTER (holds address)
    ┌────────┬────────────┐
    │  MOV   │   @R0      │
    └────────┴────────────┘
         │         │
         │         └─► R0 contains the address
         └───────────► Read from that address

Examples:
─────────
    ; R0/R1 as pointer to internal RAM
    MOV R0, #30H       ; R0 = 30H (pointer)
    MOV A, @R0         ; A = [30H] (value at address 30H)
    MOV @R0, #55H      ; [30H] = 55H
    
    ; Loop through memory
    MOV R0, #30H       ; Start address
    MOV R7, #10        ; Count
LOOP:
    MOV @R0, #0        ; Clear location
    INC R0             ; Next address
    DJNZ R7, LOOP      ; Repeat 10 times
    
    ; Accessing 8052 upper RAM (80H-FFH)
    MOV R0, #80H       ; Point to upper RAM
    MOV A, @R0         ; Read from upper RAM address 80H
                       ; (NOT SFR!)

MEMORY ACCESS COMPARISON:
────────────────────────

    MOV A, 90H         ; Direct: Reads SFR P1
    
    MOV R0, #90H
    MOV A, @R0         ; Indirect: Reads upper RAM 90H (8052)
                       ;           NOT P1!

External RAM Access (@R0, @R1, @DPTR):
─────────────────────────────────────

    ; 8-bit address (256 bytes)
    MOV R0, #40H
    MOVX A, @R0        ; Read external RAM [R0]
    MOVX @R0, A        ; Write to external RAM [R0]
    
    ; 16-bit address (64KB)
    MOV DPTR, #2000H
    MOVX A, @DPTR      ; Read external RAM [DPTR]
    MOVX @DPTR, A      ; Write to external RAM [DPTR]

Key Points:
┌────────────────────────────────────────────────────────────────┐
│ • Only R0 and R1 for internal RAM indirect                    │
│ • R0, R1, or DPTR for external RAM (MOVX)                    │
│ • @ symbol indicates indirect                                  │
│ • Essential for array/table operations                        │
│ • 8052: Only way to access upper 128 bytes RAM               │
└────────────────────────────────────────────────────────────────┘
```

---

## 6. Indexed Addressing

### 6.1 Indexed Addressing Explained

```
INDEXED ADDRESSING MODE
━━━━━━━━━━━━━━━━━━━━━━━

Used ONLY for reading program memory (ROM).
Effective address = Base + Offset

Two forms:
    MOVC A, @A+DPTR    ; EA = A + DPTR
    MOVC A, @A+PC      ; EA = A + PC

    Base Register    Offset
    ┌──────────────┬───────┐
    │  DPTR or PC  │   A   │
    └──────────────┴───────┘
          │            │
          └────────────┘
                 │
                 ▼
         Effective Address
                 │
                 ▼
         Code Memory Data

MOVC A, @A+DPTR:
────────────────

    ; Look-up table example
    MOV DPTR, #TABLE   ; Base address of table
    MOV A, #3          ; Offset (index into table)
    MOVC A, @A+DPTR    ; A = TABLE[3]
    
    ; ... code ...

TABLE:
    DB 0C0H            ; [0] = 0C0H
    DB 0F9H            ; [1] = 0F9H
    DB 0A4H            ; [2] = 0A4H
    DB 0B0H            ; [3] = 0B0H  ◄── This is read
    DB 99H             ; [4] = 99H

Effective Address = DPTR + A = TABLE + 3 = address of 0B0H

MOVC A, @A+PC:
─────────────

    ; Self-relative table access
        MOV A, #3          ; Offset
        INC A              ; Adjust for next instruction
        MOVC A, @A+PC      ; EA = PC + A
        SJMP CONTINUE      ; Skip over table data
TABLE:
        DB 0C0H, 0F9H, 0A4H, 0B0H, 99H
CONTINUE:
    ; ... more code ...

Note: PC is incremented BEFORE the addition,
      so A must be adjusted accordingly.

JMP @A+DPTR (Indexed Jump):
──────────────────────────

    ; Jump table (switch-case implementation)
    MOV DPTR, #JUMP_TABLE
    MOV A, INDEX       ; 0, 2, 4, 6... (must be even)
    JMP @A+DPTR        ; Jump to TABLE[INDEX]

JUMP_TABLE:
    AJMP CASE0         ; INDEX = 0
    AJMP CASE1         ; INDEX = 2
    AJMP CASE2         ; INDEX = 4
    AJMP CASE3         ; INDEX = 6

Note: AJMP is 2 bytes, so INDEX must be 0, 2, 4, 6...

Key Points:
┌────────────────────────────────────────────────────────────────┐
│ • ONLY for program memory access (ROM)                        │
│ • MOVC = Move Code (read-only)                                │
│ • Efficient for look-up tables, character sets               │
│ • JMP @A+DPTR for jump tables                                │
│ • A is both offset AND destination                           │
└────────────────────────────────────────────────────────────────┘
```

---

## 7. Bit Addressing

### 7.1 Bit Addressing Explained

```
BIT ADDRESSING MODE
━━━━━━━━━━━━━━━━━━━

Operates on individual bits in:
• Bit-addressable RAM (20H-2FH = bits 00H-7FH)
• Bit-addressable SFRs (addresses xx0H, xx8H)

    OPCODE    BIT ADDRESS
    ┌────────┬────────────┐
    │  SETB  │    P1.0    │
    └────────┴────────────┘
         │         │
         │         └─► Specific bit
         └───────────► Bit operation

BIT-ADDRESSABLE AREAS:
─────────────────────

1. Internal RAM (20H-2FH):
   Byte 20H = bits 00H-07H
   Byte 21H = bits 08H-0FH
   ...
   Byte 2FH = bits 78H-7FH
   
   Total: 128 bits

2. Bit-addressable SFRs:
   P0 (80H) = bits 80H-87H
   TCON (88H) = bits 88H-8FH
   P1 (90H) = bits 90H-97H
   SCON (98H) = bits 98H-9FH
   P2 (A0H) = bits A0H-A7H
   IE (A8H) = bits A8H-AFH
   P3 (B0H) = bits B0H-B7H
   IP (B8H) = bits B8H-BFH
   PSW (D0H) = bits D0H-D7H
   ACC (E0H) = bits E0H-E7H
   B (F0H) = bits F0H-F7H

BIT INSTRUCTIONS:
────────────────

    SETB bit          ; Set bit = 1
    CLR bit           ; Clear bit = 0
    CPL bit           ; Complement bit
    MOV C, bit        ; Copy bit to Carry
    MOV bit, C        ; Copy Carry to bit
    ANL C, bit        ; C = C AND bit
    ANL C, /bit       ; C = C AND (NOT bit)
    ORL C, bit        ; C = C OR bit
    ORL C, /bit       ; C = C OR (NOT bit)
    JB bit, label     ; Jump if bit = 1
    JNB bit, label    ; Jump if bit = 0
    JBC bit, label    ; Jump if bit = 1, then clear

Examples:
─────────
    SETB P1.0          ; Set Port 1 bit 0
    CLR TR0            ; Stop Timer 0 (TCON.4)
    JB RI, RECEIVE     ; Jump if serial receive flag
    JNB TF0, WAIT      ; Wait for Timer 0 overflow
    
    ; Boolean operations
    MOV C, P1.0        ; C = P1.0
    ANL C, P1.1        ; C = C AND P1.1
    MOV P2.0, C        ; P2.0 = result
    
    ; Using bit-addressable RAM
    SETB 00H           ; Set bit 0 of address 20H
    SETB 20H.0         ; Same thing, alternate notation
    JB 25H, LABEL      ; Jump if bit 25H is set

Key Points:
┌────────────────────────────────────────────────────────────────┐
│ • Carry (C) is used as 1-bit accumulator                      │
│ • Boolean processor for efficient I/O control                 │
│ • No need to read-modify-write entire byte                    │
│ • /bit means NOT bit (complement)                             │
│ • Very efficient for flags and control bits                   │
└────────────────────────────────────────────────────────────────┘
```

---

## 8. Relative Addressing

### 8.1 Relative Addressing Explained

```
RELATIVE ADDRESSING MODE
━━━━━━━━━━━━━━━━━━━━━━━━

Branch destination is PC + signed offset.
Used by: SJMP, conditional jumps (JZ, JNZ, JC, JNC, etc.)

    OPCODE    SIGNED OFFSET
    ┌────────┬────────────┐
    │  SJMP  │    rel     │
    └────────┴────────────┘
         │         │
         │         └─► -128 to +127 from PC+2
         └───────────► Short jump

Calculation:
    Target = PC + 2 + rel
    
    PC is address of current instruction
    +2 because offset is relative to NEXT instruction

Range:
    rel = -128 to +127
    Backward: up to 128 bytes before
    Forward: up to 127 bytes after

Examples:
─────────
    ; Forward jump
            ORG 0100H
    0100H   SJMP AHEAD     ; rel = +0AH
            ...
    010CH   AHEAD:         ; Target address

    ; Backward jump (loop)
    LOOP:   
            DEC R0
            JNZ LOOP       ; Jump back if not zero
            
    ; Conditional jumps
            JZ ZERO        ; Jump if A = 0
            JNZ NOT_ZERO   ; Jump if A ≠ 0
            JC CARRY       ; Jump if CY = 1
            JNC NO_CARRY   ; Jump if CY = 0
            JB P1.0, HIGH  ; Jump if bit set
            JNB P1.0, LOW  ; Jump if bit clear
            DJNZ R0, LOOP  ; Decrement and jump if not zero
            CJNE A, #55H, NOT_EQUAL  ; Compare and jump if not equal

RELATIVE ADDRESSING INSTRUCTIONS:
────────────────────────────────

    SJMP rel           ; Short Jump (unconditional)
    JZ rel             ; Jump if A = Zero
    JNZ rel            ; Jump if A ≠ Zero
    JC rel             ; Jump if Carry = 1
    JNC rel            ; Jump if Carry = 0
    JB bit, rel        ; Jump if Bit = 1
    JNB bit, rel       ; Jump if Bit = 0
    JBC bit, rel       ; Jump if Bit = 1, Clear bit
    DJNZ Rn, rel       ; Decrement Register, Jump if Not Zero
    DJNZ direct, rel   ; Decrement Direct, Jump if Not Zero
    CJNE A, #data, rel ; Compare A with Immediate, Jump if Not Equal
    CJNE A, direct, rel
    CJNE Rn, #data, rel
    CJNE @Ri, #data, rel

Key Points:
┌────────────────────────────────────────────────────────────────┐
│ • 2-byte instruction (opcode + offset)                        │
│ • Range: -128 to +127 bytes                                   │
│ • Assembler calculates offset automatically                   │
│ • Position-independent code (relocatable)                     │
│ • Error if target too far (use LJMP instead)                 │
└────────────────────────────────────────────────────────────────┘
```

---

## 9. Absolute and Long Addressing

### 9.1 Absolute Addressing (11-bit)

```
ABSOLUTE ADDRESSING MODE
━━━━━━━━━━━━━━━━━━━━━━━━

Used by AJMP and ACALL.
11-bit address allows jumps within same 2KB page.

Instruction Format:
    ┌───┬───┬───┬───┬───┬───┬───┬───┐ ┌───────────────┐
    │a10│a9 │a8 │ 0 │ 0 │ 0 │ 0 │ 1 │ │    a7-a0      │
    └───┴───┴───┴───┴───┴───┴───┴───┘ └───────────────┘
         Upper 3 bits      Opcode       Lower 8 bits

Target Address:
    Target[15:11] = PC[15:11]  (same page)
    Target[10:0]  = a10-a0     (from instruction)

Page Boundaries:
    Page 0: 0000H - 07FFH
    Page 1: 0800H - 0FFFH
    Page 2: 1000H - 17FFH
    ...
    Page 31: F800H - FFFFH

Examples:
─────────
            ORG 0500H
    0500H   AJMP 0600H     ; OK: Same page (0000-07FF)
    
            ORG 0700H
    0700H   AJMP 0100H     ; OK: Same page
    
            ORG 0800H
    0800H   AJMP 0100H     ; ERROR! Different page!
                           ; Use LJMP instead

Key Points:
┌────────────────────────────────────────────────────────────────┐
│ • 2-byte instruction                                          │
│ • Must stay within same 2KB page                              │
│ • Upper 5 bits of PC preserved                                │
│ • Smaller than LJMP (2 vs 3 bytes)                           │
│ • Assembler will error if out of page                        │
└────────────────────────────────────────────────────────────────┘
```

### 9.2 Long Addressing (16-bit)

```
LONG ADDRESSING MODE
━━━━━━━━━━━━━━━━━━━━

Used by LJMP and LCALL.
16-bit address allows jumps anywhere in 64KB.

Instruction Format:
    ┌───────────┐ ┌───────────┐ ┌───────────┐
    │  Opcode   │ │  addr15-8 │ │  addr7-0  │
    └───────────┘ └───────────┘ └───────────┘
       1 byte       1 byte        1 byte

Examples:
─────────
    LJMP 0000H         ; Jump to reset vector
    LJMP 1234H         ; Jump anywhere in 64KB
    LCALL SUBROUTINE   ; Call anywhere in 64KB
    
    ; Can jump across pages
            ORG 0700H
    0700H   LJMP 0900H     ; OK: Crosses page boundary

COMPARISON:
───────────

┌──────────┬───────┬──────────────────────────────────────┐
│ Instruct │ Bytes │ Range                                │
├──────────┼───────┼──────────────────────────────────────┤
│ SJMP     │   2   │ -128 to +127 from PC (256 byte)      │
│ AJMP     │   2   │ Same 2KB page (11-bit address)       │
│ LJMP     │   3   │ Full 64KB (16-bit address)           │
├──────────┼───────┼──────────────────────────────────────┤
│ ACALL    │   2   │ Same 2KB page                        │
│ LCALL    │   3   │ Full 64KB                            │
└──────────┴───────┴──────────────────────────────────────┘

Key Points:
┌────────────────────────────────────────────────────────────────┐
│ • 3-byte instruction                                          │
│ • Can reach any address in 64KB code space                    │
│ • Use when target might be far or unknown                     │
│ • Slightly slower than AJMP (3 vs 2 bytes)                   │
└────────────────────────────────────────────────────────────────┘
```

---

## 10. Addressing Mode Summary

```
COMPLETE ADDRESSING MODE SUMMARY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────────────────┬────────────────────────────────────────────────┐
│ Mode            │ Examples and Notes                             │
├─────────────────┼────────────────────────────────────────────────┤
│ IMMEDIATE       │ MOV A, #55H    ; Constant in instruction       │
│                 │ MOV DPTR, #1234H                               │
├─────────────────┼────────────────────────────────────────────────┤
│ REGISTER        │ MOV A, R0      ; R0-R7 in current bank         │
│                 │ ADD A, R3                                       │
├─────────────────┼────────────────────────────────────────────────┤
│ DIRECT          │ MOV A, 30H     ; Internal RAM 00-7FH           │
│                 │ MOV A, P1      ; SFRs 80-FFH                   │
├─────────────────┼────────────────────────────────────────────────┤
│ REG. INDIRECT   │ MOV A, @R0     ; R0/R1 as pointer              │
│ (Internal)      │ MOV @R1, A     ; Upper RAM (8052)              │
├─────────────────┼────────────────────────────────────────────────┤
│ REG. INDIRECT   │ MOVX A, @R0    ; 8-bit addr, external          │
│ (External)      │ MOVX A, @DPTR  ; 16-bit addr, external         │
├─────────────────┼────────────────────────────────────────────────┤
│ INDEXED         │ MOVC A, @A+DPTR ; Look-up table in ROM         │
│                 │ MOVC A, @A+PC   ; Self-relative table          │
│                 │ JMP @A+DPTR     ; Jump table                   │
├─────────────────┼────────────────────────────────────────────────┤
│ BIT             │ SETB P1.0      ; Individual bit operations     │
│                 │ CLR C          ; Carry flag                    │
│                 │ JB bit, label  ; Bit test                      │
├─────────────────┼────────────────────────────────────────────────┤
│ RELATIVE        │ SJMP label     ; -128 to +127 bytes            │
│                 │ JZ label       ; Conditional branches          │
│                 │ DJNZ R0, label ; Loop control                  │
├─────────────────┼────────────────────────────────────────────────┤
│ ABSOLUTE        │ AJMP label     ; Same 2KB page                 │
│                 │ ACALL label    ; 2-byte instruction            │
├─────────────────┼────────────────────────────────────────────────┤
│ LONG            │ LJMP label     ; Full 64KB range               │
│                 │ LCALL label    ; 3-byte instruction            │
└─────────────────┴────────────────────────────────────────────────┘
```

---

## 📋 Summary Table

| Mode | Symbol | Range/Target | Use Case |
|------|--------|--------------|----------|
| Immediate | #data | 8-bit constant | Loading constants |
| Register | Rn | R0-R7 | Fast register operations |
| Direct | addr | 00-7FH RAM, 80-FFH SFR | RAM/SFR access |
| Indirect | @R0, @R1 | Internal/External RAM | Arrays, pointers |
| Indexed | @A+DPTR | Code memory | Look-up tables |
| Bit | bit | 00-7FH, SFR bits | Flag/port control |
| Relative | rel | ±127 bytes | Short branches |
| Absolute | addr11 | 2KB page | Medium jumps |
| Long | addr16 | 64KB | Long jumps |

---

## ❓ Quick Revision Questions

1. **What is the difference between MOV A, #30H and MOV A, 30H?**
   <details>
   <summary>Show Answer</summary>
   MOV A, #30H loads the constant value 30H (48 decimal) into A (immediate addressing). MOV A, 30H loads the contents of RAM address 30H into A (direct addressing). The # indicates immediate data.
   </details>

2. **Which registers can be used for indirect addressing?**
   <details>
   <summary>Show Answer</summary>
   For internal RAM: Only R0 and R1 (@R0, @R1). For external RAM with 8-bit address: R0, R1 (MOVX @R0, @R1). For external RAM with 16-bit address: DPTR (MOVX @DPTR).
   </details>

3. **How does the 8051 access the upper 128 bytes RAM in 8052?**
   <details>
   <summary>Show Answer</summary>
   Using register indirect addressing only (@R0 or @R1). Direct addressing in the 80-FFH range accesses SFRs, not upper RAM. So MOV A, 90H reads SFR P1, while MOV R0,#90H / MOV A,@R0 reads upper RAM location 90H.
   </details>

4. **What is the range of SJMP instruction?**
   <details>
   <summary>Show Answer</summary>
   SJMP uses relative addressing with an 8-bit signed offset. Range is -128 to +127 bytes from the address of the next instruction (PC+2). If the target is farther, use AJMP or LJMP.
   </details>

5. **When would you use MOVC instead of MOVX?**
   <details>
   <summary>Show Answer</summary>
   Use MOVC to read from program memory (ROM) - for look-up tables stored with DB directives. Use MOVX to access external data memory (RAM) - for reading/writing data. MOVC is read-only; MOVX allows both read and write.
   </details>

6. **What is the difference between AJMP and LJMP?**
   <details>
   <summary>Show Answer</summary>
   AJMP is 2 bytes and can jump within the same 2KB page (upper 5 bits of PC unchanged). LJMP is 3 bytes and can jump anywhere in 64KB. Use AJMP for code optimization when staying in same page; use LJMP for safety or when crossing page boundaries.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [7.5 I/O Ports](05-io-ports.md) | [Unit 7 Index](README.md) | [Unit 8: 8051 Programming](../08-8051-Programming/README.md) |

---

*[← Previous: I/O Ports](05-io-ports.md) | [Next: Unit 8 - 8051 Programming →](../08-8051-Programming/README.md)*
