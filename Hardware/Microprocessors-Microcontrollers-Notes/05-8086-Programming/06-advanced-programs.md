# Chapter 5.6: Advanced Programs

## 📚 Chapter Overview

This chapter presents complete advanced 8086 assembly programs including sorting algorithms, searching techniques, mathematical operations, and file handling. These examples demonstrate practical application of all concepts covered in the unit.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Implement sorting algorithms in assembly
- Write searching programs
- Perform multi-byte arithmetic
- Handle file I/O operations
- Create complete application programs

---

## 1. Sorting Algorithms

### 1.1 Bubble Sort

```asm
;------------------------------------------------
; BUBBLE SORT - Sort array of bytes (ascending)
;------------------------------------------------
.MODEL SMALL
.STACK 100H

.DATA
    ARRAY   DB  64, 25, 12, 22, 11, 90, 42, 33
    COUNT   EQU $ - ARRAY
    
    MSG1    DB  'Before sorting: $'
    MSG2    DB  0DH, 0AH, 'After sorting:  $'
    SPACE   DB  ' $'
    NEWLN   DB  0DH, 0AH, '$'

.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX
    
    ; Display original array
    MOV AH, 09H
    LEA DX, MSG1
    INT 21H
    LEA SI, ARRAY
    MOV CX, COUNT
    CALL PRINT_ARRAY
    
    ; Perform bubble sort
    LEA SI, ARRAY
    MOV CX, COUNT
    CALL BUBBLE_SORT
    
    ; Display sorted array
    MOV AH, 09H
    LEA DX, MSG2
    INT 21H
    LEA SI, ARRAY
    MOV CX, COUNT
    CALL PRINT_ARRAY
    
    ; Exit
    MOV AH, 4CH
    INT 21H
MAIN ENDP

;------------------------------------------------
; BUBBLE_SORT - Sort byte array ascending
; Input: SI = array address, CX = count
;------------------------------------------------
BUBBLE_SORT PROC
    PUSH AX
    PUSH BX
    PUSH CX
    PUSH DX
    PUSH SI
    
    DEC CX                  ; Outer loop count = n-1
    JZ SORT_DONE            ; If only 1 element, done
    
OUTER_LOOP:
    PUSH CX                 ; Save outer counter
    MOV BX, SI              ; BX = start of array
    MOV DX, 0               ; Swap flag = 0
    
INNER_LOOP:
    MOV AL, [BX]            ; Get current element
    CMP AL, [BX+1]          ; Compare with next
    JBE NO_SWAP             ; If <= , no swap needed
    
    ; Swap elements
    XCHG AL, [BX+1]         ; Swap using XCHG
    MOV [BX], AL
    MOV DX, 1               ; Set swap flag
    
NO_SWAP:
    INC BX                  ; Next element
    LOOP INNER_LOOP
    
    POP CX                  ; Restore outer counter
    OR DX, DX               ; Check if any swaps
    JZ SORT_DONE            ; No swaps = sorted
    LOOP OUTER_LOOP
    
SORT_DONE:
    POP SI
    POP DX
    POP CX
    POP BX
    POP AX
    RET
BUBBLE_SORT ENDP

;------------------------------------------------
; PRINT_ARRAY - Print array elements
; Input: SI = array, CX = count
;------------------------------------------------
PRINT_ARRAY PROC
    PUSH AX
    PUSH BX
    PUSH CX
    PUSH DX
    PUSH SI
    
PRINT_LOOP:
    XOR AX, AX
    LODSB                   ; AL = [SI++]
    CALL PRINT_NUM
    
    MOV AH, 09H
    LEA DX, SPACE
    INT 21H
    
    LOOP PRINT_LOOP
    
    POP SI
    POP DX
    POP CX
    POP BX
    POP AX
    RET
PRINT_ARRAY ENDP

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

### 1.2 Selection Sort

```asm
;------------------------------------------------
; SELECTION SORT - Sort word array
;------------------------------------------------
SELECTION_SORT PROC
    ; Input: SI = array address, CX = count (words)
    PUSH AX
    PUSH BX
    PUSH CX
    PUSH DX
    PUSH SI
    PUSH DI
    
    DEC CX                  ; n-1 passes
    JZ SEL_DONE
    MOV DI, SI              ; DI = current position
    
OUTER_SEL:
    PUSH CX
    MOV BX, DI              ; BX = position of min
    MOV SI, DI
    ADD SI, 2               ; Start from next element
    
    ; Find minimum in remaining array
FIND_MIN:
    MOV AX, [BX]            ; Current minimum
    CMP AX, [SI]            ; Compare with element
    JLE NOT_SMALLER
    MOV BX, SI              ; New minimum position
NOT_SMALLER:
    ADD SI, 2
    LOOP FIND_MIN
    
    ; Swap if needed
    CMP BX, DI
    JE NO_SEL_SWAP
    MOV AX, [DI]
    XCHG AX, [BX]
    MOV [DI], AX
    
NO_SEL_SWAP:
    ADD DI, 2               ; Next position
    POP CX
    LOOP OUTER_SEL
    
SEL_DONE:
    POP DI
    POP SI
    POP DX
    POP CX
    POP BX
    POP AX
    RET
SELECTION_SORT ENDP
```

---

## 2. Searching Algorithms

### 2.1 Linear Search

```asm
;------------------------------------------------
; LINEAR SEARCH - Find element in array
;------------------------------------------------
.MODEL SMALL
.STACK 100H

.DATA
    ARRAY   DB  45, 23, 78, 12, 89, 34, 56, 90
    COUNT   EQU $ - ARRAY
    KEY     DB  34
    
    MSG1    DB  'Enter number to find: $'
    FOUND   DB  0DH, 0AH, 'Found at position: $'
    NOTFND  DB  0DH, 0AH, 'Not found!$'

.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX
    
    ; Search for KEY
    LEA SI, ARRAY
    MOV CX, COUNT
    MOV AL, KEY
    CALL LINEAR_SEARCH
    
    JNC DISPLAY_FOUND       ; CF=0 means found
    
    ; Not found
    MOV AH, 09H
    LEA DX, NOTFND
    INT 21H
    JMP EXIT
    
DISPLAY_FOUND:
    PUSH AX                 ; Save position
    MOV AH, 09H
    LEA DX, FOUND
    INT 21H
    POP AX
    INC AL                  ; 1-based position
    XOR AH, AH
    CALL PRINT_NUM
    
EXIT:
    MOV AH, 4CH
    INT 21H
MAIN ENDP

;------------------------------------------------
; LINEAR_SEARCH - Search for value in array
; Input: SI = array, CX = count, AL = value
; Output: CF=0 if found (AL = position 0-based)
;         CF=1 if not found
;------------------------------------------------
LINEAR_SEARCH PROC
    PUSH BX
    PUSH CX
    PUSH SI
    
    XOR BX, BX              ; Position counter
    
SEARCH_LOOP:
    CMP AL, [SI]            ; Compare with element
    JE FOUND_IT             ; Found!
    INC SI
    INC BX
    LOOP SEARCH_LOOP
    
    ; Not found
    STC                     ; Set carry flag
    JMP LS_EXIT
    
FOUND_IT:
    MOV AL, BL              ; Return position
    CLC                     ; Clear carry (found)
    
LS_EXIT:
    POP SI
    POP CX
    POP BX
    RET
LINEAR_SEARCH ENDP

PRINT_NUM PROC
    ; Same as before
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

### 2.2 Binary Search

```asm
;------------------------------------------------
; BINARY SEARCH - Search sorted array
; Input: SI = sorted array (words), CX = count
;        AX = value to find
; Output: CF=0 if found, BX = index
;         CF=1 if not found
;------------------------------------------------
BINARY_SEARCH PROC
    PUSH DX
    PUSH SI
    
    XOR BX, BX              ; Low = 0
    MOV DX, CX
    DEC DX                  ; High = count - 1
    
BS_LOOP:
    CMP BX, DX              ; While low <= high
    JA BS_NOTFOUND
    
    ; Calculate mid = (low + high) / 2
    PUSH AX
    MOV AX, BX
    ADD AX, DX
    SHR AX, 1               ; AX = mid
    PUSH BX
    MOV BX, AX              ; BX = mid
    SHL AX, 1               ; AX = mid * 2 (word offset)
    PUSH SI
    ADD SI, AX              ; SI points to array[mid]
    POP AX                  ; Restore to use as temp
    MOV AX, [SI]            ; Get array[mid]
    MOV SI, AX              ; SI = array[mid]
    POP AX                  ; Restore original SI
    ; BX = mid, SI = array[mid] value
    ; ... (this gets complex, simplified version:)
    
    ; Simplified approach:
    POP AX
    
    ; Calculate mid index
    MOV CX, BX
    ADD CX, DX
    SHR CX, 1               ; CX = mid
    
    ; Compare array[mid] with key
    PUSH SI
    PUSH CX
    SHL CX, 1               ; Word offset
    ADD SI, CX
    POP CX
    CMP AX, [SI]
    POP SI
    
    JE BS_FOUND
    JL BS_LEFT              ; Key < array[mid]
    
    ; Key > array[mid]: low = mid + 1
    MOV BX, CX
    INC BX
    JMP BS_LOOP
    
BS_LEFT:
    ; Key < array[mid]: high = mid - 1
    MOV DX, CX
    DEC DX
    JMP BS_LOOP
    
BS_FOUND:
    MOV BX, CX              ; Return index
    CLC
    JMP BS_EXIT
    
BS_NOTFOUND:
    STC
    
BS_EXIT:
    POP SI
    POP DX
    RET
BINARY_SEARCH ENDP
```

---

## 3. Mathematical Operations

### 3.1 Factorial Calculation

```asm
;------------------------------------------------
; FACTORIAL - Calculate n! (iterative)
;------------------------------------------------
.MODEL SMALL
.STACK 100H

.DATA
    NUM     DB  5
    RESULT  DW  ?
    MSG1    DB  'Factorial of $'
    MSG2    DB  ' = $'

.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX
    
    ; Display "Factorial of "
    MOV AH, 09H
    LEA DX, MSG1
    INT 21H
    
    ; Display number
    XOR AX, AX
    MOV AL, NUM
    PUSH AX
    CALL PRINT_NUM
    
    ; Calculate factorial
    POP AX
    CALL FACTORIAL
    MOV RESULT, AX
    
    ; Display " = "
    PUSH AX
    MOV AH, 09H
    LEA DX, MSG2
    INT 21H
    
    ; Display result
    POP AX
    CALL PRINT_NUM
    
    MOV AH, 4CH
    INT 21H
MAIN ENDP

;------------------------------------------------
; FACTORIAL - Calculate n! (0! = 1, 1! = 1)
; Input: AL = n (0-8 for 16-bit result)
; Output: AX = n!
;------------------------------------------------
FACTORIAL PROC
    PUSH BX
    PUSH CX
    
    XOR AH, AH              ; Clear high byte
    OR AL, AL               ; Check for 0
    JZ FACT_ZERO
    
    MOV CX, AX              ; CX = n
    MOV AX, 1               ; Result = 1
    
FACT_LOOP:
    MUL CX                  ; AX = AX * CX
    LOOP FACT_LOOP          ; Decrement CX, repeat
    JMP FACT_DONE
    
FACT_ZERO:
    MOV AX, 1               ; 0! = 1
    
FACT_DONE:
    POP CX
    POP BX
    RET
FACTORIAL ENDP

; PRINT_NUM procedure same as before
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

### 3.2 GCD (Greatest Common Divisor)

```asm
;------------------------------------------------
; GCD - Euclidean Algorithm
; Input: AX = num1, BX = num2
; Output: AX = GCD
;------------------------------------------------
GCD PROC
    PUSH DX
    
GCD_LOOP:
    OR BX, BX               ; Check if BX = 0
    JZ GCD_DONE
    
    XOR DX, DX              ; Clear DX for division
    DIV BX                  ; AX = quotient, DX = remainder
    MOV AX, BX              ; AX = old BX
    MOV BX, DX              ; BX = remainder
    JMP GCD_LOOP
    
GCD_DONE:
    POP DX
    RET
GCD ENDP

; LCM = (A * B) / GCD(A, B)
;------------------------------------------------
; LCM - Least Common Multiple
; Input: AX = num1, BX = num2
; Output: DX:AX = LCM
;------------------------------------------------
LCM PROC
    PUSH BX
    PUSH CX
    
    MOV CX, BX              ; Save num2
    PUSH AX                 ; Save num1
    
    CALL GCD                ; AX = GCD
    MOV BX, AX              ; BX = GCD
    
    POP AX                  ; AX = num1
    MUL CX                  ; DX:AX = num1 * num2
    DIV BX                  ; AX = (num1*num2)/GCD
    
    POP CX
    POP BX
    RET
LCM ENDP
```

### 3.3 32-bit Addition and Subtraction

```asm
;------------------------------------------------
; 32-BIT ARITHMETIC
;------------------------------------------------
.DATA
    NUM1_LO DW  5678H       ; Lower 16 bits
    NUM1_HI DW  1234H       ; Upper 16 bits (12345678H)
    NUM2_LO DW  0DCBAH
    NUM2_HI DW  0000H       ; (0000DCBAH)
    RES_LO  DW  ?
    RES_HI  DW  ?

.CODE
;------------------------------------------------
; ADD32 - Add two 32-bit numbers
;------------------------------------------------
ADD32 PROC
    ; Result = NUM1 + NUM2
    MOV AX, NUM1_LO
    ADD AX, NUM2_LO         ; Add low words
    MOV RES_LO, AX
    
    MOV AX, NUM1_HI
    ADC AX, NUM2_HI         ; Add high words with carry
    MOV RES_HI, AX
    
    RET
ADD32 ENDP

;------------------------------------------------
; SUB32 - Subtract two 32-bit numbers
;------------------------------------------------
SUB32 PROC
    ; Result = NUM1 - NUM2
    MOV AX, NUM1_LO
    SUB AX, NUM2_LO         ; Subtract low words
    MOV RES_LO, AX
    
    MOV AX, NUM1_HI
    SBB AX, NUM2_HI         ; Subtract with borrow
    MOV RES_HI, AX
    
    RET
SUB32 ENDP
```

---

## 4. File Operations

### 4.1 File Copy Program

```asm
;------------------------------------------------
; FILE COPY PROGRAM
;------------------------------------------------
.MODEL SMALL
.STACK 200H

.DATA
    SOURCE  DB  'SOURCE.TXT', 0     ; Source filename
    DEST    DB  'DEST.TXT', 0       ; Destination filename
    BUFFER  DB  512 DUP(?)          ; Read buffer
    
    SRC_H   DW  ?                   ; Source handle
    DST_H   DW  ?                   ; Destination handle
    
    MSG_OK  DB  'File copied successfully!$'
    MSG_ERR DB  'Error occurred!$'
    MSG_RD  DB  'Error opening source file!$'
    MSG_WR  DB  'Error creating destination file!$'

.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX
    
    ; Open source file
    MOV AH, 3DH
    MOV AL, 0               ; Read mode
    LEA DX, SOURCE
    INT 21H
    JC ERR_READ
    MOV SRC_H, AX           ; Save handle
    
    ; Create destination file
    MOV AH, 3CH
    MOV CX, 0               ; Normal attribute
    LEA DX, DEST
    INT 21H
    JC ERR_WRITE
    MOV DST_H, AX           ; Save handle
    
COPY_LOOP:
    ; Read from source
    MOV AH, 3FH
    MOV BX, SRC_H
    MOV CX, 512             ; Buffer size
    LEA DX, BUFFER
    INT 21H
    JC ERR_GENERAL
    
    OR AX, AX               ; Check bytes read
    JZ COPY_DONE            ; 0 = EOF
    
    ; Write to destination
    MOV CX, AX              ; Bytes to write
    MOV AH, 40H
    MOV BX, DST_H
    LEA DX, BUFFER
    INT 21H
    JC ERR_GENERAL
    
    JMP COPY_LOOP
    
COPY_DONE:
    ; Close files
    MOV AH, 3EH
    MOV BX, SRC_H
    INT 21H
    
    MOV AH, 3EH
    MOV BX, DST_H
    INT 21H
    
    ; Success message
    MOV AH, 09H
    LEA DX, MSG_OK
    INT 21H
    JMP EXIT
    
ERR_READ:
    MOV AH, 09H
    LEA DX, MSG_RD
    INT 21H
    JMP EXIT
    
ERR_WRITE:
    ; Close source if open
    MOV AH, 3EH
    MOV BX, SRC_H
    INT 21H
    MOV AH, 09H
    LEA DX, MSG_WR
    INT 21H
    JMP EXIT
    
ERR_GENERAL:
    MOV AH, 09H
    LEA DX, MSG_ERR
    INT 21H
    
EXIT:
    MOV AH, 4CH
    INT 21H
MAIN ENDP

END MAIN
```

### 4.2 Read and Display File

```asm
;------------------------------------------------
; READ AND DISPLAY TEXT FILE
;------------------------------------------------
.MODEL SMALL
.STACK 100H

.DATA
    FNAME   DB  64 DUP(0)           ; Filename buffer
    BUFFER  DB  1024 DUP(?)         ; File buffer
    HANDLE  DW  ?
    
    PROMPT  DB  'Enter filename: $'
    ERRMSG  DB  0DH, 0AH, 'Cannot open file!$'
    NEWLN   DB  0DH, 0AH, '$'

.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX
    
    ; Prompt for filename
    MOV AH, 09H
    LEA DX, PROMPT
    INT 21H
    
    ; Read filename
    LEA DI, FNAME
    CALL READ_STRING
    
    ; Open file
    MOV AH, 3DH
    MOV AL, 0               ; Read mode
    LEA DX, FNAME
    INT 21H
    JC ERROR
    MOV HANDLE, AX
    
    ; Newline
    MOV AH, 09H
    LEA DX, NEWLN
    INT 21H
    
READ_LOOP:
    ; Read chunk
    MOV AH, 3FH
    MOV BX, HANDLE
    MOV CX, 1024
    LEA DX, BUFFER
    INT 21H
    JC DONE
    
    OR AX, AX               ; EOF?
    JZ DONE
    
    ; Display buffer
    MOV CX, AX
    LEA SI, BUFFER
DISP_LOOP:
    LODSB
    MOV AH, 02H
    MOV DL, AL
    INT 21H
    LOOP DISP_LOOP
    
    JMP READ_LOOP
    
DONE:
    ; Close file
    MOV AH, 3EH
    MOV BX, HANDLE
    INT 21H
    JMP EXIT
    
ERROR:
    MOV AH, 09H
    LEA DX, ERRMSG
    INT 21H
    
EXIT:
    MOV AH, 4CH
    INT 21H
MAIN ENDP

;------------------------------------------------
; READ_STRING - Read null-terminated string
; Input: DI = buffer address
;------------------------------------------------
READ_STRING PROC
    PUSH AX
    
RS_LOOP:
    MOV AH, 01H
    INT 21H
    
    CMP AL, 0DH             ; Enter?
    JE RS_DONE
    
    STOSB                   ; Store character
    JMP RS_LOOP
    
RS_DONE:
    MOV BYTE PTR [DI], 0    ; Null terminate
    
    POP AX
    RET
READ_STRING ENDP

END MAIN
```

---

## 5. Complete Application

### 5.1 Simple Calculator

```asm
;------------------------------------------------
; SIMPLE CALCULATOR
; Supports +, -, *, / operations
;------------------------------------------------
.MODEL SMALL
.STACK 100H

.DATA
    MENU    DB  0DH, 0AH
            DB  '=== CALCULATOR ===', 0DH, 0AH
            DB  '1. Add', 0DH, 0AH
            DB  '2. Subtract', 0DH, 0AH
            DB  '3. Multiply', 0DH, 0AH
            DB  '4. Divide', 0DH, 0AH
            DB  '5. Exit', 0DH, 0AH
            DB  'Choice: $'
    
    NUM1MSG DB  0DH, 0AH, 'Enter first number: $'
    NUM2MSG DB  'Enter second number: $'
    RESMSG  DB  0DH, 0AH, 'Result: $'
    REMSG   DB  ' Remainder: $'
    DIVZER  DB  0DH, 0AH, 'Error: Division by zero!$'
    NEWLN   DB  0DH, 0AH, '$'
    
    NUM1    DW  ?
    NUM2    DW  ?
    RESULT  DW  ?

.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX
    
MAIN_LOOP:
    ; Display menu
    MOV AH, 09H
    LEA DX, MENU
    INT 21H
    
    ; Get choice
    MOV AH, 01H
    INT 21H
    
    CMP AL, '1'
    JE DO_ADD
    CMP AL, '2'
    JE DO_SUB
    CMP AL, '3'
    JE DO_MUL
    CMP AL, '4'
    JE DO_DIV
    CMP AL, '5'
    JE EXIT_PROG
    JMP MAIN_LOOP

DO_ADD:
    CALL GET_NUMBERS
    MOV AX, NUM1
    ADD AX, NUM2
    MOV RESULT, AX
    CALL SHOW_RESULT
    JMP MAIN_LOOP

DO_SUB:
    CALL GET_NUMBERS
    MOV AX, NUM1
    SUB AX, NUM2
    MOV RESULT, AX
    CALL SHOW_RESULT
    JMP MAIN_LOOP

DO_MUL:
    CALL GET_NUMBERS
    MOV AX, NUM1
    MUL NUM2
    MOV RESULT, AX          ; Low word only
    CALL SHOW_RESULT
    JMP MAIN_LOOP

DO_DIV:
    CALL GET_NUMBERS
    MOV AX, NUM2
    OR AX, AX
    JZ DIV_ZERO
    
    MOV AX, NUM1
    XOR DX, DX
    DIV NUM2
    MOV RESULT, AX
    PUSH DX                 ; Save remainder
    CALL SHOW_RESULT
    
    ; Show remainder
    MOV AH, 09H
    LEA DX, REMSG
    INT 21H
    POP AX
    CALL PRINT_NUM
    JMP MAIN_LOOP

DIV_ZERO:
    MOV AH, 09H
    LEA DX, DIVZER
    INT 21H
    JMP MAIN_LOOP

EXIT_PROG:
    MOV AH, 4CH
    INT 21H
MAIN ENDP

;------------------------------------------------
GET_NUMBERS PROC
    MOV AH, 09H
    LEA DX, NUM1MSG
    INT 21H
    CALL READ_NUM
    MOV NUM1, AX
    
    MOV AH, 09H
    LEA DX, NUM2MSG
    INT 21H
    CALL READ_NUM
    MOV NUM2, AX
    RET
GET_NUMBERS ENDP

;------------------------------------------------
SHOW_RESULT PROC
    MOV AH, 09H
    LEA DX, RESMSG
    INT 21H
    MOV AX, RESULT
    CALL PRINT_NUM
    RET
SHOW_RESULT ENDP

;------------------------------------------------
READ_NUM PROC
    ; Read decimal number from keyboard
    ; Returns: AX = number
    PUSH BX
    PUSH CX
    PUSH DX
    
    XOR BX, BX              ; Result = 0
    
RN_LOOP:
    MOV AH, 01H
    INT 21H
    
    CMP AL, 0DH             ; Enter?
    JE RN_DONE
    
    SUB AL, '0'             ; Convert ASCII to digit
    XOR AH, AH
    
    ; Result = Result * 10 + digit
    PUSH AX
    MOV AX, BX
    MOV CX, 10
    MUL CX
    MOV BX, AX
    POP AX
    ADD BX, AX
    
    JMP RN_LOOP
    
RN_DONE:
    MOV AX, BX
    
    POP DX
    POP CX
    POP BX
    RET
READ_NUM ENDP

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

| Algorithm/Operation | Key Instructions | Complexity |
|---------------------|------------------|------------|
| Bubble Sort | CMP, XCHG, LOOP | O(n²) |
| Selection Sort | CMP, MOV, LOOP | O(n²) |
| Linear Search | CMP, JE, LOOP | O(n) |
| Binary Search | CMP, SHR, JMP | O(log n) |
| Factorial | MUL, LOOP | O(n) |
| GCD | DIV, JMP | O(log min(a,b)) |
| File Read | INT 21H/3DH, 3FH | - |
| File Write | INT 21H/3CH, 40H | - |

---

## ❓ Quick Revision Questions

1. **Why is bubble sort inefficient for large arrays?**
   <details>
   <summary>Show Answer</summary>
   Bubble sort has O(n²) complexity - it needs approximately n×n comparisons and swaps for n elements. For 1000 elements, that's ~1,000,000 operations. The inner loop must run through the entire array multiple times, making it slow for large datasets compared to O(n log n) algorithms.
   </details>

2. **How does binary search achieve O(log n) efficiency?**
   <details>
   <summary>Show Answer</summary>
   Binary search halves the search space with each comparison. For 1024 elements, it needs at most 10 comparisons (log₂ 1024 = 10). By comparing the middle element and eliminating half the remaining elements, it quickly narrows down to the target.
   </details>

3. **How do you perform 32-bit addition on 16-bit 8086?**
   <details>
   <summary>Show Answer</summary>
   Add in two steps: First ADD the low words (this may set carry flag). Then ADC (Add with Carry) the high words - this includes any carry from the low word addition. Similarly for subtraction: SUB low words, then SBB (Subtract with Borrow) high words.
   </details>

4. **What are file handles in DOS?**
   <details>
   <summary>Show Answer</summary>
   File handles are 16-bit numbers that DOS uses to identify open files. When you open/create a file, DOS returns a handle. You use this handle for all subsequent operations (read, write, close). Standard handles: 0=stdin, 1=stdout, 2=stderr, 3=stdaux, 4=stdprn.
   </details>

5. **How do you check for errors in DOS file operations?**
   <details>
   <summary>Show Answer</summary>
   After INT 21H file functions, check the Carry Flag (CF). If CF=1, an error occurred and AX contains the error code. If CF=0, operation succeeded. Common errors: 2=file not found, 3=path not found, 4=too many files open, 5=access denied.
   </details>

6. **Why use XCHG for swapping in sort algorithms?**
   <details>
   <summary>Show Answer</summary>
   XCHG atomically exchanges two values in one instruction. `XCHG AL, [BX+1]` swaps AL with memory in one operation instead of needing a temporary register and 3 MOV instructions. It's more efficient and makes the code cleaner.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next Unit |
|----------|-----|-----------|
| [5.5 String Operations](05-string-operations.md) | [Unit 5 Index](README.md) | [Unit 6: Memory & I/O Interfacing](../06-Memory-IO-Interfacing/README.md) |

---

*[← Previous: String Operations](05-string-operations.md) | [Next Unit: Memory & I/O Interfacing →](../06-Memory-IO-Interfacing/README.md)*
