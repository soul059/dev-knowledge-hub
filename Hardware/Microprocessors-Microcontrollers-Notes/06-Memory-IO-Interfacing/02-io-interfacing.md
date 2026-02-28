# Chapter 6.2: I/O Interfacing Techniques

## 📚 Chapter Overview

This chapter covers the methods of connecting I/O devices to microprocessors. We'll explore memory-mapped I/O and I/O-mapped (peripheral-mapped) I/O techniques, along with various data transfer methods used in microprocessor systems.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Differentiate between memory-mapped and I/O-mapped I/O
- Design address decoding for I/O devices
- Understand programmed, interrupt-driven, and DMA data transfer
- Choose appropriate I/O method for different applications

---

## 1. I/O Addressing Methods

### 1.1 Two Approaches

```
┌─────────────────────────────────────────────────────────────────────┐
│                    I/O ADDRESSING METHODS                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌─────────────────────────┐    ┌─────────────────────────┐       │
│   │   MEMORY-MAPPED I/O     │    │    I/O-MAPPED I/O       │       │
│   │   (Isolated I/O)        │    │  (Peripheral-Mapped)    │       │
│   │                         │    │                         │       │
│   │ • I/O devices assigned  │    │ • Separate I/O address  │       │
│   │   memory addresses      │    │   space                 │       │
│   │ • Uses memory R/W       │    │ • Uses IN/OUT           │       │
│   │   instructions          │    │   instructions          │       │
│   │ • Same address space    │    │ • IO/M̄ signal used      │       │
│   │   for memory & I/O      │    │   for selection         │       │
│   │                         │    │                         │       │
│   │ Examples: 6800, ARM     │    │ Examples: 8085, 8086    │       │
│   └─────────────────────────┘    └─────────────────────────┘       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.2 Memory-Mapped I/O

```
MEMORY-MAPPED I/O
─────────────────

Address Space:
┌────────────────────────────────────┐
│ FFFFH                              │
│   ↑                                │
│   │    I/O Devices                 │ ← Shares address space
│   │    (Mapped to addresses)       │
│ F000H                              │
├────────────────────────────────────┤
│ EFFFH                              │
│   ↑                                │
│   │    RAM                         │
│   │                                │
├────────────────────────────────────┤
│   ↑                                │
│   │    ROM                         │
│ 0000H                              │
└────────────────────────────────────┘

Advantages:
✓ All memory instructions work with I/O
✓ More powerful addressing modes
✓ Can use arithmetic/logic on I/O data directly
✓ No special I/O instructions needed

Disadvantages:
✗ Reduces available memory space
✗ Longer instruction execution
✗ Address decoding more complex
```

### 1.3 I/O-Mapped I/O (Peripheral-Mapped)

```
I/O-MAPPED I/O (8085/8086)
──────────────────────────

Separate Address Spaces:

    MEMORY SPACE              I/O SPACE
    ────────────              ─────────
    (IO/M̄ = 0)               (IO/M̄ = 1)
    
    FFFFH ┌──────┐           FFH ┌──────┐
          │      │               │ I/O  │
          │ RAM  │               │Device│
          │      │               │Ports │
          ├──────┤           00H └──────┘
          │      │
          │ ROM  │           8085: 256 ports
          │      │           8086: 65536 ports
    0000H └──────┘

8085 I/O Instructions:
┌────────────────────────────────────┐
│ IN port    ; A ← [port]            │
│ OUT port   ; [port] ← A            │
│                                    │
│ Port address: 00H to FFH (8-bit)   │
└────────────────────────────────────┘

8086 I/O Instructions:
┌────────────────────────────────────┐
│ IN AL, port    ; AL ← [port]       │
│ IN AX, port    ; AX ← [port]       │
│ OUT port, AL   ; [port] ← AL       │
│ OUT port, AX   ; [port] ← AX       │
│                                    │
│ IN AL, DX      ; Indirect port     │
│ OUT DX, AL     ; (DX = port addr)  │
│                                    │
│ Fixed port: 00H-FFH (8-bit)        │
│ Variable port: 0000H-FFFFH (16-bit)│
└────────────────────────────────────┘
```

### 1.4 Comparison Table

| Feature | Memory-Mapped I/O | I/O-Mapped I/O |
|---------|-------------------|----------------|
| Address Space | Shared with memory | Separate I/O space |
| Instructions | MOV, ADD, etc. | IN, OUT only |
| Addressing Modes | All available | Direct, Indirect |
| Signal Used | MEMR̄, MEMW̄ | IOR̄, IOW̄ |
| 8085 Control | IO/M̄ = 0 | IO/M̄ = 1 |
| Address Lines | Full 16 bits | 8 bits (A0-A7) |
| Decoding | Complex | Simpler |
| Memory Space | Reduced | Full available |
| Flexibility | Higher | Lower |

---

## 2. I/O Address Decoding for 8085

### 2.1 8085 I/O Control Signals

```
8085 I/O Signal Generation:
───────────────────────────

From IO/M̄, RD̄, and WR̄ signals:

                    ┌─────────┐
        IO/M̄ ───────┤         │
                    │   AND   ├───── IOR̄ (I/O Read)
         RD̄ ────────┤         │
                    └─────────┘

                    ┌─────────┐
        IO/M̄ ───────┤         │
                    │   AND   ├───── IOW̄ (I/O Write)
         WR̄ ────────┤         │
                    └─────────┘

Signal States:
┌────────┬────────┬────────┬────────────────────┐
│ IO/M̄   │  RD̄    │  WR̄    │ Operation          │
├────────┼────────┼────────┼────────────────────┤
│   0    │   0    │   1    │ Memory Read        │
│   0    │   1    │   0    │ Memory Write       │
│   1    │   0    │   1    │ I/O Read (IN)      │
│   1    │   1    │   0    │ I/O Write (OUT)    │
└────────┴────────┴────────┴────────────────────┘
```

### 2.2 Simple I/O Port Decoding

```
Decode Port Address 80H for 8085:
─────────────────────────────────

Port 80H = 1000 0000 binary

Using 74LS138 for decoding:

     8085              74LS138
    ┌─────┐          ┌───────────┐
    │ A5  ├──────────┤ A      Y0 ├─── Port 00H
    │ A6  ├──────────┤ B      Y1 ├─── Port 20H
    │ A7  ├──────────┤ C      Y2 ├─── Port 40H
    │     │          │        Y3 ├─── Port 60H
    │IO/M̄ ├──────────┤ G1     Y4 ├─── Port 80H ← CS for our device
    │ A4  ├───NOT────┤ G2A    Y5 ├─── Port A0H
    │ A3  ├───NOT────┤ G2B    Y6 ├─── Port C0H
    │     │          │        Y7 ├─── Port E0H
    └─────┘          └───────────┘

For Port 80H:
- A7 A6 A5 = 100 → Y4 selected
- A4 A3 = 00 → G2A, G2B enabled
- IO/M̄ = 1 → G1 enabled during I/O operation
```

### 2.3 Using 74LS138 for 8 Ports

```
Decoding 8 Consecutive Ports (00H-07H):
───────────────────────────────────────

                  74LS138
               ┌───────────┐
      A0 ──────┤ A      Y0 ├─── Port 00H
      A1 ──────┤ B      Y1 ├─── Port 01H
      A2 ──────┤ C      Y2 ├─── Port 02H
               │        Y3 ├─── Port 03H
     IO/M̄ ──────┤ G1     Y4 ├─── Port 04H
               │        Y5 ├─── Port 05H
       ┌───────┤ G2A    Y6 ├─── Port 06H
       │   ┌───┤ G2B    Y7 ├─── Port 07H
       │   │   └───────────┘
       │   │
    A7-A3 all connected through AND gate to verify = 0
```

---

## 3. I/O Address Decoding for 8086

### 3.1 8086 I/O Signals

```
8086 Minimum Mode I/O Signals:
──────────────────────────────

From M/IO̅, RD̄, and WR̄:

M/IO̅ = 0 → I/O operation
M/IO̅ = 1 → Memory operation

IOR̄ = M/IO̅' AND RD̄' (active when reading I/O)
IOW̄ = M/IO̅' AND WR̄' (active when writing I/O)

8086 Maximum Mode:
Uses 8288 Bus Controller to generate:
- IORC̄ (I/O Read Command)
- IOWC̄ (I/O Write Command)
- AIOWC̄ (Advanced I/O Write Command)
```

### 3.2 8086 Port Address Decoding

```
Decode Port 0300H for 8086:
───────────────────────────

Port 0300H = 0000 0011 0000 0000 binary

Need to decode A9, A8 = 11, A15-A10 = 0, A7-A0 = 0

Using GAL/PAL or discrete logic:

            ┌─────────────────────────────────┐
            │   Address Decoder Logic         │
            │                                 │
    A15 ────┤   NOR                          │
    A14 ────┤   Gate                         │
    A13 ────┤   (All                         │
    A12 ────┤   must   ────────────┐         │
    A11 ────┤   be 0)              │         │
    A10 ────┤                      ▼         │
            │               ┌────────────┐   │
     A9 ────┤───────────────┤    AND     ├───┼──── Port 0300H Select
     A8 ────┤───────────────┤            │   │
            │               │            │   │
   M/IO̅ ────┤───(NOT)───────┤            │   │
            │               └────────────┘   │
            └─────────────────────────────────┘

For Port 0300H-030FH range (16 ports):
- Ignore A0-A3 (gives 16 ports)
- Decode A4-A15
```

---

## 4. Data Transfer Methods

### 4.1 Overview

```
DATA TRANSFER METHODS
─────────────────────

┌───────────────────────────────────────────────────────────────────┐
│                                                                   │
│   ┌────────────────┐  ┌────────────────┐  ┌────────────────┐     │
│   │   PROGRAMMED   │  │   INTERRUPT    │  │      DMA       │     │
│   │      I/O       │  │   DRIVEN I/O   │  │                │     │
│   │                │  │                │  │                │     │
│   │ CPU polls      │  │ Device signals │  │ Direct data    │     │
│   │ device status  │  │ when ready     │  │ transfer       │     │
│   │                │  │                │  │ without CPU    │     │
│   └────────────────┘  └────────────────┘  └────────────────┘     │
│          │                   │                    │              │
│          ▼                   ▼                    ▼              │
│   ┌────────────┐      ┌────────────┐      ┌────────────┐        │
│   │  Polling   │      │ Vectored   │      │  Burst     │        │
│   │  Wait loop │      │ Priority   │      │  Block     │        │
│   │            │      │            │      │  Transfer  │        │
│   └────────────┘      └────────────┘      └────────────┘        │
│                                                                   │
│   Speed:  SLOW          MEDIUM              FAST                 │
│   CPU:    100%          Low overhead        Near 0%              │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘
```

### 4.2 Programmed I/O

```
PROGRAMMED I/O (Polling)
────────────────────────

CPU actively checks device status:

    ┌─────────────────────────────────────┐
    │         START                        │
    └──────────────┬──────────────────────┘
                   ▼
    ┌─────────────────────────────────────┐
    │   Read Status Port                   │
    └──────────────┬──────────────────────┘
                   ▼
           ┌──────────────┐
           │  Device      │──NO──┐
           │  Ready?      │      │
           └──────┬───────┘      │
                  │ YES          │
                  ▼              │
    ┌─────────────────────────────┐
    │   Transfer Data             │◄─────┘
    └──────────────┬──────────────┘
                   ▼
           ┌──────────────┐
           │  Transfer    │──NO──┐
           │  Complete?   │      │
           └──────┬───────┘      │
                  │ YES          │
                  ▼              │
    ┌─────────────────────────────┐
    │         END                  │
    └─────────────────────────────┘

8085 Example - Polling:
─────────────────────────
POLL:   IN  STATUS_PORT    ; Read status
        ANI READY_BIT      ; Check ready bit
        JZ  POLL           ; Loop if not ready
        IN  DATA_PORT      ; Read data
        ...

Disadvantage: CPU wastes time in polling loop
```

### 4.3 Interrupt-Driven I/O

```
INTERRUPT-DRIVEN I/O
────────────────────

Device interrupts CPU when ready:

    CPU                          Device
    ┌────────┐                  ┌────────┐
    │        │   INT Request    │        │
    │ Running├──────────────────┤ Data   │
    │ Program│                  │ Ready  │
    │        │◄─────────────────┤        │
    │        │   Interrupt!     │        │
    └───┬────┘                  └────────┘
        │
        ▼
    ┌────────────┐
    │ Save State │
    │ (PSW, PC)  │
    └─────┬──────┘
          │
          ▼
    ┌────────────┐
    │   Execute  │
    │    ISR     │
    │ (Read Data)│
    └─────┬──────┘
          │
          ▼
    ┌────────────┐
    │  Restore   │
    │   State    │
    └─────┬──────┘
          │
          ▼
    ┌────────────┐
    │  Continue  │
    │  Program   │
    └────────────┘

Advantage: CPU does useful work while waiting
```

### 4.4 Direct Memory Access (DMA)

```
DMA - Direct Memory Access
──────────────────────────

Data transfers directly between I/O and memory:

             ┌─────────────────────────────────┐
             │           System Bus            │
             └─────────────┬───────────────────┘
                           │
    ┌──────────────┬───────┴───────┬──────────────┐
    │              │               │              │
    ▼              ▼               ▼              ▼
┌──────┐      ┌──────┐        ┌──────┐      ┌──────┐
│ CPU  │      │Memory│        │ DMA  │      │ I/O  │
│      │      │      │        │Contrl│      │Device│
└──────┘      └──────┘        └───┬──┘      └───┬──┘
                                  │             │
                                  └─────────────┘
                                   Direct Data
                                   Transfer

DMA Transfer Steps:
1. Device requests DMA (DREQ)
2. DMA controller requests bus (HRQ to CPU)
3. CPU releases bus (HLDA)
4. DMA controller takes bus control
5. Data transfers directly: Device ↔ Memory
6. DMA controller releases bus
7. CPU resumes operation

Speed: Can transfer at memory speed (no CPU overhead)
```

### 4.5 Comparison of Transfer Methods

| Feature | Programmed I/O | Interrupt I/O | DMA |
|---------|---------------|---------------|-----|
| CPU Involvement | 100% | Low | Near 0% |
| Speed | Slow | Medium | Very Fast |
| Complexity | Simple | Medium | Complex |
| Hardware | Minimal | Interrupt ctrl | DMA controller |
| Use Case | Simple devices | Keyboard, slow I/O | Disk, high-speed |
| Data Rate | ~10KB/s | ~100KB/s | ~10MB/s |

---

## 5. I/O Interfacing Examples

### 5.1 Simple Input Port (8085)

```
Input Port using 74LS244 Buffer:
────────────────────────────────

     Input Switches              8085
       ┌─────┐                  ┌─────┐
       │ SW0 ├───────┬──────────┤ AD0 │
       │ SW1 ├───────┤          │ AD1 │
       │ SW2 ├───────┤ 74LS244  │ AD2 │
       │ SW3 ├───────┤  Buffer  │ AD3 │
       │ SW4 ├───────┤          │ AD4 │
       │ SW5 ├───────┤          │ AD5 │
       │ SW6 ├───────┤          │ AD6 │
       │ SW7 ├───────┴──────────┤ AD7 │
       └─────┘          │       │     │
                        │       │     │
                    ┌───┴───┐   │     │
            Port    │Address│   │     │
            Select ─┤Decoder├───┤ IOR̄ │
                    └───────┘   └─────┘

74LS244 is enabled when port address matches
and IOR̄ is active
```

### 5.2 Simple Output Port (8085)

```
Output Port using 74LS374 Latch:
────────────────────────────────

        8085                    LEDs
       ┌─────┐                 ┌─────┐
       │ AD0 ├───────┬─────────┤ LED0│
       │ AD1 ├───────┤         │ LED1│
       │ AD2 ├───────┤ 74LS374 │ LED2│
       │ AD3 ├───────┤  Latch  │ LED3│
       │ AD4 ├───────┤         │ LED4│
       │ AD5 ├───────┤         │ LED5│
       │ AD6 ├───────┤         │ LED6│
       │ AD7 ├───────┴─────────┤ LED7│
       │     │          │CLK   └─────┘
       │ IOW̄ ├──────────┤
       │     │     ┌────┴────┐
       └─────┘     │ Address │
                   │ Decoder │
                   └─────────┘

74LS374 latches data on IOW̄ rising edge
when port address is decoded
```

### 5.3 Bidirectional Port (8085)

```
Bidirectional Port using 74LS245:
─────────────────────────────────

        8085                  74LS245                Device
       ┌─────┐               ┌───────┐               ┌─────┐
       │ AD0 ├───────────────┤ A0 B0 ├───────────────┤ D0  │
       │ AD1 ├───────────────┤ A1 B1 ├───────────────┤ D1  │
       │ AD2 ├───────────────┤ A2 B2 ├───────────────┤ D2  │
       │ AD3 ├───────────────┤ A3 B3 ├───────────────┤ D3  │
       │ AD4 ├───────────────┤ A4 B4 ├───────────────┤ D4  │
       │ AD5 ├───────────────┤ A5 B5 ├───────────────┤ D5  │
       │ AD6 ├───────────────┤ A6 B6 ├───────────────┤ D6  │
       │ AD7 ├───────────────┤ A7 B7 ├───────────────┤ D7  │
       │     │               │       │               │     │
       │ RD̄  ├───────────────┤ DIR   │               └─────┘
       │     │               │       │
       │     │    ┌──────────┤ OĒ    │
       └─────┘    │          └───────┘
                  │
             Port Select

DIR = 0: B → A (Input to CPU)
DIR = 1: A → B (Output from CPU)
```

---

## 6. Handshaking

### 6.1 Concept

```
HANDSHAKING - Synchronized Data Transfer
────────────────────────────────────────

Without Handshaking:
- Sender sends whenever ready
- Receiver may miss data
- Data can be lost or duplicated

With Handshaking:
- Sender and receiver coordinate
- Control signals ensure proper transfer
- No data loss

Two Types:
1. Strobe Handshaking (single control line)
2. Full Handshaking (two control lines)
```

### 6.2 Strobe Handshaking

```
STROBE (Single Control Line)
────────────────────────────

Input Strobe:
                    ┌─────┐
Device sends data ──┤     ├────── Microprocessor
                    │Data │       reads data
    ┌───────────────┤     │
    │ STB (Strobe)  └─────┘
    ▼
    
    STB  ────────┐      ┌────────
                 └──────┘
                    │
    Data ────────XXXXXXXXXXXXX────────
                 ◄─────────────►
                  Data Valid

Output Strobe:
                    ┌─────┐
Microprocessor ─────┤     ├────── Device
sends data          │Data │       reads data
                    │     ├───────┐
                    └─────┘       │ ACK
                                  ▼
```

### 6.3 Full Handshaking

```
FULL HANDSHAKING (Two Control Lines)
────────────────────────────────────

Input Handshaking:
                         
       Device                    Microprocessor
    ┌──────────┐               ┌──────────┐
    │          │──── Data ────►│          │
    │          │               │          │
    │          │──── IBF  ────►│          │
    │          │               │ (Input   │
    │          │◄──── STB ─────┤  Buffer  │
    └──────────┘               │  Full)   │
                               └──────────┘

Timing:
    STB  ────────┐      ┌────────────────
                 └──────┘
                    │
    Data ───────XXXXXXXXXXXXXXXX──────────
                    │
    IBF  ───────────┘     ┌───────────────
                          └───────────────
                          (CPU reads data)

Output Handshaking:
    OBF  ────┐                ┌───────────
             └────────────────┘
                     │
    Data ──────XXXXXXXXXXXXXX───────────
                     │
    ACK  ────────────┘       ┌────────────
                             └────────────
                         (Device takes data)
```

---

## 📋 Summary Table

| Topic | Key Points |
|-------|------------|
| Memory-Mapped I/O | I/O uses memory addresses, all instructions work |
| I/O-Mapped I/O | Separate I/O space, uses IN/OUT instructions |
| Programmed I/O | CPU polls device, wastes cycles |
| Interrupt I/O | Device signals CPU, efficient |
| DMA | Direct transfer, fastest, complex hardware |
| Handshaking | Synchronizes sender and receiver |

---

## ❓ Quick Revision Questions

1. **What is the difference between IO/M̄ and M/IO̅ signals?**
   <details>
   <summary>Show Answer</summary>
   IO/M̄ is used by 8085: IO/M̄=1 for I/O, IO/M̄=0 for memory. M/IO̅ is used by 8086: M/IO̅=0 for I/O, M/IO̅=1 for memory. They are inverted versions of each other!
   </details>

2. **Why is memory-mapped I/O more flexible than I/O-mapped?**
   <details>
   <summary>Show Answer</summary>
   Memory-mapped I/O can use ALL memory instructions (MOV, ADD, SUB, AND, OR, etc.) and ALL addressing modes. I/O-mapped only has IN and OUT instructions with limited addressing (direct port or DX register).
   </details>

3. **What happens during a DMA transfer?**
   <details>
   <summary>Show Answer</summary>
   1) Device requests DMA (DREQ), 2) DMA controller requests bus from CPU (HRQ), 3) CPU grants bus (HLDA) and tristates its buses, 4) DMA controller takes control and transfers data directly between I/O and memory, 5) DMA releases bus, 6) CPU resumes.
   </details>

4. **Why is handshaking necessary for I/O transfers?**
   <details>
   <summary>Show Answer</summary>
   Handshaking synchronizes data transfer between devices with different speeds. Without it, fast sender might send data before slow receiver is ready (data loss), or receiver might read same data twice (duplication). Handshaking ensures reliable transfer.
   </details>

5. **How many I/O ports can 8085 address vs 8086?**
   <details>
   <summary>Show Answer</summary>
   8085: 256 ports (8-bit address A0-A7). 8086: 65536 ports with variable addressing (16-bit using DX register) or 256 ports with fixed addressing (8-bit immediate).
   </details>

6. **What are IOR̄ and IOW̄ signals and how are they generated?**
   <details>
   <summary>Show Answer</summary>
   IOR̄ (I/O Read) and IOW̄ (I/O Write) are control signals for I/O operations. For 8085: IOR̄ = IO/M̄ AND RD̄', IOW̄ = IO/M̄ AND WR̄'. They indicate when CPU is reading from or writing to an I/O port.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [6.1 Memory Interfacing](01-memory-interfacing.md) | [Unit 6 Index](README.md) | [6.3 8255 PPI](03-8255-ppi.md) |

---

*[← Previous: Memory Interfacing](01-memory-interfacing.md) | [Next: 8255 PPI →](03-8255-ppi.md)*
