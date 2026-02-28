# Unit 8: Finite State Machines (FSM)

---

## Table of Contents
1. [Introduction to FSM](#1-introduction-to-fsm)
2. [Moore Machine](#2-moore-machine)
3. [Mealy Machine](#3-mealy-machine)
4. [Moore vs Mealy Comparison](#4-moore-vs-mealy-comparison)
5. [State Diagrams](#5-state-diagrams)
6. [State Tables](#6-state-tables)
7. [State Assignment](#7-state-assignment)
8. [FSM Design Procedure](#8-fsm-design-procedure)
9. [Complete Design Examples](#9-complete-design-examples)
10. [State Reduction](#10-state-reduction)
11. [Summary Tables](#11-summary-tables)
12. [Quick Revision Questions](#12-quick-revision-questions)

---

## 1. Introduction to FSM

### What is a Finite State Machine?

```
┌─────────────────────────────────────────────────────────────────┐
│                   FINITE STATE MACHINE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  DEFINITION:                                                    │
│    A sequential circuit whose output depends on:               │
│      1. Present state (stored in flip-flops)                   │
│      2. Current inputs                                         │
│                                                                 │
│  "FINITE" because:                                              │
│    Limited number of states (2ⁿ states for n flip-flops)       │
│                                                                 │
│  GENERAL STRUCTURE:                                             │
│                                                                 │
│    Inputs ───────┐                                              │
│                  │     ┌─────────────────┐                      │
│                  ├─────┤  Combinational  ├───── Outputs         │
│                  │     │     Logic       │                      │
│           ┌─────┐│     └────────┬────────┘                      │
│           │     ││              │                               │
│           │State││              │ Next State                    │
│           │     │◄──────────────┘                               │
│           │  FF │                                               │
│           │     │                                               │
│           └──┬──┘                                               │
│              │                                                  │
│              └───── Present State                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### FSM Components

```
┌─────────────────────────────────────────────────────────────────┐
│                    FSM COMPONENTS                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. STATES (S):                                                │
│     Finite set of states: {S₀, S₁, S₂, ..., Sₙ₋₁}             │
│     Stored in flip-flops                                       │
│                                                                 │
│  2. INPUTS (X):                                                │
│     External signals that affect state transitions             │
│                                                                 │
│  3. OUTPUTS (Z):                                               │
│     Signals produced by the FSM                                │
│                                                                 │
│  4. NEXT STATE FUNCTION (δ):                                   │
│     Next State = δ(Present State, Input)                       │
│                                                                 │
│  5. OUTPUT FUNCTION (λ):                                       │
│     Moore: Output = λ(Present State)                           │
│     Mealy: Output = λ(Present State, Input)                    │
│                                                                 │
│  6. INITIAL STATE:                                             │
│     Starting state (usually S₀ or reset state)                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Moore Machine

### Definition and Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                      MOORE MACHINE                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  DEFINITION:                                                    │
│    Output depends ONLY on the present state                    │
│    Output = λ(Present State)                                   │
│                                                                 │
│  OUTPUT IS "ATTACHED" TO STATES                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

MOORE MACHINE ARCHITECTURE:

              ┌─────────────────────────────────────────────────┐
              │                                                 │
    Inputs ───┤───────────┐                                     │
              │           │                                     │
              │           ▼                                     │
              │     ┌───────────┐     ┌───────────┐            │
              │     │   Next    │     │  Output   │            │
              │  ┌──┤   State   │     │   Logic   ├──── Output │
              │  │  │   Logic   │     │           │            │
              │  │  └─────┬─────┘     └─────┬─────┘            │
              │  │        │                 │                   │
              │  │        │ Next            │                   │
              │  │        │ State           │                   │
              │  │        ▼                 │                   │
              │  │  ┌───────────┐           │                   │
              │  │  │   State   │           │                   │
              │  │  │ Register  ├───────────┴──── Present      │
              │  │  │   (FF)    │                  State        │
              │  │  └─────┬─────┘                               │
              │  │        │                                     │
              │  └────────┘                                     │
              │       Feedback                                  │
              │                                                 │
              └─────────────────────────────────────────────────┘

KEY POINT: Output logic receives ONLY state inputs, NOT external inputs
```

### Moore State Diagram Notation

```
MOORE STATE NOTATION:

    ┌─────────────┐
    │             │
    │    State    │
    │   Name/ID   │
    │             │
    │   Output    │
    └─────────────┘

Example:
    ┌─────────────┐
    │     S0      │
    │             │
    │   Z = 0     │
    └─────────────┘

The output is written INSIDE the state circle
(because output depends only on state)

TRANSITION NOTATION:

    ┌─────┐   input   ┌─────┐
    │ S0  │───────────│ S1  │
    │ Z=0 │           │ Z=1 │
    └─────┘           └─────┘

    Transition shows only the INPUT condition
    (output is determined by destination state)
```

---

## 3. Mealy Machine

### Definition and Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                      MEALY MACHINE                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  DEFINITION:                                                    │
│    Output depends on present state AND current inputs          │
│    Output = λ(Present State, Input)                            │
│                                                                 │
│  OUTPUT IS "ATTACHED" TO TRANSITIONS                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

MEALY MACHINE ARCHITECTURE:

              ┌─────────────────────────────────────────────────┐
              │                                                 │
    Inputs ───┤───────┬───────────────────────┐                 │
              │       │                       │                 │
              │       ▼                       ▼                 │
              │ ┌───────────┐           ┌───────────┐          │
              │ │   Next    │           │  Output   │          │
              │ │   State   │           │   Logic   ├── Output │
              │ │   Logic   │           │           │          │
              │ └─────┬─────┘           └─────┬─────┘          │
              │       │                       │                 │
              │       │ Next                  │                 │
              │       │ State                 │                 │
              │       ▼                       │                 │
              │ ┌───────────┐                 │                 │
              │ │   State   │                 │                 │
              │ │ Register  ├─────────────────┴── Present      │
              │ │   (FF)    │                      State        │
              │ └─────┬─────┘                                   │
              │       │                                         │
              │       └───────────────────────────────          │
              │              Feedback                           │
              │                                                 │
              └─────────────────────────────────────────────────┘

KEY POINT: Output logic receives BOTH state AND external inputs
```

### Mealy State Diagram Notation

```
MEALY STATE NOTATION:

    ┌─────────────┐
    │             │
    │    State    │
    │   Name/ID   │
    │             │
    └─────────────┘

State circle contains ONLY the state name
(output written on transitions, not in state)

TRANSITION NOTATION:

    ┌─────┐  input/output  ┌─────┐
    │ S0  │────────────────│ S1  │
    │     │                │     │
    └─────┘                └─────┘

Example:
    ┌─────┐    0/0    ┌─────┐
    │ S0  │───────────│ S1  │
    │     │           │     │
    └─────┘           └─────┘
    
    "When in S0 and input=0, go to S1 and output=0"
```

---

## 4. Moore vs Mealy Comparison

### Side-by-Side Comparison

```
┌────────────────────────────────────────────────────────────────────────┐
│                    MOORE vs MEALY COMPARISON                           │
├───────────────────────┬────────────────────────────────────────────────┤
│       FEATURE         │     MOORE          │       MEALY              │
├───────────────────────┼────────────────────┼────────────────────────────┤
│ Output depends on     │ State only         │ State AND Input          │
├───────────────────────┼────────────────────┼────────────────────────────┤
│ Output changes        │ On clock edge      │ Asynchronously           │
│ when                  │ (with state)       │ (with input change)      │
├───────────────────────┼────────────────────┼────────────────────────────┤
│ Output label          │ In state circle    │ On transition arrow      │
├───────────────────────┼────────────────────┼────────────────────────────┤
│ Number of states      │ Usually more       │ Usually fewer            │
├───────────────────────┼────────────────────┼────────────────────────────┤
│ Output stability      │ More stable        │ May glitch               │
├───────────────────────┼────────────────────┼────────────────────────────┤
│ Response time         │ One clock delayed  │ Faster (immediate)       │
├───────────────────────┼────────────────────┼────────────────────────────┤
│ Output hazards        │ No glitches        │ Possible glitches        │
├───────────────────────┼────────────────────┼────────────────────────────┤
│ Synchronous output    │ Yes                │ No (unless registered)   │
├───────────────────────┼────────────────────┼────────────────────────────┤
│ Complexity            │ Simpler output     │ Simpler state diagram    │
│                       │ logic              │ (fewer states)           │
└───────────────────────┴────────────────────┴────────────────────────────┘
```

### Example: Sequence Detector

```
DETECT: "11" sequence (output 1 when two consecutive 1s detected)

MOORE MACHINE (3 states):
                         ┌──────┐
                   0     │  S0  │     0
              ┌─────────◄│ Z=0  │◄─────────┐
              │          └───┬──┘          │
              │              │1            │
              │              ▼             │
              │          ┌──────┐          │
              │    0     │  S1  │          │
              ├─────────◄│ Z=0  │          │
              │          └───┬──┘          │
              │              │1            │
              │              ▼             │
              │          ┌──────┐          │
              │    1     │  S2  │          │
              │    ┌────►│ Z=1  │──────────┘
              │    └─────└──────┘
              │
              ▼
           (back to S0)

MEALY MACHINE (2 states):
                          ┌──────┐
                    0/0   │      │    0/0
               ┌─────────◄│  S0  │◄─────────┐
               │          │      │          │
               │          └───┬──┘          │
               │              │1/0          │
               │              ▼             │
               │          ┌──────┐          │
               │    0/0   │      │          │
               └─────────◄│  S1  │──────────┘
                          │      │
                          └───┬──┘
                              │
                              └──── 1/1 (self-loop)

Moore needs 3 states; Mealy needs only 2 states!
```

---

## 5. State Diagrams

### State Diagram Symbols

```
┌─────────────────────────────────────────────────────────────────┐
│               STATE DIAGRAM SYMBOLS                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  STATE:                                                         │
│    ┌───────┐                                                    │
│    │       │     Circle or rounded rectangle                   │
│    │  Sₓ   │     Contains state name and output (Moore)        │
│    │       │                                                    │
│    └───────┘                                                    │
│                                                                 │
│  INITIAL STATE:                                                 │
│       ┌───────┐                                                 │
│    ──►│  S0   │     Arrow pointing to initial state            │
│       └───────┘                                                 │
│                                                                 │
│  TRANSITION:                                                    │
│    ┌───────┐  condition  ┌───────┐                             │
│    │  S0   │─────────────│  S1   │   Arrow with label          │
│    └───────┘             └───────┘                             │
│                                                                 │
│  SELF-LOOP:                                                     │
│       ┌─┐                                                       │
│       │ │ condition                                            │
│       ▼ │                                                       │
│    ┌───────┐                                                    │
│    │  Sₓ   │     Transition back to same state                 │
│    └───────┘                                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Complete State Diagram Example

```
EXAMPLE: Traffic Light Controller (simplified)

States:
  - GREEN: Main road has green light
  - YELLOW: Main road has yellow light  
  - RED: Main road has red light

Input: T (timer expired)
Output: Lights (encoded as G/Y/R)

MOORE STATE DIAGRAM:

                    ┌───────────────────────────────────────┐
                    │                                       │
                    ▼             T=0                       │
    ────►┌──────────────┐    ┌────────┐                    │
         │    GREEN     │◄───┤        │                    │
         │   (G=1,Y=0,  │    │        │                    │
         │    R=0)      │    │        │                    │
         └──────┬───────┘    │        │                    │
                │ T=1        │        │                    │
                ▼            │        │                    │
         ┌──────────────┐    │        │                    │
         │   YELLOW     │────┘        │                    │
         │   (G=0,Y=1,  │             │                    │
         │    R=0)      │◄────────────┼────┐               │
         └──────┬───────┘   T=0       │    │               │
                │ T=1                 │    │               │
                ▼                     │    │               │
         ┌──────────────┐             │    │               │
         │     RED      │─────────────┘    │               │
         │   (G=0,Y=0,  │                  │               │
         │    R=1)      │◄─────────────────┘               │
         └──────┬───────┘    T=0                           │
                │ T=1                                      │
                └──────────────────────────────────────────┘
```

---

## 6. State Tables

### Moore State Table Format

```
MOORE STATE TABLE FORMAT:

┌───────────────┬───────────────────────┬──────────┐
│ Present State │      Next State       │  Output  │
│      (PS)     │  X=0    │    X=1      │    Z     │
├───────────────┼─────────┼─────────────┼──────────┤
│      S0       │   S0    │     S1      │    0     │
│      S1       │   S0    │     S2      │    0     │
│      S2       │   S0    │     S2      │    1     │
└───────────────┴─────────┴─────────────┴──────────┘

Note: Output column depends ONLY on present state
      (same output for all input conditions in that state)
```

### Mealy State Table Format

```
MEALY STATE TABLE FORMAT:

┌───────────────┬───────────────────────┬─────────────────────┐
│ Present State │      Next State       │       Output        │
│      (PS)     │  X=0    │    X=1      │   X=0   │   X=1     │
├───────────────┼─────────┼─────────────┼─────────┼───────────┤
│      S0       │   S0    │     S1      │    0    │     0     │
│      S1       │   S0    │     S1      │    0    │     1     │
└───────────────┴─────────┴─────────────┴─────────┴───────────┘

Note: Output columns depend on BOTH present state AND input
      (different outputs possible for different inputs in same state)
```

### Converting State Table to Transition Table

```
ENCODED STATE TABLE (for implementation):

Assign binary codes to states:
  S0 = 00
  S1 = 01
  S2 = 10

TRANSITION TABLE:
┌──────────┬──────────────────┬────────┐
│ Present  │    Next State    │ Output │
│  Q₁ Q₀   │  X=0   │   X=1   │   Z    │
├──────────┼────────┼─────────┼────────┤
│   0  0   │  0 0   │   0 1   │   0    │
│   0  1   │  0 0   │   1 0   │   0    │
│   1  0   │  0 0   │   1 0   │   1    │
│   1  1   │  d d   │   d d   │   d    │ (unused)
└──────────┴────────┴─────────┴────────┘

Now ready for K-map minimization and circuit implementation!
```

---

## 7. State Assignment

### State Assignment Methods

```
┌─────────────────────────────────────────────────────────────────┐
│                  STATE ASSIGNMENT METHODS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. SEQUENTIAL/BINARY:                                          │
│     Assign states in counting order                            │
│     S0=00, S1=01, S2=10, S3=11                                 │
│     Simple but may not be optimal                              │
│                                                                 │
│  2. GRAY CODE:                                                  │
│     Adjacent states differ by one bit                          │
│     S0=00, S1=01, S2=11, S3=10                                 │
│     Reduces switching noise                                    │
│                                                                 │
│  3. ONE-HOT:                                                    │
│     One flip-flop per state                                    │
│     S0=0001, S1=0010, S2=0100, S3=1000                        │
│     Fast, simple logic, more flip-flops                        │
│                                                                 │
│  4. OUTPUT-BASED:                                               │
│     Encode states so outputs are state bits                    │
│     Reduces output logic                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Number of Flip-Flops Required

```
FLIP-FLOP CALCULATION:

Number of FF = ⌈log₂(N)⌉

Where N = number of states

Examples:
  2 states:  ⌈log₂(2)⌉ = 1 FF
  3 states:  ⌈log₂(3)⌉ = 2 FF
  4 states:  ⌈log₂(4)⌉ = 2 FF
  5 states:  ⌈log₂(5)⌉ = 3 FF
  8 states:  ⌈log₂(8)⌉ = 3 FF
  10 states: ⌈log₂(10)⌉ = 4 FF

ONE-HOT ENCODING:
  Number of FF = N (one per state)
  Trade-off: More FF but simpler next-state logic
```

### State Assignment Example

```
EXAMPLE: 3-state machine (S0, S1, S2)

BINARY ASSIGNMENT:
  S0 = 00
  S1 = 01
  S2 = 10
  (11 = unused)
  
  Need 2 flip-flops
  One unused state (needs handling)

ONE-HOT ASSIGNMENT:
  S0 = 001
  S1 = 010
  S2 = 100
  
  Need 3 flip-flops
  No unused states
  Simpler next-state logic
  
COMPARISON:
┌──────────┬───────────┬──────────────┐
│  State   │  Binary   │   One-Hot    │
├──────────┼───────────┼──────────────┤
│    S0    │    00     │     001      │
│    S1    │    01     │     010      │
│    S2    │    10     │     100      │
├──────────┼───────────┼──────────────┤
│   FFs    │    2      │      3       │
│  Unused  │    1      │      0       │
└──────────┴───────────┴──────────────┘
```

---

## 8. FSM Design Procedure

### Step-by-Step Design Process

```
┌─────────────────────────────────────────────────────────────────┐
│                   FSM DESIGN PROCEDURE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  STEP 1: UNDERSTAND THE PROBLEM                                 │
│    - Define inputs and outputs                                 │
│    - Identify states needed                                    │
│    - Choose Moore or Mealy model                               │
│                                                                 │
│  STEP 2: DRAW STATE DIAGRAM                                     │
│    - Create states                                             │
│    - Define transitions for all input combinations            │
│    - Assign outputs                                            │
│                                                                 │
│  STEP 3: CREATE STATE TABLE                                     │
│    - List all states                                           │
│    - Fill next state for each input                           │
│    - Fill output values                                        │
│                                                                 │
│  STEP 4: STATE ASSIGNMENT                                       │
│    - Assign binary codes to states                             │
│    - Calculate number of flip-flops needed                     │
│                                                                 │
│  STEP 5: CREATE TRANSITION TABLE                                │
│    - Convert state names to binary codes                       │
│    - Include FF input requirements                             │
│                                                                 │
│  STEP 6: DERIVE BOOLEAN EQUATIONS                               │
│    - Use K-maps for minimization                               │
│    - Get equations for next state (FF inputs)                  │
│    - Get equations for outputs                                 │
│                                                                 │
│  STEP 7: DRAW CIRCUIT                                           │
│    - Implement using gates and flip-flops                      │
│                                                                 │
│  STEP 8: VERIFY                                                 │
│    - Check state transitions                                   │
│    - Verify outputs                                            │
│    - Handle unused states                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9. Complete Design Examples

### Example 1: Sequence Detector (Moore)

```
┌─────────────────────────────────────────────────────────────────┐
│    DESIGN: Sequence Detector for "101" (Overlapping)           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Problem: Detect input sequence 1-0-1                          │
│           Output Z=1 when sequence detected                    │
│           Overlapping: 10101 produces two detections           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

STEP 1: Define states
  S0: Initial state (no pattern match)
  S1: Received "1"
  S2: Received "10"
  S3: Received "101" (output = 1)

STEP 2: State Diagram (Moore)

                    ┌─────────────────────────────────┐
                    │                                 │
                    │ 0                               │
              ┌─────┴───┐                             │
              │         │                             │
           ──►│   S0    │◄────────────────────────────┼────┐
              │  Z = 0  │                             │    │
              └────┬────┘                             │    │
                   │ 1                                │    │
                   ▼                                  │    │
              ┌─────────┐                             │    │
              │         │─────┐                       │    │
              │   S1    │     │ 1 (self-loop)        │    │
              │  Z = 0  │◄────┘                       │    │
              └────┬────┘                             │    │
                   │ 0                                │    │
                   ▼                                  │    │
              ┌─────────┐                             │    │
              │         │                             │    │
              │   S2    │─────────────────────────────┘    │
              │  Z = 0  │  0                               │
              └────┬────┘                                  │
                   │ 1                                     │
                   ▼                                       │
              ┌─────────┐                                  │
              │         │                                  │
              │   S3    │──────────────────────────────────┘
              │  Z = 1  │  0
              └────┬────┘
                   │ 1
                   │
                   └────► goes to S1 (overlap: "1" starts new)

STEP 3: State Table (Moore)

┌───────────────┬────────────────────┬──────────┐
│ Present State │     Next State     │  Output  │
│               │   X=0   │   X=1    │    Z     │
├───────────────┼─────────┼──────────┼──────────┤
│      S0       │   S0    │    S1    │    0     │
│      S1       │   S2    │    S1    │    0     │
│      S2       │   S0    │    S3    │    0     │
│      S3       │   S0    │    S1    │    1     │
└───────────────┴─────────┴──────────┴──────────┘

STEP 4: State Assignment

Using binary: S0=00, S1=01, S2=10, S3=11
Need 2 flip-flops: Q₁, Q₀

STEP 5: Transition Table

┌──────────┬────────────────────┬──────────┐
│ Present  │     Next State     │  Output  │
│  Q₁ Q₀   │  X=0   │    X=1    │    Z     │
├──────────┼────────┼───────────┼──────────┤
│   0  0   │  0 0   │   0 1     │    0     │
│   0  1   │  1 0   │   0 1     │    0     │
│   1  0   │  0 0   │   1 1     │    0     │
│   1  1   │  0 0   │   0 1     │    1     │
└──────────┴────────┴───────────┴──────────┘

Next state: Q₁⁺ Q₀⁺

STEP 6: K-Maps (using D flip-flops, so D = Q⁺)

K-MAP FOR D₁ (Q₁⁺):
              X
         ┌────┬────┐
         │ 0  │ 1  │
    ┌────┼────┼────┤
    │ 00 │ 0  │ 0  │
Q₁Q₀├────┼────┼────┤
    │ 01 │ 1  │ 0  │
    ├────┼────┼────┤
    │ 11 │ 0  │ 0  │
    ├────┼────┼────┤
    │ 10 │ 0  │ 1  │
    └────┴────┴────┘

D₁ = Q₁'Q₀X' + Q₁Q₀'X = Q₀X' + Q₁Q₀'X  (simplified)

Actually let me redo:
  Q₁Q₀=00, X=0: D₁=0
  Q₁Q₀=00, X=1: D₁=0
  Q₁Q₀=01, X=0: D₁=1  ← group
  Q₁Q₀=01, X=1: D₁=0
  Q₁Q₀=10, X=0: D₁=0
  Q₁Q₀=10, X=1: D₁=1  ← group
  Q₁Q₀=11, X=0: D₁=0
  Q₁Q₀=11, X=1: D₁=0

D₁ = Q₁'Q₀X' + Q₁Q₀'X

K-MAP FOR D₀ (Q₀⁺):
              X
         ┌────┬────┐
         │ 0  │ 1  │
    ┌────┼────┼────┤
    │ 00 │ 0  │ 1  │
Q₁Q₀├────┼────┼────┤
    │ 01 │ 0  │ 1  │
    ├────┼────┼────┤
    │ 11 │ 0  │ 1  │
    ├────┼────┼────┤
    │ 10 │ 0  │ 1  │
    └────┴────┴────┘

D₀ = X  (all 1s in X=1 column!)

K-MAP FOR Z:
         ┌────┐
         │ X  │  (output doesn't depend on X for Moore)
    ┌────┼────┤
    │ 00 │ 0  │
Q₁Q₀├────┼────┤
    │ 01 │ 0  │
    ├────┼────┤
    │ 11 │ 1  │
    ├────┼────┤
    │ 10 │ 0  │
    └────┴────┘

Z = Q₁Q₀

STEP 7: Final Equations

D₁ = Q₁'Q₀X' + Q₁Q₀'X
D₀ = X
Z = Q₁Q₀

STEP 8: Circuit

         ┌───────────────────────────────────────────────────────────┐
         │                                                           │
    X ───┼───────────────────────────────────────────┬───────────┐  │
         │                                           │           │  │
         │   ┌───────────────────────────────────────│───────────│──┘
         │   │                                       │           │
         │   │  ┌───────────────────────────────┐    │           │
         │   │  │  ┌──────────────────────────┐ │    │           │
         │   │  │  │  ┌─────────────────────┐ │ │    │           │
         │   ▼  ▼  ▼  ▼                     │ │ │    │           │
         │  ┌─────────────┐                 │ │ │    ▼           │
         │  │   Q₁'Q₀X'   │──┐              │ │ │  ┌─────┐       │
         │  │    +        │  │──────────────│─│─│──┤D   Q├───────┼──Q₁
         │  │   Q₁Q₀'X    │──┘              │ │ │  │    Q'├──────┼──Q₁'
         │  └─────────────┘                 │ │ │  │>    │       │
         │                                  │ │ │  └─────┘       │
         │                                  │ │ │                │
         │  ┌─────┐                         │ │ │                │
    X ───┼──┤D   Q├─────────────────────────┼─┘ │                │
         │  │    Q'├────────────────────────┘   │                │
         │  │>    │                             └────────────────┘
         │  └─────┘                                Q₀  Q₀'
         │
CLK ─────┼──────────────────────────────────────────────────────────

Z = Q₁ · Q₀
```

### Example 2: Even Parity Checker (Mealy)

```
┌─────────────────────────────────────────────────────────────────┐
│         DESIGN: Even Parity Checker (Mealy Machine)            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Problem: Track parity of input bits                           │
│           Output Z=1 when even number of 1s received           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

STEP 1: Define states
  EVEN: Even number of 1s seen
  ODD: Odd number of 1s seen

STEP 2: State Diagram (Mealy)

              ┌────────────────────────────────────┐
              │                                    │
              │  0/1 (even parity maintained)     │
              ▼  │                                 │
           ┌─────┴───┐          1/0           ┌─────────┐
           │         │────────────────────────►         │
        ──►│  EVEN   │                        │   ODD   │
           │         │◄────────────────────────│         │
           └────┬────┘          1/1           └────┬────┘
                                                   │
                                              0/0  │
                                               ◄───┘

  From EVEN:
    Input 0 → stay EVEN, output 1 (still even parity)
    Input 1 → go to ODD, output 0 (now odd parity)
  
  From ODD:
    Input 0 → stay ODD, output 0 (still odd parity)
    Input 1 → go to EVEN, output 1 (back to even)

STEP 3: State Table (Mealy)

┌───────────┬────────────────────┬────────────────────┐
│  Present  │     Next State     │       Output       │
│   State   │   X=0   │   X=1    │   X=0   │   X=1    │
├───────────┼─────────┼──────────┼─────────┼──────────┤
│   EVEN    │  EVEN   │   ODD    │    1    │    0     │
│   ODD     │  ODD    │   EVEN   │    0    │    1     │
└───────────┴─────────┴──────────┴─────────┴──────────┘

STEP 4: State Assignment

EVEN = 0, ODD = 1
Need 1 flip-flop: Q

STEP 5: Transition Table

┌─────────┬────────────────────┬────────────────────┐
│ Present │     Next State     │       Output       │
│    Q    │   X=0   │   X=1    │   X=0   │   X=1    │
├─────────┼─────────┼──────────┼─────────┼──────────┤
│    0    │    0    │    1     │    1    │    0     │
│    1    │    1    │    0     │    0    │    1     │
└─────────┴─────────┴──────────┴─────────┴──────────┘

STEP 6: Boolean Equations

For D flip-flop (D = Q⁺):

D = Q'X + QX' = Q ⊕ X  (XOR!)

Z = Q'X' + QX = (Q ⊕ X)' = Q XNOR X

Or simply: Z = Q' when X=0, Z = Q when X=1
          Z = Q ⊕ X ⊕ 1 = (Q ⊕ X)'

STEP 7: Circuit

              ┌────────────────────────────────────┐
              │                                    │
              │      ┌────────────────────────┐    │
    X ────────┼──────┤                        │    │
              │      │                        │    │
              │      │    ┌───┐               │    │
              │      ├────┤XOR├───────────────┼────┼───── Z
              │      │    └───┘               │    │
              │      │      ▲                 │    │
              │      │      │                 │    │
              │    ┌─┴───┐  │                 │    │
              │    │ XOR ├──┴───────┐         │    │
              │    └─┬───┘          │         │    │
              │      ▲              ▼         │    │
              │      │           ┌─────┐      │    │
              │      └───────────┤D   Q├──────┘    │
              │                  │    Q'│          │
              │                  │>    │           │
              │                  └─────┘           │
              │                                    │
    CLK ──────┼────────────────────────────────────┘
              │                                    
              └────────────────────────────────────

Very simple: Just 2 XOR gates and 1 D flip-flop!

Q⁺ = Q ⊕ X (toggles state on input 1)
Z = (Q ⊕ X)' or Q XNOR X (even when equal)
```

---

## 10. State Reduction

### Equivalent States

```
┌─────────────────────────────────────────────────────────────────┐
│                    STATE REDUCTION                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  TWO STATES ARE EQUIVALENT IF:                                 │
│    1. They have the same output (Moore)                        │
│       OR same output for same inputs (Mealy)                   │
│    2. They have equivalent next states for all inputs          │
│                                                                 │
│  GOAL: Reduce number of states to minimize flip-flops          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

EQUIVALENCE EXAMPLE:

Original state table:
┌───────────┬────────────────────┬──────────┐
│  Present  │     Next State     │  Output  │
│   State   │   X=0   │   X=1    │    Z     │
├───────────┼─────────┼──────────┼──────────┤
│    S0     │   S1    │    S2    │    0     │
│    S1     │   S3    │    S4    │    0     │
│    S2     │   S4    │    S3    │    0     │
│    S3     │   S5    │    S0    │    1     │
│    S4     │   S0    │    S5    │    1     │
│    S5     │   S0    │    S0    │    1     │
└───────────┴─────────┴──────────┴──────────┘

Check S3 and S4:
  - Both have output = 1 ✓
  - S3: next = S5, S0
  - S4: next = S0, S5
  - Swapped but if S5 ≡ S0... let's check

Check S5 vs others with output=1:
  - S5 next = S0, S0
  - S3 next = S5, S0
  - S4 next = S0, S5

If we continue analysis, we may find equivalent states
that can be merged to reduce total states.
```

### Implication Chart Method

```
IMPLICATION CHART METHOD:

Step 1: Create chart with all state pairs
Step 2: Mark pairs with different outputs as ✗
Step 3: For remaining pairs, list implied equivalences
Step 4: Iterate: if implied pair is ✗, mark current as ✗
Step 5: Remaining unmarked pairs are equivalent

EXAMPLE CHART:

        S0    S1    S2    S3    S4
    ┌─────┐
 S1 │     │
    ├─────┼─────┐
 S2 │     │     │
    ├─────┼─────┼─────┐
 S3 │  ✗  │  ✗  │  ✗  │          (different output)
    ├─────┼─────┼─────┼─────┐
 S4 │  ✗  │  ✗  │  ✗  │     │    (different output)
    ├─────┼─────┼─────┼─────┼─────┐
 S5 │  ✗  │  ✗  │  ✗  │     │     │ (different output)
    └─────┴─────┴─────┴─────┴─────┘

For unmarked pairs (S0-S1), (S0-S2), (S1-S2), (S3-S4), (S3-S5), (S4-S5):
  Check if their next states are equivalent...
  (Detailed analysis required for each pair)
```

---

## 11. Summary Tables

### FSM Design Quick Reference

| Step | Action | Output |
|------|--------|--------|
| 1 | Understand problem | Inputs, outputs, states |
| 2 | Draw state diagram | Visual representation |
| 3 | Create state table | Tabular form |
| 4 | State assignment | Binary codes |
| 5 | Transition table | With binary codes |
| 6 | K-map minimization | Boolean equations |
| 7 | Circuit implementation | Logic diagram |
| 8 | Verification | Testing |

### Moore vs Mealy Summary

| Aspect | Moore | Mealy |
|--------|-------|-------|
| Output depends on | State only | State + Input |
| Output timing | Synchronous | Can be asynchronous |
| Number of states | Usually more | Usually fewer |
| Output stability | Stable, glitch-free | May have glitches |
| Response | Delayed by 1 clock | Immediate |
| Diagram notation | Output in state | Output on transition |
| Output logic inputs | State bits only | State + external |

### Flip-Flop Excitation Tables (for FSM)

| Present Q | Next Q | D | T | J | K |
|-----------|--------|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 | X |
| 0 | 1 | 1 | 1 | 1 | X |
| 1 | 0 | 0 | 1 | X | 1 |
| 1 | 1 | 1 | 0 | X | 0 |

### State Assignment Comparison

| Method | FFs Needed | Logic Complexity | Speed |
|--------|------------|------------------|-------|
| Binary | ⌈log₂N⌉ | Medium | Medium |
| Gray | ⌈log₂N⌉ | Medium | Medium |
| One-Hot | N | Simple | Fast |
| Output-based | Varies | Simple output | Varies |

---

## 12. Quick Revision Questions

### Question 1
What is the difference between Moore and Mealy machines in terms of output dependency?

<details>
<summary>Click for Answer</summary>

```
MOORE MACHINE:
  Output = f(Present State ONLY)
  - Output depends solely on current state
  - Output written inside state circles
  - Output changes only at clock edges (with state)
  - More stable, no glitches
  
  Example: In state S1 with Z=1, output is 1 regardless of input

MEALY MACHINE:
  Output = f(Present State, Inputs)
  - Output depends on both state and current inputs
  - Output written on transition arrows
  - Output can change asynchronously with input
  - Can have glitches if inputs change
  
  Example: In state S1 with input X=0, output might be 0
           In state S1 with input X=1, output might be 1

Key difference: Mealy outputs respond immediately to inputs;
                Moore outputs wait for next clock.
```
</details>

### Question 2
How many flip-flops are needed for a 5-state FSM using binary encoding?

<details>
<summary>Click for Answer</summary>

```
NUMBER OF FLIP-FLOPS:

Formula: n = ⌈log₂(N)⌉

Where N = number of states = 5

n = ⌈log₂(5)⌉
n = ⌈2.32⌉
n = 3 flip-flops

With 3 flip-flops:
  - Can represent 2³ = 8 states
  - 5 states used, 3 states unused
  
State assignment example:
  S0 = 000
  S1 = 001
  S2 = 010
  S3 = 011
  S4 = 100
  (101, 110, 111 = unused)

Important: Handle unused states in design
  - Either force to valid state
  - Or treat as don't cares for optimization
```
</details>

### Question 3
Design a Moore machine state diagram for a "110" sequence detector.

<details>
<summary>Click for Answer</summary>

```
"110" SEQUENCE DETECTOR (Moore, Non-overlapping):

States needed:
  S0: Initial/Reset (no match)
  S1: Detected "1"
  S2: Detected "11"
  S3: Detected "110" (output = 1)

STATE DIAGRAM:

                 0
            ┌─────────┐
            │         │
            ▼         │
    ───►┌───────┐     │         0
        │  S0   │◄────┼─────────────────────┐
        │ Z = 0 │     │                     │
        └───┬───┘     │                     │
            │ 1       │                     │
            ▼         │                     │
        ┌───────┐     │                     │
        │  S1   │─────┘                     │
        │ Z = 0 │     0                     │
        └───┬───┘                           │
            │ 1                             │
            ▼                               │
        ┌───────┐ ─────────────┐            │
        │  S2   │      1       │            │
        │ Z = 0 │◄─────────────┘            │
        └───┬───┘                           │
            │ 0                             │
            ▼                               │
        ┌───────┐                           │
        │  S3   │───────────────────────────┘
        │ Z = 1 │
        └───┬───┘
            │ 1
            │
            └───────► goes to S1 (if overlapping allowed)
                 or to S0 (if non-overlapping)

TRANSITIONS:
  S0: 0→S0, 1→S1
  S1: 0→S0, 1→S2
  S2: 0→S3, 1→S2
  S3: 0→S0, 1→S1 (or S0 for non-overlap)
```
</details>

### Question 4
What are the advantages of one-hot encoding in FSM design?

<details>
<summary>Click for Answer</summary>

```
ONE-HOT ENCODING ADVANTAGES:

1. SIMPLE NEXT-STATE LOGIC:
   - Each state has its own flip-flop
   - Next-state equations often reduce to simple OR/AND
   - No complex decoding needed
   
2. FAST OPERATION:
   - Minimal logic levels between FF and FF
   - Reduced propagation delay
   - Higher clock speeds possible
   
3. EASY DEBUGGING:
   - State visible directly on FF outputs
   - Each state = one FF high, rest low
   - Easy to probe and observe
   
4. NO ILLEGAL STATES:
   - All patterns have meaning (though some invalid)
   - Easy to detect if multiple FFs high
   
5. GLITCH-FREE OUTPUTS:
   - State transitions are clean
   - Only one FF changes at a time
   
DISADVANTAGES:
   - More flip-flops needed (N vs log₂N)
   - More routing/wiring
   - Invalid if multiple FFs high or all low

COMPARISON (10 states):
  Binary: ⌈log₂(10)⌉ = 4 flip-flops
  One-Hot: 10 flip-flops
  
  Trade-off: More FFs but simpler logic, faster speed
  Preferred in FPGA implementations!
```
</details>

### Question 5
Explain the concept of state reduction and why it's important.

<details>
<summary>Click for Answer</summary>

```
STATE REDUCTION:

CONCEPT:
  Identifying and merging equivalent states to 
  minimize the total number of states in an FSM.

TWO STATES ARE EQUIVALENT IF:
  1. They produce the same output (for all inputs)
  2. Their next states are equivalent (for all inputs)

IMPORTANCE:

1. FEWER FLIP-FLOPS:
   - Fewer states = fewer FFs needed
   - log₂(8) = 3 FFs vs log₂(4) = 2 FFs
   - Reduces hardware cost
   
2. SIMPLER LOGIC:
   - Fewer states = simpler next-state logic
   - Smaller K-maps
   - Fewer gates
   
3. LOWER POWER:
   - Fewer FFs switching
   - Less dynamic power consumption
   
4. FASTER TIMING:
   - Simpler logic = shorter paths
   - Higher clock speeds

EXAMPLE:
  Original: 6 states → 3 flip-flops
  Reduced:  4 states → 2 flip-flops
  Savings: 1 flip-flop + simplified logic

METHODS:
  - Row matching (simple)
  - Implication chart (systematic)
  - Partition-based algorithms
```
</details>

### Question 6
Given a state table, show how to derive the next-state equations using D flip-flops.

<details>
<summary>Click for Answer</summary>

```
DERIVING NEXT-STATE EQUATIONS:

Given state table:
┌───────────┬──────────────────┬────────┐
│  Present  │   Next State     │ Output │
│   State   │  X=0  │   X=1    │   Z    │
├───────────┼───────┼──────────┼────────┤
│    A      │   A   │    B     │   0    │
│    B      │   A   │    C     │   0    │
│    C      │   A   │    C     │   1    │
└───────────┴───────┴──────────┴────────┘

STEP 1: State Assignment
  A = 00, B = 01, C = 10
  Need 2 D flip-flops: Q₁, Q₀

STEP 2: Transition Table
┌────────┬──────────────────┬────────┐
│  Q₁Q₀  │  Next (Q₁⁺Q₀⁺)  │   Z    │
│        │  X=0  │   X=1    │        │
├────────┼───────┼──────────┼────────┤
│   00   │  00   │   01     │   0    │
│   01   │  00   │   10     │   0    │
│   10   │  00   │   10     │   1    │
│   11   │  dd   │   dd     │   d    │
└────────┴───────┴──────────┴────────┘

STEP 3: K-Maps

For D₁ (= Q₁⁺):        For D₀ (= Q₀⁺):
     X=0  X=1               X=0  X=1
    ┌────┬────┐            ┌────┬────┐
 00 │ 0  │ 0  │         00 │ 0  │ 1  │
    ├────┼────┤            ├────┼────┤
 01 │ 0  │ 1  │         01 │ 0  │ 0  │
    ├────┼────┤            ├────┼────┤
 11 │ d  │ d  │         11 │ d  │ d  │
    ├────┼────┤            ├────┼────┤
 10 │ 0  │ 1  │         10 │ 0  │ 0  │
    └────┴────┘            └────┴────┘

D₁ = Q₁X + Q₀X        D₀ = Q₁'Q₀'X

For Z:
    ┌────┐
 00 │ 0  │
    ├────┤
 01 │ 0  │
    ├────┤
 11 │ d  │
    ├────┤
 10 │ 1  │
    └────┘

Z = Q₁Q₀' = Q₁ (since Q₀'=1 when Q₁=1 in valid states)

FINAL EQUATIONS:
  D₁ = X(Q₁ + Q₀)
  D₀ = Q₁'Q₀'X
  Z = Q₁
```
</details>

---

## Navigation

| Previous | [Home](README.md) | Next |
|:--------:|:-----------------:|:----:|
| [Unit 7: Registers](Unit-07-Registers.md) | Return to Index | [Unit 9: Memory & Programmable Logic](Unit-09-Memory-and-Programmable-Logic.md) |

---

*End of Unit 8*
