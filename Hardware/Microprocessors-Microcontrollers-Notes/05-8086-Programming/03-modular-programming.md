# Chapter 5.3: Modular Programming

## 📚 Chapter Overview

This chapter covers techniques for writing modular, reusable code in 8086 assembly including macros, conditional assembly, and multi-file programming. These techniques help create maintainable and organized programs.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Create and use macros effectively
- Understand macro parameters and local labels
- Use conditional assembly directives
- Create include files and libraries
- Organize multi-module programs

---

## 1. Macros

### 1.1 Macro Basics

```
MACRO CONCEPT:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   MACRO = Text substitution at assembly time                    │
│                                                                  │
│   DIFFERENCES: MACRO vs PROCEDURE                               │
│   ┌───────────────────┬────────────────────────────────────┐   │
│   │      MACRO        │         PROCEDURE                  │   │
│   ├───────────────────┼────────────────────────────────────┤   │
│   │ Expanded inline   │ Called at runtime                  │   │
│   │ No CALL/RET       │ Uses CALL/RET                      │   │
│   │ Faster execution  │ Slower (call overhead)            │   │
│   │ More code space   │ Less code space                   │   │
│   │ Assembler feature │ CPU feature                       │   │
│   │ Parameters by     │ Parameters by registers           │   │
│   │ text substitution │ or stack                          │   │
│   └───────────────────┴────────────────────────────────────┘   │
│                                                                  │
│   SYNTAX:                                                        │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   macro_name MACRO [parameter1, parameter2, ...]       │  │
│   │       ; macro body                                      │  │
│   │       ; instructions using parameters                   │  │
│   │   ENDM                                                  │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Simple Macros

```asm
;------------------------------------------------
; SIMPLE MACRO EXAMPLES
;------------------------------------------------

; Macro without parameters
NEWLINE MACRO
    PUSH AX
    PUSH DX
    MOV AH, 02H
    MOV DL, 0DH         ; Carriage return
    INT 21H
    MOV DL, 0AH         ; Line feed
    INT 21H
    POP DX
    POP AX
ENDM

; Usage:
;   NEWLINE             ; Expands to all the above code

;------------------------------------------------

; Macro with one parameter
PRINT_CHAR MACRO CHAR
    PUSH AX
    PUSH DX
    MOV AH, 02H
    MOV DL, CHAR
    INT 21H
    POP DX
    POP AX
ENDM

; Usage:
;   PRINT_CHAR 'A'      ; Print letter A
;   PRINT_CHAR BL       ; Print character in BL

;------------------------------------------------

; Macro with multiple parameters
ADD3 MACRO DEST, SRC1, SRC2, SRC3
    MOV DEST, SRC1
    ADD DEST, SRC2
    ADD DEST, SRC3
ENDM

; Usage:
;   ADD3 AX, 10, 20, 30 ; AX = 10 + 20 + 30 = 60
```

### 1.3 Macros with Local Labels

```
LOCAL LABELS IN MACROS:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Problem: If macro has labels and is used multiple times,     │
│            duplicate label errors occur                         │
│                                                                  │
│   Solution: Use LOCAL directive                                 │
│                                                                  │
│   EXAMPLE:                                                       │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   ; WRONG - will cause error if used twice             │  │
│   │   WAIT_KEY MACRO                                        │  │
│   │   WAIT_LOOP:                                            │  │
│   │       MOV AH, 01H                                       │  │
│   │       INT 16H                                           │  │
│   │       JZ WAIT_LOOP                                      │  │
│   │   ENDM                                                  │  │
│   │                                                          │  │
│   │   ; CORRECT - unique labels each expansion             │  │
│   │   WAIT_KEY MACRO                                        │  │
│   │       LOCAL WAIT_LOOP                                   │  │
│   │   WAIT_LOOP:                                            │  │
│   │       MOV AH, 01H                                       │  │
│   │       INT 16H                                           │  │
│   │       JZ WAIT_LOOP                                      │  │
│   │   ENDM                                                  │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   Each expansion gets unique label: ??0000, ??0001, etc.        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 1.4 Advanced Macro Examples

```asm
;------------------------------------------------
; PRINT STRING MACRO
;------------------------------------------------
PRINT_STR MACRO STRING
    PUSH AX
    PUSH DX
    MOV AH, 09H
    LEA DX, STRING
    INT 21H
    POP DX
    POP AX
ENDM

;------------------------------------------------
; COMPARISON MACRO
;------------------------------------------------
COMPARE MACRO VAL1, VAL2, JUMP_IF_EQUAL
    LOCAL SKIP
    PUSH AX
    MOV AX, VAL1
    CMP AX, VAL2
    POP AX
    JNE SKIP
    JMP JUMP_IF_EQUAL
SKIP:
ENDM

;------------------------------------------------
; LOOP MACRO
;------------------------------------------------
REPEAT MACRO COUNT, OPERATION
    LOCAL RLOOP
    PUSH CX
    MOV CX, COUNT
RLOOP:
    OPERATION
    LOOP RLOOP
    POP CX
ENDM

; Usage: REPEAT 5, <PRINT_CHAR '*'>  ; Print 5 stars

;------------------------------------------------
; PUSH/POP MULTIPLE REGISTERS
;------------------------------------------------
PUSHALL MACRO
    PUSH AX
    PUSH BX
    PUSH CX
    PUSH DX
    PUSH SI
    PUSH DI
ENDM

POPALL MACRO
    POP DI
    POP SI
    POP DX
    POP CX
    POP BX
    POP AX
ENDM

;------------------------------------------------
; EXIT TO DOS
;------------------------------------------------
DOS_EXIT MACRO RETCODE
    MOV AH, 4CH
    MOV AL, RETCODE
    INT 21H
ENDM

; Usage: DOS_EXIT 0     ; Exit with code 0
```

---

## 2. Conditional Assembly

### 2.1 Conditional Directives

```
CONDITIONAL ASSEMBLY DIRECTIVES:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   IF - ELSE - ENDIF:                                            │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   DEBUG = 1                                             │  │
│   │                                                          │  │
│   │   IF DEBUG                                              │  │
│   │       ; Debug code - only assembled if DEBUG != 0       │  │
│   │       CALL PRINT_DEBUG_INFO                             │  │
│   │   ELSE                                                  │  │
│   │       ; Release code                                    │  │
│   │       NOP                                               │  │
│   │   ENDIF                                                 │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   IFE (If Equal to zero):                                       │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   IFE OPTIMIZE                                          │  │
│   │       ; Assembled if OPTIMIZE = 0                       │  │
│   │   ENDIF                                                 │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   IFDEF / IFNDEF (If Defined / Not Defined):                    │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   IFDEF FEATURE_X                                       │  │
│   │       ; Assembled if FEATURE_X is defined               │  │
│   │   ENDIF                                                 │  │
│   │                                                          │  │
│   │   IFNDEF FEATURE_Y                                      │  │
│   │       ; Assembled if FEATURE_Y is NOT defined           │  │
│   │   ENDIF                                                 │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   IFB / IFNB (If Blank / Not Blank) - for macro params:         │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   MYMACRO MACRO PARAM                                   │  │
│   │       IFB <PARAM>                                       │  │
│   │           MOV AX, 0       ; Default if no param        │  │
│   │       ELSE                                              │  │
│   │           MOV AX, PARAM                                 │  │
│   │       ENDIF                                             │  │
│   │   ENDM                                                  │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Conditional Macro Example

```asm
;------------------------------------------------
; FLEXIBLE INPUT MACRO
;------------------------------------------------
GET_INPUT MACRO TYPE
    IFB <TYPE>
        ; Default: single character
        MOV AH, 01H
        INT 21H
    ELSE
        IFIDN <TYPE>, <CHAR>
            MOV AH, 01H
            INT 21H
        ENDIF
        IFIDN <TYPE>, <STRING>
            MOV AH, 0AH
            INT 21H
        ENDIF
        IFIDN <TYPE>, <NOECHO>
            MOV AH, 08H
            INT 21H
        ENDIF
    ENDIF
ENDM

; Usage:
;   GET_INPUT              ; Read char with echo
;   GET_INPUT CHAR         ; Read char with echo
;   GET_INPUT NOECHO       ; Read char without echo
;   GET_INPUT STRING       ; Read string

;------------------------------------------------
; PLATFORM-SPECIFIC CODE
;------------------------------------------------
PLATFORM = 1        ; 1 = DOS, 2 = Windows

EXIT_PROGRAM MACRO
    IF PLATFORM EQ 1
        MOV AH, 4CH
        INT 21H
    ELSEIF PLATFORM EQ 2
        ; Windows exit code would go here
        MOV AH, 4CH
        INT 21H
    ENDIF
ENDM
```

---

## 3. Include Files

### 3.1 Creating Include Files

```
INCLUDE FILE ORGANIZATION:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   INCLUDE FILE (macros.inc):                                    │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   ; MACROS.INC - Common macros library                  │  │
│   │   ; Author: [Your name]                                 │  │
│   │   ; Date: [Date]                                        │  │
│   │                                                          │  │
│   │   ; Constants                                           │  │
│   │   CR      EQU  0DH                                      │  │
│   │   LF      EQU  0AH                                      │  │
│   │   DOS     EQU  21H                                      │  │
│   │                                                          │  │
│   │   ; Macros                                              │  │
│   │   NEWLINE MACRO                                         │  │
│   │       MOV AH, 02H                                       │  │
│   │       MOV DL, CR                                        │  │
│   │       INT DOS                                           │  │
│   │       MOV DL, LF                                        │  │
│   │       INT DOS                                           │  │
│   │   ENDM                                                  │  │
│   │                                                          │  │
│   │   PRINT_STR MACRO MSG                                   │  │
│   │       MOV AH, 09H                                       │  │
│   │       LEA DX, MSG                                       │  │
│   │       INT DOS                                           │  │
│   │   ENDM                                                  │  │
│   │                                                          │  │
│   │   DOS_EXIT MACRO CODE                                   │  │
│   │       MOV AH, 4CH                                       │  │
│   │       MOV AL, CODE                                      │  │
│   │       INT DOS                                           │  │
│   │   ENDM                                                  │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   MAIN PROGRAM (main.asm):                                      │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   INCLUDE macros.inc                                    │  │
│   │                                                          │  │
│   │   .MODEL SMALL                                          │  │
│   │   .STACK 100H                                           │  │
│   │                                                          │  │
│   │   .DATA                                                 │  │
│   │   MSG DB 'Hello World!$'                                │  │
│   │                                                          │  │
│   │   .CODE                                                 │  │
│   │   MAIN PROC                                             │  │
│   │       MOV AX, @DATA                                     │  │
│   │       MOV DS, AX                                        │  │
│   │                                                          │  │
│   │       PRINT_STR MSG    ; Use macro from include        │  │
│   │       NEWLINE          ; Use macro from include        │  │
│   │       DOS_EXIT 0       ; Use macro from include        │  │
│   │   MAIN ENDP                                             │  │
│   │   END MAIN                                              │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 Standard Include Library

```asm
;================================================
; STDIO.INC - Standard I/O Macros
;================================================

; Prevent multiple inclusion
IFNDEF STDIO_INC
STDIO_INC = 1

;------------------------------------------------
; Constants
;------------------------------------------------
CR      EQU     0DH
LF      EQU     0AH
SPACE   EQU     20H
DOS     EQU     21H

;------------------------------------------------
; Print single character
;------------------------------------------------
PUTCHAR MACRO CHAR
    PUSH AX
    PUSH DX
    MOV AH, 02H
    MOV DL, CHAR
    INT DOS
    POP DX
    POP AX
ENDM

;------------------------------------------------
; Print string (must end with $)
;------------------------------------------------
PUTS MACRO STRING_ADDR
    PUSH AX
    PUSH DX
    MOV AH, 09H
    LEA DX, STRING_ADDR
    INT DOS
    POP DX
    POP AX
ENDM

;------------------------------------------------
; Print newline
;------------------------------------------------
PUTLN MACRO
    PUTCHAR CR
    PUTCHAR LF
ENDM

;------------------------------------------------
; Get character (with echo)
;------------------------------------------------
GETCHAR MACRO
    MOV AH, 01H
    INT DOS
    ; Character returned in AL
ENDM

;------------------------------------------------
; Get character (without echo)
;------------------------------------------------
GETCH MACRO
    MOV AH, 08H
    INT DOS
    ; Character returned in AL
ENDM

;------------------------------------------------
; Exit to DOS
;------------------------------------------------
EXIT MACRO CODE
    IFB <CODE>
        MOV AL, 0
    ELSE
        MOV AL, CODE
    ENDIF
    MOV AH, 4CH
    INT DOS
ENDM

ENDIF   ; STDIO_INC
```

---

## 4. Multi-Module Programming

### 4.1 PUBLIC and EXTRN Directives

```
MULTI-MODULE STRUCTURE:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   MODULE 1 (math.asm) - Provides procedures                    │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   PUBLIC ADD_NUMS, MULTIPLY     ; Export these          │  │
│   │   PUBLIC MAX_VALUE              ; Export data too        │  │
│   │                                                          │  │
│   │   .MODEL SMALL                                          │  │
│   │                                                          │  │
│   │   .DATA                                                 │  │
│   │   MAX_VALUE DW 0FFFFH                                   │  │
│   │                                                          │  │
│   │   .CODE                                                 │  │
│   │   ADD_NUMS PROC                                         │  │
│   │       ADD AX, BX                                        │  │
│   │       RET                                               │  │
│   │   ADD_NUMS ENDP                                         │  │
│   │                                                          │  │
│   │   MULTIPLY PROC                                         │  │
│   │       MUL BX                                            │  │
│   │       RET                                               │  │
│   │   MULTIPLY ENDP                                         │  │
│   │                                                          │  │
│   │   END                                                   │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   MODULE 2 (main.asm) - Uses procedures                        │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   EXTRN ADD_NUMS:PROC           ; Import from math     │  │
│   │   EXTRN MULTIPLY:PROC                                   │  │
│   │   EXTRN MAX_VALUE:WORD          ; Import data          │  │
│   │                                                          │  │
│   │   .MODEL SMALL                                          │  │
│   │   .STACK 100H                                           │  │
│   │                                                          │  │
│   │   .CODE                                                 │  │
│   │   MAIN PROC                                             │  │
│   │       MOV AX, 10                                        │  │
│   │       MOV BX, 20                                        │  │
│   │       CALL ADD_NUMS     ; Call external procedure      │  │
│   │       ; AX = 30                                         │  │
│   │                                                          │  │
│   │       MOV AX, 4CH                                       │  │
│   │       INT 21H                                           │  │
│   │   MAIN ENDP                                             │  │
│   │   END MAIN                                              │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   BUILD PROCESS:                                                 │
│   1. Assemble each module: MASM math.asm   → math.obj          │
│                            MASM main.asm   → main.obj          │
│   2. Link together:        LINK main.obj+math.obj → main.exe   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Complete Multi-Module Example

```asm
;================================================
; FILE: io.asm - Input/Output procedures
;================================================
PUBLIC PRINT_CHAR, PRINT_STRING, READ_CHAR
PUBLIC PRINT_NUMBER

.MODEL SMALL

.CODE

;------------------------------------------------
PRINT_CHAR PROC
    ; Input: DL = character to print
    PUSH AX
    MOV AH, 02H
    INT 21H
    POP AX
    RET
PRINT_CHAR ENDP

;------------------------------------------------
PRINT_STRING PROC
    ; Input: DX = address of $-terminated string
    PUSH AX
    MOV AH, 09H
    INT 21H
    POP AX
    RET
PRINT_STRING ENDP

;------------------------------------------------
READ_CHAR PROC
    ; Output: AL = character read
    MOV AH, 01H
    INT 21H
    RET
READ_CHAR ENDP

;------------------------------------------------
PRINT_NUMBER PROC
    ; Input: AX = unsigned number to print
    PUSH AX
    PUSH BX
    PUSH CX
    PUSH DX
    
    MOV BX, 10
    XOR CX, CX
    
DIV_LOOP:
    XOR DX, DX
    DIV BX
    PUSH DX
    INC CX
    OR AX, AX
    JNZ DIV_LOOP
    
PRINT_LOOP:
    POP DX
    ADD DL, '0'
    MOV AH, 02H
    INT 21H
    LOOP PRINT_LOOP
    
    POP DX
    POP CX
    POP BX
    POP AX
    RET
PRINT_NUMBER ENDP

END
```

```asm
;================================================
; FILE: math.asm - Math procedures
;================================================
PUBLIC ADD_NUMS, SUB_NUMS, MUL_NUMS, DIV_NUMS

.MODEL SMALL

.CODE

;------------------------------------------------
ADD_NUMS PROC
    ; Input: AX, BX = numbers
    ; Output: AX = AX + BX
    ADD AX, BX
    RET
ADD_NUMS ENDP

;------------------------------------------------
SUB_NUMS PROC
    ; Input: AX, BX = numbers
    ; Output: AX = AX - BX
    SUB AX, BX
    RET
SUB_NUMS ENDP

;------------------------------------------------
MUL_NUMS PROC
    ; Input: AX, BX = numbers
    ; Output: DX:AX = AX * BX
    MUL BX
    RET
MUL_NUMS ENDP

;------------------------------------------------
DIV_NUMS PROC
    ; Input: DX:AX = dividend, BX = divisor
    ; Output: AX = quotient, DX = remainder
    DIV BX
    RET
DIV_NUMS ENDP

END
```

```asm
;================================================
; FILE: main.asm - Main program
;================================================
EXTRN PRINT_CHAR:PROC
EXTRN PRINT_STRING:PROC
EXTRN PRINT_NUMBER:PROC
EXTRN ADD_NUMS:PROC

.MODEL SMALL
.STACK 100H

.DATA
MSG1    DB  'Result: $'
NEWLN   DB  0DH, 0AH, '$'

.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX
    
    ; Add two numbers
    MOV AX, 100
    MOV BX, 250
    CALL ADD_NUMS       ; AX = 350
    PUSH AX
    
    ; Print message
    LEA DX, MSG1
    CALL PRINT_STRING
    
    ; Print result
    POP AX
    CALL PRINT_NUMBER
    
    ; Newline
    LEA DX, NEWLN
    CALL PRINT_STRING
    
    ; Exit
    MOV AH, 4CH
    INT 21H
    
MAIN ENDP
END MAIN
```

---

## 5. Repeat Directives

### 5.1 REPT, IRP, IRPC

```
REPEAT ASSEMBLY DIRECTIVES:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   REPT - Repeat block n times:                                  │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   REPT 5                                                │  │
│   │       DB 0FFH        ; Creates 5 bytes of 0FFH          │  │
│   │   ENDM                                                  │  │
│   │                                                          │  │
│   │   ; Counter variable                                    │  │
│   │   COUNT = 0                                             │  │
│   │   REPT 10                                               │  │
│   │       DB COUNT                                          │  │
│   │       COUNT = COUNT + 1                                 │  │
│   │   ENDM                                                  │  │
│   │   ; Creates: 00, 01, 02, 03, 04, 05, 06, 07, 08, 09    │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   IRP - Repeat for each argument:                               │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   IRP REG, <AX, BX, CX, DX>                             │  │
│   │       PUSH REG                                          │  │
│   │   ENDM                                                  │  │
│   │   ; Expands to: PUSH AX / PUSH BX / PUSH CX / PUSH DX  │  │
│   │                                                          │  │
│   │   IRP VAL, <1, 2, 4, 8, 16>                             │  │
│   │       DW VAL                                            │  │
│   │   ENDM                                                  │  │
│   │   ; Creates: DW 1, DW 2, DW 4, DW 8, DW 16             │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│   IRPC - Repeat for each character:                             │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │   IRPC CHAR, ABCDE                                      │  │
│   │       DB '&CHAR'                                        │  │
│   │   ENDM                                                  │  │
│   │   ; Creates: DB 'A', DB 'B', DB 'C', DB 'D', DB 'E'    │  │
│   │                                                          │  │
│   │   IRPC DIGIT, 0123456789                                │  │
│   │       DB '&DIGIT'                                       │  │
│   │   ENDM                                                  │  │
│   │   ; Creates ASCII digits 0-9                           │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Practical Macro Library

```asm
;================================================
; UTILITY.INC - Complete Utility Library
;================================================

IFNDEF UTILITY_INC
UTILITY_INC = 1

;------------------------------------------------
; Constants
;------------------------------------------------
DOS         EQU     21H
VIDEO       EQU     10H

;------------------------------------------------
; SAVE_REGS - Save all general registers
;------------------------------------------------
SAVE_REGS MACRO
    PUSH AX
    PUSH BX
    PUSH CX
    PUSH DX
    PUSH SI
    PUSH DI
    PUSHF
ENDM

;------------------------------------------------
; REST_REGS - Restore all general registers
;------------------------------------------------
REST_REGS MACRO
    POPF
    POP DI
    POP SI
    POP DX
    POP CX
    POP BX
    POP AX
ENDM

;------------------------------------------------
; SET_DS - Initialize data segment
;------------------------------------------------
SET_DS MACRO
    MOV AX, @DATA
    MOV DS, AX
ENDM

;------------------------------------------------
; CLEAR_SCREEN - Clear video display
;------------------------------------------------
CLEAR_SCREEN MACRO
    SAVE_REGS
    MOV AX, 0600H       ; Scroll up, clear screen
    MOV BH, 07H         ; Normal attribute
    MOV CX, 0000H       ; Upper left
    MOV DX, 184FH       ; Lower right (24,79)
    INT VIDEO
    REST_REGS
ENDM

;------------------------------------------------
; SET_CURSOR - Set cursor position
;------------------------------------------------
SET_CURSOR MACRO ROW, COL
    SAVE_REGS
    MOV AH, 02H
    MOV BH, 00H
    MOV DH, ROW
    MOV DL, COL
    INT VIDEO
    REST_REGS
ENDM

;------------------------------------------------
; PRINT_AT - Print string at position
;------------------------------------------------
PRINT_AT MACRO ROW, COL, STRING
    SET_CURSOR ROW, COL
    PUTS STRING
ENDM

;------------------------------------------------
; DELAY - Simple delay loop
;------------------------------------------------
DELAY MACRO COUNT
    LOCAL OUTER, INNER
    PUSH CX
    PUSH DX
    MOV CX, COUNT
OUTER:
    MOV DX, 0FFFFH
INNER:
    DEC DX
    JNZ INNER
    LOOP OUTER
    POP DX
    POP CX
ENDM

;------------------------------------------------
; BEEP - Sound the speaker
;------------------------------------------------
BEEP MACRO
    PUTCHAR 07H
ENDM

ENDIF ; UTILITY_INC
```

---

## 📋 Summary Table

| Directive | Purpose | Example |
|-----------|---------|---------|
| MACRO...ENDM | Define macro | `NAME MACRO...ENDM` |
| LOCAL | Local labels in macro | `LOCAL LOOP1` |
| INCLUDE | Include file | `INCLUDE myfile.inc` |
| PUBLIC | Export symbol | `PUBLIC MYPROC` |
| EXTRN | Import symbol | `EXTRN MYPROC:PROC` |
| IF...ENDIF | Conditional assembly | `IF DEBUG...ENDIF` |
| IFDEF | If defined | `IFDEF SYMBOL` |
| IFNDEF | If not defined | `IFNDEF SYMBOL` |
| REPT | Repeat n times | `REPT 10...ENDM` |
| IRP | Repeat for each | `IRP X,<1,2,3>` |

---

## ❓ Quick Revision Questions

1. **What is the difference between a macro and a procedure?**
   <details>
   <summary>Show Answer</summary>
   Macro: Code is expanded inline at each use (text substitution at assembly time), no CALL/RET overhead, faster but uses more memory. Procedure: Single copy called at runtime with CALL/RET, slower due to call overhead but saves memory. Use macros for small, frequently used code; procedures for larger code.
   </details>

2. **Why is the LOCAL directive needed in macros?**
   <details>
   <summary>Show Answer</summary>
   Without LOCAL, if a macro with labels is used multiple times, the assembler sees duplicate label definitions (error). LOCAL tells the assembler to generate unique label names (like ??0000, ??0001) for each macro expansion, preventing conflicts.
   </details>

3. **What does INCLUDE do?**
   <details>
   <summary>Show Answer</summary>
   INCLUDE inserts the contents of another file at that point in the source code. It's like copy-pasting the file content. Used for sharing macros, constants, and common code across multiple source files. Example: `INCLUDE macros.inc` includes all definitions from macros.inc.
   </details>

4. **What is the difference between PUBLIC and EXTRN?**
   <details>
   <summary>Show Answer</summary>
   PUBLIC: Declares symbols (procedures, variables) that this module exports for other modules to use. EXTRN: Declares symbols that this module imports from other modules. PUBLIC is used in the defining module; EXTRN in the using module. They work as a pair for multi-module programming.
   </details>

5. **How do you prevent an include file from being included twice?**
   <details>
   <summary>Show Answer</summary>
   Use include guard pattern: `IFNDEF MYFILE_INC` / `MYFILE_INC = 1` / `...content...` / `ENDIF`. First inclusion defines MYFILE_INC, subsequent inclusions skip the content because MYFILE_INC is already defined. Prevents duplicate definition errors.
   </details>

6. **When should you use conditional assembly?**
   <details>
   <summary>Show Answer</summary>
   For: (1) Debug vs release builds - include debug code only when DEBUG=1, (2) Platform-specific code - different code for DOS/Windows, (3) Feature flags - enable/disable features, (4) Macro defaults - handle missing parameters. Allows single source to generate different executables.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [5.2 Stack Operations & Procedures](02-stack-operations-procedures.md) | [Unit 5 Index](README.md) | [5.4 Interrupt Programming](04-interrupt-programming.md) |

---

*[← Previous: Stack Operations & Procedures](02-stack-operations-procedures.md) | [Next: Interrupt Programming →](04-interrupt-programming.md)*
