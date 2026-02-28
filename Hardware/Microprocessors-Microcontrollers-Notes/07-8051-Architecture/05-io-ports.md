# Chapter 7.5: 8051 I/O Ports

## 📚 Chapter Overview

This chapter provides comprehensive coverage of the four I/O ports (P0, P1, P2, P3) of the 8051 microcontroller, including their internal structure, electrical characteristics, alternate functions, and programming techniques.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Understand the internal structure of each I/O port
- Differentiate between quasi-bidirectional and true bidirectional ports
- Configure ports for input and output operations
- Use alternate functions of Port 3
- Interface LEDs, switches, and other devices

---

## 1. I/O Ports Overview

### 1.1 Port Summary

```
8051 I/O PORTS SUMMARY
━━━━━━━━━━━━━━━━━━━━━━

┌──────┬─────────┬──────────────────────────────────────────────────┐
│ Port │  Pins   │ Features                                         │
├──────┼─────────┼──────────────────────────────────────────────────┤
│  P0  │ 32-39   │ • True bidirectional (open-drain)                │
│      │         │ • Needs external pull-ups for I/O                │
│      │         │ • Multiplexed Address/Data bus (AD0-AD7)         │
│      │         │ • 8 pins, address 80H                            │
├──────┼─────────┼──────────────────────────────────────────────────┤
│  P1  │  1-8    │ • Quasi-bidirectional with internal pull-ups     │
│      │         │ • General purpose I/O only                       │
│      │         │ • 8 pins, address 90H                            │
│      │         │ • 8052: T2, T2EX alternate functions             │
├──────┼─────────┼──────────────────────────────────────────────────┤
│  P2  │ 21-28   │ • Quasi-bidirectional with internal pull-ups     │
│      │         │ • Upper address byte (A8-A15)                    │
│      │         │ • 8 pins, address A0H                            │
├──────┼─────────┼──────────────────────────────────────────────────┤
│  P3  │ 10-17   │ • Quasi-bidirectional with internal pull-ups     │
│      │         │ • Alternate functions (serial, int, timer)       │
│      │         │ • 8 pins, address B0H                            │
└──────┴─────────┴──────────────────────────────────────────────────┘

All ports are bit-addressable:
• P0.0-P0.7 (bits 80H-87H)
• P1.0-P1.7 (bits 90H-97H)
• P2.0-P2.7 (bits A0H-A7H)
• P3.0-P3.7 (bits B0H-B7H)
```

---

## 2. Port 0 (P0)

### 2.1 Port 0 Internal Structure

```
PORT 0 INTERNAL STRUCTURE (One Bit)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

                        VCC
                         │
                    ┌────┴────┐
       Control ───►│ Upper   │◄─── Internal Address/Data
            MUX    │  FET    │     (during bus operations)
                   │ (P-ch)  │
                   └────┬────┘
                        │
                        ├───────────────────► P0.x (Pin)
                        │
                   ┌────┴────┐
       D ─────────►│ Lower   │
       Q ───────┬─►│  FET    │
               │  │ (N-ch)  │
               │  └─────────┘
               │       │
               │       ▼
       D Latch │      GND
       ┌───┐   │
    ───┤   ├───┘
       └───┘
         │
        Read ◄───────────────────────────────── (Input Buffer)

Key Features:
┌────────────────────────────────────────────────────────────────┐
│ • NO INTERNAL PULL-UP - Open drain output                      │
│ • Upper FET only conducts during bus operations (address/data) │
│ • For I/O use, MUST add external 10K pull-up resistors        │
│ • Can sink 1.6 mA, source 0 mA (without pull-up)              │
│ • With external pull-up, can source/sink based on pull-up     │
│ • Address/Data bus: Actively drives both high and low          │
└────────────────────────────────────────────────────────────────┘

USING P0 FOR GENERAL I/O:
────────────────────────

              VCC
               │
              ┌┴┐
              │ │ 10K (each pin)
              │ │
              └┬┘
               │
    P0.x ──────┼──────► To external circuit
               │
           Internal
           Open-drain
               │
              GND

    ; Output to P0
    MOV P0, #0FFH    ; All pins HIGH (pulled up externally)
    MOV P0, #00H     ; All pins LOW (FET pulls to ground)
    
    ; Input from P0
    MOV P0, #0FFH    ; Write 1s first (release pins)
    MOV A, P0        ; Read port pins
```

### 2.2 Port 0 as Address/Data Bus

```
P0 AS MULTIPLEXED BUS (AD0-AD7)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

During external memory access, P0 outputs lower address,
then becomes data bus:

    Machine Cycle
    ├─── S1 ───┼─── S2 ───┼─── S3 ───┼─── S4 ───┤

ALE     ┐      ┌─┐                              
        └──────┘ └──────────────────────────────

P0      ◄──A0-A7──►◄────────D0-D7──────────────►
        (Address)  (Data)

P̅S̅E̅N̅           ┐            ┌─────────────────┐
         ──────┘            └─────────────────┘

External Memory Connection:
──────────────────────────

    8051                    74373             27C256
   ┌─────┐                 ┌─────┐           ┌─────┐
   │     │ P0.0-P0.7  ────►│D   Q├──────────►│A0-A7│
   │     │      │          │     │           │     │
   │     │      └─────────►│     ├──────────►│D0-D7│
   │     │                 │  G  │           │     │
   │ ALE ├────────────────►│     │           │     │
   │     │                 └─────┘           │     │
   │P̅S̅E̅N̅ ├──────────────────────────────────►│O̅E̅   │
   └─────┘                                   └─────┘

• ALE latches address on 74373 (transparent latch)
• P̅S̅E̅N̅ enables ROM output during data phase
• NO external pull-ups needed for bus mode
```

---

## 3. Port 1 (P1)

### 3.1 Port 1 Internal Structure

```
PORT 1 INTERNAL STRUCTURE (One Bit)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

                        VCC
                         │
                    ┌────┴────┐
                    │Internal │
                    │Pull-up  │ (FET acts as ~30K resistor)
                    │  FET    │
                    └────┬────┘
                         │
       D ───────────┬────┼───────────────────► P1.x (Pin)
       Q ──────┐    │    │
               │    │    │
               ▼    │    │
          ┌───────┐ │    │
          │Output │ │    │
          │Buffer │ │    │
          │(N-FET)│ │    │
          └───┬───┘ │    │
              │     │    │
              ▼     │    │
             GND    │    │
                    │    │
       D Latch      │    │
       ┌───┐        │    │
    ───┤   ├────────┘    │
       └───┘             │
                         │
              Read ◄─────┘ (Input Buffer)

Key Features:
┌────────────────────────────────────────────────────────────────┐
│ • INTERNAL PULL-UP (~30-60K ohms, weak)                        │
│ • Quasi-bidirectional port                                     │
│ • No external pull-ups needed for most applications            │
│ • Can sink ~1.6 mA, source ~60 µA (weak)                      │
│ • General purpose I/O only (no alternate functions on 8051)    │
│ • 8052: P1.0=T2 (Timer 2 input), P1.1=T2EX (Capture/Reload)   │
│ • Write 1 before reading to enable input mode                  │
└────────────────────────────────────────────────────────────────┘

QUASI-BIDIRECTIONAL OPERATION:
─────────────────────────────

OUTPUT HIGH (Write 1):
    - N-FET OFF
    - Pin pulled HIGH by internal FET pull-up
    - Can only source ~60 µA (weak drive)

OUTPUT LOW (Write 0):
    - N-FET ON (strong)
    - Pin pulled LOW to ground
    - Can sink ~1.6 mA (strong drive)

INPUT:
    - Write 1 first (disable N-FET)
    - External device can pull pin low
    - Read actual pin voltage, not latch
```

### 3.2 Port 1 Applications

```
PORT 1 INTERFACING EXAMPLES
━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. LED Connection (Active Low - Recommended):
──────────────────────────────────────────

         VCC
          │
         ┌┴┐
    330Ω │ │
         └┬┘
          │
         ─┼─
         \│/ LED
          ▼
          │
    P1.x ─┴─

    ; Turn LED ON
    CLR P1.0        ; Pin LOW, LED lights
    
    ; Turn LED OFF
    SETB P1.0       ; Pin HIGH, LED off

2. Switch Connection (Active Low):
─────────────────────────────────

         VCC
          │ (Internal pull-up)
    P1.x ─┼─
          │
         │ │
         └─┤ SW
           │
          GND

    ; Read switch
    SETB P1.1       ; Enable input mode (write 1)
    JNB P1.1, SW_PRESSED
    JB P1.1, SW_NOT_PRESSED

3. 7-Segment Display (Common Cathode):
─────────────────────────────────────

    P1.0 ──┬──[ 330Ω ]──► a
    P1.1 ──┼──[ 330Ω ]──► b
    P1.2 ──┼──[ 330Ω ]──► c
    P1.3 ──┼──[ 330Ω ]──► d
    P1.4 ──┼──[ 330Ω ]──► e
    P1.5 ──┼──[ 330Ω ]──► f
    P1.6 ──┼──[ 330Ω ]──► g
    P1.7 ──┴──[ 330Ω ]──► dp

    ; Display digit lookup table
    ; Format: 0gfedcba
    TABLE:
        DB 3FH  ; 0: abcdef
        DB 06H  ; 1: bc
        DB 5BH  ; 2: abdeg
        DB 4FH  ; 3: abcdg
        DB 66H  ; 4: bcfg
        DB 6DH  ; 5: acdfg
        DB 7DH  ; 6: acdefg
        DB 07H  ; 7: abc
        DB 7FH  ; 8: abcdefg
        DB 6FH  ; 9: abcdfg
```

---

## 4. Port 2 (P2)

### 4.1 Port 2 Internal Structure

```
PORT 2 INTERNAL STRUCTURE
━━━━━━━━━━━━━━━━━━━━━━━━━

Similar to Port 1, with address bus capability:

                        VCC
                         │
                    ┌────┴────┐
       Control ───►│Internal │◄─── Address High (A8-A15)
            MUX    │Pull-up  │     (during external memory access)
                   │  FET    │
                   └────┬────┘
                        │
                        ├───────────────────► P2.x (Pin)
                        │
                   ┌────┴────┐
       D ─────────►│ Output  │
       Q ──────────►│  FET   │
                   └─────────┘
                        │
                        ▼
                       GND

Key Features:
┌────────────────────────────────────────────────────────────────┐
│ • Internal pull-up like P1 (quasi-bidirectional)              │
│ • During external memory access: Outputs A8-A15               │
│ • If no external memory: General purpose I/O                  │
│ • Port latch contents preserved during memory access          │
│ • Address appears only briefly, then returns to latch value   │
└────────────────────────────────────────────────────────────────┘

P2 AS ADDRESS BUS:
─────────────────

    ┌─────────────────────────────────────────────┐
    │  During External Memory Access:             │
    │                                             │
    │  P2.7  P2.6  P2.5  P2.4  P2.3  P2.2  P2.1  P2.0  │
    │   │     │     │     │     │     │     │     │   │
    │  A15   A14   A13   A12   A11   A10   A9    A8   │
    │                                             │
    │  Upper address byte for 64KB addressing     │
    └─────────────────────────────────────────────┘

    For addresses 0000H-00FFH with MOVX @R0 or @R1:
    • P2 maintains its latch contents (I/O value)
    • Only P0 carries A0-A7 and D0-D7
    
    For MOVX @DPTR (16-bit address):
    • P2 outputs DPH (upper address)
    • P0 outputs DPL (lower address), then data
```

---

## 5. Port 3 (P3)

### 5.1 Port 3 Internal Structure

```
PORT 3 INTERNAL STRUCTURE
━━━━━━━━━━━━━━━━━━━━━━━━━

Most complex port - supports alternate functions:

                        VCC
                         │
                    ┌────┴────┐
                    │Internal │
      Alternate ───►│Pull-up  │◄─── Alternate Function
      Func. Out     │  FET    │     Input
                   └────┬────┘
                        │
                        ├───────────────────► P3.x (Pin)
                        │
       D ──────┬───────┬┤
       Q ──────┼───────┼┤
               │       ││
               ▼       ▼│
          ┌───────┐ ┌──┴──┐
          │Output │ │ AND │◄─── Alternate Function Out
          │Buffer │ └─────┘
          └───┬───┘    │
              │        │
              ▼        ▼
             GND     To Pin

Key Features:
┌────────────────────────────────────────────────────────────────┐
│ • Internal pull-up (quasi-bidirectional)                       │
│ • Port latch AND alternate function control pin output         │
│ • Both must be HIGH for pin to be HIGH                         │
│ • To use alternate function: Set port latch bit = 1            │
│ • Alternate function can override port latch                   │
└────────────────────────────────────────────────────────────────┘
```

### 5.2 Port 3 Alternate Functions

```
PORT 3 ALTERNATE FUNCTIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━

┌──────┬──────┬─────────┬────────────────────────────────────────┐
│ Pin  │ Bit  │ Alt Fn  │ Description                            │
├──────┼──────┼─────────┼────────────────────────────────────────┤
│  10  │ P3.0 │  RXD    │ Serial port Receive Data input         │
│  11  │ P3.1 │  TXD    │ Serial port Transmit Data output       │
│  12  │ P3.2 │  INT0̅   │ External Interrupt 0 input (active low)│
│  13  │ P3.3 │  INT1̅   │ External Interrupt 1 input (active low)│
│  14  │ P3.4 │  T0     │ Timer 0 external clock/count input     │
│  15  │ P3.5 │  T1     │ Timer 1 external clock/count input     │
│  16  │ P3.6 │  WR̅     │ External Data Memory Write strobe      │
│  17  │ P3.7 │  RD̅     │ External Data Memory Read strobe       │
└──────┴──────┴─────────┴────────────────────────────────────────┘

ALTERNATE FUNCTION DIAGRAM:
───────────────────────────

    P3.0 ═══╤═══ RXD  ═══► Serial Data Input
            │              (from RS-232 driver)
            │
    P3.1 ═══╤═══ TXD  ═══► Serial Data Output
            │              (to RS-232 driver)
            │
    P3.2 ═══╤═══ INT0̅ ═══► External Interrupt 0
            │              (switch, sensor)
            │
    P3.3 ═══╤═══ INT1̅ ═══► External Interrupt 1
            │              (switch, sensor)
            │
    P3.4 ═══╤═══ T0   ═══► Timer 0 Counter Input
            │              (pulse counting)
            │
    P3.5 ═══╤═══ T1   ═══► Timer 1 Counter Input
            │              (frequency measurement)
            │
    P3.6 ═══╤═══ WR̅   ═══► External RAM Write
            │              (to RAM W̅E̅)
            │
    P3.7 ═══╤═══ RD̅   ═══► External RAM Read
                           (to RAM O̅E̅)

USING ALTERNATE FUNCTIONS:
─────────────────────────

To use an alternate function, the corresponding
port bit must be set to 1:

    ; Enable serial port
    SETB P3.0         ; RXD enabled
    SETB P3.1         ; TXD enabled
    ; or
    MOV P3, #0FFH     ; All alternate functions enabled
    
    ; Enable external interrupts
    SETB P3.2         ; INT0̅ pin enabled
    SETB P3.3         ; INT1̅ pin enabled
    SETB IE.0         ; Enable INT0 interrupt
    SETB IE.2         ; Enable INT1 interrupt
    SETB EA           ; Enable all interrupts

Note: After RESET, P3 = 0FFH, so all alternate
      functions are enabled by default.
```

---

## 6. Port Programming

### 6.1 Read-Modify-Write Operations

```
READ-MODIFY-WRITE ISSUE
━━━━━━━━━━━━━━━━━━━━━━━

8051 ports have two components:
1. Port LATCH (internal D flip-flop)
2. Port PIN (actual external voltage)

Some instructions read the LATCH, others read the PIN:

┌────────────────────────────────────────────────────────────────┐
│  Instructions that READ THE LATCH (Safe for R-M-W):           │
│                                                                │
│  ANL Px, A          ORL Px, A          XRL Px, A              │
│  ANL Px, #data      ORL Px, #data      XRL Px, #data          │
│  JBC Px.y, label    CPL Px.y           INC Px                 │
│  CLR Px.y           SETB Px.y          DEC Px                 │
│  DJNZ Px, label     MOV Px.y, C        (and more...)          │
│                                                                │
│  These are SAFE - they modify what's intended.                │
└────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────┐
│  Instructions that READ THE PIN:                               │
│                                                                │
│  MOV A, Px          MOV C, Px.y                               │
│  JB Px.y, label     JNB Px.y, label                           │
│                                                                │
│  These read actual pin voltage - OK for input.                │
└────────────────────────────────────────────────────────────────┘

EXAMPLE PROBLEM:
───────────────
    ; P1 connected to open-drain device that pulls some pins low
    
    MOV P1, #0FFH     ; Write 1s to latch
    ; External device pulls P1.0 low
    MOV A, P1         ; A = actual pin values = FEH (bit 0 is low)
    MOV P1, A         ; PROBLEM! Writes FEH to latch!
                      ; Now latch bit 0 = 0, stays low forever!

SOLUTION - Use read-modify-write:
─────────────────────────────────
    ; Instead of:
    MOV A, P1         ; Read PIN
    ANL A, #0FEH      ; Mask
    MOV P1, A         ; Write - WRONG!
    
    ; Use:
    ANL P1, #0FEH     ; Reads LATCH, modifies, writes back
                      ; Correct behavior!
```

### 6.2 Input/Output Examples

```
COMPLETE I/O PROGRAMMING EXAMPLES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. Basic Output:
───────────────
    ; Send data to Port 1
    MOV P1, #55H      ; P1 = 01010101
    MOV P1, #0AAH     ; P1 = 10101010
    
    ; Individual bit control
    SETB P1.0         ; Set bit 0
    CLR P1.7          ; Clear bit 7
    CPL P1.4          ; Complement bit 4

2. Basic Input:
──────────────
    ; Read from Port 2
    MOV P2, #0FFH     ; MUST write 1s first!
    MOV A, P2         ; Read port pins
    
    ; Read individual bit
    SETB P1.3         ; Enable input on bit 3
    JB P1.3, HIGH     ; Jump if pin is HIGH
    JNB P1.3, LOW     ; Jump if pin is LOW

3. Mixed I/O (Some bits in, some out):
─────────────────────────────────────
    ; P1.0-P1.3 = Output (LEDs)
    ; P1.4-P1.7 = Input (Switches)
    
    MOV P1, #0F0H     ; Lower nibble LOW (LEDs off)
                      ; Upper nibble HIGH (enable input)
    
    ; Read switches and drive LEDs
    MOV A, P1         ; Read all bits
    SWAP A            ; Swap nibbles
    ANL A, #0FH       ; Mask upper nibble
    ANL P1, #0F0H     ; Clear LED bits (keep input bits)
    ORL P1, A         ; Set LED bits from switch values

4. Waiting for Input Change:
───────────────────────────
    ; Wait for button press (active low)
    SETB P3.2         ; Enable input
WAIT_PRESS:
    JB P3.2, WAIT_PRESS    ; Loop while HIGH
    
    ; Wait for button release
WAIT_RELEASE:
    JNB P3.2, WAIT_RELEASE ; Loop while LOW

5. Debounced Switch Reading:
───────────────────────────
    ; Read switch with debounce delay
READ_SWITCH:
    SETB P1.0         ; Enable input
    JB P1.0, NOT_PRESSED
    
    CALL DELAY_10MS   ; Debounce delay
    JB P1.0, NOT_PRESSED  ; Noise spike
    
    ; Genuine press detected
    SETB F0           ; Set flag
    SJMP DONE
    
NOT_PRESSED:
    CLR F0            ; Clear flag
DONE:
    RET
```

---

## 7. Electrical Characteristics

```
PORT ELECTRICAL SPECIFICATIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────────────────┬──────────┬──────────────────────────────────┐
│ Parameter       │ Value    │ Notes                            │
├─────────────────┼──────────┼──────────────────────────────────┤
│ VOL (Output Low)│ 0.45V max│ At IOL = 1.6mA (P1, P2, P3)     │
│                 │          │ At IOL = 3.2mA (P0 bus mode)    │
├─────────────────┼──────────┼──────────────────────────────────┤
│ VOH (Output High)│ 2.4V min│ At IOH = -60µA (P1, P2, P3)     │
│ (with pull-up)  │          │ P0 needs external pull-up       │
├─────────────────┼──────────┼──────────────────────────────────┤
│ VIL (Input Low) │ 0.8V max │ Voltage recognized as LOW       │
├─────────────────┼──────────┼──────────────────────────────────┤
│ VIH (Input High)│ 2.0V min │ Voltage recognized as HIGH      │
├─────────────────┼──────────┼──────────────────────────────────┤
│ IOL (Sink Current)│ 1.6mA  │ Per pin (P1, P2, P3)            │
│                 │ 3.2mA    │ Per pin (P0 bus mode)           │
├─────────────────┼──────────┼──────────────────────────────────┤
│ Total Port IOL  │ 15mA     │ Maximum per port                │
├─────────────────┼──────────┼──────────────────────────────────┤
│ Pull-up Current │ ~60µA    │ Internal FET pull-up            │
└─────────────────┴──────────┴──────────────────────────────────┘

DRIVING LEDS:
────────────

For LED interfacing:

    VCC (5V)            Current calculation:
     │                  I = (VCC - Vf - VOL) / R
    ┌┴┐                 I = (5 - 2 - 0.45) / 330
  R │ │ 330Ω            I ≈ 7.7mA
    └┬┘                 
     │                  But port can only sink 1.6mA!
    ─┼─                 Use transistor driver for more current
    \│/ LED (Vf≈2V)     Or use LED with lower current (2mA LEDs)
     ▼                  Or use buffer IC (74LS244, ULN2803)
     │
   P1.x ─┘

For high current loads:
                         VCC
                          │
    P1.x ───[1K]───┬───┐ Load
                   │   ├──┤
                   │  \│  │
                  ─┴─ ─┴──┘
                  GND    NPN (2N2222)
                         or ULN2803 driver
```

---

## 📋 Summary Table

| Port | Address | Pull-up | I/O | Alt Function | Bus Function |
|------|---------|---------|-----|--------------|--------------|
| P0 | 80H | None (open drain) | 8-bit | None | AD0-AD7 |
| P1 | 90H | Internal (~30K) | 8-bit | T2, T2EX (8052) | None |
| P2 | A0H | Internal (~30K) | 8-bit | None | A8-A15 |
| P3 | B0H | Internal (~30K) | 8-bit | RXD,TXD,INT,T0,T1,RD,WR | None |

---

## ❓ Quick Revision Questions

1. **Why does Port 0 need external pull-up resistors?**
   <details>
   <summary>Show Answer</summary>
   Port 0 has open-drain outputs with no internal pull-ups. The upper FET only activates during bus operations. For general I/O, external 10K resistors are needed to pull pins HIGH when the N-FET is off.
   </details>

2. **What must you do before reading an I/O port as input?**
   <details>
   <summary>Show Answer</summary>
   Write 1s to the port bits you want to read. This turns off the output N-FET and allows external devices to control the pin voltage. Without writing 1s, a 0 in the latch holds the pin LOW.
   </details>

3. **List all alternate functions of Port 3.**
   <details>
   <summary>Show Answer</summary>
   P3.0=RXD (Serial In), P3.1=TXD (Serial Out), P3.2=INT0̅ (External Interrupt 0), P3.3=INT1̅ (External Interrupt 1), P3.4=T0 (Timer 0 Input), P3.5=T1 (Timer 1 Input), P3.6=WR̅ (RAM Write), P3.7=RD̅ (RAM Read).
   </details>

4. **What is a "Read-Modify-Write" instruction and why is it important?**
   <details>
   <summary>Show Answer</summary>
   R-M-W instructions (ANL, ORL, XRL to ports, SETB, CLR, CPL on port bits) read the port LATCH, not the PIN. This is important because if an external device pulls a pin low and you use MOV to read/write, you might accidentally write the wrong value back to the latch.
   </details>

5. **How much current can a port pin sink and source?**
   <details>
   <summary>Show Answer</summary>
   Ports 1, 2, 3 can sink 1.6mA per pin (maximum 15mA per port). They can source only about 60µA (weak internal pull-up). Port 0 can sink 3.2mA in bus mode but cannot source any current without external pull-ups.
   </details>

6. **What is the difference between Port 2 as I/O and as address bus?**
   <details>
   <summary>Show Answer</summary>
   As I/O: Port 2 acts like Port 1 with internal pull-ups for general purpose I/O. As Address Bus: During external memory access with MOVX @DPTR, Port 2 outputs the high byte (A8-A15) of the 16-bit address from DPH, overriding the latch contents temporarily.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [7.4 Registers & PSW](04-registers-psw.md) | [Unit 7 Index](README.md) | [7.6 Addressing Modes](06-addressing-modes.md) |

---

*[← Previous: Registers & PSW](04-registers-psw.md) | [Next: Addressing Modes →](06-addressing-modes.md)*
