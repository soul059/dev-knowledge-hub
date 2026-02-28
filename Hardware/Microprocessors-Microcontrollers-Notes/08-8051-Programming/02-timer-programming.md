# Chapter 8.2: 8051 Timer Programming

## 📚 Chapter Overview

This chapter covers the 8051 Timer/Counter system in detail, including all timer modes, programming techniques for delays and event counting, and practical calculation methods.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Understand Timer 0 and Timer 1 architecture
- Program timers in all modes (Mode 0, 1, 2, 3)
- Calculate timer values for specific delays
- Implement both delay generation and event counting

---

## 1. Timer/Counter Architecture

### 1.1 Timer Overview

```
8051 TIMER/COUNTER SYSTEM
━━━━━━━━━━━━━━━━━━━━━━━━━

┌────────────────────────────────────────────────────────────────┐
│                                                                │
│   Timer 0                          Timer 1                     │
│   ┌──────────────┐                 ┌──────────────┐           │
│   │   TH0 (8CH)  │                 │   TH1 (8DH)  │           │
│   └──────────────┘                 └──────────────┘           │
│   ┌──────────────┐                 ┌──────────────┐           │
│   │   TL0 (8AH)  │                 │   TL1 (8BH)  │           │
│   └──────────────┘                 └──────────────┘           │
│                                                                │
│   Control: TMOD (89H) - Mode selection                        │
│            TCON (88H) - Timer control and flags               │
│                                                                │
│   Clock Source Options:                                        │
│   • Internal: Crystal ÷ 12 (Machine Cycle)                    │
│   • External: T0 (P3.4) or T1 (P3.5) pins                    │
│                                                                │
└────────────────────────────────────────────────────────────────┘

KEY FEATURES:
• Two 16-bit timers (Timer 0 and Timer 1)
• Four operating modes each
• Can function as timers (internal clock) or counters (external pulses)
• Overflow flag and interrupt capability
• Timer 1 can generate baud rate for serial port
```

### 1.2 TMOD Register

```
TMOD - TIMER MODE REGISTER (Address 89H)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│GATE │ C/T̅ │ M1  │ M0  │GATE │ C/T̅ │ M1  │ M0  │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
│◄────── Timer 1 ──────►│◄────── Timer 0 ──────►│
   D7    D6    D5    D4    D3    D2    D1    D0

BIT FUNCTIONS:
──────────────

GATE: Gating Control
  0 = Timer runs when TR = 1
  1 = Timer runs when TR = 1 AND INTx = HIGH

C/T̅: Counter/Timer Select
  0 = Timer mode (internal clock)
  1 = Counter mode (external pulses on T0/T1)

M1 M0: Mode Select
  0  0 = Mode 0: 13-bit timer
  0  1 = Mode 1: 16-bit timer
  1  0 = Mode 2: 8-bit auto-reload
  1  1 = Mode 3: Split timer (Timer 0 only)

COMMON TMOD VALUES:
──────────────────

┌────────┬────────────────────────────────────────────────────┐
│ Value  │ Configuration                                      │
├────────┼────────────────────────────────────────────────────┤
│  01H   │ Timer 0 Mode 1 (16-bit), Timer 1 unused           │
│  10H   │ Timer 1 Mode 1 (16-bit), Timer 0 unused           │
│  11H   │ Both timers in Mode 1                              │
│  02H   │ Timer 0 Mode 2 (8-bit auto-reload)                │
│  20H   │ Timer 1 Mode 2 (for baud rate generation)         │
│  21H   │ Timer 0 Mode 1, Timer 1 Mode 2                    │
│  05H   │ Timer 0 Mode 1 Counter (external pulses)          │
└────────┴────────────────────────────────────────────────────┘
```

### 1.3 TCON Register

```
TCON - TIMER CONTROL REGISTER (Address 88H)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│ TF1 │ TR1 │ TF0 │ TR0 │ IE1 │ IT1 │ IE0 │ IT0 │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
  D7    D6    D5    D4    D3    D2    D1    D0
  8FH   8EH   8DH   8CH   8BH   8AH   89H   88H  ◄── Bit addresses

TIMER BITS:
───────────
TF1 (8FH): Timer 1 Overflow Flag
  Set by hardware when Timer 1 overflows (FFFFH → 0000H)
  Cleared by hardware when interrupt serviced, or by software

TR1 (8EH): Timer 1 Run Control
  1 = Start Timer 1
  0 = Stop Timer 1

TF0 (8DH): Timer 0 Overflow Flag
  Set by hardware when Timer 0 overflows
  Cleared by hardware or software

TR0 (8CH): Timer 0 Run Control
  1 = Start Timer 0
  0 = Stop Timer 0

INTERRUPT BITS:
───────────────
IE1 (8BH): External Interrupt 1 Flag
IT1 (8AH): External Interrupt 1 Type (0=level, 1=edge)
IE0 (89H): External Interrupt 0 Flag  
IT0 (88H): External Interrupt 0 Type (0=level, 1=edge)
```

---

## 2. Timer Modes

### 2.1 Mode 0: 13-bit Timer

```
MODE 0: 13-BIT TIMER/COUNTER
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Legacy mode (8048 compatible), rarely used in new designs.

Structure:
┌────────────────────────┬───────────────────┐
│        TH0/TH1         │   TL0/TL1 [4:0]   │
│       (8 bits)         │    (5 bits)       │
└────────────────────────┴───────────────────┘
│◄───────────── 13 bits ──────────────────►│

Count Range: 0000H to 1FFFH (0 to 8191)
Overflow at: 1FFFH → 0000H (after 8192 counts)

Maximum Delay @ 12 MHz:
  8192 × 1 µs = 8.192 ms

Usage:
    MOV TMOD, #00H      ; Timer 0 Mode 0
    MOV TH0, #HIGH_VAL
    MOV TL0, #LOW_VAL   ; Only lower 5 bits used
    SETB TR0            ; Start
    JNB TF0, $          ; Wait
    CLR TR0
    CLR TF0
```

### 2.2 Mode 1: 16-bit Timer

```
MODE 1: 16-BIT TIMER/COUNTER
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Most commonly used mode for delay generation.

Structure:
┌────────────────────────┬────────────────────────┐
│        TH0/TH1         │        TL0/TL1         │
│       (8 bits)         │        (8 bits)        │
└────────────────────────┴────────────────────────┘
│◄───────────────── 16 bits ─────────────────────►│

Count Range: 0000H to FFFFH (0 to 65535)
Overflow at: FFFFH → 0000H (after 65536 counts)

Maximum Delay @ 12 MHz:
  65536 × 1 µs = 65.536 ms ≈ 65.5 ms

BLOCK DIAGRAM:
─────────────

                         Machine Cycle Clock
                              (Crystal ÷ 12)
                                    │
    C/T̅ = 0 ─────────────────────►│ │
                                  │MUX├───┐
    C/T̅ = 1 ── Tx Pin (P3.4/5) ─►│ │   │
                                    │     │
                                    ▼     ▼
             ┌─────────────────────────────────┐
    GATE ───►│          Control Logic          │
             │                                 │
    INTx ───►│    TR = 1 AND (GATE=0 OR INTx) │
             └───────────────┬─────────────────┘
                             │ Enable
                             ▼
                    ┌────────────────┐
                    │   16-bit       │
                    │   Counter      │
                    │  (TH : TL)     │
                    └───────┬────────┘
                            │ Overflow
                            ▼
                       ┌─────────┐
                       │   TF    │────► Interrupt
                       └─────────┘

PROGRAMMING EXAMPLE:
───────────────────

    ; 50 ms delay using Mode 1 (@ 12 MHz crystal)
    ; Count needed = 65536 - 50000 = 15536 = 3CB0H
    
    MOV TMOD, #01H      ; Timer 0 Mode 1
    MOV TH0, #3CH       ; High byte
    MOV TL0, #0B0H      ; Low byte
    SETB TR0            ; Start timer
    JNB TF0, $          ; Wait for overflow
    CLR TR0             ; Stop timer
    CLR TF0             ; Clear flag
```

### 2.3 Mode 2: 8-bit Auto-Reload

```
MODE 2: 8-BIT AUTO-RELOAD
━━━━━━━━━━━━━━━━━━━━━━━━━

TL counts, TH holds reload value. Auto-reloads on overflow.

Structure:
┌────────────────────────┐   ┌────────────────────────┐
│   TH (Reload Value)    │   │   TL (Counter)         │
│       (8 bits)         │──►│       (8 bits)         │
└────────────────────────┘   └────────────────────────┘
                               ↓ Auto-reload on overflow

Count Range: 00H to FFH (0 to 255)
Overflow at: FFH → TH value

Maximum Delay @ 12 MHz:
  256 × 1 µs = 256 µs (single overflow)

ADVANTAGES:
• No need to reload timer value in software
• Consistent timing (no reload delay)
• Ideal for baud rate generation

BLOCK DIAGRAM:
─────────────

    Clock Source ───►┌─────────────────┐
                     │   8-bit TL      │
                     │   Counter       │──► Overflow ──► TF
                     └────────┬────────┘
                              │
                              ▼ (on overflow)
                     ┌─────────────────┐
                     │   8-bit TH      │
                     │   Reload Value  │
                     └─────────────────┘

PROGRAMMING EXAMPLE:
───────────────────

    ; Generate continuous 200 µs intervals
    ; Count = 256 - 200 = 56 = 38H
    
    MOV TMOD, #02H      ; Timer 0 Mode 2
    MOV TH0, #38H       ; Reload value
    MOV TL0, #38H       ; Initial count
    SETB TR0            ; Start timer
    
WAIT_OVF:
    JNB TF0, WAIT_OVF   ; Wait for overflow
    CLR TF0             ; Clear flag
    ; TL0 automatically reloaded with TH0!
    SJMP WAIT_OVF

    ; Baud rate generation (9600 @ 11.0592 MHz)
    ; TH1 = 256 - (11059200 / (12 × 32 × 9600)) = 256 - 3 = 253 = FDH
    
    MOV TMOD, #20H      ; Timer 1 Mode 2
    MOV TH1, #0FDH      ; -3 for 9600 baud
    MOV TL1, #0FDH
    SETB TR1            ; Start for baud rate
```

### 2.4 Mode 3: Split Timer

```
MODE 3: SPLIT TIMER (Timer 0 Only)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Timer 0 splits into two independent 8-bit timers.
Timer 1 stops in Mode 3 (but can run in Modes 0, 1, 2).

Structure:
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│   TL0 (8-bit timer)                TH0 (8-bit timer)          │
│   ┌──────────────┐                 ┌──────────────┐           │
│   │    TL0       │                 │    TH0       │           │
│   └──────────────┘                 └──────────────┘           │
│         │                                │                     │
│         ▼                                ▼                     │
│   Control: TR0, TF0                Control: TR1, TF1          │
│   Clock: Machine cycle             Clock: Machine cycle only  │
│   Gate: GATE, INT0                 Gate: None                  │
│                                                                │
└────────────────────────────────────────────────────────────────┘

Note: TH0 uses TR1 and TF1 control bits!
      Timer 1 has no run control when Timer 0 is in Mode 3.

Use Case:
• Need more than two timers
• Need Timer 1 for baud rate AND two other timing functions
• Timer 1 runs free in Mode 2 for baud, Timer 0 Mode 3 for two more timers

PROGRAMMING EXAMPLE:
───────────────────

    MOV TMOD, #23H      ; Timer 0 Mode 3, Timer 1 Mode 2
    
    ; Timer 1 for baud rate (runs free)
    MOV TH1, #0FDH      ; 9600 baud
    SETB TR1            ; This now controls TH0!
    
    ; TL0 as independent 8-bit timer
    MOV TL0, #06H
    SETB TR0            ; Controls TL0
    
    ; TH0 as another 8-bit timer
    MOV TH0, #56H
    ; TR1 already set, controls TH0
```

---

## 3. Timer Calculations

### 3.1 Delay Calculation Formulas

```
TIMER DELAY CALCULATIONS
━━━━━━━━━━━━━━━━━━━━━━━━

Basic Formula:
─────────────
Machine Cycle Time = 12 / Crystal Frequency
                   = 12 / 12 MHz = 1 µs

For Mode 1 (16-bit):
───────────────────
Delay = (65536 - Initial Value) × Machine Cycle Time

Initial Value = 65536 - (Delay / Machine Cycle Time)
              = 65536 - (Delay × Crystal / 12)

For Mode 2 (8-bit):
──────────────────
Delay per overflow = (256 - TH value) × Machine Cycle Time

TH value = 256 - (Delay / Machine Cycle Time)

EXAMPLE CALCULATIONS @ 12 MHz (1 µs machine cycle):
─────────────────────────────────────────────────────

1. Generate 1 ms delay (Mode 1):
   Initial = 65536 - 1000 = 64536 = FC18H
   TH0 = FCH, TL0 = 18H

2. Generate 10 ms delay (Mode 1):
   Initial = 65536 - 10000 = 55536 = D8F0H
   TH0 = D8H, TL0 = F0H

3. Generate 50 ms delay (Mode 1):
   Initial = 65536 - 50000 = 15536 = 3CB0H
   TH0 = 3CH, TL0 = B0H

4. Maximum Mode 1 delay:
   Max = 65536 × 1 µs = 65.536 ms

5. Generate 100 µs (Mode 2):
   TH0 = 256 - 100 = 156 = 9CH
```

### 3.2 Calculation for Different Crystals

```
COMMON CRYSTAL CALCULATIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌──────────────┬───────────────┬────────────────────────────────┐
│ Crystal      │ Machine Cycle │ Mode 1 Max Delay               │
├──────────────┼───────────────┼────────────────────────────────┤
│ 12.000 MHz   │    1.000 µs   │ 65.536 ms                      │
│ 11.0592 MHz  │    1.085 µs   │ 71.11 ms                       │
│ 24.000 MHz   │    0.500 µs   │ 32.768 ms                      │
│ 6.000 MHz    │    2.000 µs   │ 131.07 ms                      │
└──────────────┴───────────────┴────────────────────────────────┘

FORMULA FOR ANY CRYSTAL:

For delay T (in seconds):
    Count = T / (12 / Crystal)
          = T × Crystal / 12

Initial Value = 65536 - Count

EXAMPLE: 5 ms delay @ 11.0592 MHz
─────────────────────────────────
Machine Cycle = 12 / 11.0592 MHz = 1.085 µs

Count needed = 5 ms / 1.085 µs = 4608 counts

Initial = 65536 - 4608 = 60928 = EE00H
TH0 = EEH, TL0 = 00H

EXAMPLE: 1 second delay (multiple overflows)
───────────────────────────────────────────
@ 12 MHz, max delay = 65.536 ms

For 1 second: 1000 ms / 65.536 ms ≈ 15.26 overflows
Use 20 × 50ms = 1 second

    MOV R7, #20         ; 20 iterations
DELAY_1S:
    MOV TH0, #3CH       ; 50 ms
    MOV TL0, #0B0H
    SETB TR0
    JNB TF0, $
    CLR TR0
    CLR TF0
    DJNZ R7, DELAY_1S
```

---

## 4. Timer Programming Examples

### 4.1 Delay Subroutines

```
DELAY SUBROUTINES
━━━━━━━━━━━━━━━━━

; 1 ms delay @ 12 MHz
DELAY_1MS:
    MOV TMOD, #01H      ; Timer 0 Mode 1
    MOV TH0, #0FCH      ; 65536 - 1000 = 64536 = FC18H
    MOV TL0, #18H
    SETB TR0            ; Start timer
    JNB TF0, $          ; Wait for overflow
    CLR TR0             ; Stop timer
    CLR TF0             ; Clear flag
    RET

; Delay of R7 milliseconds
DELAY_MS:
    ; R7 = number of milliseconds
    MOV TMOD, #01H
DELAY_LOOP:
    MOV TH0, #0FCH
    MOV TL0, #18H
    SETB TR0
    JNB TF0, $
    CLR TR0
    CLR TF0
    DJNZ R7, DELAY_LOOP
    RET

; 1 second delay
DELAY_1S:
    MOV R6, #20         ; 20 × 50ms = 1 second
DELAY_50MS_LOOP:
    MOV TMOD, #01H
    MOV TH0, #3CH       ; 50 ms
    MOV TL0, #0B0H
    SETB TR0
    JNB TF0, $
    CLR TR0
    CLR TF0
    DJNZ R6, DELAY_50MS_LOOP
    RET

; Precise short delay using Mode 2
DELAY_100US:
    MOV TMOD, #02H      ; Mode 2 auto-reload
    MOV TH0, #9CH       ; 256 - 100 = 156
    MOV TL0, #9CH
    SETB TR0
    JNB TF0, $
    CLR TR0
    CLR TF0
    RET
```

### 4.2 Square Wave Generation

```
SQUARE WAVE GENERATION
━━━━━━━━━━━━━━━━━━━━━━

; Generate 1 kHz square wave on P1.0 (500 µs period each half)
; @ 12 MHz crystal

SQUARE_1KHZ:
    MOV TMOD, #02H      ; Timer 0 Mode 2 (auto-reload)
    MOV TH0, #06H       ; 256 - 250 = 6 (for 250 µs)
                        ; Actually need 500µs half period
                        ; TH0 = 256 - 500 = -244? Won't fit!
                        ; Use Mode 1 instead

; Corrected: Using Mode 1 for 1 kHz (500 µs half-period)
    MOV TMOD, #01H      ; Timer 0 Mode 1
AGAIN:
    MOV TH0, #0FEH      ; 65536 - 500 = 65036 = FE0CH
    MOV TL0, #0CH
    SETB TR0
    JNB TF0, $
    CLR TR0
    CLR TF0
    CPL P1.0            ; Toggle output
    SJMP AGAIN

; 10 kHz square wave (50 µs half-period)
SQUARE_10KHZ:
    MOV TMOD, #02H      ; Mode 2 auto-reload
    MOV TH0, #206       ; 256 - 50 = 206 = CEH
    MOV TL0, #206
    SETB TR0
TOGGLE:
    JNB TF0, $
    CLR TF0
    CPL P1.0
    SJMP TOGGLE
```

### 4.3 Event Counter Mode

```
EVENT COUNTER MODE
━━━━━━━━━━━━━━━━━━

Count external pulses on T0 (P3.4) or T1 (P3.5) pins.

; Count pulses on T0 pin
COUNTER_SETUP:
    MOV TMOD, #05H      ; Timer 0 Mode 1, Counter (C/T̅=1)
    MOV TH0, #0         ; Start from 0
    MOV TL0, #0
    SETB TR0            ; Start counting
    
    ; Read count
    MOV A, TL0
    MOV R0, A           ; Low byte of count
    MOV A, TH0
    MOV R1, A           ; High byte of count

; Count and display number of pulses
PULSE_COUNTER:
    MOV TMOD, #05H      ; Counter mode
    MOV TH0, #0
    MOV TL0, #0
    SETB TR0            ; Start counting

DISPLAY_COUNT:
    MOV A, TL0
    MOV P1, A           ; Display low byte on Port 1
    CALL DELAY_1S       ; Wait 1 second
    SJMP DISPLAY_COUNT  ; Update display

; Count specific number of events (e.g., 100 items)
; and generate alarm
COUNT_100:
    MOV TMOD, #05H      ; Counter mode
    MOV TH0, #0FFH      ; 65536 - 100 = 65436 = FF9CH
    MOV TL0, #9CH       ; Will overflow after 100 counts
    SETB TR0
    JNB TF0, $          ; Wait for 100 pulses
    CLR TR0
    CLR TF0
    SETB P2.0           ; Activate alarm
    RET
```

### 4.4 Gate Control Mode

```
GATE CONTROL FOR TIME MEASUREMENT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GATE = 1: Timer runs only when TR = 1 AND INTx pin = HIGH

Use Case: Measure pulse width on INT0 pin

                 ┌──────────────────────────┐
    INT0 pin ────┤         ┌────┐           │
                 │    ─────┘    └───────    │
                 │         │    │           │
                 │    Start│    │Stop       │
                 │    count│    │count      │
                 └──────────────────────────┘

; Measure pulse width on INT0 (P3.2)
MEASURE_PULSE:
    MOV TMOD, #09H      ; Timer 0 Mode 1, GATE=1
    MOV TH0, #0
    MOV TL0, #0
    SETB TR0            ; Enable (but won't run until INT0=HIGH)
    
    JB P3.2, $          ; Wait for INT0 to go LOW first
    JNB P3.2, $         ; Wait for INT0 to go HIGH (start timing)
    JB P3.2, $          ; Wait for INT0 to go LOW (stop timing)
    
    CLR TR0
    ; TH0:TL0 now contains pulse width in machine cycles
    
    MOV A, TL0
    MOV R0, A           ; Low byte
    MOV A, TH0
    MOV R1, A           ; High byte
    RET
```

---

## 5. Timer 2 (8052 Only)

```
TIMER 2 (8052/8032 ONLY)
━━━━━━━━━━━━━━━━━━━━━━━━

Additional 16-bit timer with enhanced features.

Features:
• 16-bit timer/counter
• Auto-reload capability
• Capture mode
• Baud rate generation for serial port
• Uses T2CON (C8H), TH2/TL2, RCAP2H/RCAP2L

T2CON Register (C8H):
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│ TF2 │ EXF2│RCLK │TCLK │EXEN2│ TR2 │ C/T2̅│CP/RL2̅│
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
  D7    D6    D5    D4    D3    D2    D1    D0

Modes:
• Auto-reload (CP/RL2̅=0): Reload from RCAP2H:RCAP2L on overflow
• Capture (CP/RL2̅=1): Capture TH2:TL2 to RCAP2 on T2EX falling edge
• Baud rate (RCLK=1 or TCLK=1): Generate serial baud rate

EXAMPLE: Timer 2 for baud rate
─────────────────────────────
    MOV T2CON, #34H     ; RCLK=1, TCLK=1, TR2=1
    MOV RCAP2H, #0FFH   ; Reload value for baud rate
    MOV RCAP2L, #0DCH
```

---

## 📋 Summary Table

| Mode | Size | Range | Max Delay @12MHz | Use Case |
|------|------|-------|------------------|----------|
| Mode 0 | 13-bit | 0-8191 | 8.192 ms | Legacy |
| Mode 1 | 16-bit | 0-65535 | 65.536 ms | Delays |
| Mode 2 | 8-bit auto | 0-255 | 256 µs | Baud rate |
| Mode 3 | 2×8-bit | 0-255 each | 256 µs each | Extra timers |

---

## ❓ Quick Revision Questions

1. **Calculate timer values for 10 ms delay at 12 MHz.**
   <details>
   <summary>Show Answer</summary>
   Counts needed = 10,000 (10 ms × 1 MHz). Initial value = 65536 - 10000 = 55536 = D8F0H. TH0 = D8H, TL0 = F0H. Use Mode 1.
   </details>

2. **What is the maximum delay in Mode 1 with 11.0592 MHz crystal?**
   <details>
   <summary>Show Answer</summary>
   Machine cycle = 12/11.0592 MHz = 1.085 µs. Max delay = 65536 × 1.085 µs = 71.11 ms.
   </details>

3. **Why is Mode 2 preferred for baud rate generation?**
   <details>
   <summary>Show Answer</summary>
   Mode 2 auto-reloads TL from TH on overflow, providing consistent timing without software intervention. This ensures accurate, continuous baud rate clock generation.
   </details>

4. **How do you configure Timer 0 as a counter for external events?**
   <details>
   <summary>Show Answer</summary>
   Set C/T̅ bit = 1 in TMOD. For Mode 1 counter: MOV TMOD, #05H. External pulses on T0 pin (P3.4) will increment the counter.
   </details>

5. **What does GATE = 1 do in timer operation?**
   <details>
   <summary>Show Answer</summary>
   When GATE = 1, the timer runs only when TR = 1 AND the corresponding INTx pin is HIGH. This allows external control of timing (e.g., pulse width measurement).
   </details>

6. **How do you generate a 1-second delay using timers?**
   <details>
   <summary>Show Answer</summary>
   Max Mode 1 delay is ~65.5 ms. Use multiple overflows: 20 × 50 ms = 1 second. Use a counter register (R7 = 20), loop with 50 ms delays, decrement until zero.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [8.1 Instruction Set](01-instruction-set.md) | [Unit 8 Index](README.md) | [8.3 Interrupt Programming](03-interrupt-programming.md) |

---

*[← Previous: Instruction Set](01-instruction-set.md) | [Next: Interrupt Programming →](03-interrupt-programming.md)*
