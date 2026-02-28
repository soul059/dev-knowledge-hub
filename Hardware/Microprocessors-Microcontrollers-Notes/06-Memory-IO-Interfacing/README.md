# Unit 6: Memory and I/O Interfacing

## 📋 Unit Overview

This unit covers the crucial aspects of interfacing memory and I/O devices with 8085 and 8086 microprocessors. Understanding interfacing is essential for designing complete microprocessor-based systems. We'll explore memory organization, address decoding, and programmable peripheral devices.

---

## 🎯 Unit Objectives

By the end of this unit, you will:
- Understand memory organization and interfacing techniques
- Design address decoding circuits
- Interface I/O devices using memory-mapped and I/O-mapped techniques
- Program and interface 8255 PPI (Programmable Peripheral Interface)
- Configure and use 8253/8254 Timer/Counter
- Understand 8259 PIC (Programmable Interrupt Controller)
- Learn DMA concepts and 8257 DMA controller

---

## 📊 Interfacing Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MICROPROCESSOR SYSTEM                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│    ┌──────────────┐                                                 │
│    │    CPU       │                                                 │
│    │  (8085/8086) │                                                 │
│    └──────┬───────┘                                                 │
│           │                                                         │
│    ┌──────┴──────┐                                                  │
│    │             │                                                  │
│    ▼             ▼                                                  │
│  Address       Data                                                 │
│   Bus           Bus         ──────────────────────────────          │
│    │             │         │     Control Bus               │        │
│    ├──────┬──────┤         │  (RD, WR, IO/M, ALE, etc.)   │        │
│    │      │      │         └─────────────┬─────────────────┘        │
│    ▼      ▼      ▼                       │                          │
│  ┌────┐ ┌────┐ ┌────┐                    │                          │
│  │Addr│ │Data│ │Data│                    │                          │
│  │Dcdr│ │Buff│ │Ltch│                    │                          │
│  └─┬──┘ └─┬──┘ └─┬──┘                    │                          │
│    │      │      │                       │                          │
│    └──────┴──────┴───────────────────────┘                          │
│                  │                                                  │
│    ┌─────────────┼─────────────────────────────┐                    │
│    │             │             │               │                    │
│    ▼             ▼             ▼               ▼                    │
│ ┌──────┐    ┌──────┐     ┌─────────┐     ┌─────────┐               │
│ │ ROM  │    │ RAM  │     │  8255   │     │  8253   │               │
│ │Memory│    │Memory│     │   PPI   │     │  Timer  │               │
│ └──────┘    └──────┘     └─────────┘     └─────────┘               │
│                               │               │                     │
│                    ┌──────────┴───────────────┴─────────┐          │
│                    │      Peripheral Devices            │          │
│                    │  (Keyboard, Display, ADC, etc.)    │          │
│                    └────────────────────────────────────┘          │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📚 Chapter Contents

### [Chapter 1: Memory Interfacing](01-memory-interfacing.md)
- Memory types: RAM, ROM, EPROM, EEPROM
- Memory organization and capacity
- Address decoding techniques
- Memory map design
- Timing analysis

### [Chapter 2: I/O Interfacing Techniques](02-io-interfacing.md)
- Memory-mapped I/O
- I/O-mapped (peripheral-mapped) I/O
- Comparison and applications
- Data transfer methods

### [Chapter 3: 8255 Programmable Peripheral Interface](03-8255-ppi.md)
- 8255 architecture and pin diagram
- Operating modes (Mode 0, 1, 2)
- Control word format
- Programming examples
- Practical applications

### [Chapter 4: 8253/8254 Programmable Interval Timer](04-8253-timer.md)
- Architecture and features
- Operating modes (0-5)
- Control word format
- Programming for timing and counting
- Waveform generation

### [Chapter 5: 8259 Programmable Interrupt Controller](05-8259-pic.md)
- Architecture and pin diagram
- Interrupt handling mechanism
- Command words (ICW, OCW)
- Cascading multiple 8259s
- Priority modes

### [Chapter 6: DMA and 8257 Controller](06-dma-controller.md)
- Direct Memory Access concept
- DMA operation and timing
- 8257 architecture
- Programming 8257
- Applications

---

## 🔗 Key Peripheral Chips

| Chip | Function | Ports/Channels | Key Feature |
|------|----------|----------------|-------------|
| 8255 | Parallel I/O | 3 ports (24 bits) | 3 operating modes |
| 8253 | Timer/Counter | 3 channels | 6 operating modes |
| 8254 | Enhanced Timer | 3 channels | Read-back command |
| 8259 | Interrupt Control | 8 IRQ lines | Cascading support |
| 8257 | DMA Control | 4 channels | 4 DMA modes |
| 8251 | Serial I/O | 1 channel | USART |
| 8279 | Keyboard/Display | 64 keys, 16 displays | Scanned input |

---

## 📖 Essential Formulas

### Memory Calculations
```
Memory Size = 2^n bytes  (where n = address lines)
Chip Select Lines = log₂(Number of memory chips)
Address Range = Starting Address to (Starting + Size - 1)
```

### Timer Calculations
```
Output Frequency = Input Clock / Count Value
Time Delay = Count Value × Clock Period
Count Value = Delay Required / Clock Period
```

### Interrupt Priority
```
Highest Priority: IR0
Lowest Priority: IR7
Priority Order: IR0 > IR1 > IR2 > ... > IR7
```

---

## 🎓 Exam Preparation Tips

1. **Memory Interfacing**: Practice address decoding for different memory chips
2. **8255 PPI**: Know all three modes and their control words
3. **8253 Timer**: Calculate count values for given frequencies
4. **8259 PIC**: Understand ICW and OCW commands
5. **DMA**: Know the difference between DMA and programmed I/O

---

## 🧭 Navigation

| Previous Unit | Home | Next Unit |
|---------------|------|-----------|
| [Unit 5: 8086 Programming](../05-8086-Programming/README.md) | [Course Home](../README.md) | [Unit 7: 8051 Architecture](../07-8051-Architecture/README.md) |

---

*[← Previous Unit: 8086 Programming](../05-8086-Programming/README.md) | [Next Unit: 8051 Architecture →](../07-8051-Architecture/README.md)*
