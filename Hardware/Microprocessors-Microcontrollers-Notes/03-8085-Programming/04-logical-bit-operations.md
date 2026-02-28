# Chapter 3.4: Logical and Bit Operations

## 📚 Chapter Overview

Logical operations are fundamental for bit manipulation, masking, testing, and data processing. This chapter covers AND, OR, XOR, complement, rotate, and comparison operations.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Perform logical AND, OR, XOR operations
- Mask and extract specific bits
- Set, clear, and toggle individual bits
- Use rotate instructions for bit shifting
- Compare values and make decisions

---

## 1. Logical AND Operations

### 1.1 AND Instructions

```
AND OPERATIONS:

ANA r     A = A AND r
ANA M     A = A AND M[HL]
ANI data  A = A AND data

FLAGS: S, Z, P affected; CY = 0, AC = 1

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   AND TRUTH TABLE:                                              │
│   ┌─────┬─────┬───────┐                                         │
│   │  A  │  B  │ A AND B│                                        │
│   ├─────┼─────┼───────┤                                         │
│   │  0  │  0  │   0   │                                         │
│   │  0  │  1  │   0   │                                         │
│   │  1  │  0  │   0   │                                         │
│   │  1  │  1  │   1   │                                         │
│   └─────┴─────┴───────┘                                         │
│                                                                  │
│   USES:                                                          │
│   • Masking bits (extract specific bits)                        │
│   • Clearing bits (AND with 0)                                  │
│   • Testing bits                                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Masking Examples

```
MASKING - Extract Specific Bits:

EXAMPLE 1: Extract lower nibble (bits 0-3)
───────────────────────────────────────────────────────────────────

        MVI A, 7BH      ; A = 0111 1011
        ANI 0FH         ; Mask = 0000 1111
                        ; A = 0000 1011 = 0BH

; Calculation:
;   0111 1011 (7BH)
; AND
;   0000 1111 (0FH)
; ───────────
;   0000 1011 (0BH)


EXAMPLE 2: Extract upper nibble (bits 4-7)
───────────────────────────────────────────────────────────────────

        MVI A, 7BH      ; A = 0111 1011
        ANI F0H         ; Mask = 1111 0000
                        ; A = 0111 0000 = 70H
        
        ; To get value 7, rotate right 4 times:
        RRC
        RRC
        RRC
        RRC             ; A = 07H


EXAMPLE 3: Extract bit 5
───────────────────────────────────────────────────────────────────

        MVI A, 7BH      ; A = 0111 1011
        ANI 20H         ; Mask = 0010 0000 (bit 5)
                        ; A = 0010 0000 = 20H (non-zero if bit 5 = 1)
        
        ; Result: A = 20H (bit 5 was 1)
        ; If bit 5 was 0, A = 00H
```

### 1.3 Clearing Bits

```
CLEARING BITS - AND with 0:

EXAMPLE: Clear bits 0 and 1 (make number divisible by 4)
───────────────────────────────────────────────────────────────────

        MVI A, 7FH      ; A = 0111 1111
        ANI FCH         ; Mask = 1111 1100
                        ; A = 0111 1100 = 7CH

; Bit positions with 0 in mask get cleared
; Bit positions with 1 in mask are preserved


EXAMPLE: Clear upper nibble
───────────────────────────────────────────────────────────────────

        MVI A, 0ABH     ; A = 1010 1011
        ANI 0FH         ; Mask = 0000 1111
                        ; A = 0000 1011 = 0BH
```

---

## 2. Logical OR Operations

### 2.1 OR Instructions

```
OR OPERATIONS:

ORA r     A = A OR r
ORA M     A = A OR M[HL]
ORI data  A = A OR data

FLAGS: S, Z, P affected; CY = 0, AC = 0

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   OR TRUTH TABLE:                                               │
│   ┌─────┬─────┬──────┐                                          │
│   │  A  │  B  │ A OR B│                                         │
│   ├─────┼─────┼──────┤                                          │
│   │  0  │  0  │   0  │                                          │
│   │  0  │  1  │   1  │                                          │
│   │  1  │  0  │   1  │                                          │
│   │  1  │  1  │   1  │                                          │
│   └─────┴─────┴──────┘                                          │
│                                                                  │
│   USES:                                                          │
│   • Setting bits (OR with 1)                                    │
│   • Combining bit fields                                        │
│   • Testing for any bits set (ORA A to check Z flag)           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Setting Bits

```
SETTING BITS - OR with 1:

EXAMPLE 1: Set bit 7 (make number negative in signed)
───────────────────────────────────────────────────────────────────

        MVI A, 3CH      ; A = 0011 1100
        ORI 80H         ; Mask = 1000 0000
                        ; A = 1011 1100 = BCH

; Bit positions with 1 in mask get SET
; Bit positions with 0 in mask are preserved


EXAMPLE 2: Set bits 0 and 1
───────────────────────────────────────────────────────────────────

        MVI A, 3CH      ; A = 0011 1100
        ORI 03H         ; Mask = 0000 0011
                        ; A = 0011 1111 = 3FH


EXAMPLE 3: Convert lowercase to uppercase ASCII
───────────────────────────────────────────────────────────────────

; ASCII 'a' = 61H, 'A' = 41H
; Difference is bit 5 (20H)

        MVI A, 61H      ; A = 'a'
        ANI DFH         ; Clear bit 5: A = 41H = 'A'

; Convert uppercase to lowercase:
        MVI A, 41H      ; A = 'A'
        ORI 20H         ; Set bit 5: A = 61H = 'a'
```

### 2.3 Combining Nibbles

```
COMBINING NIBBLES:

EXAMPLE: Pack two BCD digits into one byte
───────────────────────────────────────────────────────────────────

; Upper digit in B = 5, Lower digit in C = 3
; Result: A = 53H

        MOV A, B        ; A = 05H
        RLC             ; Shift left 4 times
        RLC
        RLC
        RLC             ; A = 50H
        ORA C           ; Combine with C: A = 50H OR 03H = 53H
```

---

## 3. Logical XOR Operations

### 3.1 XOR Instructions

```
XOR OPERATIONS:

XRA r     A = A XOR r
XRA M     A = A XOR M[HL]
XRI data  A = A XOR data

FLAGS: S, Z, P affected; CY = 0, AC = 0

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   XOR TRUTH TABLE:                                              │
│   ┌─────┬─────┬───────┐                                         │
│   │  A  │  B  │ A XOR B│                                        │
│   ├─────┼─────┼───────┤                                         │
│   │  0  │  0  │   0   │                                         │
│   │  0  │  1  │   1   │                                         │
│   │  1  │  0  │   1   │                                         │
│   │  1  │  1  │   0   │                                         │
│   └─────┴─────┴───────┘                                         │
│                                                                  │
│   USES:                                                          │
│   • Toggle bits (XOR with 1)                                    │
│   • Clear register (XOR with itself)                            │
│   • Compare for equality                                         │
│   • Encryption/decryption                                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 XOR Applications

```
CLEAR REGISTER (Fast method):

        XRA A           ; A = A XOR A = 00H
                        ; 1 byte, 4 T-states
        ; vs
        MVI A, 00H      ; 2 bytes, 7 T-states

; XRA A is faster and smaller!


TOGGLE BITS:
───────────────────────────────────────────────────────────────────

        MVI A, 55H      ; A = 0101 0101
        XRI FFH         ; Toggle all bits
                        ; A = 1010 1010 = AAH

; Toggle specific bits:
        MVI A, 55H      ; A = 0101 0101
        XRI 0FH         ; Toggle bits 0-3
                        ; A = 0101 1010 = 5AH


CHECK IF TWO VALUES ARE EQUAL:
───────────────────────────────────────────────────────────────────

        MOV A, B        ; A = first value
        XRA C           ; XOR with second value
        JZ EQUAL        ; If result = 0, they're equal

; If A XOR C = 0, then A = C


SWAP WITHOUT TEMP (XOR trick):
───────────────────────────────────────────────────────────────────

; Exchange A and B without temp register
        XRA B           ; A = A XOR B
        MOV B, A        ; B = A XOR B
        XRA B           ; A = (A XOR B) XOR B = A
        ; Wait... this doesn't work for 8085!
        
; Actually, for 8085, the temp register method is better:
        MOV C, A        ; C = A
        MOV A, B        ; A = B
        MOV B, C        ; B = original A
```

---

## 4. Complement Operations

### 4.1 CMA Instruction

```
CMA - COMPLEMENT ACCUMULATOR:

CMA       A = NOT A (1's complement)

EXAMPLE:
───────────────────────────────────────────────────────────────────

        MVI A, 5AH      ; A = 0101 1010
        CMA             ; A = 1010 0101 = A5H

; Every bit is inverted (1 → 0, 0 → 1)


2's COMPLEMENT (for negation):
───────────────────────────────────────────────────────────────────

        MVI A, 05H      ; A = 05H (5 decimal)
        CMA             ; 1's complement: A = FAH
        INR A           ; 2's complement: A = FBH (-5)

; -5 in 2's complement = FBH
; Verify: FBH + 05H = 100H (carry, low byte = 00)
```

### 4.2 Carry Flag Operations

```
CARRY FLAG OPERATIONS:

STC       CY = 1 (Set Carry)
CMC       CY = NOT CY (Complement Carry)

EXAMPLES:
───────────────────────────────────────────────────────────────────

        STC             ; CY = 1
        CMC             ; CY = 0
        CMC             ; CY = 1


USE CASE - Subtract using addition:
───────────────────────────────────────────────────────────────────

; A = A - B  is same as  A = A + (-B)
; -B = 2's complement of B = NOT B + 1

        MOV A, B        ; A = B
        CMA             ; A = NOT B
        INR A           ; A = -B (2's complement)
        ADD C           ; A = C + (-B) = C - B
```

---

## 5. Rotate Instructions

### 5.1 Rotate Left

```
ROTATE LEFT OPERATIONS:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   RLC (Rotate Left Circular):                                   │
│   ┌────────────────────────────────────────┐                    │
│   │  ┌──◄────────────────────────────────┐ │                    │
│   │  │   ┌─┬─┬─┬─┬─┬─┬─┬─┐              │ │                    │
│   │  └──►│7│6│5│4│3│2│1│0│──────────────┘ │                    │
│   │      └─┴─┴─┴─┴─┴─┴─┴─┘                │                    │
│   │          │                             │                    │
│   │          └──► CY (copy of bit 7)       │                    │
│   └────────────────────────────────────────┘                    │
│                                                                  │
│   RAL (Rotate Left through Carry):                              │
│   ┌────────────────────────────────────────┐                    │
│   │      ┌───◄───────────────────────────┐ │                    │
│   │  ┌───┐   ┌─┬─┬─┬─┬─┬─┬─┬─┐          │ │                    │
│   │  │CY │◄──│7│6│5│4│3│2│1│0│◄─────────┘ │                    │
│   │  └───┘   └─┴─┴─┴─┴─┴─┴─┴─┘            │                    │
│   └────────────────────────────────────────┘                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

EXAMPLE - RLC:
───────────────────────────────────────────────────────────────────

        MVI A, 85H      ; A = 1000 0101, CY = ?
        RLC             ; A = 0000 1011, CY = 1
                        ;     ↑ Bit 7 went to bit 0 AND to CY

; 85H << 1 circular = 0AH + 1 = 0BH


EXAMPLE - RAL:
───────────────────────────────────────────────────────────────────

        STC             ; CY = 1
        MVI A, 85H      ; A = 1000 0101
        RAL             ; A = 0000 1011, CY = 1
                        ;          ↑ Old CY (1) went to bit 0
                        ;     Bit 7 went to new CY
```

### 5.2 Rotate Right

```
ROTATE RIGHT OPERATIONS:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   RRC (Rotate Right Circular):                                  │
│   ┌────────────────────────────────────────┐                    │
│   │  ┌────────────────────────────────►──┐ │                    │
│   │  │   ┌─┬─┬─┬─┬─┬─┬─┬─┐              │ │                    │
│   │  └───│7│6│5│4│3│2│1│0│◄─────────────┘ │                    │
│   │      └─┴─┴─┴─┴─┴─┴─┴─┘                │                    │
│   │                    │                   │                    │
│   │                    └──► CY             │                    │
│   └────────────────────────────────────────┘                    │
│                                                                  │
│   RAR (Rotate Right through Carry):                             │
│   ┌────────────────────────────────────────┐                    │
│   │  ┌─────────────────────────────────►─┐ │                    │
│   │  │   ┌─┬─┬─┬─┬─┬─┬─┬─┐   ┌───┐     │ │                    │
│   │  └──►│7│6│5│4│3│2│1│0│──►│CY │─────┘ │                    │
│   │      └─┴─┴─┴─┴─┴─┴─┴─┘   └───┘       │                    │
│   └────────────────────────────────────────┘                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

EXAMPLE - RRC:
───────────────────────────────────────────────────────────────────

        MVI A, 85H      ; A = 1000 0101
        RRC             ; A = 1100 0010, CY = 1
                        ;     Bit 0 went to bit 7 AND to CY
                        ;     = C2H


EXAMPLE - RAR:
───────────────────────────────────────────────────────────────────

        XRA A           ; CY = 0
        MVI A, 85H      ; A = 1000 0101
        RAR             ; A = 0100 0010, CY = 1
                        ;     ↑ Old CY (0) went to bit 7
                        ;     Bit 0 went to new CY
                        ;     = 42H
```

### 5.3 Multiplication/Division by 2

```
SHIFT OPERATIONS FOR MULTIPLY/DIVIDE:

MULTIPLY BY 2 (Shift Left):
───────────────────────────────────────────────────────────────────

        MVI A, 15H      ; A = 21 decimal
        RLC             ; A = 2AH = 42 decimal
        ; 21 × 2 = 42 ✓

; Works if result fits in 8 bits (no overflow)


DIVIDE BY 2 (Shift Right):
───────────────────────────────────────────────────────────────────

        MVI A, 2AH      ; A = 42 decimal
        RRC             ; A = 15H = 21 decimal
        ; 42 ÷ 2 = 21 ✓

; CY contains remainder (0 or 1)


MULTIPLY BY 4 (Two left shifts):
───────────────────────────────────────────────────────────────────

        MVI A, 0AH      ; A = 10
        RLC             ; A = 20
        RLC             ; A = 40
        ; 10 × 4 = 40 ✓
```

---

## 6. Compare Operations

### 6.1 CMP and CPI Instructions

```
COMPARE OPERATIONS:

CMP r     Compare A with r (A - r, result discarded)
CMP M     Compare A with M[HL]
CPI data  Compare A with immediate data

FLAGS set based on (A - operand):
• If A = operand: Z = 1, CY = 0
• If A < operand: Z = 0, CY = 1
• If A > operand: Z = 0, CY = 0

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   COMPARISON DECISION TABLE:                                    │
│   ┌───────────────┬───────┬───────┬────────────────────────┐    │
│   │   Condition   │   Z   │  CY   │    Jump Instruction    │    │
│   ├───────────────┼───────┼───────┼────────────────────────┤    │
│   │    A = B      │   1   │   0   │    JZ  (Jump if Zero)  │    │
│   │    A ≠ B      │   0   │   x   │    JNZ (Jump Not Zero) │    │
│   │    A < B      │   0   │   1   │    JC  (Jump if Carry) │    │
│   │    A ≥ B      │   x   │   0   │    JNC (Jump No Carry) │    │
│   │    A > B      │   0   │   0   │    JNC, then JNZ       │    │
│   │    A ≤ B      │   x   │   x   │    JZ or JC            │    │
│   └───────────────┴───────┴───────┴────────────────────────┘    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 Comparison Examples

```
EXAMPLE: Check if equal
───────────────────────────────────────────────────────────────────

        LDA 2000H       ; Get first value
        MOV B, A
        LDA 2001H       ; Get second value
        CMP B           ; Compare A with B
        JZ EQUAL        ; Jump if A = B
        JMP NOTEQUAL
        
EQUAL:  MVI A, 01H      ; Flag = equal
        JMP DONE
NOTEQUAL:
        MVI A, 00H      ; Flag = not equal
DONE:   STA 2002H


EXAMPLE: Find larger of two numbers
───────────────────────────────────────────────────────────────────

        LDA 2000H       ; First number
        MOV B, A
        LDA 2001H       ; Second number
        CMP B           ; Compare A with B
        JNC AISBIGGER   ; If A ≥ B, A is bigger or equal
        MOV A, B        ; B is bigger, so A = B
AISBIGGER:
        STA 2002H       ; Store larger value
        HLT


EXAMPLE: Range check (10 ≤ A ≤ 100)
───────────────────────────────────────────────────────────────────

        LDA 2000H       ; Value to check
        
        CPI 0AH         ; Compare with 10
        JC OUTOFRANGE   ; If A < 10, out of range
        
        CPI 65H         ; Compare with 101
        JNC OUTOFRANGE  ; If A ≥ 101, out of range
        
        ; A is in range [10, 100]
        MVI B, 01H      ; Set flag = in range
        JMP DONE
        
OUTOFRANGE:
        MVI B, 00H      ; Set flag = out of range
        
DONE:   MOV A, B
        STA 2001H
        HLT
```

---

## 7. Bit Testing

### 7.1 Test Specific Bits

```
TESTING INDIVIDUAL BITS:

Method: AND with mask, check Z flag

EXAMPLE: Test if bit 3 is set
───────────────────────────────────────────────────────────────────

        LDA 2000H       ; Get value
        ANI 08H         ; Mask for bit 3 (0000 1000)
        JZ BIT3_CLEAR   ; If result = 0, bit was 0
        
        ; Bit 3 is set (1)
        MVI A, 01H
        JMP DONE
        
BIT3_CLEAR:
        MVI A, 00H
        
DONE:   STA 2001H       ; Store result


EXAMPLE: Test if number is odd (bit 0 = 1)
───────────────────────────────────────────────────────────────────

        LDA 2000H
        ANI 01H         ; Check bit 0
        JZ EVEN
        ; ODD
        JMP DONE
EVEN:   ; EVEN
DONE:   HLT


EXAMPLE: Test if number is negative (bit 7 = 1)
───────────────────────────────────────────────────────────────────

        LDA 2000H
        ANI 80H         ; Check bit 7
        JZ POSITIVE
        ; NEGATIVE
        JMP DONE
POSITIVE:
        ; POSITIVE (or zero)
DONE:   HLT

; Alternative using Sign flag:
        LDA 2000H
        ORA A           ; Set flags based on A
        JM NEGATIVE     ; Jump if Minus (S = 1)
```

### 7.2 Count Set Bits

```
PROGRAM: Count number of 1s in a byte

        ORG 0100H
        
        LDA 2000H       ; Number to count
        MVI B, 00H      ; Bit counter = 0
        MVI C, 08H      ; Loop counter = 8
        
LOOP:   RRC             ; Rotate right, LSB → CY
        JNC SKIP        ; If CY = 0, skip count
        INR B           ; Count this bit
SKIP:   DCR C           ; Next bit
        JNZ LOOP
        
        MOV A, B        ; Result in A
        STA 2001H       ; Store count
        HLT

; EXAMPLE: A = A5H = 1010 0101
; Bits set: 1,0,1,0,0,1,0,1 → 4 ones
; Result: 04H
```

---

## 8. Complete Programs

### 8.1 Separate Even and Odd Numbers

```
PROGRAM: Separate array into even and odd

; Input:  Array at 2000H, count at 2010H
; Output: Even numbers at 2020H, Odd at 2030H
;         Counts at 2040H (even) and 2041H (odd)

        ORG 0100H
        
        LXI H, 2000H    ; Source array
        LXI B, 2020H    ; Even destination
        LXI D, 2030H    ; Odd destination
        
        LDA 2010H       ; Get count
        MOV E, A        ; Save count (using E temporarily)
        PUSH D          ; Save DE
        LXI D, 2030H    ; Restore odd pointer
        
        MVI A, 00H
        STA 2040H       ; Even count = 0
        STA 2041H       ; Odd count = 0
        
        LDA 2010H       ; Reload count
        ORA A
        JZ DONE
        
LOOP:   MOV A, M        ; Get array element
        ANI 01H         ; Check bit 0
        MOV A, M        ; Reload element
        JNZ ODD
        
EVEN:   STAX B          ; Store at even array
        INX B
        PUSH H
        LXI H, 2040H
        INR M           ; Increment even count
        POP H
        JMP NEXT
        
ODD:    STAX D          ; Store at odd array
        INX D
        PUSH H
        LXI H, 2041H
        INR M           ; Increment odd count
        POP H
        
NEXT:   INX H           ; Next source element
        LDA 2010H       ; Get original count
        DCR A
        STA 2010H       ; Update count
        JNZ LOOP
        
DONE:   HLT
```

---

## 📋 Summary Table

| Instruction | Operation | CY After | AC After | Notes |
|-------------|-----------|----------|----------|-------|
| ANA r | A = A AND r | 0 | 1 | Masking |
| ORA r | A = A OR r | 0 | 0 | Setting |
| XRA r | A = A XOR r | 0 | 0 | Toggle |
| CMA | A = NOT A | No change | No change | 1's comp |
| STC | CY = 1 | 1 | No change | Set carry |
| CMC | CY = NOT CY | Toggled | No change | Comp carry |
| RLC | Rotate left | Bit 7 | No change | Circular |
| RRC | Rotate right | Bit 0 | No change | Circular |
| RAL | Rotate left thru CY | Bit 7 | No change | 9-bit |
| RAR | Rotate right thru CY | Bit 0 | No change | 9-bit |
| CMP r | A - r (flags only) | Set | Set | Compare |

---

## ❓ Quick Revision Questions

1. **How do you extract bits 4-7 from a byte?**
   <details>
   <summary>Show Answer</summary>
   ANI F0H (mask 1111 0000) to isolate upper nibble. To get the value 0-F, follow with four RRC instructions to shift right.
   </details>

2. **What is the fastest way to clear the accumulator?**
   <details>
   <summary>Show Answer</summary>
   XRA A (1 byte, 4 T-states). This XORs A with itself, always resulting in 00H. Faster than MVI A, 00H (2 bytes, 7 T-states).
   </details>

3. **How does RLC differ from RAL?**
   <details>
   <summary>Show Answer</summary>
   RLC: 8-bit circular rotate, bit 7 goes to both bit 0 and CY. RAL: 9-bit rotate through carry, bit 7 goes to CY, old CY goes to bit 0. RLC ignores old CY, RAL includes it.
   </details>

4. **Write code to check if a number is even or odd.**
   <details>
   <summary>Show Answer</summary>
   ANI 01H / JZ EVEN. This masks all bits except bit 0. If bit 0 = 0 (result is zero), number is even; if bit 0 = 1, number is odd.
   </details>

5. **How do you toggle bit 5 of the accumulator?**
   <details>
   <summary>Show Answer</summary>
   XRI 20H. XOR with 0010 0000 will toggle bit 5 (changes 0 to 1 or 1 to 0) while leaving other bits unchanged.
   </details>

6. **After CMP B, what do the flags indicate if A=45H and B=50H?**
   <details>
   <summary>Show Answer</summary>
   A - B = 45H - 50H = F5H (with borrow). So: CY = 1 (borrow occurred, A < B), Z = 0 (result not zero, A ≠ B), S = 1 (result negative).
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [3.3 Arithmetic Operations](03-arithmetic-operations.md) | [Unit 3 Index](README.md) | [3.5 Array and String Operations](05-array-string-operations.md) |

---

*[← Previous: Arithmetic Operations](03-arithmetic-operations.md) | [Next: Array and String Operations →](05-array-string-operations.md)*
