# Chapter 8.3: 8051 Interrupt Programming

## 📚 Chapter Overview

This chapter covers the 8051 interrupt system comprehensively, including all interrupt sources, priority levels, enabling/disabling interrupts, and writing interrupt service routines.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Understand the 8051 interrupt architecture
- Configure and enable various interrupt sources
- Write effective Interrupt Service Routines (ISR)
- Manage interrupt priorities
- Handle external and internal interrupts

---

## 1. Interrupt Fundamentals

### 1.1 What is an Interrupt?

```
INTERRUPT CONCEPT
━━━━━━━━━━━━━━━━━

An interrupt is a signal that temporarily halts the main program
to execute a special routine (ISR), then returns to main program.

WITHOUT INTERRUPTS (Polling):
────────────────────────────

┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   Main Program                                                  │
│   ┌──────────────────────────────────────────────────────────┐ │
│   │  Code...                                                 │ │
│   │  Check button? ─► No ─┐                                  │ │
│   │  More code...         │                                  │ │
│   │  Check button? ─► No ─┤   Wastes CPU time               │ │
│   │  More code...         │   continuously checking         │ │
│   │  Check button? ─► Yes!│                                  │ │
│   │  Handle button...     │                                  │ │
│   └──────────────────────────────────────────────────────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

WITH INTERRUPTS:
───────────────

┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   Main Program          Interrupt Occurs!                       │
│   ┌──────────────┐            │                                │
│   │  Code...     │            │                                │
│   │  More code...│◄───────────┤ CPU saves state               │
│   │  More code...│    ┌───────▼───────┐                        │
│   │              │    │     ISR       │ Handles event          │
│   │              │    │  Handle event │                        │
│   │              │    │     RETI      │                        │
│   │  Continues..◄┼────┴───────────────┘ Resumes main          │
│   └──────────────┘                                             │
│                                                                 │
│   Advantages:                                                   │
│   • CPU does useful work until event occurs                    │
│   • Faster response to events                                  │
│   • More efficient use of processor time                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 8051 Interrupt Sources

```
8051 INTERRUPT SOURCES
━━━━━━━━━━━━━━━━━━━━━━

Five Interrupt Sources:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Interrupt    Source              Vector      Priority (default)│
│  ──────────────────────────────────────────────────────────────│
│  INT0         External Int 0      0003H       Highest (1)       │
│  TF0          Timer 0 Overflow    000BH       (2)               │
│  INT1         External Int 1      0013H       (3)               │
│  TF1          Timer 1 Overflow    001BH       (4)               │
│  RI/TI        Serial Port         0023H       Lowest (5)        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

For 8052 (additional):
  TF2/EXF2      Timer 2             002BH

MEMORY MAP WITH INTERRUPT VECTORS:
─────────────────────────────────

    Address    Content
    ────────────────────────────
    0000H  ┌───────────────────┐
           │  Reset Vector     │─► Program starts here
           │  (LJMP MAIN)      │
    0003H  ├───────────────────┤
           │  INT0 ISR         │─► External Interrupt 0
           │  (8 bytes)        │
    000BH  ├───────────────────┤
           │  Timer 0 ISR      │─► Timer 0 Overflow
           │  (8 bytes)        │
    0013H  ├───────────────────┤
           │  INT1 ISR         │─► External Interrupt 1
           │  (8 bytes)        │
    001BH  ├───────────────────┤
           │  Timer 1 ISR      │─► Timer 1 Overflow
           │  (8 bytes)        │
    0023H  ├───────────────────┤
           │  Serial ISR       │─► Serial Port
           │  (8 bytes)        │
    002BH  ├───────────────────┤
           │  Timer 2 ISR      │─► Timer 2 (8052 only)
           │  (8 bytes)        │
    0033H  ├───────────────────┤
           │  Main Program     │
           │  ...              │
           └───────────────────┘

NOTE: Only 8 bytes between vectors! 
      Use LJMP to jump to actual ISR elsewhere in memory.
```

---

## 2. Interrupt Control Registers

### 2.1 IE Register (Interrupt Enable)

```
IE - INTERRUPT ENABLE REGISTER (Address A8H)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│ EA  │  -  │ ET2 │ ES  │ ET1 │ EX1 │ ET0 │ EX0 │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
  D7    D6    D5    D4    D3    D2    D1    D0
  AFH         ADH   ACH   ABH   AAH   A9H   A8H  ◄── Bit addresses

BIT FUNCTIONS:
──────────────

EA (AFH): Global Enable/Disable
  1 = Enable interrupts (individual bits must also be set)
  0 = Disable ALL interrupts (master switch)

ET2 (ADH): Timer 2 Interrupt Enable (8052 only)
  1 = Enable Timer 2 interrupt
  0 = Disable Timer 2 interrupt

ES (ACH): Serial Port Interrupt Enable
  1 = Enable Serial port interrupt
  0 = Disable Serial port interrupt

ET1 (ABH): Timer 1 Interrupt Enable
  1 = Enable Timer 1 overflow interrupt
  0 = Disable Timer 1 interrupt

EX1 (AAH): External Interrupt 1 Enable
  1 = Enable External interrupt 1 (INT1/P3.3)
  0 = Disable External interrupt 1

ET0 (A9H): Timer 0 Interrupt Enable
  1 = Enable Timer 0 overflow interrupt
  0 = Disable Timer 0 interrupt

EX0 (A8H): External Interrupt 0 Enable
  1 = Enable External interrupt 0 (INT0/P3.2)
  0 = Disable External interrupt 0

COMMON IE VALUES:
────────────────

┌────────┬────────────────────────────────────────────────────────┐
│ Value  │ Configuration                                          │
├────────┼────────────────────────────────────────────────────────┤
│  81H   │ Enable External Interrupt 0 only                       │
│  82H   │ Enable Timer 0 interrupt only                          │
│  84H   │ Enable External Interrupt 1 only                       │
│  88H   │ Enable Timer 1 interrupt only                          │
│  90H   │ Enable Serial interrupt only                           │
│  83H   │ Enable EX0 and Timer 0                                 │
│  8FH   │ Enable all interrupts (EX0, T0, EX1, T1)              │
│  9FH   │ Enable all including serial                            │
└────────┴────────────────────────────────────────────────────────┘
```

### 2.2 IP Register (Interrupt Priority)

```
IP - INTERRUPT PRIORITY REGISTER (Address B8H)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│  -  │  -  │ PT2 │ PS  │ PT1 │ PX1 │ PT0 │ PX0 │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
  D7    D6    D5    D4    D3    D2    D1    D0
              BDH   BCH   BBH   BAH   B9H   B8H  ◄── Bit addresses

BIT FUNCTIONS:
──────────────
0 = Low priority
1 = High priority

PT2: Timer 2 priority (8052 only)
PS:  Serial port priority
PT1: Timer 1 priority
PX1: External Interrupt 1 priority
PT0: Timer 0 priority
PX0: External Interrupt 0 priority

PRIORITY LEVELS:
───────────────

Two-level priority system:
• High priority (IP bit = 1)
• Low priority (IP bit = 0)

Within same priority level, polling sequence decides:
    INT0 > TF0 > INT1 > TF1 > Serial

PRIORITY RULES:
──────────────
1. High priority can interrupt low priority
2. Low priority CANNOT interrupt high priority
3. Same priority level: earlier in polling wins
4. No interrupt can interrupt itself

EXAMPLE: Make Timer 1 and Serial high priority
─────────────────────────────────────────────

    MOV IP, #18H        ; PT1=1, PS=1
    ; Binary: 0001 1000
    
    ; Or using bit operations:
    SETB PT1            ; Set Timer 1 high priority
    SETB PS             ; Set Serial high priority
```

### 2.3 TCON Interrupt Bits

```
TCON INTERRUPT BITS (Address 88H)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Lower 4 bits control external interrupts:

┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│ TF1 │ TR1 │ TF0 │ TR0 │ IE1 │ IT1 │ IE0 │ IT0 │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
                          D3    D2    D1    D0
                          8BH   8AH   89H   88H

EXTERNAL INTERRUPT BITS:
───────────────────────

IE1 (8BH): External Interrupt 1 Flag
  Set when INT1 interrupt is detected
  Cleared automatically for edge-triggered
  Must be cleared by software for level-triggered

IT1 (8AH): Interrupt 1 Type
  0 = Level triggered (low level)
  1 = Edge triggered (falling edge)

IE0 (89H): External Interrupt 0 Flag
  Set when INT0 interrupt is detected

IT0 (88H): Interrupt 0 Type
  0 = Level triggered
  1 = Edge triggered

LEVEL vs EDGE TRIGGERED:
───────────────────────

Level Triggered (ITx = 0):
    ┌──────────────────────────────
    │  Interrupt remains active
─────┘  as long as INTx is LOW

    • Must hold low for at least 12 clock cycles
    • Source must return high before ISR completes
    • Continuous interrupt if still low

Edge Triggered (ITx = 1):
    ──────┐
          │  Falling edge triggers
          └──────────────────────

    • Single interrupt per falling edge
    • Flag set on falling edge
    • Cleared automatically when ISR starts
    • Most common choice
```

---

## 3. Writing Interrupt Service Routines

### 3.1 ISR Structure

```
INTERRUPT SERVICE ROUTINE (ISR) STRUCTURE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

BASIC TEMPLATE:
──────────────

    ORG 0000H
    LJMP MAIN           ; Reset vector - jump to main

    ORG 0003H           ; INT0 vector
    LJMP INT0_ISR       ; Jump to ISR (only 8 bytes available)

    ORG 000BH           ; Timer 0 vector
    LJMP T0_ISR

    ORG 0013H           ; INT1 vector
    LJMP INT1_ISR

    ORG 001BH           ; Timer 1 vector
    LJMP T1_ISR

    ORG 0023H           ; Serial vector
    LJMP SERIAL_ISR

    ORG 0030H           ; Main program starts here
MAIN:
    ; Initialization code
    MOV IE, #...        ; Enable interrupts
    ; Main program loop
    SJMP $

; Actual ISR code (can be anywhere in memory)
INT0_ISR:
    ; Save context if needed
    PUSH ACC
    PUSH PSW
    
    ; ISR code here
    
    ; Restore context
    POP PSW
    POP ACC
    RETI                ; Return from interrupt (NOT RET!)

IMPORTANT RULES:
───────────────
1. Use RETI, not RET (RETI clears interrupt-in-service flag)
2. Keep ISR short and fast
3. Save/restore registers you modify
4. Clear interrupt flags if not auto-cleared
5. Don't call functions that use same registers
```

### 3.2 External Interrupt Example

```
EXTERNAL INTERRUPT PROGRAMMING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

; Toggle LED on P1.0 when button on INT0 (P3.2) is pressed
; Edge-triggered interrupt

    ORG 0000H
    LJMP MAIN

    ORG 0003H           ; INT0 vector
    LJMP INT0_ISR

    ORG 0030H
MAIN:
    ; Initialize
    MOV P1, #00H        ; LED off initially
    SETB IT0            ; Edge triggered
    SETB EX0            ; Enable External INT0
    SETB EA             ; Enable global interrupts
    
LOOP:
    ; Main program does other tasks
    ; or simply waits
    SJMP LOOP

INT0_ISR:
    CPL P1.0            ; Toggle LED
    RETI

; Level-triggered example with debouncing
INT0_LEVEL:
    CLR IT0             ; Level triggered
    SETB EX0            ; Enable INT0
    SETB EA
    SJMP $

INT0_ISR_LEVEL:
    PUSH ACC
    CLR P1.0            ; LED ON
    
    ; Wait for button release
WAIT_RELEASE:
    JNB P3.2, WAIT_RELEASE
    
    ; Debounce delay
    MOV R7, #50
DEBOUNCE:
    DJNZ R7, DEBOUNCE
    
    SETB P1.0           ; LED OFF
    POP ACC
    RETI
```

### 3.3 Timer Interrupt Example

```
TIMER INTERRUPT PROGRAMMING
━━━━━━━━━━━━━━━━━━━━━━━━━━━

; Generate 1 Hz LED blink using Timer 0 interrupt
; @ 12 MHz crystal

    ORG 0000H
    LJMP MAIN

    ORG 000BH           ; Timer 0 vector
    LJMP T0_ISR

    ORG 0030H
MAIN:
    MOV TMOD, #01H      ; Timer 0 Mode 1
    MOV TH0, #3CH       ; 50 ms delay value
    MOV TL0, #0B0H
    MOV R7, #20         ; 20 × 50ms = 1 second
    
    SETB ET0            ; Enable Timer 0 interrupt
    SETB EA             ; Enable global
    SETB TR0            ; Start Timer 0
    
    SJMP $              ; Main loop does nothing

T0_ISR:
    CLR TR0             ; Stop timer
    
    ; Reload timer
    MOV TH0, #3CH
    MOV TL0, #0B0H
    
    DJNZ R7, SKIP_TOGGLE
    
    CPL P1.0            ; Toggle LED every second
    MOV R7, #20         ; Reset counter
    
SKIP_TOGGLE:
    SETB TR0            ; Restart timer
    RETI

; Using Mode 2 for automatic reload
TIMER_MODE2:
    MOV TMOD, #02H      ; Timer 0 Mode 2
    MOV TH0, #0CEH      ; 256-206 = 50 µs
    MOV TL0, #0CEH
    SETB ET0
    SETB EA
    SETB TR0
    
T0_ISR_MODE2:
    ; TL0 auto-reloaded from TH0
    ; Just process the overflow
    INC R7
    CJNE R7, #200, SKIP ; Count 200 × 50µs = 10ms
    MOV R7, #0
    CPL P1.0
SKIP:
    RETI
```

### 3.4 Serial Port Interrupt Example

```
SERIAL PORT INTERRUPT PROGRAMMING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

; Receive data via serial port using interrupt
; Echo back received data
; @ 11.0592 MHz, 9600 baud

    ORG 0000H
    LJMP MAIN

    ORG 0023H           ; Serial vector
    LJMP SERIAL_ISR

    ORG 0030H
MAIN:
    ; Configure serial port
    MOV TMOD, #20H      ; Timer 1 Mode 2 for baud rate
    MOV TH1, #0FDH      ; 9600 baud
    MOV TL1, #0FDH
    MOV SCON, #50H      ; Mode 1, REN enabled
    SETB TR1            ; Start Timer 1
    
    ; Enable serial interrupt
    SETB ES             ; Enable serial interrupt
    SETB EA             ; Enable global
    
MAIN_LOOP:
    ; Main program does other tasks
    SJMP MAIN_LOOP

SERIAL_ISR:
    PUSH ACC
    PUSH PSW
    
    JNB RI, CHECK_TI    ; Check if receive interrupt
    
    ; Receive interrupt
    CLR RI              ; Clear receive flag
    MOV A, SBUF         ; Read received byte
    MOV P1, A           ; Display on Port 1
    MOV SBUF, A         ; Echo back (transmit)
    SJMP SERIAL_EXIT
    
CHECK_TI:
    JNB TI, SERIAL_EXIT ; Check if transmit interrupt
    CLR TI              ; Clear transmit flag
    ; Transmission complete
    
SERIAL_EXIT:
    POP PSW
    POP ACC
    RETI

; Buffered serial receive
; Store received data in memory buffer

BUFFER_START EQU 40H
BUFFER_SIZE  EQU 16

SERIAL_BUFFERED:
    MOV R0, #BUFFER_START
    MOV R1, #0          ; Buffer index

SERIAL_ISR_BUF:
    PUSH ACC
    
    JNB RI, TX_INT
    CLR RI
    MOV A, SBUF
    MOV @R0, A          ; Store in buffer
    INC R0
    INC R1
    CJNE R1, #BUFFER_SIZE, SERIAL_DONE
    MOV R0, #BUFFER_START  ; Wrap around
    MOV R1, #0
    SJMP SERIAL_DONE
    
TX_INT:
    CLR TI
    
SERIAL_DONE:
    POP ACC
    RETI
```

---

## 4. Multiple Interrupts

### 4.1 Interrupt Priority Programming

```
INTERRUPT PRIORITY EXAMPLE
━━━━━━━━━━━━━━━━━━━━━━━━━━

; Configure priorities:
;   Timer 0: High priority (time-critical)
;   INT0: High priority (emergency button)
;   Others: Low priority

    ORG 0000H
    LJMP MAIN

    ORG 0003H
    LJMP INT0_ISR

    ORG 000BH
    LJMP T0_ISR

    ORG 0013H
    LJMP INT1_ISR

    ORG 001BH
    LJMP T1_ISR

    ORG 0030H
MAIN:
    ; Set priorities
    MOV IP, #03H        ; PX0=1, PT0=1 (high priority)
                        ; Binary: 0000 0011
    
    ; Or using bit operations
    SETB PX0            ; INT0 high priority
    SETB PT0            ; Timer 0 high priority
    ; PX1, PT1, PS remain 0 (low priority)
    
    ; Enable interrupts
    MOV IE, #8FH        ; EA=1, enable EX0, ET0, EX1, ET1
    
    ; Initialize timers
    MOV TMOD, #11H      ; Both timers Mode 1
    SETB TR0
    SETB TR1
    
    SJMP $

; High priority ISR - can interrupt low priority
T0_ISR:
    PUSH ACC
    ; Critical timing code
    MOV TH0, #3CH
    MOV TL0, #0B0H
    POP ACC
    RETI

; Low priority ISR - can be interrupted by high priority
T1_ISR:
    PUSH ACC
    ; Less critical code
    ; This can be interrupted by T0_ISR or INT0_ISR
    POP ACC
    RETI
```

### 4.2 Nested Interrupts

```
NESTED INTERRUPT EXAMPLE
━━━━━━━━━━━━━━━━━━━━━━━━

High priority interrupt can nest into low priority ISR:

Scenario:
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│   Main Program                                                 │
│        │                                                       │
│        ▼ T1 interrupt (low priority)                          │
│   ┌─────────────────┐                                         │
│   │  T1_ISR running │                                         │
│   │      │          │                                         │
│   │      ▼ T0 interrupt (high priority)                       │
│   │   ┌──────────────┐                                        │
│   │   │  T0_ISR      │ Nested!                                │
│   │   │  (runs)      │                                        │
│   │   │  RETI        │                                        │
│   │   └──────────────┘                                        │
│   │      │                                                     │
│   │      ▼ Return to T1_ISR                                   │
│   │  Continue T1_ISR │                                        │
│   │  RETI            │                                        │
│   └─────────────────┘                                         │
│        │                                                       │
│        ▼ Return to Main                                       │
│                                                                │
└────────────────────────────────────────────────────────────────┘

IMPORTANT NOTES:
───────────────
• Low priority ISR can be interrupted by high priority
• High priority ISR cannot be interrupted
• Same priority level cannot nest
• Stack must have enough space for nested ISRs
```

---

## 5. Interrupt Latency and Timing

```
INTERRUPT LATENCY
━━━━━━━━━━━━━━━━━

Latency = time from interrupt event to ISR execution

Components:
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  1. Complete current instruction (1-4 machine cycles)        │
│  2. Hardware saves PC to stack (2 cycles)                    │
│  3. Load vector address (2 cycles)                           │
│  4. Execute LJMP if present (2 cycles)                       │
│                                                              │
│  Minimum: 3 machine cycles                                   │
│  Maximum: ~10 machine cycles (if MUL/DIV executing)          │
│                                                              │
│  @ 12 MHz: 3-10 µs typical                                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘

INTERRUPT RESPONSE TIME:
───────────────────────

For time-critical applications:
• Use high priority for fastest response
• Keep high priority ISR very short
• Avoid long instructions (MUL, DIV) in critical sections
• Use edge-triggered interrupts for clean detection

EXAMPLE TIMING ANALYSIS:
───────────────────────

External interrupt (edge-triggered):
  1. INT0 falling edge detected
  2. IE0 flag set (next machine cycle)
  3. Current instruction completes (1-4 cycles)
  4. PC pushed, vector loaded (2 cycles)
  5. LJMP to ISR (2 cycles)
  6. First ISR instruction executes
  
  Total: 5-8 machine cycles = 5-8 µs @ 12 MHz
```

---

## 6. Practical Applications

### 6.1 Keypad Interrupt Handler

```
KEYPAD WITH INTERRUPT
━━━━━━━━━━━━━━━━━━━━━

; Keypad connected with interrupt line on INT1
; Key press generates interrupt

    ORG 0000H
    LJMP MAIN

    ORG 0013H           ; INT1 vector
    LJMP KEYPAD_ISR

    ORG 0030H
MAIN:
    MOV P2, #0FFH       ; Keypad port
    SETB IT1            ; Edge triggered
    SETB EX1            ; Enable INT1
    SETB EA
    
    MOV R0, #0          ; Key storage
    
MAIN_LOOP:
    MOV A, R0           ; Check last key
    JZ NO_KEY
    ; Process key
    MOV P1, A           ; Display key code
    MOV R0, #0          ; Clear key
NO_KEY:
    SJMP MAIN_LOOP

KEYPAD_ISR:
    PUSH ACC
    PUSH B
    
    ; Debounce
    MOV R7, #100
DEBOUNCE:
    DJNZ R7, DEBOUNCE
    
    ; Scan keypad
    CALL SCAN_KEYPAD
    MOV R0, A           ; Store key code
    
    POP B
    POP ACC
    RETI

SCAN_KEYPAD:
    ; Keypad scanning routine
    ; Returns key code in A
    ; (Implementation depends on hardware)
    RET
```

### 6.2 Real-Time Clock

```
REAL-TIME CLOCK USING TIMER INTERRUPT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

; Maintain seconds, minutes, hours using Timer 0
; @ 12 MHz crystal

; Time storage in internal RAM
SECONDS EQU 30H
MINUTES EQU 31H
HOURS   EQU 32H
TICKS   EQU 33H         ; 50ms tick counter

    ORG 0000H
    LJMP MAIN

    ORG 000BH
    LJMP TIMER0_ISR

    ORG 0030H
MAIN:
    ; Initialize time
    MOV SECONDS, #0
    MOV MINUTES, #0
    MOV HOURS, #0
    MOV TICKS, #0
    
    ; Configure Timer 0
    MOV TMOD, #01H      ; Mode 1
    MOV TH0, #3CH       ; 50 ms
    MOV TL0, #0B0H
    
    SETB ET0            ; Enable Timer 0 interrupt
    SETB EA
    SETB TR0            ; Start timer
    
MAIN_LOOP:
    ; Display time on LCD or 7-segment
    CALL DISPLAY_TIME
    SJMP MAIN_LOOP

TIMER0_ISR:
    PUSH ACC
    
    ; Reload timer
    CLR TR0
    MOV TH0, #3CH
    MOV TL0, #0B0H
    SETB TR0
    
    ; Increment ticks
    INC TICKS
    MOV A, TICKS
    CJNE A, #20, T0_EXIT    ; 20 × 50ms = 1 second
    
    ; One second elapsed
    MOV TICKS, #0
    INC SECONDS
    MOV A, SECONDS
    CJNE A, #60, T0_EXIT
    
    ; One minute elapsed
    MOV SECONDS, #0
    INC MINUTES
    MOV A, MINUTES
    CJNE A, #60, T0_EXIT
    
    ; One hour elapsed
    MOV MINUTES, #0
    INC HOURS
    MOV A, HOURS
    CJNE A, #24, T0_EXIT
    
    ; Day rollover
    MOV HOURS, #0
    
T0_EXIT:
    POP ACC
    RETI

DISPLAY_TIME:
    ; Display routine (hardware dependent)
    RET
```

---

## 📋 Summary Table

| Interrupt | Vector | Flag | Enable | Priority | Trigger |
|-----------|--------|------|--------|----------|---------|
| INT0 | 0003H | IE0 | EX0 | PX0 | Level/Edge |
| Timer 0 | 000BH | TF0 | ET0 | PT0 | Overflow |
| INT1 | 0013H | IE1 | EX1 | PX1 | Level/Edge |
| Timer 1 | 001BH | TF1 | ET1 | PT1 | Overflow |
| Serial | 0023H | RI/TI | ES | PS | Rx/Tx complete |
| Timer 2 | 002BH | TF2/EXF2 | ET2 | PT2 | Overflow/External |

---

## ❓ Quick Revision Questions

1. **What is the difference between RET and RETI?**
   <details>
   <summary>Show Answer</summary>
   RET simply pops PC from stack and returns. RETI pops PC AND clears the interrupt-in-service flag, allowing same-level interrupts. Always use RETI to end ISR.
   </details>

2. **How do you configure INT0 for edge-triggered operation?**
   <details>
   <summary>Show Answer</summary>
   Set IT0 = 1 (SETB IT0), then enable with EX0 = 1 (SETB EX0), and EA = 1 (SETB EA). Falling edge on P3.2 will trigger interrupt.
   </details>

3. **Why is there only 8 bytes between interrupt vectors?**
   <details>
   <summary>Show Answer</summary>
   Historic design decision for code space efficiency. For longer ISRs, place LJMP at vector location to jump to actual ISR elsewhere in memory.
   </details>

4. **Can a low-priority interrupt interrupt a high-priority ISR?**
   <details>
   <summary>Show Answer</summary>
   No. High priority ISR cannot be interrupted by any interrupt. Only high priority interrupts can interrupt low priority ISRs.
   </details>

5. **What must you do in a serial port ISR?**
   <details>
   <summary>Show Answer</summary>
   Check both RI and TI flags to determine interrupt source. Clear the flag (CLR RI or CLR TI) manually as they are not auto-cleared.
   </details>

6. **How do you make Timer 0 interrupt highest priority?**
   <details>
   <summary>Show Answer</summary>
   Set PT0 = 1 in IP register (SETB PT0 or MOV IP, #02H). This gives Timer 0 high priority. For absolute highest, ensure PX0 = 0 since INT0 has higher default priority at same level.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [8.2 Timer Programming](02-timer-programming.md) | [Unit 8 Index](README.md) | [8.4 Serial Communication](04-serial-communication.md) |

---

*[← Previous: Timer Programming](02-timer-programming.md) | [Next: Serial Communication →](04-serial-communication.md)*
