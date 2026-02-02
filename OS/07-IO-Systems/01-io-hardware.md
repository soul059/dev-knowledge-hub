# Chapter 7.1: I/O Hardware

## Overview

I/O (Input/Output) devices allow the computer to communicate with the external world. This chapter covers the hardware aspects of I/O systems, including device types, controllers, and how the CPU communicates with devices.

---

## I/O Device Categories

```
┌──────────────────────────────────────────────────────────────────┐
│                 I/O DEVICE CATEGORIES                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  BY FUNCTION:                                                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Human-Readable     │ Keyboard, mouse, monitor, printer  │   │
│  │ Machine-Readable   │ Disk, tape, sensors, controllers   │   │
│  │ Communication      │ Network cards, modems, Bluetooth   │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  BY DATA TRANSFER:                                               │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Block Devices      │ Data in fixed-size blocks          │   │
│  │                    │ Random access, seekable            │   │
│  │                    │ Examples: HDD, SSD, USB drive      │   │
│  ├────────────────────┼─────────────────────────────────────┤   │
│  │ Character Devices  │ Stream of characters               │   │
│  │                    │ Sequential, not seekable           │   │
│  │                    │ Examples: Keyboard, mouse, printer │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  BY SPEED:                                                       │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Device          │ Data Rate                             │   │
│  ├──────────────────┼───────────────────────────────────────┤   │
│  │ Keyboard        │ 10 bytes/sec                          │   │
│  │ Mouse           │ 100 bytes/sec                         │   │
│  │ USB 2.0         │ 60 MB/sec                             │   │
│  │ Gigabit Ethernet│ 125 MB/sec                            │   │
│  │ USB 3.0         │ 625 MB/sec                            │   │
│  │ SSD (SATA)      │ 600 MB/sec                            │   │
│  │ SSD (NVMe)      │ 3,500 MB/sec                          │   │
│  └──────────────────┴───────────────────────────────────────┘   │
│                                                                  │
│  Challenge: OS must handle huge range of speeds and types!      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Device Controllers

```
┌──────────────────────────────────────────────────────────────────┐
│                   DEVICE CONTROLLER                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Devices don't connect directly to CPU                           │
│  They connect through CONTROLLERS (adapters)                     │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                                                             ││
│  │    ┌─────┐     ┌──────────────────┐     ┌───────────────┐  ││
│  │    │ CPU │←───→│       BUS        │←───→│  Controller   │  ││
│  │    └─────┘     └──────────────────┘     └───────┬───────┘  ││
│  │                                                  │          ││
│  │                                                  ▼          ││
│  │                                          ┌──────────────┐   ││
│  │                                          │    Device    │   ││
│  │                                          │  (HDD, etc)  │   ││
│  │                                          └──────────────┘   ││
│  │                                                             ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
│  Controller components:                                          │
│  • Data register: Buffer for data transfer                      │
│  • Status register: Device state (busy, ready, error)           │
│  • Command register: Instructions to device                     │
│  • Internal logic: Converts bus signals to device signals       │
│                                                                  │
│  Examples:                                                       │
│  • SATA controller for hard drives                              │
│  • USB controller for USB devices                               │
│  • NIC (Network Interface Card) for network                    │
│  • GPU for display                                              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## I/O Ports and Registers

```
┌──────────────────────────────────────────────────────────────────┐
│                I/O PORTS AND REGISTERS                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Controller presents registers to CPU:                           │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    CONTROLLER                             │   │
│  │  ┌─────────────────────────────────────────────────────┐ │   │
│  │  │ STATUS REGISTER                                      │ │   │
│  │  │ ┌───┬───┬───┬───┬───┬───┬───┬───┐                   │ │   │
│  │  │ │ 0 │ 1 │ 0 │ 0 │ 0 │ 0 │ 1 │ 0 │                   │ │   │
│  │  │ └───┴───┴───┴───┴───┴───┴───┴───┘                   │ │   │
│  │  │   ↑                           ↑                      │ │   │
│  │  │  Busy                       Error                    │ │   │
│  │  └─────────────────────────────────────────────────────┘ │   │
│  │  ┌─────────────────────────────────────────────────────┐ │   │
│  │  │ COMMAND REGISTER                                     │ │   │
│  │  │   Write operation code (read, write, seek, etc.)    │ │   │
│  │  └─────────────────────────────────────────────────────┘ │   │
│  │  ┌─────────────────────────────────────────────────────┐ │   │
│  │  │ DATA REGISTER                                        │ │   │
│  │  │   Buffer for data being transferred                  │ │   │
│  │  └─────────────────────────────────────────────────────┘ │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  Common status bits:                                             │
│  • Busy: Controller is processing command                       │
│  • Ready: Controller ready for new command                      │
│  • Error: Something went wrong                                  │
│  • Data-ready: Data available in data register                  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Accessing I/O Devices

```
┌──────────────────────────────────────────────────────────────────┐
│              METHODS TO ACCESS I/O DEVICES                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. PORT-MAPPED I/O (Isolated I/O)                              │
│     • Separate I/O address space                                │
│     • Special I/O instructions: IN, OUT                         │
│     • Example: x86 architecture                                 │
│                                                                  │
│     in al, 0x60      ; Read from keyboard port                  │
│     out 0x21, al     ; Write to interrupt controller            │
│                                                                  │
│     Memory:     0x00000000 ────────────────── 0xFFFFFFFF        │
│     I/O Ports:  0x0000 ─────────────────────── 0xFFFF           │
│                 (separate address space)                         │
│                                                                  │
│  ─────────────────────────────────────────────────────────────  │
│                                                                  │
│  2. MEMORY-MAPPED I/O                                            │
│     • Device registers mapped to memory addresses               │
│     • Use regular load/store instructions                       │
│     • No special I/O instructions needed                        │
│                                                                  │
│     Memory space:                                                │
│     0x00000000 ┌──────────────────────┐                         │
│                │        RAM           │                         │
│     0x80000000 ├──────────────────────┤                         │
│                │    Device Memory     │ ← Controller registers  │
│     0xA0000000 ├──────────────────────┤                         │
│                │    More RAM          │                         │
│     0xFFFFFFFF └──────────────────────┘                         │
│                                                                  │
│     // Read device status                                       │
│     status = *(volatile int*)0x80001000;                        │
│                                                                  │
│  Modern systems: Often use BOTH methods                         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Hard Disk Structure

```
┌──────────────────────────────────────────────────────────────────┐
│                   HARD DISK STRUCTURE                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │           Physical Structure                                │ │
│  │                                                             │ │
│  │       Spindle                                               │ │
│  │          │                                                  │ │
│  │    ┌─────┴─────┐                                           │ │
│  │    │    ○○○○○  │ ← Platters (magnetic disks)              │ │
│  │    │    ○○○○○  │   stacked on spindle                     │ │
│  │    │    ○○○○○  │                                          │ │
│  │    └───────────┘                                           │ │
│  │          ↑                                                  │ │
│  │         Head (read/write) on arm                           │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │           Logical Structure                                 │ │
│  │                                                             │ │
│  │            ┌─────────────────────┐                         │ │
│  │           ╱│   Track 0          │╲                        │ │
│  │          ╱ │   Track 1          │ ╲                       │ │
│  │         ╱  │   Track 2          │  ╲  ← Concentric        │ │
│  │        ╱   │   ...              │   ╲   rings             │ │
│  │       ╱    │                    │    ╲                    │ │
│  │      ╱     └────────────────────┘     ╲                   │ │
│  │             Sector (smallest unit)                         │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  TERMS:                                                          │
│  • Platter: Circular magnetic surface                           │
│  • Track: Concentric circle on platter                          │
│  • Sector: Arc of a track (typically 512B or 4KB)              │
│  • Cylinder: Same track on all platters                         │
│  • Head: Read/write mechanism (one per surface)                 │
│                                                                  │
│  Access time = Seek time + Rotational latency + Transfer time  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Solid State Drives (SSD)

```
┌──────────────────────────────────────────────────────────────────┐
│               SOLID STATE DRIVES (SSD)                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  No moving parts (unlike HDD)                                    │
│  Uses NAND flash memory                                          │
│                                                                  │
│  Structure:                                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                       SSD Controller                        │ │
│  │ ┌───────────┬───────────┬───────────┬───────────┐         │ │
│  │ │   NAND    │   NAND    │   NAND    │   NAND    │  ← Chips│ │
│  │ │   Chip    │   Chip    │   Chip    │   Chip    │         │ │
│  │ └─────┬─────┴─────┬─────┴─────┬─────┴─────┬─────┘         │ │
│  │       │           │           │           │                │ │
│  │ ┌─────┴───────────┴───────────┴───────────┴─────┐         │ │
│  │ │              Flash Translation Layer          │         │ │
│  │ │              (Logical → Physical mapping)     │         │ │
│  │ └───────────────────────────────────────────────┘         │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  Key differences from HDD:                                       │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │ Aspect         │ HDD            │ SSD                     │  │
│  ├────────────────┼────────────────┼─────────────────────────┤  │
│  │ Seek time      │ 5-10 ms        │ ~0 (no seeking)        │  │
│  │ Random access  │ Slow           │ Fast                    │  │
│  │ Sequential     │ Good           │ Excellent               │  │
│  │ Power          │ Higher         │ Lower                   │  │
│  │ Durability     │ Moving parts   │ Limited write cycles   │  │
│  │ Cost per GB    │ Lower          │ Higher                  │  │
│  └────────────────┴────────────────┴─────────────────────────┘  │
│                                                                  │
│  SSD challenges:                                                 │
│  • Write amplification                                          │
│  • Wear leveling (distribute writes evenly)                    │
│  • TRIM command (notify of deleted blocks)                     │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Bus Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                    BUS ARCHITECTURE                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Buses connect CPU, memory, and I/O devices                      │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │    ┌─────┐                 ┌──────────┐                    │ │
│  │    │ CPU │                 │  Memory  │                    │ │
│  │    └──┬──┘                 └────┬─────┘                    │ │
│  │       │                         │                          │ │
│  │       └────────────┬────────────┘                          │ │
│  │                    │                                        │ │
│  │    ═══════════════════════════════════  System Bus (FSB)   │ │
│  │                    │                                        │ │
│  │              ┌─────┴─────┐                                  │ │
│  │              │  Chipset  │                                  │ │
│  │              │ (North/   │                                  │ │
│  │              │  South)   │                                  │ │
│  │              └─────┬─────┘                                  │ │
│  │                    │                                        │ │
│  │    ═══════════════════════════════════  I/O Bus             │ │
│  │        │       │        │        │                          │ │
│  │     ┌──┴──┐ ┌──┴──┐ ┌───┴───┐ ┌──┴──┐                     │ │
│  │     │PCIe │ │ USB │ │ SATA  │ │ NIC │                     │ │
│  │     │GPU  │ │     │ │ Disk  │ │     │                     │ │
│  │     └─────┘ └─────┘ └───────┘ └─────┘                     │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  Common bus types:                                               │
│  • PCIe: High-speed expansion (GPU, NVMe SSD)                   │
│  • USB: Universal Serial Bus (peripherals)                      │
│  • SATA: Storage devices                                        │
│  • Thunderbolt: High-speed, daisy-chainable                    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## RAID Levels

```
┌──────────────────────────────────────────────────────────────────┐
│                      RAID LEVELS                                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  RAID: Redundant Array of Independent Disks                     │
│  Combine multiple disks for performance and/or reliability      │
│                                                                  │
│  RAID 0 (Striping):                                              │
│  ┌─────┐ ┌─────┐    Data split across disks                    │
│  │ A1  │ │ A2  │    + Fast (parallel reads/writes)             │
│  │ A3  │ │ A4  │    - No redundancy (one disk fails = all lost)│
│  └─────┘ └─────┘                                                │
│                                                                  │
│  RAID 1 (Mirroring):                                             │
│  ┌─────┐ ┌─────┐    Identical copies                           │
│  │ A1  │ │ A1  │    + Redundancy (survives one disk failure)   │
│  │ A2  │ │ A2  │    - 50% storage efficiency                   │
│  └─────┘ └─────┘                                                │
│                                                                  │
│  RAID 5 (Striping with parity):                                  │
│  ┌─────┐ ┌─────┐ ┌─────┐   Parity distributed across disks     │
│  │ A1  │ │ A2  │ │ Ap  │   + Good performance + redundancy     │
│  │ B1  │ │ Bp  │ │ B2  │   + Efficient (N-1 usable capacity)   │
│  │ Cp  │ │ C1  │ │ C2  │   - Slow rebuild                      │
│  └─────┘ └─────┘ └─────┘                                        │
│                                                                  │
│  RAID 6 (Double parity):                                         │
│  Like RAID 5 but survives 2 disk failures                       │
│                                                                  │
│  RAID 10 (1+0):                                                  │
│  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐   Mirror + Stripe             │
│  │ A1  │ │ A1  │ │ A2  │ │ A2  │   + Fast + redundant          │
│  │ A3  │ │ A3  │ │ A4  │ │ A4  │   - 50% efficiency            │
│  └─────┘ └─────┘ └─────┘ └─────┘                                │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **Block Device** | Fixed-size blocks, random access (disk) |
| **Character Device** | Stream of characters (keyboard) |
| **Controller** | Interface between device and bus |
| **Port-Mapped I/O** | Separate I/O address space, IN/OUT instructions |
| **Memory-Mapped I/O** | Device registers in memory space |
| **Sector** | Smallest addressable unit on disk |
| **SSD** | Flash-based storage, no moving parts |
| **RAID** | Multiple disks for performance/reliability |

---

## Quick Revision Questions

1. What is the difference between block and character devices?
2. Explain port-mapped vs memory-mapped I/O.
3. What are the components of disk access time?
4. How does an SSD differ from an HDD internally?
5. What is RAID 5 and how does it provide fault tolerance?

---

[← Previous: File System Implementation](../06-File-Systems/05-file-system-implementation.md) | [Next: I/O Methods →](./02-io-methods.md)
