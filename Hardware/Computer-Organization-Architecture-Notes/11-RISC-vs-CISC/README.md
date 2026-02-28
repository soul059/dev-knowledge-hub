# Unit 11: RISC vs CISC Architectures

## 📋 Unit Overview

This unit explores the two fundamental approaches to processor design: Reduced Instruction Set Computing (RISC) and Complex Instruction Set Computing (CISC). Understanding these philosophies helps explain modern processor evolution and design trade-offs.

---

## 🗂️ Unit Contents

| Chapter | Topic | Key Concepts |
|---------|-------|--------------|
| 11.1 | [RISC Architecture](01-risc-architecture.md) | RISC principles, load-store, fixed format |
| 11.2 | [CISC Architecture](02-cisc-architecture.md) | CISC principles, complex addressing, microcode |
| 11.3 | [Comparison and Modern Trends](03-comparison-and-trends.md) | Design trade-offs, hybrid approaches, evolution |

---

## 🎯 Learning Objectives

After completing this unit, you will be able to:

1. Explain the core principles of RISC architecture
2. Describe CISC design philosophy and characteristics
3. Compare instruction formats and addressing modes
4. Analyze the performance implications of each approach
5. Understand how modern processors blend both philosophies

---

## 📊 Architecture Philosophy Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    RISC vs CISC PHILOSOPHY                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   Design Philosophy:                                                    │
│                                                                          │
│   CISC (1960s-1980s)                    RISC (1980s-present)           │
│   ┌───────────────────────┐             ┌───────────────────────┐      │
│   │                       │             │                       │      │
│   │   "Make hardware      │             │   "Keep hardware      │      │
│   │    do the work"       │             │    simple, let        │      │
│   │                       │             │    compiler optimize" │      │
│   │   • Close the         │             │                       │      │
│   │     semantic gap      │             │   • Simple ops        │      │
│   │   • Support HLLs      │             │     execute fast      │      │
│   │     directly          │             │   • Compiler does     │      │
│   │   • Reduce program    │             │     heavy lifting     │      │
│   │     size              │             │   • Pipeline          │      │
│   │                       │             │     efficiency        │      │
│   │                       │             │                       │      │
│   └───────────────────────┘             └───────────────────────┘      │
│                                                                          │
│   Key Metrics Comparison:                                               │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   Metric              │ CISC           │ RISC                    │ │
│   │   ────────────────────┼────────────────┼─────────────────────────│ │
│   │   Instructions        │ 200-300+       │ 50-150                  │ │
│   │   Instruction length  │ Variable       │ Fixed (32-bit)          │ │
│   │   Addressing modes    │ 12-24          │ 3-5                     │ │
│   │   Registers          │ 8-16           │ 32-128+                 │ │
│   │   Cycles/instruction  │ Variable (1-20)│ Mostly 1                │ │
│   │   Memory access       │ Any instruction│ Load/Store only         │ │
│   │   Control unit        │ Microprogrammed│ Hardwired               │ │
│   │   Code density        │ Higher         │ Lower                   │ │
│   │   Pipeline friendly   │ Difficult      │ Designed for it         │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Examples:                                                             │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   CISC:  x86/x64 (Intel, AMD)                                    │ │
│   │          VAX, Motorola 68000                                     │ │
│   │                                                                   │ │
│   │   RISC:  ARM, MIPS, RISC-V                                       │ │
│   │          PowerPC, SPARC                                          │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📐 Key Formulas

**Performance Equation:**
$$\text{CPU Time} = \text{IC} \times \text{CPI} \times \text{T}$$

Where:
- IC = Instruction Count (program size)
- CPI = Cycles Per Instruction
- T = Clock cycle time

**RISC Strategy:** Minimize CPI × T (simple, fast instructions)
**CISC Strategy:** Minimize IC (powerful instructions, fewer needed)

---

## 🔗 Prerequisites

- Understanding of basic CPU organization
- Familiarity with instruction sets and addressing modes
- Knowledge of pipelining concepts (Unit 8)

---

## 🧭 Navigation

| Previous Unit | Course Home | Next Unit |
|---------------|-------------|-----------|
| [Unit 10: I/O Organization](../10-IO-Organization/README.md) | [Course Home](../README.md) | - |

---

[← Previous Unit](../10-IO-Organization/README.md) | [Course Home](../README.md) | [Start Chapter 11.1 →](01-risc-architecture.md)
