# Chapter 3.5: Array and String Operations

## 📚 Chapter Overview

Arrays and strings are fundamental data structures in programming. This chapter covers techniques for traversing, searching, and manipulating arrays and strings in 8085 assembly language.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Traverse and process array elements
- Find maximum and minimum values in arrays
- Count elements meeting specific criteria
- Perform string operations (length, copy, compare)
- Implement block data transfer operations

---

## 1. Array Basics

### 1.1 Array Memory Layout

```
ARRAY ORGANIZATION IN MEMORY:

Array of 5 bytes starting at 2000H:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Address    Content    Index                                   │
│   ────────────────────────────                                  │
│   2000H      25H        [0] ◄── Base address                    │
│   2001H      3AH        [1]                                     │
│   2002H      12H        [2]                                     │
│   2003H      8FH        [3]                                     │
│   2004H      5CH        [4]                                     │
│                                                                  │
│   Element address = Base + Index                                 │
│   Example: Element[3] at 2000H + 3 = 2003H                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

COMMON ARRAY ACCESS PATTERNS:

Using HL as pointer:
───────────────────────────────────────────────────────────────────

        LXI H, 2000H    ; Point to base address
        MOV A, M        ; A = Array[0]
        INX H           ; Point to next element
        MOV A, M        ; A = Array[1]
        ; ... and so on


Using counter for iteration:
───────────────────────────────────────────────────────────────────

        LXI H, 2000H    ; Base address
        MVI C, 05H      ; Count = 5
LOOP:   MOV A, M        ; Get element
        ; ... process element ...
        INX H           ; Next element
        DCR C           ; Decrement counter
        JNZ LOOP        ; Repeat if not done
```

---

## 2. Finding Maximum and Minimum

### 2.1 Find Maximum in Array

```
PROGRAM: Find largest element in array

; Input:  Array at 2000H, count at 2010H
; Output: Maximum value at 2011H

; ALGORITHM:
; 1. Assume first element is maximum
; 2. Compare with each subsequent element
; 3. If larger found, update maximum
; 4. Repeat until all elements checked

        ORG 0100H
        
        LDA 2010H       ; Get count
        DCR A           ; Count - 1 (first element is initial max)
        MOV C, A        ; C = loop counter
        
        LXI H, 2000H    ; Point to array
        MOV A, M        ; First element = initial max
        MOV B, A        ; B = current maximum
        INX H           ; Point to second element
        
LOOP:   MOV A, M        ; Get next element
        CMP B           ; Compare with max
        JC SKIP         ; If A < B, skip update
        MOV B, A        ; New maximum found
SKIP:   INX H           ; Next element
        DCR C           ; Decrement counter
        JNZ LOOP        ; Repeat
        
        MOV A, B        ; Result in A
        STA 2011H       ; Store maximum
        HLT

; EXAMPLE:
; Array: 45H, 89H, 23H, 67H, 12H at 2000H
; Count: 05H at 2010H
; Maximum: 89H stored at 2011H

; TRACE:
; Initial: B = 45H (first element)
; Compare 89H: 89H > 45H → B = 89H
; Compare 23H: 23H < 89H → B unchanged
; Compare 67H: 67H < 89H → B unchanged
; Compare 12H: 12H < 89H → B unchanged
; Result: B = 89H
```

### 2.2 Find Minimum in Array

```
PROGRAM: Find smallest element in array

; Input:  Array at 2000H, count at 2010H
; Output: Minimum value at 2011H

        ORG 0100H
        
        LDA 2010H       ; Get count
        DCR A           ; Count - 1
        MOV C, A        ; C = loop counter
        
        LXI H, 2000H    ; Point to array
        MOV A, M        ; First element = initial min
        MOV B, A        ; B = current minimum
        INX H           ; Point to second element
        
LOOP:   MOV A, M        ; Get next element
        CMP B           ; Compare with min
        JNC SKIP        ; If A ≥ B, skip update
        MOV B, A        ; New minimum found
SKIP:   INX H           ; Next element
        DCR C           ; Decrement counter
        JNZ LOOP        ; Repeat
        
        MOV A, B        ; Result in A
        STA 2011H       ; Store minimum
        HLT

; Note: Only difference from max is JNC instead of JC
```

### 2.3 Find Both Max and Min

```
PROGRAM: Find both maximum and minimum

; Input:  Array at 2000H, count at 2010H
; Output: Maximum at 2011H, Minimum at 2012H

        ORG 0100H
        
        LDA 2010H       ; Get count
        DCR A           ; Count - 1
        MOV C, A        ; C = loop counter
        
        LXI H, 2000H    ; Point to array
        MOV A, M        ; First element
        MOV B, A        ; B = current maximum
        MOV D, A        ; D = current minimum
        INX H           ; Point to second element
        
LOOP:   MOV A, M        ; Get element
        
        ; Check for new maximum
        CMP B           ; Compare with max
        JC CHECKMIN     ; If A < B, check minimum
        MOV B, A        ; New maximum
        
CHECKMIN:
        MOV A, M        ; Reload element
        CMP D           ; Compare with min
        JNC NEXT        ; If A ≥ D, skip
        MOV D, A        ; New minimum
        
NEXT:   INX H           ; Next element
        DCR C           ; Decrement counter
        JNZ LOOP        ; Repeat
        
        MOV A, B
        STA 2011H       ; Store maximum
        MOV A, D
        STA 2012H       ; Store minimum
        HLT
```

---

## 3. Counting Operations

### 3.1 Count Positive and Negative Numbers

```
PROGRAM: Count positive and negative numbers in array

; Input:  Signed array at 2000H, count at 2010H
; Output: Positive count at 2011H, Negative count at 2012H

        ORG 0100H
        
        LXI H, 2000H    ; Point to array
        LDA 2010H       ; Get count
        MOV C, A        ; C = loop counter
        MVI D, 00H      ; D = positive count
        MVI E, 00H      ; E = negative count
        
LOOP:   MOV A, M        ; Get element
        ANI 80H         ; Check bit 7 (sign bit)
        JNZ NEGATIVE    ; If bit 7 = 1, negative
        
        INR D           ; Positive count++
        JMP NEXT
        
NEGATIVE:
        INR E           ; Negative count++
        
NEXT:   INX H           ; Next element
        DCR C           ; Decrement counter
        JNZ LOOP
        
        MOV A, D
        STA 2011H       ; Store positive count
        MOV A, E
        STA 2012H       ; Store negative count
        HLT

; EXAMPLE:
; Array: 25H, 89H, F3H, 67H, 82H (signed: +37, -119, -13, +103, -126)
; Positive: 25H, 67H → count = 2
; Negative: 89H, F3H, 82H → count = 3
```

### 3.2 Count Zeros in Array

```
PROGRAM: Count zero elements

; Input:  Array at 2000H, count at 2010H
; Output: Zero count at 2011H

        ORG 0100H
        
        LXI H, 2000H    ; Point to array
        LDA 2010H       ; Get count
        MOV C, A        ; C = loop counter
        MVI B, 00H      ; B = zero count
        
LOOP:   MOV A, M        ; Get element
        ORA A           ; Set flags (test if zero)
        JNZ NOTZERO     ; If not zero, skip
        INR B           ; Zero count++
NOTZERO:
        INX H           ; Next element
        DCR C
        JNZ LOOP
        
        MOV A, B
        STA 2011H       ; Store zero count
        HLT
```

### 3.3 Count Even Numbers

```
PROGRAM: Count even numbers in array

        ORG 0100H
        
        LXI H, 2000H    ; Point to array
        LDA 2010H       ; Get count
        MOV C, A        ; C = loop counter
        MVI B, 00H      ; B = even count
        
LOOP:   MOV A, M        ; Get element
        ANI 01H         ; Check bit 0
        JNZ ODD         ; If bit 0 = 1, odd
        INR B           ; Even count++
ODD:    INX H           ; Next element
        DCR C
        JNZ LOOP
        
        MOV A, B
        STA 2011H       ; Store even count
        HLT
```

### 3.4 Count Occurrences of a Value

```
PROGRAM: Count how many times value X appears in array

; Input:  Array at 2000H, count at 2010H, value X at 2011H
; Output: Occurrence count at 2012H

        ORG 0100H
        
        LXI H, 2000H    ; Point to array
        LDA 2010H       ; Get count
        MOV C, A        ; C = loop counter
        LDA 2011H       ; Get value to find
        MOV B, A        ; B = target value
        MVI D, 00H      ; D = occurrence count
        
LOOP:   MOV A, M        ; Get element
        CMP B           ; Compare with target
        JNZ NOTFOUND    ; If not equal, skip
        INR D           ; Occurrence count++
NOTFOUND:
        INX H           ; Next element
        DCR C
        JNZ LOOP
        
        MOV A, D
        STA 2012H       ; Store count
        HLT

; EXAMPLE:
; Array: 05H, 08H, 05H, 03H, 05H
; Target: 05H
; Count: 03H (appears 3 times)
```

---

## 4. Array Summation

### 4.1 Sum of Array Elements (8-bit)

```
PROGRAM: Calculate sum of array (assuming no overflow)

; Input:  Array at 2000H, count at 2010H
; Output: Sum at 2011H

        ORG 0100H
        
        LXI H, 2000H    ; Point to array
        LDA 2010H       ; Get count
        MOV C, A        ; C = loop counter
        XRA A           ; A = 0 (sum)
        
LOOP:   ADD M           ; A = A + M[HL]
        INX H           ; Next element
        DCR C
        JNZ LOOP
        
        STA 2011H       ; Store sum
        HLT
```

### 4.2 Sum with Carry (16-bit Result)

```
PROGRAM: Sum of array with 16-bit result

; Input:  Array at 2000H, count at 2010H
; Output: Sum (16-bit) at 2011H (low), 2012H (high)

        ORG 0100H
        
        LXI H, 2000H    ; Point to array
        LDA 2010H       ; Get count
        MOV C, A        ; C = loop counter
        XRA A           ; A = 0
        MOV B, A        ; B = 0 (high byte of sum)
        MOV D, A        ; D = 0 (low byte of sum)
        
LOOP:   MOV A, D        ; Get low byte
        ADD M           ; Add array element
        MOV D, A        ; Save low byte
        JNC NOCARRY     ; If no carry, skip
        INR B           ; Increment high byte
NOCARRY:
        INX H           ; Next element
        DCR C
        JNZ LOOP
        
        MOV A, D
        STA 2011H       ; Store low byte
        MOV A, B
        STA 2012H       ; Store high byte
        HLT

; EXAMPLE:
; Array: 80H, 90H, A0H, B0H, C0H (128+144+160+176+192 = 800 = 320H)
; Result: 2011H = 20H (low), 2012H = 03H (high)
```

---

## 5. Block Operations

### 5.1 Block Copy (Non-Overlapping)

```
PROGRAM: Copy block of data from source to destination

; Input:  Source at 2000H, Destination at 3000H
;         Count at 2050H
; Output: Data copied to destination

        ORG 0100H
        
        LXI H, 2000H    ; Source address
        LXI D, 3000H    ; Destination address
        LDA 2050H       ; Get count
        MOV C, A        ; C = loop counter
        
LOOP:   MOV A, M        ; Get from source
        STAX D          ; Store to destination
        INX H           ; Next source
        INX D           ; Next destination
        DCR C
        JNZ LOOP
        
        HLT

; DIAGRAM:
;
;   SOURCE (2000H)          DESTINATION (3000H)
;   ┌────────┐              ┌────────┐
;   │  12H   │    ────►     │  12H   │
;   ├────────┤              ├────────┤
;   │  34H   │    ────►     │  34H   │
;   ├────────┤              ├────────┤
;   │  56H   │    ────►     │  56H   │
;   └────────┘              └────────┘
```

### 5.2 Block Exchange

```
PROGRAM: Exchange two blocks of data

; Input:  Block1 at 2000H, Block2 at 3000H
;         Count at 2050H
; Output: Contents of blocks exchanged

        ORG 0100H
        
        LXI H, 2000H    ; Block 1
        LXI D, 3000H    ; Block 2
        LDA 2050H       ; Get count
        MOV C, A
        
LOOP:   MOV A, M        ; Get from Block1
        MOV B, A        ; Save in B
        LDAX D          ; Get from Block2
        MOV M, A        ; Store to Block1
        MOV A, B        ; Retrieve Block1 value
        STAX D          ; Store to Block2
        INX H
        INX D
        DCR C
        JNZ LOOP
        
        HLT
```

### 5.3 Block Fill

```
PROGRAM: Fill memory block with a value

; Input:  Start at 2000H, Count at 2050H, Value at 2051H
; Output: Memory filled with value

        ORG 0100H
        
        LXI H, 2000H    ; Start address
        LDA 2051H       ; Get fill value
        MOV B, A        ; B = fill value
        LDA 2050H       ; Get count
        MOV C, A
        
LOOP:   MOV M, B        ; Fill location
        INX H           ; Next location
        DCR C
        JNZ LOOP
        
        HLT
```

### 5.4 Block Compare

```
PROGRAM: Compare two blocks of data

; Input:  Block1 at 2000H, Block2 at 3000H
;         Count at 2050H
; Output: Result at 2051H (00 = equal, FF = not equal)

        ORG 0100H
        
        LXI H, 2000H    ; Block 1
        LXI D, 3000H    ; Block 2
        LDA 2050H       ; Get count
        MOV C, A
        
LOOP:   MOV A, M        ; Get from Block1
        MOV B, A
        LDAX D          ; Get from Block2
        CMP B           ; Compare
        JNZ NOTEQUAL    ; If not equal, exit
        INX H
        INX D
        DCR C
        JNZ LOOP
        
        ; All bytes equal
        MVI A, 00H
        JMP STORE
        
NOTEQUAL:
        MVI A, FFH
        
STORE:  STA 2051H
        HLT
```

---

## 6. String Operations

### 6.1 String Length

```
PROGRAM: Find length of null-terminated string

; Input:  String at 2000H (null-terminated)
; Output: Length at 2050H

; A string is a sequence of ASCII characters ending with 00H

        ORG 0100H
        
        LXI H, 2000H    ; Point to string
        MVI C, 00H      ; Length counter = 0
        
LOOP:   MOV A, M        ; Get character
        ORA A           ; Check if null (00H)
        JZ DONE         ; If null, string ended
        INR C           ; Length++
        INX H           ; Next character
        JMP LOOP
        
DONE:   MOV A, C
        STA 2050H       ; Store length
        HLT

; EXAMPLE:
; String: "HELLO" = 48H, 45H, 4CH, 4CH, 4FH, 00H
; Length: 05H
```

### 6.2 String Copy

```
PROGRAM: Copy string from source to destination

; Input:  Source string at 2000H, Destination at 3000H
; Output: String copied including null terminator

        ORG 0100H
        
        LXI H, 2000H    ; Source
        LXI D, 3000H    ; Destination
        
LOOP:   MOV A, M        ; Get character
        STAX D          ; Copy to destination
        ORA A           ; Check if null
        JZ DONE         ; If null, done
        INX H
        INX D
        JMP LOOP
        
DONE:   HLT
```

### 6.3 String Compare

```
PROGRAM: Compare two strings (lexicographic)

; Input:  String1 at 2000H, String2 at 3000H
; Output: 00H if equal, 01H if String1 > String2, FFH if String1 < String2
;         Result at 2050H

        ORG 0100H
        
        LXI H, 2000H    ; String 1
        LXI D, 3000H    ; String 2
        
LOOP:   MOV A, M        ; Get char from String1
        MOV B, A
        LDAX D          ; Get char from String2
        CMP B           ; Compare
        JC STR1_GREATER ; If A < B (String2 char < String1 char)
        JNZ STR2_GREATER ; If not equal and not carry
        
        ; Characters equal, check for end
        ORA A           ; Is it null?
        JZ EQUAL        ; Both strings ended, equal
        
        INX H
        INX D
        JMP LOOP
        
EQUAL:  MVI A, 00H
        JMP STORE
        
STR1_GREATER:
        MVI A, 01H
        JMP STORE
        
STR2_GREATER:
        MVI A, FFH
        
STORE:  STA 2050H
        HLT
```

### 6.4 String Reverse

```
PROGRAM: Reverse a string in place

; Input:  String at 2000H (null-terminated)
; Output: String reversed in same location

        ORG 0100H
        
        ; First, find string length
        LXI H, 2000H
        MVI C, 00H
        
FINDLEN:
        MOV A, M
        ORA A
        JZ GOTLEN
        INR C
        INX H
        JMP FINDLEN
        
GOTLEN: ; C = length, HL points to null
        MOV A, C
        ORA A
        JZ DONE         ; Empty string
        CPI 01H
        JZ DONE         ; Single character
        
        ; Set up pointers
        DCX H           ; HL = end of string (last char)
        LXI D, 2000H    ; DE = start of string
        
        ; Calculate number of swaps = length/2
        MOV A, C
        RRC             ; Divide by 2
        MOV C, A        ; C = swap count
        
SWAP:   ; Exchange characters at HL and DE
        LDAX D          ; A = start char
        MOV B, A        ; Save in B
        MOV A, M        ; A = end char
        STAX D          ; Store end at start
        MOV M, B        ; Store start at end
        
        INX D           ; Move start forward
        DCX H           ; Move end backward
        DCR C
        JNZ SWAP
        
DONE:   HLT

; EXAMPLE:
; Before: "HELLO" = 48H, 45H, 4CH, 4CH, 4FH, 00H
; After:  "OLLEH" = 4FH, 4CH, 4CH, 45H, 48H, 00H
```

### 6.5 String Concatenate

```
PROGRAM: Concatenate two strings

; Input:  String1 at 2000H, String2 at 3000H
; Output: String1 + String2 at 4000H

        ORG 0100H
        
        LXI H, 2000H    ; Source 1
        LXI D, 4000H    ; Destination
        
        ; Copy String1
COPY1:  MOV A, M
        ORA A
        JZ COPY2_INIT
        STAX D
        INX H
        INX D
        JMP COPY1
        
        ; Point to String2
COPY2_INIT:
        LXI H, 3000H    ; Source 2
        
        ; Copy String2 (including null)
COPY2:  MOV A, M
        STAX D
        ORA A
        JZ DONE
        INX H
        INX D
        JMP COPY2
        
DONE:   HLT

; EXAMPLE:
; String1: "HELLO" (5 chars + null)
; String2: "WORLD" (5 chars + null)
; Result:  "HELLOWORLD" (10 chars + null)
```

---

## 7. ASCII Conversion

### 7.1 Hex to ASCII

```
PROGRAM: Convert hex digit (0-F) to ASCII

; Input:  Hex digit at 2000H (00H-0FH)
; Output: ASCII character at 2001H

        ORG 0100H
        
        LDA 2000H       ; Get hex digit
        CPI 0AH         ; Compare with 10
        JC NUMERIC      ; If < 10, it's 0-9
        
        ; It's A-F
        ADI 37H         ; Add 37H ('A'-10 = 41H-0AH = 37H)
        JMP STORE
        
NUMERIC:
        ADI 30H         ; Add 30H ('0' = 30H)
        
STORE:  STA 2001H
        HLT

; EXAMPLES:
; Input 05H → Output 35H ('5')
; Input 0AH → Output 41H ('A')
; Input 0FH → Output 46H ('F')
```

### 7.2 ASCII to Hex

```
PROGRAM: Convert ASCII character to hex digit

; Input:  ASCII char at 2000H ('0'-'9', 'A'-'F')
; Output: Hex value at 2001H

        ORG 0100H
        
        LDA 2000H       ; Get ASCII char
        CPI 3AH         ; Compare with ':'  (one past '9')
        JC NUMERIC      ; If < ':', it's '0'-'9'
        
        ; It's 'A'-'F'
        SUI 37H         ; Subtract 37H
        JMP STORE
        
NUMERIC:
        SUI 30H         ; Subtract 30H
        
STORE:  STA 2001H
        HLT

; EXAMPLES:
; Input '5' (35H) → Output 05H
; Input 'A' (41H) → Output 0AH
; Input 'F' (46H) → Output 0FH
```

### 7.3 Byte to ASCII String

```
PROGRAM: Convert byte to 2-character hex string

; Input:  Byte at 2000H
; Output: ASCII string at 2001H-2002H

        ORG 0100H
        
        LDA 2000H       ; Get byte
        MOV B, A        ; Save copy
        
        ; Process upper nibble
        RRC
        RRC
        RRC
        RRC             ; Shift upper nibble to lower
        ANI 0FH         ; Mask
        CPI 0AH
        JC UPPER_NUM
        ADI 37H
        JMP STORE_UPPER
UPPER_NUM:
        ADI 30H
STORE_UPPER:
        STA 2001H       ; Store upper digit
        
        ; Process lower nibble
        MOV A, B
        ANI 0FH         ; Mask lower nibble
        CPI 0AH
        JC LOWER_NUM
        ADI 37H
        JMP STORE_LOWER
LOWER_NUM:
        ADI 30H
STORE_LOWER:
        STA 2002H       ; Store lower digit
        
        HLT

; EXAMPLE:
; Input: 5AH
; Output: "5A" → 2001H = 35H ('5'), 2002H = 41H ('A')
```

---

## 📋 Summary Table

| Operation | Key Instructions | Notes |
|-----------|-----------------|-------|
| Array access | MOV A,M / INX H | HL as pointer |
| Find max | CMP / JC | Compare, skip if less |
| Find min | CMP / JNC | Compare, skip if greater or equal |
| Count elements | DCR / JNZ | Loop with counter |
| Block copy | MOV A,M / STAX D | HL=src, DE=dst |
| String length | ORA A / JZ | Check for null (00H) |
| Hex to ASCII | ADI 30H/37H | +30 for 0-9, +37 for A-F |
| ASCII to Hex | SUI 30H/37H | Reverse of above |

---

## ❓ Quick Revision Questions

1. **How do you find the end of a null-terminated string?**
   <details>
   <summary>Show Answer</summary>
   Load each character and test if it's zero using ORA A instruction. If Z flag is set (result is zero), you've found the null terminator marking the end of the string.
   </details>

2. **What's the difference between finding max and min in an array?**
   <details>
   <summary>Show Answer</summary>
   For maximum: use JC (jump if carry) after CMP - this skips the update when new element is smaller. For minimum: use JNC (jump no carry) - this skips when new element is greater or equal. Rest of the logic is identical.
   </details>

3. **How do you count occurrences of a specific value in an array?**
   <details>
   <summary>Show Answer</summary>
   Load target value into register B, traverse array with HL pointer, compare each element with B using CMP B. If Z flag is set (equal), increment counter. Continue until count reaches zero.
   </details>

4. **How do you convert hex digit 0AH to ASCII 'A'?**
   <details>
   <summary>Show Answer</summary>
   For digits A-F: add 37H. ASCII 'A' = 41H, hex 0AH. So 0AH + 37H = 41H. For digits 0-9: add 30H (ASCII '0' = 30H). First compare with 0AH to determine which path.
   </details>

5. **What's the key consideration when copying overlapping memory blocks?**
   <details>
   <summary>Show Answer</summary>
   If source and destination overlap, direction matters. If destination is higher than source, copy backwards (end to start) to avoid overwriting source data before it's copied. If destination is lower, copy forwards.
   </details>

6. **Write the comparison sequence to check if A > B (strictly greater).**
   <details>
   <summary>Show Answer</summary>
   CMP B followed by checking both flags: JC NOT_GREATER (A < B), JZ NOT_GREATER (A = B), otherwise A > B. Or use JNC followed by JNZ. Cannot be determined with single conditional jump.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [3.4 Logical and Bit Operations](04-logical-bit-operations.md) | [Unit 3 Index](README.md) | [3.6 Sorting and Searching](06-sorting-searching.md) |

---

*[← Previous: Logical and Bit Operations](04-logical-bit-operations.md) | [Next: Sorting and Searching →](06-sorting-searching.md)*
