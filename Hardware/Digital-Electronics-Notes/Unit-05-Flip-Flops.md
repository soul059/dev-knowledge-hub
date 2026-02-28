# Unit 5: Sequential Circuits - Flip-Flops

---

## Table of Contents
1. [Introduction to Sequential Circuits](#1-introduction-to-sequential-circuits)
2. [Latches](#2-latches)
3. [Flip-Flop Fundamentals](#3-flip-flop-fundamentals)
4. [SR Flip-Flop](#4-sr-flip-flop)
5. [D Flip-Flop](#5-d-flip-flop)
6. [JK Flip-Flop](#6-jk-flip-flop)
7. [T Flip-Flop](#7-t-flip-flop)
8. [Flip-Flop Conversions](#8-flip-flop-conversions)
9. [Timing Parameters](#9-timing-parameters)
10. [Master-Slave Configuration](#10-master-slave-configuration)
11. [Applications](#11-applications)
12. [Summary Tables](#12-summary-tables)
13. [Quick Revision Questions](#13-quick-revision-questions)

---

## 1. Introduction to Sequential Circuits

### Combinational vs Sequential

```
┌─────────────────────────────────────────────────────────────────┐
│               SEQUENTIAL CIRCUIT CONCEPT                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  COMBINATIONAL CIRCUIT:                                         │
│    Output = f(Current Inputs)                                  │
│                                                                 │
│        Inputs ────┤ Logic ├──── Outputs                        │
│                                                                 │
│  SEQUENTIAL CIRCUIT:                                            │
│    Output = f(Current Inputs, Present State)                   │
│                                                                 │
│        Inputs ────┤        ├──── Outputs                       │
│                   │ Logic  │                                    │
│              ┌────┤        ├────┐                              │
│              │    └────────┘    │                              │
│              │                  │                              │
│              │   ┌────────┐     │                              │
│              └───┤ Memory ├─────┘                              │
│                  │(State) │                                     │
│                  └────────┘                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Types of Sequential Circuits

```
┌─────────────────────────────────────────────────────────────────┐
│               CLASSIFICATION                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ASYNCHRONOUS:                                                  │
│    - State changes immediately with input changes              │
│    - No clock signal                                           │
│    - Faster but prone to timing hazards                        │
│    - Examples: Latches                                         │
│                                                                 │
│  SYNCHRONOUS:                                                   │
│    - State changes only at clock edges                         │
│    - Uses clock signal for synchronization                     │
│    - Easier to design and analyze                              │
│    - Examples: Flip-flops, Counters                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Memory Elements

```
┌─────────────────────────────────────────────────────────────────┐
│                  MEMORY ELEMENTS                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  LATCH:                                                         │
│    - Level sensitive (transparent when enabled)                │
│    - Output follows input while enabled                        │
│    - Simpler, faster                                           │
│    - Timing analysis more complex                              │
│                                                                 │
│  FLIP-FLOP:                                                     │
│    - Edge sensitive (triggers on clock edge)                   │
│    - Output changes only at clock edge                         │
│    - More predictable timing                                   │
│    - Standard building block                                   │
│                                                                 │
│                  Latch              Flip-Flop                  │
│              ┌─────────┐         ┌─────────┐                   │
│         D ───┤         ├── Q     D ───┤         ├── Q          │
│              │         │              │         │              │
│        En ───┤         ├── Q'   CLK ──┤>        ├── Q'         │
│              └─────────┘              └─────────┘              │
│              Level-triggered       Edge-triggered              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Latches

### 2.1 SR Latch (using NOR gates)

```
SR LATCH WITH NOR GATES:

               ┌─────┐
       S ──────┤     │
               │ NOR ├─────┬────── Q
           ┌───┤     │     │
           │   └─────┘     │
           │               │
           │   ┌─────┐     │
           │   │     │     │
           └───┤ NOR ├─────┼────── Q'
       R ──────┤     │     │
               └─────┘     │
                   ▲       │
                   └───────┘
                 (cross-coupled)

TRUTH TABLE (NOR SR Latch):
┌─────┬─────┬───────┬────────┬─────────────────────┐
│  S  │  R  │ Q(t+1)│ Q'(t+1)│      Condition      │
├─────┼─────┼───────┼────────┼─────────────────────┤
│  0  │  0  │  Q(t) │ Q'(t)  │  No Change (Hold)   │
│  0  │  1  │   0   │   1    │  Reset              │
│  1  │  0  │   1   │   0    │  Set                │
│  1  │  1  │   0   │   0    │  Invalid/Forbidden  │
└─────┴─────┴───────┴────────┴─────────────────────┘

CHARACTERISTIC EQUATION:
Q(t+1) = S + R'Q(t)   with constraint: SR = 0

TIMING DIAGRAM:
    S   ────┐     ┌───────────────┐     ┌────────
            └─────┘               └─────┘
            
    R   ──────────────┐     ┌─────────────────────
                      └─────┘

    Q   ────┐     ┌───┘     └───┐     ┌───────────
            └─────┘             └─────┘

    Q'  ────┘     └───┐     ┌───┘     └───────────
                      └─────┘
```

### 2.2 SR Latch (using NAND gates)

```
SR LATCH WITH NAND GATES:

               ┌─────┐
       S' ─────┤     │
               │NAND ├─────┬────── Q
           ┌───┤     │     │
           │   └─────┘     │
           │               │
           │   ┌─────┐     │
           │   │     │     │
           └───┤NAND ├─────┼────── Q'
       R' ─────┤     │     │
               └─────┘     │
                   ▲       │
                   └───────┘

NOTE: Active-low inputs (S' and R')

TRUTH TABLE (NAND SR Latch):
┌─────┬─────┬───────┬────────┬─────────────────────┐
│  S' │  R' │ Q(t+1)│ Q'(t+1)│      Condition      │
├─────┼─────┼───────┼────────┼─────────────────────┤
│  1  │  1  │  Q(t) │ Q'(t)  │  No Change (Hold)   │
│  1  │  0  │   0   │   1    │  Reset              │
│  0  │  1  │   1   │   0    │  Set                │
│  0  │  0  │   1   │   1    │  Invalid/Forbidden  │
└─────┴─────┴───────┴────────┴─────────────────────┘

Comparison:
┌───────────┬─────────────┬─────────────┐
│           │   NOR SR    │   NAND SR   │
├───────────┼─────────────┼─────────────┤
│ Set       │   S=1,R=0   │   S'=0,R'=1 │
│ Reset     │   S=0,R=1   │   S'=1,R'=0 │
│ Hold      │   S=0,R=0   │   S'=1,R'=1 │
│ Invalid   │   S=1,R=1   │   S'=0,R'=0 │
└───────────┴─────────────┴─────────────┘
```

### 2.3 Gated SR Latch

```
GATED SR LATCH (SR Latch with Enable):

           ┌─────┐
       S ──┤     │     ┌─────┐
           │ AND ├─────┤     │
      En ──┤     │     │NAND ├─────┬────── Q
           └─────┘ ┌───┤     │     │
                   │   └─────┘     │
           ┌─────┐ │               │
       R ──┤     │ │   ┌─────┐     │
           │ AND ├─│───┤     │     │
      En ──┤     │ └───┤NAND ├─────┼────── Q'
           └─────┘     │     │     │
                       └─────┘     │
                           ▲       │
                           └───────┘

TRUTH TABLE:
┌─────┬─────┬─────┬────────┬───────────────────┐
│ En  │  S  │  R  │ Q(t+1) │     Action        │
├─────┼─────┼─────┼────────┼───────────────────┤
│  0  │  X  │  X  │  Q(t)  │ Hold (disabled)   │
│  1  │  0  │  0  │  Q(t)  │ Hold              │
│  1  │  0  │  1  │   0    │ Reset             │
│  1  │  1  │  0  │   1    │ Set               │
│  1  │  1  │  1  │   ?    │ Invalid           │
└─────┴─────┴─────┴────────┴───────────────────┘

TIMING (Level-sensitive):
    En  ─────┐        ┌──────────────────────────
             └────────┘
             
    S   ───────────┐         ┌───────────────────
                   └─────────┘
    
    Q   ──────┐    │    ┌────┘         
              └────┴────┘
               ^        ^
               │        │
        Q changes    Q changes
        when S=1     when S goes high
        and En=1     while En=1
```

### 2.4 D Latch (Transparent Latch)

```
D LATCH:

Eliminates invalid state by deriving R from D

           ┌─────┐
       D ──┤     │     ┌─────┐
           │ AND ├─────┤     │
      En ──┤     │     │NAND ├─────┬────── Q
           └─────┘ ┌───┤     │     │
                   │   └─────┘     │
               ┌───│───────────────│───┐
               │   │   ┌─────┐     │   │
       D ──|>o─│   │   │     │     │   │
               │   └───┤NAND ├─────┼───┼── Q'
      En ──────┼───────┤     │     │   │
               │       └─────┘     │   │
               │           ▲       │   │
               │           └───────┘   │
               └───────────────────────┘

TRUTH TABLE:
┌─────┬─────┬────────┬───────────────────┐
│ En  │  D  │ Q(t+1) │     Action        │
├─────┼─────┼────────┼───────────────────┤
│  0  │  X  │  Q(t)  │ Hold (latched)    │
│  1  │  0  │   0    │ Transparent (Q=D) │
│  1  │  1  │   1    │ Transparent (Q=D) │
└─────┴─────┴────────┴───────────────────┘

CHARACTERISTIC EQUATION:
Q(t+1) = En·D + En'·Q(t)

When En=1: Q = D (transparent)
When En=0: Q = Q(t) (latched)

TIMING DIAGRAM:
    En  ─────┐     ┌──────────┐     ┌────────────
             └─────┘          └─────┘

    D   ─────────┐     ┌──────────────┐     ┌────
                 └─────┘              └─────┘

    Q   ─────────┐     ┌──────────────┐     └────
                 └─────┘              │
                                     hold
         transparent   transparent    (Q stays)
```

---

## 3. Flip-Flop Fundamentals

### Edge-Triggered Operation

```
┌─────────────────────────────────────────────────────────────────┐
│                  EDGE TRIGGERING                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  POSITIVE EDGE-TRIGGERED:                                       │
│    State changes on rising edge (0→1) of clock                 │
│                                                                 │
│        CLK ────┐     ┌──────┐     ┌──────                      │
│                └─────┘      └─────┘                            │
│                ↑           ↑                                    │
│            samples      samples                                │
│            inputs       inputs                                  │
│                                                                 │
│  Symbol:  ┌─────┐                                              │
│       D ──┤     ├── Q                                          │
│           │>    │        (> indicates edge-triggered)          │
│      CLK──┤     ├── Q'                                         │
│           └─────┘                                              │
│                                                                 │
│  NEGATIVE EDGE-TRIGGERED:                                       │
│    State changes on falling edge (1→0) of clock                │
│                                                                 │
│        CLK ────┐     ┌──────┐     ┌──────                      │
│                └─────┘      └─────┘                            │
│                     ↑            ↑                              │
│                 samples      samples                           │
│                 inputs       inputs                             │
│                                                                 │
│  Symbol:  ┌─────┐                                              │
│       D ──┤     ├── Q                                          │
│           │>o   │        (>o indicates falling edge)           │
│      CLK──┤     ├── Q'                                         │
│           └─────┘                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Flip-Flop Block Diagram

```
GENERIC FLIP-FLOP:

              ┌─────────────────────┐
              │                     │
     Preset ──┤ PR              Q   ├─────────── Q
              │                     │
        D  ───┤ D                   │
        (or   │                     │
        S,R   │>                    │
        J,K   │                     │
        T)    │               Q'    ├─────────── Q'
              │                     │
      CLK  ───┤ CLK            CLR  │
              │                     │
              └─────────┬───────────┘
                        │
                      Clear

Preset (PR): Asynchronous set (Q = 1)
Clear (CLR): Asynchronous reset (Q = 0)
CLK: Clock input
```

---

## 4. SR Flip-Flop

### Truth Table and Operation

```
SR FLIP-FLOP TRUTH TABLE:
┌─────┬─────┬─────┬────────┬───────────────────┐
│ CLK │  S  │  R  │ Q(t+1) │     Action        │
├─────┼─────┼─────┼────────┼───────────────────┤
│  ↑  │  0  │  0  │  Q(t)  │ No change (Hold)  │
│  ↑  │  0  │  1  │   0    │ Reset             │
│  ↑  │  1  │  0  │   1    │ Set               │
│  ↑  │  1  │  1  │   ?    │ Invalid/Race      │
└─────┴─────┴─────┴────────┴───────────────────┘

CHARACTERISTIC EQUATION:
Q(t+1) = S + R'Q(t)    with constraint: SR = 0

EXCITATION TABLE (for designing counters):
┌───────┬────────┬─────┬─────┐
│  Q(t) │ Q(t+1) │  S  │  R  │
├───────┼────────┼─────┼─────┤
│   0   │   0    │  0  │  X  │
│   0   │   1    │  1  │  0  │
│   1   │   0    │  0  │  1  │
│   1   │   1    │  X  │  0  │
└───────┴────────┴─────┴─────┘

BLOCK SYMBOL:
              ┌─────────────┐
              │   ________  │
      PR ─────┤  |        | │
              │             │
       S ─────┤ S       Q   ├───── Q
              │             │
     CLK ─────┤>            │
              │             │
       R ─────┤ R       Q'  ├───── Q'
              │   ________  │
      CLR ────┤  |        | │
              └─────────────┘

CIRCUIT (Edge-triggered using NAND gates):

         S ───────────┐      ┌─────┐
                      │      │     │
         CLK ────┬────┼──────┤NAND ├───┬──── Q
                 │    │  ┌───┤     │   │
                 │    │  │   └─────┘   │
                 │    │  │             │
                 │    │  │   ┌─────┐   │
                 │    │  │   │     │   │
                 │    └──│───┤NAND ├───┼──── Q'
                 │       └───┤     │   │
         R ──────┴───────────┤     │   │
                             └─────┘   │
                                 ▲     │
                                 └─────┘
```

### Timing Diagram

```
TIMING DIAGRAM (Positive Edge-Triggered SR FF):

    CLK ──────┐     ┌─────┐     ┌─────┐     ┌─────┐     ┌─────
              └─────┘     └─────┘     └─────┘     └─────┘
              1     2     3     4     5     6     7     8

    S   ──────────────┐     ┌─────────────────────────────────
                      └─────┘

    R   ──────┐     ┌─────────────────┐     ┌─────────────────
              └─────┘                 └─────┘

    Q   ──────┐           ┌─────────────────┐     ┌───────────
              └───────────┘                 └─────┘

    Analysis:
    Edge 1: S=0, R=1 → Q=0 (Reset)
    Edge 2: S=0, R=0 → Q=0 (Hold)
    Edge 3: S=1, R=0 → Q=1 (Set)
    Edge 4: S=0, R=0 → Q=1 (Hold)
    Edge 5: S=0, R=1 → Q=0 (Reset)
    Edge 6: S=1, R=0 → Q=1 (Set)
```

---

## 5. D Flip-Flop

### Truth Table and Operation

```
D FLIP-FLOP TRUTH TABLE:
┌─────┬─────┬────────┬───────────────────┐
│ CLK │  D  │ Q(t+1) │     Action        │
├─────┼─────┼────────┼───────────────────┤
│  ↑  │  0  │   0    │ Clear (Q=0)       │
│  ↑  │  1  │   1    │ Set (Q=1)         │
└─────┴─────┴────────┴───────────────────┘

CHARACTERISTIC EQUATION:
Q(t+1) = D

(Simplest flip-flop - output follows input at clock edge)

EXCITATION TABLE:
┌───────┬────────┬─────┐
│  Q(t) │ Q(t+1) │  D  │
├───────┼────────┼─────┤
│   0   │   0    │  0  │
│   0   │   1    │  1  │
│   1   │   0    │  0  │
│   1   │   1    │  1  │
└───────┴────────┴─────┘

D = Q(t+1)  ← D equals the desired next state!

BLOCK SYMBOL:
              ┌─────────────┐
              │   ________  │
      PR ─────┤  |        | │
              │             │
       D ─────┤ D       Q   ├───── Q
              │             │
     CLK ─────┤>            │
              │             │
              │         Q'  ├───── Q'
              │   ________  │
      CLR ────┤  |        | │
              └─────────────┘

D FLIP-FLOP USING NAND GATES (Master-Slave):

    D ────┬─────────────────────────────────────────────────┐
          │                                                 │
          │  ┌─────┐      ┌─────┐     ┌─────┐     ┌─────┐  │
          └──┤     │      │     │     │     │     │     │  │
             │NAND ├──────┤NAND ├──┬──┤NAND ├──┬──┤NAND ├──┴── Q
   CLK ──┬───┤     │  ┌───┤     │  │  │     │  │  │     │
         │   └─────┘  │   └─────┘  │  └─────┘  │  └─────┘
         │            │            │           │
         │  ┌─────┐   │   ┌─────┐  │  ┌─────┐  │  ┌─────┐
         │  │     │   │   │     │  │  │     │  │  │     │
   D' ───┴──┤NAND ├───┴───┤NAND ├──┴──┤NAND ├──┴──┤NAND ├───── Q'
            │     │       │     │     │     │     │     │
            └─────┘       └─────┘     └─────┘     └─────┘
         
            ←─── Master ───→←───── Slave ─────→
```

### Timing Diagram

```
TIMING DIAGRAM (Positive Edge-Triggered D FF):

    CLK ──────┐     ┌─────┐     ┌─────┐     ┌─────┐     ┌─────
              └─────┘     └─────┘     └─────┘     └─────┘
              ↑     ↑     ↑     ↑     ↑     ↑     ↑     ↑
              1     2     3     4     5     6     7     8

    D   ──────┐     ┌─────────────────────────────┐     ┌─────
              └─────┘                             └─────┘

    Q   ──────────────────┐     ┌─────────────────────────────
                          └─────┘

    Analysis (samples D at rising edge):
    Edge 1: D=0 → Q=0
    Edge 2: D=0 → Q=0
    Edge 3: D=1 → Q=1  ← D sampled as 1
    Edge 4: D=1 → Q=1
    Edge 5: D=1 → Q=1
    Edge 6: D=1 → Q=1
    Edge 7: D=0 → Q=0  ← D sampled as 0
    Edge 8: D=1 → Q=1
```

### D Flip-Flop Applications

```
1. DATA REGISTER:
   - Store data bits in parallel
   
       D₀ ──┤ D   Q ├── Q₀
            │>     │
      CLK ──┤      │
            └──────┘
            
2. SHIFT REGISTER:
   - Chain D flip-flops
   
    Din ──┤ D   Q ├───┤ D   Q ├───┤ D   Q ├── Dout
          │>     │    │>     │    │>     │
     CLK ─┴──────┘────┴──────┘────┴──────┘

3. FREQUENCY DIVIDER:
   - Connect Q' to D
   
          ┌──────────────┐
          │              │
          └──┤ D   Q ├───┴── fout = fin/2
             │>     │
       fin ──┤      │
             └──────┘
```

---

## 6. JK Flip-Flop

### Truth Table and Operation

```
JK FLIP-FLOP TRUTH TABLE:
┌─────┬─────┬─────┬────────┬───────────────────┐
│ CLK │  J  │  K  │ Q(t+1) │     Action        │
├─────┼─────┼─────┼────────┼───────────────────┤
│  ↑  │  0  │  0  │  Q(t)  │ No change (Hold)  │
│  ↑  │  0  │  1  │   0    │ Reset             │
│  ↑  │  1  │  0  │   1    │ Set               │
│  ↑  │  1  │  1  │  Q'(t) │ Toggle            │
└─────┴─────┴─────┴────────┴───────────────────┘

CHARACTERISTIC EQUATION:
Q(t+1) = JQ'(t) + K'Q(t)

Derivation using K-map:
           K
      0       1
    ┌─────┬─────┐
  0 │Q(t) │  0  │
J   ├─────┼─────┤
  1 │  1  │Q'(t)│
    └─────┴─────┘

Q(t+1) = JQ' + K'Q

EXCITATION TABLE:
┌───────┬────────┬─────┬─────┐
│  Q(t) │ Q(t+1) │  J  │  K  │
├───────┼────────┼─────┼─────┤
│   0   │   0    │  0  │  X  │
│   0   │   1    │  1  │  X  │
│   1   │   0    │  X  │  1  │
│   1   │   1    │  X  │  0  │
└───────┴────────┴─────┴─────┘

BLOCK SYMBOL:
              ┌─────────────┐
              │   ________  │
      PR ─────┤  |        | │
              │             │
       J ─────┤ J       Q   ├───── Q
              │             │
     CLK ─────┤>            │
              │             │
       K ─────┤ K       Q'  ├───── Q'
              │   ________  │
      CLR ────┤  |        | │
              └─────────────┘
```

### JK Flip-Flop Circuit

```
JK FLIP-FLOP USING SR FLIP-FLOP + FEEDBACK:

                    ┌───────────────────────────┐
                    │                           │
       J ────┬──────┤     ┌─────┐               │
             │      │     │     │     ┌─────────┼──── Q
      CLK ───┼──────┼─────┤NAND ├─────┤         │
             │      │     │     │     │   SR    │
       Q'────┼──────┤     └─────┘     │   FF    │
             │      │                 │         │
             │      │     ┌─────┐     │         │
             │      │     │     │     │         │
       K ────┼──────┼─────┤NAND ├─────┤         │
             │      │     │     │     │         ├──── Q'
      CLK ───┼──────┼─────┤     │     │         │
             │      │     └─────┘     └─────────┘
       Q ────┴──────┘                       │
             │                              │
             └──────────────────────────────┘

Key insight: J is ANDed with Q', K is ANDed with Q
This creates the toggle behavior when J=K=1
```

### Timing Diagram

```
TIMING DIAGRAM (Positive Edge-Triggered JK FF):

    CLK ──────┐     ┌─────┐     ┌─────┐     ┌─────┐     ┌─────
              └─────┘     └─────┘     └─────┘     └─────┘
              1     2     3     4     5     6     7     8

    J   ──────────────────────────────┐     ┌─────────────────
                                      └─────┘

    K   ──────┐     ┌─────────────────────────────────────────
              └─────┘

    Q   ──────────────────┐     ┌───────────┐     ┌───────────
                          └─────┘           └─────┘

    Analysis:
    Edge 1: J=0, K=1 → Q=0 (Reset)
    Edge 2: J=0, K=0 → Q=0 (Hold)
    Edge 3: J=0, K=0 → Q=0 (Hold) - K changes mid-cycle, ignored
    Edge 4: J=1, K=1 → Q=1 (Toggle: 0→1)
    Edge 5: J=1, K=1 → Q=0 (Toggle: 1→0)
    Edge 6: J=0, K=1 → Q=0 (Reset/Hold)
    Edge 7: J=1, K=1 → Q=1 (Toggle: 0→1)
    Edge 8: J=1, K=1 → Q=0 (Toggle: 1→0)
```

### JK vs SR Flip-Flop

```
┌─────────────────────────────────────────────────────────────────┐
│                 JK vs SR FLIP-FLOP                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SIMILARITY:                                                    │
│    J=0,K=0 ↔ S=0,R=0 → Hold                                    │
│    J=0,K=1 ↔ S=0,R=1 → Reset                                   │
│    J=1,K=0 ↔ S=1,R=0 → Set                                     │
│                                                                 │
│  KEY DIFFERENCE:                                                │
│    J=1,K=1 → Toggle (output complemented)                      │
│    S=1,R=1 → Invalid (race condition)                          │
│                                                                 │
│  JK flip-flop is "Universal" - more versatile                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. T Flip-Flop

### Truth Table and Operation

```
T FLIP-FLOP TRUTH TABLE:
┌─────┬─────┬────────┬───────────────────┐
│ CLK │  T  │ Q(t+1) │     Action        │
├─────┼─────┼────────┼───────────────────┤
│  ↑  │  0  │  Q(t)  │ No change (Hold)  │
│  ↑  │  1  │  Q'(t) │ Toggle            │
└─────┴─────┴────────┴───────────────────┘

CHARACTERISTIC EQUATION:
Q(t+1) = T ⊕ Q(t) = TQ'(t) + T'Q(t)

EXCITATION TABLE:
┌───────┬────────┬─────┐
│  Q(t) │ Q(t+1) │  T  │
├───────┼────────┼─────┤
│   0   │   0    │  0  │
│   0   │   1    │  1  │
│   1   │   0    │  1  │
│   1   │   1    │  0  │
└───────┴────────┴─────┘

T = Q(t) ⊕ Q(t+1)

BLOCK SYMBOL:
              ┌─────────────┐
              │   ________  │
      PR ─────┤  |        | │
              │             │
       T ─────┤ T       Q   ├───── Q
              │             │
     CLK ─────┤>            │
              │             │
              │         Q'  ├───── Q'
              │   ________  │
      CLR ────┤  |        | │
              └─────────────┘
```

### T Flip-Flop Implementations

```
T FLIP-FLOP FROM JK FLIP-FLOP:

Connect J = K = T

              ┌─────────────┐
              │             │
       T ─────┤ J       Q   ├───── Q
              │             │
     CLK ─────┤>            │
              │             │
       T ─────┤ K       Q'  ├───── Q'
              │             │
              └─────────────┘

When T=0: J=K=0 → Hold
When T=1: J=K=1 → Toggle


T FLIP-FLOP FROM D FLIP-FLOP:

              ┌──────────────────┐
              │                  │
       T ─────┤     ┌─────┐      │
              │     │     │      │
              ├─────┤ XOR ├──────┤ D   Q ├───┬── Q
              │     │     │      │>     │   │
       Q ─────│─────┤     │      │      │   │
              │     └─────┘     CLK     │   │
              │                  └──────┘   │
              │                             │
              └─────────────────────────────┘

D = T ⊕ Q
```

### Timing Diagram

```
TIMING DIAGRAM (T Flip-Flop, T always 1):

    CLK ──────┐     ┌─────┐     ┌─────┐     ┌─────┐     ┌─────
              └─────┘     └─────┘     └─────┘     └─────┘
              1     2     3     4     5     6     7     8

    T   ──────────────────────────────────────────────────────
          (T = 1 always)

    Q   ──────┐     ┌─────┐     ┌─────┐     ┌─────┐     ┌─────
              └─────┘     └─────┘     └─────┘     └─────┘
              
Output toggles on every rising edge when T=1

This is a FREQUENCY DIVIDER: fout = fin/2

T FLIP-FLOP AS DIVIDE-BY-2:

                    ┌─────────────┐
          1 (VCC)───┤ T       Q   ├───── fout = fin/2
                    │             │
       fin ─────────┤>            │
                    │             │
                    │         Q'  ├───── 
                    └─────────────┘
```

---

## 8. Flip-Flop Conversions

### Conversion Procedure

```
┌─────────────────────────────────────────────────────────────────┐
│            FLIP-FLOP CONVERSION PROCEDURE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  To convert flip-flop X to flip-flop Y:                        │
│                                                                 │
│  1. Write characteristic table of desired flip-flop (Y)        │
│  2. Write excitation table of available flip-flop (X)          │
│  3. Create conversion table combining both                     │
│  4. Derive input expressions using K-maps                      │
│  5. Draw circuit                                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### SR to JK Conversion

```
CONVERSION TABLE:
┌───────┬─────┬─────┬────────┬─────┬─────┐
│  Q(t) │  J  │  K  │ Q(t+1) │  S  │  R  │
├───────┼─────┼─────┼────────┼─────┼─────┤
│   0   │  0  │  0  │   0    │  0  │  X  │
│   0   │  0  │  1  │   0    │  0  │  X  │
│   0   │  1  │  0  │   1    │  1  │  0  │
│   0   │  1  │  1  │   1    │  1  │  0  │
│   1   │  0  │  0  │   1    │  X  │  0  │
│   1   │  0  │  1  │   0    │  0  │  1  │
│   1   │  1  │  0  │   1    │  X  │  0  │
│   1   │  1  │  1  │   0    │  0  │  1  │
└───────┴─────┴─────┴────────┴─────┴─────┘

K-MAPS:
        JK                      JK
      00  01  11  10          00  01  11  10
    ┌───┬───┬───┬───┐       ┌───┬───┬───┬───┐
  0 │ 0 │ 0 │ 1 │ 1 │     0 │ X │ X │ 0 │ 0 │
Q   ├───┼───┼───┼───┤   Q   ├───┼───┼───┼───┤
  1 │ X │ 0 │ 0 │ X │     1 │ 0 │ 1 │ 1 │ 0 │
    └───┴───┴───┴───┘       └───┴───┴───┴───┘
          S                       R

S = JQ'                   R = KQ

CIRCUIT:
              ┌─────────────┐
              │             │
       J ─────┤───────┐     │
              │       │     │
       Q'─────┤─┐     │D────┤ S       Q   ├───┬── Q
              │ │     │     │             │   │
     CLK ─────│─│─────│─────┤>            │   │
              │ │     │     │             │   │
       K ─────┤─│─┐   │     │ R       Q'  ├───┼── Q'
              │ │ │   │     │             │   │
       Q ─────┤─│─│─┐ │D────┤             │   │
              │ │ │ │       └─────────────┘   │
              │ └─│─│──────────────────────────┘
              │   │ │
              │   └─│─────────┐
              │     └─────────│─┐
              │         AND   └─│── To R
              │               AND
              │                 └── To S
              └──────────────────────────────────
```

### JK to D Conversion

```
CONVERSION TABLE:
┌───────┬─────┬────────┬─────┬─────┐
│  Q(t) │  D  │ Q(t+1) │  J  │  K  │
├───────┼─────┼────────┼─────┼─────┤
│   0   │  0  │   0    │  0  │  X  │
│   0   │  1  │   1    │  1  │  X  │
│   1   │  0  │   0    │  X  │  1  │
│   1   │  1  │   1    │  X  │  0  │
└───────┴─────┴────────┴─────┴─────┘

K-MAPS:
         D                       D
       0     1                 0     1
     ┌─────┬─────┐           ┌─────┬─────┐
   0 │  0  │  1  │         0 │  X  │  X  │
 Q   ├─────┼─────┤       Q   ├─────┼─────┤
   1 │  X  │  X  │         1 │  1  │  0  │
     └─────┴─────┘           └─────┴─────┘
           J                       K

J = D                      K = D'

CIRCUIT:
              ┌─────────────┐
              │             │
       D ─────┤ J       Q   ├───── Q
              │             │
     CLK ─────┤>            │
              │             │
       D ──|>o┤ K       Q'  ├───── Q'
              │             │
              └─────────────┘
```

### JK to T Conversion

```
CONVERSION TABLE:
┌───────┬─────┬────────┬─────┬─────┐
│  Q(t) │  T  │ Q(t+1) │  J  │  K  │
├───────┼─────┼────────┼─────┼─────┤
│   0   │  0  │   0    │  0  │  X  │
│   0   │  1  │   1    │  1  │  X  │
│   1   │  0  │   1    │  X  │  0  │
│   1   │  1  │   0    │  X  │  1  │
└───────┴─────┴────────┴─────┴─────┘

K-MAPS:
         T                       T
       0     1                 0     1
     ┌─────┬─────┐           ┌─────┬─────┐
   0 │  0  │  1  │         0 │  X  │  X  │
 Q   ├─────┼─────┤       Q   ├─────┼─────┤
   1 │  X  │  X  │         1 │  0  │  1  │
     └─────┴─────┘           └─────┴─────┘
           J                       K

J = T                      K = T

CIRCUIT:
              ┌─────────────┐
              │             │
       T ──┬──┤ J       Q   ├───── Q
           │  │             │
     CLK ──│──┤>            │
           │  │             │
           └──┤ K       Q'  ├───── Q'
              │             │
              └─────────────┘
```

### D to JK Conversion

```
CONVERSION TABLE:
┌───────┬─────┬─────┬────────┬─────┐
│  Q(t) │  J  │  K  │ Q(t+1) │  D  │
├───────┼─────┼─────┼────────┼─────┤
│   0   │  0  │  0  │   0    │  0  │
│   0   │  0  │  1  │   0    │  0  │
│   0   │  1  │  0  │   1    │  1  │
│   0   │  1  │  1  │   1    │  1  │
│   1   │  0  │  0  │   1    │  1  │
│   1   │  0  │  1  │   0    │  0  │
│   1   │  1  │  0  │   1    │  1  │
│   1   │  1  │  1  │   0    │  0  │
└───────┴─────┴─────┴────────┴─────┘

K-MAP FOR D:
        JK
      00  01  11  10
    ┌───┬───┬───┬───┐
  0 │ 0 │ 0 │ 1 │ 1 │
Q   ├───┼───┼───┼───┤
  1 │ 1 │ 0 │ 0 │ 1 │
    └───┴───┴───┴───┘

D = JQ' + K'Q

CIRCUIT:
        J ────────┐
                  │D─────┐
        Q' ───────┤      │
                  │      │  ┌─────────────┐
                         └──┤ D       Q   ├───── Q
                            │             │
        K' ───────┐      ┌──┤>            │
                  │D─────┘  │             │
        Q  ───────┤         │         Q'  ├───── Q'
                            │             │
       CLK ─────────────────┤             │
                            └─────────────┘
```

---

## 9. Timing Parameters

### Critical Timing Specifications

```
┌─────────────────────────────────────────────────────────────────┐
│               FLIP-FLOP TIMING PARAMETERS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. SETUP TIME (tsu):                                          │
│     - Minimum time data must be stable BEFORE clock edge       │
│     - If violated: Metastability possible                      │
│                                                                 │
│  2. HOLD TIME (th):                                            │
│     - Minimum time data must remain stable AFTER clock edge    │
│     - If violated: Wrong data captured                         │
│                                                                 │
│  3. PROPAGATION DELAY (tpd):                                   │
│     - Time from clock edge to output change                    │
│     - tpLH: Low-to-High transition                             │
│     - tpHL: High-to-Low transition                             │
│                                                                 │
│  4. CLOCK PULSE WIDTH:                                         │
│     - twH: Minimum high time                                   │
│     - twL: Minimum low time                                    │
│                                                                 │
│  5. CLOCK FREQUENCY (fmax):                                    │
│     - Maximum operating frequency                              │
│     - fmax = 1 / (tpd + tsu + tskew)                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Timing Diagram with Parameters

```
DETAILED TIMING DIAGRAM:

    CLK ─────────┐          ┌─────────────────
                 └──────────┘
                      │
                      ▼ clock edge
    
    D   ─────┐   ┌──────────────┐   ┌─────────
             └───┘              └───┘
         
         ◄─tsu─►◄th►
         
    Q   ─────────────────┐          ┌─────────
                         └──────────┘
                   
         ◄───────tpd────►

    
PARAMETERS ILLUSTRATION:

         ┌───────────────────────────────────┐
    D ───┤   MUST BE    │  MUST BE   │       │
         │   STABLE     │  STABLE    │       │
         └───────────────────────────────────┘
               ◄──tsu───►◄──th──►
                        ↑
                   Clock edge
                        │
                        │
                        ├───tpd───►
                                   │
    Q ─────────────────────────────┴────── changes

TYPICAL VALUES (74 series TTL):
┌──────────────┬───────────┬───────────┐
│  Parameter   │   7474    │  74HC74   │
├──────────────┼───────────┼───────────┤
│ tsu          │   20 ns   │   15 ns   │
│ th           │   5 ns    │   5 ns    │
│ tpd          │   25 ns   │   20 ns   │
│ fmax         │   25 MHz  │   40 MHz  │
└──────────────┴───────────┴───────────┘
```

### Setup and Hold Time Violations

```
METASTABILITY:

When setup/hold times are violated, flip-flop enters 
metastable state - output oscillates or settles to 
unpredictable value.

NORMAL OPERATION:
    D ───────────┐
                 └─────────────── (stable during window)
                    
    CLK ─────────┐     ┌──────────
                 └─────┘
                 ◄tsu►◄th►
                 
    Q   ──────────────────┐       (clean transition)
                          └───────

SETUP TIME VIOLATION:
    D ──────────────┐
                    └──────────── (changes too close to edge)
                    
    CLK ─────────┐     ┌──────────
                 └─────┘
                 ◄─►◄──th──►
                 X (< tsu)
                 
    Q   ──────────────────?????   (metastable - unpredictable)


HOLD TIME VIOLATION:
    D ───────────┐
                 └─────────────── (changes too soon after edge)
                    
    CLK ─────────┐     ┌──────────
                 └─────┘
                 ◄─tsu─►X
                        (< th)
                 
    Q   ──────────────────?????   (wrong value captured)
```

---

## 10. Master-Slave Configuration

### Concept

```
┌─────────────────────────────────────────────────────────────────┐
│               MASTER-SLAVE FLIP-FLOP                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Purpose: Eliminate race condition in JK flip-flop             │
│                                                                 │
│  Structure: Two latches connected in series                    │
│    - Master: Enabled when CLK = 1                              │
│    - Slave: Enabled when CLK = 0                               │
│                                                                 │
│  Behavior:                                                      │
│    - Master captures data when CLK = 1                         │
│    - Slave transfers data when CLK goes 0                      │
│    - Never both transparent simultaneously                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Master-Slave JK Flip-Flop

```
MASTER-SLAVE JK FLIP-FLOP:

       J ─────────────┐
                      │     ┌─────────────┐     ┌─────────────┐
                      ├─────┤             │     │             │
       Q' ────────────┤     │   MASTER    │     │    SLAVE    │
                      │ AND ┤    SR       ├─────┤    SR       ├──── Q
                      │     │   LATCH     │     │   LATCH     │
                      │     │             │     │             │
       K ─────────────┤     │             │     │             ├──── Q'
                      │     │     En      │     │     En      │
       Q  ────────────┤ AND │     │       │     │     │       │
                      │     └─────┼───────┘     └─────┼───────┘
                      │           │                   │
                      │           │                   │
      CLK ────────────┴───────────┴───────|>o─────────┘
                                    CLK        CLK'

TIMING BEHAVIOR:

    CLK ─────────┐          ┌─────────────────┐          ┌────
                 └──────────┘                 └──────────┘
                 
                 ◄─Master─►◄─────Slave─────►◄─Master─►
                  enabled     enabled         enabled
                  
    J,K  ────────────────╳───────────────────╳────────────────
                sampled here                 sampled here
                
    Qm   ────────────────╳                   ╳────────────────
              master updates                 master updates
              
    Q    ─────────────────────╳                   ╳───────────
                       slave updates       slave updates
                       (output changes)    (output changes)
```

### 1's Catching Problem

```
1's CATCHING (Problem with MS FF):

The master latch is transparent for entire CLK high period.
Any glitch on J or K during this time gets captured.

    CLK ─────────┐          ┌─────────
                 └──────────┘
                 
    J   ─────┐   ┌───────────────────
             └───┘
             ↑
             glitch captured by master!

    Qm  ─────────────────────┐
                             └────────
                             ↑
                             captured!

SOLUTION: Edge-triggered flip-flops
  - Only sample at clock edge
  - Immune to glitches during CLK high/low
```

---

## 11. Applications

### 11.1 Data Storage Register

```
4-BIT PARALLEL LOAD REGISTER:

    D₀ ──┤ D   Q ├── Q₀
         │>     │
    D₁ ──┤ D   Q ├── Q₁
         │>     │
    D₂ ──┤ D   Q ├── Q₂
         │>     │
    D₃ ──┤ D   Q ├── Q₃
         │>     │
         │      │
   CLK ──┴──────┘

All 4 bits stored simultaneously on clock edge
```

### 11.2 Shift Register (SISO)

```
SERIAL-IN SERIAL-OUT SHIFT REGISTER:

    Din ──┤ D   Q ├───┤ D   Q ├───┤ D   Q ├───┤ D   Q ├── Dout
          │>     │    │>     │    │>     │    │>     │
          │      │    │      │    │      │    │      │
    CLK ──┴──────┘────┴──────┘────┴──────┘────┴──────┘

Data shifts one position right on each clock
```

### 11.3 Frequency Divider

```
DIVIDE-BY-N USING T FLIP-FLOPS:

DIVIDE-BY-2:
          ┌──────────────────┐
          │                  │
    1 ────┤ T           Q    ├─── fout = fin/2
          │                  │
    fin ──┤>                 │
          └──────────────────┘

DIVIDE-BY-4 (Ripple Counter):
          ┌────────────┐     ┌────────────┐
          │            │     │            │
    1 ────┤ T      Q   ├──┬──┤ T      Q   ├─── fout = fin/4
          │            │  │  │            │
    fin ──┤>           │  └──┤>           │
          └────────────┘     └────────────┘
          
          f/2              f/4
```

### 11.4 Debounce Circuit

```
SWITCH DEBOUNCING USING SR LATCH:

Mechanical switches bounce - multiple transitions on single press

BOUNCING SIGNAL:                 DEBOUNCED OUTPUT:
    ┌─┐ ┌─┐                          ┌─────────────
────┘ └─┘ └───────               ────┘

DEBOUNCE CIRCUIT:
                VCC
                 │
          ┌──────┴──────┐
          │             │
       ───┤ R       Q   ├───── Clean output
    ┌─────┤             │
    │  ┌──┤ S       Q'  ├─────
    │  │  └─────────────┘
    │  │
  SPDT Switch
  ══╪══╪══
    └──┴── to GND
```

### 11.5 Synchronizer

```
SYNCHRONIZER (for asynchronous inputs):

External signals may change at any time - need to synchronize
to internal clock to prevent metastability.

TWO-FLIP-FLOP SYNCHRONIZER:

   Async ──┤ D   Q ├───┤ D   Q ├─── Synchronized
   Input   │>     │    │>     │     Output
           │      │    │      │
   CLK  ───┴──────┘────┴──────┘

First FF may go metastable, but has full clock period to resolve
before second FF samples.
```

---

## 12. Summary Tables

### Flip-Flop Comparison

| Type | Inputs | Characteristic Equation | Toggle | Applications |
|------|--------|-------------------------|--------|--------------|
| SR | S, R | Q+ = S + R'Q (SR=0) | No | Basic storage |
| D | D | Q+ = D | No | Registers, shift |
| JK | J, K | Q+ = JQ' + K'Q | Yes (J=K=1) | Counters, universal |
| T | T | Q+ = T⊕Q | Yes (T=1) | Counters, dividers |

### Excitation Tables Summary

| Q(t) | Q(t+1) | S | R | D | J | K | T |
|------|--------|---|---|---|---|---|---|
| 0 | 0 | 0 | X | 0 | 0 | X | 0 |
| 0 | 1 | 1 | 0 | 1 | 1 | X | 1 |
| 1 | 0 | 0 | 1 | 0 | X | 1 | 1 |
| 1 | 1 | X | 0 | 1 | X | 0 | 0 |

### Latch vs Flip-Flop

| Aspect | Latch | Flip-Flop |
|--------|-------|-----------|
| Triggering | Level-sensitive | Edge-sensitive |
| Transparency | Yes (when enabled) | No |
| Timing | Harder to analyze | Easier |
| Speed | Faster | Slightly slower |
| IC Example | 74HC75 | 74HC74 |

### Common IC Numbers

| Type | TTL | CMOS | Description |
|------|-----|------|-------------|
| D Flip-Flop | 7474 | 4013 | Dual D, +edge |
| JK Flip-Flop | 7476 | 4027 | Dual JK, -edge |
| D Latch | 7475 | 4042 | Quad D latch |
| JK (MS) | 7473 | - | Dual JK master-slave |

---

## 13. Quick Revision Questions

### Question 1
What is the difference between a latch and a flip-flop?

<details>
<summary>Click for Answer</summary>

```
LATCH:
- Level-sensitive (transparent when enabled)
- Output can change anytime enable is active
- Simpler structure, faster
- Example: D latch, SR latch

FLIP-FLOP:
- Edge-sensitive (triggers on clock edge only)
- Output changes only at clock transition
- More predictable timing behavior
- Example: D flip-flop, JK flip-flop

Key Difference:
- Latch: Transparent window during enable
- Flip-flop: Samples only at instant of clock edge
```
</details>

### Question 2
Draw the circuit to convert a JK flip-flop to a D flip-flop.

<details>
<summary>Click for Answer</summary>

```
From conversion:
J = D
K = D'

CIRCUIT:
              ┌─────────────┐
              │             │
       D ──┬──┤ J       Q   ├───── Q
           │  │             │
     CLK ──│──┤>            │
           │  │             │
       D ──┴─o┤ K       Q'  ├───── Q'
              │             │
              └─────────────┘

Simply connect D to J, and D' (inverted D) to K.
- When D=1: J=1, K=0 → Q becomes 1
- When D=0: J=0, K=1 → Q becomes 0
```
</details>

### Question 3
What is setup time and hold time? What happens if they are violated?

<details>
<summary>Click for Answer</summary>

```
SETUP TIME (tsu):
- Minimum time D must be stable BEFORE clock edge
- Required for flip-flop to "see" the correct data

HOLD TIME (th):
- Minimum time D must remain stable AFTER clock edge
- Ensures data is properly latched

VIOLATION CONSEQUENCES:
1. Metastability: Output oscillates between 0 and 1
2. Unpredictable output: May settle to wrong value
3. Can cause system failure

Example timing requirement:
- If tsu = 5ns and th = 2ns
- D must be stable from 5ns BEFORE to 2ns AFTER clock edge
```
</details>

### Question 4
Write the characteristic equation for a JK flip-flop and explain each term.

<details>
<summary>Click for Answer</summary>

```
CHARACTERISTIC EQUATION:
Q(t+1) = JQ'(t) + K'Q(t)

EXPLANATION:

Term 1: JQ'(t)
- Q(t+1) = 1 if J=1 AND Q(t)=0
- This is the SET condition
- "If J wants to set and current state is 0, set it"

Term 2: K'Q(t)
- Q(t+1) = 1 if K=0 AND Q(t)=1
- This is the HOLD 1 condition
- "If K doesn't want to reset and current state is 1, keep it"

Combined behavior:
- J=0,K=0: Q+ = 0 + Q = Q (Hold)
- J=0,K=1: Q+ = 0 + 0 = 0 (Reset)
- J=1,K=0: Q+ = Q' + Q = 1 (Set)
- J=1,K=1: Q+ = Q' + 0 = Q' (Toggle)
```
</details>

### Question 5
How do you implement a frequency divider using a D flip-flop?

<details>
<summary>Click for Answer</summary>

```
Connect Q' back to D:

              ┌──────────────────┐
              │                  │
              │   ┌─────────────┐│
              │   │             ││
              └───┤ D       Q   ├┴─── fout = fin/2
                  │             │
        fin ──────┤>            │
                  │             │
                  │         Q'  ├──── 
                  └─────────────┘

OPERATION:
- At each rising edge, Q takes value of D (which is Q')
- So Q toggles on each clock edge

TIMING:
    fin ──┐  ┌──┐  ┌──┐  ┌──┐  ┌──
          └──┘  └──┘  └──┘  └──┘

   fout ──────┐     ┌─────┐     ┌──
              └─────┘     └─────┘

fout frequency = fin/2

For divide-by-N, cascade N flip-flops
```
</details>

### Question 6
Explain the 1's catching problem in master-slave flip-flops.

<details>
<summary>Click for Answer</summary>

```
1's CATCHING PROBLEM:

In master-slave JK flip-flop:
- Master is transparent while CLK = 1
- Any glitch on J or K during CLK high gets captured

EXAMPLE:
    CLK ─────────┐          ┌─────────
                 └──────────┘
                 ◄──────────►
                 Master transparent
                 
    J   ─────────────┐  ┌─────────────
                     └──┘
                     ↑
                     Glitch captured!

Even though J is low at falling edge, the 
glitch caused Q to change.

SOLUTIONS:
1. Use edge-triggered flip-flops (not MS)
2. Use short clock pulse width
3. Ensure clean input signals during CLK high
```
</details>

---

## Navigation

| Previous | [Home](README.md) | Next |
|:--------:|:-----------------:|:----:|
| [Unit 4: Combinational](Unit-04-Combinational-Circuits.md) | Return to Index | [Unit 6: Counters](Unit-06-Counters.md) |

---

*End of Unit 5*
