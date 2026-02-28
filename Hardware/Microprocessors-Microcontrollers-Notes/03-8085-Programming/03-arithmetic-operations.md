# Chapter 3.3: Arithmetic Operations

## 📚 Chapter Overview

This chapter covers arithmetic operations in 8085 assembly language, including addition, subtraction, multiplication, and division for both 8-bit and 16-bit operands.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Perform 8-bit and 16-bit addition and subtraction
- Handle carry and borrow in multi-byte arithmetic
- Implement multiplication algorithms
- Implement division algorithms
- Perform BCD arithmetic using DAA

---

## 1. 8-Bit Addition

### 1.1 Basic Addition

```
8-BIT ADDITION:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   ADD r     A = A + r                                           │
│   ADD M     A = A + Memory[HL]                                  │
│   ADI data  A = A + data (immediate)                            │
│   ADC r     A = A + r + CY  (with carry)                        │
│   ACI data  A = A + data + CY                                   │
│                                                                  │
│   FLAGS AFFECTED: S, Z, AC, P, CY                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

EXAMPLE: Add two 8-bit numbers
───────────────────────────────────────────────────────────────────

; Add 45H and 32H
        MVI A, 45H      ; A = 45H
        MVI B, 32H      ; B = 32H
        ADD B           ; A = 45H + 32H = 77H
        
        HLT

; Trace:
;   0100 0101 (45H)
; + 0011 0010 (32H)
; ───────────
;   0111 0111 (77H)
; CY = 0, Z = 0, S = 0


EXAMPLE: Add numbers from memory
───────────────────────────────────────────────────────────────────

; Add contents of 2000H and 2001H, store at 2002H
        LDA 2000H       ; A = first number
        LXI H, 2001H    ; HL points to second number
        ADD M           ; A = A + Memory[HL]
        STA 2002H       ; Store result
        
        HLT
```

### 1.2 Addition with Carry

```
ADDITION WITH CARRY FLAG:

When sum > FFH (255), carry is generated

EXAMPLE: 85H + 9CH = 121H (with carry)
───────────────────────────────────────────────────────────────────

        MVI A, 85H      ; A = 85H
        ADI 9CH         ; A = 85H + 9CH
        
; Calculation:
;   1000 0101 (85H = 133)
; + 1001 1100 (9CH = 156)
; ───────────
; 1 0010 0001 (121H = 289)
;   └── A = 21H, CY = 1

; Result in A = 21H, CY = 1
; True result = 100H + 21H = 121H


STORING 8-BIT RESULT WITH CARRY:
───────────────────────────────────────────────────────────────────

; Add two numbers, store 16-bit result
        LDA 2000H       ; First number
        MOV B, A
        LDA 2001H       ; Second number
        ADD B           ; Add (may set carry)
        STA 2002H       ; Store low byte of result
        
        MVI A, 00H      ; A = 0
        ADC A           ; A = 0 + 0 + CY = CY
        STA 2003H       ; Store carry as high byte
        
        HLT

; If 85H + 9CH:
;   M[2002H] = 21H (low byte)
;   M[2003H] = 01H (carry/high byte)
```

---

## 2. 16-Bit Addition

### 2.1 Using DAD Instruction

```
DAD INSTRUCTION (Double Add):

DAD rp    HL = HL + rp

Only affects CY flag (no other flags!)

EXAMPLE: Add two 16-bit numbers
───────────────────────────────────────────────────────────────────

; Add 1234H + 5678H
        LXI H, 1234H    ; HL = 1234H
        LXI B, 5678H    ; BC = 5678H
        DAD B           ; HL = HL + BC = 68ACH
        
        ; Result: HL = 68ACH

; Verification:
;   1234H = 4660
; + 5678H = 22136
; ─────────────
;   68ACH = 26796 ✓
```

### 2.2 Using ADD/ADC (Byte-by-Byte)

```
16-BIT ADDITION USING 8-BIT OPERATIONS:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Number 1 (16-bit) at 2000H-2001H                              │
│   Number 2 (16-bit) at 2002H-2003H                              │
│   Result   (16-bit) at 2004H-2005H                              │
│                                                                  │
│   Low byte:  M[2000H] + M[2002H] → M[2004H]                     │
│   High byte: M[2001H] + M[2003H] + CY → M[2005H]                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

PROGRAM:
───────────────────────────────────────────────────────────────────

        ORG 0100H
        
        ; Add low bytes
        LDA 2000H       ; Low byte of first number
        LXI H, 2002H
        ADD M           ; Add low byte of second number
        STA 2004H       ; Store low byte of result
        
        ; Add high bytes with carry
        LDA 2001H       ; High byte of first number
        LXI H, 2003H
        ADC M           ; Add high byte + carry
        STA 2005H       ; Store high byte of result
        
        HLT

; EXAMPLE:
; M[2000H]=34H, M[2001H]=12H → 1234H
; M[2002H]=78H, M[2003H]=56H → 5678H
; 
; Low:  34H + 78H = ACH, CY = 0
; High: 12H + 56H + 0 = 68H
; Result: 68ACH at 2004H-2005H
```

### 2.3 16-Bit Result with Carry

```
PROGRAM: 16-bit addition with 24-bit result (handle overflow)

        ORG 0100H
        
        LHLD 2000H      ; HL = first 16-bit number
        XCHG            ; DE = first number
        LHLD 2002H      ; HL = second 16-bit number
        DAD D           ; HL = HL + DE
        SHLD 2004H      ; Store 16-bit result
        
        MVI A, 00H      ; Clear A
        ADC A           ; A = CY (carry from DAD)
        STA 2006H       ; Store carry byte
        
        HLT

; This handles sums > FFFFH
; Example: FFFFH + 0002H = 10001H
; M[2004H-2005H] = 0001H
; M[2006H] = 01H
```

---

## 3. 8-Bit Subtraction

### 3.1 Basic Subtraction

```
8-BIT SUBTRACTION:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   SUB r     A = A - r                                           │
│   SUB M     A = A - Memory[HL]                                  │
│   SUI data  A = A - data                                        │
│   SBB r     A = A - r - CY  (with borrow)                       │
│   SBI data  A = A - data - CY                                   │
│                                                                  │
│   FLAGS: CY = 1 if borrow occurred (A < subtrahend)             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

EXAMPLE: Simple subtraction (no borrow)
───────────────────────────────────────────────────────────────────

; Subtract 32H from 45H
        MVI A, 45H      ; A = 45H
        SUI 32H         ; A = 45H - 32H = 13H
        
; Result: A = 13H, CY = 0 (no borrow)


EXAMPLE: Subtraction with borrow
───────────────────────────────────────────────────────────────────

; Subtract 45H from 32H
        MVI A, 32H      ; A = 32H
        SUI 45H         ; A = 32H - 45H
        
; Calculation:
;   0011 0010 (32H)
; - 0100 0101 (45H)
; ───────────
;   1110 1101 (EDH) with borrow

; Result: A = EDH, CY = 1 (borrow)
; EDH = 2's complement of 13H
; Interpretation: Result is -13H
```

### 3.2 Finding Absolute Difference

```
PROGRAM: Calculate |A - B| (absolute difference)

        ORG 0100H
        
        LDA 2000H       ; First number
        MOV B, A        ; Save to B
        LDA 2001H       ; Second number
        
        CMP B           ; Compare A with B
        JC SWAP         ; If A < B, swap
        
        ; A >= B
        SUB B           ; A = A - B
        JMP STORE
        
SWAP:   ; A < B, so compute B - A
        MOV C, A        ; Save A
        MOV A, B        ; A = B
        SUB C           ; A = B - A
        
STORE:  STA 2002H       ; Store absolute difference
        HLT

; EXAMPLE:
; M[2000H] = 45H, M[2001H] = 32H → Result = 13H
; M[2000H] = 32H, M[2001H] = 45H → Result = 13H (same!)
```

---

## 4. 16-Bit Subtraction

### 4.1 Byte-by-Byte Subtraction

```
16-BIT SUBTRACTION:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Number 1 at 2000H-2001H (minuend)                             │
│   Number 2 at 2002H-2003H (subtrahend)                          │
│   Result   at 2004H-2005H (difference)                          │
│                                                                  │
│   Algorithm:                                                     │
│   1. Subtract low bytes                                         │
│   2. Subtract high bytes with borrow (SBB)                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

PROGRAM:
───────────────────────────────────────────────────────────────────

        ORG 0100H
        
        ; Subtract low bytes
        LDA 2000H       ; Low byte of first number
        LXI H, 2002H
        SUB M           ; Subtract low byte of second
        STA 2004H       ; Store low result
        
        ; Subtract high bytes with borrow
        LDA 2001H       ; High byte of first number
        LXI H, 2003H
        SBB M           ; Subtract high byte + borrow
        STA 2005H       ; Store high result
        
        HLT

; EXAMPLE: 5678H - 1234H = 4444H
; Low:  78H - 34H = 44H, CY = 0
; High: 56H - 12H - 0 = 44H
; Result: 4444H


; EXAMPLE: 1234H - 5678H = ?
; Low:  34H - 78H = BCH, CY = 1 (borrow)
; High: 12H - 56H - 1 = BBH (with borrow)
; Result: BBBCH (negative, 2's complement of 4444H)
```

---

## 5. BCD Arithmetic

### 5.1 BCD Addition with DAA

```
BCD (Binary Coded Decimal) ARITHMETIC:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   BCD represents each decimal digit with 4 bits                 │
│   Valid digits: 0-9 (0000-1001)                                 │
│   Example: 45 decimal = 0100 0101 = 45H in BCD                 │
│                                                                  │
│   DAA (Decimal Adjust Accumulator):                             │
│   Corrects result after BCD addition                            │
│                                                                  │
│   Algorithm:                                                     │
│   If lower nibble > 9 or AC = 1: Add 06H                       │
│   If upper nibble > 9 or CY = 1: Add 60H                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

EXAMPLE: BCD Addition 29 + 48 = 77
───────────────────────────────────────────────────────────────────

        MVI A, 29H      ; BCD for 29
        ADI 48H         ; Add BCD for 48
        ; A = 29H + 48H = 71H (incorrect BCD)
        DAA             ; Decimal adjust
        ; Lower nibble: 1 ≤ 9, AC = 0 → no adjustment
        ; Upper nibble: 7 ≤ 9, CY = 0 → no adjustment
        ; Result: A = 71H... Wait, this IS correct!
        
        ; Let's try another:
        
EXAMPLE: BCD Addition 56 + 27 = 83
───────────────────────────────────────────────────────────────────

        MVI A, 56H      ; BCD for 56
        ADI 27H         ; Add BCD for 27
        ; A = 56H + 27H = 7DH (invalid BCD, D > 9)
        DAA             ; Decimal adjust
        ; Lower nibble: D = 13 > 9 → Add 06H
        ;   7DH + 06H = 83H
        ; Upper nibble: 8 ≤ 9, CY = 0 → no adjustment
        ; Result: A = 83H (correct!)


EXAMPLE: BCD Addition 99 + 01 = 100
───────────────────────────────────────────────────────────────────

        MVI A, 99H      ; BCD for 99
        ADI 01H         ; Add 01
        ; A = 9AH (invalid BCD)
        DAA             ; Decimal adjust
        ; Lower nibble: A = 10 > 9 → Add 06H
        ;   9AH + 06H = A0H, AC = 1
        ; Upper nibble: A = 10 > 9 → Add 60H
        ;   A0H + 60H = 00H, CY = 1
        ; Result: A = 00H, CY = 1
        ; Full result: 100 (CY:A = 1:00)
```

### 5.2 Multi-Digit BCD Addition

```
PROGRAM: Add two 2-digit BCD numbers (4 digits total)

; Numbers at 2000H-2001H and 2002H-2003H (BCD, low digit first)
; Result at 2004H-2005H

        ORG 0100H
        
        ; Add lower 2 digits
        LDA 2000H
        LXI H, 2002H
        ADD M
        DAA             ; BCD adjust
        STA 2004H
        
        ; Add upper 2 digits with carry
        LDA 2001H
        LXI H, 2003H
        ADC M
        DAA             ; BCD adjust
        STA 2005H
        
        HLT

; EXAMPLE: 1234 + 5678 = 6912 (decimal!)
; M[2000H]=34H, M[2001H]=12H → 1234 BCD
; M[2002H]=78H, M[2003H]=56H → 5678 BCD
;
; Lower: 34H + 78H = ACH → DAA → 12H, CY=1
; Upper: 12H + 56H + 1 = 69H (valid BCD)
; Result: 6912 BCD at 2004H-2005H
```

---

## 6. 8-Bit Multiplication

### 6.1 Repeated Addition Method

```
MULTIPLICATION BY REPEATED ADDITION:

Algorithm: A × B = A + A + A + ... (B times)

PROGRAM: Multiply two 8-bit numbers
───────────────────────────────────────────────────────────────────

        ORG 0100H
        
        LDA 2000H       ; Multiplicand
        MOV B, A        ; B = multiplicand
        LDA 2001H       ; Multiplier
        MOV C, A        ; C = multiplier (counter)
        
        MVI A, 00H      ; Clear accumulator (result low)
        MVI D, 00H      ; Clear D (result high)
        
        ORA C           ; Check if multiplier is 0
        JZ DONE         ; If 0, result is 0
        
LOOP:   ADD B           ; A = A + multiplicand
        JNC SKIP        ; If no carry, skip
        INR D           ; D = D + 1 (carry to high byte)
SKIP:   DCR C           ; Decrement counter
        JNZ LOOP        ; Repeat if counter ≠ 0
        
DONE:   STA 2002H       ; Store low byte of result
        MOV A, D
        STA 2003H       ; Store high byte of result
        HLT

; EXAMPLE: 25H × 4 = 94H
; 25H + 25H = 4AH
; 4AH + 25H = 6FH
; 6FH + 25H = 94H
; Result: 0094H
```

### 6.2 Shift-and-Add Method

```
MULTIPLICATION BY SHIFT-AND-ADD:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   More efficient than repeated addition                         │
│   Uses binary multiplication principles                         │
│                                                                  │
│   For each bit of multiplier:                                   │
│   - If bit = 1: Add multiplicand to result                     │
│   - Shift multiplicand left (×2)                                │
│   - Shift multiplier right (check next bit)                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

EXAMPLE: 05H × 03H (5 × 3 = 15)
───────────────────────────────────────────────────────────────────

Multiplicand = 05H = 0000 0101
Multiplier   = 03H = 0000 0011

Step 1: Multiplier bit 0 = 1
        Result = 0 + 05H = 05H
        Shift multiplicand: 0AH

Step 2: Multiplier bit 1 = 1
        Result = 05H + 0AH = 0FH
        Shift multiplicand: 14H

Step 3-7: Multiplier bits = 0
        No addition
        
Final Result: 0FH = 15 ✓


PROGRAM (8×8 = 16-bit result):
───────────────────────────────────────────────────────────────────

        ORG 0100H
        
        LDA 2000H       ; Multiplicand
        MOV B, A
        LDA 2001H       ; Multiplier
        MOV C, A
        
        LXI H, 0000H    ; Result = 0
        MVI D, 08H      ; Counter = 8 bits
        
MULT:   MOV A, C
        RAR             ; Rotate right, LSB → CY
        MOV C, A
        JNC NOADD       ; If bit = 0, skip add
        MOV A, L
        ADD B           ; Add multiplicand to result low
        MOV L, A
        JNC NOADD
        INR H           ; Propagate carry to result high
        
NOADD:  MOV A, B
        RAL             ; Shift multiplicand left
        MOV B, A
        JNC NOCARR
        INR H           ; Handle carry in multiplicand shift
        
NOCARR: DCR D           ; Decrement bit counter
        JNZ MULT        ; Repeat for all 8 bits
        
        SHLD 2002H      ; Store 16-bit result
        HLT
```

---

## 7. 8-Bit Division

### 7.1 Repeated Subtraction Method

```
DIVISION BY REPEATED SUBTRACTION:

Algorithm: Count how many times divisor fits into dividend

PROGRAM: Divide 8-bit dividend by 8-bit divisor
───────────────────────────────────────────────────────────────────

        ORG 0100H
        
        LDA 2000H       ; Dividend
        MOV B, A        ; B = dividend
        LDA 2001H       ; Divisor
        MOV C, A        ; C = divisor
        
        MVI D, 00H      ; D = quotient = 0
        MOV A, B        ; A = dividend
        
        ; Check for divide by zero
        ORA C           ; Check C
        JZ ERROR        ; If divisor = 0, error
        
LOOP:   CMP C           ; Compare A (remaining) with divisor
        JC DONE         ; If A < divisor, done
        SUB C           ; A = A - divisor
        INR D           ; Quotient++
        JMP LOOP
        
DONE:   STA 2002H       ; Store remainder
        MOV A, D
        STA 2003H       ; Store quotient
        HLT
        
ERROR:  MVI A, FFH      ; Error flag
        STA 2002H
        STA 2003H
        HLT

; EXAMPLE: 19 ÷ 5 = 3 remainder 4
; 19 - 5 = 14, quotient = 1
; 14 - 5 = 9,  quotient = 2
; 9 - 5 = 4,   quotient = 3
; 4 < 5, STOP
; Result: Quotient = 3, Remainder = 4
```

### 7.2 16-Bit by 8-Bit Division

```
PROGRAM: Divide 16-bit number by 8-bit number

; Dividend (16-bit) at 2000H-2001H
; Divisor (8-bit) at 2002H
; Quotient (16-bit) at 2003H-2004H
; Remainder (8-bit) at 2005H

        ORG 0100H
        
        LHLD 2000H      ; HL = dividend
        LDA 2002H       ; A = divisor
        MOV C, A        ; C = divisor
        LXI D, 0000H    ; DE = quotient = 0
        
DIVLOOP:
        MOV A, L
        SUB C           ; Try subtracting divisor
        MOV L, A
        MOV A, H
        SBI 00H         ; Subtract borrow from high byte
        MOV H, A
        JC RESTORE      ; If borrow, dividend < divisor
        
        INX D           ; Increment quotient
        JMP DIVLOOP
        
RESTORE:
        ; Add divisor back (we subtracted too much)
        MOV A, L
        ADD C
        MOV L, A
        MOV A, H
        ACI 00H
        MOV H, A
        
        ; Store results
        XCHG            ; HL = quotient, DE = remainder
        SHLD 2003H      ; Store quotient
        MOV A, E        ; Remainder is in E (low byte)
        STA 2005H       ; Store remainder
        
        HLT
```

---

## 8. Complete Programs

### 8.1 Sum of Array Elements

```
PROGRAM: Find sum of N 8-bit numbers

; Count at 2000H
; Array starting at 2001H
; Result (16-bit) at 2050H-2051H

        ORG 0100H
        
        LDA 2000H       ; Get count
        MOV C, A        ; C = counter
        LXI H, 2001H    ; HL points to array
        LXI D, 0000H    ; DE = sum = 0
        
        ORA A           ; Check if count = 0
        JZ DONE
        
LOOP:   MOV A, E        ; Get low byte of sum
        ADD M           ; Add array element
        MOV E, A        ; Store back
        JNC NOCARRY
        INR D           ; Add carry to high byte
        
NOCARRY:
        INX H           ; Next array element
        DCR C           ; Decrement counter
        JNZ LOOP
        
DONE:   XCHG            ; HL = sum
        SHLD 2050H      ; Store 16-bit sum
        HLT

; EXAMPLE:
; Count = 5, Array = 10H, 20H, 30H, 40H, 50H
; Sum = 10H + 20H + 30H + 40H + 50H = F0H
; Result at 2050H-2051H: 00F0H
```

---

## 📋 Summary Table

| Operation | Instructions | Flags | Notes |
|-----------|--------------|-------|-------|
| 8-bit Add | ADD, ADC, ADI, ACI | S,Z,AC,P,CY | ADC includes carry |
| 16-bit Add | DAD | CY only | HL = HL + rp |
| 8-bit Sub | SUB, SBB, SUI, SBI | S,Z,AC,P,CY | SBB includes borrow |
| Increment | INR r, INR M | S,Z,AC,P | No CY effect |
| Decrement | DCR r, DCR M | S,Z,AC,P | No CY effect |
| 16-bit Inc/Dec | INX, DCX | None | No flags affected |
| BCD Adjust | DAA | S,Z,AC,P,CY | After BCD add only |

---

## ❓ Quick Revision Questions

1. **What is the difference between ADD and ADC?**
   <details>
   <summary>Show Answer</summary>
   ADD: A = A + r (doesn't include carry). ADC: A = A + r + CY (includes carry from previous operation). ADC is used for multi-byte addition to propagate carry.
   </details>

2. **Why doesn't INR/DCR affect the carry flag?**
   <details>
   <summary>Show Answer</summary>
   INR/DCR are designed for loop counters where carry shouldn't interfere. If incrementing 0FFH, result is 00H with Z=1 but CY unchanged. This preserves carry from previous arithmetic operations.
   </details>

3. **What does DAA do and when is it used?**
   <details>
   <summary>Show Answer</summary>
   DAA (Decimal Adjust Accumulator) corrects the result in A after BCD addition. If lower nibble >9 or AC=1, adds 06H. If upper nibble >9 or CY=1, adds 60H. Used after ADD/ADC with BCD operands.
   </details>

4. **Write code to add two 16-bit numbers at 2000H and 2002H.**
   <details>
   <summary>Show Answer</summary>
   LHLD 2000H / XCHG / LHLD 2002H / DAD D / SHLD 2004H / HLT. This loads first number to HL, exchanges to DE, loads second to HL, adds them with DAD D, stores result.
   </details>

5. **How do you detect overflow in 8-bit addition?**
   <details>
   <summary>Show Answer</summary>
   Check CY flag after addition. If CY=1, result exceeded 255. For signed overflow, check if sign of result differs from expected (positive + positive = negative indicates overflow).
   </details>

6. **What is the maximum product of two 8-bit numbers?**
   <details>
   <summary>Show Answer</summary>
   FFH × FFH = FE01H (255 × 255 = 65025). The product can be up to 16 bits, so 8×8 multiplication requires 16-bit result storage.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [3.2 Data Transfer Programs](02-data-transfer-programs.md) | [Unit 3 Index](README.md) | [3.4 Logical and Bit Operations](04-logical-bit-operations.md) |

---

*[← Previous: Data Transfer Programs](02-data-transfer-programs.md) | [Next: Logical and Bit Operations →](04-logical-bit-operations.md)*
