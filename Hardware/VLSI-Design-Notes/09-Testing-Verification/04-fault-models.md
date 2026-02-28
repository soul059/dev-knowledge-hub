# Chapter 9.4: Fault Models

## 📋 Chapter Overview

**Fault models** are abstractions of physical defects that enable practical test generation. This chapter covers stuck-at faults, transition faults, path delay faults, and their relationship to real manufacturing defects.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Understand the fault modeling hierarchy
- Apply stuck-at fault analysis
- Distinguish between different fault types
- Relate physical defects to fault models

---

## 9.4.1 Fault Modeling Hierarchy

```
┌─────────────────────────────────────────────────────────────────────┐
│                    FROM DEFECTS TO FAULTS                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   PHYSICAL DEFECTS (real world):                                   │
│   ─────────────────────────────                                     │
│   • Particle contamination                                         │
│   • Oxide pinholes                                                 │
│   • Metal shorts/opens                                             │
│   • Lithography errors                                             │
│   • Implant variations                                             │
│                                                                      │
│           ▼ Abstraction                                             │
│                                                                      │
│   CIRCUIT-LEVEL FAULTS:                                            │
│   ─────────────────────                                             │
│   • Transistor stuck-on/stuck-off                                  │
│   • Bridging faults (two nodes shorted)                           │
│   • Open faults (broken connection)                               │
│   • Resistive faults (partial contact)                            │
│                                                                      │
│           ▼ Abstraction                                             │
│                                                                      │
│   GATE-LEVEL FAULTS (most common):                                 │
│   ────────────────────────────────                                  │
│   • Stuck-at faults (SA0, SA1)                                    │
│   • Transition faults (slow-to-rise, slow-to-fall)               │
│   • Path delay faults                                              │
│                                                                      │
│           ▼ Abstraction                                             │
│                                                                      │
│   BEHAVIORAL FAULTS:                                                │
│   ──────────────────                                                │
│   • Functional errors                                              │
│   • State machine errors                                           │
│                                                                      │
│                                                                      │
│   Why use abstract fault models?                                   │
│   ┌──────────────────────────────────────────────────────────────┐  │
│   │ • Physical defects → infinite variety                        │  │
│   │ • Transistor faults → millions per chip                     │  │
│   │ • Gate-level faults → manageable number (2N for N lines)   │  │
│   │ • Practical for ATPG and fault simulation                   │  │
│   └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 9.4.2 Stuck-At Fault Model

```
┌─────────────────────────────────────────────────────────────────────┐
│                    STUCK-AT FAULTS                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Assumption: A signal line is permanently stuck at logic 0 or 1  │
│                                                                      │
│   Notation:                                                         │
│   • SA0: Stuck-at-0 (line always 0)                               │
│   • SA1: Stuck-at-1 (line always 1)                               │
│   • Line/SA0 or Line/SA1                                          │
│                                                                      │
│                                                                      │
│   Example circuit:                                                  │
│                                                                      │
│   A ────┬───────────────►│    │                                     │
│   (SA0) │                │AND ├──► C ──►│    │                     │
│         │                │    │         │ OR ├──► F                │
│   B ────┴───────────────►│    │         │    │                     │
│                                    D ──►│    │                     │
│                                                                      │
│   Possible faults: A/SA0, A/SA1, B/SA0, B/SA1, C/SA0, C/SA1,      │
│                   D/SA0, D/SA1, F/SA0, F/SA1                       │
│                                                                      │
│   Total: 2 × (number of signal lines)                              │
│                                                                      │
│                                                                      │
│   Fault detection (example: detect A/SA0):                         │
│   ───────────────────────────────────────                           │
│                                                                      │
│   Step 1: ACTIVATE - Set A to opposite of stuck value             │
│           A = 1 (fault makes it 0)                                 │
│                                                                      │
│   Step 2: PROPAGATE - Create path to output                        │
│           B = 1 (so AND gate propagates A's value)                │
│           D = 0 (so OR gate propagates C's value)                 │
│                                                                      │
│   Good circuit: F = (1 AND 1) OR 0 = 1                             │
│   Faulty circuit: F = (0 AND 1) OR 0 = 0                           │
│                                                                      │
│   Test vector: (A=1, B=1, D=0) detects A/SA0                      │
│                                                                      │
│                                                                      │
│   Fault collapsing (reduces fault list):                           │
│   ──────────────────────────────────────                            │
│                                                                      │
│   A ──►│    │                                                       │
│        │AND ├──► C                                                  │
│   B ──►│    │                                                       │
│                                                                      │
│   Equivalent faults: A/SA0 ≡ B/SA0 ≡ C/SA0                        │
│   (Any one detected → all detected)                                │
│                                                                      │
│   Collapse ratio: typically 2:1 to 4:1                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 9.4.3 Transition Fault Model

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TRANSITION FAULTS                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Models delay defects: circuit works correctly but too slowly     │
│                                                                      │
│   Types:                                                            │
│   • Slow-to-Rise (STR): Node is slow transitioning 0→1            │
│   • Slow-to-Fall (STF): Node is slow transitioning 1→0            │
│                                                                      │
│                                                                      │
│   Timing diagram (slow-to-rise fault):                             │
│                                                                      │
│   Input                                                             │
│         ┌───────────                                                │
│         │                                                           │
│   ──────┘                                                           │
│                                                                      │
│   Good output                                                       │
│         ┌───────────                                                │
│         │                                                           │
│   ──────┘                                                           │
│                                                                      │
│   Faulty output (STR)                                              │
│              ┌──────                                                │
│             ╱                                                       │
│   ────────╱                                                         │
│           ↑                                                         │
│       Slow rise                                                     │
│                                                                      │
│   CLK sample point                                                  │
│         ↓                                                           │
│         │                                                           │
│   Good: samples 1 ✓                                                │
│   Faulty: samples 0 ✗                                              │
│                                                                      │
│                                                                      │
│   Detection requirements:                                           │
│   ────────────────────                                              │
│   1. Initialize node to opposite value                             │
│   2. Launch transition (0→1 for STR, 1→0 for STF)                 │
│   3. Capture at-speed (one clock period later)                    │
│                                                                      │
│   Two-pattern test (V1, V2):                                       │
│   V1 = initialization vector                                       │
│   V2 = launch vector (transition occurs)                           │
│                                                                      │
│   Example:                                                          │
│   ┌──────────────────────────────────────────────────────────────┐  │
│   │ To detect line X / STR:                                      │  │
│   │                                                              │  │
│   │ V1: Set X = 0, propagation path sensitized                  │  │
│   │ V2: Set X = 1, same path sensitized                         │  │
│   │                                                              │  │
│   │ Apply V1, wait, apply V2 with at-speed clock                │  │
│   │ If X is slow: output incorrect at capture                   │  │
│   └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│                                                                      │
│   Transition fault count: Same as stuck-at (2 per line)           │
│   But tests are more complex (two-vector pairs)                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 9.4.4 Path Delay Fault Model

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PATH DELAY FAULTS                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Models cumulative delay along a complete path                    │
│                                                                      │
│   Why path delay (vs transition):                                  │
│   • Transition: tests single gate delay                           │
│   • Path delay: tests end-to-end timing                           │
│   • Small delays can accumulate to cause failure                  │
│                                                                      │
│                                                                      │
│   Path definition:                                                  │
│                                                                      │
│   A ──►│G1├──►│G2├──►│G3├──►│G4├──► Output                         │
│                                                                      │
│        │ Path P1: A → G1 → G2 → G3 → G4 → Out │                    │
│                                                                      │
│   Total path delay = Σ (gate delays + wire delays)                 │
│                                                                      │
│                                                                      │
│   Robust vs Non-robust tests:                                       │
│   ───────────────────────────                                       │
│                                                                      │
│   A ──►│    │                                                       │
│        │AND ├──►│    │                                              │
│   B ──►│    │   │AND ├──► Out                                       │
│             C ──►│    │                                              │
│                                                                      │
│   Path: A → AND1 → AND2 → Out                                      │
│                                                                      │
│   ROBUST test: Detects fault regardless of other path delays      │
│   • Side inputs must be at non-controlling value                   │
│   • B=1 (stable), C=1 (stable)                                    │
│   • A transitions 0→1 or 1→0                                      │
│                                                                      │
│   NON-ROBUST test: May not detect if other paths also slow        │
│   • Less restrictive side input requirements                       │
│   • Higher coverage but less reliable                              │
│                                                                      │
│                                                                      │
│   Path explosion problem:                                           │
│   ──────────────────────                                            │
│                                                                      │
│   Number of paths can be exponential!                              │
│                                                                      │
│        ┌─►○──┐    ┌─►○──┐    ┌─►○──┐                               │
│   IN ──┤     ├────┤     ├────┤     ├──► OUT                        │
│        └─►○──┘    └─►○──┘    └─►○──┘                               │
│                                                                      │
│   n stages, 2 paths each: Total = 2^n paths                        │
│   10 stages = 1024 paths                                           │
│   32 stages = 4 billion paths!                                     │
│                                                                      │
│   Solution: Test only critical/longest paths                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 9.4.5 Other Fault Models

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ADDITIONAL FAULT MODELS                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   1. BRIDGING FAULTS                                               │
│   ──────────────────                                                │
│   Two signal lines shorted together                                │
│                                                                      │
│   A ──────┬──────────► (intended A)                                │
│           ╳ Bridge                                                  │
│   B ──────┴──────────► (intended B)                                │
│                                                                      │
│   Result depends on driver strengths:                              │
│   • Wired-AND: Output = A AND B                                   │
│   • Wired-OR: Output = A OR B                                     │
│   • Dominant: Stronger driver wins                                 │
│                                                                      │
│   When A=0, B=1 (or vice versa):                                  │
│   • Contention → intermediate voltage                             │
│   • May cause IDDQ (elevated current)                             │
│                                                                      │
│                                                                      │
│   2. OPEN FAULTS                                                   │
│   ──────────────                                                    │
│   Signal line broken (high impedance)                              │
│                                                                      │
│   Driver ────╳────○──── Receiver                                   │
│              Open                                                   │
│                                                                      │
│   Result:                                                           │
│   • CMOS: Receiver may float to previous state (sequential!)      │
│   • Or may settle to random value                                 │
│   • Hard to detect with stuck-at model                            │
│                                                                      │
│                                                                      │
│   3. IDDQ FAULTS                                                   │
│   ──────────────                                                    │
│   Measure quiescent (static) supply current                        │
│                                                                      │
│   Normal CMOS: IDDQ ≈ nanoamps (leakage only)                     │
│   Faulty CMOS: IDDQ ≈ microamps-milliamps                         │
│                                                                      │
│   VDD ────┬──────── VDD                                            │
│           │                                                         │
│          ═╪═ PMOS                                                   │
│           ├─────────► Out                                          │
│          ═╪═ NMOS (stuck-on!)                                      │
│           │                                                         │
│   GND ────┴──────── GND                                            │
│           │                                                         │
│           └──► DC path from VDD to GND                             │
│                                                                      │
│   IDDQ testing detects many physical defects!                      │
│                                                                      │
│                                                                      │
│   4. CELL-AWARE FAULTS                                             │
│   ────────────────────                                              │
│   Based on actual transistor-level defects in standard cells      │
│                                                                      │
│   Library cell analyzed for all possible:                          │
│   • Transistor stuck-on/stuck-off                                 │
│   • Internal bridging                                              │
│   • Internal opens                                                 │
│                                                                      │
│   Provides better defect coverage than gate-level stuck-at        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 9.4.6 Fault Coverage Metrics

```
┌─────────────────────────────────────────────────────────────────────┐
│                    COVERAGE CALCULATIONS                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Basic definitions:                                                │
│                                                                      │
│   $\text{Fault Coverage} = \frac{\text{Detected Faults}}{\text{Total Faults}} × 100\%$│
│                                                                      │
│   $\text{Test Efficiency} = \frac{\text{Detected Faults}}{\text{Detectable Faults}} × 100\%$│
│                                                                      │
│   Some faults are undetectable:                                    │
│   • Redundant logic (fault has no effect on output)              │
│   • Untestable (cannot satisfy detection requirements)            │
│                                                                      │
│                                                                      │
│   Example calculation:                                              │
│   ┌──────────────────────────────────────────────────────────────┐  │
│   │ Total faults in design:      10,000                          │  │
│   │ Detected by test set:         9,500                          │  │
│   │ Undetectable (redundant):       200                          │  │
│   │ Undetected (test missing):      300                          │  │
│   │                                                              │  │
│   │ Fault Coverage = 9500/10000 = 95.0%                         │  │
│   │ Test Efficiency = 9500/9800 = 96.9%                         │  │
│   └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│                                                                      │
│   Industry targets:                                                  │
│   ┌──────────────────────────────────────────────────────────────┐  │
│   │ Application          │ Stuck-at   │ Transition             │  │
│   ├──────────────────────────────────────────────────────────────┤  │
│   │ Consumer electronics │ >95%       │ >90%                    │  │
│   │ Industrial          │ >98%       │ >95%                    │  │
│   │ Automotive          │ >99%       │ >98%                    │  │
│   │ Medical/Aerospace   │ >99.5%     │ >99%                    │  │
│   └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│                                                                      │
│   Relationship to defect level:                                     │
│                                                                      │
│   Higher coverage → Lower defect level (escapes)                   │
│                                                                      │
│   At 90% yield:                                                     │
│   • 95% coverage → 0.5% defect level                              │
│   • 99% coverage → 0.1% defect level                              │
│   • 99.9% coverage → 0.01% defect level                           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Fault Model | What it Models | Test Type | Complexity |
|-------------|----------------|-----------|------------|
| Stuck-at | Permanent logic errors | Single vector | Low |
| Transition | Gate delay defects | Two-vector pair | Medium |
| Path delay | Cumulative path delay | Two-vector, robust | High |
| Bridging | Signal shorts | Pattern dependent | Medium |
| IDDQ | Current-based defects | Current measurement | Low |

---

## ❓ Quick Revision Questions

1. **Why do we use abstract fault models instead of testing for physical defects?**

2. **How many stuck-at faults exist in a circuit with 100 signal lines?**

3. **What is fault equivalence and how does it reduce the fault list?**

4. **Explain the difference between stuck-at and transition fault testing.**

5. **What is the path explosion problem in path delay testing?**

6. **Why is IDDQ testing effective for detecting bridging faults?**

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| ← [Built-In Self-Test](03-built-in-self-test.md) | [Unit 9 Home](README.md) | [ATPG and Coverage →](05-atpg-coverage.md) |
