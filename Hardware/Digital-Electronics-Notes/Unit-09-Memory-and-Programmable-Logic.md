# Unit 9: Memory and Programmable Logic

---

## Table of Contents
1. [Introduction to Memory](#1-introduction-to-memory)
2. [Memory Organization](#2-memory-organization)
3. [Read-Only Memory (ROM)](#3-read-only-memory-rom)
4. [Static RAM (SRAM)](#4-static-ram-sram)
5. [Dynamic RAM (DRAM)](#5-dynamic-ram-dram)
6. [Memory Expansion](#6-memory-expansion)
7. [Programmable Logic Array (PLA)](#7-programmable-logic-array-pla)
8. [Programmable Array Logic (PAL)](#8-programmable-array-logic-pal)
9. [Complex PLDs and FPGAs](#9-complex-plds-and-fpgas)
10. [Summary Tables](#10-summary-tables)
11. [Quick Revision Questions](#11-quick-revision-questions)

---

## 1. Introduction to Memory

### What is Memory?

```
┌─────────────────────────────────────────────────────────────────┐
│                    MEMORY BASICS                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  DEFINITION:                                                    │
│    Electronic circuit that stores binary information           │
│                                                                 │
│  BASIC OPERATIONS:                                              │
│    • READ: Retrieve stored data                                │
│    • WRITE: Store new data (for RAM)                           │
│                                                                 │
│  MEMORY CELL:                                                   │
│    Smallest unit that stores 1 bit                             │
│                                                                 │
│  MEMORY WORD:                                                   │
│    Group of bits stored/accessed together                      │
│    Common sizes: 8, 16, 32, 64 bits                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Memory Classification

```
┌─────────────────────────────────────────────────────────────────┐
│                MEMORY CLASSIFICATION                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                        MEMORY                                   │
│                          │                                      │
│          ┌───────────────┴───────────────┐                     │
│          │                               │                      │
│     VOLATILE                        NON-VOLATILE               │
│   (loses data                       (retains data              │
│    when power off)                   when power off)           │
│          │                               │                      │
│     ┌────┴────┐                  ┌───────┴───────┐             │
│     │         │                  │               │              │
│   SRAM      DRAM               ROM            FLASH            │
│ (Static)  (Dynamic)        (Read Only)    (Electrically       │
│                                            Erasable)           │
│                                  │                              │
│                    ┌─────────────┼─────────────┐               │
│                    │             │             │                │
│                  PROM         EPROM        EEPROM              │
│               (One-time      (UV           (Electrically       │
│               Program)      Erasable)      Erasable)           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

VOLATILE: SRAM, DRAM (need constant power)
NON-VOLATILE: ROM, PROM, EPROM, EEPROM, Flash
```

---

## 2. Memory Organization

### Memory Structure

```
MEMORY ORGANIZATION (2ⁿ × m):

    n = address bits
    m = word size (data bits)
    Total capacity = 2ⁿ × m bits

EXAMPLE: 1K × 8 Memory (1024 words × 8 bits)

    n = 10 address bits (2¹⁰ = 1024)
    m = 8 data bits
    
    Total = 1024 × 8 = 8192 bits = 1 KB

MEMORY BLOCK DIAGRAM:

                   ┌─────────────────────────────────────┐
                   │                                     │
    Address ──────►│  ┌─────────────────────────────┐   │
    (n bits)       │  │                             │   │
     A₀ ──────────►│  │        MEMORY ARRAY         │   │
     A₁ ──────────►│  │                             │   │
      .            │  │     2ⁿ rows × m columns     │   │
      .            │  │                             │   │
    Aₙ₋₁─────────►│  │                             │   │
                   │  └─────────────────────────────┘   │
                   │              │ │ │ │ │ │ │ │       │
                   │              ▼ ▼ ▼ ▼ ▼ ▼ ▼ ▼       │
                   │         Data (m bits)              │
      CS ─────────►│              D₀...Dₘ₋₁             │◄───► Data Bus
      R/W'────────►│                                    │
      OE ─────────►│                                    │
                   │                                    │
                   └─────────────────────────────────────┘

    CS:  Chip Select (enable this chip)
    R/W': Read/Write' (1=Read, 0=Write)
    OE:  Output Enable (enable data output)
```

### Memory Decoding

```
ADDRESS DECODING INSIDE MEMORY:

For 2ⁿ × m memory:
  - n address lines → 2ⁿ word lines
  - Decoder selects one row at a time

INTERNAL STRUCTURE (4 × 4 example):

         A₁  A₀
          │   │
          ▼   ▼
       ┌──────────┐
       │  2-to-4  │
       │ DECODER  │
       └────┬─────┘
            │
    ┌───────┼───────┐
    │  ┌────┴────┐  │
    │  │ W₀ W₁ W₂ W₃│
    │  └─┬──┬──┬──┬─┘
    │    │  │  │  │
    │ ┌──▼──▼──▼──▼──┐
    │ │              │     ┌───┐
    │ │  ■  ■  ■  ■  │────►│   │───► D₀
    │ │              │     │   │
    │ │  ■  ■  ■  ■  │────►│MUX│───► D₁
    │ │              │     │   │
    │ │  ■  ■  ■  ■  │────►│   │───► D₂
    │ │              │     │   │
    │ │  ■  ■  ■  ■  │────►│   │───► D₃
    │ │              │     └───┘
    │ └──────────────┘
    │    Memory Array
    │    (4 × 4 bits)
    └────────────────────────────────

Word line W₀ selects row 0
Word line W₁ selects row 1, etc.
```

---

## 3. Read-Only Memory (ROM)

### ROM Types

```
┌─────────────────────────────────────────────────────────────────┐
│                      ROM TYPES                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. MASK ROM:                                                   │
│     - Programmed during manufacturing                          │
│     - Cannot be changed                                        │
│     - Lowest cost for high volume                              │
│                                                                 │
│  2. PROM (Programmable ROM):                                    │
│     - Programmed once by user                                  │
│     - Uses fusible links (blown = 0)                           │
│     - Cannot be erased                                         │
│                                                                 │
│  3. EPROM (Erasable PROM):                                      │
│     - Erased by UV light through window                        │
│     - Reprogrammable multiple times                            │
│     - ~20 minutes to erase completely                          │
│                                                                 │
│  4. EEPROM (Electrically Erasable PROM):                        │
│     - Erased electrically                                      │
│     - Byte-by-byte erasure possible                            │
│     - Limited write cycles (~10⁶)                              │
│                                                                 │
│  5. FLASH MEMORY:                                               │
│     - Type of EEPROM                                           │
│     - Sector/block erasure                                     │
│     - Fast, high density                                       │
│     - Used in SSDs, USB drives                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### ROM Internal Structure

```
ROM AS DECODER + OR ARRAY:

Example: 4 × 3 ROM (4 addresses × 3-bit words)

         A₁  A₀
          │   │
          ▼   ▼
       ┌──────────┐
       │  2-to-4  │
       │ DECODER  │
       └────┬─────┘
            │
    W₀ ─────┼────────●─────────●─────────○
    W₁ ─────┼────────●─────────○─────────●
    W₂ ─────┼────────○─────────●─────────●
    W₃ ─────┘────────●─────────●─────────●
                     │         │         │
                     ▼         ▼         ▼
                  ┌─────┐   ┌─────┐   ┌─────┐
                  │ OR  │   │ OR  │   │ OR  │
                  └──┬──┘   └──┬──┘   └──┬──┘
                     │         │         │
                     D₂        D₁        D₀

    ● = connection (1 programmed)
    ○ = no connection (0 programmed)

    Address 00 (W₀=1): D₂D₁D₀ = 110
    Address 01 (W₁=1): D₂D₁D₀ = 101
    Address 10 (W₂=1): D₂D₁D₀ = 011
    Address 11 (W₃=1): D₂D₁D₀ = 111

ROM TRUTH TABLE:
┌─────┬─────┬─────┬─────┬─────┐
│ A₁  │ A₀  │ D₂  │ D₁  │ D₀  │
├─────┼─────┼─────┼─────┼─────┤
│  0  │  0  │  1  │  1  │  0  │
│  0  │  1  │  1  │  0  │  1  │
│  1  │  0  │  0  │  1  │  1  │
│  1  │  1  │  1  │  1  │  1  │
└─────┴─────┴─────┴─────┴─────┘
```

### ROM Applications

```
ROM APPLICATIONS:

1. FUNCTION GENERATOR (Look-up Table):
   - Store function values
   - Address = input, Data = output
   
   Example: Square root look-up
   Address = N, Data = √N (pre-calculated)

2. CODE CONVERTER:
   - BCD to 7-segment
   - Binary to Gray code
   
3. CHARACTER GENERATOR:
   - Store font patterns
   - Address = ASCII code
   - Data = pixel pattern

4. MICROPROGRAM STORAGE:
   - CPU control unit
   - Microinstruction storage

5. BOOT/BIOS STORAGE:
   - Computer startup code
   - Fixed firmware

EXAMPLE: BCD TO 7-SEGMENT DECODER

                ┌────────────────────┐
    BCD ────────┤                    │
   (4 bits)     │       ROM          │──────── 7-segment
    A₃A₂A₁A₀ ──►│    (16 × 7)        │          (a-g)
                │                    │
                └────────────────────┘

ROM Contents:
┌─────────────┬──────────────┐
│  Address    │  Data (a-g)  │
│  (BCD)      │  7-segment   │
├─────────────┼──────────────┤
│  0000 (0)   │  1111110     │  ┌───┐
│  0001 (1)   │  0110000     │  │ a │
│  0010 (2)   │  1101101     │  ├───┤
│  0011 (3)   │  1111001     │  │fgb│
│     ...     │     ...      │  ├───┤
│  1001 (9)   │  1111011     │  │ d │
└─────────────┴──────────────┘  └───┘
```

---

## 4. Static RAM (SRAM)

### SRAM Cell

```
┌─────────────────────────────────────────────────────────────────┐
│                     SRAM CELL                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Uses cross-coupled inverters to store data                    │
│  Retains data as long as power is applied                      │
│                                                                 │
│  6-TRANSISTOR (6T) SRAM CELL:                                   │
│                                                                 │
│          Word Line (WL)                                         │
│               │                                                 │
│          ┌────┴────┐                                            │
│          │    │    │                                            │
│       ┌──┴─┐  │  ┌─┴──┐                                        │
│       │ M5 │  │  │ M6 │   (Access transistors)                 │
│       └──┬─┘  │  └─┬──┘                                        │
│          │    │    │                                            │
│   BL ────┤    │    ├──── BL'                                    │
│          │    │    │                                            │
│          ├────●────┤                                            │
│          │    │    │                                            │
│       ┌──┴────┴────┴──┐                                        │
│       │               │                                         │
│       │  ┌───┐ ┌───┐  │                                        │
│       │  │M1 │ │M2 │  │   (Cross-coupled inverters)            │
│       │  └─┬─┘ └─┬─┘  │                                        │
│       │    │     │    │                                         │
│       │    ●─────●    │                                         │
│       │    │     │    │                                         │
│       │  ┌─┴─┐ ┌─┴─┐  │                                        │
│       │  │M3 │ │M4 │  │                                        │
│       │  └───┘ └───┘  │                                        │
│       │               │                                         │
│       └───────────────┘                                        │
│             │                                                   │
│            GND                                                  │
│                                                                 │
│  BL, BL': Bit Lines (complementary)                            │
│  WL: Word Line (selects row)                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### SRAM Operation

```
SRAM READ OPERATION:

1. Precharge BL and BL' to VDD/2 or VDD
2. Assert Word Line (WL = HIGH)
3. Access transistors M5, M6 turn ON
4. Cell drives BL and BL' to different levels
5. Sense amplifier detects voltage difference
6. Data is read

READ TIMING:
         ┌───────────────────────────────
    WL   │                               
    ─────┘
         
    BL   ════╲                           
              ╲═════════════════════════ (drops to 0 if stored 0)
              
    BL'  ════════════════════════════════ (stays high)
         
    Data ─────────────────┬──────────────
    Out                   │
                       Data Valid


SRAM WRITE OPERATION:

1. Drive BL to data value, BL' to complement
2. Assert Word Line (WL = HIGH)
3. Bit line drivers overpower cell
4. New data stored in cross-coupled inverters
5. Deassert WL

WRITE TIMING:
         ┌───────────────────────────────
    WL   │                               
    ─────┘
         
    BL   ────────────────────────────────
         (driven to write data)
         
    BL'  ════════════════════════════════
         (driven to complement)
```

### SRAM Characteristics

```
SRAM CHARACTERISTICS:

┌─────────────────────┬──────────────────────┐
│     Feature         │        Value         │
├─────────────────────┼──────────────────────┤
│ Cell size           │ 6 transistors        │
│ Speed               │ Fast (ns range)      │
│ Refresh needed      │ No                   │
│ Power (active)      │ Moderate             │
│ Power (standby)     │ Low                  │
│ Density             │ Lower than DRAM      │
│ Cost/bit            │ Higher than DRAM     │
│ Volatility          │ Volatile             │
└─────────────────────┴──────────────────────┘

ADVANTAGES:
  + Fast access time
  + No refresh required
  + Simple interface
  
DISADVANTAGES:
  - Lower density (6T per bit)
  - Higher cost per bit
  - More power for same capacity

TYPICAL USES:
  - CPU cache (L1, L2, L3)
  - Register files
  - Small, fast memories
```

---

## 5. Dynamic RAM (DRAM)

### DRAM Cell

```
┌─────────────────────────────────────────────────────────────────┐
│                      DRAM CELL                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Stores data as charge on a capacitor                          │
│  Needs periodic refresh (charge leaks)                         │
│                                                                 │
│  1-TRANSISTOR (1T1C) DRAM CELL:                                │
│                                                                 │
│        Word Line (WL)                                           │
│             │                                                   │
│        ┌────┴────┐                                              │
│        │         │                                              │
│        │   M     │ ← Access transistor                         │
│        │         │                                              │
│        └────┬────┘                                              │
│             │                                                   │
│   Bit Line ─┤                                                   │
│     (BL)    │                                                   │
│             │                                                   │
│          ───┴───                                                │
│          ───┬─── C ← Storage capacitor                         │
│             │                                                   │
│            ─┴─ (plate/ground)                                   │
│                                                                 │
│                                                                 │
│  Much smaller than SRAM! (1T vs 6T)                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### DRAM Operation

```
DRAM WRITE OPERATION:

1. Drive Bit Line to data value (VDD or 0)
2. Assert Word Line (WL = HIGH)
3. Transistor turns ON
4. Capacitor charges/discharges to BL voltage
5. Deassert WL (transistor OFF, charge trapped)

DRAM READ OPERATION (Destructive Read):

1. Precharge Bit Line to VDD/2
2. Assert Word Line
3. Charge sharing between BL capacitance and cell C
4. Sense amplifier detects small voltage change
5. Read amplifies and restores data
6. Data written back (refresh)

READ TIMING:
         ┌───────────────────────────────
    WL   │                               
    ─────┘
         
    BL   ────╱╲──────────────────────────
    (VDD/2)  small change (ΔV ~ 100-200mV)
             │
             └── Sense amplifier triggers
         
    Data ─────────────────────┬──────────
    Out                       │
                           Valid


CHARGE SHARING:

ΔV = (C_cell / (C_cell + C_BL)) × VDD

If C_cell << C_BL:
   ΔV is small → need sensitive sense amplifier
```

### DRAM Refresh

```
DRAM REFRESH REQUIREMENT:

Problem: Capacitor charge leaks over time
         Data lost in milliseconds without refresh

Solution: Periodically read and rewrite all cells

REFRESH CYCLE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│    ┌────────┐         ┌────────┐         ┌────────┐            │
│    │        │         │        │         │        │            │
│    │ Charge │  leak   │ Refresh│  leak   │ Refresh│            │
│    │ = VDD  │────────►│ Read/  │────────►│ Read/  │ ...        │
│    │        │         │ Write  │         │ Write  │            │
│    └────────┘         └────────┘         └────────┘            │
│         │◄────────────────►│◄────────────────►│                │
│               ~64 ms            ~64 ms                         │
│           Refresh interval                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

REFRESH METHODS:

1. RAS-Only Refresh (ROR):
   - Activate row, sense amplifiers restore data
   
2. CAS-Before-RAS (CBR) Refresh:
   - Internal address counter used
   - Simpler for controller

3. Self-Refresh:
   - DRAM internally generates refresh cycles
   - For low-power/sleep modes

REFRESH OVERHEAD:
  Typical: 8192 rows, 64ms interval
  Each row: ~50ns refresh time
  Total: 8192 × 50ns = 410μs per 64ms
  Overhead: ~0.64%
```

### SRAM vs DRAM Comparison

```
┌──────────────────────┬───────────────────┬───────────────────┐
│      Feature         │       SRAM        │       DRAM        │
├──────────────────────┼───────────────────┼───────────────────┤
│ Cell structure       │ 6 transistors     │ 1T + 1 capacitor  │
│ Storage mechanism    │ Cross-coupled inv │ Capacitor charge  │
│ Speed                │ Faster (~ns)      │ Slower (~10ns)    │
│ Refresh needed       │ No                │ Yes (~64ms)       │
│ Density              │ Lower             │ Higher (4×)       │
│ Cost per bit         │ Higher            │ Lower             │
│ Power (active)       │ Lower             │ Higher            │
│ Power (standby)      │ Very low          │ Needs refresh     │
│ Complexity           │ Simple interface  │ Needs controller  │
│ Read operation       │ Non-destructive   │ Destructive       │
├──────────────────────┼───────────────────┼───────────────────┤
│ Typical use          │ Cache memory      │ Main memory       │
│ Capacity range       │ KB to MB          │ GB                │
└──────────────────────┴───────────────────┴───────────────────┘
```

---

## 6. Memory Expansion

### Increasing Word Size (Wider Data)

```
EXPANDING WORD SIZE (4K × 4 → 4K × 8):

Use multiple chips with same address, different data bits

                 ┌───────────────────┐
    Address ────►│                   │
    (12 bits)    │   4K × 4 ROM     │───► D₃ D₂ D₁ D₀
                 │     Chip 1        │
        CS ─────►│                   │
                 └───────────────────┘
                 
                 ┌───────────────────┐
    Address ────►│                   │
    (12 bits)    │   4K × 4 ROM     │───► D₇ D₆ D₅ D₄
                 │     Chip 2        │
        CS ─────►│                   │
                 └───────────────────┘

Both chips receive:
  - Same 12-bit address
  - Same CS signal

Result: 4K × 8 memory (8-bit words)
```

### Increasing Address Space (Larger Capacity)

```
EXPANDING ADDRESS SPACE (4K × 8 → 8K × 8):

Use decoder to select different chips

           A₁₂ (new high address bit)
             │
             ▼
         ┌───────┐
         │ 1-to-2│
         │DECODER│
         └───┬───┘
             │
      ┌──────┴──────┐
      │             │
      ▼ CS₀         ▼ CS₁
  ┌──────────┐  ┌──────────┐
  │ 4K × 8   │  │ 4K × 8   │
  │ Chip 0   │  │ Chip 1   │
  │          │  │          │
  └────┬─────┘  └────┬─────┘
       │             │
       └──────┬──────┘
              │
              ▼
         Data Bus (8 bits)

Address  A₁₂:
  A₁₂ = 0 → Chip 0 selected (address 0000-0FFF)
  A₁₂ = 1 → Chip 1 selected (address 1000-1FFF)

A₁₁-A₀ → Both chips (internal address)

Result: 8K × 8 memory
```

### Complete Memory System Example

```
EXAMPLE: 32K × 8 MEMORY FROM 8K × 8 CHIPS

Need: 4 chips (4 × 8K = 32K)
Address: 15 bits (A₁₄-A₀)
Decoder: 2-to-4 (uses A₁₄, A₁₃)

                  A₁₄ A₁₃
                    │  │
                    ▼  ▼
                ┌────────┐
                │ 2-to-4 │
                │DECODER │
                └────┬───┘
           ┌────┬───┬┴───┬────┐
           │    │   │    │    │
           ▼    ▼   ▼    ▼
          CS₀  CS₁ CS₂  CS₃
          
    ┌──────────────────────────────────────┐
    │  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐ │
    │  │8K×8 │  │8K×8 │  │8K×8 │  │8K×8 │ │
    │  │Chip0│  │Chip1│  │Chip2│  │Chip3│ │
    │  └──┬──┘  └──┬──┘  └──┬──┘  └──┬──┘ │
    │     │        │        │        │    │
    │     └────────┴────────┴────────┘    │
    │                  │                   │
    └──────────────────┴───────────────────┘
                       │
                  Data Bus (D₀-D₇)
                       │
    A₁₂-A₀ ───────────►│ (13 bits to all chips)

ADDRESS MAP:
┌────────────────────┬───────────────────┐
│   Address Range    │      Chip         │
├────────────────────┼───────────────────┤
│ 0000h - 1FFFh      │    Chip 0         │
│ 2000h - 3FFFh      │    Chip 1         │
│ 4000h - 5FFFh      │    Chip 2         │
│ 6000h - 7FFFh      │    Chip 3         │
└────────────────────┴───────────────────┘
```

---

## 7. Programmable Logic Array (PLA)

### PLA Structure

```
┌─────────────────────────────────────────────────────────────────┐
│              PROGRAMMABLE LOGIC ARRAY (PLA)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Structure: Programmable AND array + Programmable OR array     │
│                                                                 │
│  Implements Sum-of-Products (SOP) functions                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

PLA ARCHITECTURE:

    Inputs: A, B, C (and complements)
    
           A   A'  B   B'  C   C'
           │   │   │   │   │   │
           ▼   ▼   ▼   ▼   ▼   ▼
    ┌──────┴───┴───┴───┴───┴───┴──────┐
    │                                  │
    │    PROGRAMMABLE AND ARRAY        │
    │         (Product Terms)          │
    │                                  │
    │  ─×───×───○───○───×───○─ P₀     │  A·A'·C
    │  ─○───×───×───○───○───×─ P₁     │  A'·B·C'
    │  ─×───○───×───○───×───○─ P₂     │  A·B·C
    │  ─○───○───○───×───×───○─ P₃     │  B'·C
    │                                  │
    │  × = programmed connection       │
    │  ○ = no connection               │
    │                                  │
    └─────────────┬────────────────────┘
                  │ P₀  P₁  P₂  P₃
                  │  │   │   │   │
                  ▼  ▼   ▼   ▼   ▼
    ┌─────────────────────────────────┐
    │                                  │
    │    PROGRAMMABLE OR ARRAY         │
    │         (Sum Terms)              │
    │                                  │
    │  ─×───○───×───○─ F₀ = P₀ + P₂   │
    │  ─○───×───○───×─ F₁ = P₁ + P₃   │
    │  ─×───×───×───○─ F₂ = P₀ + P₁ + P₂│
    │                                  │
    └─────────────────────────────────┘
                  │
                  ▼
            Outputs: F₀, F₁, F₂
```

### PLA Design Example

```
DESIGN EXAMPLE: Implement these functions

F₁(A,B,C) = Σm(0, 1, 4, 5)
F₂(A,B,C) = Σm(2, 3, 5, 7)

STEP 1: Minimize functions
F₁ = A'B' + AB' = B'(A' + A) = B'  (Actually: A'B' + AB' = B')
  Wait, let me recalculate:
  m0 = A'B'C', m1 = A'B'C, m4 = AB'C', m5 = AB'C
  F₁ = B' (all minterms have B'=1)

F₂ = m2 + m3 + m5 + m7
   = A'BC' + A'BC + AB'C + ABC
   = A'B(C' + C) + AC(B' + B)
   = A'B + AC

STEP 2: List product terms needed
P₁ = B'   (for F₁)
P₂ = A'B  (for F₂)
P₃ = AC   (for F₂)

STEP 3: PLA Implementation

           A   A'  B   B'  C   C'
           │   │   │   │   │   │
           ▼   ▼   ▼   ▼   ▼   ▼
    ┌──────────────────────────────────┐
    │                                  │
    │    PROGRAMMABLE AND ARRAY        │
    │                                  │
    │  ─○───○───○───×───○───○─ P₁=B'  │
    │  ─○───×───×───○───○───○─ P₂=A'B │
    │  ─×───○───○───○───×───○─ P₃=AC  │
    │                                  │
    └──────────────┬───────────────────┘
                   │ P₁  P₂  P₃
                   ▼  ▼   ▼   ▼
    ┌──────────────────────────────────┐
    │                                  │
    │    PROGRAMMABLE OR ARRAY         │
    │                                  │
    │  ─×───○───○─ F₁ = P₁ = B'       │
    │  ─○───×───×─ F₂ = P₂ + P₃ = A'B + AC│
    │                                  │
    └──────────────────────────────────┘
```

---

## 8. Programmable Array Logic (PAL)

### PAL Structure

```
┌─────────────────────────────────────────────────────────────────┐
│             PROGRAMMABLE ARRAY LOGIC (PAL)                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Structure: Programmable AND array + FIXED OR array            │
│                                                                 │
│  Difference from PLA:                                          │
│    - OR array is fixed (not programmable)                      │
│    - Simpler, faster, cheaper                                  │
│    - Less flexible (fixed product terms per output)            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

PAL ARCHITECTURE:

           A   A'  B   B'  C   C'
           │   │   │   │   │   │
           ▼   ▼   ▼   ▼   ▼   ▼
    ┌──────────────────────────────────┐
    │                                  │
    │    PROGRAMMABLE AND ARRAY        │
    │                                  │
    │  ─×───×───○───○───×───○─ P₀ ─┐  │
    │  ─○───×───×───○───○───×─ P₁ ─┼──│──► F₀ = P₀ + P₁
    │                               │  │
    │  ─×───○───×───○───×───○─ P₂ ─┼──│──► F₁ = P₂ + P₃
    │  ─○───○───○───×───×───○─ P₃ ─┘  │
    │                                  │
    │  × = programmable connection     │
    │                                  │
    └──────────────────────────────────┘
                  │
                  │    FIXED OR GATES
                  │    (not programmable)
                  │
            F₀ = P₀ + P₁
            F₁ = P₂ + P₃

Each output has a FIXED number of product terms (e.g., 2, 4, or 8)
```

### PAL vs PLA Comparison

```
┌─────────────────────────────┬───────────────────┬───────────────┐
│        Feature              │       PLA         │      PAL      │
├─────────────────────────────┼───────────────────┼───────────────┤
│ AND array                   │ Programmable      │ Programmable  │
│ OR array                    │ Programmable      │ Fixed         │
│ Flexibility                 │ High              │ Lower         │
│ Speed                       │ Slower            │ Faster        │
│ Complexity                  │ More complex      │ Simpler       │
│ Cost                        │ Higher            │ Lower         │
│ Product term sharing        │ Yes               │ No            │
│ Products per output         │ Variable          │ Fixed         │
├─────────────────────────────┼───────────────────┼───────────────┤
│ Best for                    │ Complex logic     │ Simple logic  │
│                             │ Many shared terms │ Standard apps │
└─────────────────────────────┴───────────────────┴───────────────┘

PLA ADVANTAGE:
  Any product term can be connected to any output
  Fewer product terms needed if sharing possible

PAL ADVANTAGE:
  Faster (fewer programmable connections)
  Simpler to program
  Lower cost
```

### Registered PAL

```
REGISTERED PAL (with flip-flops):

Adds D flip-flops to PAL outputs
Allows sequential logic implementation

           Inputs
              │
              ▼
    ┌─────────────────────┐
    │                     │
    │   Programmable      │
    │    AND Array        │
    │                     │
    └──────────┬──────────┘
               │
               ▼
         Fixed OR Array
               │
               ▼
    ┌──────────────────────────────┐
    │                              │
    │   ┌───┐   ┌───┐   ┌───┐     │
    │   │ D │   │ D │   │ D │     │
    │   │   │   │   │   │   │     │
    │   │ Q │   │ Q │   │ Q │     │  Registered Outputs
    │   └─┬─┘   └─┬─┘   └─┬─┘     │
    │     │       │       │        │
    │     ▼       ▼       ▼        │
    │   Output  Output  Output     │
    │                              │
    │         ↑                    │
    │        CLK                   │
    │                              │
    │   Feedback to AND array      │
    │   (allows state machines)    │
    │                              │
    └──────────────────────────────┘

APPLICATIONS:
  - Counters
  - State machines
  - Registered I/O
```

---

## 9. Complex PLDs and FPGAs

### CPLD (Complex PLD)

```
┌─────────────────────────────────────────────────────────────────┐
│                        CPLD                                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Multiple PAL-like blocks connected by programmable interconnect│
│                                                                 │
│  CPLD ARCHITECTURE:                                             │
│                                                                 │
│    ┌─────────────────────────────────────────────────────┐     │
│    │                                                     │     │
│    │  ┌───────┐  ┌───────┐  ┌───────┐  ┌───────┐       │     │
│    │  │ Logic │  │ Logic │  │ Logic │  │ Logic │       │     │
│    │  │ Block │  │ Block │  │ Block │  │ Block │       │     │
│    │  │  (PAL)│  │  (PAL)│  │  (PAL)│  │  (PAL)│       │     │
│    │  └───┬───┘  └───┬───┘  └───┬───┘  └───┬───┘       │     │
│    │      │          │          │          │           │     │
│    │      └──────────┴──────────┴──────────┘           │     │
│    │                     │                              │     │
│    │         ┌───────────┴───────────┐                 │     │
│    │         │ PROGRAMMABLE          │                 │     │
│    │         │ INTERCONNECT MATRIX   │                 │     │
│    │         └───────────┬───────────┘                 │     │
│    │                     │                              │     │
│    │    ┌────────────────┼────────────────┐            │     │
│    │    │                │                │            │     │
│    │ ┌──┴──┐          ┌──┴──┐          ┌──┴──┐        │     │
│    │ │ I/O │          │ I/O │          │ I/O │        │     │
│    │ │Block│          │Block│          │Block│        │     │
│    │ └─────┘          └─────┘          └─────┘        │     │
│    │                                                     │     │
│    └─────────────────────────────────────────────────────┘     │
│                                                                 │
│  FEATURES:                                                      │
│    • Non-volatile (flash based)                                │
│    • Predictable timing                                        │
│    • Instant-on                                                │
│    • Complexity: 100s to 1000s of gates                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### FPGA (Field Programmable Gate Array)

```
┌─────────────────────────────────────────────────────────────────┐
│                         FPGA                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Array of configurable logic blocks with programmable routing  │
│                                                                 │
│  FPGA ARCHITECTURE:                                             │
│                                                                 │
│    ┌────────────────────────────────────────────────────────┐  │
│    │ I/O  I/O  I/O  I/O  I/O  I/O  I/O  I/O  I/O  I/O  I/O │  │
│    ├────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┤  │
│    │I/O │ CLB│    │ CLB│    │ CLB│    │ CLB│    │ CLB│I/O │  │
│    ├────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤  │
│    │    │    │====│    │====│    │====│    │====│    │    │  │
│    ├────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤  │
│    │I/O │ CLB│    │ CLB│    │ CLB│    │ CLB│    │ CLB│I/O │  │
│    ├────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤  │
│    │    │    │====│    │====│    │====│    │====│    │    │  │
│    ├────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤  │
│    │I/O │ CLB│    │ CLB│    │ CLB│    │ CLB│    │ CLB│I/O │  │
│    ├────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┤  │
│    │ I/O  I/O  I/O  I/O  I/O  I/O  I/O  I/O  I/O  I/O  I/O │  │
│    └────────────────────────────────────────────────────────┘  │
│                                                                 │
│    CLB = Configurable Logic Block                              │
│    ==== = Programmable Routing (Switch Matrix)                 │
│    I/O = Input/Output Block                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

CLB (CONFIGURABLE LOGIC BLOCK):

    ┌─────────────────────────────────────┐
    │                                     │
    │   Inputs ─────┐                     │
    │               ▼                     │
    │         ┌───────────┐               │
    │         │  Look-Up  │               │
    │         │   Table   │               │
    │         │   (LUT)   │               │
    │         └─────┬─────┘               │
    │               │                     │
    │           ┌───┴───┐                 │
    │           │  MUX  │                 │
    │           └───┬───┘                 │
    │               │                     │
    │           ┌───┴───┐                 │
    │           │   D   │                 │
    │           │  F/F  │                 │
    │           └───┬───┘                 │
    │               │                     │
    │   Output ◄────┘                     │
    │                                     │
    └─────────────────────────────────────┘

LUT (Look-Up Table):
  - Small memory (e.g., 16×1 for 4-input LUT)
  - Can implement ANY 4-input function
  - Address = inputs, Data = output
```

### CPLD vs FPGA Comparison

```
┌─────────────────────────────┬───────────────────┬──────────────────┐
│        Feature              │       CPLD        │       FPGA       │
├─────────────────────────────┼───────────────────┼──────────────────┤
│ Architecture                │ AND-OR structure  │ LUT-based        │
│ Configuration storage       │ Non-volatile      │ Usually SRAM     │
│ Power-on                    │ Instant           │ Needs config     │
│ Timing                      │ Predictable       │ Variable         │
│ Complexity                  │ 100-10K gates     │ 10K-10M gates    │
│ Power consumption           │ Lower             │ Higher           │
│ Cost per gate               │ Higher            │ Lower            │
│ Routing flexibility         │ Lower             │ Higher           │
│ On-chip memory              │ Limited           │ Large blocks     │
│ DSP blocks                  │ No                │ Often included   │
│ Embedded processors         │ No                │ Often available  │
├─────────────────────────────┼───────────────────┼──────────────────┤
│ Best for                    │ Glue logic,       │ Complex designs, │
│                             │ simple control    │ prototyping,     │
│                             │ state machines    │ signal processing│
└─────────────────────────────┴───────────────────┴──────────────────┘
```

---

## 10. Summary Tables

### Memory Types Comparison

| Type | Volatile | Read | Write | Erase | Speed | Density | Cost |
|------|----------|------|-------|-------|-------|---------|------|
| SRAM | Yes | Fast | Fast | N/A | Fastest | Low | High |
| DRAM | Yes | Medium | Medium | N/A | Medium | High | Low |
| ROM | No | Fast | N/A | N/A | Fast | Medium | Low |
| PROM | No | Fast | Once | Never | Fast | Medium | Medium |
| EPROM | No | Fast | Many | UV | Fast | Medium | Medium |
| EEPROM | No | Fast | Many | Electric | Medium | Medium | High |
| Flash | No | Fast | Many | Electric | Medium | High | Medium |

### Programmable Logic Comparison

| Device | AND Array | OR Array | Flip-Flops | Complexity |
|--------|-----------|----------|------------|------------|
| PLA | Programmable | Programmable | Optional | Medium |
| PAL | Programmable | Fixed | Optional | Low |
| GAL | Programmable | Fixed | Yes | Low-Med |
| CPLD | Programmable | Fixed | Yes | Medium |
| FPGA | LUT-based | N/A | Yes | High |

### Memory Capacity Terminology

| Term | Size | Exact Value |
|------|------|-------------|
| 1 Kb | 1 Kilobit | 1,024 bits |
| 1 KB | 1 Kilobyte | 1,024 bytes = 8,192 bits |
| 1 Mb | 1 Megabit | 1,048,576 bits |
| 1 MB | 1 Megabyte | 1,048,576 bytes |
| 1 Gb | 1 Gigabit | ~1 billion bits |
| 1 GB | 1 Gigabyte | ~1 billion bytes |

### Address/Capacity Relationship

| Address Bits | Number of Locations | Capacity (8-bit words) |
|--------------|--------------------|-----------------------|
| 8 | 256 | 256 bytes |
| 10 | 1,024 | 1 KB |
| 12 | 4,096 | 4 KB |
| 16 | 65,536 | 64 KB |
| 20 | 1,048,576 | 1 MB |
| 24 | 16,777,216 | 16 MB |
| 32 | 4,294,967,296 | 4 GB |

---

## 11. Quick Revision Questions

### Question 1
What is the difference between SRAM and DRAM?

<details>
<summary>Click for Answer</summary>

```
SRAM (Static RAM):
  - Uses 6 transistors per cell
  - Data stored in cross-coupled inverters
  - No refresh required
  - Faster access time (~1-10 ns)
  - Lower density, higher cost
  - Used for: Cache memory

DRAM (Dynamic RAM):
  - Uses 1 transistor + 1 capacitor per cell
  - Data stored as charge on capacitor
  - Requires periodic refresh (~64 ms)
  - Slower access time (~10-50 ns)
  - Higher density, lower cost
  - Used for: Main memory

KEY DIFFERENCES:
┌────────────────┬──────────────┬──────────────┐
│    Feature     │     SRAM     │     DRAM     │
├────────────────┼──────────────┼──────────────┤
│ Cell size      │ 6T           │ 1T1C         │
│ Refresh        │ No           │ Yes          │
│ Speed          │ Faster       │ Slower       │
│ Density        │ Lower        │ Higher       │
│ Power          │ Lower        │ Higher       │
│ Cost/bit       │ Higher       │ Lower        │
└────────────────┴──────────────┴──────────────┘
```
</details>

### Question 2
How many address lines are needed for a 64KB memory with 8-bit words?

<details>
<summary>Click for Answer</summary>

```
CALCULATION:

Given:
  - Memory size: 64 KB = 64 × 1024 bytes = 65,536 bytes
  - Word size: 8 bits = 1 byte

Number of locations:
  64 KB ÷ 1 byte = 65,536 locations

Number of address lines:
  2ⁿ = 65,536
  n = log₂(65,536)
  n = 16 address lines

VERIFICATION:
  2¹⁶ = 65,536 locations ✓

ANSWER: 16 address lines (A₀ to A₁₅)

GENERAL FORMULA:
  Address lines = ⌈log₂(Memory size in words)⌉
```
</details>

### Question 3
What is the difference between PLA and PAL?

<details>
<summary>Click for Answer</summary>

```
PLA (Programmable Logic Array):
  - Programmable AND array
  - Programmable OR array
  - Maximum flexibility
  - Product terms shared between outputs
  - Slower (more programmable connections)
  - More expensive

PAL (Programmable Array Logic):
  - Programmable AND array
  - FIXED OR array
  - Less flexibility
  - Fixed number of product terms per output
  - Faster (fewer programmable connections)
  - Less expensive

STRUCTURE COMPARISON:

PLA:
  Inputs → [Programmable AND] → [Programmable OR] → Outputs
           (all terms to all outputs possible)

PAL:
  Inputs → [Programmable AND] → [Fixed OR] → Outputs
           (fixed terms per output)

WHEN TO USE:
  PLA: Complex logic with many shared product terms
  PAL: Simpler logic, speed critical, cost sensitive
```
</details>

### Question 4
Explain why DRAM needs refresh and how often.

<details>
<summary>Click for Answer</summary>

```
WHY DRAM NEEDS REFRESH:

1. STORAGE MECHANISM:
   - Data stored as charge on tiny capacitor
   - Capacitor has leakage current
   - Charge gradually drains away

2. CHARGE LEAKAGE:
   - Due to transistor junction leakage
   - Increases with temperature
   - Data lost in milliseconds without refresh

3. REFRESH PROCESS:
   - Read data from cell
   - Sense amplifier restores full voltage
   - Write data back (refresh complete)

REFRESH TIMING:

Standard: Every 64 ms (all rows)

For 8192-row DRAM:
  - 8192 rows ÷ 64 ms = 128 rows per 1 ms
  - One row refreshed every ~7.8 μs
  
Modern DDR:
  - May use 32ms or 64ms intervals
  - Temperature-dependent refresh rate

REFRESH OVERHEAD:
  Refresh takes ~1-2% of available bandwidth
  During refresh, memory cannot be accessed

SELF-REFRESH:
  For low-power/sleep modes
  DRAM generates internal refresh cycles
```
</details>

### Question 5
Design a 16K × 8 memory system using 4K × 4 memory chips.

<details>
<summary>Click for Answer</summary>

```
DESIGN: 16K × 8 from 4K × 4 chips

ANALYSIS:
  Target: 16K × 8 = 16,384 locations × 8 bits
  Chip:   4K × 4 = 4,096 locations × 4 bits

CHIPS NEEDED:
  For 8-bit width: 2 chips (4+4 = 8 bits)
  For 16K depth: 4 rows (4K × 4 = 16K)
  Total: 2 × 4 = 8 chips

ORGANIZATION:
  4 rows × 2 columns = 8 chips total

ADDRESS ALLOCATION:
  Total address: 14 bits (2¹⁴ = 16,384)
  Chip address: 12 bits (2¹² = 4,096)
  Chip select: 2 bits

CIRCUIT:
           A₁₃ A₁₂
             │  │
             ▼  ▼
         ┌────────┐
         │ 2-to-4 │
         │DECODER │
         └────┬───┘
      ┌───┬──┬┴─┬───┐
      ▼   ▼  ▼  ▼   
     CS₀ CS₁ CS₂ CS₃

Row 0:  [4K×4]  [4K×4] → D₇-D₄, D₃-D₀  ← CS₀
Row 1:  [4K×4]  [4K×4] → D₇-D₄, D₃-D₀  ← CS₁
Row 2:  [4K×4]  [4K×4] → D₇-D₄, D₃-D₀  ← CS₂
Row 3:  [4K×4]  [4K×4] → D₇-D₄, D₃-D₀  ← CS₃

A₁₁-A₀ → All 8 chips (internal address)

ADDRESS MAP:
  0000h-0FFFh: Row 0 (CS₀ active)
  1000h-1FFFh: Row 1 (CS₁ active)
  2000h-2FFFh: Row 2 (CS₂ active)
  3000h-3FFFh: Row 3 (CS₃ active)
```
</details>

### Question 6
What is a Look-Up Table (LUT) in an FPGA?

<details>
<summary>Click for Answer</summary>

```
LOOK-UP TABLE (LUT):

DEFINITION:
  Small memory that implements combinational logic
  Can implement ANY boolean function of n inputs

STRUCTURE (4-input LUT):
  - 16 × 1 bit memory (2⁴ = 16 entries)
  - 4 inputs act as address
  - Output = stored value at that address

HOW IT WORKS:

  Inputs: A, B, C, D (4 inputs)
  
  ┌─────────────────────────────┐
  │                             │
  │    ABCD → Address (0-15)    │
  │         │                   │
  │         ▼                   │
  │    ┌─────────────┐          │
  │    │   16 × 1    │          │
  │    │   Memory    │          │
  │    │             │          │
  │    │  0: F(0000) │          │
  │    │  1: F(0001) │          │
  │    │  2: F(0010) │          │
  │    │     ...     │          │
  │    │ 15: F(1111) │          │
  │    └──────┬──────┘          │
  │           │                 │
  │           ▼                 │
  │        Output               │
  │                             │
  └─────────────────────────────┘

EXAMPLE: Implement F = A·B + C·D

  Truth table for F:
    ABCD=0000: F=0, ABCD=0001: F=0, ...
    ABCD=0011: F=1 (CD=1)
    ABCD=1100: F=1 (AB=1)
    etc.

  Store these 16 values in LUT memory
  Any input combination → correct output instantly!

ADVANTAGES:
  - Can implement ANY 4-input function
  - Uniform delay regardless of complexity
  - Easy to reconfigure

TYPICAL SIZES:
  - 4-input LUT (16 entries) - older FPGAs
  - 6-input LUT (64 entries) - modern FPGAs
```
</details>

---

## Navigation

| Previous | [Home](README.md) | Next |
|:--------:|:-----------------:|:----:|
| [Unit 8: FSM](Unit-08-Finite-State-Machines.md) | Return to Index | [Unit 10: Logic Families](Unit-10-Logic-Families.md) |

---

*End of Unit 9*
