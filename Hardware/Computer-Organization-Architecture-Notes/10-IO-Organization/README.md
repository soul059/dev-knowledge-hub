# Unit 10: I/O Organization

## 📋 Unit Overview

Input/Output (I/O) organization deals with communication between the CPU and external devices. This unit covers I/O techniques from simple programmed I/O to sophisticated DMA transfers and I/O processors.

---

## 🎯 Unit Objectives

- Understand I/O interface design and addressing
- Master interrupt-driven I/O mechanisms
- Learn Direct Memory Access (DMA) principles
- Analyze I/O processors and channels

---

## 📚 Chapters in This Unit

| Chapter | Topic | Key Concepts |
|---------|-------|--------------|
| 10.1 | [I/O Interface](01-io-interface.md) | Peripheral interface, handshaking, parallel/serial I/O |
| 10.2 | [I/O Techniques](02-io-techniques.md) | Programmed I/O, interrupt-driven I/O, polling |
| 10.3 | [Direct Memory Access](03-dma.md) | DMA controller, burst/cycle stealing, bus arbitration |
| 10.4 | [I/O Processors](04-io-processors.md) | IOP, channels, command chaining, device independence |

---

## 🔑 Key Concepts

| Concept | Description |
|---------|-------------|
| Programmed I/O | CPU directly controls data transfer |
| Interrupt I/O | Device signals CPU when ready |
| DMA | Hardware controller transfers data directly to memory |
| IOP | Dedicated processor handles I/O operations |

---

## 📊 I/O Techniques Comparison

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    I/O TECHNIQUES SPECTRUM                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│                                                                          │
│   CPU                                                                    │
│   Involvement                                                            │
│       ▲                                                                  │
│       │                                                                  │
│   High│  ┌───────────────┐                                               │
│       │  │  Programmed   │                                               │
│       │  │     I/O       │                                               │
│       │  └───────────────┘                                               │
│       │         │                                                        │
│       │         ▼                                                        │
│       │  ┌───────────────┐                                               │
│       │  │  Interrupt    │                                               │
│       │  │   Driven      │                                               │
│       │  └───────────────┘                                               │
│       │         │                                                        │
│       │         ▼                                                        │
│       │  ┌───────────────┐                                               │
│       │  │     DMA       │                                               │
│       │  │  (Direct      │                                               │
│       │  │   Memory)     │                                               │
│       │  └───────────────┘                                               │
│       │         │                                                        │
│       │         ▼                                                        │
│   Low │  ┌───────────────┐                                               │
│       │  │     IOP       │                                               │
│       │  │ (I/O Channel) │                                               │
│       └──┴───────────────┴──────────────────────────────────────────►   │
│                                                          Data Rate       │
│          Low                                             High            │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 🔗 Prerequisites

- Unit 1: Basic Computer Organization (bus concepts)
- Unit 4: Register Transfer Language
- Unit 8: Pipeline Processing (for understanding I/O impact)

---

## 📖 Reading Guide

1. Start with I/O Interface to understand hardware basics
2. Study I/O Techniques for software control methods
3. Learn DMA for high-speed transfers
4. Complete with I/O Processors for advanced systems

---

[← Previous Unit: Memory Organization](../09-Memory-Organization/README.md) | [Course Home](../README.md) | [First Chapter: I/O Interface →](01-io-interface.md)
