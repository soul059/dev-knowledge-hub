# Chapter 5.1: Serial Communication Basics

## Introduction

Serial communication transmits data one bit at a time over a communication channel. Understanding the fundamental concepts of serial communication is essential for implementing any embedded communication protocol.

---

## Data Transmission Modes

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TRANSMISSION MODES                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SIMPLEX: One direction only                                        │
│                                                                      │
│     TX ────────────────────────────────────────────▶ RX             │
│                                                                      │
│     Example: Broadcast radio, TV                                    │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  HALF DUPLEX: Bidirectional, one at a time                         │
│                                                                      │
│     A ◀───────────────────────────────────────────── B              │
│                      OR                                              │
│     A ─────────────────────────────────────────────▶ B              │
│                                                                      │
│     Example: Walkie-talkie, RS-485                                  │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  FULL DUPLEX: Both directions simultaneously                        │
│                                                                      │
│     TX ────────────────────────────────────────────▶ RX             │
│     RX ◀──────────────────────────────────────────── TX             │
│                                                                      │
│     Example: UART, SPI, Telephone                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Baud Rate vs Bit Rate

```
┌─────────────────────────────────────────────────────────────────────┐
│                    BAUD RATE vs BIT RATE                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  BAUD RATE: Number of signal changes (symbols) per second          │
│  BIT RATE:  Number of data bits transmitted per second             │
│                                                                      │
│  For BINARY signaling (2 levels): Baud Rate = Bit Rate             │
│                                                                      │
│  Example: 9600 baud = 9600 bits/second                              │
│                                                                      │
│  ───────────────────────────────────────────────────────────────    │
│                                                                      │
│  For MULTI-LEVEL signaling: Bit Rate > Baud Rate                   │
│                                                                      │
│  If each symbol represents N bits:                                  │
│     Bit Rate = Baud Rate × log₂(Number of levels)                  │
│                                                                      │
│  Example: 4-level signaling (2 bits per symbol)                     │
│     1000 baud × 2 bits/symbol = 2000 bps                           │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Signaling  │ Levels │ Bits/Symbol │ 1000 baud = bit rate  │     │
│  ├────────────────────────────────────────────────────────────┤     │
│  │ Binary     │ 2      │ 1           │ 1000 bps              │     │
│  │ 4-level    │ 4      │ 2           │ 2000 bps              │     │
│  │ 8-level    │ 8      │ 3           │ 3000 bps              │     │
│  │ 16-level   │ 16     │ 4           │ 4000 bps              │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  COMMON UART BAUD RATES:                                             │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Rate       │ Bit time  │ Byte time │ Use case             │     │
│  ├────────────────────────────────────────────────────────────┤     │
│  │ 9600       │ 104.2 μs  │ ~1 ms     │ Legacy, low-speed    │     │
│  │ 19200      │ 52.1 μs   │ ~0.5 ms   │ Moderate speed       │     │
│  │ 38400      │ 26.0 μs   │ ~0.26 ms  │ Faster peripherals   │     │
│  │ 115200     │ 8.68 μs   │ ~86.8 μs  │ Debug console        │     │
│  │ 921600     │ 1.09 μs   │ ~10.9 μs  │ High-speed transfer  │     │
│  │ 1000000    │ 1.0 μs    │ ~10 μs    │ Maximum common       │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Baud Rate Calculation

For a given clock frequency, baud rate is calculated:

$$\text{Baud Rate} = \frac{f_{clock}}{16 \times \text{USART\_DIV}}$$

Where USART_DIV is the baud rate register value.

To find USART_DIV:
$$\text{USART\_DIV} = \frac{f_{clock}}{16 \times \text{Baud Rate}}$$

**Example**: 16 MHz clock, 9600 baud
$$\text{USART\_DIV} = \frac{16,000,000}{16 \times 9600} = 104.17$$

---

## Data Framing

```
┌─────────────────────────────────────────────────────────────────────┐
│                    UART FRAME STRUCTURE                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Standard 8N1 Frame (8 data, No parity, 1 stop):                    │
│                                                                      │
│  Idle │Start│ D0 │ D1 │ D2 │ D3 │ D4 │ D5 │ D6 │ D7 │Stop│ Idle    │
│       │     │    │    │    │    │    │    │    │    │    │          │
│  ─────┐     ┌────┬────┬────┬────┬────┬────┬────┬────┐    ┌─────     │
│       │     │    │    │    │    │    │    │    │    │    │          │
│       └─────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘          │
│   1   │  0  │ LSB          DATA BITS          MSB │  1  │  1        │
│       │     │◀──────────── 8 bits ──────────────▶│     │            │
│                                                                      │
│  Frame = Start(1) + Data(8) + Stop(1) = 10 bits for 8 data bits    │
│  Efficiency = 8/10 = 80%                                            │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  Frame with Parity (8E1 or 8O1):                                    │
│                                                                      │
│  │Start│ D0 │ D1 │ D2 │ D3 │ D4 │ D5 │ D6 │ D7 │Par│Stop│          │
│                                                                      │
│  Frame = Start(1) + Data(8) + Parity(1) + Stop(1) = 11 bits        │
│  Efficiency = 8/11 = 72.7%                                          │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  PARITY TYPES:                                                       │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Type       │ Parity Bit Value                              │     │
│  ├────────────────────────────────────────────────────────────┤     │
│  │ Even       │ Makes total 1s count even                     │     │
│  │ Odd        │ Makes total 1s count odd                      │     │
│  │ Mark       │ Always 1                                      │     │
│  │ Space      │ Always 0                                      │     │
│  │ None       │ No parity bit                                 │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  Example: Data = 0x35 = 00110101 (4 ones)                           │
│    Even parity bit = 0 (already even)                               │
│    Odd parity bit = 1 (make it odd)                                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Bit Ordering

```
┌─────────────────────────────────────────────────────────────────────┐
│                    BIT ORDERING (ENDIANNESS)                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  LSB FIRST (Little Endian) - Most common for UART:                  │
│                                                                      │
│  Data: 0x41 = 'A' = 01000001                                        │
│                                                                      │
│  Transmission order:                                                 │
│  Time ───────────────────────────────────────────────▶              │
│        Start  D0   D1   D2   D3   D4   D5   D6   D7  Stop           │
│         [0]   [1]  [0]  [0]  [0]  [0]  [0]  [1]  [0]  [1]           │
│               LSB                              MSB                   │
│                                                                      │
│  Wire:    ─┐    ┌────────────────────────────┐    ┌──               │
│            └────┘                            └────┘                  │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  MSB FIRST (Big Endian) - Used in SPI, I2C:                         │
│                                                                      │
│  Data: 0x41 = 01000001                                              │
│                                                                      │
│  Transmission order:                                                 │
│  Time ───────────────────────────────────────────────▶              │
│         D7   D6   D5   D4   D3   D2   D1   D0                       │
│         [0]  [1]  [0]  [0]  [0]  [0]  [0]  [1]                      │
│         MSB                              LSB                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Clock Synchronization

### Asynchronous Recovery

```
┌─────────────────────────────────────────────────────────────────────┐
│                    UART BIT SAMPLING                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  UART uses oversampling (typically 16x) to find bit center:        │
│                                                                      │
│  RX Line:  ─────┐                   ┌───────────────                │
│                 └───────────────────┘                                │
│            Idle │    Start bit      │    D0 (=1)                    │
│                                                                      │
│  16x Clock: ││││││││││││││││││││││││││││││││││││││││                │
│             0         8         16        24        32               │
│                       ▲          ▲         ▲                         │
│              Detect  Sample    Sample    Sample                      │
│              falling  start     D0       D1...                       │
│              edge                                                    │
│                                                                      │
│  SAMPLING ALGORITHM:                                                 │
│  1. Wait for falling edge (start bit)                               │
│  2. Wait 8 clocks (half bit time) → sample start bit (verify 0)    │
│  3. Wait 16 clocks → sample D0                                      │
│  4. Repeat for D1-D7, parity, stop                                  │
│                                                                      │
│  This samples at bit CENTER for best noise immunity:                │
│                                                                      │
│      ┌──────────────────────┐                                        │
│      │     One bit time     │                                        │
│      │          ▼           │                                        │
│  ────┼───────── ◯ ──────────┼────                                   │
│      │        center        │                                        │
│      │     sample point     │                                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Clock Tolerance

For UART to work, both ends must have similar baud rates:

$$\text{Max clock error} = \frac{0.5 \text{ bit}}{10 \text{ bits}} = ±5\%$$

For each bit, we can tolerate ±0.5 bit timing error across the entire frame (about 10 bits).

In practice, ±2% is recommended for reliable operation.

---

## Line Coding

```
┌─────────────────────────────────────────────────────────────────────┐
│                    LINE CODING SCHEMES                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  NRZ (Non-Return to Zero) - Used in UART:                           │
│                                                                      │
│  Data:   1   0   1   1   0   0   1   0                              │
│                                                                      │
│  Signal: ───┐   ┌───────┐       ┌───┐                               │
│             └───┘       └───────┘   └───                            │
│                                                                      │
│  • High = 1, Low = 0                                                │
│  • No transition required within bit                                │
│  • Long runs of same bit can cause sync loss                        │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  NRZI (Non-Return to Zero Inverted) - Used in USB:                  │
│                                                                      │
│  Data:   1   0   1   1   0   0   1   0                              │
│                                                                      │
│  Signal: ─────┐   ┌───────────┐   ┌───┐                             │
│               └───┘           └───┘   └─                            │
│          ↑   ↑   ↑           ↑   ↑   ↑                              │
│          T       T           T   T   T                               │
│                                                                      │
│  • Transition = 0, No transition = 1                                │
│  • Better for clock recovery                                        │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  MANCHESTER - Used in Ethernet 10Base-T:                            │
│                                                                      │
│  Data:   1   0   1   1   0   0   1   0                              │
│                                                                      │
│  Signal: ─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌                           │
│           └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘                           │
│          1↓  0↑  1↓  1↓  0↑  0↑  1↓  0↑                             │
│                                                                      │
│  • 0 = Low→High transition, 1 = High→Low transition                │
│  • Self-clocking (transition every bit)                             │
│  • 2x bandwidth required                                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Error Detection

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ERROR DETECTION METHODS                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PARITY (Single bit error detection):                               │
│                                                                      │
│  Data: 10110100 (4 ones)                                            │
│  Even parity: 0 → 101101000 (still 4 ones)                         │
│                                                                      │
│  Received: 101111000 (bit flip error)                               │
│  Check: 5 ones ≠ even → ERROR DETECTED                             │
│                                                                      │
│  Limitation: Cannot detect even number of bit errors               │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  CHECKSUM (Sum of bytes):                                           │
│                                                                      │
│  Data: 0x12, 0x34, 0x56                                             │
│  Sum: 0x12 + 0x34 + 0x56 = 0x9C                                     │
│  Checksum: 0x100 - 0x9C = 0x64 (two's complement)                   │
│                                                                      │
│  Verify: 0x12 + 0x34 + 0x56 + 0x64 = 0x100 (low byte = 0)          │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  CRC (Cyclic Redundancy Check):                                     │
│                                                                      │
│  • Polynomial division                                              │
│  • Much stronger error detection                                    │
│  • Detects burst errors                                             │
│                                                                      │
│  Common CRCs:                                                        │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ CRC      │ Polynomial        │ Use Case                   │     │
│  ├────────────────────────────────────────────────────────────┤     │
│  │ CRC-8    │ x⁸+x²+x+1         │ Simple protocols           │     │
│  │ CRC-16   │ x¹⁶+x¹⁵+x²+1      │ Modbus, USB                │     │
│  │ CRC-32   │ (complex)         │ Ethernet, ZIP              │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Voltage Levels

```
┌─────────────────────────────────────────────────────────────────────┐
│                    VOLTAGE STANDARDS                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  TTL/CMOS (3.3V or 5V Logic):                                       │
│                                                                      │
│  Voltage                                                             │
│      │                                                               │
│   5V ┼────────────────────── Logic 1                                │
│      │                                                               │
│   0V ┼────────────────────── Logic 0                                │
│      │                                                               │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  RS-232 (inverted, ±12V):                                           │
│                                                                      │
│  Voltage                                                             │
│      │                                                               │
│ +12V ┼────────────────────── Logic 0 (Space)                        │
│      │                                                               │
│  +3V ┼- - - - - - - - - - -  Undefined                              │
│  -3V ┼- - - - - - - - - - -  region                                 │
│      │                                                               │
│ -12V ┼────────────────────── Logic 1 (Mark)                         │
│      │                                                               │
│                                                                      │
│  RS-232 is INVERTED: Negative = 1 (Mark), Positive = 0 (Space)     │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  RS-485 (Differential):                                             │
│                                                                      │
│      A ─────────────────┐                                           │
│                         │  Difference > +200mV = Logic 1            │
│      B ─────────────────┘  Difference < -200mV = Logic 0            │
│                                                                      │
│  • Immune to common-mode noise                                      │
│  • Long distance (1200m at 100kbps)                                │
│  • Multi-drop capable                                               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Parameter | Description | Common Values |
|-----------|-------------|---------------|
| **Baud Rate** | Symbols/second | 9600, 115200 |
| **Data Bits** | Bits per frame | 7, 8, 9 |
| **Parity** | Error detection | None, Even, Odd |
| **Stop Bits** | Frame end | 1, 1.5, 2 |
| **Bit Order** | Transmission order | LSB first (UART) |
| **Oversampling** | Clock per bit | 16x typical |
| **Clock Tolerance** | Acceptable error | ±2-5% |

---

## Quick Revision Questions

1. **What is the difference between baud rate and bit rate? When are they different?**

2. **Calculate the time to transmit 100 bytes at 9600 baud with 8N1 framing.**

3. **Explain how a UART receiver synchronizes to incoming data without a clock signal.**

4. **What is the purpose of parity? What types of errors can it NOT detect?**

5. **Compare TTL-level UART with RS-232. Why is RS-232 used for long cables?**

6. **A UART uses 16x oversampling at 115200 baud. What is the sampling clock frequency?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Unit 5: Communication Protocols](README.md) | [Unit Index](README.md) | [Chapter 5.2: UART Protocol](02-uart-protocol.md) |

---
