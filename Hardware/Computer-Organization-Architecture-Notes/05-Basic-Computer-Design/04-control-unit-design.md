# Chapter 5.4: Control Unit Design

## 📋 Chapter Overview

The Control Unit is the "brain" of the CPU - it generates all the control signals that coordinate the operation of the processor. This chapter covers the design of hardwired control units using combinational logic and introduces the concept of microprogrammed control.

---

## 🎯 Learning Objectives

- Understand the function of the control unit
- Design hardwired control logic for basic computer
- Derive control signal expressions
- Understand control unit timing
- Compare hardwired vs microprogrammed control

---

## 1. Control Unit Function

### 📌 Role of Control Unit

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    CONTROL UNIT OVERVIEW                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   The Control Unit:                                                      │
│   • Decodes instructions (from IR)                                      │
│   • Generates timing signals (from SC)                                  │
│   • Produces control signals for all components                         │
│   • Sequences the microoperations                                       │
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   Inputs to Control Unit:                                         │ │
│   │   ┌─────────────────────────────────────────────────────────────┐ │ │
│   │   │  • IR (Instruction Register) - opcode, I bit, address      │ │ │
│   │   │  • SC (Sequence Counter) - timing T₀, T₁, T₂...            │ │ │
│   │   │  • Flags (E, FGI, FGO, R, IEN, etc.)                       │ │ │
│   │   │  • AC bits (for conditional skips)                         │ │ │
│   │   └─────────────────────────────────────────────────────────────┘ │ │
│   │                                                                   │ │
│   │   Outputs from Control Unit:                                      │ │
│   │   ┌─────────────────────────────────────────────────────────────┐ │ │
│   │   │  • Bus selection (S₂, S₁, S₀)                              │ │ │
│   │   │  • Register load signals (LD_AR, LD_PC, LD_DR, etc.)       │ │ │
│   │   │  • Register increment/clear (INR_PC, CLR_AC, etc.)         │ │ │
│   │   │  • ALU operation select (ADD, AND, CMA, etc.)              │ │ │
│   │   │  • Memory read/write signals                               │ │ │
│   │   │  • Flag set/clear signals                                  │ │ │
│   │   └─────────────────────────────────────────────────────────────┘ │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Control Unit Block Diagram:                                            │
│                                                                          │
│        IR (15-12)                                                       │
│            │                                                            │
│            ▼                                                            │
│   ┌────────────────┐        ┌────────────────────────────────────────┐ │
│   │   3×8 Decoder  │───────►│                                        │ │
│   │  (D₀...D₇)     │        │                                        │ │
│   └────────────────┘        │                                        │ │
│                             │       CONTROL                          │ │
│        SC (3-0)             │        LOGIC                           │──► Control
│            │                │       (gates)                          │   Signals
│            ▼                │                                        │ │
│   ┌────────────────┐        │                                        │ │
│   │   4×16 Decoder │───────►│                                        │ │
│   │  (T₀...T₁₅)    │        │                                        │ │
│   └────────────────┘        │                                        │ │
│                             │                                        │ │
│   I, E, FGI, FGO, etc. ────►│                                        │ │
│                             └────────────────────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Timing Signal Generation

### 📌 Timing Decoder

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    TIMING SIGNAL GENERATION                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   Sequence Counter → Decoder → Timing Signals                           │
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │        ┌────────────┐                                             │ │
│   │   CLK ─┤   4-bit    │                                             │ │
│   │        │  Counter   │                                             │ │
│   │  CLR ──┤    (SC)    │──────────────────────────────────────┐      │ │
│   │        └────────────┘                                      │      │ │
│   │              │                                             │      │ │
│   │              │ Q₃ Q₂ Q₁ Q₀                                 │      │ │
│   │              │                                             │      │ │
│   │              ▼                                             │      │ │
│   │        ┌────────────┐                                      │      │ │
│   │        │   4×16     │                                      │      │ │
│   │        │  Decoder   │                                      │      │ │
│   │        └────────────┘                                      │      │ │
│   │              │                                             │      │ │
│   │     ┌────────┼────────┬────────┬────────┬────────┐        │      │ │
│   │     │        │        │        │        │        │        │      │ │
│   │     ▼        ▼        ▼        ▼        ▼        ▼        │      │ │
│   │    T₀       T₁       T₂       T₃      ...      T₁₅       │      │ │
│   │                                                           │      │ │
│   └───────────────────────────────────────────────────────────┘      │ │
│                                                                       │ │
│   Decoder Logic:                                                      │ │
│   ┌────────────────────────────────────────────────────────────────┐ │ │
│   │   T₀ = Q₃'Q₂'Q₁'Q₀'  (SC = 0000)                              │ │ │
│   │   T₁ = Q₃'Q₂'Q₁'Q₀   (SC = 0001)                              │ │ │
│   │   T₂ = Q₃'Q₂'Q₁ Q₀'  (SC = 0010)                              │ │ │
│   │   T₃ = Q₃'Q₂'Q₁ Q₀   (SC = 0011)                              │ │ │
│   │   ...                                                          │ │ │
│   │   T₁₅ = Q₃ Q₂ Q₁ Q₀  (SC = 1111)                              │ │ │
│   └────────────────────────────────────────────────────────────────┘ │ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Instruction Decoder

### 📌 Opcode Decoding

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    INSTRUCTION DECODING                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   IR bits 12-14 decoded to select instruction type:                    │
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   IR(14)──┐                                                       │ │
│   │           │     ┌────────────┐                                    │ │
│   │   IR(13)──┼────►│   3×8      │─────► D₀ (AND)                     │ │
│   │           │     │  Decoder   │─────► D₁ (ADD)                     │ │
│   │   IR(12)──┘     │            │─────► D₂ (LDA)                     │ │
│   │                 │            │─────► D₃ (STA)                     │ │
│   │                 │            │─────► D₄ (BUN)                     │ │
│   │                 │            │─────► D₅ (BSA)                     │ │
│   │                 │            │─────► D₆ (ISZ)                     │ │
│   │                 │            │─────► D₇ (Register/I/O)            │ │
│   │                 └────────────┘                                    │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Decoder Equations:                                                     │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │   D₀ = IR(14)'·IR(13)'·IR(12)'    (000)                         │  │
│   │   D₁ = IR(14)'·IR(13)'·IR(12)     (001)                         │  │
│   │   D₂ = IR(14)'·IR(13) ·IR(12)'    (010)                         │  │
│   │   D₃ = IR(14)'·IR(13) ·IR(12)     (011)                         │  │
│   │   D₄ = IR(14) ·IR(13)'·IR(12)'    (100)                         │  │
│   │   D₅ = IR(14) ·IR(13)'·IR(12)     (101)                         │  │
│   │   D₆ = IR(14) ·IR(13) ·IR(12)'    (110)                         │  │
│   │   D₇ = IR(14) ·IR(13) ·IR(12)     (111)                         │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│   I bit from IR(15):                                                    │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │   I = IR(15)                                                     │  │
│   │   I' = IR(15)' (complement)                                      │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Deriving Control Signals

### 📌 Fetch Cycle Control Signals

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    FETCH CYCLE CONTROL SIGNALS                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   T₀: AR ← PC                                                            │
│   T₁: IR ← M[AR], PC ← PC + 1                                           │
│   T₂: D₀...D₇ ← Decode IR(12-14), AR ← IR(0-11), I ← IR(15)            │
│                                                                          │
│   Control Signal Equations for Fetch:                                   │
│                                                                          │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │   T₀ Signals:                                                    │  │
│   │   ────────────                                                   │  │
│   │   Bus Select = 010 (PC on bus)                                   │  │
│   │     S₂ = 0, S₁ = 1, S₀ = 0  at T₀                               │  │
│   │   LD_AR = T₀                                                     │  │
│   │                                                                   │  │
│   │   T₁ Signals:                                                    │  │
│   │   ────────────                                                   │  │
│   │   Bus Select = 111 (Memory on bus)                               │  │
│   │     S₂ = 1, S₁ = 1, S₀ = 1  at T₁                               │  │
│   │   READ = T₁                                                      │  │
│   │   LD_IR = T₁                                                     │  │
│   │   INR_PC = T₁                                                    │  │
│   │                                                                   │  │
│   │   T₂ Signals:                                                    │  │
│   │   ────────────                                                   │  │
│   │   (Decoding happens in decoder hardware)                         │  │
│   │   Bus Select = 101 (IR on bus, only low 12 bits used)            │  │
│   │   LD_AR = T₂                                                     │  │
│   │                                                                   │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 📌 Memory Reference Instruction Control

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    MEMORY REFERENCE CONTROL SIGNALS                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   Indirect Addressing (T₃, when I=1):                                   │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │   D₇'·I·T₃: AR ← M[AR]                                           │  │
│   │                                                                   │  │
│   │   Control Signals:                                               │  │
│   │   READ = D₇'·I·T₃                                                │  │
│   │   LD_AR = D₇'·I·T₃                                               │  │
│   │   Bus = 111 (Memory)                                             │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│   AND Instruction (D₀):                                                  │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │   D₀·T₄: DR ← M[AR]                                              │  │
│   │   D₀·T₅: AC ← AC ∧ DR, SC ← 0                                    │  │
│   │                                                                   │  │
│   │   Control Signals:                                               │  │
│   │   READ = D₀·T₄                                                   │  │
│   │   LD_DR = D₀·T₄                                                  │  │
│   │   ALU_AND = D₀·T₅                                                │  │
│   │   LD_AC = D₀·T₅                                                  │  │
│   │   CLR_SC = D₀·T₅                                                 │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│   ADD Instruction (D₁):                                                  │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │   D₁·T₄: DR ← M[AR]                                              │  │
│   │   D₁·T₅: AC ← AC + DR, E ← Cₒᵤₜ, SC ← 0                         │  │
│   │                                                                   │  │
│   │   Control Signals:                                               │  │
│   │   READ = D₁·T₄                                                   │  │
│   │   LD_DR = D₁·T₄                                                  │  │
│   │   ALU_ADD = D₁·T₅                                                │  │
│   │   LD_AC = D₁·T₅                                                  │  │
│   │   LD_E = D₁·T₅                                                   │  │
│   │   CLR_SC = D₁·T₅                                                 │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│   LDA Instruction (D₂):                                                  │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │   D₂·T₄: DR ← M[AR]                                              │  │
│   │   D₂·T₅: AC ← DR, SC ← 0                                         │  │
│   │                                                                   │  │
│   │   LD_DR = D₂·T₄                                                  │  │
│   │   LD_AC = D₂·T₅     (from bus, not ALU)                          │  │
│   │   CLR_SC = D₂·T₅                                                 │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│   STA Instruction (D₃):                                                  │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │   D₃·T₄: M[AR] ← AC, SC ← 0                                      │  │
│   │                                                                   │  │
│   │   WRITE = D₃·T₄                                                  │  │
│   │   Bus = 100 (AC on bus)                                          │  │
│   │   CLR_SC = D₃·T₄                                                 │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│   BUN Instruction (D₄):                                                  │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │   D₄·T₄: PC ← AR, SC ← 0                                         │  │
│   │                                                                   │  │
│   │   Bus = 001 (AR on bus)                                          │  │
│   │   LD_PC = D₄·T₄                                                  │  │
│   │   CLR_SC = D₄·T₄                                                 │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 📌 Combined Control Signal Expressions

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    COMBINED CONTROL EXPRESSIONS                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   Each control signal is OR of all conditions that activate it:        │
│                                                                          │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │                                                                   │  │
│   │   LD_AR = T₀ + T₂ + D₇'·I·T₃                                     │  │
│   │          ↑    ↑         ↑                                        │  │
│   │       fetch fetch   indirect                                     │  │
│   │                                                                   │  │
│   │   LD_PC = D₄·T₄ + D₅·T₅                                          │  │
│   │          ↑        ↑                                              │  │
│   │        BUN      BSA                                              │  │
│   │                                                                   │  │
│   │   INR_PC = T₁ + (r·IR(4)·AC(15)') + (r·IR(3)·AC(15))            │  │
│   │           ↑              ↑                   ↑                   │  │
│   │        fetch          SPA                  SNA                   │  │
│   │           + (r·IR(2)·(AC=0)) + (r·IR(1)·E')                      │  │
│   │                  ↑                ↑                              │  │
│   │                SZA              SZE                              │  │
│   │                                                                   │  │
│   │   LD_DR = D₀·T₄ + D₁·T₄ + D₂·T₄ + D₆·T₄                         │  │
│   │          ↑        ↑        ↑        ↑                            │  │
│   │        AND      ADD      LDA      ISZ                            │  │
│   │                                                                   │  │
│   │   LD_AC = D₀·T₅ + D₁·T₅ + D₂·T₅ + (r·IR(11)) + (r·IR(9))        │  │
│   │         + (r·IR(7)) + (r·IR(6)) + (r·IR(5)) + (p·IR(11))         │  │
│   │           ↑          ↑          ↑          ↑                     │  │
│   │         CIR        CIL        INC        INP                     │  │
│   │                                                                   │  │
│   │   READ = T₁ + D₀·T₄ + D₁·T₄ + D₂·T₄ + D₆·T₄ + D₇'·I·T₃         │  │
│   │                                                                   │  │
│   │   WRITE = D₃·T₄ + D₅·T₄ + D₆·T₆                                  │  │
│   │                                                                   │  │
│   │   CLR_SC = D₀·T₅ + D₁·T₅ + D₂·T₅ + D₃·T₄ + D₄·T₄ + D₅·T₅       │  │
│   │          + D₆·T₆ + (r·T₃) + (p·T₃) + (interrupt handling)        │  │
│   │                                                                   │  │
│   └──────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│   Where:                                                                 │
│   r = D₇·I'·T₃   (register reference condition)                        │
│   p = D₇·I·T₃    (I/O instruction condition)                           │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Hardwired Control Unit Design

### 📌 Hardwired Control Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    HARDWIRED CONTROL UNIT                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │     ┌──────────┐     ┌──────────┐                                │ │
│   │     │   SC     │────►│ 4×16     │──────► T₀, T₁, ... T₁₅        │ │
│   │     │ Counter  │     │ Decoder  │                                │ │
│   │     └──────────┘     └──────────┘                                │ │
│   │           │                                                       │ │
│   │           │ CLR_SC                                                │ │
│   │           │                                                       │ │
│   │     ┌─────┴─────────────────────────────────────────────────┐    │ │
│   │     │                                                        │    │ │
│   │     │                 CONTROL LOGIC                          │    │ │
│   │     │                                                        │    │ │
│   │     │   ┌────────────────────────────────────────────────┐  │    │ │
│   │     │   │            AND/OR Gate Network                 │  │    │ │
│   │     │   │                                                │  │    │ │
│   │     │   │   Each output is a Boolean function of:       │  │    │ │
│   │     │   │   • Timing signals (T₀ - T₁₅)                 │  │    │ │
│   │     │   │   • Decoded opcode (D₀ - D₇)                  │  │    │ │
│   │     │   │   • I bit                                     │  │    │ │
│   │     │   │   • IR bits (for register/I/O)                │  │    │ │
│   │     │   │   • Status flags (E, FGI, FGO, etc.)          │  │    │ │
│   │     │   │                                                │  │    │ │
│   │     │   └────────────────────────────────────────────────┘  │    │ │
│   │     │                         │                              │    │ │
│   │     └─────────────────────────┼──────────────────────────────┘    │ │
│   │                               │                                   │ │
│   │                               ▼                                   │ │
│   │     ┌─────────────────────────────────────────────────────┐      │ │
│   │     │              Control Signals                         │      │ │
│   │     │                                                      │      │ │
│   │     │  LD_AR  LD_PC  LD_DR  LD_AC  LD_IR  LD_TR           │      │ │
│   │     │  INR_AR INR_PC INR_DR INR_AC                        │      │ │
│   │     │  CLR_AR CLR_PC CLR_AC CLR_E  CLR_SC                 │      │ │
│   │     │  READ   WRITE                                        │      │ │
│   │     │  S₂     S₁     S₀                                   │      │ │
│   │     │  ALU_ADD ALU_AND ALU_CMA ALU_SHR ALU_SHL            │      │ │
│   │     │  ... (more signals)                                  │      │ │
│   │     │                                                      │      │ │
│   │     └─────────────────────────────────────────────────────┘      │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Inputs from IR and Flags:                                              │
│                                                                          │
│   IR(12-14) ─────► 3×8 Decoder ─────► D₀, D₁, D₂, D₃, D₄, D₅, D₆, D₇   │
│   IR(15) = I                                                            │
│   IR(0-11) ─────► Address bits (for register/I/O decode)                │
│   E, FGI, FGO, AC(15), AC=0, R, IEN ─────► Status inputs                │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 📌 Example Gate Network

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    CONTROL GATE EXAMPLES                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   LD_AR Generation:                                                      │
│   ┌────────────────────────────────────────────────────────────────────┐│
│   │                                                                    ││
│   │   T₀ ──────────────────────────────────────┐                       ││
│   │                                            │                       ││
│   │   T₂ ──────────────────────────────────────┼────►│ OR │──► LD_AR  ││
│   │                                            │     │    │           ││
│   │   D₇' ────┐                                │     └────┘           ││
│   │           │ AND                            │                       ││
│   │   I  ─────┼─────►│    │──────────────────┬─┘                       ││
│   │           │      │AND │                  │                         ││
│   │   T₃ ─────┘      │    │                  │                         ││
│   │                  └────┘                  │                         ││
│   │                                                                    ││
│   └────────────────────────────────────────────────────────────────────┘│
│                                                                          │
│   CLR_SC Generation (partial):                                          │
│   ┌────────────────────────────────────────────────────────────────────┐│
│   │                                                                    ││
│   │   D₀ ─────┐                                                        ││
│   │           │ AND                                                    ││
│   │   T₅ ─────┴─────►│    │─────┐                                     ││
│   │                  │AND │     │                                     ││
│   │                  └────┘     │                                     ││
│   │                             │                                     ││
│   │   D₁ ─────┐                 │                                     ││
│   │           │ AND             │                                     ││
│   │   T₅ ─────┴─────►│    │─────┼───────►│ OR │──► CLR_SC            ││
│   │                  │AND │     │        │    │                       ││
│   │                  └────┘     │        └────┘                       ││
│   │                             │          ↑                          ││
│   │   D₃ ─────┐                 │          │                          ││
│   │           │ AND             │       (more                         ││
│   │   T₄ ─────┴─────►│    │─────┘       inputs)                       ││
│   │                  │AND │                                           ││
│   │                  └────┘                                           ││
│   │                                                                    ││
│   │   ... (similar for all other sources of CLR_SC)                   ││
│   │                                                                    ││
│   └────────────────────────────────────────────────────────────────────┘│
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Hardwired vs Microprogrammed Control

### 📌 Comparison

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    CONTROL UNIT APPROACHES                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │   Feature          │  Hardwired       │  Microprogrammed       │   │
│   ├─────────────────────────────────────────────────────────────────┤   │
│   │   Implementation   │  Logic gates     │  Control memory (ROM)  │   │
│   │                    │  Combinational   │  with microinstructions│   │
│   ├─────────────────────────────────────────────────────────────────┤   │
│   │   Speed            │  Faster          │  Slower (memory access)│   │
│   │                    │                  │                        │   │
│   ├─────────────────────────────────────────────────────────────────┤   │
│   │   Flexibility      │  Fixed           │  Can be modified       │   │
│   │                    │  Must redesign   │  Change microcode      │   │
│   │                    │  hardware        │                        │   │
│   ├─────────────────────────────────────────────────────────────────┤   │
│   │   Design Time      │  Longer          │  Shorter               │   │
│   │                    │  Complex logic   │  Write microcode       │   │
│   ├─────────────────────────────────────────────────────────────────┤   │
│   │   Cost             │  Lower for       │  Lower for             │   │
│   │                    │  simple CPUs     │  complex CPUs          │   │
│   ├─────────────────────────────────────────────────────────────────┤   │
│   │   Debugging        │  Difficult       │  Easier                │   │
│   │                    │  Logic analyzer  │  Modify microcode      │   │
│   ├─────────────────────────────────────────────────────────────────┤   │
│   │   Irregular ISA    │  Very complex    │  Easier to implement   │   │
│   │                    │                  │                        │   │
│   ├─────────────────────────────────────────────────────────────────┤   │
│   │   Used in          │  RISC processors │  CISC processors       │   │
│   │                    │  Simple systems  │  Complex ISAs          │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│   Hardwired Control:                                                     │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   IR ──────► Decoder ──────► Logic Gates ──────► Control Signals │ │
│   │   SC ──────►         ──────►                                     │ │
│   │   Flags ───►                                                     │ │
│   │                                                                   │ │
│   │   All in combinational logic - very fast                         │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Microprogrammed Control:                                               │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   IR ──────► Mapping ──────► Control ──────► Control Signals     │ │
│   │              Logic          Memory                               │ │
│   │                             (ROM)                                │ │
│   │                               │                                   │ │
│   │                               ▼                                   │ │
│   │                         Microinstruction                         │ │
│   │                               │                                   │ │
│   │                     ┌─────────┴─────────┐                        │ │
│   │                     │ Next Address │ Control │                   │ │
│   │                     │    Logic     │  Fields │                   │ │
│   │                     └─────────────────────────┘                   │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Control Signal | Expression |
|---------------|------------|
| **LD_AR** | T₀ + T₂ + D₇'·I·T₃ |
| **LD_PC** | D₄·T₄ + D₅·T₅ |
| **INR_PC** | T₁ + skip conditions |
| **LD_DR** | D₀·T₄ + D₁·T₄ + D₂·T₄ + D₆·T₄ |
| **LD_AC** | D₀·T₅ + D₁·T₅ + D₂·T₅ + r·CLA + r·CMA + ... |
| **LD_IR** | T₁ |
| **READ** | T₁ + D₀·T₄ + D₁·T₄ + D₂·T₄ + D₆·T₄ + D₇'·I·T₃ |
| **WRITE** | D₃·T₄ + D₅·T₄ + D₆·T₆ |
| **CLR_SC** | End of each instruction |

---

## ❓ Quick Revision Questions

1. **Q**: What are the two main inputs to the control unit?
   
   **A**: The Instruction Register (IR) for opcode and I bit, and Sequence Counter (SC) for timing signals.

2. **Q**: Why is CLR_SC activated at the end of each instruction?
   
   **A**: To reset the sequence counter to T₀, starting the fetch cycle for the next instruction.

3. **Q**: How is the READ signal generated?
   
   **A**: READ = T₁ + D₀·T₄ + D₁·T₄ + D₂·T₄ + D₆·T₄ + D₇'·I·T₃. It's needed for fetch and all memory read operations.

4. **Q**: What is the main advantage of hardwired control?
   
   **A**: Speed. Combinational logic is faster than accessing control memory. Best for RISC processors.

5. **Q**: Why would microprogrammed control be preferred for CISC processors?
   
   **A**: CISC has complex, irregular instructions. Microprogramming makes implementing them easier - just write microcode instead of designing complex gate networks.

6. **Q**: How many outputs does the timing decoder have if SC is 4 bits?
   
   **A**: 2⁴ = 16 outputs (T₀ through T₁₅), though not all may be used.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Instruction Cycle](03-instruction-cycle.md) | [Unit 5 Home](README.md) | [Interrupt Handling](05-interrupt-handling.md) |

---

[← Previous Chapter](03-instruction-cycle.md) | [Course Home](../README.md) | [Next Chapter →](05-interrupt-handling.md)
