# Chapter 7.4: Microprogrammed CPU Design

## 📋 Chapter Overview

This chapter presents a complete microprogrammed CPU design, integrating control memory, microinstruction format, and sequencing logic. We'll trace through a full example showing how microprograms implement machine instructions.

---

## 🎯 Learning Objectives

- Understand complete microprogrammed control unit architecture
- Study microinstruction encoding for a real CPU
- Trace microprogram execution for sample instructions
- Compare microprogrammed vs hardwired implementations

---

## 1. CPU Architecture Overview

### 📌 Complete System Block Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    MICROPROGRAMMED CPU ARCHITECTURE                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │                     ┌─────────────────────────────────────────┐   │ │
│   │                     │         CONTROL UNIT                    │   │ │
│   │                     │  ┌─────────────────────────────────────┐│   │ │
│   │   IR ──────────────►│  │                                     ││   │ │
│   │                     │  │    ┌───────────────────────────┐    ││   │ │
│   │                     │  │    │    CONTROL MEMORY         │    ││   │ │
│   │                     │  │    │    (128 × 20-bit ROM)     │    ││   │ │
│   │                     │  │    └─────────────┬─────────────┘    ││   │ │
│   │                     │  │                  │                  ││   │ │
│   │                     │  │                  ▼                  ││   │ │
│   │                     │  │    ┌───────────────────────────┐    ││   │ │
│   │                     │  │    │    μIR (20 bits)          │    ││   │ │
│   │                     │  │    └─────────────┬─────────────┘    ││   │ │
│   │                     │  │                  │                  ││   │ │
│   │                     │  │    ┌─────────────┴─────────────┐    ││   │ │
│   │                     │  │    │  Sequencing Logic         │    ││   │ │
│   │                     │  │    │  (μAR, SBR, MUX)          │    ││   │ │
│   │                     │  │    └───────────────────────────┘    ││   │ │
│   │                     │  │                                     ││   │ │
│   │   Flags ───────────►│  └─────────────────────────────────────┘│   │ │
│   │                     │                    │                    │   │ │
│   │                     └────────────────────│────────────────────┘   │ │
│   │                                          │                        │ │
│   │                            Control Signals                        │ │
│   │                                          │                        │ │
│   │                                          ▼                        │ │
│   │   ┌─────────────────────────────────────────────────────────────┐│ │
│   │   │                      DATAPATH                               ││ │
│   │   │                                                             ││ │
│   │   │  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐         ││ │
│   │   │  │  PC  │  │  AR  │  │  IR  │  │  DR  │  │  AC  │         ││ │
│   │   │  └──┬───┘  └──┬───┘  └──┬───┘  └──┬───┘  └──┬───┘         ││ │
│   │   │     │         │         │         │         │              ││ │
│   │   │     └────┬────┴────┬────┴────┬────┴────┬────┘              ││ │
│   │   │          │         │         │         │                   ││ │
│   │   │          ▼         ▼         ▼         ▼                   ││ │
│   │   │     ════════════════════════════════════════               ││ │
│   │   │                    COMMON BUS                              ││ │
│   │   │     ════════════════════════════════════════               ││ │
│   │   │          │                        │                        ││ │
│   │   │          ▼                        ▼                        ││ │
│   │   │     ┌─────────┐             ┌─────────┐                   ││ │
│   │   │     │   ALU   │             │ MEMORY  │                   ││ │
│   │   │     └─────────┘             └─────────┘                   ││ │
│   │   │                                                             ││ │
│   │   └─────────────────────────────────────────────────────────────┘│ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Microinstruction Format Design

### 📌 20-bit Microinstruction

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    MICROINSTRUCTION FORMAT (20 bits)                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   Bit:  19  18  17  16  15  14  13  12  11  10   9   8   7   ... 0│ │
│   │       ┌────────────┬──────────┬──────────┬─────┬─────┬──────────┐│ │
│   │       │     F1     │    F2    │    F3    │ CD  │ BR  │   AD     ││ │
│   │       │  (3 bits)  │ (3 bits) │ (3 bits) │(2b) │(2b) │ (7 bits) ││ │
│   │       └────────────┴──────────┴──────────┴─────┴─────┴──────────┘│ │
│   │                                                                   │ │
│   │   Total: 3 + 3 + 3 + 2 + 2 + 7 = 20 bits                        │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Field Encodings:                                                       │
│                                                                          │
│   F1 (Bits 19-17): Microoperations Group 1                              │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   ┌──────┬─────────────────────────────────────────────────────┐ │ │
│   │   │ Code │ Operation                                           │ │ │
│   │   ├──────┼─────────────────────────────────────────────────────┤ │ │
│   │   │ 000  │ NOP (no operation)                                  │ │ │
│   │   │ 001  │ AR ← PC                                             │ │ │
│   │   │ 010  │ AR ← IR(0-11) (address field of IR)                 │ │ │
│   │   │ 011  │ AR ← DR                                             │ │ │
│   │   │ 100  │ PC ← AR                                             │ │ │
│   │   │ 101  │ PC ← PC + 1                                         │ │ │
│   │   │ 110  │ DR ← M[AR] (memory read)                            │ │ │
│   │   │ 111  │ M[AR] ← DR (memory write)                           │ │ │
│   │   └──────┴─────────────────────────────────────────────────────┘ │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   F2 (Bits 16-14): Microoperations Group 2                              │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   ┌──────┬─────────────────────────────────────────────────────┐ │ │
│   │   │ Code │ Operation                                           │ │ │
│   │   ├──────┼─────────────────────────────────────────────────────┤ │ │
│   │   │ 000  │ NOP                                                 │ │ │
│   │   │ 001  │ AC ← 0 (Clear AC)                                   │ │ │
│   │   │ 010  │ AC ← AC + DR                                        │ │ │
│   │   │ 011  │ AC ← AC ∧ DR (AND)                                  │ │ │
│   │   │ 100  │ AC ← DR                                             │ │ │
│   │   │ 101  │ AC ← AC'  (complement)                              │ │ │
│   │   │ 110  │ AC ← SHL(AC) (shift left)                           │ │ │
│   │   │ 111  │ AC ← SHR(AC) (shift right)                          │ │ │
│   │   └──────┴─────────────────────────────────────────────────────┘ │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   F3 (Bits 13-11): Microoperations Group 3                              │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   ┌──────┬─────────────────────────────────────────────────────┐ │ │
│   │   │ Code │ Operation                                           │ │ │
│   │   ├──────┼─────────────────────────────────────────────────────┤ │ │
│   │   │ 000  │ NOP                                                 │ │ │
│   │   │ 001  │ DR ← AC                                             │ │ │
│   │   │ 010  │ IR ← DR                                             │ │ │
│   │   │ 011  │ DR ← DR + 1 (increment DR)                          │ │ │
│   │   │ 100  │ OUT ← AC (output)                                   │ │ │
│   │   │ 101  │ AC ← INP (input)                                    │ │ │
│   │   │ 110  │ PC ← PC + 1, IEN ← 0                                │ │ │
│   │   │ 111  │ Reserved                                            │ │ │
│   │   └──────┴─────────────────────────────────────────────────────┘ │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   CD (Bits 10-9): Condition Select                                      │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │   00 = Always (unconditional)                                     │ │
│   │   01 = I (indirect bit of IR)                                     │ │
│   │   10 = S (sign of AC, AC[15])                                     │ │
│   │   11 = Z (AC = 0)                                                 │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   BR (Bits 8-7): Branch Control                                         │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │   00 = NEXT (μAR ← μAR + 1)                                       │ │
│   │   01 = JUMP (if condition: μAR ← AD)                              │ │
│   │   10 = CALL (SBR ← μAR + 1, μAR ← AD)                             │ │
│   │   11 = MAP (μAR ← Mapping ROM output)                             │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   AD (Bits 6-0): Address Field (7 bits = 128 locations)                 │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Control Memory Contents

### 📌 Microprogram Listing

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    CONTROL MEMORY MAP                                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   FETCH ROUTINE (μAddr 0-3):                                     │ │
│   │   ──────────────────────────                                     │ │
│   │   0: AR←PC          | F1=001 F2=000 F3=000 CD=00 BR=00 AD=0000001│ │
│   │   1: DR←M, PC++     | F1=110 F2=000 F3=000 CD=00 BR=00 AD=0000010│ │
│   │   2: IR←DR          | F1=000 F2=000 F3=010 CD=00 BR=00 AD=0000011│ │
│   │   3: Map to instr   | F1=000 F2=000 F3=000 CD=00 BR=11 AD=-------│ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   MAPPING ROM (Opcode → μAddr):                                  │ │
│   │   ────────────────────────────                                   │ │
│   │   ┌────────┬──────────┬─────────────────────────────────────┐    │ │
│   │   │ Opcode │ μP Start │ Instruction                         │    │ │
│   │   ├────────┼──────────┼─────────────────────────────────────┤    │ │
│   │   │  0000  │    64    │ AND - Logical AND                   │    │ │
│   │   │  0001  │    68    │ ADD - Add to AC                     │    │ │
│   │   │  0010  │    72    │ LDA - Load AC                       │    │ │
│   │   │  0011  │    76    │ STA - Store AC                      │    │ │
│   │   │  0100  │    80    │ BUN - Branch Unconditional          │    │ │
│   │   │  0101  │    84    │ BSA - Branch and Save Return        │    │ │
│   │   │  0110  │    88    │ ISZ - Increment and Skip if Zero    │    │ │
│   │   │  0111  │    92    │ (reserved)                          │    │ │
│   │   │  1xxx  │    96+   │ Register-reference instructions     │    │ │
│   │   └────────┴──────────┴─────────────────────────────────────┘    │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   INSTRUCTION MICROPROGRAMS:                                     │ │
│   │   ──────────────────────────                                     │ │
│   │                                                                   │ │
│   │   AND (μAddr 64-67):                                             │ │
│   │   64: AR←IR(addr)    | F1=010 F2=000 F3=000 CD=00 BR=00 AD=65   │ │
│   │   65: Check I        | F1=000 F2=000 F3=000 CD=01 BR=01 AD=120  │ │
│   │   66: DR←M           | F1=110 F2=000 F3=000 CD=00 BR=00 AD=67   │ │
│   │   67: AC←AC∧DR,Fetch | F1=000 F2=011 F3=000 CD=00 BR=01 AD=00   │ │
│   │                                                                   │ │
│   │   ADD (μAddr 68-71):                                             │ │
│   │   68: AR←IR(addr)    | F1=010 F2=000 F3=000 CD=00 BR=00 AD=69   │ │
│   │   69: Check I        | F1=000 F2=000 F3=000 CD=01 BR=01 AD=120  │ │
│   │   70: DR←M           | F1=110 F2=000 F3=000 CD=00 BR=00 AD=71   │ │
│   │   71: AC←AC+DR,Fetch | F1=000 F2=010 F3=000 CD=00 BR=01 AD=00   │ │
│   │                                                                   │ │
│   │   LDA (μAddr 72-75):                                             │ │
│   │   72: AR←IR(addr)    | F1=010 F2=000 F3=000 CD=00 BR=00 AD=73   │ │
│   │   73: Check I        | F1=000 F2=000 F3=000 CD=01 BR=01 AD=120  │ │
│   │   74: DR←M           | F1=110 F2=000 F3=000 CD=00 BR=00 AD=75   │ │
│   │   75: AC←DR,Fetch    | F1=000 F2=100 F3=000 CD=00 BR=01 AD=00   │ │
│   │                                                                   │ │
│   │   STA (μAddr 76-79):                                             │ │
│   │   76: AR←IR(addr)    | F1=010 F2=000 F3=000 CD=00 BR=00 AD=77   │ │
│   │   77: Check I        | F1=000 F2=000 F3=000 CD=01 BR=01 AD=120  │ │
│   │   78: DR←AC          | F1=000 F2=000 F3=001 CD=00 BR=00 AD=79   │ │
│   │   79: M←DR,Fetch     | F1=111 F2=000 F3=000 CD=00 BR=01 AD=00   │ │
│   │                                                                   │ │
│   │   INDIRECT SUBROUTINE (μAddr 120-122):                           │ │
│   │   120: DR←M          | F1=110 F2=000 F3=000 CD=00 BR=00 AD=121  │ │
│   │   121: AR←DR         | F1=011 F2=000 F3=000 CD=00 BR=00 AD=122  │ │
│   │   122: RET           | F1=000 F2=000 F3=000 CD=00 BR=00 AD=SBR  │ │
│   │                      (Return: special encoding for BR)           │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Execution Trace

### 📌 ADD Direct Instruction Trace

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    EXECUTION TRACE: ADD X (Direct)                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   Machine Instruction: ADD 500 (add contents of address 500 to AC)      │
│   Initial: PC=100, AC=25, M[500]=30                                     │
│   I-bit = 0 (direct addressing)                                         │
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   Cycle │ μAddr │ Operation           │ Register Changes         │ │
│   │   ──────┼───────┼─────────────────────┼─────────────────────────  │ │
│   │                                                                   │ │
│   │   === FETCH PHASE ===                                            │ │
│   │                                                                   │ │
│   │     1   │   0   │ AR ← PC             │ AR = 100                 │ │
│   │         │       │                     │                           │ │
│   │     2   │   1   │ DR ← M[AR]          │ DR = instruction at 100  │ │
│   │         │       │ PC ← PC + 1         │ PC = 101                 │ │
│   │         │       │                     │                           │ │
│   │     3   │   2   │ IR ← DR             │ IR = 0001|0|000111110100 │ │
│   │         │       │                     │     (ADD, I=0, addr=500)  │ │
│   │         │       │                     │                           │ │
│   │     4   │   3   │ Map IR opcode       │ μAR ← 68 (ADD routine)   │ │
│   │         │       │                     │                           │ │
│   │   === EXECUTE PHASE ===                                          │ │
│   │                                                                   │ │
│   │     5   │  68   │ AR ← IR(addr)       │ AR = 500                 │ │
│   │         │       │                     │                           │ │
│   │     6   │  69   │ Check I bit         │ I=0, condition false     │ │
│   │         │       │ (I=0, no branch)    │ Continue to μAddr 70     │ │
│   │         │       │                     │                           │ │
│   │     7   │  70   │ DR ← M[AR]          │ DR = M[500] = 30         │ │
│   │         │       │                     │                           │ │
│   │     8   │  71   │ AC ← AC + DR        │ AC = 25 + 30 = 55        │ │
│   │         │       │ Jump to 0           │ μAR = 0 (fetch next)     │ │
│   │         │       │                     │                           │ │
│   │   Result: AC = 55, PC = 101, ready for next instruction         │ │
│   │                                                                   │ │
│   │   Total cycles: 8 microinstruction cycles                        │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 📌 ADD Indirect Instruction Trace

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    EXECUTION TRACE: ADD @X (Indirect)                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   Machine Instruction: ADD @500 (add using indirect addressing)         │
│   Initial: PC=100, AC=25, M[500]=600, M[600]=30                        │
│   I-bit = 1 (indirect addressing)                                       │
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   Cycle │ μAddr │ Operation           │ Register Changes         │ │
│   │   ──────┼───────┼─────────────────────┼─────────────────────────  │ │
│   │                                                                   │ │
│   │   === FETCH PHASE ===                                            │ │
│   │                                                                   │ │
│   │     1   │   0   │ AR ← PC             │ AR = 100                 │ │
│   │     2   │   1   │ DR←M, PC++          │ DR=instr, PC=101         │ │
│   │     3   │   2   │ IR ← DR             │ IR=ADD @500 (I=1)        │ │
│   │     4   │   3   │ Map                 │ μAR = 68                 │ │
│   │                                                                   │ │
│   │   === EXECUTE PHASE ===                                          │ │
│   │                                                                   │ │
│   │     5   │  68   │ AR ← IR(addr)       │ AR = 500                 │ │
│   │     6   │  69   │ Check I bit         │ I=1, condition TRUE      │ │
│   │         │       │ CALL to 120         │ SBR=70, μAR=120          │ │
│   │         │       │                     │                           │ │
│   │   === INDIRECT SUBROUTINE ===                                    │ │
│   │                                                                   │ │
│   │     7   │ 120   │ DR ← M[AR]          │ DR = M[500] = 600        │ │
│   │     8   │ 121   │ AR ← DR             │ AR = 600 (actual addr)   │ │
│   │     9   │ 122   │ RET                 │ μAR = SBR = 70           │ │
│   │                                                                   │ │
│   │   === BACK TO ADD ===                                            │ │
│   │                                                                   │ │
│   │    10   │  70   │ DR ← M[AR]          │ DR = M[600] = 30         │ │
│   │    11   │  71   │ AC ← AC + DR        │ AC = 25 + 30 = 55        │ │
│   │         │       │ Jump to 0           │ μAR = 0                  │ │
│   │                                                                   │ │
│   │   Result: AC = 55                                                 │ │
│   │   Total cycles: 11 (vs 8 for direct) - 3 extra for indirect     │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Control Unit Hardware

### 📌 Complete Control Unit Design

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    CONTROL UNIT HARDWARE DETAIL                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │                 ┌─────────────────────────────────────────────┐   │ │
│   │                 │         CONTROL MEMORY (ROM)                │   │ │
│   │                 │         128 × 20 bits                       │   │ │
│   │                 │                                             │   │ │
│   │   μAR ─────────►│  Address ─►  ┌─────────────────────────┐   │   │ │
│   │   (7-bit)       │              │ F1│F2│F3│CD│BR│  AD     │   │   │ │
│   │                 │              │3b │3b│3b│2b│2b│ 7b      │   │   │ │
│   │                 │              └───┬─┬──┬──┬──┬──────────┘   │   │ │
│   │                 └──────────────────│─│──│──│──│──────────────┘   │ │
│   │                                    │ │  │  │  │                   │ │
│   │                                    ▼ ▼  ▼  │  │                   │ │
│   │                               ┌─────────────┐│  │                   │ │
│   │                               │  DECODERS   ││  │                   │ │
│   │                               │ F1│F2│F3    ││  │                   │ │
│   │                               │3:8│3:8│3:8  ││  │                   │ │
│   │                               └──────┬──────┘│  │                   │ │
│   │                                      │       │  │                   │ │
│   │                                      ▼       │  │                   │ │
│   │                            Control Signals   │  │                   │ │
│   │                            to Datapath       │  │                   │ │
│   │                                              │  │                   │ │
│   │   ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│─ │─ ─ ─ ─ ─ ─ ─ ─  │ │
│   │                                              │  │                   │ │
│   │                          ┌───────────────────┘  │                   │ │
│   │                          ▼                      ▼                   │ │
│   │                    ┌─────────┐            ┌─────────┐             │ │
│   │                    │   CD    │            │   BR    │             │ │
│   │                    │  2-bit  │            │  2-bit  │             │ │
│   │                    └────┬────┘            └────┬────┘             │ │
│   │                         │                      │                   │ │
│   │                         ▼                      │                   │ │
│   │   ┌──────────────────────────────────────────┐│                   │ │
│   │   │           CONDITION MUX (4:1)            ││                   │ │
│   │   │  ┌───────────────────────────────────┐   ││                   │ │
│   │   │  │ 00: 1 (always)                    │   ││                   │ │
│   │   │  │ 01: I-bit                         │◄──┴┘                   │ │
│   │   │  │ 10: AC sign                       │                        │ │
│   │   │  │ 11: AC zero                       │                        │ │
│   │   │  └─────────────────────┬─────────────┘                        │ │
│   │   │                        │ Condition Result                     │ │
│   │   └────────────────────────┼──────────────────────────────────────┘ │
│   │                            ▼                                       │ │
│   │   ┌──────────────────────────────────────────────────────────────┐│ │
│   │   │             NEXT ADDRESS LOGIC                               ││ │
│   │   │  ┌─────────────────────────────────────────────────────────┐ ││ │
│   │   │  │                                                         │ ││ │
│   │   │  │   μAR+1 ─────────────────┐                             │ ││ │
│   │   │  │   AD field ──────────────┤      ┌───────┐              │ ││ │
│   │   │  │   Mapping ROM out ───────┼─────►│  MUX  │─────► μAR    │ ││ │
│   │   │  │   SBR ───────────────────┤      └───────┘              │ ││ │
│   │   │  │                          │          ▲                  │ ││ │
│   │   │  │                          │          │                  │ ││ │
│   │   │  │                    BR, Condition────┘                  │ ││ │
│   │   │  │                                                         │ ││ │
│   │   │  └─────────────────────────────────────────────────────────┘ ││ │
│   │   └──────────────────────────────────────────────────────────────┘│ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Comparison: Microprogrammed vs Hardwired

### 📌 Design Trade-offs

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    MICROPROGRAMMED vs HARDWIRED CONTROL                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌─────────────────────────────────────────────────────────────────────┐
│   │                                                                     │
│   │   Characteristic      │ Microprogrammed    │ Hardwired             │
│   │   ────────────────────┼────────────────────┼───────────────────────│
│   │   Design Complexity   │ Simpler (software) │ Complex (hardware)    │
│   │   Design Changes      │ Modify microcode   │ Redesign circuits     │
│   │   Design Time         │ Shorter            │ Longer                │
│   │   Debugging           │ Easier             │ Harder                │
│   │   Speed               │ Slower             │ Faster                │
│   │   Control Memory      │ Required (ROM)     │ Not needed            │
│   │   Instruction Set     │ Can be complex     │ Best for simple ISA   │
│   │   Flexibility         │ High               │ Low                   │
│   │   Power               │ Higher             │ Lower                 │
│   │   Die Area            │ ROM takes space    │ Random logic          │
│   │   CPI (typical)       │ 3-10+ cycles       │ 1-5 cycles            │
│   │                                                                     │
│   └─────────────────────────────────────────────────────────────────────┘
│                                                                          │
│   When to Use Each:                                                      │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   USE MICROPROGRAMMED:                                           │ │
│   │   • Complex instruction sets (CISC)                              │ │
│   │   • Variable-length instructions                                 │ │
│   │   • Need for field upgradability (bugs, new features)           │ │
│   │   • Development time is critical                                 │ │
│   │   • Multiple implementations of same ISA                        │ │
│   │                                                                   │ │
│   │   USE HARDWIRED:                                                 │ │
│   │   • Simple, fixed instruction set (RISC)                        │ │
│   │   • Maximum performance required                                 │ │
│   │   • Low power critical (embedded)                               │ │
│   │   • Instruction set is stable/final                             │ │
│   │   • Pipelining (modern approach)                                │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Historical Context:                                                   │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   1960s-1980s: Microprogrammed control dominant                  │ │
│   │   • Complex ISA (VAX, IBM 360/370, 68000)                       │ │
│   │   • Microcode fixes possible after shipping                     │ │
│   │                                                                   │ │
│   │   1980s-Present: RISC revolution                                │ │
│   │   • Hardwired control for simple instructions                    │ │
│   │   • Pipelining made microprogramming obsolete for data path     │ │
│   │                                                                   │ │
│   │   Modern x86: Hybrid approach                                    │ │
│   │   • Complex CISC instructions decoded to micro-ops              │ │
│   │   • Micro-ops executed by hardwired, pipelined core             │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Microcode Development

### 📌 Writing Microprograms

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    MICROCODE DEVELOPMENT PROCESS                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   Step 1: Analyze Machine Instruction                                   │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   Instruction: ISZ X (Increment and Skip if Zero)                │ │
│   │                                                                   │ │
│   │   Behavior:                                                       │ │
│   │   1. Read M[X] (or M[M[X]] if indirect)                          │ │
│   │   2. Increment the value                                         │ │
│   │   3. Write back to memory                                        │ │
│   │   4. If result = 0, skip next instruction (PC++)                 │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Step 2: Identify Required Microoperations                             │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   1. AR ← IR(address)      ; Get effective address               │ │
│   │   2. [Indirect check]      ; If I=1, call indirect routine       │ │
│   │   3. DR ← M[AR]            ; Read memory                         │ │
│   │   4. DR ← DR + 1           ; Increment                           │ │
│   │   5. M[AR] ← DR            ; Write back                          │ │
│   │   6. If DR=0: PC++         ; Skip if zero                        │ │
│   │   7. Return to fetch                                             │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Step 3: Encode Microinstructions                                      │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   μAddr │ F1  │ F2  │ F3  │ CD │ BR │  AD  │ Comment             │ │
│   │   ──────┼─────┼─────┼─────┼────┼────┼──────┼──────────────────── │ │
│   │    88   │ 010 │ 000 │ 000 │ 00 │ 00 │  89  │ AR←IR(addr)        │ │
│   │    89   │ 000 │ 000 │ 000 │ 01 │ 10 │ 120  │ If I: CALL 120     │ │
│   │    90   │ 110 │ 000 │ 000 │ 00 │ 00 │  91  │ DR←M[AR]           │ │
│   │    91   │ 000 │ 000 │ 011 │ 00 │ 00 │  92  │ DR←DR+1            │ │
│   │    92   │ 111 │ 000 │ 000 │ 00 │ 00 │  93  │ M[AR]←DR           │ │
│   │    93   │ 000 │ 000 │ 000 │ 11 │ 01 │  94  │ If Z: goto 94      │ │
│   │    94   │ 000 │ 000 │ 110 │ 00 │ 01 │  00  │ PC++, goto fetch   │ │
│   │    95   │ 000 │ 000 │ 000 │ 00 │ 01 │  00  │ goto fetch (no skip)│ │
│   │                                                                   │ │
│   │   Note: μAddr 93 tests if DR=0; if true jumps to 94 (skip),     │ │
│   │         if false continues to 95 (no skip, just fetch)          │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Component | Size/Bits | Function |
|-----------|-----------|----------|
| **Control Memory** | 128 × 20 bits | Stores all microprograms |
| **μAR** | 7 bits | Addresses control memory |
| **μIR** | 20 bits | Holds current microinstruction |
| **SBR** | 7 bits | Subroutine return address |
| **F1 Field** | 3 bits | Register/memory operations |
| **F2 Field** | 3 bits | ALU operations |
| **F3 Field** | 3 bits | Miscellaneous operations |
| **CD Field** | 2 bits | Condition select (4 conditions) |
| **BR Field** | 2 bits | Branch type (4 types) |
| **AD Field** | 7 bits | Branch target address |

---

## ❓ Quick Revision Questions

1. **Q**: How many microinstruction cycles does a fetch require in our design?
   
   **A**: Four cycles: (0) AR←PC, (1) DR←M, PC++, (2) IR←DR, (3) Map to instruction microprogram.

2. **Q**: How does the indirect addressing subroutine save return address?
   
   **A**: When BR=10 (CALL), SBR←μAR+1 (saves return address), then μAR←AD (jumps to subroutine). When subroutine executes RET, μAR←SBR (returns).

3. **Q**: What is the purpose of having three separate operation fields (F1, F2, F3)?
   
   **A**: Allows up to three independent microoperations in parallel (one from each field). Example: F1 can do memory read while F2 does ALU operation - but only if no conflict for resources.

4. **Q**: How many locations in control memory are allocated per instruction (using mapping)?
   
   **A**: In our design, instructions are mapped to non-uniform locations allowing variable microprogram lengths. Simple instructions (BUN) need 3-4 locations; complex ones (ISZ) need 7-8.

5. **Q**: Why does indirect addressing ADD take more cycles than direct ADD?
   
   **A**: Indirect ADD calls the indirect subroutine (3 extra cycles at μAddr 120-122) to read the pointer and update AR with the actual address before continuing.

6. **Q**: How would you add a new instruction to this CPU?
   
   **A**: (1) Add opcode to ISA and mapping ROM pointing to unused μAddr, (2) Write microprogram at that address using F1/F2/F3 for operations and BR/CD for control flow, (3) End with jump to fetch (μAddr 0).

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Microprogram Sequencing](03-microprogram-sequencing.md) | [Unit 7 Home](README.md) | [Unit 8: Pipeline Processing](../08-Pipeline-Processing/README.md) |

---

[← Previous Chapter](03-microprogram-sequencing.md) | [Course Home](../README.md) | [Next Unit →](../08-Pipeline-Processing/README.md)
