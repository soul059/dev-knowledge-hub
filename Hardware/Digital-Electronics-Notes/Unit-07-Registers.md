# Unit 7: Sequential Circuits - Registers

---

## Table of Contents
1. [Introduction to Registers](#1-introduction-to-registers)
2. [Buffer/Storage Registers](#2-bufferstorage-registers)
3. [Shift Register Fundamentals](#3-shift-register-fundamentals)
4. [SISO (Serial-In Serial-Out)](#4-siso-serial-in-serial-out)
5. [SIPO (Serial-In Parallel-Out)](#5-sipo-serial-in-parallel-out)
6. [PISO (Parallel-In Serial-Out)](#6-piso-parallel-in-serial-out)
7. [PIPO (Parallel-In Parallel-Out)](#7-pipo-parallel-in-parallel-out)
8. [Bidirectional Shift Register](#8-bidirectional-shift-register)
9. [Universal Shift Register](#9-universal-shift-register)
10. [Register Applications](#10-register-applications)
11. [Summary Tables](#11-summary-tables)
12. [Quick Revision Questions](#12-quick-revision-questions)

---

## 1. Introduction to Registers

### What is a Register?

```
┌─────────────────────────────────────────────────────────────────┐
│                    REGISTER BASICS                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  DEFINITION:                                                    │
│    A group of flip-flops used to store binary data             │
│    Each flip-flop stores one bit                               │
│                                                                 │
│  KEY CONCEPT:                                                   │
│    n flip-flops = n-bit register = stores n bits               │
│                                                                 │
│  BASIC 4-BIT REGISTER:                                          │
│                                                                 │
│    D₀──┤D Q├──Q₀   D₁──┤D Q├──Q₁   D₂──┤D Q├──Q₂   D₃──┤D Q├──Q₃│
│        │>  │           │>  │           │>  │           │>  │   │
│        └───┘           └───┘           └───┘           └───┘   │
│           │               │               │               │    │
│           └───────────────┴───────────────┴───────────────┘    │
│                                   │                             │
│                                  CLK                            │
│                                                                 │
│  All 4 bits stored simultaneously on clock edge                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Types of Registers

```
┌─────────────────────────────────────────────────────────────────┐
│                 REGISTER CLASSIFICATION                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  BY FUNCTION:                                                   │
│    • Storage Register: Holds data (parallel load/output)       │
│    • Shift Register: Moves data bit by bit                     │
│    • Counter: Special register with count sequence             │
│                                                                 │
│  BY DATA MOVEMENT:                                              │
│                                                                 │
│  ┌──────────────┬──────────────────────────────────────────┐   │
│  │    Type      │          Data Flow                        │   │
│  ├──────────────┼──────────────────────────────────────────┤   │
│  │    SISO      │  Serial In → Shift → Serial Out          │   │
│  │    SIPO      │  Serial In → Shift → Parallel Out        │   │
│  │    PISO      │  Parallel In → Shift → Serial Out        │   │
│  │    PIPO      │  Parallel In → Direct → Parallel Out     │   │
│  └──────────────┴──────────────────────────────────────────┘   │
│                                                                 │
│  BY SHIFT DIRECTION:                                            │
│    • Unidirectional: Right shift OR Left shift                 │
│    • Bidirectional: Both right and left shift                  │
│    • Universal: All modes + parallel load                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Buffer/Storage Registers

### Simple Storage Register

```
4-BIT STORAGE REGISTER WITH PARALLEL LOAD:

              ┌─────────────────────────────────────────────┐
              │           4-BIT STORAGE REGISTER            │
              │                                             │
      D₃ ─────┤──┬────┐                                     │
      D₂ ─────┤──┼──┬─┼───┐                                 │
      D₁ ─────┤──┼──┼─┼───┼──┐                              │
      D₀ ─────┤──┼──┼─┼───┼──┼──┐                           │
              │  │  │ │   │  │  │                           │
              │  ▼  ▼ ▼   ▼  ▼  ▼                           │
              │ ┌──┐┌──┐ ┌──┐┌──┐                           │
              │ │D ││D │ │D ││D │                           │
     CLK ─────┤─┤> │┤> │─┤> │┤> │                           │
              │ │Q ││Q │ │Q ││Q │                           │
              │ └┬─┘└┬─┘ └┬─┘└┬─┘                           │
              │  │   │    │   │                             │
              └──┼───┼────┼───┼─────────────────────────────┘
                 │   │    │   │
                 Q₃  Q₂   Q₁  Q₀

OPERATION:
  On rising edge of CLK:
    Q₃ ← D₃, Q₂ ← D₂, Q₁ ← D₁, Q₀ ← D₀
    
  All 4 bits loaded simultaneously
```

### Register with Load Enable

```
4-BIT REGISTER WITH LOAD ENABLE:

              ┌───────────────────────────────────────────────────┐
              │                                                   │
      D₃ ─────┤──┬─────────────────────────────────────────────── │
      D₂ ─────┤──┼──┬──────────────────────────────────────────── │
      D₁ ─────┤──┼──┼──┬───────────────────────────────────────── │
      D₀ ─────┤──┼──┼──┼──┬────────────────────────────────────── │
              │  │  │  │  │                                       │
    LOAD ─────┤──┼──┼──┼──┼──┬──┬──┬──┬                           │
              │  │  │  │  │  │  │  │  │                           │
              │  ▼  │  ▼  │  ▼  │  ▼  │                           │
              │ ┌─┴─┐│ ┌─┴─┐│ ┌─┴─┐│ ┌─┴─┐                        │
              │ │MUX││ │MUX││ │MUX││ │MUX│                        │
              │ │   ││ │   ││ │   ││ │   │                        │
              │ └─┬─┘│ └─┬─┘│ └─┬─┘│ └─┬─┘                        │
              │   │  │   │  │   │  │   │                          │
              │   ▼  │   ▼  │   ▼  │   ▼                          │
              │ ┌───┐│ ┌───┐│ ┌───┐│ ┌───┐                        │
              │ │ D ││ │ D ││ │ D ││ │ D │                        │
     CLK ─────┤─┤ > │┤─┤ > │┤─┤ > │┤─┤ > │                        │
              │ │ Q ││ │ Q ││ │ Q ││ │ Q │                        │
              │ └─┬─┘│ └─┬─┘│ └─┬─┘│ └─┬─┘                        │
              │   │  │   │  │   │  │   │                          │
              └───┼──┴───┼──┴───┼──┴───┼──────────────────────────┘
                  │      │      │      │
                  Q₃     Q₂     Q₁     Q₀

MUX FUNCTION:
  When LOAD = 1: D input passes to FF
  When LOAD = 0: Q (current value) feeds back to FF (hold)

TRUTH TABLE:
┌──────┬─────┬────────────────────────┐
│ LOAD │ CLK │        Action          │
├──────┼─────┼────────────────────────┤
│   0  │  ↑  │ Hold (Q ← Q)           │
│   1  │  ↑  │ Load (Q ← D)           │
└──────┴─────┴────────────────────────┘
```

### Register with Clear

```
REGISTER WITH SYNCHRONOUS AND ASYNCHRONOUS CLEAR:

SYNCHRONOUS CLEAR (clears on clock edge):
              ┌───────────────────────────────────┐
              │                                   │
      D ──────┤─┬─────────────────────────────────│
              │ │                                 │
    CLR ──────┤─┼──┐                              │
              │ │  │      ┌───┐                   │
              │ └──┼──────┤AND├───┐               │
              │    │      └───┘   │               │
              │    │              ▼               │
              │    │           ┌─────┐            │
              │    └───|>o─────┤  D  │            │
     CLK ─────┤────────────────┤> Q  ├──── Q      │
              │                └─────┘            │
              └───────────────────────────────────┘

D_effective = D · CLR'

ASYNCHRONOUS CLEAR (clears immediately):
              ┌───────────────────────────────────┐
              │                                   │
      D ──────┤─────────────────┐                 │
              │                 │                 │
              │              ┌──▼──┐              │
     CLK ─────┤──────────────┤> D  │              │
              │              │  Q  ├──── Q        │
   CLR' ──────┤──────────────┤CLR  │              │
              │              └─────┘              │
              └───────────────────────────────────┘

When CLR' = 0: Q immediately becomes 0 (regardless of clock)
```

---

## 3. Shift Register Fundamentals

### Basic Shift Operation

```
┌─────────────────────────────────────────────────────────────────┐
│                  SHIFT REGISTER CONCEPT                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Data moves from one flip-flop to the next on each clock        │
│                                                                 │
│  RIGHT SHIFT (→):                                               │
│    Before: Q₃ Q₂ Q₁ Q₀                                         │
│            │   │   │   │                                        │
│            ▼   ▼   ▼   ▼                                        │
│    After:  Din Q₃ Q₂ Q₁    (Q₀ shifted out)                    │
│                                                                 │
│  LEFT SHIFT (←):                                                │
│    Before: Q₃ Q₂ Q₁ Q₀                                         │
│            │   │   │   │                                        │
│            ▼   ▼   ▼   ▼                                        │
│    After:  Q₂ Q₁ Q₀ Din    (Q₃ shifted out)                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4-Bit Right Shift Register

```
4-BIT RIGHT SHIFT REGISTER:

    Serial    ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐    Serial
    Input ────┤D   Q├────┤D   Q├────┤D   Q├────┤D   Q├──── Output
    (SI)      │     │    │     │    │     │    │     │    (SO)
              │>    │    │>    │    │>    │    │>    │
              └──┬──┘    └──┬──┘    └──┬──┘    └──┬──┘
                 │          │          │          │
                 Q₃         Q₂         Q₁         Q₀
                 │          │          │          │
                 └──────────┴──────────┴──────────┘
                                 │
                                CLK

Data shifts right on each clock edge:
  Q₃ ← SI (serial input)
  Q₂ ← Q₃
  Q₁ ← Q₂
  Q₀ ← Q₁
  SO ← Q₀ (before shift)

TIMING EXAMPLE:
Loading 1011 serially (MSB first):

┌───────┬────┬─────┬─────┬─────┬─────┐
│ Clock │ SI │ Q₃  │ Q₂  │ Q₁  │ Q₀  │
├───────┼────┼─────┼─────┼─────┼─────┤
│ Init  │  - │  0  │  0  │  0  │  0  │
│   1   │  1 │  1  │  0  │  0  │  0  │
│   2   │  0 │  0  │  1  │  0  │  0  │
│   3   │  1 │  1  │  0  │  1  │  0  │
│   4   │  1 │  1  │  1  │  0  │  1  │
└───────┴────┴─────┴─────┴─────┴─────┘

After 4 clocks: Q₃Q₂Q₁Q₀ = 1011 ✓
```

---

## 4. SISO (Serial-In Serial-Out)

### Concept and Circuit

```
┌─────────────────────────────────────────────────────────────────┐
│                 SISO SHIFT REGISTER                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  INPUT: One bit at a time (serial)                             │
│  OUTPUT: One bit at a time (serial)                            │
│                                                                 │
│  Application: Time delay element                               │
│    - Data takes n clock cycles to pass through n-bit register  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

4-BIT SISO:

    SI ──────┤D Q├────┤D Q├────┤D Q├────┤D Q├────── SO
             │>  │    │>  │    │>  │    │>  │
             └───┘    └───┘    └───┘    └───┘
               │        │        │        │
               └────────┴────────┴────────┘
                           │
                          CLK

TIMING DIAGRAM:

    CLK ──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──
          └──┘  └──┘  └──┘  └──┘  └──┘  └──┘  └──┘

    SI  ──┐     ┌──────────┐     ┌─────────────────
          └─────┘          └─────┘
          (1)   (0)   (0)   (1)

    Q₃  ────────┐     ┌──────────┐     ┌───────────
                └─────┘          └─────┘

    Q₂  ──────────────┐     ┌──────────┐     ┌─────
                      └─────┘          └─────┘

    Q₁  ────────────────────┐     ┌──────────┐
                            └─────┘          └─────

    SO  ──────────────────────────┐     ┌──────────
    (Q₀)                          └─────┘

    Data: 1-0-0-1 in → delayed by 4 clocks → 1-0-0-1 out
```

### Applications of SISO

```
1. TIME DELAY:
   n-bit SISO provides n clock-cycle delay
   
   Delay = n / f_clock
   
   Example: 8-bit SISO at 1MHz = 8μs delay

2. TEMPORARY STORAGE:
   Store serial data during processing
   
3. SERIAL DATA TRANSMISSION LINE:
   Act as buffer in serial communication
```

---

## 5. SIPO (Serial-In Parallel-Out)

### Concept and Circuit

```
┌─────────────────────────────────────────────────────────────────┐
│                 SIPO SHIFT REGISTER                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  INPUT: One bit at a time (serial)                             │
│  OUTPUT: All bits available simultaneously (parallel)          │
│                                                                 │
│  Application: Serial-to-Parallel Conversion                    │
│    - Receives serial data, outputs parallel data               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

4-BIT SIPO:

                     ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐
    Serial ──────────┤D   Q├────┤D   Q├────┤D   Q├────┤D   Q├────
    Input (SI)       │     │    │     │    │     │    │     │
                     │>    │    │>    │    │>    │    │>    │
                     └──┬──┘    └──┬──┘    └──┬──┘    └──┬──┘
                        │          │          │          │
                        Q₃         Q₂         Q₁         Q₀
                        │          │          │          │
                        ▼          ▼          ▼          ▼
                     ┌──┴──────────┴──────────┴──────────┴──┐
                     │        PARALLEL OUTPUT               │
                     │          Q₃ Q₂ Q₁ Q₀                 │
                     └──────────────────────────────────────┘
                                    │
                                   CLK
```

### Serial-to-Parallel Conversion

```
SERIAL-TO-PARALLEL CONVERSION EXAMPLE:

Converting serial data 1101 to parallel:

STEP-BY-STEP:
┌───────┬────┬─────┬─────┬─────┬─────┬───────────────────┐
│ Clock │ SI │ Q₃  │ Q₂  │ Q₁  │ Q₀  │      Action       │
├───────┼────┼─────┼─────┼─────┼─────┼───────────────────┤
│ Init  │  - │  0  │  0  │  0  │  0  │ Clear register    │
│   1   │  1 │  1  │  0  │  0  │  0  │ Shift in MSB (1)  │
│   2   │  1 │  1  │  1  │  0  │  0  │ Shift in (1)      │
│   3   │  0 │  0  │  1  │  1  │  0  │ Shift in (0)      │
│   4   │  1 │  1  │  0  │  1  │  1  │ Shift in LSB (1)  │
└───────┴────┴─────┴─────┴─────┴─────┴───────────────────┘

After 4 clocks:
  Parallel output = Q₃Q₂Q₁Q₀ = 1011
  
Wait... that's not 1101!

IMPORTANT: Data order depends on which bit sent first!

If LSB first (1-0-1-1):
  Final parallel = 1101 ✓ (Q₃=1, Q₂=1, Q₁=0, Q₀=1)

TIMING DIAGRAM:

    CLK ────┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐
            └───┘   └───┘   └───┘   └───┘   └──

    SI  ────┐       ┌───────────┐       ┌──────
            └───────┘           └───────┘
            LSB=1    1    0    MSB=1

    Q₃  ────────────┐           ┌───────┐
                    └───────────┘       └──────
    Q₂  ────────────────────────┐
                                └──────────────
    Q₁  ────────────────────────────────┐
                                        └──────
    Q₀  ────┐       ┌───────────┐
            └───────┘           └──────────────

After 4th clock: Parallel out = 1101
```

### Applications

```
1. SERIAL COMMUNICATION RECEIVER:
   - UART receiver
   - SPI slave
   - I²C interface

2. DATA ACQUISITION:
   - ADC serial output to parallel

3. KEYBOARD INTERFACE:
   - Serial key codes to parallel bus

BLOCK DIAGRAM - Serial Receiver:

    Serial     ┌───────────┐    ┌────────────┐
    Line ──────┤   SIPO    ├────┤  PARALLEL  ├──── To CPU
               │  Register │    │   BUFFER   │
               └─────┬─────┘    └─────┬──────┘
                     │                │
                  Shift             Load
                  Clock            Signal
```

---

## 6. PISO (Parallel-In Serial-Out)

### Concept and Circuit

```
┌─────────────────────────────────────────────────────────────────┐
│                 PISO SHIFT REGISTER                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  INPUT: All bits loaded simultaneously (parallel)              │
│  OUTPUT: One bit at a time (serial)                            │
│                                                                 │
│  Application: Parallel-to-Serial Conversion                    │
│    - Loads parallel data, shifts out serially                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

4-BIT PISO WITH LOAD CONTROL:

    D₃         D₂         D₁         D₀
    │          │          │          │
    ▼          ▼          ▼          ▼
  ┌───┐      ┌───┐      ┌───┐      ┌───┐
  │MUX│      │MUX│      │MUX│      │MUX│
  │ 1 │      │ 1 │      │ 1 │      │ 1 │
  │ 0 │      │ 0 │      │ 0 │      │ 0 │
  └─┬─┘      └─┬─┘      └─┬─┘      └─┬─┘
    │   ┌──────┘   ┌──────┘   ┌──────┘
    │   │          │          │
    ▼   ▼          ▼          ▼
  ┌─────┐      ┌─────┐      ┌─────┐      ┌─────┐
  │D   Q├──────┤D   Q├──────┤D   Q├──────┤D   Q├──── SO
  │>    │      │>    │      │>    │      │>    │
  └─────┘      └─────┘      └─────┘      └─────┘
      │            │            │            │
      Q₃           Q₂           Q₁           Q₀
      │            │            │            │
      └────────────┴────────────┴────────────┘
                        │
                       CLK

LOAD/SHIFT control to all MUX:
  LOAD/SHIFT' = 1: Load parallel data (D → Q)
  LOAD/SHIFT' = 0: Shift right (data moves toward SO)
```

### Parallel-to-Serial Conversion

```
PARALLEL-TO-SERIAL CONVERSION EXAMPLE:

Converting parallel data 1101 to serial:

STEP-BY-STEP:
┌────────────┬─────┬─────┬─────┬─────┬─────┬───────────────┐
│   Mode     │ L/S'│ Q₃  │ Q₂  │ Q₁  │ Q₀  │     SO        │
├────────────┼─────┼─────┼─────┼─────┼─────┼───────────────┤
│ Load       │  1  │  1  │  1  │  0  │  1  │     1         │
│ Shift 1    │  0  │  0  │  1  │  1  │  0  │     1 → out   │
│ Shift 2    │  0  │  0  │  0  │  1  │  1  │     0 → out   │
│ Shift 3    │  0  │  0  │  0  │  0  │  1  │     1 → out   │
│ Shift 4    │  0  │  0  │  0  │  0  │  0  │     1 → out   │
└────────────┴─────┴─────┴─────┴─────┴─────┴───────────────┘

Serial output sequence: 1-0-1-1 (LSB first)

TIMING DIAGRAM:

    L/S'────┐                                         ┌────
            └─────────────────────────────────────────┘
            Load                Shift

    CLK ────┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐
            └───┘   └───┘   └───┘   └───┘   └───┘   └──
             1      2      3      4      5

    D   ────────────────────────────────────────────────
    (1101)

    SO  ────────────┐       ┌───────────┐       ┌──────
    (Q₀)            └───────┘           └───────┘
                    LSB=1    0     1    MSB=1

    Data loaded: 1101
    Serial out: 1-0-1-1 (LSB first) or 1-1-0-1 (MSB first)
```

### Applications

```
1. SERIAL COMMUNICATION TRANSMITTER:
   - UART transmitter
   - SPI master
   - Parallel port to serial

2. DATA SERIALIZATION:
   - CPU data bus to serial link
   - Multiplexing parallel lines

3. LED DISPLAY DRIVER:
   - Send display data serially to reduce wires

BLOCK DIAGRAM - Serial Transmitter:

    From     ┌────────────┐    ┌───────────┐    Serial
    CPU ─────┤  PARALLEL  ├────┤   PISO    ├──── Line
             │   BUFFER   │    │  Register │
             └─────┬──────┘    └─────┬─────┘
                   │                 │
                 Load              Shift
                Signal             Clock
```

---

## 7. PIPO (Parallel-In Parallel-Out)

### Concept and Circuit

```
┌─────────────────────────────────────────────────────────────────┐
│                 PIPO SHIFT REGISTER                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  INPUT: All bits loaded simultaneously (parallel)              │
│  OUTPUT: All bits available simultaneously (parallel)          │
│                                                                 │
│  Actually: This is a standard storage register!                │
│                                                                 │
│  Can also support shift operations                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

4-BIT PIPO:

    D₃          D₂          D₁          D₀
    │           │           │           │
    ▼           ▼           ▼           ▼
  ┌─────┐     ┌─────┐     ┌─────┐     ┌─────┐
  │D   Q├     │D   Q├     │D   Q├     │D   Q├
  │     │     │     │     │     │     │     │
  │>    │     │>    │     │>    │     │>    │
  └──┬──┘     └──┬──┘     └──┬──┘     └──┬──┘
     │           │           │           │
     Q₃          Q₂          Q₁          Q₀
     │           │           │           │
     ▼           ▼           ▼           ▼
   OUTPUT      OUTPUT      OUTPUT      OUTPUT
   
   All inputs to outputs in ONE clock cycle!

TIMING DIAGRAM:

    CLK ────────┐       ┌───────────────────
                └───────┘
                    ↑
                  Load

    D   ──────────╳═══════════════════════
            Old    New Data (stable)

    Q   ──────────────────╳═══════════════
                   Old     New Data
                          ↑
                      Data appears
                      immediately
                      after clock
```

### Applications

```
1. BUFFER REGISTER:
   - Temporary data storage
   - Pipeline stages in CPU

2. DATA LATCH:
   - Hold data from bus
   - Interface between circuits

3. ACCUMULATOR:
   - Store computation results
   - With feedback for accumulation

PIPO WITH ACCUMULATE FUNCTION:

              ┌──────────────────────────────────────┐
              │                                      │
    D ────────┤───┬──────────────────────────────────│
              │   │                                  │
              │   ▼                                  │
              │ ┌─────┐                              │
    Q(prev)───│─┤ ADD ├───┐                         │
              │ └─────┘   │                         │
              │           ▼                         │
              │       ┌─────┐                       │
              │       │ PIPO│                       │
     CLK ─────│───────┤>    ├───────────── Q (new) │
              │       └─────┘                       │
              │           │                         │
              └───────────┴─────────────────────────┘
              
    Q(new) = Q(prev) + D  (Accumulator)
```

---

## 8. Bidirectional Shift Register

### Concept

```
┌─────────────────────────────────────────────────────────────────┐
│              BIDIRECTIONAL SHIFT REGISTER                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Can shift data in either direction:                           │
│    - RIGHT SHIFT: Data moves MSB → LSB                         │
│    - LEFT SHIFT: Data moves LSB → MSB                          │
│                                                                 │
│  Control signal determines direction                           │
│                                                                 │
│  DIR = 1: Right shift    DIR = 0: Left shift                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Circuit Design

```
4-BIT BIDIRECTIONAL SHIFT REGISTER:

  SI_L ───────────────────────────────────────────────────┐
  SI_R ──┐                                                │
         │                                                │
         │    ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐   │
         │    │     │    │     │    │     │    │     │   │
         └────┤1    │────┤1    │────┤1    │────┤1    │   │
              │ MUX │    │ MUX │    │ MUX │    │ MUX │   │
         ┌────┤0    │    │0    │    │0    │    │0    │───┘
         │    └──┬──┘    └──┬──┘    └──┬──┘    └──┬──┘
         │       │          │          │          │
         │       ▼          ▼          ▼          ▼
         │    ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐
         │    │D   Q├────┤D   Q├────┤D   Q├────┤D   Q├──── SO_R
         │    │     │    │     │    │     │    │     │
         │    │>    │    │>    │    │>    │    │>    │
         │    └──┬──┘    └──┬──┘    └──┬──┘    └──┬──┘
         │       │          │          │          │
         │       Q₃         Q₂         Q₁         Q₀
         │       │          │          │          │
         └───────┴──────────┴──────────┴──────────┘
         ↑       ↑          ↑          ↑
         │       │          │          │
         │       Left shift path (→)   │
         │                             │
         Right shift path (←)──────────┘
         
         DIR ────────────────────────────────
                    (to all MUX select)

MUX LOGIC:
  DIR = 1 (Right): Each FF gets input from left neighbor
  DIR = 0 (Left):  Each FF gets input from right neighbor

  D₃ = DIR·SI_R + DIR'·Q₂
  D₂ = DIR·Q₃ + DIR'·Q₁
  D₁ = DIR·Q₂ + DIR'·Q₀
  D₀ = DIR·Q₁ + DIR'·SI_L
```

### Operation Example

```
BIDIRECTIONAL SHIFT EXAMPLE:

Initial: Q₃Q₂Q₁Q₀ = 1010

RIGHT SHIFT (DIR=1), SI_R = 0:
  Before: 1 0 1 0
  After:  0 1 0 1  (0 shifted in from left)

LEFT SHIFT (DIR=0), SI_L = 1:
  Before: 1 0 1 0
  After:  0 1 0 1  (1 shifted in from right)

TIMING DIAGRAM:

    DIR ──────────┐                     ┌──────────────
    (0=L, 1=R)    └─────────────────────┘
                   Left                  Right

    CLK ──────┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌───
              └───┘   └───┘   └───┘   └───┘   └───┘

    Q   ══════╳═══════╳═══════╳═══════╳═══════╳═══════
          0101  1011   0111   1110   0111   0011
              shift   shift  shift  shift  shift
               left    left   left  right  right
```

---

## 9. Universal Shift Register

### Concept

```
┌─────────────────────────────────────────────────────────────────┐
│              UNIVERSAL SHIFT REGISTER                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Combines ALL shift register functions:                        │
│                                                                 │
│    • Parallel Load                                             │
│    • Serial In (left or right)                                 │
│    • Shift Right                                               │
│    • Shift Left                                                │
│    • Hold (no change)                                          │
│                                                                 │
│  Control: Mode select lines S₁S₀                               │
│                                                                 │
│  ┌─────┬─────┬──────────────────────────────────┐              │
│  │ S₁  │ S₀  │           Mode                   │              │
│  ├─────┼─────┼──────────────────────────────────┤              │
│  │  0  │  0  │  Hold (no change)                │              │
│  │  0  │  1  │  Shift Right                     │              │
│  │  1  │  0  │  Shift Left                      │              │
│  │  1  │  1  │  Parallel Load                   │              │
│  └─────┴─────┴──────────────────────────────────┘              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Circuit (74194 Style)

```
4-BIT UNIVERSAL SHIFT REGISTER (74194):

              ┌───────────────────────────────────────────────────┐
              │                                                   │
              │      D₃         D₂         D₁         D₀          │
              │      │          │          │          │           │
              │      ▼          ▼          ▼          ▼           │
  Serial   ───┤───┬─────┬───┬─────┬───┬─────┬───┬─────┬           │
  Right       │   │ 4:1 │   │ 4:1 │   │ 4:1 │   │ 4:1 │           │
  (SR)        │   │ MUX │   │ MUX │   │ MUX │   │ MUX │           │
              │   └──┬──┘   └──┬──┘   └──┬──┘   └──┬──┘           │
              │      │          │          │          │           │
  Serial   ───┤      ▼          ▼          ▼          ▼    ───────┤
  Left        │   ┌─────┐   ┌─────┐   ┌─────┐   ┌─────┐           │
  (SL)        │   │D   Q├───┤D   Q├───┤D   Q├───┤D   Q│           │
              │   │     │   │     │   │     │   │     │           │
  CLK    ─────┤───┤>    │───┤>    │───┤>    │───┤>    │           │
              │   └──┬──┘   └──┬──┘   └──┬──┘   └──┬──┘           │
              │      │          │          │          │           │
              │      Q₃         Q₂         Q₁         Q₀          │
              │      │          │          │          │           │
  S₁ ─────────┤──────┴──────────┴──────────┴──────────┴───────────┤
  S₀ ─────────┤                                                   │
              │                                                   │
  CLR' ───────┤                                                   │
              └───────────────────────────────────────────────────┘

4:1 MUX FOR EACH FF:
┌───────────────────────────────────────────────────────────────┐
│                                                               │
│  For Q₃:                         For Q₀:                      │
│    S₁S₀=00: Q₃ (hold)             S₁S₀=00: Q₀ (hold)         │
│    S₁S₀=01: SR (right in)         S₁S₀=01: Q₁ (from left)    │
│    S₁S₀=10: Q₂ (from right)       S₁S₀=10: SL (left in)      │
│    S₁S₀=11: D₃ (load)             S₁S₀=11: D₀ (load)         │
│                                                               │
└───────────────────────────────────────────────────────────────┘

BLOCK SYMBOL (74194):

              ┌─────────────────────┐
              │                     │
      CLR' ───┤ CLR                 │
              │                     │
       SR  ───┤ SR              Q₀  ├─── Q₀
       SL  ───┤ SL              Q₁  ├─── Q₁
              │                 Q₂  ├─── Q₂
       D₀  ───┤ D₀              Q₃  ├─── Q₃
       D₁  ───┤ D₁                  │
       D₂  ───┤ D₂      74194      │
       D₃  ───┤ D₃                  │
              │                     │
       S₀  ───┤ S₀                  │
       S₁  ───┤ S₁                  │
              │                     │
      CLK  ───┤>                    │
              └─────────────────────┘
```

### Function Table

```
74194 FUNCTION TABLE:

┌──────┬─────┬─────┬─────┬────────────────────────────────────────┐
│ CLR' │ CLK │ S₁  │ S₀  │              Function                  │
├──────┼─────┼─────┼─────┼────────────────────────────────────────┤
│   0  │  X  │  X  │  X  │  Clear (Q = 0000)                      │
│   1  │  ↑  │  0  │  0  │  Hold (no change)                      │
│   1  │  ↑  │  0  │  1  │  Shift Right (SR → Q₃ → Q₂ → Q₁ → Q₀)  │
│   1  │  ↑  │  1  │  0  │  Shift Left (Q₃ ← Q₂ ← Q₁ ← Q₀ ← SL)   │
│   1  │  ↑  │  1  │  1  │  Parallel Load (Qᵢ ← Dᵢ)               │
└──────┴─────┴─────┴─────┴────────────────────────────────────────┘

OPERATION EXAMPLE:

Initial: Q = 0000

1. Parallel Load 1011:
   S₁S₀ = 11, D = 1011 → Q = 1011

2. Shift Right with SR = 0:
   S₁S₀ = 01 → Q = 0101 (1011 → 0101)

3. Shift Left with SL = 1:
   S₁S₀ = 10 → Q = 1011 (0101 → 1011)

4. Hold:
   S₁S₀ = 00 → Q = 1011 (unchanged)
```

---

## 10. Register Applications

### 10.1 Serial Data Communication

```
UART (Universal Asynchronous Receiver/Transmitter):

TRANSMITTER (uses PISO):
              ┌───────────────────────────────────────┐
              │                                       │
    Parallel  │    ┌─────────┐    ┌─────────┐        │
    Data ─────│────┤  PISO   ├────┤  Line   ├────────│──── TX
              │    │Register │    │ Driver  │        │
              │    └─────────┘    └─────────┘        │
              │         │                            │
              │     Baud Rate                        │
              │      Clock                           │
              └───────────────────────────────────────┘

RECEIVER (uses SIPO):
              ┌───────────────────────────────────────┐
              │                                       │
    RX ───────│────┤  Line   ├────┤  SIPO   ├────────│──── Parallel
              │    │Receiver │    │Register │        │     Data
              │    └─────────┘    └─────────┘        │
              │                       │              │
              │                   Baud Rate          │
              │                    Clock             │
              └───────────────────────────────────────┘
```

### 10.2 Sequence Generator

```
SEQUENCE GENERATOR USING SHIFT REGISTER:

Example: Generate sequence 1011 repeatedly

    ┌────────────────────────────────────────────────────────────┐
    │                                                            │
    │  ┌───────────────────────────────────────────────────┐    │
    │  │                                                   │    │
    │  │   ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐       │    │
    │  └───┤D   Q├────┤D   Q├────┤D   Q├────┤D   Q├───────┘    │
    │      │     │    │     │    │     │    │     │            │
    │      │>    │    │>    │    │>    │    │>    │            │
    │      └──┬──┘    └──┬──┘    └──┬──┘    └──┬──┘            │
    │         │          │          │          │               │
    │         Q₃         Q₂         Q₁         Q₀              │
    │                                          │               │
    │                                     Sequence             │
    │                                      Output              │
    └──────────────────────────────────────────────────────────┘

Initialize with 1011, then recirculates.
Output: 1-0-1-1-1-0-1-1-1-0-1-1...

PSEUDO-RANDOM SEQUENCE (LFSR):
Linear Feedback Shift Register generates pseudo-random bits

    ┌────────────────────────────────────────────────────────────┐
    │                                                            │
    │   ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐                │
    │   │D   Q├────┤D   Q├────┤D   Q├────┤D   Q├──── Output     │
    │   │     │    │     │    │     │    │     │                │
    │   │>    │    │>    │    │>    │    │>    │                │
    │   └──┬──┘    └──┬──┘    └──┬──┘    └──┬──┘                │
    │      │          │          │          │                   │
    │      Q₃         Q₂         Q₁         Q₀                  │
    │      │                                │                   │
    │      └─────────────XOR────────────────┘                   │
    │                     │                                     │
    │                     └─────────────────────────────────────┘
    │                           Feedback                        │
    │                                                            │
    └────────────────────────────────────────────────────────────┘

Feedback: D₀ = Q₃ ⊕ Q₀
Generates 2ⁿ - 1 states (for proper tap selection)
```

### 10.3 Arithmetic Operations

```
SHIFT REGISTER FOR MULTIPLICATION/DIVISION:

MULTIPLY BY 2 (Left Shift):
  Before: 0101₂ = 5₁₀
  After:  1010₂ = 10₁₀
  
  5 × 2 = 10 ✓

DIVIDE BY 2 (Right Shift):
  Before: 1010₂ = 10₁₀
  After:  0101₂ = 5₁₀
  
  10 ÷ 2 = 5 ✓

SERIAL ADDER USING SHIFT REGISTERS:

    A ──────┤ SIPO ├──┐
            │      │  │     ┌─────┐
            └──────┘  ├─────┤     │
                      │     │ FA  ├───────┤ SISO ├──── Sum
    B ──────┤ SIPO ├──┘  ┌──┤     │       │      │
            │      │     │  └─────┘       └──────┘
            └──────┘     │      │
                         │      │
                         └──────┘
                         Carry FF

Add two n-bit numbers using only:
  - 2 SIPO registers (hold A and B)
  - 1 Full Adder
  - 1 Carry flip-flop
  - 1 SISO register (accumulate sum)
```

### 10.4 LED Display Multiplexing

```
LED MATRIX DRIVER USING SHIFT REGISTERS:

8x8 LED Matrix with only 3 control wires!

    ┌───────────────────────────────────────────────────────────┐
    │                                                           │
    │  Data ────┤ SIPO ├─────┐                                 │
    │           │ (Row)│     │                                 │
    │           └──────┘     │    ┌───────────────────────┐    │
    │                        └────┤                       │    │
    │                             │      8x8 LED          │    │
    │  Data ────┤ SIPO ├─────┬────┤       Matrix          │    │
    │           │(Col) │     │    │                       │    │
    │           └──────┘     │    └───────────────────────┘    │
    │                        │                                 │
    │                     Column                               │
    │                    Drivers                               │
    │                                                           │
    │  CLK ─────────────────────                               │
    │  LATCH ───────────────────                               │
    │                                                           │
    └───────────────────────────────────────────────────────────┘

Only 3 wires (Data, Clock, Latch) control 64 LEDs!
```

---

## 11. Summary Tables

### Shift Register Types

| Type | Serial In | Serial Out | Parallel In | Parallel Out |
|------|-----------|------------|-------------|--------------|
| SISO | ✓ | ✓ | ✗ | ✗ |
| SIPO | ✓ | Optional | ✗ | ✓ |
| PISO | ✗ | ✓ | ✓ | Optional |
| PIPO | ✗ | ✗ | ✓ | ✓ |

### Operation Summary

| Register Type | Primary Function | Clock Cycles for n-bits |
|---------------|------------------|-------------------------|
| SISO | Time delay | n cycles (in) + n cycles (out) |
| SIPO | Serial-to-Parallel | n cycles to load |
| PISO | Parallel-to-Serial | 1 load + n cycles out |
| PIPO | Storage | 1 cycle |
| Bidirectional | Left/Right shift | 1 cycle per shift |
| Universal | All modes | Depends on mode |

### Common Register ICs

| IC Number | Type | Bits | Features |
|-----------|------|------|----------|
| 7495 | PISO/SIPO | 4 | Right shift, parallel load |
| 74164 | SIPO | 8 | Serial in, parallel out |
| 74165 | PISO | 8 | Parallel load, serial out |
| 74194 | Universal | 4 | All modes, bidirectional |
| 74195 | Universal | 4 | Parallel load, J-K input |
| 74299 | Universal | 8 | Tri-state outputs |

### Mode Selection (Universal Register 74194)

| S₁ | S₀ | Mode |
|----|-----|------|
| 0 | 0 | Hold |
| 0 | 1 | Shift Right |
| 1 | 0 | Shift Left |
| 1 | 1 | Parallel Load |

---

## 12. Quick Revision Questions

### Question 1
How many clock pulses are needed to load 8 bits into an 8-bit SIPO register?

<details>
<summary>Click for Answer</summary>

```
SIPO (Serial-In Parallel-Out):

8 clock pulses are needed.

Each clock pulse shifts in one bit.
After 8 pulses, all 8 bits are loaded and available
at the parallel outputs simultaneously.

TIMING:
  CLK:  1  2  3  4  5  6  7  8
  Bit:  b₇ b₆ b₅ b₄ b₃ b₂ b₁ b₀
                              ↑
                         After 8th clock,
                         all bits available
```
</details>

### Question 2
What is the difference between SIPO and PISO registers?

<details>
<summary>Click for Answer</summary>

```
SIPO (Serial-In Parallel-Out):
  - Data enters one bit at a time (serial)
  - Data exits all bits at once (parallel)
  - Used for: Serial-to-Parallel conversion
  - Example: Serial receiver
  
  [Serial In] → □→□→□→□ → [Parallel Out]
                          Q₃Q₂Q₁Q₀

PISO (Parallel-In Serial-Out):
  - Data enters all bits at once (parallel)
  - Data exits one bit at a time (serial)
  - Used for: Parallel-to-Serial conversion
  - Example: Serial transmitter

  [Parallel In] → □□□□ → [Serial Out]
   D₃D₂D₁D₀       ↓↓↓↓

KEY DIFFERENCE:
  SIPO: n clocks to load, instant read
  PISO: instant load, n clocks to read
```
</details>

### Question 3
Design a 4-bit right shift register using D flip-flops. Show the circuit.

<details>
<summary>Click for Answer</summary>

```
4-BIT RIGHT SHIFT REGISTER:

    SI ──────┤D  Q├────┤D  Q├────┤D  Q├────┤D  Q├──── SO
             │    │    │    │    │    │    │    │
             │>   │    │>   │    │>   │    │>   │
             └────┘    └────┘    └────┘    └────┘
                │          │         │         │
               Q₃         Q₂        Q₁        Q₀
               
    CLK ────────┴──────────┴─────────┴─────────┘

Connection Rule:
  - Q of each FF connects to D of next FF
  - All FFs share the same clock
  - SI (Serial Input) goes to leftmost D
  - SO (Serial Output) from rightmost Q

Operation:
  On each clock edge:
    Q₀ ← Q₁
    Q₁ ← Q₂
    Q₂ ← Q₃
    Q₃ ← SI
```
</details>

### Question 4
Explain the function of a universal shift register with its mode table.

<details>
<summary>Click for Answer</summary>

```
UNIVERSAL SHIFT REGISTER:

A register that can perform ALL shift operations:
  1. Hold (store data, no change)
  2. Shift Right
  3. Shift Left
  4. Parallel Load

MODE TABLE (74194):
┌─────┬─────┬──────────────────────────────┐
│ S₁  │ S₀  │         Function             │
├─────┼─────┼──────────────────────────────┤
│  0  │  0  │ Hold (Q unchanged)           │
│  0  │  1  │ Shift Right (SR → Q₃→Q₂→Q₁→Q₀)│
│  1  │  0  │ Shift Left (Q₃←Q₂←Q₁←Q₀← SL) │
│  1  │  1  │ Parallel Load (Q ← D)        │
└─────┴─────┴──────────────────────────────┘

Implementation:
  - Uses 4:1 MUX at each FF input
  - MUX selects between:
    0: Current Q (hold)
    1: Right neighbor (shift right)
    2: Left neighbor (shift left)
    3: Parallel data D (load)

Applications:
  - General purpose data manipulation
  - Arithmetic shift (multiply/divide by 2)
  - Data conversion (serial ↔ parallel)
```
</details>

### Question 5
What is LFSR? Give one application.

<details>
<summary>Click for Answer</summary>

```
LFSR (Linear Feedback Shift Register):

A shift register with feedback through XOR gates.
Generates pseudo-random binary sequence (PRBS).

STRUCTURE:
    ┌──────────────────────────────────────────────────┐
    │                                                  │
    │   □───□───□───□ ─── Output                      │
    │   ↑       │   │                                 │
    │   └──XOR──┴───┘                                 │
    │   Feedback taps                                 │
    └──────────────────────────────────────────────────┘

For n-bit LFSR with proper taps:
  - Generates 2ⁿ - 1 unique states
  - Sequence appears random but repeatable
  - Same seed = same sequence

APPLICATIONS:
1. Pseudo-Random Number Generator (PRNG)
2. Data Encryption/Scrambling
3. Error Detection (CRC calculation)
4. Built-In Self-Test (BIST)
5. Spread Spectrum Communication
6. White Noise Generator

Example: 4-bit LFSR with taps at Q₃ and Q₀
  Generates 15 unique states before repeating
  Period = 2⁴ - 1 = 15
```
</details>

### Question 6
How can a shift register be used for multiplication and division?

<details>
<summary>Click for Answer</summary>

```
MULTIPLICATION BY 2 (Left Shift):

Original: 0011₂ = 3₁₀
Left shift by 1:
  0011 → 0110₂ = 6₁₀

3 × 2 = 6 ✓

Left shift by n positions = multiply by 2ⁿ

Example: 3 × 8 = 3 × 2³
  0011 → 0110 → 1100 → 11000₂ = 24₁₀ ✓


DIVISION BY 2 (Right Shift):

Original: 1100₂ = 12₁₀
Right shift by 1:
  1100 → 0110₂ = 6₁₀

12 ÷ 2 = 6 ✓

Right shift by n positions = divide by 2ⁿ

Example: 24 ÷ 4 = 24 ÷ 2²
  11000 → 01100 → 00110₂ = 6₁₀ ✓

NOTE: For signed numbers, use arithmetic shift
(preserves sign bit) rather than logical shift.
```
</details>

---

## Navigation

| Previous | [Home](README.md) | Next |
|:--------:|:-----------------:|:----:|
| [Unit 6: Counters](Unit-06-Counters.md) | Return to Index | [Unit 8: FSM](Unit-08-Finite-State-Machines.md) |

---

*End of Unit 7*
