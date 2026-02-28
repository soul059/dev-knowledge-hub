# Unit 2: 8085 Microprocessor

## 📋 Unit Overview

The Intel 8085 is an 8-bit microprocessor that serves as the foundation for understanding microprocessor architecture. This unit provides a comprehensive study of the 8085's architecture, registers, instruction set, timing, and interrupt system.

## 🎯 Learning Objectives

After completing this unit, you will be able to:

- Draw and explain the complete 8085 architecture
- Identify all pins and their functions
- Understand register organization and flag operations
- Classify and use 8085 instructions
- Analyze timing diagrams for various machine cycles
- Implement interrupt-driven programs

## 📑 Chapter Contents

| Chapter | Topic | Description |
|---------|-------|-------------|
| 2.1 | [Architecture and Pin Diagram](01-architecture-pin-diagram.md) | Complete 8085 internal structure and pins |
| 2.2 | [Register Organization](02-register-organization.md) | General purpose and special registers |
| 2.3 | [Instruction Set Classification](03-instruction-set-classification.md) | Categories and formats of instructions |
| 2.4 | [Addressing Modes](04-addressing-modes.md) | Methods to specify operands |
| 2.5 | [Timing Diagrams](05-timing-diagrams.md) | Machine cycles and T-states |
| 2.6 | [Interrupts](06-interrupts.md) | Hardware and software interrupts |

## 🔑 Key Concepts in This Unit

```
┌─────────────────────────────────────────────────────────────────┐
│                    8085 MICROPROCESSOR                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│   Key Specifications:                                            │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  • 8-bit data bus, 16-bit address bus                   │   │
│   │  • 64 KB memory addressing capability                   │   │
│   │  • Single +5V power supply                              │   │
│   │  • 3 MHz to 5 MHz clock frequency                       │   │
│   │  • 40-pin DIP package                                   │   │
│   │  • 74 instruction types (246 opcodes)                   │   │
│   │  • 5 hardware interrupts                                │   │
│   │  • Serial I/O capability (SID, SOD)                     │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                   │
│   Register Set:                                                  │
│   ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐        │
│   │  A  │  │  B  │  │  C  │  │  D  │  │  E  │  │  H  │  │  L  │        │
│   └─────┘  └─────┘  └─────┘  └─────┘  └─────┘  └─────┘        │
│      ▲         └──┬──┘           └──┬──┘           └──┬──┘        │
│      │           BC              DE                HL           │
│   Accumulator  (16-bit pairs for extended operations)          │
│                                                                   │
│   Flags: S | Z | AC | P | CY                                    │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

## 📊 8085 Quick Reference

### Pin Categories
| Category | Pins | Description |
|----------|------|-------------|
| Address/Data | AD0-AD7 | Multiplexed lower address and data |
| Address | A8-A15 | Higher order address lines |
| Control | RD̄, WR̄, IO/M̄, ALE | Memory/IO control signals |
| Interrupt | TRAP, RST 7.5, 6.5, 5.5, INTR | Interrupt inputs |
| Serial | SID, SOD | Serial I/O data |
| DMA | HOLD, HLDA | Direct Memory Access signals |
| Clock | X1, X2, CLK OUT | Clock input and output |
| Power | Vcc, Vss | +5V and Ground |

### Machine Cycles
| Cycle Type | T-States | Control Signals |
|------------|----------|-----------------|
| Opcode Fetch | 4-6 | IO/M̄=0, RD̄=0 |
| Memory Read | 3 | IO/M̄=0, RD̄=0 |
| Memory Write | 3 | IO/M̄=0, WR̄=0 |
| I/O Read | 3 | IO/M̄=1, RD̄=0 |
| I/O Write | 3 | IO/M̄=1, WR̄=0 |
| Interrupt Ack | 6 | INTA=0 |

## ⏱️ Estimated Study Time

| Chapter | Time Required |
|---------|---------------|
| 2.1 Architecture & Pin Diagram | 90 minutes |
| 2.2 Register Organization | 60 minutes |
| 2.3 Instruction Set | 90 minutes |
| 2.4 Addressing Modes | 60 minutes |
| 2.5 Timing Diagrams | 90 minutes |
| 2.6 Interrupts | 75 minutes |
| **Total** | **~8 hours** |

---

## 🔗 Prerequisites

Before starting this unit, ensure you understand:
- [Basic Microprocessor Architecture](../01-Introduction-to-Microprocessors/03-basic-microprocessor-architecture.md)
- [Bus Organization](../01-Introduction-to-Microprocessors/04-bus-organization.md)
- Number systems and binary arithmetic

---

## 🧭 Navigation

| Previous | Home | Next |
|----------|------|------|
| [Unit 1: Introduction](../01-Introduction-to-Microprocessors/README.md) | [Main Index](../README.md) | [Unit 3: 8085 Programming](../03-8085-Programming/README.md) |

---

*Start with [Chapter 2.1: Architecture and Pin Diagram](01-architecture-pin-diagram.md) →*
