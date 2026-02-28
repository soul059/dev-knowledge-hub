# Computer Organization and Architecture

## 📚 Complete Study Notes

✅ **COURSE COMPLETE** - All 11 Units Fully Documented

A comprehensive guide to understanding how computers work at the hardware and architectural level. These notes cover everything from basic computer organization to advanced topics like pipelining, memory hierarchy, and RISC vs CISC architectures.

---

## 🎯 Course Overview

**Computer Organization** deals with the structural relationships that are not visible to the programmer, such as interfaces to peripheral devices, clock frequency, and the technology used for memory.

**Computer Architecture** refers to those attributes of a system visible to a programmer, including instruction sets, data types, I/O mechanisms, and memory addressing techniques.

### Prerequisites
- Digital Electronics fundamentals
- Basic understanding of number systems
- Logic gates and Boolean algebra

### Learning Objectives
Upon completing these notes, you will be able to:
- Understand computer system organization and architecture
- Analyze and design basic computer components
- Explain data representation and computer arithmetic
- Understand CPU organization and control unit design
- Analyze memory hierarchy and I/O systems
- Compare RISC and CISC architectures

---

## 📋 Complete Table of Contents

### [Unit 1: Basic Computer Organization](01-Basic-Computer-Organization/README.md)
| Chapter | Topic | Description |
|---------|-------|-------------|
| 1.1 | [Von Neumann vs Harvard Architecture](01-Basic-Computer-Organization/01-von-neumann-harvard-architecture.md) | Fundamental computer architectures |
| 1.2 | [CPU, Memory, I/O Organization](01-Basic-Computer-Organization/02-cpu-memory-io-organization.md) | Core components of a computer |
| 1.3 | [Bus Structures](01-Basic-Computer-Organization/03-bus-structures.md) | Address, data, and control buses |
| 1.4 | [System Interconnection](01-Basic-Computer-Organization/04-system-interconnection.md) | How components communicate |

### [Unit 2: Data Representation](02-Data-Representation/README.md)
| Chapter | Topic | Description |
|---------|-------|-------------|
| 2.1 | [Fixed Point Representation](02-Data-Representation/01-fixed-point-representation.md) | Integer and fractional representation |
| 2.2 | [Floating Point Representation](02-Data-Representation/02-floating-point-representation.md) | Real number representation |
| 2.3 | [IEEE 754 Standard](02-Data-Representation/03-ieee-754-standard.md) | Industry standard for floating point |
| 2.4 | [Character Representation](02-Data-Representation/04-character-representation.md) | ASCII, Unicode, and text encoding |

### [Unit 3: Computer Arithmetic](03-Computer-Arithmetic/README.md)
| Chapter | Topic | Description |
|---------|-------|-------------|
| 3.1 | [Integer Addition and Subtraction](03-Computer-Arithmetic/01-integer-addition-subtraction.md) | Basic arithmetic operations |
| 3.2 | [Integer Multiplication](03-Computer-Arithmetic/02-integer-multiplication.md) | Hardware multiplication methods |
| 3.3 | [Booth's Algorithm](03-Computer-Arithmetic/03-booths-algorithm.md) | Efficient signed multiplication |
| 3.4 | [Integer Division](03-Computer-Arithmetic/04-integer-division.md) | Restoring and non-restoring division |
| 3.5 | [Floating Point Arithmetic](03-Computer-Arithmetic/05-floating-point-arithmetic.md) | Operations on real numbers |

### [Unit 4: Register Transfer and Microoperations](04-Register-Transfer-Microoperations/README.md)
| Chapter | Topic | Description |
|---------|-------|-------------|
| 4.1 | [Register Transfer Language](04-Register-Transfer-Microoperations/01-register-transfer-language.md) | Symbolic notation for transfers |
| 4.2 | [Arithmetic Microoperations](04-Register-Transfer-Microoperations/02-arithmetic-microoperations.md) | Add, subtract, increment operations |
| 4.3 | [Logic Microoperations](04-Register-Transfer-Microoperations/03-logic-microoperations.md) | AND, OR, XOR operations |
| 4.4 | [Shift Microoperations](04-Register-Transfer-Microoperations/04-shift-microoperations.md) | Logical, arithmetic, circular shifts |

### [Unit 5: Basic Computer Design](05-Basic-Computer-Design/README.md)
| Chapter | Topic | Description |
|---------|-------|-------------|
| 5.1 | [Instruction Codes](05-Basic-Computer-Design/01-instruction-codes.md) | Operation codes and addressing |
| 5.2 | [Computer Registers](05-Basic-Computer-Design/02-computer-registers.md) | Types and functions of registers |
| 5.3 | [Computer Instructions](05-Basic-Computer-Design/03-computer-instructions.md) | Instruction set design |
| 5.4 | [Timing and Control](05-Basic-Computer-Design/04-timing-and-control.md) | Control unit operation |
| 5.5 | [Instruction Cycle](05-Basic-Computer-Design/05-instruction-cycle.md) | Fetch-decode-execute cycle |

### [Unit 6: CPU Organization](06-CPU-Organization/README.md)
| Chapter | Topic | Description |
|---------|-------|-------------|
| 6.1 | [General Register Organization](06-CPU-Organization/01-general-register-organization.md) | Register file design |
| 6.2 | [Stack Organization](06-CPU-Organization/02-stack-organization.md) | Stack-based architectures |
| 6.3 | [Instruction Formats](06-CPU-Organization/03-instruction-formats.md) | Zero, one, two, three address formats |
| 6.4 | [Addressing Modes](06-CPU-Organization/04-addressing-modes.md) | Ways to specify operands |

### [Unit 7: Microprogrammed Control](07-Microprogrammed-Control/README.md)
| Chapter | Topic | Description |
|---------|-------|-------------|
| 7.1 | [Control Memory](07-Microprogrammed-Control/01-control-memory.md) | Storing microinstructions |
| 7.2 | [Microinstruction Format](07-Microprogrammed-Control/02-microinstruction-format.md) | Structure of microinstructions |
| 7.3 | [Microprogram Sequencer](07-Microprogrammed-Control/03-microprogram-sequencer.md) | Address generation logic |
| 7.4 | [Horizontal vs Vertical Microprogramming](07-Microprogrammed-Control/04-horizontal-vertical-microprogramming.md) | Design trade-offs |

### [Unit 8: Pipeline Processing](08-Pipeline-Processing/README.md)
| Chapter | Topic | Description |
|---------|-------|-------------|
| 8.1 | [Instruction Pipeline](08-Pipeline-Processing/01-instruction-pipeline.md) | Pipeline concepts and stages |
| 8.2 | [Pipeline Hazards](08-Pipeline-Processing/02-pipeline-hazards.md) | Structural, data, control hazards |
| 8.3 | [Hazard Resolution Techniques](08-Pipeline-Processing/03-hazard-resolution.md) | Forwarding, stalling, prediction |
| 8.4 | [Superscalar and VLIW](08-Pipeline-Processing/04-superscalar-vliw.md) | Advanced pipeline architectures |

### [Unit 9: Memory Organization](09-Memory-Organization/README.md)
| Chapter | Topic | Description |
|---------|-------|-------------|
| 9.1 | [Memory Hierarchy](09-Memory-Organization/01-memory-hierarchy.md) | Levels of memory |
| 9.2 | [Cache Memory](09-Memory-Organization/02-cache-memory.md) | Cache organization and policies |
| 9.3 | [Virtual Memory](09-Memory-Organization/03-virtual-memory.md) | Address translation and paging |
| 9.4 | [Memory Interleaving](09-Memory-Organization/04-memory-interleaving.md) | Parallel memory access |

### [Unit 10: I/O Organization](10-IO-Organization/README.md)
| Chapter | Topic | Description |
|---------|-------|-------------|
| 10.1 | [I/O Interface](10-IO-Organization/01-io-interface.md) | Peripheral interfacing, memory-mapped vs isolated I/O |
| 10.2 | [I/O Techniques](10-IO-Organization/02-io-techniques.md) | Programmed I/O, interrupt-driven I/O, polling |
| 10.3 | [Direct Memory Access](10-IO-Organization/03-dma.md) | DMA controller, transfer modes, bus arbitration |
| 10.4 | [I/O Processors](10-IO-Organization/04-io-processors.md) | IOP architecture, channels, CCW |

### [Unit 11: RISC vs CISC](11-RISC-vs-CISC/README.md)
| Chapter | Topic | Description |
|---------|-------|-------------|
| 11.1 | [RISC Architecture](11-RISC-vs-CISC/01-risc-architecture.md) | RISC principles, fixed-length instructions, pipelining |
| 11.2 | [CISC Architecture](11-RISC-vs-CISC/02-cisc-architecture.md) | Complex instructions, microprogrammed control |
| 11.3 | [Comparison and Modern Trends](11-RISC-vs-CISC/03-comparison-and-trends.md) | RISC vs CISC analysis, modern hybrid designs |

---

## 📖 How to Use These Notes

### Study Approach
1. **Sequential Learning**: Start from Unit 1 and progress through each unit
2. **Concept Building**: Each unit builds upon previous concepts
3. **Practice Problems**: Attempt revision questions at the end of each chapter
4. **ASCII Diagrams**: Study the circuit diagrams carefully for visual understanding

### Symbols Used
| Symbol | Meaning |
|--------|---------|
| 📌 | Important concept |
| ⚠️ | Common mistake or caution |
| 💡 | Tip or trick |
| 📝 | Note or additional information |
| 🔗 | Cross-reference to related topic |

---

## 🗺️ Architecture Overview Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         COMPUTER SYSTEM                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                    CENTRAL PROCESSING UNIT (CPU)                 │    │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │    │
│  │  │   Control Unit  │  │ Arithmetic Logic│  │    Registers    │  │    │
│  │  │                 │  │   Unit (ALU)    │  │                 │  │    │
│  │  │  - Sequencer    │  │                 │  │  - PC, IR, MAR  │  │    │
│  │  │  - Decoder      │  │  - Adder        │  │  - MDR, AC      │  │    │
│  │  │  - Timing       │  │  - Shifter      │  │  - GPRs         │  │    │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘  │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                              ↕ ↕ ↕                                       │
│  ════════════════════════════════════════════════════════════════════   │
│  ║    ADDRESS BUS    ║    DATA BUS    ║    CONTROL BUS    ║             │
│  ════════════════════════════════════════════════════════════════════   │
│                              ↕ ↕ ↕                                       │
│  ┌─────────────────────┐          ┌─────────────────────────────────┐   │
│  │       MEMORY        │          │         I/O SUBSYSTEM           │   │
│  │  ┌───────────────┐  │          │  ┌─────────┐  ┌─────────────┐   │   │
│  │  │  Cache (L1/L2)│  │          │  │   I/O   │  │ Peripherals │   │   │
│  │  ├───────────────┤  │          │  │Interface│  │             │   │   │
│  │  │  Main Memory  │  │          │  │         │  │ - Keyboard  │   │   │
│  │  │    (RAM)      │  │          │  │ - Ports │  │ - Display   │   │   │
│  │  ├───────────────┤  │          │  │ - DMA   │  │ - Storage   │   │   │
│  │  │ Secondary     │  │          │  │ - IOP   │  │ - Network   │   │   │
│  │  │ Storage       │  │          │  └─────────┘  └─────────────┘   │   │
│  │  └───────────────┘  │          └─────────────────────────────────┘   │
│  └─────────────────────┘                                                 │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📚 Quick Reference

### Key Formulas

| Concept | Formula |
|---------|---------|
| CPU Time | $\text{CPU Time} = \frac{\text{Instruction Count} \times \text{CPI}}{\text{Clock Rate}}$ |
| MIPS | $\text{MIPS} = \frac{\text{Instruction Count}}{\text{Execution Time} \times 10^6}$ |
| Speedup (Amdahl's Law) | $S = \frac{1}{(1-P) + \frac{P}{N}}$ |
| Cache Hit Ratio | $\text{Hit Ratio} = \frac{\text{Hits}}{\text{Hits} + \text{Misses}}$ |
| Effective Access Time | $\text{EAT} = h \times t_c + (1-h) \times t_m$ |
| Pipeline Speedup | $S = \frac{n \times k}{n + k - 1}$ |

### Important Abbreviations

| Abbreviation | Full Form |
|--------------|-----------|
| ALU | Arithmetic Logic Unit |
| CPI | Cycles Per Instruction |
| DMA | Direct Memory Access |
| IOP | Input/Output Processor |
| MAR | Memory Address Register |
| MDR | Memory Data Register |
| PC | Program Counter |
| IR | Instruction Register |
| RISC | Reduced Instruction Set Computer |
| CISC | Complex Instruction Set Computer |
| TLB | Translation Lookaside Buffer |

---

## 🚀 Getting Started

Begin your journey with [Unit 1: Basic Computer Organization](01-Basic-Computer-Organization/README.md)

---

## 📝 License

These notes are for educational purposes. Feel free to use and share for learning.

---

*Last Updated: January 2025*

---

## 📊 Course Statistics

| Metric | Count |
|--------|-------|
| **Total Units** | 11 |
| **Total Chapters** | 41 |
| **Practice Questions** | ~246 |
| **ASCII Diagrams** | 150+ |
