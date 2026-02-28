# Chapter 1.3: Bus Structures

## 📋 Chapter Overview

A bus is a communication pathway that connects multiple components in a computer system. Understanding bus structures is essential for comprehending how data, addresses, and control signals flow between the CPU, memory, and I/O devices.

---

## 🎯 Learning Objectives

- Understand the concept and purpose of system buses
- Explain the three types of buses: address, data, and control
- Analyze bus characteristics and timing
- Compare different bus architectures
- Understand bus arbitration and protocols

---

## 1. Introduction to Buses

### 📌 Definition

A **bus** is a collection of parallel wires (or traces on a PCB) that carry signals between computer components. It provides a shared communication path, reducing the number of connections needed.

### Why Buses?

```
WITHOUT BUS (Point-to-Point):              WITH BUS (Shared):
┌─────────────────────────────┐            ┌─────────────────────────────┐
│                             │            │                             │
│  CPU ◄─────────────► RAM    │            │       ═══════════════       │
│   │\                 /│     │            │       ║  SYSTEM BUS  ║      │
│   │ \               / │     │            │       ═══════════════       │
│   │  \             /  │     │            │           │ │ │             │
│   │   ◄───────────►   │     │            │   ┌───────┘ │ └───────┐    │
│   │   /\          /\  │     │            │   ▼         ▼         ▼    │
│   │  /  \        /  \ │     │            │  CPU       RAM       I/O   │
│   │ /    \      /    \│     │            │                             │
│   ◄───────►◄───►◄─────►     │            │  Fewer connections!        │
│  I/O      ROM   Disk        │            │  Standardized interface    │
│                             │            │                             │
│  Many wires needed!         │            │                             │
└─────────────────────────────┘            └─────────────────────────────┘
```

### Bus Advantages and Disadvantages

| Advantages | Disadvantages |
|------------|---------------|
| Reduced wiring complexity | Bandwidth bottleneck |
| Standardized interface | Requires arbitration |
| Easy to add devices | All devices share bandwidth |
| Lower cost | Single point of failure |
| Modular design | Speed limited by slowest device |

---

## 2. Types of Buses

### 📌 The Three-Bus System

A typical computer system has three main buses:

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            THREE-BUS ARCHITECTURE                                │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│   ┌─────────────────────────────────────────────────────────────────────────┐   │
│   │                             CPU                                          │   │
│   │   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐              │   │
│   │   │     MAR     │     │     MDR     │     │   Control   │              │   │
│   │   │  (n bits)   │     │  (m bits)   │     │    Unit     │              │   │
│   │   └──────┬──────┘     └──────┬──────┘     └──────┬──────┘              │   │
│   └──────────┼───────────────────┼───────────────────┼──────────────────────┘   │
│              │                   │                   │                          │
│              ▼                   ▼                   ▼                          │
│   ════════════════════════════════════════════════════════════════════════════  │
│   ║   ADDRESS BUS        ║   DATA BUS         ║   CONTROL BUS              ║  │
│   ║   A0, A1, A2...An   ║   D0, D1, D2...Dm ║   RD, WR, CLK, IRQ...      ║  │
│   ║   (Unidirectional)   ║   (Bidirectional)  ║   (Various directions)    ║  │
│   ║                      ║                    ║                            ║  │
│   ║   Width: 16-64 bits  ║   Width: 8-64 bits ║   Width: ~10-20 lines    ║  │
│   ════════════════════════════════════════════════════════════════════════════  │
│              │                   │                   │                          │
│              ▼                   ▼                   ▼                          │
│   ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐            │
│   │      Memory      │  │      Memory      │  │      Memory      │            │
│   │   (Decoder)      │  │   (Data Lines)   │  │   (RD/WR/CS)     │            │
│   └──────────────────┘  └──────────────────┘  └──────────────────┘            │
│              │                   │                   │                          │
│   ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐            │
│   │    I/O Devices   │  │    I/O Devices   │  │    I/O Devices   │            │
│   └──────────────────┘  └──────────────────┘  └──────────────────┘            │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 2.1 Address Bus

The address bus carries the memory or I/O address from the CPU to memory/peripherals.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                            ADDRESS BUS                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   CPU (Source)                              Memory/I/O (Destination)    │
│   ┌──────────┐                               ┌──────────────────┐       │
│   │   MAR    │───────────────────────────────►│ Address Decoder │       │
│   │          │  A15 A14 A13 ... A2 A1 A0    │                  │       │
│   └──────────┘  ═══════════════════════════  └──────────────────┘       │
│                                                                          │
│   Direction: Unidirectional (CPU → Memory/I/O)                          │
│                                                                          │
│   Address Bus Width and Addressable Memory:                             │
│   ┌──────────────────────────────────────────────────────────────┐      │
│   │  n-bit address bus can address 2^n locations                 │      │
│   │                                                               │      │
│   │    8-bit  →  2^8  = 256 bytes                                │      │
│   │   16-bit  →  2^16 = 64 KB                                    │      │
│   │   20-bit  →  2^20 = 1 MB     (8086)                          │      │
│   │   24-bit  →  2^24 = 16 MB    (80286)                         │      │
│   │   32-bit  →  2^32 = 4 GB     (80386, Pentium)                │      │
│   │   64-bit  →  2^64 = 16 EB   (Modern processors)              │      │
│   └──────────────────────────────────────────────────────────────┘      │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

**Key Points**:
- **Unidirectional**: Only CPU sends addresses
- **Width determines addressable memory**: $\text{Addressable Locations} = 2^n$
- **All address lines change simultaneously**

### 2.2 Data Bus

The data bus carries the actual data being transferred between components.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              DATA BUS                                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   CPU                                                   Memory/I/O      │
│   ┌──────────┐                                         ┌──────────┐     │
│   │   MDR    │◄═══════════════════════════════════════►│   Data   │     │
│   │          │  D7  D6  D5  D4  D3  D2  D1  D0        │  Buffer  │     │
│   └──────────┘                                         └──────────┘     │
│                                                                          │
│   Direction: Bidirectional (CPU ↔ Memory/I/O)                           │
│                                                                          │
│   Data Bus Width and Transfer Capability:                               │
│   ┌──────────────────────────────────────────────────────────────┐      │
│   │  Data bus width determines bits transferred per cycle        │      │
│   │                                                               │      │
│   │    8-bit  →  1 byte per transfer   (8080, 8085)              │      │
│   │   16-bit  →  2 bytes per transfer  (8086)                    │      │
│   │   32-bit  →  4 bytes per transfer  (80386, Pentium)          │      │
│   │   64-bit  →  8 bytes per transfer  (Modern processors)       │      │
│   │  128-bit  → 16 bytes per transfer  (Graphics, memory buses)  │      │
│   └──────────────────────────────────────────────────────────────┘      │
│                                                                          │
│   Tri-state Buffers:                                                     │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │  Data lines use tri-state buffers for bus sharing           │       │
│   │                                                              │       │
│   │         Enable                                               │       │
│   │           │                                                  │       │
│   │           ▼                                                  │       │
│   │  Input ──►╱╲──► Output                                       │       │
│   │          ╲╱   (High, Low, or High-Impedance/Float)          │       │
│   │                                                              │       │
│   └─────────────────────────────────────────────────────────────┘       │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

**Key Points**:
- **Bidirectional**: Data flows both ways
- **Uses tri-state buffers**: Prevents conflicts
- **Width affects performance**: Wider = more data per transfer

### 2.3 Control Bus

The control bus carries command and timing signals that coordinate system operations.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                            CONTROL BUS                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │                    CONTROL BUS SIGNALS                           │   │
│   ├─────────────────────────────────────────────────────────────────┤   │
│   │                                                                  │   │
│   │   CPU Generated (Output):                                        │   │
│   │   ┌──────────────┬───────────────────────────────────────────┐  │   │
│   │   │   Signal     │              Function                      │  │   │
│   │   ├──────────────┼───────────────────────────────────────────┤  │   │
│   │   │   RD/RD̄     │   Read strobe (active low)                 │  │   │
│   │   │   WR/WR̄     │   Write strobe (active low)                │  │   │
│   │   │   M/IŌ      │   Memory or I/O select                     │  │   │
│   │   │   ALE       │   Address Latch Enable                     │  │   │
│   │   │   INTĀ      │   Interrupt Acknowledge                    │  │   │
│   │   │   HLDA      │   Hold Acknowledge                         │  │   │
│   │   └──────────────┴───────────────────────────────────────────┘  │   │
│   │                                                                  │   │
│   │   External Signals (Input to CPU):                               │   │
│   │   ┌──────────────┬───────────────────────────────────────────┐  │   │
│   │   │   Signal     │              Function                      │  │   │
│   │   ├──────────────┼───────────────────────────────────────────┤  │   │
│   │   │   CLK        │   System clock                             │  │   │
│   │   │   RESET      │   System reset                             │  │   │
│   │   │   INTR       │   Interrupt Request                        │  │   │
│   │   │   NMI        │   Non-Maskable Interrupt                   │  │   │
│   │   │   HOLD       │   Bus Hold Request (DMA)                   │  │   │
│   │   │   READY      │   Memory/IO Ready                          │  │   │
│   │   └──────────────┴───────────────────────────────────────────┘  │   │
│   │                                                                  │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Bus Timing Diagrams

### 📌 Synchronous Bus Timing

In synchronous buses, all operations are synchronized to a common clock.

#### Read Operation Timing

```
                    BUS CYCLE (READ OPERATION)
     ────────────────────────────────────────────────────────────►
                                                              Time

     CLK    ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐
     ───────┘   └───┘   └───┘   └───┘   └───┘   └───┘   └───────
            T1      T2      T3      Tw      T4
           │       │       │       │       │
           │       │       │       │       │
     ADDR  ─┐ ┌─────────────────────────────────────┐ ┌─────────
     ──────┘ │           Valid Address              │ └─────────
             └─────────────────────────────────────┘
           │       │       │       │       │
           │       │       │       │       │
     RD̄    ────────┐                               ┌───────────
     ───────────────└───────────────────────────────┘
           │       │       │       │       │
           │       │       │       │       │
     DATA  ─────────────────────────┐ ┌─────────────────────────
     ──────────────────────────────┘ │   Valid Data  │ ─────────
                                     └───────────────┘
           │       │       │       │       │
           │       │       │       │       │
     READY ────────────────┐       ┌───────────────────────────
     ───────────────────────└───────┘
                           │       │
                         Wait States (if READY low)

     Legend:
     T1 = Address setup
     T2 = RD̄ asserted, wait for memory
     T3 = Data appears on bus (if fast memory)
     Tw = Wait state (inserted if memory slow)
     T4 = Data latched, RD̄ deasserted
```

#### Write Operation Timing

```
                    BUS CYCLE (WRITE OPERATION)
     ────────────────────────────────────────────────────────────►
                                                              Time

     CLK    ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐
     ───────┘   └───┘   └───┘   └───┘   └───┘   └───┘   └───────
            T1      T2      T3      T4
           │       │       │       │
           │       │       │       │
     ADDR  ─┐ ┌─────────────────────────────────┐ ┌─────────────
     ──────┘ │           Valid Address          │ └─────────────
             └─────────────────────────────────┘
           │       │       │       │
           │       │       │       │
     DATA  ─────────┐ ┌─────────────────────────┐ ┌─────────────
     ──────────────┘ │      Valid Data          │ └─────────────
                     └─────────────────────────┘
           │       │       │       │
           │       │       │       │
     WR̄    ─────────────────┐               ┌───────────────────
     ────────────────────────└───────────────┘
                             │               │
                           Write pulse (data latched on rising edge)
```

### 📌 Asynchronous Bus Timing

In asynchronous buses, devices use handshaking signals instead of a common clock.

```
               ASYNCHRONOUS READ OPERATION (Handshaking)
     ────────────────────────────────────────────────────────────►
                                                              Time

     ADDR   ┌───────────────────────────────────────┐
     ───────┘            Valid Address               └──────────
            │                                       │
            │                                       │
     REQ̄    ────┐                                   ┌───────────
     (Master)────└───────────────────────────────────┘
            │   │                                   │
            │   │                                   │
     DATA   ────────────────┐ ┌─────────────────────────────────
     ──────────────────────┘ │     Valid Data       │───────────
                             └─────────────────────┘
                             │                       │
                             │                       │
     ACK̄    ─────────────────┐                       ┌──────────
     (Slave)──────────────────└───────────────────────┘
                             │                       │
                          Slave ready           Master done

     Handshake Protocol:
     1. Master places address, asserts REQ̄
     2. Slave decodes address, prepares data
     3. Slave places data, asserts ACK̄
     4. Master reads data, deasserts REQ̄
     5. Slave deasserts ACK̄
```

### Comparison: Synchronous vs Asynchronous

| Aspect | Synchronous | Asynchronous |
|--------|-------------|--------------|
| **Timing** | Clock-based | Handshake-based |
| **Speed** | Fixed by clock | Adapts to device speed |
| **Design** | Simpler | More complex |
| **Flexibility** | Less flexible | More flexible |
| **Wait States** | Inserted as needed | Not needed |
| **Latency** | Predictable | Variable |

---

## 4. Bus Architecture Types

### 4.1 Single Bus Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       SINGLE BUS ARCHITECTURE                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ════════════════════════════════════════════════════════════════════  │
│   ║                        SYSTEM BUS                                ║  │
│   ════════════════════════════════════════════════════════════════════  │
│        │           │           │           │           │                │
│        ▼           ▼           ▼           ▼           ▼                │
│   ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐          │
│   │   CPU   │ │  Memory │ │   I/O   │ │   I/O   │ │   I/O   │          │
│   │         │ │         │ │  Dev 1  │ │  Dev 2  │ │  Dev 3  │          │
│   └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘          │
│                                                                          │
│   Pros: Simple, low cost, easy to implement                             │
│   Cons: Bottleneck, all devices share bandwidth                         │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 4.2 Dual Bus Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        DUAL BUS ARCHITECTURE                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ════════════════════════════════════════════════════════════════════  │
│   ║                        MEMORY BUS (Fast)                         ║  │
│   ════════════════════════════════════════════════════════════════════  │
│        │                        │                                       │
│        ▼                        ▼                                       │
│   ┌─────────┐              ┌─────────┐                                  │
│   │   CPU   │              │  Memory │                                  │
│   │         │              │  (RAM)  │                                  │
│   └────┬────┘              └─────────┘                                  │
│        │                                                                 │
│   ┌────┴────┐                                                           │
│   │  Bridge │                                                           │
│   └────┬────┘                                                           │
│        │                                                                 │
│   ════════════════════════════════════════════════════════════════════  │
│   ║                         I/O BUS (Slower)                         ║  │
│   ════════════════════════════════════════════════════════════════════  │
│        │           │           │           │                            │
│        ▼           ▼           ▼           ▼                            │
│   ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐                      │
│   │   I/O   │ │   I/O   │ │   I/O   │ │   I/O   │                      │
│   │  Dev 1  │ │  Dev 2  │ │  Dev 3  │ │  Dev 4  │                      │
│   └─────────┘ └─────────┘ └─────────┘ └─────────┘                      │
│                                                                          │
│   Pros: Memory access doesn't conflict with I/O                         │
│   Cons: Requires bridge chip, more complex                              │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 4.3 Hierarchical Bus Architecture (Modern Systems)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                   HIERARCHICAL BUS ARCHITECTURE                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌─────────────────────────────────────────────────────────────┐       │
│   │                         CPU                                  │       │
│   │   ┌────────────────────────────────────────────────────┐    │       │
│   │   │              Internal Bus (Fastest)                 │    │       │
│   │   │   [Registers] ↔ [L1 Cache] ↔ [ALU]                 │    │       │
│   │   └────────────────────────────────────────────────────┘    │       │
│   └────────────────────────────┬────────────────────────────────┘       │
│                                │                                         │
│   ══════════════════════════════════════════════════════════════════    │
│   ║            FRONT-SIDE BUS / SYSTEM BUS (Very Fast)             ║    │
│   ══════════════════════════════════════════════════════════════════    │
│                │                               │                         │
│                ▼                               ▼                         │
│   ┌────────────────────┐           ┌────────────────────┐              │
│   │    North Bridge    │           │    L2/L3 Cache     │              │
│   │  (Memory Controller│           │                    │              │
│   │    Hub - MCH)      │           └────────────────────┘              │
│   └──────────┬─────────┘                                                │
│              │                                                           │
│   ════════════════════════════════════════════════════════════════════  │
│   ║                     MEMORY BUS (Fast)                            ║  │
│   ════════════════════════════════════════════════════════════════════  │
│              │              │              │                            │
│              ▼              ▼              ▼                            │
│   ┌──────────────┐ ┌──────────────┐ ┌──────────────┐                   │
│   │   DIMM 1     │ │   DIMM 2     │ │   DIMM 3     │                   │
│   │   (RAM)      │ │   (RAM)      │ │   (RAM)      │                   │
│   └──────────────┘ └──────────────┘ └──────────────┘                   │
│                                                                          │
│   ┌────────────────────┐                                                │
│   │   North Bridge     │                                                │
│   └──────────┬─────────┘                                                │
│              │                                                           │
│   ════════════════════════════════════════════════════════════════════  │
│   ║                  PCIe / AGP BUS (High Speed)                     ║  │
│   ════════════════════════════════════════════════════════════════════  │
│              │                               │                          │
│              ▼                               ▼                          │
│   ┌──────────────────┐            ┌──────────────────┐                 │
│   │   Graphics Card  │            │   NVMe SSD       │                 │
│   │      (GPU)       │            │                  │                 │
│   └──────────────────┘            └──────────────────┘                 │
│                                                                          │
│   ┌────────────────────┐                                                │
│   │    South Bridge    │                                                │
│   │  (I/O Controller   │                                                │
│   │    Hub - ICH)      │                                                │
│   └──────────┬─────────┘                                                │
│              │                                                           │
│   ════════════════════════════════════════════════════════════════════  │
│   ║                     PCI / USB / SATA BUS                         ║  │
│   ════════════════════════════════════════════════════════════════════  │
│        │          │          │          │          │                    │
│        ▼          ▼          ▼          ▼          ▼                    │
│   ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐              │
│   │  USB   │ │  SATA  │ │  Audio │ │Network │ │  PCI   │              │
│   │Devices │ │  HDDs  │ │  Card  │ │  Card  │ │ Cards  │              │
│   └────────┘ └────────┘ └────────┘ └────────┘ └────────┘              │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Bus Arbitration

### 📌 Why Arbitration?

When multiple devices want to use the bus simultaneously, arbitration determines which device gets access.

### Arbitration Methods

#### 1. Daisy Chain (Serial) Arbitration

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    DAISY CHAIN ARBITRATION                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌───────────┐                                                         │
│   │ Arbiter   │                                                         │
│   │           │                                                         │
│   └─────┬─────┘                                                         │
│         │                                                                │
│   BUS   │ GRANT                                                         │
│  REQUEST│   │                                                           │
│    ▲    │   ▼                                                           │
│    │    │  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐       │
│    │    └──►│ Device 1│──►│ Device 2│──►│ Device 3│──►│ Device 4│       │
│    │       │(Highest)│   │         │   │         │   │(Lowest) │       │
│    │       └────┬────┘   └────┬────┘   └────┬────┘   └────┬────┘       │
│    │            │             │             │             │             │
│    └────────────┴─────────────┴─────────────┴─────────────┘             │
│                 (Common Bus Request Line - OR'd)                         │
│                                                                          │
│   Priority: Device 1 > Device 2 > Device 3 > Device 4                   │
│                                                                          │
│   Pros: Simple, low cost                                                 │
│   Cons: Fixed priority, unfair to low-priority devices                  │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

#### 2. Parallel (Independent Request) Arbitration

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    PARALLEL ARBITRATION                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│                    ┌───────────────────────┐                            │
│                    │       ARBITER         │                            │
│                    │                       │                            │
│                    │   Priority Encoder    │                            │
│                    │         +             │                            │
│                    │   Grant Logic         │                            │
│                    └───────────────────────┘                            │
│                      │  │  │  │   │  │  │  │                            │
│             REQ Lines│  │  │  │   │  │  │  │GRANT Lines                 │
│                      │  │  │  │   │  │  │  │                            │
│         REQ0─────────┘  │  │  │   │  │  │  └──────────GNT0              │
│         REQ1────────────┘  │  │   │  │  └─────────────GNT1              │
│         REQ2───────────────┘  │   │  └────────────────GNT2              │
│         REQ3──────────────────┘   └───────────────────GNT3              │
│           │                                              │               │
│   ┌───────┴────────┐                        ┌───────────┴────────┐      │
│   │    Device 0    │                        │    Device 0        │      │
│   │    Device 1    │                        │    Device 1        │      │
│   │    Device 2    │                        │    Device 2        │      │
│   │    Device 3    │                        │    Device 3        │      │
│   └────────────────┘                        └────────────────────┘      │
│                                                                          │
│   Pros: Faster arbitration, flexible priorities                         │
│   Cons: More hardware, more control lines                               │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

#### 3. Rotating Priority (Round Robin)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    ROTATING PRIORITY ARBITRATION                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   Initially:    D0 > D1 > D2 > D3    (D0 highest priority)              │
│                                                                          │
│   After D0 uses bus:                                                     │
│                 D1 > D2 > D3 > D0    (D0 now lowest)                    │
│                                                                          │
│   After D1 uses bus:                                                     │
│                 D2 > D3 > D0 > D1    (D1 now lowest)                    │
│                                                                          │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │                                                                  │   │
│   │     ┌────┐    ┌────┐    ┌────┐    ┌────┐                        │   │
│   │     │ D0 │───►│ D1 │───►│ D2 │───►│ D3 │───┐                    │   │
│   │     └────┘    └────┘    └────┘    └────┘   │                    │   │
│   │       ▲                                     │                    │   │
│   │       └─────────────────────────────────────┘                    │   │
│   │                 (Circular rotation)                              │   │
│   │                                                                  │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│   Pros: Fair to all devices, prevents starvation                        │
│   Cons: More complex logic, slightly slower                             │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Arbitration Comparison

| Method | Speed | Fairness | Complexity | Lines Needed |
|--------|-------|----------|------------|--------------|
| **Daisy Chain** | Slow | Low | Low | 2 (REQ, GNT) |
| **Parallel** | Fast | Configurable | Medium | 2n (n REQ, n GNT) |
| **Round Robin** | Medium | High | High | 2n |

---

## 6. Common Bus Standards

### Bus Standards Comparison Table

| Bus | Width | Speed | Max Devices | Application |
|-----|-------|-------|-------------|-------------|
| **ISA** | 16-bit | 8 MHz | 6 | Legacy PCs |
| **PCI** | 32/64-bit | 33/66 MHz | 5 | General I/O |
| **AGP** | 32-bit | 66 MHz × 8 | 1 | Graphics |
| **PCIe x1** | 1 lane | 250 MB/s (v1) | Many | General |
| **PCIe x16** | 16 lanes | 16 GB/s (v3) | Many | Graphics |
| **USB 2.0** | Serial | 480 Mbps | 127 | Peripherals |
| **USB 3.0** | Serial | 5 Gbps | 127 | Peripherals |
| **SATA III** | Serial | 6 Gbps | 1 per port | Storage |
| **DDR4** | 64-bit | 3200 MT/s | 2/channel | Memory |

---

## 📊 Summary Table

| Concept | Key Points |
|---------|------------|
| **Address Bus** | Unidirectional, carries memory addresses, width = log₂(addressable memory) |
| **Data Bus** | Bidirectional, carries data, width = bits per transfer |
| **Control Bus** | Various signals (RD, WR, IRQ, CLK), coordinates operations |
| **Synchronous Bus** | Clock-based timing, simpler design |
| **Asynchronous Bus** | Handshake-based, adapts to device speed |
| **Single Bus** | Simple but bottleneck-prone |
| **Hierarchical** | Multiple levels, optimizes for speed differences |
| **Arbitration** | Resolves bus contention, can be serial or parallel |

---

## ❓ Quick Revision Questions

1. **Q**: What determines the maximum addressable memory in a system?
   
   **A**: The width of the address bus. An n-bit address bus can address $2^n$ memory locations.

2. **Q**: Why does the data bus need tri-state buffers?
   
   **A**: To allow multiple devices to share the bus without conflict. When a device is not transmitting, its buffers go to high-impedance state, effectively disconnecting it from the bus.

3. **Q**: What is the difference between synchronous and asynchronous buses?
   
   **A**: Synchronous buses use a common clock for timing; asynchronous buses use handshake signals (REQ/ACK) allowing devices of different speeds to communicate.

4. **Q**: Explain daisy chain arbitration and its main disadvantage.
   
   **A**: In daisy chain arbitration, the grant signal passes through devices in sequence. Each device can block the grant from reaching lower-priority devices. Main disadvantage: fixed priority causes unfairness and potential starvation.

5. **Q**: What are the three main signals on the control bus for memory operations?
   
   **A**: (1) RD̄ (Read) - indicates read operation, (2) WR̄ (Write) - indicates write operation, (3) M/IŌ - selects between memory and I/O access.

6. **Q**: Draw a timing diagram for a synchronous memory read operation.
   
   **A**: See Section 3 - shows CLK, Address, RD̄, and Data signals with their timing relationships over T1-T4 clock cycles.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [CPU, Memory, I/O Organization](02-cpu-memory-io-organization.md) | [Unit 1 Home](README.md) | [System Interconnection](04-system-interconnection.md) |

---

[← Previous Chapter](02-cpu-memory-io-organization.md) | [Course Home](../README.md) | [Next Chapter →](04-system-interconnection.md)
