# Unit 6: Sequential Circuits - Counters

---

## Table of Contents
1. [Introduction to Counters](#1-introduction-to-counters)
2. [Asynchronous (Ripple) Counters](#2-asynchronous-ripple-counters)
3. [Synchronous Counters](#3-synchronous-counters)
4. [Up/Down Counters](#4-updown-counters)
5. [Mod-N Counters](#5-mod-n-counters)
6. [Ring Counter](#6-ring-counter)
7. [Johnson (Twisted Ring) Counter](#7-johnson-twisted-ring-counter)
8. [Counter Design Procedure](#8-counter-design-procedure)
9. [Counter Applications](#9-counter-applications)
10. [Summary Tables](#10-summary-tables)
11. [Quick Revision Questions](#11-quick-revision-questions)

---

## 1. Introduction to Counters

### What is a Counter?

```
┌─────────────────────────────────────────────────────────────────┐
│                    COUNTER BASICS                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  DEFINITION:                                                    │
│    A sequential circuit that cycles through a fixed sequence   │
│    of states in response to clock pulses.                      │
│                                                                 │
│  BASIC OPERATION:                                               │
│                                                                 │
│       CLK ──┤        ├── Q₀   ┌───────────────────────────┐   │
│             │Counter ├── Q₁   │  0→1→2→3→4→5→6→7→0→...   │   │
│             │        ├── Q₂   │     (Counting sequence)   │   │
│             │        ├── Q₃   └───────────────────────────┘   │
│                                                                 │
│  MODULUS (MOD):                                                 │
│    - Number of unique states in count sequence                 │
│    - n-bit counter: MOD-2ⁿ (maximum)                           │
│    - Example: 3-bit counter = MOD-8 (counts 0-7)               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Classification of Counters

```
┌─────────────────────────────────────────────────────────────────┐
│                 COUNTER CLASSIFICATION                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  BY CLOCKING:                                                   │
│    ┌─────────────────────────────────────────────┐             │
│    │ ASYNCHRONOUS (Ripple)   │ SYNCHRONOUS       │             │
│    ├─────────────────────────┼───────────────────┤             │
│    │ FFs clocked sequentially│ All FFs clocked   │             │
│    │ by previous FF output   │ simultaneously    │             │
│    │ Simple, but slow        │ Fast, but complex │             │
│    └─────────────────────────┴───────────────────┘             │
│                                                                 │
│  BY DIRECTION:                                                  │
│    • UP Counter: 0→1→2→3→...→max→0                             │
│    • DOWN Counter: max→...→3→2→1→0→max                         │
│    • UP/DOWN Counter: Both directions (controlled)             │
│                                                                 │
│  BY MODULUS:                                                    │
│    • Binary: MOD-2ⁿ (2, 4, 8, 16...)                           │
│    • Decade: MOD-10 (BCD counter)                              │
│    • MOD-N: Any count sequence                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Asynchronous (Ripple) Counters

### Concept

```
┌─────────────────────────────────────────────────────────────────┐
│             ASYNCHRONOUS COUNTER CONCEPT                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  "Ripple" because clock ripples through the flip-flops         │
│                                                                 │
│  Key Characteristics:                                           │
│    • Only first FF receives external clock                     │
│    • Each subsequent FF clocked by previous FF's output        │
│    • Propagation delay accumulates                             │
│    • Simple design, fewer gates                                │
│    • Limited maximum frequency                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3-Bit Ripple UP Counter

```
3-BIT ASYNCHRONOUS UP COUNTER (using negative-edge T flip-flops):

                                T=1 means toggle on each clock edge

    CLK ────────────┤>o  T  Q ├────────┤>o  T  Q ├────────┤>o  T  Q ├
                    │    FF₀  │        │    FF₁  │        │    FF₂  │
                    │         ├────────┤         ├────────┤         │
              1 ────┤ T   Q'  │  1 ────┤ T   Q'  │  1 ────┤ T   Q'  │
                    └─────────┘        └─────────┘        └─────────┘
                         │                  │                  │
                         Q₀ (LSB)          Q₁                 Q₂ (MSB)

COUNTING SEQUENCE:
┌───────┬─────┬─────┬─────┬─────────┐
│ Clock │ Q₂  │ Q₁  │ Q₀  │ Decimal │
├───────┼─────┼─────┼─────┼─────────┤
│ Init  │  0  │  0  │  0  │    0    │
│   1   │  0  │  0  │  1  │    1    │
│   2   │  0  │  1  │  0  │    2    │
│   3   │  0  │  1  │  1  │    3    │
│   4   │  1  │  0  │  0  │    4    │
│   5   │  1  │  0  │  1  │    5    │
│   6   │  1  │  1  │  0  │    6    │
│   7   │  1  │  1  │  1  │    7    │
│   8   │  0  │  0  │  0  │    0    │ (recycles)
└───────┴─────┴─────┴─────┴─────────┘

TIMING DIAGRAM:
    CLK ────┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐
            └──┘  └──┘  └──┘  └──┘  └──┘  └──┘  └──┘  └──┘  └──

    Q₀  ────────┐     ┌─────┐     ┌─────┐     ┌─────┐     ┌────
                └─────┘     └─────┘     └─────┘     └─────┘

    Q₁  ────────────────────┐           ┌─────────────────┐
                            └───────────┘                 └────

    Q₂  ────────────────────────────────────────┐
                                                └──────────────

        0    1    2    3    4    5    6    7    0    1    2    ...
```

### 3-Bit Ripple DOWN Counter

```
3-BIT ASYNCHRONOUS DOWN COUNTER:

Clock from Q (not Q') of each flip-flop OR use positive-edge FF

    CLK ────────────┤>   T  Q ├────────┤>   T  Q ├────────┤>   T  Q ├
                    │   FF₀   │        │   FF₁   │        │   FF₂   │
                    │     Q'  ├────────┤     Q'  ├────────┤     Q'  │
              1 ────┤ T       │  1 ────┤ T       │  1 ────┤ T       │
                    └─────────┘        └─────────┘        └─────────┘
                         │                  │                  │
                         Q₀ (LSB)          Q₁                 Q₂ (MSB)

Alternative: Use Q' outputs for count, keeping Q as clock

COUNTING SEQUENCE:
┌───────┬─────┬─────┬─────┬─────────┐
│ Clock │ Q₂  │ Q₁  │ Q₀  │ Decimal │
├───────┼─────┼─────┼─────┼─────────┤
│ Init  │  1  │  1  │  1  │    7    │
│   1   │  1  │  1  │  0  │    6    │
│   2   │  1  │  0  │  1  │    5    │
│   3   │  1  │  0  │  0  │    4    │
│   4   │  0  │  1  │  1  │    3    │
│   5   │  0  │  1  │  0  │    2    │
│   6   │  0  │  0  │  1  │    1    │
│   7   │  0  │  0  │  0  │    0    │
│   8   │  1  │  1  │  1  │    7    │ (recycles)
└───────┴─────┴─────┴─────┴─────────┘
```

### Propagation Delay Problem

```
RIPPLE COUNTER TIMING PROBLEM:

For n-bit ripple counter:
    Total delay = n × tpd (flip-flop propagation delay)

Example: 4-bit counter with tpd = 20ns each

    CLK ──────────────┐
                      └──────────────────────────────────────
                      │
    Q₀  ──────────────┼───┐
                      │   └──────────────────────────────────
                      │   20ns
    Q₁  ──────────────┼───────┐
                      │       └──────────────────────────────
                      │       40ns
    Q₂  ──────────────┼───────────┐
                      │           └──────────────────────────
                      │           60ns
    Q₃  ──────────────┼───────────────┐
                      │               └──────────────────────
                      │               80ns
                      
Maximum frequency: fmax = 1 / (n × tpd)
For 4-bit with 20ns: fmax = 1/(4×20ns) = 12.5 MHz

GLITCH PROBLEM:
During transition 0111→1000, count briefly shows:
0111 → 0110 → 0100 → 0000 → 1000 (glitches!)
```

---

## 3. Synchronous Counters

### Concept

```
┌─────────────────────────────────────────────────────────────────┐
│             SYNCHRONOUS COUNTER CONCEPT                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  All flip-flops triggered by the SAME clock edge               │
│                                                                 │
│  Advantages:                                                    │
│    • No accumulated propagation delay                          │
│    • Higher maximum operating frequency                        │
│    • No glitches between states                                │
│                                                                 │
│  Disadvantages:                                                 │
│    • More complex combinational logic                          │
│    • More gates required                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3-Bit Synchronous UP Counter (using JK FF)

```
3-BIT SYNCHRONOUS UP COUNTER:

Design using JK flip-flops:

State Table:
┌───────────────┬───────────────┬─────────────────────────────────┐
│ Present State │  Next State   │      FF Inputs (J,K)            │
├───┬───┬───┬───┼───┬───┬───┬───┼───────┬───────┬───────┬─────────┤
│ # │Q₂ │Q₁ │Q₀ │Q₂⁺│Q₁⁺│Q₀⁺│ # │J₂ K₂  │J₁ K₁  │J₀ K₀  │         │
├───┼───┼───┼───┼───┼───┼───┼───┼───────┼───────┼───────┼─────────┤
│ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 1 │ 1 │ 0  X  │ 0  X  │ 1  X  │         │
│ 1 │ 0 │ 0 │ 1 │ 0 │ 1 │ 0 │ 2 │ 0  X  │ 1  X  │ X  1  │         │
│ 2 │ 0 │ 1 │ 0 │ 0 │ 1 │ 1 │ 3 │ 0  X  │ X  0  │ 1  X  │         │
│ 3 │ 0 │ 1 │ 1 │ 1 │ 0 │ 0 │ 4 │ 1  X  │ X  1  │ X  1  │         │
│ 4 │ 1 │ 0 │ 0 │ 1 │ 0 │ 1 │ 5 │ X  0  │ 0  X  │ 1  X  │         │
│ 5 │ 1 │ 0 │ 1 │ 1 │ 1 │ 0 │ 6 │ X  0  │ 1  X  │ X  1  │         │
│ 6 │ 1 │ 1 │ 0 │ 1 │ 1 │ 1 │ 7 │ X  0  │ X  0  │ 1  X  │         │
│ 7 │ 1 │ 1 │ 1 │ 0 │ 0 │ 0 │ 0 │ X  1  │ X  1  │ X  1  │         │
└───┴───┴───┴───┴───┴───┴───┴───┴───────┴───────┴───────┴─────────┘

K-Maps for J₀, K₀:
       Q₁Q₀                    Q₁Q₀
      00  01  11  10          00  01  11  10
    ┌───┬───┬───┬───┐       ┌───┬───┬───┬───┐
 Q₂=0│ 1 │ X │ X │ 1 │    Q₂=0│ X │ 1 │ 1 │ X │
    ├───┼───┼───┼───┤       ├───┼───┼───┼───┤
 Q₂=1│ 1 │ X │ X │ 1 │    Q₂=1│ X │ 1 │ 1 │ X │
    └───┴───┴───┴───┘       └───┴───┴───┴───┘
         J₀ = 1                  K₀ = 1

K-Maps for J₁, K₁:
       Q₁Q₀                    Q₁Q₀
      00  01  11  10          00  01  11  10
    ┌───┬───┬───┬───┐       ┌───┬───┬───┬───┐
 Q₂=0│ 0 │ 1 │ X │ X │    Q₂=0│ X │ X │ 1 │ 0 │
    ├───┼───┼───┼───┤       ├───┼───┼───┼───┤
 Q₂=1│ 0 │ 1 │ X │ X │    Q₂=1│ X │ X │ 1 │ 0 │
    └───┴───┴───┴───┘       └───┴───┴───┴───┘
         J₁ = Q₀                 K₁ = Q₀

K-Maps for J₂, K₂:
       Q₁Q₀                    Q₁Q₀
      00  01  11  10          00  01  11  10
    ┌───┬───┬───┬───┐       ┌───┬───┬───┬───┐
 Q₂=0│ 0 │ 0 │ 1 │ 0 │    Q₂=0│ X │ X │ X │ X │
    ├───┼───┼───┼───┤       ├───┼───┼───┼───┤
 Q₂=1│ X │ X │ X │ X │    Q₂=1│ 0 │ 0 │ 1 │ 0 │
    └───┴───┴───┴───┘       └───┴───┴───┴───┘
         J₂ = Q₁Q₀               K₂ = Q₁Q₀

FINAL EQUATIONS:
  J₀ = K₀ = 1  (always toggle)
  J₁ = K₁ = Q₀
  J₂ = K₂ = Q₁Q₀

CIRCUIT:
                                                   
     1 ─────┬─────────────────────────────────────┐
            │                                     │
            │           Q₀ ─────────────────┬─────┤
            │               │               │     │
            │               │  Q₁Q₀ ────┬───┼─────┤
            │               │      │    │   │     │
            │               │      │    │   │     │
            ▼               ▼      │    ▼   ▼     ▼
         ┌─────┐         ┌─────┐   │ ┌─────┐   ┌─────┐
         │     │         │     │   │ │     │   │     │
     1 ──┤J  Q ├──Q₀   ──┤J  Q ├──Q₁─┤J  Q ├──Q₂    │
         │     │         │     │     │     │        │
    CLK ─┤>    │    CLK ─┤>    │CLK ─┤>    │        │
         │     │         │     │     │     │        │
     1 ──┤K  Q'├──     ──┤K  Q'├──  ─┤K  Q'├──      │
         │     │         │     │     │     │        │
         └─────┘         └─────┘     └─────┘        │
            │               │           │          │
            └───────────────┴───────────┴──────────┘
                           CLK (common to all)
```

### 4-Bit Synchronous UP Counter with Enable and Clear

```
4-BIT SYNCHRONOUS COUNTER (74163 type):

Features:
  - Synchronous clear (CLR')
  - Count enable (ENP, ENT)
  - Parallel load (LOAD')
  - Ripple carry output (RCO)

              ┌───────────────────────┐
              │                       │
       D₀ ────┤ D₀              Q₀    ├──── Q₀
       D₁ ────┤ D₁              Q₁    ├──── Q₁
       D₂ ────┤ D₂    74163    Q₂    ├──── Q₂
       D₃ ────┤ D₃              Q₃    ├──── Q₃
              │                       │
      CLK ────┤ CLK             RCO   ├──── Carry
              │                       │
     LOAD'────┤ LOAD'                 │
      CLR'────┤ CLR'                  │
      ENP ────┤ ENP                   │
      ENT ────┤ ENT                   │
              └───────────────────────┘

FUNCTION TABLE:
┌──────┬───────┬─────┬─────┬─────────────────────┐
│ CLR' │ LOAD' │ ENP │ ENT │      Function       │
├──────┼───────┼─────┼─────┼─────────────────────┤
│  0   │   X   │  X  │  X  │  Clear (Q=0000)     │
│  1   │   0   │  X  │  X  │  Load D₃D₂D₁D₀      │
│  1   │   1   │  0  │  X  │  Hold               │
│  1   │   1   │  X  │  0  │  Hold               │
│  1   │   1   │  1  │  1  │  Count              │
└──────┴───────┴─────┴─────┴─────────────────────┘

RCO = ENT · Q₃ · Q₂ · Q₁ · Q₀ (high at count 15)
```

---

## 4. Up/Down Counters

### Up/Down Control Logic

```
UP/DOWN COUNTER CONCEPT:

Control signal U/D':
  - U/D' = 1: Count UP
  - U/D' = 0: Count DOWN

Toggle condition for each bit:
  UP:   Toggle Qₙ when all lower bits are 1
  DOWN: Toggle Qₙ when all lower bits are 0

3-BIT UP/DOWN COUNTER:

Flip-flop equations:
  T₀ = 1 (always toggle)
  T₁ = (U/D')Q₀ + (U/D)Q₀' = Q₀ ⊕ (U/D)
  T₂ = (U/D')Q₁Q₀ + (U/D)Q₁'Q₀'

CIRCUIT:
                        ┌───────────────────────────────────┐
       U/D' ────────────┤                                   │
                        │                                   │
        1 ──────────────┤ T₀                                │
                        │                                   │
       (U/D')Q₀ + ──────┤ T₁                                │
       (U/D)Q₀'         │        3-BIT UP/DOWN              │
                        │          COUNTER                  │
       (U/D')Q₁Q₀ + ────┤ T₂                                │
       (U/D)Q₁'Q₀'      │                                   │
                        │                                   │
       CLK ─────────────┤>                                  │
                        └───────────────────────────────────┘

UP SEQUENCE (U/D' = 1):   0→1→2→3→4→5→6→7→0
DOWN SEQUENCE (U/D' = 0): 0→7→6→5→4→3→2→1→0
```

### 4-Bit Synchronous Up/Down Counter

```
4-BIT SYNCHRONOUS UP/DOWN COUNTER (74193 type):

              ┌───────────────────────┐
              │                       │
       D₀ ────┤ D₀              Q₀    ├──── Q₀
       D₁ ────┤ D₁              Q₁    ├──── Q₁
       D₂ ────┤ D₂    74193    Q₂    ├──── Q₂
       D₃ ────┤ D₃              Q₃    ├──── Q₃
              │                       │
       UP ────┤ UP             TCU    ├──── Carry (UP)
     DOWN ────┤ DOWN           TCD    ├──── Borrow (DOWN)
              │                       │
    LOAD' ────┤ LOAD'                 │
      CLR ────┤ CLR                   │
              └───────────────────────┘

OPERATION:
  - Count UP: Pulse on UP input (DOWN held high)
  - Count DOWN: Pulse on DOWN input (UP held high)
  - Load: LOAD' = 0 loads parallel data
  - Clear: CLR = 1 clears counter

TCU (Terminal Count Up) = 1 when count = 1111 and UP pulse
TCD (Terminal Count Down) = 1 when count = 0000 and DOWN pulse
```

---

## 5. Mod-N Counters

### Concept

```
┌─────────────────────────────────────────────────────────────────┐
│                   MOD-N COUNTERS                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  A MOD-N counter counts through N states (0 to N-1)            │
│                                                                 │
│  Methods to create MOD-N from MOD-2ⁿ counter:                  │
│    1. Synchronous clear at count N                             │
│    2. Asynchronous clear at count N                            │
│    3. Preset to specific starting value                        │
│    4. Custom state machine design                              │
│                                                                 │
│  Formula: For MOD-N counter, need ⌈log₂N⌉ flip-flops           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### MOD-10 (Decade/BCD) Counter

```
MOD-10 COUNTER: Counts 0-9 then resets

Method: Detect count 10 (1010) and reset

Using asynchronous clear:

    CLK ──────┤>        ├── Q₀
              │         │
              │  4-BIT  ├── Q₁
              │ COUNTER │
              │         ├── Q₂ ───────┐
              │         │             │
              │   CLR   ├── Q₃ ───┬───┤
              └────┬────┘         │   │
                   │              │   │
                   └──────────────┤AND├─┘
                                  │   │
                                  └───┘
              Detect Q₃Q₂Q₁Q₀ = 1010

TIMING ISSUE: Brief glitch at 1010 before clear
    
    Count: 0→1→2→3→4→5→6→7→8→9→(1010)→0
                                  ↑
                              very brief

STATE DIAGRAM:
         ┌───────────────────────────────────┐
         │                                   │
         ▼                                   │
    ┌───────┐   ┌───────┐       ┌───────┐   │
    │   0   │──▶│   1   │──▶...▶│   9   │───┘
    │ 0000  │   │ 0001  │       │ 1001  │
    └───────┘   └───────┘       └───────┘

Synchronous MOD-10 (BCD): Use enable logic
  Enable counting only for valid BCD states
```

### MOD-6 Counter

```
MOD-6 COUNTER: Counts 0-5 then resets

Detect 6 (0110) and clear:

    ┌───────────────────────────────────────┐
    │         3-BIT COUNTER                 │
    │                                       │
    │  CLK ──┤>                    Q₀ ├───────
    │        │                        │
    │        │                    Q₁ ├───┬───
    │        │                        │   │
    │        │                    Q₂ ├─┬─┼───
    │        │                        │ │ │
    │        └────────┬───────────────┘ │ │
    │                 │                 │ │
    │               CLR                 │ │
    │                 ▲                 │ │
    └─────────────────│─────────────────┘ │
                      │                   │
                      └───────────────────┤AND├
                                          │   │
                      Detect Q₂'Q₁Q₀' = 110

Actually detect 110 (6):

TRUTH TABLE:
┌─────┬─────┬─────┬───────┬─────────┐
│ Q₂  │ Q₁  │ Q₀  │ Count │ Action  │
├─────┼─────┼─────┼───────┼─────────┤
│  0  │  0  │  0  │   0   │ Count   │
│  0  │  0  │  1  │   1   │ Count   │
│  0  │  1  │  0  │   2   │ Count   │
│  0  │  1  │  1  │   3   │ Count   │
│  1  │  0  │  0  │   4   │ Count   │
│  1  │  0  │  1  │   5   │ Count   │
│  1  │  1  │  0  │   6   │ CLEAR   │
└─────┴─────┴─────┴───────┴─────────┘

CLR = Q₂ · Q₁ (when Q₂=1 and Q₁=1, clear)
```

### MOD-12 Counter

```
MOD-12 COUNTER (for 12-hour clock):

Using 4-bit counter, detect 12 (1100) and clear:

CLR = Q₃ · Q₂ (detect 12)

State sequence: 0→1→2→3→4→5→6→7→8→9→10→11→0

CIRCUIT:
    ┌───────────────────────────────────────┐
    │         4-BIT COUNTER                 │
    │                                       │
    │  CLK ──┤>                    Q₀ ├─────│
    │        │                        │     │
    │        │                    Q₁ ├─────│
    │        │                        │     │
    │        │                    Q₂ ├───┬─│
    │        │                        │   │ │
    │        │                    Q₃ ├─┬─┼─│
    │        │                        │ │ │ │
    │        └────────┬───────────────┘ │ │ │
    │                 │                 │ │ │
    │               CLR                 │ │ │
    │                 ▲                 │ │ │
    └─────────────────│─────────────────┘ │ │
                      │                   │ │
                      └───────────────────┤AND├
                                          └───┘
                      CLR = Q₃ · Q₂
```

### Cascading Counters for Higher Modulus

```
MOD-60 COUNTER (for seconds/minutes):

MOD-60 = MOD-6 × MOD-10

    ┌──────────────┐          ┌──────────────┐
    │              │          │              │
    │   MOD-10     │   Carry  │    MOD-6     │
    │   Counter    ├──────────┤   Counter    │
    │   (ones)     │    Q₃    │   (tens)     │
    │              │          │              │
    │     Q₀-Q₃    │          │    Q₀-Q₂     │
    └──────────────┘          └──────────────┘
          │                         │
          ▼                         ▼
       0-9 (BCD)                  0-5 (BCD)
       
    Combined display: 00-59 (seconds or minutes)

Cascade using terminal count/carry of first counter
to clock/enable second counter.
```

---

## 6. Ring Counter

### Concept and Operation

```
┌─────────────────────────────────────────────────────────────────┐
│                    RING COUNTER                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  A shift register with output fed back to input                │
│  Single '1' circulates through the register                    │
│                                                                 │
│  Characteristics:                                               │
│    • n flip-flops = MOD-n (n states)                           │
│    • Self-decoding (each state = one hot)                      │
│    • Simple design, no decoding logic needed                   │
│    • Inefficient use of flip-flops (n FFs for n states)        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4-Bit Ring Counter

```
4-BIT RING COUNTER:

              ┌──────────────────────────────────────────┐
              │                                          │
              │     ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐
              │     │     │    │     │    │     │    │     │
              └─────┤D  Q ├────┤D  Q ├────┤D  Q ├────┤D  Q ├───
                    │     │    │     │    │     │    │     │
              CLK ──┤>    │────┤>    │────┤>    │────┤>    │
                    │     │    │     │    │     │    │     │
                    └─────┘    └─────┘    └─────┘    └─────┘
                      Q₀         Q₁         Q₂         Q₃

Initial state: 1000 (preset Q₀=1, others=0)

STATE SEQUENCE:
┌───────┬─────┬─────┬─────┬─────┬───────┐
│ Clock │ Q₀  │ Q₁  │ Q₂  │ Q₃  │ State │
├───────┼─────┼─────┼─────┼─────┼───────┤
│ Init  │  1  │  0  │  0  │  0  │  S₀   │
│   1   │  0  │  1  │  0  │  0  │  S₁   │
│   2   │  0  │  0  │  1  │  0  │  S₂   │
│   3   │  0  │  0  │  0  │  1  │  S₃   │
│   4   │  1  │  0  │  0  │  0  │  S₀   │ (repeats)
└───────┴─────┴─────┴─────┴─────┴───────┘

TIMING DIAGRAM:
    CLK ──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──
          └──┘  └──┘  └──┘  └──┘  └──┘

    Q₀  ──────┐           ┌──────────────
              └───────────┘

    Q₁  ──────────┐           ┌──────────
                  └───────────┘

    Q₂  ──────────────┐           ┌──────
                      └───────────┘

    Q₃  ──────────────────┐           ┌──
                          └───────────┘

STATE DIAGRAM:
    ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐
    │1000 │───▶│0100 │───▶│0010 │───▶│0001 │──┐
    │ S₀  │    │ S₁  │    │ S₂  │    │ S₃  │  │
    └─────┘    └─────┘    └─────┘    └─────┘  │
        ▲                                      │
        └──────────────────────────────────────┘
```

### Self-Correcting Ring Counter

```
PROBLEM: If ring counter enters invalid state (e.g., 0000 or 0011),
it stays in invalid sequence forever.

SELF-CORRECTING RING COUNTER:

Add logic to detect invalid states and correct

For 4-bit: D₀ = Q₁' · Q₂' · Q₃' (only 1 when Q=0001 or Q=0000)

              ┌──────────────────────────────────────────────────┐
              │     ┌─────────────┐                              │
              │     │             │                              │
              │   ┌─┤ Q₁'·Q₂'·Q₃' │                              │
              │   │ │   (NOR)     │                              │
              │   │ └─────────────┘                              │
              │   │                                              │
              │   ▼                                              │
              │  ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐       │
              │  │     │    │     │    │     │    │     │       │
              └──┤D  Q ├────┤D  Q ├────┤D  Q ├────┤D  Q ├───    │
                 │     │    │     │    │     │    │     │       │
           CLK ──┤>    │────┤>    │────┤>    │────┤>    │       │
                 │     │    │     │    │     │    │     │       │
                 └──┬──┘    └──┬──┘    └──┬──┘    └──┬──┘       │
                    │          │          │          │          │
                    Q₀         Q₁         Q₂         Q₃         │
                               │          │          │          │
                               └──────────┴──────────┴──────────┘
                                     (to NOR gate)

D₀ = Q₃ + (Q₁' · Q₂' · Q₃')

This ensures:
  - Normal: Q₃ feeds back
  - If stuck in 0000: D₀ becomes 1, counter self-corrects to 1000
```

---

## 7. Johnson (Twisted Ring) Counter

### Concept and Operation

```
┌─────────────────────────────────────────────────────────────────┐
│               JOHNSON (TWISTED RING) COUNTER                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Like ring counter, but Q' (inverted) fed back to input        │
│                                                                 │
│  Characteristics:                                               │
│    • n flip-flops = MOD-2n (2n states)                         │
│    • Better utilization than ring counter                      │
│    • Simple decoding (each state decoded by 2 signals)         │
│    • Glitch-free decoded outputs                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4-Bit Johnson Counter

```
4-BIT JOHNSON COUNTER:

              ┌──────────────────────────────────────────────────┐
              │                                                  │
              │     ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐    │
              │     │     │    │     │    │     │    │     │    │
              └─|>o─┤D  Q ├────┤D  Q ├────┤D  Q ├────┤D  Q ├────┘
                    │     │    │     │    │     │    │     │
              CLK ──┤>    │────┤>    │────┤>    │────┤>    │
                    │     │    │     │    │     │    │     │
                    └─────┘    └─────┘    └─────┘    └─────┘
                      Q₀         Q₁         Q₂         Q₃
                      
              Feedback: Q₃' → D₀ (inverted feedback)

STATE SEQUENCE:
┌───────┬─────┬─────┬─────┬─────┬───────┐
│ Clock │ Q₀  │ Q₁  │ Q₂  │ Q₃  │ State │
├───────┼─────┼─────┼─────┼─────┼───────┤
│ Init  │  0  │  0  │  0  │  0  │  S₀   │
│   1   │  1  │  0  │  0  │  0  │  S₁   │
│   2   │  1  │  1  │  0  │  0  │  S₂   │
│   3   │  1  │  1  │  1  │  0  │  S₃   │
│   4   │  1  │  1  │  1  │  1  │  S₄   │
│   5   │  0  │  1  │  1  │  1  │  S₅   │
│   6   │  0  │  0  │  1  │  1  │  S₆   │
│   7   │  0  │  0  │  0  │  1  │  S₇   │
│   8   │  0  │  0  │  0  │  0  │  S₀   │ (repeats)
└───────┴─────┴─────┴─────┴─────┴───────┘

8 states from 4 flip-flops (MOD-8)

STATE DIAGRAM:
    ┌───────────────────────────────────────────────────┐
    │                                                   │
    ▼                                                   │
  ┌────┐   ┌────┐   ┌────┐   ┌────┐                    │
  │0000│──▶│1000│──▶│1100│──▶│1110│──┐                 │
  └────┘   └────┘   └────┘   └────┘  │                 │
    ▲                                ▼                 │
    │      ┌────┐   ┌────┐   ┌────┐  │                 │
    └──────│0001│◀──│0011│◀──│0111│◀─┤                 │
           └────┘   └────┘   └────┘  │                 │
              │                      │                 │
              └──────────────────────┼──▶────┐        │
                                     │       │        │
                                     └───────┴────────┘
                                           1111
```

### Decoding Johnson Counter

```
DECODING (only 2-input gates needed):

Each state differs from adjacent by only ONE bit.
Decode using adjacent bits:

┌───────┬─────────┬────────────────────┐
│ State │  Q₃Q₂Q₁Q₀│ Decode Logic       │
├───────┼─────────┼────────────────────┤
│  S₀   │  0000   │ Q₀' · Q₃'          │
│  S₁   │  1000   │ Q₀  · Q₁'          │
│  S₂   │  1100   │ Q₁  · Q₂'          │
│  S₃   │  1110   │ Q₂  · Q₃'          │
│  S₄   │  1111   │ Q₃  · Q₀           │
│  S₅   │  0111   │ Q₀' · Q₁           │
│  S₆   │  0011   │ Q₁' · Q₂           │
│  S₇   │  0001   │ Q₂' · Q₃           │
└───────┴─────────┴────────────────────┘

DECODED OUTPUT WAVEFORMS (glitch-free):

    S₀  ──────┐                                   ┌────
              └───────────────────────────────────┘

    S₁  ──────────┐                           ┌────────
                  └───────────────────────────┘

    S₂  ──────────────┐                   ┌────────────
                      └───────────────────┘

    S₃  ──────────────────┐           ┌────────────────
                          └───────────┘
                          
    ... (continues for S₄-S₇)

No glitches because only one bit changes between states.
```

### Ring vs Johnson Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│            RING vs JOHNSON COUNTER COMPARISON                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Feature           │ Ring Counter    │ Johnson Counter         │
│  ─────────────────┼─────────────────┼────────────────────────  │
│  States (n FFs)    │      n          │       2n                │
│  4 FF example      │      4          │        8                │
│  Feedback          │    Q → D        │      Q' → D             │
│  Decoding          │  None needed    │  2-input AND            │
│  Self-start        │     No          │       No                │
│  Invalid states    │    2ⁿ-n         │     2ⁿ-2n               │
│  Glitch-free       │     Yes         │       Yes               │
│  FF utilization    │    Poor         │      Better             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. Counter Design Procedure

### Synchronous Counter Design Steps

```
┌─────────────────────────────────────────────────────────────────┐
│           SYNCHRONOUS COUNTER DESIGN PROCEDURE                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Step 1: SPECIFICATION                                          │
│    - Define count sequence and modulus                         │
│    - Determine number of flip-flops: n = ⌈log₂(MOD)⌉           │
│                                                                 │
│  Step 2: STATE DIAGRAM                                          │
│    - Draw states and transitions                               │
│    - Assign state codes (binary, Gray, etc.)                   │
│                                                                 │
│  Step 3: STATE TABLE                                            │
│    - List present states and next states                       │
│    - Determine flip-flop inputs using excitation tables        │
│                                                                 │
│  Step 4: K-MAP SIMPLIFICATION                                   │
│    - Create K-maps for each flip-flop input                    │
│    - Simplify to get minimal Boolean expressions               │
│                                                                 │
│  Step 5: CIRCUIT IMPLEMENTATION                                 │
│    - Draw logic circuit                                        │
│    - Add clear/preset if needed                                │
│                                                                 │
│  Step 6: VERIFICATION                                           │
│    - Verify state sequence                                     │
│    - Check for lockup states                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Design Example: MOD-5 Counter

```
DESIGN MOD-5 UP COUNTER (0→1→2→3→4→0)

Step 1: Specification
  - MOD-5: counts 0, 1, 2, 3, 4
  - FFs needed: ⌈log₂5⌉ = 3 flip-flops (Q₂Q₁Q₀)
  - Using JK flip-flops

Step 2: State Diagram
                    ┌───────────────────────────┐
                    │                           │
    ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐    │    ┌─────┐
    │ 000 │───▶│ 001 │───▶│ 010 │───▶│ 011 │───▶└───▶│ 100 │──┐
    │  0  │    │  1  │    │  2  │    │  3  │         │  4  │  │
    └─────┘    └─────┘    └─────┘    └─────┘         └─────┘  │
        ▲                                                      │
        └──────────────────────────────────────────────────────┘

Step 3: State Table with JK Excitation
┌─────────────────┬─────────────────┬───────────────────────────────┐
│  Present State  │   Next State    │         JK Inputs             │
├────┬────┬────┬──┼────┬────┬────┬──┼─────────┬─────────┬───────────┤
│ Q₂ │ Q₁ │ Q₀ │# │Q₂⁺ │Q₁⁺ │Q₀⁺ │# │  J₂K₂   │  J₁K₁   │   J₀K₀    │
├────┼────┼────┼──┼────┼────┼────┼──┼─────────┼─────────┼───────────┤
│ 0  │ 0  │ 0  │0 │ 0  │ 0  │ 1  │1 │  0  X   │  0  X   │   1  X    │
│ 0  │ 0  │ 1  │1 │ 0  │ 1  │ 0  │2 │  0  X   │  1  X   │   X  1    │
│ 0  │ 1  │ 0  │2 │ 0  │ 1  │ 1  │3 │  0  X   │  X  0   │   1  X    │
│ 0  │ 1  │ 1  │3 │ 1  │ 0  │ 0  │4 │  1  X   │  X  1   │   X  1    │
│ 1  │ 0  │ 0  │4 │ 0  │ 0  │ 0  │0 │  X  1   │  0  X   │   0  X    │
│ 1  │ 0  │ 1  │- │ X  │ X  │ X  │- │  X  X   │  X  X   │   X  X    │ don't care
│ 1  │ 1  │ 0  │- │ X  │ X  │ X  │- │  X  X   │  X  X   │   X  X    │ don't care
│ 1  │ 1  │ 1  │- │ X  │ X  │ X  │- │  X  X   │  X  X   │   X  X    │ don't care
└────┴────┴────┴──┴────┴────┴────┴──┴─────────┴─────────┴───────────┘

Step 4: K-Maps
J₀:         K₀:
  Q₁Q₀        Q₁Q₀
 00 01 11 10   00 01 11 10
┌──┬──┬──┬──┐ ┌──┬──┬──┬──┐
│1 │X │X │1 │ │X │1 │1 │X │
Q₂├──┼──┼──┼──┤ ├──┼──┼──┼──┤
│0 │X │X │X │ │X │X │X │X │
└──┴──┴──┴──┘ └──┴──┴──┴──┘
J₀ = Q₂'       K₀ = 1

J₁:         K₁:
  Q₁Q₀        Q₁Q₀
 00 01 11 10   00 01 11 10
┌──┬──┬──┬──┐ ┌──┬──┬──┬──┐
│0 │1 │X │X │ │X │X │1 │0 │
Q₂├──┼──┼──┼──┤ ├──┼──┼──┼──┤
│0 │X │X │X │ │X │X │X │X │
└──┴──┴──┴──┘ └──┴──┴──┴──┘
J₁ = Q₂'Q₀     K₁ = Q₀

J₂:         K₂:
  Q₁Q₀        Q₁Q₀
 00 01 11 10   00 01 11 10
┌──┬──┬──┬──┐ ┌──┬──┬──┬──┐
│0 │0 │1 │0 │ │X │X │X │X │
Q₂├──┼──┼──┼──┤ ├──┼──┼──┼──┤
│X │X │X │X │ │1 │X │X │X │
└──┴──┴──┴──┘ └──┴──┴──┴──┘
J₂ = Q₁Q₀      K₂ = 1

Step 5: Final Equations
  J₀ = Q₂'     K₀ = 1
  J₁ = Q₂'Q₀   K₁ = Q₀
  J₂ = Q₁Q₀    K₂ = 1

Step 6: Circuit
           Q₂'──────────────────────────────────────┐
                │                                   │
           Q₂'Q₀ ──────────────────────┐           │
                   │                   │           │
           Q₁Q₀ ────────────┐         │           │
                     │      │         │           │
                     │      │         │           │
              ┌──────┴──┐   │  ┌──────┴──┐ ┌──────┴──┐
              │         │   │  │         │ │         │
           ───┤J₂    Q₂ ├───│──┤J₁    Q₁ ├─┤J₀    Q₀ ├───
              │         │   │  │         │ │         │
        CLK ──┤>        │   │  │>        │ │>        │
              │         │   │  │         │ │         │
           1 ─┤K₂    Q₂'├   └──┤K₁    Q₁'├ │K₀    Q₀'├
              │         │   Q₀─┤         │1┤         │
              └─────────┘      └─────────┘ └─────────┘
```

---

## 9. Counter Applications

### 9.1 Digital Clock

```
DIGITAL CLOCK USING COUNTERS:

60-SECOND/MINUTE COUNTER:

    1 Hz ────┤ MOD-10 ├───┤ MOD-6  ├───┤ MOD-10 ├───┤ MOD-6  ├──
    Clock    │ Counter│   │Counter │   │ Counter│   │Counter │
             │(0-9)   │   │(0-5)   │   │(0-9)   │   │(0-5)   │
             └────────┘   └────────┘   └────────┘   └────────┘
                  │            │            │            │
                  ▼            ▼            ▼            ▼
             Seconds      Seconds      Minutes      Minutes
              Ones         Tens         Ones         Tens

12-HOUR COUNTER:

    From ────┤ MOD-10 ├───┤ Special  ├
    Minutes  │ Counter│   │ Counter  │
             │(0-9)   │   │(01-12)   │
             └────────┘   └──────────┘
                  │            │
                  ▼            ▼
              Hours        Hours
              Ones         Tens
```

### 9.2 Frequency Counter

```
FREQUENCY COUNTER BLOCK DIAGRAM:

                    ┌───────────────────────────────────┐
                    │                                   │
    Unknown ────────┤                                   │
    Frequency       │       COUNTER                     ├──── To Display
                    │                                   │
                    │    (counts input pulses           │
    Gate ───────────┤     during gate time)             │
    Signal          │                                   │
    (1 sec)         └───────────────────────────────────┘

Operation:
1. Counter cleared
2. Gate opens for exactly 1 second
3. Counter counts input cycles during gate time
4. Display shows frequency in Hz
```

### 9.3 Event Counter

```
EVENT COUNTER:

Counts occurrences of external events

    Event ────┬────────┤ Counter ├──── Count Display
    Input     │        │         │
              │        └────┬────┘
              │             │
    Debounce ─┘        Reset├──── Reset Button
    Circuit

Applications:
  - People counter
  - Product counter on assembly line
  - Parking lot space counter
```

### 9.4 Timer/Delay Generator

```
TIMER USING COUNTER:

    Clock ───────┤ MOD-N  ├───────── Output Pulse
    (known f)    │Counter │
                 └────────┘

Time delay = N / f

Example: 1ms delay with 1MHz clock
  N = 1ms × 1MHz = 1000
  Use MOD-1000 counter

PROGRAMMABLE TIMER:

    Clock ───────┤ Counter ├──────┐
                 │         │      │
    Preset ──────┤ Compare ├──────┴──── Output
    Value N      │         │
                 └─────────┘

Count until counter = preset value N
```

### 9.5 Address Generator

```
ADDRESS GENERATOR FOR MEMORY:

    Clock ────┤ Binary  ├──── Address Bus
              │ Counter │     A₀-Aₙ₋₁
              └─────────┘

Generates sequential addresses for:
  - ROM reading
  - RAM refresh
  - DMA transfers

Example: 4-bit counter generates addresses 0000-1111 (0-15)
```

---

## 10. Summary Tables

### Counter Types Comparison

| Type | Structure | Speed | Complexity | Glitches |
|------|-----------|-------|------------|----------|
| Ripple (Async) | Series FFs | Slow | Low | Yes |
| Synchronous | Parallel FFs | Fast | Higher | No |
| Ring | Shift register | Fast | Low | No |
| Johnson | Twisted ring | Fast | Low | No |

### Counter Characteristics

| Counter | FFs | States | Decoding |
|---------|-----|--------|----------|
| n-bit Binary | n | 2ⁿ | n-input gates |
| n-bit Ring | n | n | None |
| n-bit Johnson | n | 2n | 2-input gates |
| MOD-N | ⌈log₂N⌉ | N | Varies |

### Common Counter ICs

| IC Number | Type | Description |
|-----------|------|-------------|
| 7490 | TTL | Decade counter (÷2, ÷5) |
| 7493 | TTL | 4-bit binary counter |
| 74161 | TTL | Sync 4-bit, sync clear |
| 74163 | TTL | Sync 4-bit, sync load |
| 74190 | TTL | BCD up/down counter |
| 74193 | TTL | 4-bit up/down counter |
| 4017 | CMOS | Decade counter/divider |
| 4020 | CMOS | 14-bit binary counter |

### Excitation Table Quick Reference

| Q→Q⁺ | JK | D | T |
|------|-----|---|---|
| 0→0 | 0X | 0 | 0 |
| 0→1 | 1X | 1 | 1 |
| 1→0 | X1 | 0 | 1 |
| 1→1 | X0 | 1 | 0 |

---

## 11. Quick Revision Questions

### Question 1
What is the maximum count for a 5-bit binary counter? How many flip-flops are needed for a MOD-20 counter?

<details>
<summary>Click for Answer</summary>

```
5-BIT BINARY COUNTER:
Maximum count = 2⁵ - 1 = 31
Counts: 0 to 31 (32 states, MOD-32)

MOD-20 COUNTER:
FFs needed = ⌈log₂(20)⌉ = ⌈4.32⌉ = 5 flip-flops

(4 FFs give only 16 states, not enough)
(5 FFs give 32 states, use feedback to skip 12 states)
```
</details>

### Question 2
Explain why synchronous counters are faster than asynchronous counters.

<details>
<summary>Click for Answer</summary>

```
ASYNCHRONOUS (RIPPLE) COUNTER:
- Clock ripples through FFs sequentially
- Each FF waits for previous FF to change
- Total delay = n × tpd
- Example: 4-bit with 10ns each = 40ns

SYNCHRONOUS COUNTER:
- All FFs clocked simultaneously
- All FFs change at same time
- Total delay = 1 × tpd + combinational logic delay
- Example: 4-bit = ~15ns

WHY FASTER:
1. No accumulated propagation delay
2. All outputs valid after one tpd
3. No intermediate glitch states
4. Maximum frequency much higher

fmax(sync) >> fmax(async)
```
</details>

### Question 3
Design a MOD-6 synchronous counter using T flip-flops.

<details>
<summary>Click for Answer</summary>

```
MOD-6: Counts 0→1→2→3→4→5→0
Need 3 flip-flops (Q₂Q₁Q₀)

STATE TABLE:
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│ Q₂  │ Q₁  │ Q₀  │Q₂⁺  │Q₁⁺  │Q₀⁺  │ T₂  │ T₁  │ T₀  │
├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
│  0  │  0  │  0  │  0  │  0  │  1  │  0  │  0  │  1  │
│  0  │  0  │  1  │  0  │  1  │  0  │  0  │  1  │  1  │
│  0  │  1  │  0  │  0  │  1  │  1  │  0  │  0  │  1  │
│  0  │  1  │  1  │  1  │  0  │  0  │  1  │  1  │  1  │
│  1  │  0  │  0  │  1  │  0  │  1  │  0  │  0  │  1  │
│  1  │  0  │  1  │  0  │  0  │  0  │  1  │  0  │  1  │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘

From K-maps (with don't cares for 110, 111):
  T₀ = 1
  T₁ = Q₀ · Q₂'
  T₂ = Q₀ · Q₁
```
</details>

### Question 4
How many states does a 4-bit Johnson counter have? List them.

<details>
<summary>Click for Answer</summary>

```
4-BIT JOHNSON COUNTER:
States = 2n = 2(4) = 8 states

STATE SEQUENCE:
┌───────┬──────────┐
│ State │ Q₃Q₂Q₁Q₀ │
├───────┼──────────┤
│  S₀   │   0000   │
│  S₁   │   1000   │
│  S₂   │   1100   │
│  S₃   │   1110   │
│  S₄   │   1111   │
│  S₅   │   0111   │
│  S₆   │   0011   │
│  S₇   │   0001   │
└───────┴──────────┘

Pattern: 1s "fill in" from left, then 0s "fill in"

Feedback: Q₃' → D₀
```
</details>

### Question 5
What is the purpose of the terminal count (TC) or ripple carry output (RCO) in a counter IC?

<details>
<summary>Click for Answer</summary>

```
TERMINAL COUNT (TC) / RIPPLE CARRY (RCO):

PURPOSE:
1. Indicates counter has reached maximum count
2. Enables cascading multiple counters

TC/RCO = 1 when counter = maximum value

CASCADING EXAMPLE (8-bit from two 4-bit):

    CLK ───┤ 4-bit  ├──────────┤ 4-bit  ├
           │Counter │    RCO   │Counter │
           │ (LSB)  ├──────────┤ ENT    │
           │        │          │ (MSB)  │
           └────────┘          └────────┘

When first counter reaches 1111:
  - RCO goes HIGH
  - Enables second counter for one clock
  - First counter wraps to 0000
  - Second counter increments

Result: 8-bit counter (00000000 to 11111111)
```
</details>

### Question 6
Compare ring counter and Johnson counter for the same number of flip-flops.

<details>
<summary>Click for Answer</summary>

```
FOR 4 FLIP-FLOPS:

┌─────────────────────┬──────────────┬──────────────┐
│    Feature          │ Ring Counter │   Johnson    │
├─────────────────────┼──────────────┼──────────────┤
│ Number of states    │      4       │      8       │
│ Feedback            │  Q₃ → D₀     │  Q₃' → D₀    │
│ Decoding needed     │  None        │  2-input AND │
│ State pattern       │ One-hot      │  Shifting 1s │
│ Invalid states      │     12       │      8       │
│ Self-starting       │     No       │      No      │
│ Glitch-free decode  │     Yes      │      Yes     │
│ FF utilization      │  25% (4/16)  │  50% (8/16)  │
└─────────────────────┴──────────────┴──────────────┘

STATES COMPARISON:
Ring: 1000, 0100, 0010, 0001 (4 states)
Johnson: 0000, 1000, 1100, 1110, 1111, 0111, 0011, 0001 (8 states)

Johnson is more efficient for number of states per flip-flop.
```
</details>

---

## Navigation

| Previous | [Home](README.md) | Next |
|:--------:|:-----------------:|:----:|
| [Unit 5: Flip-Flops](Unit-05-Flip-Flops.md) | Return to Index | [Unit 7: Registers](Unit-07-Registers.md) |

---

*End of Unit 6*
