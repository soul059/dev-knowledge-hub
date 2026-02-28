# Unit 4: 8086 Microprocessor

## 📚 Unit Overview

The Intel 8086 is a 16-bit microprocessor that revolutionized the personal computer industry. This unit covers the architecture, registers, addressing modes, instruction set, and memory organization of the 8086 microprocessor.

---

## 🎯 Unit Objectives

After completing this unit, you will be able to:
- Understand 8086 architecture and pin diagram
- Explain the concept of segmented memory
- Describe all registers and their functions
- Understand different addressing modes
- Compare 8086 with 8085 microprocessor
- Explain minimum and maximum mode operations

---

## 📖 Chapter List

### [4.1 Architecture and Pin Diagram](01-architecture-pin-diagram.md)
- 8086 internal architecture
- 40-pin configuration
- BIU and EU components
- Signal descriptions

### [4.2 Register Organization](02-register-organization.md)
- General purpose registers
- Segment registers
- Pointer and index registers
- Flag register (16-bit)

### [4.3 Memory Segmentation](03-memory-segmentation.md)
- Segmented memory concept
- Physical address calculation
- Segment:Offset notation
- 1MB addressing with 16-bit registers

### [4.4 Addressing Modes](04-addressing-modes.md)
- Data addressing modes
- Memory addressing modes
- Effective address calculation
- Examples and applications

### [4.5 Instruction Set Overview](05-instruction-set.md)
- Data transfer instructions
- Arithmetic and logical instructions
- String instructions
- Control transfer instructions

### [4.6 Minimum and Maximum Mode](06-min-max-mode.md)
- Minimum mode configuration
- Maximum mode configuration
- Bus controller 8288
- Multi-processor systems

---

## 🔑 Key Concepts

```
8086 vs 8085 COMPARISON:

┌─────────────────────────┬─────────────┬─────────────┐
│       Feature           │    8085     │    8086     │
├─────────────────────────┼─────────────┼─────────────┤
│ Data Bus Width          │   8-bit     │   16-bit    │
│ Address Bus Width       │   16-bit    │   20-bit    │
│ Addressable Memory      │   64 KB     │   1 MB      │
│ Instruction Queue       │   None      │   6 bytes   │
│ Pipelining              │   No        │   Yes       │
│ Registers               │   8-bit     │   16-bit    │
│ Multiplication/Division │   No        │   Yes       │
│ Memory Segmentation     │   No        │   Yes       │
│ Clock Speed             │   3-5 MHz   │   5-10 MHz  │
│ I/O Ports               │   256       │   65,536    │
│ Hardware Interrupts     │   5         │   2 + NMI   │
└─────────────────────────┴─────────────┴─────────────┘
```

---

## 🏗️ Architecture Overview

```
8086 INTERNAL ARCHITECTURE:

┌─────────────────────────────────────────────────────────────────┐
│                         8086 CPU                                │
│  ┌────────────────────────┬────────────────────────────────┐   │
│  │    BUS INTERFACE UNIT  │    EXECUTION UNIT              │   │
│  │         (BIU)          │         (EU)                   │   │
│  │  ┌──────────────────┐  │  ┌──────────────────────────┐  │   │
│  │  │   CS, DS, ES, SS │  │  │  AX, BX, CX, DX          │  │   │
│  │  │  Segment Regs    │  │  │  General Registers       │  │   │
│  │  └──────────────────┘  │  └──────────────────────────┘  │   │
│  │  ┌──────────────────┐  │  ┌──────────────────────────┐  │   │
│  │  │       IP         │  │  │  SP, BP, SI, DI          │  │   │
│  │  │ Instruction Ptr  │  │  │  Pointer/Index Regs      │  │   │
│  │  └──────────────────┘  │  └──────────────────────────┘  │   │
│  │  ┌──────────────────┐  │  ┌──────────────────────────┐  │   │
│  │  │  Instruction     │  │  │        ALU               │  │   │
│  │  │    Queue         │◄─┼─►│   (16-bit)               │  │   │
│  │  │   (6 bytes)      │  │  │                          │  │   │
│  │  └──────────────────┘  │  └──────────────────────────┘  │   │
│  │  ┌──────────────────┐  │  ┌──────────────────────────┐  │   │
│  │  │ Address Adder    │  │  │      Flag Register       │  │   │
│  │  │  (20-bit)        │  │  │       (16-bit)           │  │   │
│  │  └──────────────────┘  │  └──────────────────────────┘  │   │
│  │           │            │             │                  │   │
│  └───────────┼────────────┴─────────────┼──────────────────┘   │
│              │                          │                       │
│              ▼                          ▼                       │
│     ┌────────────────┐         ┌────────────────┐              │
│     │  20-bit Address│         │ Control Signals│              │
│     │  16-bit Data   │         │                │              │
│     └────────────────┘         └────────────────┘              │
└─────────────────────────────────────────────────────────────────┘

BIU Functions:
• Fetch instructions from memory
• Read/write operands from/to memory
• Calculate physical addresses
• Manages instruction queue (prefetch)

EU Functions:
• Decode instructions
• Execute instructions
• Perform arithmetic/logical operations
• Update flags
```

---

## 💾 Memory Organization

```
8086 MEMORY MAP (1 MB):

     FFFFFH ┌───────────────────────────────────┐
            │                                   │
            │    Available for user programs    │
            │           (960 KB)                │
            │                                   │
     10000H ├───────────────────────────────────┤
            │    High Memory Area               │
     0FFFFH ├───────────────────────────────────┤
            │    Conventional Memory            │
            │    (First 1 MB addressable)       │
     00500H ├───────────────────────────────────┤
            │    BIOS Data Area                 │
     00400H ├───────────────────────────────────┤
            │    Interrupt Vector Table         │
            │    (256 vectors × 4 bytes = 1KB)  │
     00000H └───────────────────────────────────┘

SEGMENTED MEMORY CONCEPT:

Physical Address = Segment × 16 + Offset
                 = Segment × 10H + Offset

Example: CS:IP = 1234H:5678H
Physical = 1234H × 10H + 5678H
         = 12340H + 5678H
         = 179B8H
```

---

## 🔗 Related Resources

- **Prerequisites**: [Unit 2: 8085 Microprocessor](../02-8085-Microprocessor/README.md)
- **Next Unit**: [Unit 5: 8086 Programming](../05-8086-Programming/README.md)
- **Advanced Topics**: [Unit 9: Advanced Processors](../09-Advanced-Processors/README.md)

---

## 📝 Study Tips

1. **Master segmentation first** - Understanding physical address calculation is crucial
2. **Compare with 8085** - Many concepts build on 8085 knowledge
3. **Focus on register purposes** - Each register has specific uses
4. **Practice addressing modes** - They determine operand locations
5. **Understand pipelining** - BIU and EU work in parallel

---

## 🧭 Navigation

| Previous Unit | Course Home | Next Unit |
|---------------|-------------|-----------|
| [Unit 3: 8085 Programming](../03-8085-Programming/README.md) | [Main Index](../README.md) | [Unit 5: 8086 Programming](../05-8086-Programming/README.md) |

---

*[← Back to Main Index](../README.md)*
