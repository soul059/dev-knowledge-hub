# Unit 5: 8086 Programming

## 📚 Unit Overview

This unit focuses on assembly language programming for the 8086 microprocessor. You will learn about stack operations, procedures, macros, interrupt programming, and develop practical programs for various applications.

---

## 📑 Table of Contents

| Chapter | Topic | Description |
|---------|-------|-------------|
| [5.1](01-assembly-language-basics.md) | Assembly Language Basics | Assembler directives, program structure, data definition |
| [5.2](02-stack-operations-procedures.md) | Stack Operations & Procedures | PUSH, POP, CALL, RET, parameter passing |
| [5.3](03-modular-programming.md) | Modular Programming | Macros, procedures, libraries, code reuse |
| [5.4](04-interrupt-programming.md) | Interrupt Programming | DOS/BIOS interrupts, ISR writing, INT 21H services |
| [5.5](05-string-operations.md) | String Operations | String instructions, REP prefix, text processing |
| [5.6](06-advanced-programs.md) | Advanced Programs | Sorting, searching, arithmetic, file handling |

---

## 🎯 Learning Outcomes

After completing this unit, you will be able to:

1. Write assembly language programs using proper syntax
2. Use assembler directives effectively
3. Implement procedures with parameter passing
4. Create and use macros for code reuse
5. Write interrupt service routines
6. Develop programs for string manipulation
7. Create complete applications using 8086 assembly

---

## 🔧 Development Environment

```
8086 ASSEMBLY DEVELOPMENT WORKFLOW:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   1. EDIT                    2. ASSEMBLE                        │
│   ┌──────────────┐          ┌──────────────┐                   │
│   │  Source.ASM  │  ──────► │    MASM      │                   │
│   │  (Text file) │          │   or TASM    │                   │
│   └──────────────┘          └──────┬───────┘                   │
│                                     │                           │
│                                     ▼                           │
│   4. DEBUG                  3. LINK                             │
│   ┌──────────────┐          ┌──────────────┐                   │
│   │   DEBUG      │  ◄────── │   LINK.EXE   │                   │
│   │  or AFD      │          │              │                   │
│   └──────────────┘          └──────┬───────┘                   │
│                                     │                           │
│                                     ▼                           │
│                              ┌──────────────┐                   │
│                              │ Program.EXE  │                   │
│                              │ (Executable) │                   │
│                              └──────────────┘                   │
│                                                                  │
│   TOOLS:                                                        │
│   • MASM (Microsoft Assembler) / TASM (Turbo Assembler)        │
│   • LINK (Linker)                                               │
│   • DEBUG / AFD (Debugger)                                      │
│   • EMU8086 (Emulator - for practice)                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Program Types

```
TYPES OF 8086 PROGRAMS:

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   COM FILES:                                                    │
│   • Single segment (≤ 64KB)                                     │
│   • CS = DS = SS = ES                                          │
│   • Origin at 100H (after PSP)                                 │
│   • Smaller, simpler                                           │
│   • No separate data segment                                   │
│                                                                  │
│   EXE FILES:                                                    │
│   • Multiple segments                                          │
│   • Can be larger than 64KB                                    │
│   • Separate code, data, stack segments                        │
│   • More flexible                                              │
│   • Standard for complex programs                              │
│                                                                  │
│   PROGRAM STRUCTURE (EXE):                                      │
│   ┌──────────────────────────────────────┐                     │
│   │  .MODEL SMALL                        │ Memory model        │
│   │  .STACK 100H                         │ Stack segment       │
│   │  .DATA                               │ Data segment        │
│   │      variables and constants         │                     │
│   │  .CODE                               │ Code segment        │
│   │  MAIN PROC                           │                     │
│   │      instructions                    │                     │
│   │  MAIN ENDP                           │                     │
│   │  END MAIN                            │                     │
│   └──────────────────────────────────────┘                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 💡 Key Concepts Preview

### Assembler Directives

| Directive | Purpose | Example |
|-----------|---------|---------|
| DB | Define Byte | `MSG DB 'Hello$'` |
| DW | Define Word | `NUM DW 1234H` |
| DD | Define Doubleword | `ADDR DD 12345678H` |
| EQU | Equate (constant) | `CR EQU 0DH` |
| ORG | Origin | `ORG 100H` |
| PROC | Procedure start | `MAIN PROC` |
| ENDP | Procedure end | `MAIN ENDP` |
| MACRO | Macro start | `DISPLAY MACRO` |
| ENDM | Macro end | `ENDM` |

### Important Interrupts

| Interrupt | Function |
|-----------|----------|
| INT 21H | DOS services |
| INT 10H | Video BIOS |
| INT 16H | Keyboard BIOS |
| INT 13H | Disk BIOS |
| INT 1AH | Time BIOS |

---

## ⚡ Quick Example

```asm
; HELLO WORLD PROGRAM (EXE format)
.MODEL SMALL
.STACK 100H

.DATA
    MSG DB 'Hello, World!$'

.CODE
MAIN PROC
    ; Initialize data segment
    MOV AX, @DATA
    MOV DS, AX
    
    ; Display message
    MOV AH, 09H        ; DOS print string
    LEA DX, MSG        ; Load address of message
    INT 21H            ; Call DOS
    
    ; Exit program
    MOV AH, 4CH        ; DOS terminate
    MOV AL, 00H        ; Return code
    INT 21H            ; Call DOS
MAIN ENDP
END MAIN
```

---

## 🧭 Navigation

| Previous Unit | Course Home | Next Unit |
|---------------|-------------|-----------|
| [Unit 4: 8086 Microprocessor](../04-8086-Microprocessor/README.md) | [Course Index](../README.md) | [Unit 6: Memory & I/O Interfacing](../06-Memory-IO-Interfacing/README.md) |

---

*[← Previous Unit: 8086 Microprocessor](../04-8086-Microprocessor/README.md) | [Next Unit: Memory & I/O Interfacing →](../06-Memory-IO-Interfacing/README.md)*
