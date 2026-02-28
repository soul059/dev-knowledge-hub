# Chapter 5.5: String Operations

## 📚 Chapter Overview

This chapter covers the powerful string instructions of the 8086 processor. These instructions operate on blocks of data efficiently, making them ideal for string manipulation, memory operations, and data processing.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Use string instructions for data movement
- Apply REP prefix for repetition
- Perform string comparisons and searches
- Implement efficient memory operations
- Create text processing programs

---

## 1. String Instruction Basics

### 1.1 String Instruction Overview

```
STRING INSTRUCTION FUNDAMENTALS:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   REGISTER USAGE:                                                │
│   • SI (Source Index) - Points to source string in DS          │
│   • DI (Destination Index) - Points to destination in ES       │
│   • CX - Counter for REP prefix                                 │
│   • AL/AX - Data for LODS, STOS, SCAS                          │
│   • DF (Direction Flag) - Controls direction                   │
│                                                                  │
│   DIRECTION FLAG:                                                │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   CLD - Clear Direction Flag (DF = 0)                   │  │
│   │   • SI and DI INCREMENT after each operation            │  │
│   │   • Process from low to high addresses                  │  │
│   │                                                          │  │
│   │   STD - Set Direction Flag (DF = 1)                     │  │
│   │   • SI and DI DECREMENT after each operation            │  │
│   │   • Process from high to low addresses                  │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   SEGMENT DEFAULTS:                                              │
│   • Source: DS:SI (can override with segment prefix)           │
│   • Destination: ES:DI (cannot override)                       │
│                                                                  │
│   AUTO-INCREMENT/DECREMENT:                                      │
│   • Byte operations: ±1                                        │
│   • Word operations: ±2                                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 String Instructions Summary

```
STRING INSTRUCTIONS:

┌───────────────────────────────────────────────────────────────────┐
│                                                                    │
│   Instruction   │ Operation                │ Operands Modified    │
│   ──────────────┼──────────────────────────┼─────────────────────│
│   MOVSB/MOVSW   │ Move string (byte/word)  │ [ES:DI]←[DS:SI]     │
│   CMPSB/CMPSW   │ Compare strings          │ Flags from          │
│                 │                          │ [DS:SI]-[ES:DI]     │
│   SCASB/SCASW   │ Scan string              │ Flags from          │
│                 │                          │ AL/AX - [ES:DI]     │
│   LODSB/LODSW   │ Load string              │ AL/AX ← [DS:SI]     │
│   STOSB/STOSW   │ Store string             │ [ES:DI] ← AL/AX     │
│                                                                    │
│   REP PREFIXES:                                                    │
│   ──────────────┼──────────────────────────┼─────────────────────│
│   REP           │ Repeat CX times          │ For MOVS, STOS, LODS│
│   REPE/REPZ     │ Repeat while equal/zero  │ For CMPS, SCAS     │
│   REPNE/REPNZ   │ Repeat while not equal   │ For CMPS, SCAS     │
│                                                                    │
└───────────────────────────────────────────────────────────────────┘
```

---

## 2. Individual String Instructions

### 2.1 MOVS - Move String

```
MOVS - Move String:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   MOVSB - Move String Byte:                                     │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   [ES:DI] ← [DS:SI]          ; Move byte                │  │
│   │   If DF=0: SI++, DI++                                   │  │
│   │   If DF=1: SI--, DI--                                   │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   MOVSW - Move String Word:                                     │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   [ES:DI] ← [DS:SI]          ; Move word                │  │
│   │   If DF=0: SI+=2, DI+=2                                 │  │
│   │   If DF=1: SI-=2, DI-=2                                 │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   EXAMPLE: Copy 100 bytes                                        │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   CLD                     ; Forward direction           │  │
│   │   LEA SI, SOURCE          ; DS:SI = source             │  │
│   │   LEA DI, DEST            ; ES:DI = destination        │  │
│   │   MOV CX, 100             ; Byte count                 │  │
│   │   REP MOVSB               ; Copy 100 bytes             │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   EXAMPLE: Copy overlapping regions (backward)                  │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   ; When destination > source and they overlap         │  │
│   │   STD                     ; Backward direction          │  │
│   │   LEA SI, SOURCE+99       ; Start from end             │  │
│   │   LEA DI, DEST+99                                       │  │
│   │   MOV CX, 100                                           │  │
│   │   REP MOVSB                                             │  │
│   │   CLD                     ; Restore forward             │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 CMPS - Compare Strings

```
CMPS - Compare Strings:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   CMPSB - Compare String Byte:                                  │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   Flags ← [DS:SI] - [ES:DI]  ; Compare, set flags      │  │
│   │   If DF=0: SI++, DI++                                   │  │
│   │   If DF=1: SI--, DI--                                   │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   CMPSW - Compare String Word:                                  │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   Flags ← [DS:SI] - [ES:DI]  ; Compare words           │  │
│   │   SI and DI adjusted by ±2                             │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   FLAGS AFFECTED: CF, ZF, SF, OF, PF, AF                        │
│                                                                  │
│   EXAMPLE: Compare two strings                                   │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   ; Compare STR1 with STR2 (10 bytes)                   │  │
│   │   CLD                                                   │  │
│   │   LEA SI, STR1                                          │  │
│   │   LEA DI, STR2                                          │  │
│   │   MOV CX, 10                                            │  │
│   │   REPE CMPSB              ; Compare while equal         │  │
│   │   JE STRINGS_EQUAL        ; ZF=1 if all matched        │  │
│   │   JA STR1_GREATER         ; STR1 > STR2                │  │
│   │   JB STR1_LESS            ; STR1 < STR2                │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   IMPORTANT:                                                     │
│   • After REPE CMPSB, SI and DI point PAST mismatch            │
│   • To find mismatch position: DEC SI, DEC DI                  │
│   • CX = remaining count (0 if all matched)                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.3 SCAS - Scan String

```
SCAS - Scan String:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   SCASB - Scan String Byte:                                     │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   Flags ← AL - [ES:DI]       ; Compare AL with memory  │  │
│   │   If DF=0: DI++                                         │  │
│   │   If DF=1: DI--                                         │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   SCASW - Scan String Word:                                     │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   Flags ← AX - [ES:DI]       ; Compare AX with memory  │  │
│   │   DI adjusted by ±2                                    │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   EXAMPLE: Find character in string                              │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   ; Find 'X' in STRING (max 100 chars)                  │  │
│   │   CLD                                                   │  │
│   │   LEA DI, STRING                                        │  │
│   │   MOV AL, 'X'             ; Character to find          │  │
│   │   MOV CX, 100             ; Max length                 │  │
│   │   REPNE SCASB             ; Scan while not equal       │  │
│   │   JNE NOT_FOUND           ; ZF=0 if not found          │  │
│   │   DEC DI                  ; DI points past match       │  │
│   │   ; DI now points to 'X'                               │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   EXAMPLE: Count spaces in string                                │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   CLD                                                   │  │
│   │   LEA DI, STRING                                        │  │
│   │   MOV AL, ' '                                           │  │
│   │   MOV CX, LENGTH                                        │  │
│   │   XOR BX, BX              ; Counter = 0                │  │
│   │   JCXZ DONE               ; Handle empty string        │  │
│   │                                                          │  │
│   │   COUNT_LOOP:                                           │  │
│   │       SCASB                                             │  │
│   │       JNE NOT_SPACE                                     │  │
│   │       INC BX              ; Count++                    │  │
│   │   NOT_SPACE:                                            │  │
│   │       LOOP COUNT_LOOP                                   │  │
│   │   DONE:                                                 │  │
│   │   ; BX = number of spaces                              │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.4 LODS - Load String

```
LODS - Load String:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   LODSB - Load String Byte:                                     │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   AL ← [DS:SI]               ; Load byte into AL       │  │
│   │   If DF=0: SI++                                         │  │
│   │   If DF=1: SI--                                         │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   LODSW - Load String Word:                                     │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   AX ← [DS:SI]               ; Load word into AX       │  │
│   │   If DF=0: SI+=2                                        │  │
│   │   If DF=1: SI-=2                                        │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   NOTE: REP LODS rarely used (overwrites AL/AX each time)      │
│         Usually used in loops with processing                  │
│                                                                  │
│   EXAMPLE: Convert lowercase to uppercase                        │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   CLD                                                   │  │
│   │   LEA SI, SOURCE                                        │  │
│   │   LEA DI, DEST                                          │  │
│   │   MOV CX, LENGTH                                        │  │
│   │                                                          │  │
│   │   CONVERT_LOOP:                                         │  │
│   │       LODSB               ; AL = [DS:SI++]              │  │
│   │       CMP AL, 'a'                                       │  │
│   │       JB STORE            ; Below 'a', no change       │  │
│   │       CMP AL, 'z'                                       │  │
│   │       JA STORE            ; Above 'z', no change       │  │
│   │       SUB AL, 20H         ; Convert to uppercase       │  │
│   │   STORE:                                                │  │
│   │       STOSB               ; [ES:DI++] = AL             │  │
│   │       LOOP CONVERT_LOOP                                 │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.5 STOS - Store String

```
STOS - Store String:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   STOSB - Store String Byte:                                    │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   [ES:DI] ← AL               ; Store AL to memory      │  │
│   │   If DF=0: DI++                                         │  │
│   │   If DF=1: DI--                                         │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   STOSW - Store String Word:                                    │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   [ES:DI] ← AX               ; Store AX to memory      │  │
│   │   If DF=0: DI+=2                                        │  │
│   │   If DF=1: DI-=2                                        │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   EXAMPLE: Fill memory with value                                │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   ; Fill 1000 bytes with 0                             │  │
│   │   CLD                                                   │  │
│   │   LEA DI, BUFFER                                        │  │
│   │   XOR AL, AL              ; AL = 0                     │  │
│   │   MOV CX, 1000                                          │  │
│   │   REP STOSB               ; Fill with zeros            │  │
│   │                                                          │  │
│   │   ; Faster: Fill 500 words with 0                      │  │
│   │   LEA DI, BUFFER                                        │  │
│   │   XOR AX, AX              ; AX = 0                     │  │
│   │   MOV CX, 500                                           │  │
│   │   REP STOSW               ; Fill with word zeros       │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   EXAMPLE: Fill with pattern                                     │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   ; Fill 80 bytes with '*'                             │  │
│   │   CLD                                                   │  │
│   │   LEA DI, LINE                                          │  │
│   │   MOV AL, '*'                                           │  │
│   │   MOV CX, 80                                            │  │
│   │   REP STOSB                                             │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. REP Prefixes

### 3.1 REP Prefix Operation

```
REP PREFIX BEHAVIOR:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   REP - Repeat While CX ≠ 0:                                    │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   Used with: MOVS, STOS, LODS                           │  │
│   │                                                          │  │
│   │   Operation:                                            │  │
│   │   WHILE CX ≠ 0:                                         │  │
│   │       Execute string instruction                        │  │
│   │       CX = CX - 1                                       │  │
│   │   END WHILE                                             │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   REPE/REPZ - Repeat While Equal (ZF=1) AND CX ≠ 0:            │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   Used with: CMPS, SCAS                                 │  │
│   │                                                          │  │
│   │   Operation:                                            │  │
│   │   WHILE CX ≠ 0 AND ZF = 1:                              │  │
│   │       Execute string instruction                        │  │
│   │       CX = CX - 1                                       │  │
│   │   END WHILE                                             │  │
│   │                                                          │  │
│   │   Stops when: CX=0 OR mismatch found (ZF=0)            │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   REPNE/REPNZ - Repeat While Not Equal (ZF=0) AND CX ≠ 0:      │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   Used with: CMPS, SCAS                                 │  │
│   │                                                          │  │
│   │   Operation:                                            │  │
│   │   WHILE CX ≠ 0 AND ZF = 0:                              │  │
│   │       Execute string instruction                        │  │
│   │       CX = CX - 1                                       │  │
│   │   END WHILE                                             │  │
│   │                                                          │  │
│   │   Stops when: CX=0 OR match found (ZF=1)               │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   CHECKING RESULTS:                                              │
│   • After REPE CMPS: ZF=1 means all matched, ZF=0 mismatch    │
│   • After REPNE SCAS: ZF=1 means found, ZF=0 not found        │
│   • CX = remaining count                                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Practical String Programs

### 4.1 String Copy Function

```asm
;------------------------------------------------
; STRCPY - Copy null-terminated string
; Input:  SI = source, DI = destination
; Output: String copied
;------------------------------------------------
STRCPY PROC
    PUSH AX
    CLD
    
COPY_LOOP:
    LODSB               ; AL = [DS:SI++]
    STOSB               ; [ES:DI++] = AL
    OR AL, AL           ; Check for null
    JNZ COPY_LOOP       ; Continue if not null
    
    POP AX
    RET
STRCPY ENDP
```

### 4.2 String Length Function

```asm
;------------------------------------------------
; STRLEN - Get length of null-terminated string
; Input:  DI = string address (ES:DI)
; Output: CX = length (not including null)
;------------------------------------------------
STRLEN PROC
    PUSH AX
    PUSH DI
    CLD
    
    XOR AL, AL          ; Looking for null
    MOV CX, 0FFFFH      ; Max length
    REPNE SCASB         ; Scan for null
    
    NOT CX              ; CX = -(remaining+1)
    DEC CX              ; CX = length
    
    POP DI
    POP AX
    RET
STRLEN ENDP
```

### 4.3 String Compare Function

```asm
;------------------------------------------------
; STRCMP - Compare two null-terminated strings
; Input:  SI = string1, DI = string2
; Output: ZF=1 if equal, ZF=0 if not
;         AX=0 if equal, >0 if str1>str2, <0 if str1<str2
;------------------------------------------------
STRCMP PROC
    PUSH SI
    PUSH DI
    CLD
    
COMPARE_LOOP:
    LODSB               ; AL = [DS:SI++]
    SCASB               ; Compare with [ES:DI++]
    JNE NOT_EQUAL       ; If not equal, exit
    OR AL, AL           ; Check for end of string
    JNZ COMPARE_LOOP    ; Continue if not end
    
    XOR AX, AX          ; Strings are equal
    JMP STRCMP_EXIT
    
NOT_EQUAL:
    MOV AH, 0
    SUB AL, [DI-1]      ; Difference
    CBW                 ; Sign extend to AX
    
STRCMP_EXIT:
    POP DI
    POP SI
    RET
STRCMP ENDP
```

### 4.4 String Search (Find Substring)

```asm
;------------------------------------------------
; STRSTR - Find substring in string
; Input:  SI = main string, DI = substring
;         CX = main string length
;         DX = substring length
; Output: CF=0 and SI=position if found
;         CF=1 if not found
;------------------------------------------------
STRSTR PROC
    PUSH BX
    PUSH CX
    PUSH DI
    CLD
    
    ; Save original values
    MOV BX, SI          ; Save start of main string
    
SEARCH_LOOP:
    PUSH SI
    PUSH DI
    PUSH CX
    
    MOV CX, DX          ; Substring length
    REPE CMPSB          ; Compare
    
    POP CX
    POP DI
    POP SI
    
    JE FOUND            ; ZF=1 means match
    
    INC SI              ; Try next position
    LOOP SEARCH_LOOP    ; Decrement main counter
    
    ; Not found
    STC                 ; Set carry flag
    JMP STRSTR_EXIT
    
FOUND:
    CLC                 ; Clear carry - found!
    ; SI points to match position
    
STRSTR_EXIT:
    POP DI
    POP CX
    POP BX
    RET
STRSTR ENDP
```

### 4.5 Complete String Program

```asm
;------------------------------------------------
; STRING PROCESSING PROGRAM
;------------------------------------------------
.MODEL SMALL
.STACK 100H

.DATA
    STR1    DB  'Hello World', 0
    STR2    DB  'HELLO WORLD', 0
    STR3    DB  50 DUP(0)
    
    MSG1    DB  'String 1: $'
    MSG2    DB  0DH, 0AH, 'String 2: $'
    MSG3    DB  0DH, 0AH, 'Strings are equal$'
    MSG4    DB  0DH, 0AH, 'Strings are different$'
    MSG5    DB  0DH, 0AH, 'Length: $'
    MSG6    DB  0DH, 0AH, 'Reversed: $'
    NEWLN   DB  0DH, 0AH, '$'

.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX
    MOV ES, AX          ; Important for string ops
    
    ; Display strings
    MOV AH, 09H
    LEA DX, MSG1
    INT 21H
    LEA DX, STR1
    CALL PRINT_ASCIIZ
    
    MOV AH, 09H
    LEA DX, MSG2
    INT 21H
    LEA DX, STR2
    CALL PRINT_ASCIIZ
    
    ; Compare strings (case sensitive)
    LEA SI, STR1
    LEA DI, STR2
    CALL STRCMP
    JZ ARE_EQUAL
    
    MOV AH, 09H
    LEA DX, MSG4
    INT 21H
    JMP SHOW_LENGTH
    
ARE_EQUAL:
    MOV AH, 09H
    LEA DX, MSG3
    INT 21H
    
SHOW_LENGTH:
    ; Get and display length
    MOV AH, 09H
    LEA DX, MSG5
    INT 21H
    
    LEA DI, STR1
    CALL STRLEN
    MOV AX, CX
    CALL PRINT_NUM
    
    ; Reverse string
    LEA SI, STR1
    LEA DI, STR3
    CALL STRREV
    
    MOV AH, 09H
    LEA DX, MSG6
    INT 21H
    LEA DX, STR3
    CALL PRINT_ASCIIZ
    
    ; Exit
    MOV AH, 4CH
    INT 21H
MAIN ENDP

;------------------------------------------------
; PRINT_ASCIIZ - Print null-terminated string
; Input: DX = string address
;------------------------------------------------
PRINT_ASCIIZ PROC
    PUSH AX
    PUSH SI
    MOV SI, DX
    
PRINT_LOOP:
    LODSB
    OR AL, AL
    JZ PRINT_DONE
    MOV AH, 02H
    MOV DL, AL
    INT 21H
    JMP PRINT_LOOP
    
PRINT_DONE:
    POP SI
    POP AX
    RET
PRINT_ASCIIZ ENDP

;------------------------------------------------
; STRREV - Reverse string
; Input: SI = source, DI = destination
;------------------------------------------------
STRREV PROC
    PUSH AX
    PUSH SI
    PUSH DI
    PUSH CX
    
    ; First, find length
    PUSH DI
    MOV DI, SI
    CALL STRLEN         ; CX = length
    POP DI
    
    ; Point to end of source
    ADD SI, CX
    DEC SI
    
    ; Copy in reverse
    STD                 ; Decrement SI
REV_LOOP:
    LODSB               ; AL = [DS:SI--]
    MOV [DI], AL        ; Store forward
    INC DI
    LOOP REV_LOOP
    
    MOV BYTE PTR [DI], 0 ; Null terminate
    CLD                 ; Restore direction
    
    POP CX
    POP DI
    POP SI
    POP AX
    RET
STRREV ENDP

;------------------------------------------------
; STRLEN - String length
;------------------------------------------------
STRLEN PROC
    PUSH AX
    PUSH DI
    CLD
    XOR AL, AL
    MOV CX, 0FFFFH
    REPNE SCASB
    NOT CX
    DEC CX
    POP DI
    POP AX
    RET
STRLEN ENDP

;------------------------------------------------
; STRCMP - String compare
;------------------------------------------------
STRCMP PROC
    PUSH SI
    PUSH DI
    CLD
CMP_LOOP:
    LODSB
    SCASB
    JNE CMP_DONE
    OR AL, AL
    JNZ CMP_LOOP
CMP_DONE:
    POP DI
    POP SI
    RET
STRCMP ENDP

;------------------------------------------------
; PRINT_NUM - Print number in AX
;------------------------------------------------
PRINT_NUM PROC
    PUSH AX
    PUSH BX
    PUSH CX
    PUSH DX
    
    MOV BX, 10
    XOR CX, CX
    
PN_DIV:
    XOR DX, DX
    DIV BX
    PUSH DX
    INC CX
    OR AX, AX
    JNZ PN_DIV
    
PN_PRINT:
    POP DX
    ADD DL, '0'
    MOV AH, 02H
    INT 21H
    LOOP PN_PRINT
    
    POP DX
    POP CX
    POP BX
    POP AX
    RET
PRINT_NUM ENDP

END MAIN
```

---

## 📋 Summary Table

| Instruction | Operation | Registers Used |
|-------------|-----------|----------------|
| MOVSB/W | [ES:DI] ← [DS:SI] | SI, DI |
| CMPSB/W | Flags ← [DS:SI] - [ES:DI] | SI, DI, FLAGS |
| SCASB/W | Flags ← AL/AX - [ES:DI] | AL/AX, DI, FLAGS |
| LODSB/W | AL/AX ← [DS:SI] | AL/AX, SI |
| STOSB/W | [ES:DI] ← AL/AX | AL/AX, DI |
| REP | Repeat CX times | CX |
| REPE/REPZ | Repeat while ZF=1 | CX, FLAGS |
| REPNE/REPNZ | Repeat while ZF=0 | CX, FLAGS |
| CLD | Clear DF (forward) | DF |
| STD | Set DF (backward) | DF |

---

## ❓ Quick Revision Questions

1. **What is the default segment for source and destination in string operations?**
   <details>
   <summary>Show Answer</summary>
   Source uses DS:SI by default (can be overridden with segment prefix). Destination uses ES:DI (cannot be overridden). This is why ES must be set up correctly before string operations.
   </details>

2. **What does the Direction Flag control?**
   <details>
   <summary>Show Answer</summary>
   DF controls whether SI and DI increment (DF=0, set by CLD) or decrement (DF=1, set by STD) after each string operation. CLD = forward direction (low to high addresses), STD = backward direction (high to low addresses).
   </details>

3. **What's the difference between REPE and REPNE?**
   <details>
   <summary>Show Answer</summary>
   REPE (Repeat while Equal) continues while ZF=1 AND CX≠0 - stops at mismatch or when count exhausted. REPNE (Repeat while Not Equal) continues while ZF=0 AND CX≠0 - stops at match or when count exhausted. Used with CMPS and SCAS.
   </details>

4. **How do you find a character in a string using string instructions?**
   <details>
   <summary>Show Answer</summary>
   Load character to find into AL, set CX to string length, point DI to string, use CLD for forward direction, then REPNE SCASB. If ZF=1 after, character was found and DI points one past the match (DEC DI to get position). If ZF=0, character not found.
   </details>

5. **Why is REP LODSB rarely useful?**
   <details>
   <summary>Show Answer</summary>
   REP LODSB loads successive bytes into AL, but each load overwrites the previous value. Only the last byte remains in AL. It's more useful to use LODSB in a loop where you process each byte between loads (like character conversion or searching).
   </details>

6. **How do you handle overlapping memory regions when copying?**
   <details>
   <summary>Show Answer</summary>
   If source < destination (and they overlap), copy forward (CLD, REP MOVSB) might overwrite source before reading. Solution: copy backward using STD, start from end of both regions, REP MOVSB. If destination < source, forward copy is safe.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [5.4 Interrupt Programming](04-interrupt-programming.md) | [Unit 5 Index](README.md) | [5.6 Advanced Programs](06-advanced-programs.md) |

---

*[← Previous: Interrupt Programming](04-interrupt-programming.md) | [Next: Advanced Programs →](06-advanced-programs.md)*
