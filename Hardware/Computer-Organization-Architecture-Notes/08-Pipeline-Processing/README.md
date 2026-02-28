# Unit 8: Pipeline Processing

## 📋 Unit Overview

Pipelining is a fundamental technique for improving CPU performance by overlapping instruction execution. This unit covers the principles of pipelining, hazards that arise, and solutions to maintain correctness while maximizing throughput.

---

## 🎯 Unit Objectives

- Understand pipeline concepts and terminology
- Study instruction pipeline stages
- Analyze pipeline hazards and their solutions
- Learn branch prediction techniques

---

## 📊 Pipeline Concept

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    SEQUENTIAL vs PIPELINED EXECUTION                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   SEQUENTIAL EXECUTION (Non-pipelined):                                 │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   Time:   1   2   3   4   5   6   7   8   9  10  11  12          │ │
│   │          ┌───────────────────┐                                    │ │
│   │   I1:    │ IF  ID  EX  MEM WB│                                    │ │
│   │          └───────────────────┘                                    │ │
│   │                              ┌───────────────────┐                │ │
│   │   I2:                        │ IF  ID  EX  MEM WB│                │ │
│   │                              └───────────────────┘                │ │
│   │                                                  ┌────────────────│ │
│   │   I3:                                            │ IF  ID  EX ... │ │
│   │                                                  └────────────────│ │
│   │                                                                   │ │
│   │   3 instructions take 15 cycles (5 cycles each)                  │ │
│   │   CPI = 5 (one instruction completes every 5 cycles)             │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   PIPELINED EXECUTION:                                                  │
│   ┌───────────────────────────────────────────────────────────────────┐ │
│   │                                                                   │ │
│   │   Time:   1   2   3   4   5   6   7                              │ │
│   │          ┌───┬───┬───┬───┬───┐                                   │ │
│   │   I1:    │IF │ID │EX │MEM│WB │                                   │ │
│   │          └───┼───┼───┼───┼───┘                                   │ │
│   │              │   │   │   │                                        │ │
│   │          ┌───┼───┼───┼───┼───┐                                   │ │
│   │   I2:        │IF │ID │EX │MEM│WB │                               │ │
│   │              └───┼───┼───┼───┼───┘                               │ │
│   │                  │   │   │   │                                    │ │
│   │          ┌───────┼───┼───┼───┼───┐                               │ │
│   │   I3:            │IF │ID │EX │MEM│WB │                           │ │
│   │                  └───┴───┴───┴───┴───┘                           │ │
│   │                                                                   │ │
│   │   3 instructions take 7 cycles (after pipeline fills)           │ │
│   │   Steady-state: One instruction completes per cycle (CPI ≈ 1)   │ │
│   │                                                                   │ │
│   └───────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│   Speedup (ideal) = Number of pipeline stages = 5×                      │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📚 Chapters in This Unit

| # | Chapter | Topics |
|---|---------|--------|
| 1 | [Instruction Pipeline](01-instruction-pipeline.md) | Pipeline stages, timing, throughput |
| 2 | [Pipeline Hazards](02-pipeline-hazards.md) | Structural, data, control hazards |
| 3 | [Data Forwarding](03-data-forwarding.md) | Bypass paths, forwarding logic |
| 4 | [Branch Handling](04-branch-handling.md) | Branch prediction, delay slots |

---

## 🔑 Key Terminology

| Term | Definition |
|------|------------|
| **Pipeline Stage** | One step in the pipeline; each handles a different part of instruction |
| **Pipeline Register** | Storage between stages holding intermediate results |
| **Throughput** | Number of instructions completed per unit time |
| **Latency** | Time for one instruction to complete (fill to drain) |
| **Hazard** | Condition that prevents next instruction from executing |
| **Stall** | Pipeline bubble inserted to resolve hazard |
| **Forwarding** | Bypassing data to avoid stalls |
| **Branch Prediction** | Guessing branch outcome before it's known |

---

## 📊 Comparison: Sequential vs Pipelined

| Aspect | Sequential | Pipelined |
|--------|-----------|-----------|
| **CPI (ideal)** | n (stages) | 1 |
| **Clock period** | Short (one operation) | Equal (slowest stage) |
| **Hardware** | Simple | Pipeline registers, hazard logic |
| **Throughput** | Low | High (n× improvement) |
| **Latency per instruction** | Lower | Higher (overhead) |
| **Complexity** | Low | High (hazard handling) |

---

## 🧭 Unit Navigation

| Previous Unit | Current | Next Unit |
|--------------|---------|-----------|
| [Microprogrammed Control](../07-Microprogrammed-Control/README.md) | **Unit 8** | [Memory Organization](../09-Memory-Organization/README.md) |

---

[← Previous Unit](../07-Microprogrammed-Control/README.md) | [Course Home](../README.md) | [First Chapter →](01-instruction-pipeline.md)
