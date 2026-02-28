# Unit 9: Testing and Verification

## 📋 Unit Overview

**Testing and verification** ensure that fabricated VLSI chips function correctly. This unit covers Design for Testability (DFT), scan-based testing, Built-In Self-Test (BIST), and fault modeling approaches.

---

## 🎯 Unit Learning Objectives

After completing this unit, you will be able to:
- Understand the importance and economics of VLSI testing
- Apply DFT techniques to improve testability
- Design scan chains for structural testing
- Implement memory and logic BIST
- Analyze fault models and test coverage

---

## 📊 Testing Landscape

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TESTING IN IC LIFECYCLE                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Design ──► Fabrication ──► Testing ──► Packaging ──► System      │
│     │            │             │            │            │          │
│     │            │             ▼            │            │          │
│     │            │     ┌──────────────┐     │            │          │
│     │            │     │ Wafer Test   │     │            │          │
│     │            │     │ (Die Sort)   │     │            │          │
│     │            │     └──────┬───────┘     │            │          │
│     │            │            │             ▼            │          │
│     │            │            │     ┌──────────────┐     │          │
│     │            │            │     │ Package Test │     │          │
│     │            │            │     │ (Final Test) │     │          │
│     │            │            │     └──────┬───────┘     │          │
│     │            │            │            │             ▼          │
│     │            │            │            │     ┌──────────────┐   │
│     │            │            │            │     │ System Test  │   │
│     │            │            │            │     │ (Board Level)│   │
│     │            │            │            │     └──────────────┘   │
│     │            │            │            │                        │
│   Simulation  Defects      Screen      Binning    In-field         │
│   Emulation   Particles   Defective   by Speed   Diagnostics       │
│   Formal      Shorts      Dies        Grade                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📈 Test Economics

```
┌─────────────────────────────────────────────────────────────────────┐
│                    COST OF TESTING                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Rule of Ten:                                                      │
│   Cost to detect defect multiplies by ~10× at each level           │
│                                                                      │
│   Stage               │ Relative Cost                              │
│   ────────────────────┼───────────────                              │
│   Wafer test          │ $0.01 - $0.10                              │
│   Package test        │ $0.10 - $1.00                              │
│   Board test          │ $1 - $10                                   │
│   System test         │ $10 - $100                                 │
│   Field failure       │ $100 - $1000+                              │
│                                                                      │
│                                                                      │
│   Key metrics:                                                      │
│                                                                      │
│   Yield = (Good dies / Total dies) × 100%                          │
│                                                                      │
│   Defect Level = Faulty chips that pass test                       │
│                  ─────────────────────────                          │
│                   Total chips shipped                               │
│                                                                      │
│   Target: < 1 ppm (parts per million) for automotive               │
│           < 100 ppm for consumer electronics                        │
│                                                                      │
│                                                                      │
│   Test coverage vs defect level:                                    │
│                                                                      │
│   $DL = (1 - Y)^{(1-FC)}$                                          │
│                                                                      │
│   DL = defect level (escapes)                                      │
│   Y  = yield                                                        │
│   FC = fault coverage                                               │
│                                                                      │
│   Example: 90% yield, 99% fault coverage                           │
│   DL = (1-0.9)^(1-0.99) = 0.1^0.01 = 0.977                         │
│   Still 2.3% escapes! Need higher coverage.                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📚 Chapter Index

| Chapter | Topic | Key Concepts |
|---------|-------|--------------|
| 9.1 | [Design for Testability](01-design-for-testability.md) | Controllability, observability, testability rules |
| 9.2 | [Scan Design](02-scan-design.md) | Scan chains, mux-D scan, compression |
| 9.3 | [Built-In Self-Test](03-built-in-self-test.md) | LFSR, MISR, logic BIST, memory BIST |
| 9.4 | [Fault Models](04-fault-models.md) | Stuck-at, transition, path delay faults |
| 9.5 | [ATPG and Coverage](05-atpg-coverage.md) | Automatic test pattern generation |

---

## 🔑 Key Testing Terminology

| Term | Definition |
|------|------------|
| DFT | Design for Testability - techniques to improve test access |
| ATPG | Automatic Test Pattern Generation |
| BIST | Built-In Self-Test |
| Stuck-at fault | Modeled as line permanently at 0 or 1 |
| Fault coverage | Percentage of faults detected by test set |
| Test escape | Faulty chip that passes all tests |

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| ← [Unit 8: Power Optimization](../08-Power-Optimization/README.md) | [Course Home](../README.md) | [Design for Testability →](01-design-for-testability.md) |
