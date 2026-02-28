# Chapter 6.1: Memory Interfacing

## 📚 Chapter Overview

Memory interfacing is the process of connecting memory devices to a microprocessor system. This chapter covers memory types, organization, address decoding techniques, and practical interfacing circuits for both 8085 and 8086 microprocessors.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Understand different types of memory devices
- Design address decoding circuits
- Calculate memory address ranges
- Interface RAM and ROM with 8085/8086
- Create memory maps for systems

---

## 1. Types of Memory

### 1.1 Classification

```
                        MEMORY
                           │
           ┌───────────────┴───────────────┐
           │                               │
     PRIMARY MEMORY                  SECONDARY MEMORY
     (Main Memory)                   (Storage Devices)
           │
    ┌──────┴──────┐
    │             │
   RAM           ROM
(Read/Write)  (Read Only)
    │             │
┌───┴───┐    ┌────┴────────────────────┐
│       │    │         │       │       │
SRAM  DRAM  PROM    EPROM   EEPROM  Flash
│       │
│   Needs    Programmable  UV      Electrically
│   Refresh  Once          Erasable Erasable
│
Static
No Refresh
```

### 1.2 Memory Comparison Table

| Type | Volatile | Erasable | Speed | Cost | Usage |
|------|----------|----------|-------|------|-------|
| SRAM | Yes | N/A | Very Fast | High | Cache |
| DRAM | Yes | N/A | Fast | Low | Main Memory |
| ROM | No | No | Fast | Low | BIOS, Firmware |
| PROM | No | No | Fast | Medium | One-time program |
| EPROM | No | UV Light | Fast | Medium | Development |
| EEPROM | No | Electrical | Medium | High | Parameters |
| Flash | No | Electrical | Fast | Medium | Storage, BIOS |

### 1.3 Memory Chip Parameters

```
Key Parameters:
┌─────────────────────────────────────────────────────┐
│ 1. Capacity: Size in bits or bytes                  │
│    - 2716: 2K × 8 = 16 Kbits = 2KB                 │
│    - 6264: 8K × 8 = 64 Kbits = 8KB                 │
│                                                     │
│ 2. Organization: Width × Depth                      │
│    - 2K × 8: 2048 locations, 8 bits each           │
│                                                     │
│ 3. Access Time: Time to read/write data            │
│    - Typical: 70ns to 200ns                        │
│                                                     │
│ 4. Chip Enable (CE): Activates the chip            │
│    - Active LOW or Active HIGH                      │
│                                                     │
│ 5. Output Enable (OE): Enables data output         │
│                                                     │
│ 6. Write Enable (WE): Enables write operation      │
└─────────────────────────────────────────────────────┘
```

---

## 2. Memory Organization

### 2.1 8085 Memory Space

```
        8085 MEMORY MAP (64KB)
        ───────────────────────
        
FFFFH ┌───────────────────────┐
      │                       │
      │    RAM Area           │
      │    (Read/Write)       │
      │                       │
8000H ├───────────────────────┤
      │                       │
      │    ROM Area           │
      │    (Read Only)        │
      │                       │
0000H └───────────────────────┘

Address Bus: 16 bits (A0-A15)
Addressable Space: 2^16 = 65,536 = 64KB
```

### 2.2 8086 Memory Space

```
        8086 MEMORY MAP (1MB)
        ─────────────────────
        
FFFFFH ┌───────────────────────┐
       │   Reset Vector        │
       │   (FFFF0H - FFFFFH)   │
       ├───────────────────────┤
       │                       │
       │   ROM/EPROM Area      │
       │   (Top of Memory)     │
       │                       │
       ├───────────────────────┤
       │                       │
       │   RAM Area            │
       │   (User Programs)     │
       │                       │
       ├───────────────────────┤
       │   DOS/System Area     │
       │                       │
00400H ├───────────────────────┤
       │   Interrupt Vector    │
       │   Table (1KB)         │
00000H └───────────────────────┘

Address Bus: 20 bits (A0-A19)
Addressable Space: 2^20 = 1,048,576 = 1MB
```

---

## 3. Address Decoding

### 3.1 Why Address Decoding?

```
Problem:
- Multiple memory chips share same data/address bus
- Only ONE chip should respond at a time
- Each chip needs unique address range

Solution: Address Decoding
- Higher address lines select which chip
- Lower address lines select location within chip

Example: 64KB system with 4 × 16KB chips
┌────────────────────────────────────────────┐
│  A15  A14  │  Address Range  │   Chip      │
│────────────┼─────────────────┼─────────────│
│   0    0   │  0000H - 3FFFH  │   Chip 0    │
│   0    1   │  4000H - 7FFFH  │   Chip 1    │
│   1    0   │  8000H - BFFFH  │   Chip 2    │
│   1    1   │  C000H - FFFFH  │   Chip 3    │
└────────────────────────────────────────────┘
```

### 3.2 Address Decoding Methods

#### Method 1: Using Logic Gates

```
Example: Decode 8000H-FFFFH for 8085

Address Lines: A15 = 1 (for 8000H-FFFFH range)

Simple Decoder:
                    ┌─────┐
         A15 ───────┤     │
                    │ NOT ├────→ CS̄ (to memory chip)
                    │     │     (Active when A15=1)
                    └─────┘

For A15=1: NOT(1) = 0 → CS̄ active (chip selected)
For A15=0: NOT(0) = 1 → CS̄ inactive (chip not selected)
```

#### Method 2: Using NAND Gates

```
Example: Decode C000H-FFFFH (A15=1, A14=1)

                    ┌─────┐
         A15 ───────┤     │
                    │NAND ├────→ CS̄
         A14 ───────┤     │
                    └─────┘

When A15=1 AND A14=1: Output = 0 → Chip Selected
Otherwise: Output = 1 → Chip Not Selected
```

#### Method 3: Using 74LS138 (3-to-8 Decoder)

```
                74LS138 Decoder
         ┌──────────────────────────┐
         │                          │
    A ───┤ A                    Y0  ├─── CS0 (0000H-1FFFH)
    B ───┤ B                    Y1  ├─── CS1 (2000H-3FFFH)
    C ───┤ C                    Y2  ├─── CS2 (4000H-5FFFH)
         │                      Y3  ├─── CS3 (6000H-7FFFH)
   G1 ───┤ G1 (Enable)          Y4  ├─── CS4 (8000H-9FFFH)
  G2A ───┤ G2A (Enable)         Y5  ├─── CS5 (A000H-BFFFH)
  G2B ───┤ G2B (Enable)         Y6  ├─── CS6 (C000H-DFFFH)
         │                      Y7  ├─── CS7 (E000H-FFFFH)
         └──────────────────────────┘

Connections for 8085:
- A = A13
- B = A14  
- C = A15
- G1 = +5V
- G2A = GND
- G2B = GND

Each output selects 8KB block (2^13 = 8192 bytes)
```

### 3.3 Complete Decoding vs Partial Decoding

```
COMPLETE DECODING:
- Uses ALL address lines
- Each memory location has unique address
- No address overlap
- More hardware required

Example: 2KB ROM at 0000H-07FFH
- Uses A0-A10 for chip (2^11 = 2048)
- Uses A11-A15 for selection (all must be 0)

┌─────────────────────────────────┐
│ A15 A14 A13 A12 A11 │ A10-A0    │
│  0   0   0   0   0  │ Internal  │
│      Chip Select    │ Address   │
└─────────────────────────────────┘

PARTIAL DECODING:
- Uses only SOME address lines
- Multiple addresses map to same location
- Address overlap (foldback)
- Less hardware, but wastes address space

Example: Same 2KB ROM with partial decode
- Uses A0-A10 for chip
- Uses only A15 for selection

Foldback addresses:
0000H-07FFH  }
2000H-27FFH  }  All map to same
4000H-47FFH  }  2KB ROM
...          }
```

---

## 4. Memory Interfacing with 8085

### 4.1 Interfacing 2716 EPROM (2KB)

```
2716 EPROM (2K × 8)
- 11 Address lines (A0-A10)
- 8 Data lines (D0-D7)
- CĒ (Chip Enable)
- OĒ (Output Enable)

8085 to 2716 Interfacing:
                                    
     8085                2716 EPROM
    ┌─────┐              ┌─────────┐
    │     │  A0-A10      │         │
    │ A0  ├──────────────┤ A0      │
    │ ... │              │ ...     │
    │ A10 ├──────────────┤ A10     │
    │     │              │         │
    │ D0  ├──────────────┤ D0      │
    │ ... │   D0-D7      │ ...     │
    │ D7  ├──────────────┤ D7      │
    │     │              │         │
    │ A15 ├──────┬───────┤ CĒ      │ (A15=0 selects chip)
    │     │      │       │         │
    │ RD̄  ├──────┴───────┤ OĒ      │
    │     │              │         │
    └─────┘              └─────────┘
    
Address Range: 0000H to 07FFH (when A15-A11 = 00000)
```

### 4.2 Interfacing 6116 RAM (2KB)

```
6116 RAM (2K × 8)
- 11 Address lines (A0-A10)
- 8 Data lines (D0-D7)
- CĒ (Chip Enable)
- OĒ (Output Enable)
- WĒ (Write Enable)

     8085                 6116 RAM
    ┌─────┐              ┌─────────┐
    │     │  A0-A10      │         │
    │ A0  ├──────────────┤ A0      │
    │ ... │              │ ...     │
    │ A10 ├──────────────┤ A10     │
    │     │              │         │
    │ AD0 ├──────────────┤ D0      │
    │ ... │   D0-D7      │ ...     │
    │ AD7 ├──────────────┤ D7      │
    │     │              │         │
    │     │     ┌────────┤ CĒ      │
    │     │     │        │         │
    │ RD̄  ├─────┼────────┤ OĒ      │
    │     │     │        │         │
    │ WR̄  ├─────┼────────┤ WĒ      │
    │     │     │        │         │
    │ A15 ├─────┘        │         │
    │ IO/M̄├──┐           │         │
    │     │  │           │         │
    └─────┘  └───────────┤ (NAND)  │
                         └─────────┘

CĒ = A15 + IO/M̄ (NAND gate output)
Address Range: 8000H to 87FFH
```

### 4.3 Complete 8085 Memory System

```
64KB System with 8KB ROM + 8KB RAM
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

                      74LS138
                    ┌───────────┐
         A13 ───────┤ A      Y0 ├─── ROM 0 (0000-1FFF)
         A14 ───────┤ B      Y1 ├─── ROM 1 (2000-3FFF)
         A15 ───────┤ C      Y2 ├─── ROM 2 (4000-5FFF)
                    │        Y3 ├─── ROM 3 (6000-7FFF)
         +5V ───────┤ G1     Y4 ├─── RAM 0 (8000-9FFF)
         GND ───────┤ G2A    Y5 ├─── RAM 1 (A000-BFFF)
         GND ───────┤ G2B    Y6 ├─── RAM 2 (C000-DFFF)
                    │        Y7 ├─── RAM 3 (E000-FFFF)
                    └───────────┘

Memory Map:
┌────────────────────────────────────┐
│ FFFFH                              │
│   ↑     RAM 3 (8KB) - Stack Area   │
│ E000H                              │
├────────────────────────────────────┤
│ DFFFH                              │
│   ↑     RAM 2 (8KB)                │
│ C000H                              │
├────────────────────────────────────┤
│ BFFFH                              │
│   ↑     RAM 1 (8KB)                │
│ A000H                              │
├────────────────────────────────────┤
│ 9FFFH                              │
│   ↑     RAM 0 (8KB)                │
│ 8000H                              │
├────────────────────────────────────┤
│ 7FFFH                              │
│   ↑     ROM 3 (8KB)                │
│ 6000H                              │
├────────────────────────────────────┤
│ 5FFFH                              │
│   ↑     ROM 2 (8KB)                │
│ 4000H                              │
├────────────────────────────────────┤
│ 3FFFH                              │
│   ↑     ROM 1 (8KB)                │
│ 2000H                              │
├────────────────────────────────────┤
│ 1FFFH                              │
│   ↑     ROM 0 (8KB) - Reset Vector │
│ 0000H                              │
└────────────────────────────────────┘
```

---

## 5. Memory Interfacing with 8086

### 5.1 8086 Memory Organization

```
8086 has two memory banks for 16-bit data bus:

        EVEN BANK              ODD BANK
        (D0-D7)                (D8-D15)
        
FFFFEH ┌─────────┐  FFFFFH ┌─────────┐
       │         │         │         │
       │  Even   │         │   Odd   │
       │ Address │         │ Address │
       │  Bytes  │         │  Bytes  │
       │         │         │         │
00002H ├─────────┤  00003H ├─────────┤
00000H │         │  00001H │         │
       └─────────┘         └─────────┘
            │                   │
            ▼                   ▼
          BHĒ=1               BHĒ=0
          A0=0                A0=1

Bank Selection:
┌──────┬──────┬─────────────────────────────┐
│ A0   │ BHĒ  │ Data Transfer               │
├──────┼──────┼─────────────────────────────┤
│  0   │  0   │ Word at even address        │
│  0   │  1   │ Byte from even bank (D0-D7) │
│  1   │  0   │ Byte from odd bank (D8-D15) │
│  1   │  1   │ No transfer                 │
└──────┴──────┴─────────────────────────────┘
```

### 5.2 Interfacing 64KB RAM with 8086

```
8086 to 2 × 62256 (32K × 8) RAM Banks
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

       8086                Even Bank           Odd Bank
      ┌─────┐            (62256-1)           (62256-2)
      │     │            ┌────────┐         ┌────────┐
      │A1-15├────────────┤A0-A14  │         │A0-A14  │
      │     │            │        │         │        │
      │D0-D7├────────────┤D0-D7   │         │        │
      │     │            │        │         │        │
      │D8-15│            │        │─────────┤D0-D7   │
      │     │            │        │         │        │
      │  A0 ├────────────┤CĒ      │         │        │
      │     │            │        │         │        │
      │ BHĒ ├────────────│────────│─────────┤CĒ      │
      │     │            │        │         │        │
      │ RD̄  ├────────────┤OĒ      │─────────┤OĒ      │
      │     │            │        │         │        │
      │ WR̄  ├────────────┤WĒ      │─────────┤WĒ      │
      │     │            │        │         │        │
      └─────┘            └────────┘         └────────┘

Note: A0 is used to select even bank (when A0=0)
      BHĒ is used to select odd bank (when BHĒ=0)
      A1-A15 are connected to A0-A14 of memory chips
```

---

## 6. Timing Considerations

### 6.1 Memory Read Timing (8085)

```
            T1        T2        T3
         ┌─────────┬─────────┬─────────┐
CLK      │    ┌┐   │   ┌┐    │   ┌┐    │
    ─────┘    └┘   ┘   └┘    ┘   └┘    └─────
         
ALE      ─────┐         ┌────────────────────
              └─────────┘
         
Address  ─────┬─────────────────────────┬────
(A8-A15)      │     VALID ADDRESS       │
         ─────┴─────────────────────────┴────
         
AD0-AD7  ─────┬─────────┬───────────────┬────
              │ Address │     DATA      │
         ─────┴─────────┴───────────────┴────
         
RD̄        ────────────────┐             ┌────
                          └─────────────┘
         
         ◄──────────────────────────────►
                    Memory Access Time
                    (must be < 3 T-states)
```

### 6.2 Access Time Calculation

```
For 8085 at 3 MHz:
- T-state = 1/3MHz = 333 ns
- Memory cycle = 3 T-states = 1000 ns
- Available access time ≈ 575 ns

Memory chip selection:
- If access time < 575 ns → No wait states needed
- If access time > 575 ns → Add wait states

Wait State Insertion:
┌────────────────────────────────────────┐
│ Each WAIT state adds one T-state       │
│ (333 ns at 3 MHz)                      │
│                                        │
│ Slow memory (200ns) + decode (50ns)    │
│ = 250 ns < 575 ns → OK, no wait needed │
└────────────────────────────────────────┘
```

---

## 📋 Summary Table

| Topic | 8085 | 8086 |
|-------|------|------|
| Address Bus | 16-bit (64KB) | 20-bit (1MB) |
| Data Bus | 8-bit | 16-bit |
| Memory Banks | Single | Even + Odd |
| Bank Select | N/A | A0, BHĒ |
| Decoder Chip | 74LS138 | 74LS138 |
| Typical ROM | 2716, 2732 | 27256, 27512 |
| Typical RAM | 6116, 6264 | 62256, 628128 |

---

## ❓ Quick Revision Questions

1. **What is the difference between complete and partial address decoding?**
   <details>
   <summary>Show Answer</summary>
   Complete decoding uses ALL address lines, giving each location a unique address with no overlap. Partial decoding uses only some address lines, resulting in multiple addresses mapping to the same location (foldback). Complete decoding requires more hardware but uses address space efficiently.
   </details>

2. **Why does 8086 need two memory banks?**
   <details>
   <summary>Show Answer</summary>
   8086 has a 16-bit data bus but memory is byte-addressable. Two banks (even and odd) allow accessing either a byte from either bank or a word from both banks simultaneously. A0 selects even bank, BHĒ selects odd bank.
   </details>

3. **How many address lines are needed for 8KB memory?**
   <details>
   <summary>Show Answer</summary>
   8KB = 8192 bytes = 2^13 bytes. So 13 address lines (A0-A12) are needed to address 8KB of memory.
   </details>

4. **What is the function of the 74LS138 decoder in memory interfacing?**
   <details>
   <summary>Show Answer</summary>
   The 74LS138 is a 3-to-8 decoder that converts 3 address input lines into 8 chip select outputs. Only one output is active (LOW) at a time based on the input combination. It's used to generate chip select signals for up to 8 memory devices.
   </details>

5. **What happens if memory access time exceeds the processor's timing?**
   <details>
   <summary>Show Answer</summary>
   If memory is too slow, wait states must be inserted into the bus cycle. The processor's READY input is held LOW to extend the cycle. Each wait state adds one clock cycle to the memory access, giving the slow memory time to respond.
   </details>

6. **Calculate the address range for a 4KB ROM starting at C000H.**
   <details>
   <summary>Show Answer</summary>
   4KB = 4096 = 1000H bytes. Starting at C000H, the ending address = C000H + 1000H - 1 = CFFFH. So the address range is C000H to CFFFH.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Unit 6 Index](README.md) | [Unit 6 Index](README.md) | [6.2 I/O Interfacing](02-io-interfacing.md) |

---

*[← Unit 6 Index](README.md) | [Next: I/O Interfacing →](02-io-interfacing.md)*
