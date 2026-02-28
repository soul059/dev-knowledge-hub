# Chapter 8.3: Data Forwarding

## 📋 Chapter Overview

Data forwarding (also called bypassing) is a technique to resolve data hazards by sending computed results directly to where they are needed, without waiting for the register file write. This chapter covers forwarding paths, detection logic, and implementation.

---

## 🎯 Learning Objectives

- Understand the concept of data forwarding
- Identify forwarding paths in the pipeline
- Design forwarding detection logic
- Analyze forwarding unit implementation

---

## 1. Forwarding Concept

### 📌 Why Forwarding Works

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    DATA FORWARDING PRINCIPLE                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   Without Forwarding:                                                   │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   ADD R1, R2, R3     ; Writes R1 in WB (cycle 5)                 │ │
│   │   SUB R4, R1, R5     ; Needs R1 in EX (cycle 4)                  │ │
│   │                                                                   │ │
│   │   Cycle:    1    2    3    4    5    6    7                      │ │
│   │                                                                   │ │
│   │   ADD:      IF   ID   EX   MEM  WB                               │ │
│   │                        ↓         ↓                                │ │
│   │                    Result     Written                             │ │
│   │                    ready      to R1                               │ │
│   │                                                                   │ │
│   │   SUB:           IF   ID   stall stall EX   MEM  WB              │ │
│   │                             waiting for R1                        │ │
│   │                                                                   │ │
│   │   Result exists (end of cycle 3) but isn't "visible" until WB   │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Key Insight:                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   The ADD result EXISTS at end of EX stage (cycle 3)            │ │
│   │   It's stored in EX/MEM pipeline register                        │ │
│   │   SUB needs it in EX stage (cycle 4)                            │ │
│   │                                                                   │ │
│   │   → Forward the value from EX/MEM register directly to ALU      │ │
│   │   → No need to wait for register file write!                     │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   With Forwarding:                                                      │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   Cycle:    1    2    3    4    5    6                           │ │
│   │                                                                   │ │
│   │   ADD:      IF   ID   EX   MEM  WB                               │ │
│   │                        │    │                                     │ │
│   │                        │    └───────┐                             │ │
│   │                        └───────┐    │                             │ │
│   │                                ▼    ▼                             │ │
│   │   SUB:           IF   ID   EX   MEM  WB                          │ │
│   │                            ↑                                      │ │
│   │                      Value forwarded                              │ │
│   │                      from EX/MEM                                  │ │
│   │                                                                   │ │
│   │   No stalls needed! Both complete on time.                       │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Forwarding Paths

### 📌 All Possible Forwarding Paths

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    FORWARDING PATHS IN 5-STAGE PIPELINE                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   Path 1: EX/MEM → EX (from ALU output to ALU input)            │ │
│   │   ────────────────────────────────────────────────               │ │
│   │   Producer in MEM stage, consumer in EX stage                   │ │
│   │   Distance: 1 instruction apart                                 │ │
│   │                                                                   │ │
│   │   I1 (producer): IF   ID   EX   MEM  WB                         │ │
│   │                             │                                    │ │
│   │   I2 (consumer):      IF   ID   EX   MEM  WB                    │ │
│   │                             ↑                                    │ │
│   │                        Forward from EX/MEM                       │ │
│   │                                                                   │ │
│   │   Path 2: MEM/WB → EX (from memory or previous ALU to ALU)      │ │
│   │   ──────────────────────────────────────────────────             │ │
│   │   Producer in WB stage, consumer in EX stage                    │ │
│   │   Distance: 2 instructions apart                                │ │
│   │                                                                   │ │
│   │   I1 (producer): IF   ID   EX   MEM  WB                         │ │
│   │                                   │                              │ │
│   │   I2:                 IF   ID   EX   MEM  WB                    │ │
│   │   I3 (consumer):           IF   ID   EX   MEM  WB               │ │
│   │                                   ↑                              │ │
│   │                            Forward from MEM/WB                   │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Datapath with Forwarding Paths:                                       │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │                ID/EX        EX/MEM       MEM/WB                  │ │
│   │                  │            │            │                      │ │
│   │          ┌───────┴─────┐ ┌───┴────┐ ┌────┴─────┐                │ │
│   │          │             │ │        │ │          │                 │ │
│   │   Rs ───►│─────────────┼─┼────────┼─┼──────────┤                │ │
│   │          │     ┌───┐   │ │        │ │          │                 │ │
│   │          │     │MUX├───┼─┼──►ALU  │ │          │                 │ │
│   │          │     │   │   │ │   │    │ │          │                 │ │
│   │   Rt ───►│─────┤   │   │ │   │    │ │          │                 │ │
│   │          │     └───┘   │ │   │    │ │          │                 │ │
│   │          │       ▲ ▲   │ │   ▼    │ │          │                 │ │
│   │          │       │ │   │ │ Result │ │          │                 │ │
│   │          └───────│─│───┘ └───┬────┘ └────┬─────┘                │ │
│   │                  │ │         │           │                        │ │
│   │                  │ └─────────┘           │                        │ │
│   │                  │     EX/MEM.Result     │                        │ │
│   │                  │                       │                        │ │
│   │                  └───────────────────────┘                        │ │
│   │                        MEM/WB.Result                              │ │
│   │                                                                   │ │
│   │   MUX inputs: 00 = Register file, 01 = EX/MEM, 10 = MEM/WB      │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Forwarding Unit Design

### 📌 Forwarding Detection Logic

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    FORWARDING UNIT LOGIC                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   The Forwarding Unit determines if forwarding is needed and from where │
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   ForwardA (for ALU input 1 - Rs):                               │ │
│   │   ────────────────────────────────                                │ │
│   │                                                                   │ │
│   │   // EX Hazard (forward from EX/MEM)                             │ │
│   │   if (EX/MEM.RegWrite == 1 AND                                   │ │
│   │       EX/MEM.Rd ≠ 0 AND                                          │ │
│   │       EX/MEM.Rd == ID/EX.Rs)                                     │ │
│   │       then ForwardA = 10   // Forward from EX/MEM                │ │
│   │                                                                   │ │
│   │   // MEM Hazard (forward from MEM/WB)                            │ │
│   │   else if (MEM/WB.RegWrite == 1 AND                              │ │
│   │            MEM/WB.Rd ≠ 0 AND                                     │ │
│   │            NOT (EX/MEM.RegWrite AND EX/MEM.Rd ≠ 0 AND            │ │
│   │                 EX/MEM.Rd == ID/EX.Rs) AND                       │ │
│   │            MEM/WB.Rd == ID/EX.Rs)                                │ │
│   │       then ForwardA = 01   // Forward from MEM/WB                │ │
│   │                                                                   │ │
│   │   else ForwardA = 00       // No forwarding, use register file   │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   ForwardB (for ALU input 2 - Rt):                               │ │
│   │   ────────────────────────────────                                │ │
│   │                                                                   │ │
│   │   // Same logic but comparing ID/EX.Rt instead of ID/EX.Rs       │ │
│   │                                                                   │ │
│   │   if (EX/MEM.RegWrite AND EX/MEM.Rd ≠ 0 AND                      │ │
│   │       EX/MEM.Rd == ID/EX.Rt)                                     │ │
│   │       then ForwardB = 10                                         │ │
│   │                                                                   │ │
│   │   else if (MEM/WB.RegWrite AND MEM/WB.Rd ≠ 0 AND                 │ │
│   │            NOT (EX Hazard for Rt) AND                            │ │
│   │            MEM/WB.Rd == ID/EX.Rt)                                │ │
│   │       then ForwardB = 01                                         │ │
│   │                                                                   │ │
│   │   else ForwardB = 00                                             │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Priority: EX/MEM (closer) takes priority over MEM/WB                  │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 📌 Forwarding Control Values

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    FORWARDING MUX CONTROL                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   ForwardA / ForwardB Values:                                    │ │
│   │                                                                   │ │
│   │   ┌─────────┬───────────────────────────────────────────────────┐│ │
│   │   │ Value   │ Source                                            ││ │
│   │   ├─────────┼───────────────────────────────────────────────────┤│ │
│   │   │   00    │ Register file (ID/EX register)                    ││ │
│   │   │         │ No hazard - use value read in ID stage            ││ │
│   │   ├─────────┼───────────────────────────────────────────────────┤│ │
│   │   │   01    │ MEM/WB pipeline register                          ││ │
│   │   │         │ Forward from memory stage result                  ││ │
│   │   ├─────────┼───────────────────────────────────────────────────┤│ │
│   │   │   10    │ EX/MEM pipeline register                          ││ │
│   │   │         │ Forward from prior ALU result                     ││ │
│   │   └─────────┴───────────────────────────────────────────────────┘│ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Why EX/MEM has priority:                                              │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   ADD R1, R1, R2    ; Writes R1                                  │ │
│   │   ADD R1, R1, R3    ; Writes R1 (uses previous R1)               │ │
│   │   ADD R4, R1, R5    ; Reads R1 - which value?                    │ │
│   │                                                                   │ │
│   │   Cycle:    1    2    3    4    5    6    7                      │ │
│   │                                                                   │ │
│   │   ADD R1:   IF   ID   EX   MEM  WB                               │ │
│   │   ADD R1:        IF   ID   EX   MEM  WB                          │ │
│   │   ADD R4:             IF   ID   EX   MEM  WB                     │ │
│   │                            ↑    ↑                                 │ │
│   │                            │    │                                 │ │
│   │                            │    MEM/WB has first ADD's R1        │ │
│   │                            EX/MEM has second ADD's R1            │ │
│   │                                                                   │ │
│   │   ADD R4 needs the SECOND ADD's result (most recent write)      │ │
│   │   → EX/MEM (closer) must have priority                          │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Forwarding Implementation

### 📌 Datapath with Forwarding Unit

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    FORWARDING UNIT IN DATAPATH                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │                                                                   │ │
│   │   From ID/EX                                    EX/MEM   MEM/WB   │ │
│   │   ─────────                                     ─────    ─────    │ │
│   │   ID/EX.Rs ─────────────────────┐                                 │ │
│   │   ID/EX.Rt ──────────────────┐  │                                 │ │
│   │                              │  │                                 │ │
│   │                              ▼  ▼                                 │ │
│   │                         ┌─────────────┐                           │ │
│   │   EX/MEM.Rd ──────────►│             │                           │ │
│   │   EX/MEM.RegWrite ────►│  Forwarding │──► ForwardA               │ │
│   │   MEM/WB.Rd ──────────►│    Unit     │──► ForwardB               │ │
│   │   MEM/WB.RegWrite ────►│             │                           │ │
│   │                         └─────────────┘                           │ │
│   │                                                                   │ │
│   │   ─────────────────────────────────────────────────────────────  │ │
│   │                                                                   │ │
│   │   ID/EX.Rs_data ──┐                                              │ │
│   │                   │     ForwardA                                  │ │
│   │   EX/MEM.Result ──┼──►┌─────┐                                    │ │
│   │                   │   │ MUX │───────────► ALU Input A            │ │
│   │   MEM/WB.Result ──┼──►│ 3:1 │                                    │ │
│   │                   │   └─────┘                                    │ │
│   │                   │                                              │ │
│   │   ID/EX.Rt_data ──┤                                              │ │
│   │                   │     ForwardB                                  │ │
│   │   EX/MEM.Result ──┼──►┌─────┐                                    │ │
│   │                   │   │ MUX │───► MUX ──► ALU Input B            │ │
│   │   MEM/WB.Result ──┘   │ 3:1 │     (ALUSrc)                       │ │
│   │                       └─────┘        │                            │ │
│   │                                      │                            │ │
│   │   ID/EX.Immediate ───────────────────┘                           │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Forwarding Examples

### 📌 Example 1: Back-to-Back Dependencies

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    FORWARDING EXAMPLE 1                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   Code:                                                                 │
│   I1: ADD R1, R2, R3                                                    │
│   I2: SUB R4, R1, R5     ; Depends on R1 from I1                        │
│   I3: AND R6, R4, R7     ; Depends on R4 from I2                        │
│   I4: OR  R8, R4, R9     ; Depends on R4 from I2                        │
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   Cycle:    1    2    3    4    5    6    7    8                 │ │
│   │                                                                   │ │
│   │   I1 ADD:   IF   ID   EX   MEM  WB                               │ │
│   │                        │    │                                     │ │
│   │                        │    └─────┐                               │ │
│   │                        │          │                               │ │
│   │   I2 SUB:        IF   ID   EX   MEM  WB                          │ │
│   │                            ↑    │    │                            │ │
│   │                  Forward   │    │    └─────┐                      │ │
│   │                  EX/MEM    │    │          │                      │ │
│   │                            │    │          │                      │ │
│   │   I3 AND:             IF   ID   EX   MEM  WB                     │ │
│   │                                 ↑    │                            │ │
│   │                       Forward   │    │                            │ │
│   │                       EX/MEM    │    │                            │ │
│   │                                 │    │                            │ │
│   │   I4 OR:                   IF   ID   EX   MEM  WB                │ │
│   │                                      ↑                            │ │
│   │                            Forward from MEM/WB                   │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Forwarding Actions:                                                   │
│   • Cycle 4: I2 EX gets R1 from EX/MEM (I1's result)                   │
│   • Cycle 5: I3 EX gets R4 from EX/MEM (I2's result)                   │
│   • Cycle 6: I4 EX gets R4 from MEM/WB (I2's result)                   │
│                                                                          │
│   No stalls needed! All four instructions complete in 8 cycles.        │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 📌 Example 2: Double Data Hazard

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    FORWARDING EXAMPLE 2: DOUBLE HAZARD                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   Code:                                                                 │
│   I1: ADD R1, R1, R2     ; R1 = R1 + R2                                 │
│   I2: ADD R1, R1, R3     ; R1 = R1 + R3 (uses result of I1)             │
│   I3: ADD R1, R1, R4     ; R1 = R1 + R4 (uses result of I2)             │
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   Cycle:    1    2    3    4    5    6    7                      │ │
│   │                                                                   │ │
│   │   I1:       IF   ID   EX   MEM  WB                               │ │
│   │                        ↓                                          │ │
│   │   I2:            IF   ID   EX   MEM  WB                          │ │
│   │                            ↑↓                                     │ │
│   │   I3:                 IF   ID   EX   MEM  WB                     │ │
│   │                                 ↑                                 │ │
│   │                                                                   │ │
│   │   At cycle 4 (I2 in EX):                                        │ │
│   │   • I2 needs R1 from I1                                         │ │
│   │   • Forward from EX/MEM (I1's result)                           │ │
│   │   • ForwardA = 10                                                │ │
│   │                                                                   │ │
│   │   At cycle 5 (I3 in EX):                                        │ │
│   │   • I3 needs R1 from I2                                         │ │
│   │   • I1 is in MEM/WB, I2 is in EX/MEM                            │ │
│   │   • Both have R1 as destination!                                │ │
│   │   • Must forward from EX/MEM (I2's result) - most recent        │ │
│   │   • ForwardA = 10 (not 01)                                       │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   This is why EX/MEM takes priority over MEM/WB!                       │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Load-Use Hazard (Forwarding Limitation)

### 📌 When Forwarding Isn't Enough

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    LOAD-USE HAZARD: STALL STILL NEEDED                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   Code:                                                                 │
│   LW  R1, 0(R2)          ; Load R1 from memory                          │
│   ADD R3, R1, R4         ; Use R1 immediately                           │
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   Cycle:    1    2    3    4    5    6                           │ │
│   │                                                                   │ │
│   │   LW:       IF   ID   EX   MEM  WB                               │ │
│   │                             ↓                                     │ │
│   │                         Data available                           │ │
│   │                         (end of MEM)                             │ │
│   │                                                                   │ │
│   │   ADD:           IF   ID   EX   MEM  WB                          │ │
│   │                            ↑                                      │ │
│   │                       Need R1 here                               │ │
│   │                       (start of EX)                              │ │
│   │                                                                   │ │
│   │   Problem: ADD needs R1 at START of cycle 4                      │ │
│   │            LW produces R1 at END of cycle 4                      │ │
│   │            Time travel not possible!                             │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Solution: 1 Stall Cycle + Forward from MEM/WB                         │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   Cycle:    1    2    3    4    5    6    7                      │ │
│   │                                                                   │ │
│   │   LW:       IF   ID   EX   MEM  WB                               │ │
│   │                             ↓                                     │ │
│   │                             └──────┐                              │ │
│   │                                    │                              │ │
│   │   ADD:           IF   ID  STALL  EX   MEM  WB                    │ │
│   │                              │    ↑                               │ │
│   │                         bubble  Forward from MEM/WB              │ │
│   │                                                                   │ │
│   │   After 1-cycle stall, LW result is in MEM/WB                   │ │
│   │   ADD can get it via forwarding                                 │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Load-use penalty: 1 cycle (even with forwarding)                     │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 📌 Load-Use Stall Detection

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    HAZARD DETECTION UNIT                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   Load-use hazard detection (in ID stage):                              │
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   Stall = ID/EX.MemRead AND                                      │ │
│   │           ((ID/EX.Rt == IF/ID.Rs) OR                             │ │
│   │            (ID/EX.Rt == IF/ID.Rt))                               │ │
│   │                                                                   │ │
│   │   Where:                                                          │ │
│   │   • ID/EX.MemRead: Previous instruction is a load               │ │
│   │   • ID/EX.Rt: Destination register of the load (Rt for LW)      │ │
│   │   • IF/ID.Rs, IF/ID.Rt: Source registers of current instruction │ │
│   │                                                                   │ │
│   │   When Stall = 1:                                                │ │
│   │   1. PCWrite = 0        (freeze PC)                              │ │
│   │   2. IF/IDWrite = 0     (freeze IF/ID)                           │ │
│   │   3. Clear ID/EX controls (insert bubble/NOP)                    │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Hazard Detection vs Forwarding Unit:                                  │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   Hazard Detection Unit:                                         │ │
│   │   • Located in ID stage                                          │ │
│   │   • Detects load-use hazards                                    │ │
│   │   • Controls stalling (PCWrite, IF/IDWrite)                      │ │
│   │   • Inserts bubbles                                              │ │
│   │                                                                   │ │
│   │   Forwarding Unit:                                               │ │
│   │   • Located in EX stage                                          │ │
│   │   • Detects RAW hazards (non-load)                              │ │
│   │   • Controls forwarding MUXes                                    │ │
│   │   • No stalling - only data routing                             │ │
│   │                                                                   │ │
│   │   Both work together for correct execution                      │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Forwarding Summary

### 📌 Complete Forwarding Scenarios

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    ALL FORWARDING SCENARIOS                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   Distance │ Producer   │ Consumer  │ Source    │ Stall?         │ │
│   │   (cycles) │ Stage      │ Stage     │           │                │ │
│   │   ─────────┼────────────┼───────────┼───────────┼───────────────  │ │
│   │      1     │ ALU (MEM)  │ ALU (EX)  │ EX/MEM    │ No             │ │
│   │      2     │ ALU (WB)   │ ALU (EX)  │ MEM/WB    │ No             │ │
│   │      1     │ Load (MEM) │ ALU (EX)  │ Cannot!   │ Yes (1 cycle)  │ │
│   │      2     │ Load (WB)  │ ALU (EX)  │ MEM/WB    │ No             │ │
│   │     ≥3     │ Any (past) │ ALU (EX)  │ Reg file  │ No             │ │
│   │                                                                   │ │
│   │   Key: Load followed immediately by use = STALL                  │ │
│   │        All other RAW hazards = FORWARD (no stall)                │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Forwarding in Different Architectures:                               │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   • MIPS: Full forwarding from EX/MEM and MEM/WB                │ │
│   │   • ARM: Similar forwarding with additional paths for Thumb     │ │
│   │   • x86: Complex out-of-order, many bypass paths                │ │
│   │   • RISC-V: Standard 5-stage forwarding model                   │ │
│   │                                                                   │ │
│   │   Modern high-performance CPUs may have 10+ forwarding paths    │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Scenario | Producer | Consumer | Forward From | Stall Needed? |
|----------|----------|----------|--------------|---------------|
| ALU→ALU (dist 1) | EX→MEM | ID→EX | EX/MEM | No |
| ALU→ALU (dist 2) | MEM→WB | ID→EX | MEM/WB | No |
| Load→ALU (dist 1) | MEM | EX | N/A | Yes (1 cycle) |
| Load→ALU (dist 2) | WB | EX | MEM/WB | No |
| ALU→ALU (dist ≥3) | WB | ID | Register File | No |

| ForwardA/B | Meaning |
|------------|---------|
| 00 | Use register file value |
| 01 | Forward from MEM/WB |
| 10 | Forward from EX/MEM |

---

## ❓ Quick Revision Questions

1. **Q**: What is the purpose of data forwarding?
   
   **A**: To send computed results directly from where they are produced (pipeline registers) to where they are needed (ALU inputs), without waiting for the result to be written to and read from the register file.

2. **Q**: Why does EX/MEM forwarding take priority over MEM/WB?
   
   **A**: When the same register is written by consecutive instructions, the most recent value (closer instruction in EX/MEM) must be used, not the older value in MEM/WB.

3. **Q**: Can forwarding eliminate load-use hazards?
   
   **A**: No. Load produces data at the end of MEM stage, but the next instruction needs it at the start of EX. This one-cycle gap requires a stall. After the stall, forwarding from MEM/WB provides the data.

4. **Q**: What conditions trigger forwarding from EX/MEM?
   
   **A**: (1) EX/MEM.RegWrite = 1, (2) EX/MEM.Rd ≠ 0 (not R0), (3) EX/MEM.Rd matches the source register of instruction in EX stage.

5. **Q**: What is the difference between the Hazard Detection Unit and Forwarding Unit?
   
   **A**: Hazard Detection Unit detects load-use hazards and triggers stalls (operates in ID stage). Forwarding Unit detects forwarding opportunities and controls MUXes (operates in EX stage).

6. **Q**: How many stall cycles are needed for a RAW hazard between two ALU instructions?
   
   **A**: Zero stalls with forwarding. The result is forwarded from EX/MEM or MEM/WB directly to the ALU input.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Pipeline Hazards](02-pipeline-hazards.md) | [Unit 8 Home](README.md) | [Branch Handling](04-branch-handling.md) |

---

[← Previous Chapter](02-pipeline-hazards.md) | [Course Home](../README.md) | [Next Chapter →](04-branch-handling.md)
