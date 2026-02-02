# Chapter 7.2: I/O Methods

## Overview

The CPU can communicate with I/O devices in several ways. This chapter covers the main methods: Programmed I/O (Polling), Interrupt-Driven I/O, and Direct Memory Access (DMA).

---

## The I/O Problem

```
┌──────────────────────────────────────────────────────────────────┐
│                    THE I/O PROBLEM                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Speed mismatch between CPU and I/O devices:                    │
│                                                                  │
│  CPU: Billions of operations per second                         │
│  Keyboard: 10 characters per second                             │
│  Disk: Millions of bytes per second (still slow vs CPU)        │
│                                                                  │
│  Questions:                                                      │
│  1. How does CPU know when device is ready?                     │
│  2. How is data transferred between CPU and device?             │
│  3. How to minimize CPU overhead?                               │
│                                                                  │
│  Three main approaches:                                          │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Method              │ CPU Involvement                    │   │
│  ├─────────────────────┼────────────────────────────────────┤   │
│  │ Polling             │ High (busy waiting)                │   │
│  │ Interrupts          │ Medium (per event)                 │   │
│  │ DMA                 │ Low (per block)                    │   │
│  └─────────────────────┴────────────────────────────────────┘   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 1. Programmed I/O (Polling)

### Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                   PROGRAMMED I/O (POLLING)                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CPU repeatedly checks device status (busy wait)                │
│  Also called "polling" or "busy waiting"                        │
│                                                                  │
│  To print a character:                                           │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  while (status_register & BUSY)                            │ │
│  │      ; // keep checking (busy wait)                        │ │
│  │  data_register = character;                                │ │
│  │  command_register = PRINT;                                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  Timeline:                                                       │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ CPU                                                       │   │
│  │ │                                                         │   │
│  │ │ Check → Check → Check → Check → Ready! → Send data     │   │
│  │ │   │       │       │       │       │                    │   │
│  │ │   ▼       ▼       ▼       ▼       ▼                    │   │
│  │ │ Busy   Busy    Busy    Busy    Done                    │   │
│  │ │                                                         │   │
│  │ │ ←────── Wasted CPU cycles ──────→                      │   │
│  │ └─────────────────────────────────────────────────────────┘  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Advantages and Disadvantages

```
ADVANTAGES:
✓ Simple to implement
✓ Predictable timing (no interrupt latency)
✓ Useful for very fast devices or when CPU has nothing else to do

DISADVANTAGES:
✗ Wastes CPU cycles (busy waiting)
✗ CPU cannot do other work while polling
✗ Inefficient for slow devices
✗ Poor for multiple devices

When used:
• Simple embedded systems
• Very fast devices (GPU command queues)
• Real-time systems requiring predictable timing
```

---

## 2. Interrupt-Driven I/O

### Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                  INTERRUPT-DRIVEN I/O                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Device notifies CPU when ready via INTERRUPT                   │
│  CPU can do other work while waiting                            │
│                                                                  │
│  Timeline:                                                       │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                                                          │   │
│  │  CPU          Device                                     │   │
│  │   │             │                                        │   │
│  │   │──Issue I/O──│                                        │   │
│  │   │   Request   │                                        │   │
│  │   │             │──────┐                                 │   │
│  │   │  Do other   │      │ Device works...                │   │
│  │   │   work      │      │                                 │   │
│  │   │             │◄─────┘                                 │   │
│  │   │◄──INTERRUPT─│                                        │   │
│  │   │             │                                        │   │
│  │   │──Handler────│                                        │   │
│  │   │  executes   │                                        │   │
│  │   │             │                                        │   │
│  │   │──Resume─────│                                        │   │
│  │   │  work       │                                        │   │
│  │   ▼             ▼                                        │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  CPU is FREE while device works!                                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Interrupt Handling

```
┌──────────────────────────────────────────────────────────────────┐
│                   INTERRUPT HANDLING                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step-by-step process:                                           │
│                                                                  │
│  1. Device raises interrupt signal                               │
│  2. CPU finishes current instruction                            │
│  3. CPU acknowledges interrupt                                   │
│  4. CPU saves current state (PC, registers)                     │
│  5. CPU looks up interrupt handler address (via IVT)            │
│  6. Handler executes (process the I/O)                          │
│  7. Handler returns, CPU restores state                         │
│  8. CPU resumes interrupted program                             │
│                                                                  │
│  Interrupt Vector Table (IVT):                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Vector # │ Device        │ Handler Address              │   │
│  ├──────────┼───────────────┼──────────────────────────────┤   │
│  │    0     │ Timer         │ 0x00001000                   │   │
│  │    1     │ Keyboard      │ 0x00002000                   │   │
│  │    2     │ Disk          │ 0x00003000                   │   │
│  │    3     │ Network       │ 0x00004000                   │   │
│  │   ...    │ ...           │ ...                          │   │
│  └──────────┴───────────────┴──────────────────────────────┘   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Interrupt Types

```
┌──────────────────────────────────────────────────────────────────┐
│                     INTERRUPT TYPES                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  HARDWARE INTERRUPTS (External):                                 │
│  • Generated by devices                                         │
│  • Asynchronous (can occur anytime)                             │
│  • Examples: keyboard press, disk complete, timer tick          │
│                                                                  │
│  SOFTWARE INTERRUPTS (Internal):                                 │
│  • Triggered by instructions                                    │
│  • Synchronous                                                   │
│  • Used for system calls (trap/syscall instruction)            │
│                                                                  │
│  EXCEPTIONS:                                                     │
│  • Errors during execution                                      │
│  • Examples: divide by zero, page fault, invalid opcode        │
│                                                                  │
│  MASKABLE vs NON-MASKABLE:                                       │
│  • Maskable: Can be temporarily disabled                        │
│  • Non-maskable (NMI): Cannot be disabled (critical errors)    │
│                                                                  │
│  INTERRUPT PRIORITY:                                             │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ High Priority │ NMI, Timer, Power fail                  │    │
│  │ Medium        │ Disk, Network                           │    │
│  │ Low Priority  │ Keyboard, Mouse                         │    │
│  └─────────────────────────────────────────────────────────┘    │
│  Higher priority interrupts can preempt lower priority ones     │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Advantages and Disadvantages

```
ADVANTAGES:
✓ CPU can do useful work while waiting
✓ Efficient for slow devices
✓ Good for multiple devices
✓ Event-driven (no wasted polling)

DISADVANTAGES:
✗ Interrupt overhead (save/restore context)
✗ Complex hardware and software
✗ Can be overwhelmed by too many interrupts
✗ Latency between event and response

When used:
• Most general-purpose computing
• Keyboard, mouse, network
• Slow to moderate-speed devices
```

---

## 3. Direct Memory Access (DMA)

### Concept

```
┌──────────────────────────────────────────────────────────────────┐
│              DIRECT MEMORY ACCESS (DMA)                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Special DMA controller transfers data directly to/from memory  │
│  CPU only involved at start and end of transfer                 │
│                                                                  │
│  Without DMA (CPU moves each byte):                              │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  CPU: Read → Store → Read → Store → ... (1000 times)    │   │
│  │  CPU fully occupied during transfer                      │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  With DMA:                                                       │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  CPU: Setup DMA → Do other work → Handle completion IRQ │   │
│  │  DMA: Transfer 1000 bytes directly to memory            │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                                                             ││
│  │    ┌─────┐      ┌──────────────────┐      ┌───────────┐   ││
│  │    │ CPU │◄────►│   Memory         │◄────►│   DMA     │   ││
│  │    └─────┘      └──────────────────┘      │Controller │   ││
│  │       │                 ▲                 └─────┬─────┘   ││
│  │       │                 │                       │          ││
│  │       │                 └───────────────────────┘          ││
│  │       │                     Direct transfer                ││
│  │       ▼                                                    ││
│  │    ┌────────────────────────────────────────────┐          ││
│  │    │              Device                         │          ││
│  │    └────────────────────────────────────────────┘          ││
│  │                                                             ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### DMA Operation

```
┌──────────────────────────────────────────────────────────────────┐
│                     DMA OPERATION                                 │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 1: CPU programs DMA controller                            │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ DMA Controller Registers:                                │   │
│  │ • Memory address register: Where to put/get data        │   │
│  │ • Count register: How many bytes                        │   │
│  │ • Control register: Direction, mode                     │   │
│  │ • Device address: Which device                          │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  Step 2: DMA controller takes over                               │
│  • Requests bus access from CPU                                 │
│  • Transfers data between device and memory                     │
│  • Decrements count, increments address                         │
│                                                                  │
│  Step 3: Transfer complete                                       │
│  • DMA controller raises interrupt                              │
│  • CPU handles completion (check status, next operation)        │
│                                                                  │
│  Timeline:                                                       │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ CPU:  Setup │ Work │ Work │ Work │ Work │ Work │ IRQ    │   │
│  │       ─────►│      │      │      │      │      │◄─────  │   │
│  │ DMA:        │ Xfer │ Xfer │ Xfer │ Xfer │ Xfer │ Done   │   │
│  │             └──────┴──────┴──────┴──────┴──────┘        │   │
│  │             CPU does other work during transfer!         │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### DMA Modes

```
┌──────────────────────────────────────────────────────────────────┐
│                      DMA MODES                                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. BURST MODE (Block Transfer)                                  │
│     • DMA takes bus, transfers entire block                     │
│     • CPU blocked during transfer                               │
│     • Fastest for device, blocks CPU                            │
│                                                                  │
│     Bus: DMA─DMA─DMA─DMA─DMA─DMA─DMA─DMA (all DMA)             │
│                                                                  │
│  2. CYCLE STEALING                                               │
│     • DMA steals one bus cycle at a time                        │
│     • CPU and DMA alternate                                     │
│     • CPU slowed but not blocked                                │
│                                                                  │
│     Bus: CPU─DMA─CPU─DMA─CPU─DMA─CPU─DMA (interleaved)         │
│                                                                  │
│  3. TRANSPARENT MODE                                             │
│     • DMA only uses bus when CPU doesn't need it               │
│     • Slowest for DMA but doesn't affect CPU                   │
│     • Best CPU performance                                      │
│                                                                  │
│     Bus: CPU─CPU─CPU─ ─ ─DMA─CPU─CPU─ ─ ─DMA (gaps)           │
│                       ↑                                         │
│                   Bus idle                                       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Advantages and Disadvantages

```
ADVANTAGES:
✓ Minimal CPU involvement
✓ Efficient for large data transfers
✓ CPU can do other work during transfer
✓ Faster than CPU moving data byte-by-byte

DISADVANTAGES:
✗ Requires DMA controller hardware
✗ Complex memory management (cache coherence)
✗ Initial setup overhead (not worth for small transfers)
✗ Bus contention with CPU

When used:
• Disk I/O (reading/writing large files)
• Network I/O (large packets)
• Audio/Video streaming
• Any large data transfer
```

---

## Comparison of I/O Methods

```
┌──────────────────────────────────────────────────────────────────┐
│                COMPARISON OF I/O METHODS                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────┬────────────┬─────────────┬──────────────┐   │
│  │ Aspect         │ Polling    │ Interrupt   │ DMA          │   │
│  ├────────────────┼────────────┼─────────────┼──────────────┤   │
│  │ CPU usage      │ Very High  │ Medium      │ Low          │   │
│  │ during I/O     │ (busy wait)│ (per event) │ (setup only) │   │
│  ├────────────────┼────────────┼─────────────┼──────────────┤   │
│  │ Best for       │ Fast dev   │ Slow dev    │ Large xfers  │   │
│  │                │ Simple sys │ Most I/O    │ Bulk data    │   │
│  ├────────────────┼────────────┼─────────────┼──────────────┤   │
│  │ Hardware       │ Simple     │ Int. ctrl   │ DMA ctrl     │   │
│  │ complexity     │            │ needed      │ needed       │   │
│  ├────────────────┼────────────┼─────────────┼──────────────┤   │
│  │ Software       │ Simple     │ Handlers    │ Complex      │   │
│  │ complexity     │ loop       │ needed      │ setup        │   │
│  ├────────────────┼────────────┼─────────────┼──────────────┤   │
│  │ Latency        │ Low        │ Handler     │ Setup +      │   │
│  │                │            │ overhead    │ interrupt    │   │
│  └────────────────┴────────────┴─────────────┴──────────────┘   │
│                                                                  │
│  Common combination:                                             │
│  • Polling for very fast operations                             │
│  • Interrupts for event notification                            │
│  • DMA for actual data transfer                                 │
│                                                                  │
│  Example: Disk read                                              │
│  1. CPU initiates read, sets up DMA                            │
│  2. DMA transfers data from disk to memory                     │
│  3. Interrupt signals completion                                │
│  4. CPU processes the data                                      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## I/O Software Layers

```
┌──────────────────────────────────────────────────────────────────┐
│                   I/O SOFTWARE LAYERS                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │              User-Level I/O Software                        │ │
│  │       (printf, scanf, file I/O libraries)                   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                   │
│                              ▼                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │           Device-Independent OS Software                    │ │
│  │    (naming, protection, blocking, buffering, caching)      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                   │
│                              ▼                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                   Device Drivers                            │ │
│  │      (device-specific code, registers, commands)           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                   │
│                              ▼                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                Interrupt Handlers                           │ │
│  │          (respond to hardware interrupts)                   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              │                                   │
│                              ▼                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                     Hardware                                │ │
│  │               (actual devices)                              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Method | How It Works | Best For |
|--------|--------------|----------|
| **Polling** | CPU repeatedly checks status | Fast devices, simple systems |
| **Interrupts** | Device signals CPU when ready | General I/O, multiple devices |
| **DMA** | Controller transfers data directly | Large data transfers |

---

## Quick Revision Questions

1. Why is polling inefficient for slow devices?
2. Describe the steps in interrupt handling.
3. How does DMA reduce CPU overhead?
4. What is the difference between burst mode and cycle stealing in DMA?
5. When would you use each I/O method?

---

[← Previous: I/O Hardware](./01-io-hardware.md) | [Next: Disk Scheduling →](./03-disk-scheduling.md)
