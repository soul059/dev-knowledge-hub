# Chapter 5.3: Instruction Cycle

## 📋 Chapter Overview

The instruction cycle is the fundamental operation of a CPU - the process of fetching an instruction from memory, decoding it, and executing it. This chapter details each phase of the instruction cycle, timing sequences, and the microoperations involved in executing different types of instructions.

---

## 🎯 Learning Objectives

- Understand the phases of the instruction cycle
- Trace fetch, decode, and execute operations
- Analyze timing diagrams for instruction execution
- Write RTL statements for complete instructions
- Understand indirect address resolution

---

## 1. Instruction Cycle Overview

### 📌 Basic Phases

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    INSTRUCTION CYCLE PHASES                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌────────────────────────────────────────────────────────────────────┐│
│   │                                                                    ││
│   │       ┌─────────┐        ┌─────────┐        ┌─────────┐           ││
│   │       │  FETCH  │───────►│ DECODE  │───────►│ EXECUTE │           ││
│   │       └────┬────┘        └─────────┘        └────┬────┘           ││
│   │            │                                      │                ││
│   │            │                                      │                ││
│   │            └──────────────────────────────────────┘                ││
│   │                        (Repeat forever)                            ││
│   │                                                                    ││
│   └────────────────────────────────────────────────────────────────────┘│
│                                                                          │
│   Detailed Breakdown:                                                    │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │   Phase        │  Activities                                      │ │
│   ├───────────────────────────────────────────────────────────────────┤ │
│   │   FETCH        │  • Place PC contents in AR                       │ │
│   │                │  • Read instruction from M[AR]                   │ │
│   │                │  • Place instruction in IR                       │ │
│   │                │  • Increment PC                                  │ │
│   ├───────────────────────────────────────────────────────────────────┤ │
│   │   DECODE       │  • Decode opcode (determine operation)           │ │
│   │                │  • Determine addressing mode                     │ │
│   │                │  • If indirect: fetch effective address          │ │
│   ├───────────────────────────────────────────────────────────────────┤ │
│   │   EXECUTE      │  • Fetch operand (if memory reference)           │ │
│   │                │  • Perform operation                             │ │
│   │                │  • Store result                                  │ │
│   │                │  • Update flags                                  │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Timing and Sequence Counter

### 📌 Timing Generation

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    TIMING SEQUENCE GENERATION                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   Sequence Counter (SC) generates timing signals T₀, T₁, T₂, ...       │
│                                                                          │
│   SC Structure:                                                          │
│   ┌────────────────────────────────────────────────────────────────────┐│
│   │                                                                    ││
│   │   CLK ──────┐                                                     ││
│   │             │  ┌─────────┐      ┌──────────────┐                  ││
│   │   CLR_SC ───┼─►│   4-bit │─────►│   4×16       │                  ││
│   │             │  │ Counter │      │   Decoder    │                  ││
│   │             │  │   SC    │      │              │                  ││
│   │             └─►│         │      │ T₀ T₁ ... T₁₅│                  ││
│   │                └─────────┘      └──────┬───────┘                  ││
│   │                                        │                          ││
│   │                                        ▼                          ││
│   │                              Timing Signals                       ││
│   │                                                                    ││
│   └────────────────────────────────────────────────────────────────────┘│
│                                                                          │
│   Timing Diagram:                                                        │
│                                                                          │
│   CLK ──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──             │
│         └──┘  └──┘  └──┘  └──┘  └──┘  └──┘  └──┘  └──┘                 │
│                                                                          │
│   SC    │  0  │  1  │  2  │  3  │  4  │  5  │  0  │  1  │              │
│         ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤              │
│                                             ↑                           │
│                                          SC cleared                     │
│                                                                          │
│   T₀    ─────┐                                    ┌─────                │
│              └────────────────────────────────────┘                     │
│   T₁         ┌─────┐                                                    │
│         ─────┘     └────────────────────────────────────                │
│   T₂               ┌─────┐                                              │
│         ───────────┘     └──────────────────────────────                │
│   T₃                     ┌─────┐                                        │
│         ─────────────────┘     └────────────────────────                │
│                                                                          │
│   Each Tᵢ is active for exactly one clock cycle                        │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Fetch Cycle

### 📌 Fetch Phase Microoperations

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    FETCH CYCLE                                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   The fetch cycle is SAME for ALL instructions:                         │
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │   Time  │  Microoperation           │  Description                │ │
│   ├───────────────────────────────────────────────────────────────────┤ │
│   │   T₀    │  AR ← PC                  │  Address of instruction     │ │
│   │   T₁    │  IR ← M[AR], PC ← PC + 1  │  Fetch & increment PC       │ │
│   │   T₂    │  Decode IR(12-14)         │  Decode opcode              │ │
│   │         │  AR ← IR(0-11)            │  Get address field          │ │
│   │         │  I ← IR(15)               │  Get indirect bit           │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Data Flow During Fetch:                                                │
│                                                                          │
│   T₀: AR ← PC                                                            │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │    PC                              AR                             │ │
│   │   ┌───────┐                       ┌───────┐                       │ │
│   │   │  100  │──────► BUS ──────────►│  100  │                       │ │
│   │   └───────┘                       └───────┘                       │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   T₁: IR ← M[AR], PC ← PC + 1                                           │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   MEMORY                            IR                            │ │
│   │   ┌───────┐                       ┌───────────────┐               │ │
│   │   │ M[100]│──────► BUS ──────────►│  Instruction  │               │ │
│   │   └───────┘                       └───────────────┘               │ │
│   │                                                                   │ │
│   │    PC                                                             │ │
│   │   ┌───────┐      ┌───────┐                                        │ │
│   │   │  100  │──────│  +1   │────────►│  101  │                      │ │
│   │   └───────┘      └───────┘         └───────┘                      │ │
│   │              (internal incrementer, parallel)                     │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   T₂: Decode and Prepare                                                │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │    IR                                                             │ │
│   │   ┌─────────────────────────────────────────┐                     │ │
│   │   │ Opcode │ I │        Address             │                     │ │
│   │   │ (3bit) │   │        (12 bit)            │                     │ │
│   │   └───┬────┴─┬─┴────────────┬───────────────┘                     │ │
│   │       │      │              │                                     │ │
│   │       ▼      ▼              ▼                                     │ │
│   │    Decoder   I flag        AR                                     │ │
│   │   (select   (direct/     (operand                                 │ │
│   │   operation) indirect)   address)                                 │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Decode Phase

### 📌 Instruction Decoding

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    DECODE PHASE                                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   After T₂, instruction is decoded based on opcode and I bit:          │
│                                                                          │
│   Instruction Format:                                                    │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │  15 │ 14-12 │ 11-0                                                │ │
│   │  I  │Opcode │ Address                                             │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Opcode Decoder:                                                        │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   IR(12-14) ──────► 3×8 Decoder                                   │ │
│   │                           │                                       │ │
│   │   Outputs: D₀, D₁, D₂, D₃, D₄, D₅, D₆, D₇                        │ │
│   │                                                                   │ │
│   │   D₀ = AND    D₄ = BUN                                           │ │
│   │   D₁ = ADD    D₅ = BSA                                           │ │
│   │   D₂ = LDA    D₆ = ISZ                                           │ │
│   │   D₃ = STA    D₇ = (Register or I/O)                             │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Instruction Type Determination:                                        │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   D₇' = 0: Memory Reference Instruction                          │ │
│   │             (D₀ to D₆ active)                                     │ │
│   │                                                                   │ │
│   │   D₇ · I' = 1: Register Reference Instruction                    │ │
│   │                 (opcode = 111, I = 0)                             │ │
│   │                                                                   │ │
│   │   D₇ · I  = 1: Input/Output Instruction                          │ │
│   │                 (opcode = 111, I = 1)                             │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Timing after Decode:                                                   │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   D₇'I · T₃: (Indirect) AR ← M[AR]                               │ │
│   │              Fetch effective address                              │ │
│   │                                                                   │ │
│   │   D₇'I' · T₃: (Direct) Nothing, AR already has address          │ │
│   │               Ready to execute                                    │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 📌 Indirect Address Resolution

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    INDIRECT ADDRESSING                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   When I = 1: The address in IR points to memory containing            │
│               the actual operand address                                │
│                                                                          │
│   Example: ADD @500 (indirect add from address 500)                     │
│                                                                          │
│   Memory:                                                                │
│   ┌───────┬─────────────┐                                              │
│   │  500  │    800      │  ◄── Points to actual data                   │
│   ├───────┼─────────────┤                                              │
│   │  ...  │    ...      │                                              │
│   ├───────┼─────────────┤                                              │
│   │  800  │    42       │  ◄── Actual operand                          │
│   └───────┴─────────────┘                                              │
│                                                                          │
│   Direct (I=0):                                                          │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │   T₃: DR ← M[AR]           // Get operand from M[500] = ERROR!   │  │
│   │   T₄: AC ← AC + DR         // Wrong result (using 800 as data)   │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│   Indirect (I=1):                                                        │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │   T₃: AR ← M[AR]           // AR = M[500] = 800 (get address)    │  │
│   │   T₄: DR ← M[AR]           // DR = M[800] = 42 (get operand)     │  │
│   │   T₅: AC ← AC + DR         // Correct result                     │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│   Indirect addressing adds one extra clock cycle                        │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Execute Phase - Memory Reference Instructions

### 📌 Memory Reference Instruction Execution

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    MEMORY REFERENCE INSTRUCTIONS                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   After fetch (T₀-T₂) and possible indirect (T₃ if I=1):               │
│                                                                          │
│   AND (D₀):  AC ← AC ∧ M[AR]                                           │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │   D₀T₄: DR ← M[AR]                // Read memory operand         │  │
│   │   D₀T₅: AC ← AC ∧ DR, SC ← 0      // AND and clear SC            │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│   ADD (D₁):  AC ← AC + M[AR], E ← Carry                                │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │   D₁T₄: DR ← M[AR]                // Read memory operand         │  │
│   │   D₁T₅: AC ← AC + DR, E ← Cₒᵤₜ    // Add with carry              │  │
│   │         SC ← 0                                                    │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│   LDA (D₂):  AC ← M[AR]                                                │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │   D₂T₄: DR ← M[AR]                // Read memory operand         │  │
│   │   D₂T₅: AC ← DR, SC ← 0           // Transfer to AC              │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│   STA (D₃):  M[AR] ← AC                                                │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │   D₃T₄: M[AR] ← AC, SC ← 0        // Write AC to memory          │  │
│   │         (Only 1 execute cycle!)                                   │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│   BUN (D₄):  PC ← AR (Unconditional Branch)                            │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │   D₄T₄: PC ← AR, SC ← 0           // Jump to address in AR       │  │
│   │         (Only 1 execute cycle!)                                   │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│   BSA (D₅):  M[AR] ← PC, PC ← AR + 1 (Branch and Save Return)         │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │   D₅T₄: M[AR] ← PC, AR ← AR + 1   // Save return, point to code  │  │
│   │   D₅T₅: PC ← AR, SC ← 0           // Jump to subroutine          │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│   ISZ (D₆):  M[AR] ← M[AR] + 1, skip if zero                          │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │   D₆T₄: DR ← M[AR]                // Read counter                │  │
│   │   D₆T₅: DR ← DR + 1               // Increment                   │  │
│   │   D₆T₆: M[AR] ← DR,               // Write back                  │  │
│   │         if (DR = 0) PC ← PC + 1   // Skip next if zero           │  │
│   │         SC ← 0                                                    │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Execute Phase - Register Reference Instructions

### 📌 Register Reference Instructions

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    REGISTER REFERENCE INSTRUCTIONS                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   Condition: D₇ · I' · T₃   (opcode=111, I=0, at T₃)                   │
│   Address bits (0-11) specify the operation                            │
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │   Instruction │   IR(0-11)    │   Operation                       │ │
│   ├───────────────────────────────────────────────────────────────────┤ │
│   │   CLA         │   800 (hex)   │   AC ← 0                          │ │
│   │   CLE         │   400         │   E ← 0                           │ │
│   │   CMA         │   200         │   AC ← AC'                        │ │
│   │   CME         │   100         │   E ← E'                          │ │
│   │   CIR         │   080         │   Rotate right AC and E           │ │
│   │   CIL         │   040         │   Rotate left AC and E            │ │
│   │   INC         │   020         │   AC ← AC + 1                     │ │
│   │   SPA         │   010         │   if (AC > 0) PC ← PC + 1         │ │
│   │   SNA         │   008         │   if (AC < 0) PC ← PC + 1         │ │
│   │   SZA         │   004         │   if (AC = 0) PC ← PC + 1         │ │
│   │   SZE         │   002         │   if (E = 0) PC ← PC + 1          │ │
│   │   HLT         │   001         │   S ← 0 (Halt)                    │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   RTL for Each:                                                          │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │   r = D₇·I'·T₃    (register reference condition)                 │  │
│   │                                                                   │  │
│   │   r·IR(11): AC ← 0                    (CLA)                      │  │
│   │   r·IR(10): E ← 0                     (CLE)                      │  │
│   │   r·IR(9):  AC ← AC'                  (CMA)                      │  │
│   │   r·IR(8):  E ← E'                    (CME)                      │  │
│   │   r·IR(7):  AC ← shr(AC), AC(15) ← E, E ← AC(0)  (CIR)          │  │
│   │   r·IR(6):  AC ← shl(AC), AC(0) ← E, E ← AC(15)  (CIL)          │  │
│   │   r·IR(5):  AC ← AC + 1               (INC)                      │  │
│   │   r·IR(4):  if (AC(15) = 0) PC ← PC + 1  (SPA, skip if positive)│  │
│   │   r·IR(3):  if (AC(15) = 1) PC ← PC + 1  (SNA, skip if negative)│  │
│   │   r·IR(2):  if (AC = 0) PC ← PC + 1   (SZA)                      │  │
│   │   r·IR(1):  if (E = 0) PC ← PC + 1    (SZE)                      │  │
│   │   r·IR(0):  S ← 0                     (HLT)                      │  │
│   │                                                                   │  │
│   │   All followed by: SC ← 0                                        │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│   Rotate Operations:                                                     │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │                                                                   │  │
│   │   CIR (Circular Right):                                          │  │
│   │   ┌───┐  ┌─────────────────────────────────────────┐             │  │
│   │   │ E │◄─│ AC₁₅ AC₁₄ ... AC₂ AC₁ AC₀             │◄─┐          │  │
│   │   └─┬─┘  └─────────────────────────────────────────┘  │          │  │
│   │     └──────────────────────────────────────────────────┘          │  │
│   │                                                                   │  │
│   │   CIL (Circular Left):                                           │  │
│   │   ┌───┐  ┌─────────────────────────────────────────┐             │  │
│   │ ┌►│ E │─►│ AC₁₅ AC₁₄ ... AC₂ AC₁ AC₀             │──┐          │  │
│   │ │ └───┘  └─────────────────────────────────────────┘  │          │  │
│   │ └──────────────────────────────────────────────────────┘          │  │
│   │                                                                   │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Execute Phase - I/O Instructions

### 📌 Input/Output Instructions

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    I/O INSTRUCTIONS                                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   Condition: D₇ · I · T₃   (opcode=111, I=1, at T₃)                    │
│   Address bits specify I/O operation                                    │
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │   Instruction │   IR(0-11)  │   Operation                         │ │
│   ├───────────────────────────────────────────────────────────────────┤ │
│   │   INP         │    800      │   AC(0-7) ← INPR, FGI ← 0           │ │
│   │   OUT         │    400      │   OUTR ← AC(0-7), FGO ← 0           │ │
│   │   SKI         │    200      │   if (FGI = 1) PC ← PC + 1          │ │
│   │   SKO         │    100      │   if (FGO = 1) PC ← PC + 1          │ │
│   │   ION         │    080      │   IEN ← 1 (Enable Interrupt)        │ │
│   │   IOF         │    040      │   IEN ← 0 (Disable Interrupt)       │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   RTL:                                                                   │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │   p = D₇·I·T₃    (I/O instruction condition)                     │  │
│   │                                                                   │  │
│   │   p·IR(11): AC(0-7) ← INPR, FGI ← 0       (INP)                  │  │
│   │   p·IR(10): OUTR ← AC(0-7), FGO ← 0       (OUT)                  │  │
│   │   p·IR(9):  if (FGI = 1) PC ← PC + 1      (SKI)                  │  │
│   │   p·IR(8):  if (FGO = 1) PC ← PC + 1      (SKO)                  │  │
│   │   p·IR(7):  IEN ← 1                       (ION)                  │  │
│   │   p·IR(6):  IEN ← 0                       (IOF)                  │  │
│   │                                                                   │  │
│   │   All followed by: SC ← 0                                        │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│   Programmed I/O Example (polling):                                     │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │   LOOP: SKI        // Skip next if input ready                   │  │
│   │         BUN LOOP   // Not ready, keep polling                    │  │
│   │         INP        // Ready, read character                      │  │
│   │         STA DATA   // Store character                            │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 8. Complete Instruction Cycle Flowchart

### 📌 Flowchart

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    INSTRUCTION CYCLE FLOWCHART                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│                         ┌─────────┐                                     │
│                         │  START  │                                     │
│                         └────┬────┘                                     │
│                              ▼                                          │
│                    ┌───────────────────┐                                │
│              T₀:   │    AR ← PC        │                                │
│                    └─────────┬─────────┘                                │
│                              ▼                                          │
│                    ┌───────────────────┐                                │
│              T₁:   │ IR ← M[AR]        │                                │
│                    │ PC ← PC + 1       │                                │
│                    └─────────┬─────────┘                                │
│                              ▼                                          │
│                    ┌───────────────────┐                                │
│              T₂:   │ Decode opcode     │                                │
│                    │ AR ← IR(0-11)     │                                │
│                    │ I ← IR(15)        │                                │
│                    └─────────┬─────────┘                                │
│                              ▼                                          │
│                       ╔═══════════╗                                     │
│                       ║   D₇=1?   ║                                     │
│                       ╚═════╤═════╝                                     │
│                  No ───────┬┴───────── Yes                              │
│                  ▼                       ▼                              │
│        ┌──────────────────┐      ╔═══════════╗                         │
│        │ Memory Reference │      ║   I=1?    ║                         │
│        │   Instruction    │      ╚═════╤═════╝                         │
│        └────────┬─────────┘       No ──┴── Yes                         │
│                 │                 ▼         ▼                          │
│                 │         ┌─────────┐ ┌─────────┐                      │
│          ╔══════╧══════╗  │Register │ │   I/O   │                      │
│          ║    I=1?     ║  │Reference│ │  Instr  │                      │
│          ╚══════╤══════╝  └────┬────┘ └────┬────┘                      │
│           No ───┴─── Yes       │           │                           │
│           ▼          ▼         │           │                           │
│    ┌──────────┐ ┌──────────┐   │           │                           │
│ T₃:│ (Direct) │ │AR ← M[AR]│   │           │                           │
│    │ nothing  │ │(Indirect)│   │           │                           │
│    └────┬─────┘ └────┬─────┘   │           │                           │
│         │            │         ▼           ▼                           │
│         └─────┬──────┘   ┌───────────────────┐                         │
│               ▼          │ Execute (T₃)      │                         │
│        ┌───────────────┐ │ SC ← 0            │                         │
│        │ Execute       │ └─────────┬─────────┘                         │
│        │ (T₄, T₅, ...) │           │                                   │
│        │ SC ← 0        │           │                                   │
│        └───────┬───────┘           │                                   │
│                │                   │                                   │
│                └─────────┬─────────┘                                   │
│                          │                                              │
│                          ▼                                              │
│                    ┌──────────┐                                        │
│                    │  T₀ (loop│                                        │
│                    │   back)  │                                        │
│                    └──────────┘                                        │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Instruction Type | Opcode | Execute Timing | Key Microoperations |
|-----------------|--------|----------------|---------------------|
| **AND** | 000 | T₄-T₅ | DR←M[AR], AC←AC∧DR |
| **ADD** | 001 | T₄-T₅ | DR←M[AR], AC←AC+DR, E←Cout |
| **LDA** | 010 | T₄-T₅ | DR←M[AR], AC←DR |
| **STA** | 011 | T₄ | M[AR]←AC |
| **BUN** | 100 | T₄ | PC←AR |
| **BSA** | 101 | T₄-T₅ | M[AR]←PC, PC←AR+1 |
| **ISZ** | 110 | T₄-T₆ | DR←M[AR], DR←DR+1, M[AR]←DR, skip if 0 |
| **Register** | 111, I=0 | T₃ | CLA, CLE, CMA, etc. |
| **I/O** | 111, I=1 | T₃ | INP, OUT, SKI, etc. |

---

## ❓ Quick Revision Questions

1. **Q**: What happens during T₀ of every instruction cycle?
   
   **A**: AR ← PC. The program counter content is transferred to the address register to prepare for fetching the instruction.

2. **Q**: How many clock cycles does an indirect memory reference instruction take compared to direct?
   
   **A**: One extra cycle. T₃ is used to fetch the effective address from memory (AR ← M[AR]) before the execute phase.

3. **Q**: Why does STA require only one execute cycle (T₄) while ADD needs two (T₄-T₅)?
   
   **A**: STA only writes AC to memory. ADD needs to first read the operand (T₄: DR←M[AR]), then add (T₅: AC←AC+DR).

4. **Q**: What is the purpose of clearing SC at the end of instruction execution?
   
   **A**: SC←0 resets the sequence counter to T₀, starting the next instruction fetch cycle.

5. **Q**: How does the computer distinguish between register and I/O instructions?
   
   **A**: Both have opcode=111. I=0 indicates register reference, I=1 indicates I/O instruction.

6. **Q**: What is the function of the BSA (Branch and Save Address) instruction?
   
   **A**: It saves the return address (PC) at M[AR], then jumps to AR+1. Used for subroutine calls.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Computer Registers and Memory](02-computer-registers-memory.md) | [Unit 5 Home](README.md) | [Control Unit Design](04-control-unit-design.md) |

---

[← Previous Chapter](02-computer-registers-memory.md) | [Course Home](../README.md) | [Next Chapter →](04-control-unit-design.md)
