# Chapter 2.4: Addressing Modes

## 📚 Chapter Overview

Addressing modes define how the operand of an instruction is specified. The 8085 supports five primary addressing modes, each serving different programming needs. Understanding these modes is crucial for efficient assembly language programming.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Identify and explain all 8085 addressing modes
- Select appropriate addressing mode for different operations
- Calculate effective addresses for each mode
- Write assembly code using different addressing modes

---

## 1. Introduction to Addressing Modes

### 1.1 What are Addressing Modes?

```
ADDRESSING MODE CONCEPT:

┌─────────────────────────────────────────────────────────────────┐
│                    INSTRUCTION STRUCTURE                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│   ┌────────────────┐  ┌──────────────────────────────────────┐  │
│   │    OPCODE      │  │           OPERAND                    │  │
│   │   (What to do) │  │        (On what data)                │  │
│   └────────────────┘  └──────────────────────────────────────┘  │
│                                 │                                │
│                                 ▼                                │
│              ┌────────────────────────────────────┐             │
│              │     How to find the operand?       │             │
│              │          ADDRESSING MODE           │             │
│              └────────────────────────────────────┘             │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 8085 Addressing Modes Overview

```
8085 ADDRESSING MODES:

┌────────────────────────────────────────────────────────────────────┐
│                                                                     │
│   1. IMMEDIATE ADDRESSING                                          │
│      └── Operand is in instruction itself                          │
│                                                                     │
│   2. REGISTER ADDRESSING                                           │
│      └── Operand is in a CPU register                              │
│                                                                     │
│   3. DIRECT (ABSOLUTE) ADDRESSING                                  │
│      └── Memory address is in instruction                          │
│                                                                     │
│   4. INDIRECT ADDRESSING                                           │
│      └── Register pair contains memory address                     │
│                                                                     │
│   5. IMPLIED (IMPLICIT) ADDRESSING                                 │
│      └── Operand is implicit (fixed by opcode)                     │
│                                                                     │
└────────────────────────────────────────────────────────────────────┘
```

---

## 2. Immediate Addressing Mode

### 2.1 Concept

```
IMMEDIATE ADDRESSING:

The operand value is part of the instruction itself.
─────────────────────────────────────────────────────────────────

Instruction Format:
┌─────────┬─────────┐         ┌─────────┬─────────┬─────────┐
│ Opcode  │  Data   │    OR   │ Opcode  │ Data-Lo │ Data-Hi │
└─────────┴─────────┘         └─────────┴─────────┴─────────┘
   2-Byte                              3-Byte


Example: MVI A, 45H
─────────────────────────────────────────────────────────────────

Memory:                          CPU:
┌─────────┬─────────┐           ┌───────────────┐
│  2000   │   3E    │  Opcode   │               │
├─────────┼─────────┤           │  Accumulator  │
│  2001   │   45    │──────────►│     45H       │
└─────────┴─────────┘   Data    └───────────────┘

The value 45H is stored immediately after opcode.
No memory access needed to get operand.
```

### 2.2 Immediate Addressing Instructions

| Instruction | Bytes | Operation | Example |
|-------------|-------|-----------|---------|
| MVI r, data | 2 | r ← data | MVI A, 32H |
| MVI M, data | 2 | M[HL] ← data | MVI M, 00H |
| ADI data | 2 | A ← A + data | ADI 05H |
| ACI data | 2 | A ← A + data + CY | ACI 10H |
| SUI data | 2 | A ← A - data | SUI 02H |
| SBI data | 2 | A ← A - data - CY | SBI 01H |
| ANI data | 2 | A ← A AND data | ANI 0FH |
| ORI data | 2 | A ← A OR data | ORI 80H |
| XRI data | 2 | A ← A XOR data | XRI FFH |
| CPI data | 2 | Compare A with data | CPI 30H |
| LXI rp, data16 | 3 | rp ← data16 | LXI H, 2050H |

### 2.3 Memory Representation

```
LXI H, 2050H (Opcode: 21H):

Memory Layout:
┌─────────┬─────────────┐
│ Address │   Content   │
├─────────┼─────────────┤
│  2000   │     21      │ ← Opcode (LXI H)
│  2001   │     50      │ ← Low byte of 2050H
│  2002   │     20      │ ← High byte of 2050H
└─────────┴─────────────┘

After Execution:
H = 20H, L = 50H
HL = 2050H

NOTE: 8085 uses LITTLE ENDIAN format
      (Low byte first, then High byte)
```

---

## 3. Register Addressing Mode

### 3.1 Concept

```
REGISTER ADDRESSING:

The operand is in one of the CPU registers.
─────────────────────────────────────────────────────────────────

Instruction Format:
┌─────────────────┐
│     Opcode      │  (Register encoded in opcode)
│   01 DDD SSS    │
└─────────────────┘
    1-Byte


Example: MOV A, B
─────────────────────────────────────────────────────────────────

CPU Registers:
┌───────────────┐       ┌───────────────┐
│       B       │──────►│       A       │
│      45H      │       │      45H      │
└───────────────┘       └───────────────┘
    Source              Destination

Opcode: 78H = 0111 1000
              │││  └─┴─┴── SSS = 000 (B)
              ││└── DDD = 111 (A)
              └┴─── 01 (MOV)

Fastest addressing mode - no memory access!
```

### 3.2 Register Addressing Instructions

| Instruction | Opcode Pattern | Operation | Example |
|-------------|----------------|-----------|---------|
| MOV r1, r2 | 01 DDD SSS | r1 ← r2 | MOV A, B |
| ADD r | 10 000 SSS | A ← A + r | ADD C |
| ADC r | 10 001 SSS | A ← A + r + CY | ADC D |
| SUB r | 10 010 SSS | A ← A - r | SUB E |
| SBB r | 10 011 SSS | A ← A - r - CY | SBB H |
| ANA r | 10 100 SSS | A ← A AND r | ANA L |
| ORA r | 10 110 SSS | A ← A OR r | ORA B |
| XRA r | 10 101 SSS | A ← A XOR r | XRA A |
| CMP r | 10 111 SSS | Compare A with r | CMP B |
| INR r | 00 DDD 100 | r ← r + 1 | INR A |
| DCR r | 00 DDD 101 | r ← r - 1 | DCR B |

### 3.3 Register Codes

```
REGISTER ENCODING IN OPCODES:

┌──────────┬──────┐
│ Register │ Code │
├──────────┼──────┤
│    B     │ 000  │
│    C     │ 001  │
│    D     │ 010  │
│    E     │ 011  │
│    H     │ 100  │
│    L     │ 101  │
│    M     │ 110  │  ← Memory reference (not a register)
│    A     │ 111  │
└──────────┴──────┘

EXAMPLE - Decoding MOV D, E (Opcode: 53H):

53H = 0101 0011
      01 010 011
      │   │   │
      │   │   └── SSS = 011 = E (Source)
      │   └────── DDD = 010 = D (Destination)
      └────────── 01 = MOV instruction

Result: D ← E
```

---

## 4. Direct (Absolute) Addressing Mode

### 4.1 Concept

```
DIRECT ADDRESSING:

The instruction contains the complete memory address of operand.
─────────────────────────────────────────────────────────────────

Instruction Format:
┌─────────┬──────────┬──────────┐
│ Opcode  │ Addr-Low │ Addr-Hi  │
└─────────┴──────────┴──────────┘
        3-Byte Instruction


Example: LDA 2050H
─────────────────────────────────────────────────────────────────

Memory:                          
┌─────────┬─────────┐            
│  2000   │   3A    │  Opcode   
├─────────┼─────────┤            
│  2001   │   50    │  Low address
├─────────┼─────────┤            
│  2002   │   20    │  High address
└─────────┴─────────┘            
        ...                      
┌─────────┬─────────┐    ┌───────────────┐
│  2050   │   7F    │───►│  Accumulator  │
└─────────┴─────────┘    │      7F       │
                         └───────────────┘

Effective Address = 2050H (from instruction)
Data at 2050H loaded into A
```

### 4.2 Direct Addressing Instructions

| Instruction | Opcode | Bytes | Operation |
|-------------|--------|-------|-----------|
| LDA addr | 3AH | 3 | A ← M[addr] |
| STA addr | 32H | 3 | M[addr] ← A |
| LHLD addr | 2AH | 3 | L ← M[addr], H ← M[addr+1] |
| SHLD addr | 22H | 3 | M[addr] ← L, M[addr+1] ← H |
| JMP addr | C3H | 3 | PC ← addr |
| CALL addr | CDH | 3 | Push PC, PC ← addr |
| Jcond addr | varies | 3 | If condition, PC ← addr |

### 4.3 Memory Diagram

```
LHLD 3000H:

Step-by-step:
┌─────────────────────────────────────────────────────────────┐
│                                                              │
│   Program Memory:              Data Memory:                 │
│   ┌─────────┬───────┐         ┌─────────┬───────┐          │
│   │  2000   │  2A   │ Opcode  │  3000   │  34   │──────┐   │
│   │  2001   │  00   │ Lo addr │  3001   │  12   │────┐ │   │
│   │  2002   │  30   │ Hi addr └─────────┴───────┘    │ │   │
│   └─────────┴───────┘                                 │ │   │
│                                                       │ │   │
│   Registers After Execution:                          │ │   │
│   ┌───────────────┐                                   │ │   │
│   │  H   │   L    │                                   │ │   │
│   │ 12   │  34    │◄─────────────────────────────────┘ │   │
│   └──────┴────────┘                                     │   │
│       ▲                                                 │   │
│       └─────────────────────────────────────────────────┘   │
│                                                              │
│   Result: HL = 1234H                                        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 5. Indirect Addressing Mode

### 5.1 Concept

```
INDIRECT ADDRESSING:

A register pair holds the memory address of the operand.
─────────────────────────────────────────────────────────────────

The register pair acts as a POINTER to memory.

Example: MOV A, M (where HL points to memory)
─────────────────────────────────────────────────────────────────

  Register Pair HL:                Memory:
  ┌───────────────────┐           ┌─────────┬─────────┐
  │   H    │    L     │           │  2050   │   45    │
  │   20   │   50     │──────────►│         │         │
  └────────┴──────────┘           └─────────┴─────────┘
        │                                │
        │      Address = 2050H           │
        │                                ▼
        │                         ┌───────────────┐
        │                         │  Accumulator  │
        └────────────────────────►│      45H      │
                                  └───────────────┘

Effective Address = Contents of HL = 2050H
Data at 2050H (45H) is loaded into A
```

### 5.2 Register Indirect Instructions

```
REGISTER INDIRECT INSTRUCTIONS:

Using HL as pointer:
─────────────────────────────────────────
MOV r, M    r ← M[HL]       Load from memory
MOV M, r    M[HL] ← r       Store to memory
ADD M       A ← A + M[HL]   Add memory to A
SUB M       A ← A - M[HL]   Subtract
ANA M       A ← A AND M[HL] Logical AND
ORA M       A ← A OR M[HL]  Logical OR
XRA M       A ← A XOR M[HL] Logical XOR
CMP M       Compare A with M[HL]
INR M       M[HL] ← M[HL] + 1
DCR M       M[HL] ← M[HL] - 1


Using BC or DE as pointer:
─────────────────────────────────────────
LDAX B      A ← M[BC]       Load using BC
LDAX D      A ← M[DE]       Load using DE
STAX B      M[BC] ← A       Store using BC
STAX D      M[DE] ← A       Store using DE
```

### 5.3 Comparison: M vs LDAX/STAX

```
COMPARING INDIRECT ADDRESSING METHODS:

M Reference (uses HL):
─────────────────────────────────────────
- More versatile - works with MOV, ADD, SUB, etc.
- Only uses HL pair
- Example: MOV A, M ; ADD M ; MOV M, B

LDAX/STAX (uses BC or DE):
─────────────────────────────────────────
- Limited to Load/Store only
- Uses BC or DE pair
- Useful when HL is busy with other operations
- Example: LDAX B ; STAX D

PRACTICAL USE - Memory Copy:
─────────────────────────────────────────
; Copy 100 bytes from 2000H to 3000H
    LXI H, 2000H    ; Source pointer
    LXI D, 3000H    ; Destination pointer  
    MVI C, 64H      ; Counter = 100
    
LOOP:
    MOV A, M        ; A ← M[HL] (source)
    STAX D          ; M[DE] ← A (destination)
    INX H           ; HL = HL + 1
    INX D           ; DE = DE + 1
    DCR C           ; C = C - 1
    JNZ LOOP        ; Repeat if C ≠ 0
```

---

## 6. Implied (Implicit) Addressing Mode

### 6.1 Concept

```
IMPLIED ADDRESSING:

The operand is implicitly specified by the instruction opcode.
No operand field needed - the instruction implies what to operate on.
─────────────────────────────────────────────────────────────────

Instruction Format:
┌─────────────────┐
│     Opcode      │  (Operand is implicit)
└─────────────────┘
    1-Byte


Examples:
─────────────────────────────────────────────────────────────────

CMA (Complement Accumulator):
┌───────────────┐       ┌───────────────┐
│  A = 5AH      │──────►│  A = A5H      │
│  0101 1010    │       │  1010 0101    │
└───────────────┘       └───────────────┘
Implied: Operate on Accumulator


STC (Set Carry):
┌───────────────┐       ┌───────────────┐
│   CY = 0      │──────►│   CY = 1      │
└───────────────┘       └───────────────┘
Implied: Operate on Carry flag


XCHG (Exchange HL and DE):
Before:                    After:
┌─────┬─────┐             ┌─────┬─────┐
│ H=20│ L=50│◄───────────►│ H=30│ L=00│
└─────┴─────┘             └─────┴─────┘
┌─────┬─────┐             ┌─────┬─────┐
│ D=30│ E=00│◄───────────►│ D=20│ E=50│
└─────┴─────┘             └─────┴─────┘
Implied: Exchange HL ↔ DE
```

### 6.2 Implied Addressing Instructions

| Instruction | Opcode | Operation | Implied Operand |
|-------------|--------|-----------|-----------------|
| CMA | 2F | A ← NOT A | Accumulator |
| STC | 37 | CY ← 1 | Carry flag |
| CMC | 3F | CY ← NOT CY | Carry flag |
| DAA | 27 | Decimal adjust A | Accumulator |
| RLC | 07 | Rotate A left | Accumulator, CY |
| RRC | 0F | Rotate A right | Accumulator, CY |
| RAL | 17 | Rotate A left thru CY | Accumulator, CY |
| RAR | 1F | Rotate A right thru CY | Accumulator, CY |
| XCHG | EB | HL ↔ DE | HL, DE |
| XTHL | E3 | HL ↔ Stack top | HL, Stack |
| SPHL | F9 | SP ← HL | SP, HL |
| PCHL | E9 | PC ← HL | PC, HL |
| NOP | 00 | No operation | None |
| HLT | 76 | Halt processor | Control unit |
| DI | F3 | Disable interrupts | Interrupt system |
| EI | FB | Enable interrupts | Interrupt system |
| RET | C9 | Return | PC, Stack |

---

## 7. Addressing Mode Comparison

### 7.1 Quick Reference Table

```
ADDRESSING MODE COMPARISON:

┌──────────────┬──────────┬───────────────────┬─────────────────────┐
│    Mode      │  Bytes   │  Operand Location │      Example        │
├──────────────┼──────────┼───────────────────┼─────────────────────┤
│  Immediate   │   2-3    │  In instruction   │  MVI A, 45H         │
│  Register    │    1     │  In CPU register  │  MOV A, B           │
│  Direct      │    3     │  Address in inst  │  LDA 2050H          │
│  Indirect    │    1     │  Address in reg   │  MOV A, M           │
│  Implied     │    1     │  Fixed by opcode  │  CMA                │
└──────────────┴──────────┴───────────────────┴─────────────────────┘
```

### 7.2 Speed Comparison

```
EXECUTION SPEED (T-states):

┌──────────────────────────────────────────────────────────────────┐
│                                                                   │
│  FASTEST ─────────────────────────────────► SLOWEST              │
│                                                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐         │
│  │ Register │  │ Implied  │  │ Indirect │  │ Immediate│         │
│  │  4 T     │  │  4 T     │  │  7 T     │  │  7 T     │         │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘         │
│                                                                   │
│                              ┌──────────┐                        │
│                              │  Direct  │                        │
│                              │ 13-16 T  │                        │
│                              └──────────┘                        │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘

Register: No memory access for operand
Implied:  No operand fetch needed
Indirect: One memory access (HL already has address)
Immediate: One memory fetch for data
Direct:   Two memory accesses (address bytes + data)
```

### 7.3 Practical Applications

```
WHEN TO USE EACH MODE:

┌──────────────┬────────────────────────────────────────────────────┐
│    Mode      │              Best For                              │
├──────────────┼────────────────────────────────────────────────────┤
│  Immediate   │  Loading constants, initialization                 │
│              │  Example: MVI A, 00H (clear A)                     │
├──────────────┼────────────────────────────────────────────────────┤
│  Register    │  Fast data manipulation, arithmetic                │
│              │  Example: ADD B (register-to-register)             │
├──────────────┼────────────────────────────────────────────────────┤
│  Direct      │  Fixed memory locations (I/O ports, constants)     │
│              │  Example: LDA 8000H (read from I/O)                │
├──────────────┼────────────────────────────────────────────────────┤
│  Indirect    │  Array processing, tables, dynamic memory          │
│              │  Example: MOV A, M (traverse array)                │
├──────────────┼────────────────────────────────────────────────────┤
│  Implied     │  Flag manipulation, special operations             │
│              │  Example: CMA (complement accumulator)             │
└──────────────┴────────────────────────────────────────────────────┘
```

---

## 8. Effective Address Calculation

```
EFFECTIVE ADDRESS SUMMARY:

┌───────────────────────────────────────────────────────────────────┐
│                                                                    │
│  Mode          Effective Address (EA)                             │
│  ─────────────────────────────────────────────────────────────    │
│                                                                    │
│  Immediate:    EA = N/A (data in instruction)                     │
│                                                                    │
│  Register:     EA = N/A (data in register)                        │
│                                                                    │
│  Direct:       EA = Address specified in instruction              │
│                EA = addr (16-bit from instruction bytes)          │
│                                                                    │
│  Indirect:     EA = Contents of register pair                     │
│                EA = HL, BC, or DE (pointer value)                 │
│                                                                    │
│  Implied:      EA = N/A (operand fixed by instruction)            │
│                                                                    │
└───────────────────────────────────────────────────────────────────┘

EXAMPLE CALCULATIONS:

1. LDA 2050H
   EA = 2050H (from instruction)
   
2. MOV A, M (HL = 3000H)
   EA = HL = 3000H
   
3. LDAX D (DE = 4500H)
   EA = DE = 4500H
   
4. STAX B (BC = 2100H)
   EA = BC = 2100H
```

---

## 📋 Summary Table

| Addressing Mode | Operand Source | Instruction Size | Speed | Example |
|-----------------|----------------|------------------|-------|---------|
| Immediate | In instruction | 2-3 bytes | Medium | MVI A, 45H |
| Register | CPU register | 1 byte | Fast | MOV A, B |
| Direct | Memory (addr in inst) | 3 bytes | Slow | LDA 2050H |
| Indirect | Memory (addr in reg) | 1 byte | Medium | MOV A, M |
| Implied | Fixed by opcode | 1 byte | Fast | CMA |

---

## ❓ Quick Revision Questions

1. **How many addressing modes does 8085 support? Name them.**
   <details>
   <summary>Show Answer</summary>
   8085 supports 5 addressing modes: (1) Immediate, (2) Register, (3) Direct/Absolute, (4) Indirect/Register Indirect, (5) Implied/Implicit.
   </details>

2. **What is the difference between Direct and Indirect addressing?**
   <details>
   <summary>Show Answer</summary>
   Direct: The 16-bit memory address is specified in the instruction itself (3 bytes). Example: LDA 2050H.
   Indirect: A register pair contains the memory address. Example: MOV A, M (HL contains address). Indirect is faster (1 byte) but requires setting up the register pair first.
   </details>

3. **Why is Register addressing the fastest mode?**
   <details>
   <summary>Show Answer</summary>
   Register addressing is fastest because: (1) It's a 1-byte instruction (only opcode fetch needed), (2) No memory access required for operand - data is already in CPU register, (3) Register access is much faster than memory access.
   </details>

4. **What register pairs can be used for indirect addressing?**
   <details>
   <summary>Show Answer</summary>
   HL pair: Used with M operand (MOV, ADD, SUB, etc.). Most versatile.
   BC pair: Used with LDAX B and STAX B (load/store only).
   DE pair: Used with LDAX D and STAX D (load/store only).
   </details>

5. **Identify the addressing mode: MVI B, 30H**
   <details>
   <summary>Show Answer</summary>
   Immediate Addressing Mode. The value 30H is specified immediately in the instruction (after opcode). The instruction loads the immediate value 30H into register B.
   </details>

6. **What is the effective address for MOV A, M if HL = 2500H?**
   <details>
   <summary>Show Answer</summary>
   EA = 2500H. In indirect addressing using M, the effective address equals the contents of the HL register pair. The data at memory location 2500H will be loaded into accumulator A.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [2.3 Instruction Set Classification](03-instruction-set-classification.md) | [Unit 2 Index](README.md) | [2.5 Timing Diagrams](05-timing-diagrams.md) |

---

*[← Previous: Instruction Set Classification](03-instruction-set-classification.md) | [Next: Timing Diagrams →](05-timing-diagrams.md)*
