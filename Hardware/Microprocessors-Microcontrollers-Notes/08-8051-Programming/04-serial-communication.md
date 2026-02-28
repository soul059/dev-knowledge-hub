# Chapter 8.4: 8051 Serial Communication

## 📚 Chapter Overview

This chapter covers the 8051 serial communication subsystem in detail, including UART operation, all serial modes, baud rate calculations, and practical programming examples.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Understand serial vs parallel communication
- Configure 8051 UART for different modes
- Calculate baud rates accurately
- Write programs for serial data transmission and reception
- Interface with RS-232 devices

---

## 1. Serial Communication Fundamentals

### 1.1 Serial vs Parallel Communication

```
SERIAL vs PARALLEL COMMUNICATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PARALLEL COMMUNICATION:
──────────────────────

    Sender                              Receiver
    ┌──────┐  D0 ─────────────────────  ┌──────┐
    │      │  D1 ─────────────────────  │      │
    │      │  D2 ─────────────────────  │      │
    │      │  D3 ─────────────────────  │      │
    │  TX  │  D4 ─────────────────────  │  RX  │
    │      │  D5 ─────────────────────  │      │
    │      │  D6 ─────────────────────  │      │
    │      │  D7 ─────────────────────  │      │
    └──────┘  GND ────────────────────  └──────┘

    • 8 data lines (or more)
    • Fast (all bits at once)
    • Short distances only
    • More wires = more cost
    • Used: Printer ports, memory buses

SERIAL COMMUNICATION:
────────────────────

    Sender                              Receiver
    ┌──────┐                            ┌──────┐
    │      │  TXD ──────────────────── │      │
    │  TX  │                      RXD  │  RX  │
    │      │  GND ──────────────────── │      │
    └──────┘                            └──────┘

    Data: 01101001 sent as:
    ──┐ ┌─┐ ┌───┐ ┌─┐ ┌──
      └─┘ └─┘   └─┘ └─┘
     1  0  0  1  0  1  1  0

    • 1-2 data lines
    • Slower (one bit at a time)
    • Long distances possible
    • Fewer wires = less cost
    • Used: USB, RS-232, Ethernet
```

### 1.2 Asynchronous Serial Format

```
ASYNCHRONOUS SERIAL DATA FORMAT (UART)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Standard 8-N-1 Format (8 data, No parity, 1 stop):

    Idle  Start    D0   D1   D2   D3   D4   D5   D6   D7   Stop  Idle
     │      │      │    │    │    │    │    │    │    │     │     │
─────┐      ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌───────────
     │      │    │ │    │ │    │ │    │ │    │ │    │ │    │ │
     └──────┘    └─┘    └─┘    └─┘    └─┘    └─┘    └─┘    └─┘
     
     │◄─────────────── 10 bit times ──────────────────────►│

FRAME COMPONENTS:
────────────────
• Idle Line: HIGH when no data
• Start Bit: LOW (signals data coming)
• Data Bits: 8 bits, LSB first
• Parity Bit: Optional error checking
• Stop Bit(s): HIGH (1 or 2 bits)

EXAMPLE: Sending 'A' (ASCII 41H = 0100 0001)
───────────────────────────────────────────

                     LSB                    MSB
    Idle Start  1    0    0    0    0    0    1    0   Stop Idle
─────┐     ┌───┐    ┌───────────────────────┐    ┌───┐    ┌─────
     │     │   │    │                       │    │   │    │
     └─────┘   └────┘                       └────┘   └────┘
      Start  D0=1 D1=0 D2=0 D3=0 D4=0 D5=0 D6=1 D7=0 Stop

Time per bit = 1 / Baud Rate
At 9600 baud: 1/9600 = 104.16 µs per bit
Frame time = 10 × 104.16 µs = 1.04 ms
Characters per second ≈ 960 (at 9600 baud)
```

### 1.3 Baud Rate

```
BAUD RATE CONCEPTS
━━━━━━━━━━━━━━━━━━

Baud Rate = number of signal changes per second
          = bits per second (for binary signaling)

COMMON BAUD RATES:
─────────────────

┌───────────┬─────────────┬──────────────────────────────┐
│ Baud Rate │ Bit Time    │ Common Use                   │
├───────────┼─────────────┼──────────────────────────────┤
│    300    │  3333 µs    │ Legacy modems               │
│   1200    │   833 µs    │ Old serial devices          │
│   2400    │   417 µs    │ Legacy equipment            │
│   4800    │   208 µs    │ Older industrial            │
│   9600    │   104 µs    │ Standard/Default            │
│  19200    │    52 µs    │ Faster serial               │
│  38400    │    26 µs    │ High speed serial           │
│  57600    │    17 µs    │ High speed                  │
│ 115200    │   8.7 µs    │ Maximum common rate         │
└───────────┴─────────────┴──────────────────────────────┘

DATA RATE vs BAUD RATE:
──────────────────────
At 9600 baud with 8-N-1:
  Actual data rate = 9600 × 8/10 = 7680 bits/sec
                   = 960 bytes/sec
  (10 bits per byte: 1 start + 8 data + 1 stop)
```

---

## 2. 8051 Serial Port

### 2.1 Serial Port Hardware

```
8051 SERIAL PORT ARCHITECTURE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌────────────────────────────────────────────────────────────────┐
│                                                                │
│      ┌─────────────────────────────────────────────────────┐  │
│      │                    SBUF (99H)                        │  │
│      │  ┌─────────────────────┐ ┌─────────────────────────┐│  │
│      │  │  TX Buffer (Write)  │ │  RX Buffer (Read)       ││  │
│      │  │  (Write-only)       │ │  (Read-only)            ││  │
│      │  └──────────┬──────────┘ └──────────┬──────────────┘│  │
│      └─────────────┼────────────────────────┼──────────────┘  │
│                    │                        │                  │
│                    ▼                        ▲                  │
│      ┌─────────────────────┐  ┌─────────────────────┐        │
│      │   TX Shift Register │  │  RX Shift Register  │        │
│      └──────────┬──────────┘  └──────────┬──────────┘        │
│                 │                        │                    │
│                 ▼                        │                    │
│              TXD (P3.1) ◄─────┐    ┌─────► RXD (P3.0)        │
│                              │    │                          │
│    ┌─────────────────────────┴────┴─────────────────────────┐│
│    │                   External Pins                         ││
│    └─────────────────────────────────────────────────────────┘│
│                                                                │
│    Control: SCON (98H) - Mode, flags, enable                  │
│    Baud:    Timer 1 Mode 2 (usually)                          │
│                                                                │
└────────────────────────────────────────────────────────────────┘

KEY POINTS:
• SBUF is two separate registers (TX and RX) at same address
• Writing to SBUF loads TX buffer, starts transmission
• Reading SBUF reads RX buffer
• Full duplex - can transmit and receive simultaneously
• TXD (P3.1) for transmit, RXD (P3.0) for receive
```

### 2.2 SCON Register

```
SCON - SERIAL CONTROL REGISTER (Address 98H)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│SM0  │SM1  │SM2  │ REN │TB8  │RB8  │ TI  │ RI  │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
  D7    D6    D5    D4    D3    D2    D1    D0
  9FH   9EH   9DH   9CH   9BH   9AH   99H   98H  ◄── Bit addresses

BIT FUNCTIONS:
──────────────

SM0, SM1: Serial Mode Select
┌─────┬─────┬──────────────────────────────────────────────────┐
│SM0  │SM1  │ Mode Description                                 │
├─────┼─────┼──────────────────────────────────────────────────┤
│  0  │  0  │ Mode 0: Shift register, fixed baud (fOSC/12)    │
│  0  │  1  │ Mode 1: 8-bit UART, variable baud               │
│  1  │  0  │ Mode 2: 9-bit UART, fixed baud (fOSC/32 or /64) │
│  1  │  1  │ Mode 3: 9-bit UART, variable baud               │
└─────┴─────┴──────────────────────────────────────────────────┘

SM2: Multiprocessor Communication Enable
  Mode 0: Should be 0
  Mode 1: If 1, RI set only if valid stop bit received
  Mode 2/3: If 1, RI set only if RB8=1 (address byte)

REN: Receive Enable
  1 = Enable serial reception
  0 = Disable reception

TB8: 9th Transmitted Bit (Mode 2, 3)
  Used as parity or address/data indicator

RB8: 9th Received Bit (Mode 2, 3)
  Contains 9th bit or stop bit (Mode 1)

TI: Transmit Interrupt Flag
  Set by hardware when transmission complete
  Must be cleared by software

RI: Receive Interrupt Flag
  Set by hardware when reception complete
  Must be cleared by software

COMMON SCON VALUES:
──────────────────

┌────────┬────────────────────────────────────────────────────────┐
│ Value  │ Configuration                                          │
├────────┼────────────────────────────────────────────────────────┤
│  40H   │ Mode 1, 8-bit UART, receive disabled                  │
│  50H   │ Mode 1, 8-bit UART, receive enabled (most common)     │
│  42H   │ Mode 1, receive disabled, TI set (ready to send)      │
│  52H   │ Mode 1, receive enabled, TI set                       │
│  90H   │ Mode 2, 9-bit UART, receive enabled                   │
│  D0H   │ Mode 3, 9-bit UART, receive enabled                   │
└────────┴────────────────────────────────────────────────────────┘
```

---

## 3. Serial Port Modes

### 3.1 Mode 0: Synchronous Shift Register

```
MODE 0: SYNCHRONOUS SHIFT REGISTER
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

• Synchronous mode (clock provided)
• RXD (P3.0) used for data
• TXD (P3.1) used for shift clock
• Fixed baud rate = fOSC / 12
• 8 data bits, LSB first

TIMING DIAGRAM:
──────────────

         ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐
TXD/CLK  ┘  └──┘  └──┘  └──┘  └──┘  └──┘  └──┘  └──┘  └──
         
RXD/Data ──┬──┬──┬──┬──┬──┬──┬──┬──
           D0  D1  D2  D3  D4  D5  D6  D7

@ 12 MHz crystal: Baud = 12MHz/12 = 1 MHz (1 Mbps!)

USE CASE:
• Interface with shift registers (74HC595, 74HC165)
• Fast I/O expansion
• NOT for RS-232 communication

EXAMPLE: Expand output using 74HC595
───────────────────────────────────

    MOV SCON, #00H      ; Mode 0
    MOV SBUF, A         ; Send byte to shift register
    JNB TI, $           ; Wait for completion
    CLR TI
    SETB P1.0           ; Latch data
    CLR P1.0
```

### 3.2 Mode 1: 8-bit UART

```
MODE 1: 8-BIT UART (Most Common)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

• 8 data bits, 1 start, 1 stop (10 bits total)
• Variable baud rate (Timer 1 controlled)
• Full-duplex asynchronous communication
• Standard RS-232 compatible format

FRAME FORMAT:
────────────
┌───────┬────┬────┬────┬────┬────┬────┬────┬────┬────────┐
│ Start │ D0 │ D1 │ D2 │ D3 │ D4 │ D5 │ D6 │ D7 │  Stop  │
│  (0)  │    │    │    │    │    │    │    │    │   (1)  │
└───────┴────┴────┴────┴────┴────┴────┴────┴────┴────────┘

BAUD RATE GENERATION:
───────────────────

Using Timer 1 Mode 2 (8-bit auto-reload):

                    Timer 1 Overflow Rate
Baud Rate = ─────────────────────────────
                         32

Timer 1 Overflow Rate = fOSC / (12 × (256 - TH1))

Therefore:
                          fOSC
Baud Rate = ─────────────────────────────
            12 × 32 × (256 - TH1)

                          fOSC
Baud Rate = ─────────────────────────────
                384 × (256 - TH1)

Solving for TH1:
                      fOSC
TH1 = 256 - ─────────────────
             384 × Baud Rate

COMMON TH1 VALUES:
─────────────────

┌───────────┬──────────────────┬──────────────────┐
│ Baud Rate │ TH1 @ 11.0592MHz │ TH1 @ 12MHz      │
├───────────┼──────────────────┼──────────────────┤
│    1200   │  E8H (-24)       │  E6H (-26)*      │
│    2400   │  F4H (-12)       │  F3H (-13)*      │
│    4800   │  FAH (-6)        │  F9H (-7)*       │
│    9600   │  FDH (-3)        │  FDH (-3)*       │
│   19200   │  FEH (-2)*       │  FFH (-1)*       │
│   57600   │  FFH (-1)        │  Not possible    │
└───────────┴──────────────────┴──────────────────┘
* = Slight error at non-standard crystal

WHY 11.0592 MHz?
───────────────
11.0592 MHz / 384 = 28800 (exact division!)
28800 / 3 = 9600 baud (exact!)
28800 / 1 = 28800 baud
This crystal gives zero baud rate error!
```

### 3.3 Mode 2 and Mode 3: 9-bit UART

```
MODE 2 & 3: 9-BIT UART
━━━━━━━━━━━━━━━━━━━━━━

9 data bits (8 + TB8/RB8), 1 start, 1 stop

FRAME FORMAT:
────────────
┌───────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────────┐
│ Start │ D0 │ D1 │ D2 │ D3 │ D4 │ D5 │ D6 │ D7 │ D8 │  Stop  │
│  (0)  │    │    │    │    │    │    │    │    │TB8 │   (1)  │
└───────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────────┘

MODE 2 (SCON = 80H/90H):
──────────────────────
• Fixed baud rate: fOSC/32 or fOSC/64 (SMOD bit)
• SMOD=0: Baud = fOSC/64 = 187.5 kbps @ 12MHz
• SMOD=1: Baud = fOSC/32 = 375 kbps @ 12MHz
• No Timer 1 needed

MODE 3 (SCON = C0H/D0H):
──────────────────────
• Variable baud rate (same as Mode 1)
• Uses Timer 1 for baud rate
• 9th bit for parity or multiprocessor

9TH BIT USES:
────────────

1. Parity Bit:
   TB8 = Parity of data
   Even parity: TB8 makes total 1s even
   
2. Multiprocessor Communication:
   TB8 = 1: Address byte (all slaves check)
   TB8 = 0: Data byte (only addressed slave receives)
   
3. Wake-up Bit:
   Used to wake slaves from idle

EXAMPLE: Mode 2 with parity
──────────────────────────

    MOV SCON, #90H      ; Mode 2, REN=1
    
SEND_PARITY:
    MOV A, #55H         ; Data to send
    MOV C, P            ; Get parity flag
    MOV TB8, C          ; Set 9th bit = parity
    MOV SBUF, A         ; Send
    JNB TI, $
    CLR TI
    RET
```

### 3.4 SMOD Bit (Baud Rate Doubler)

```
SMOD BIT - BAUD RATE DOUBLER
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Location: PCON.7 (Power Control Register, 87H)
NOT bit-addressable - must use MOV

SMOD = 0: Normal baud rate
SMOD = 1: Double baud rate

Effect on Baud Rate:
──────────────────

Mode 1 & 3:
                   2^SMOD × fOSC
Baud Rate = ─────────────────────────────
             32 × 12 × (256 - TH1)

Mode 2:
                   2^SMOD × fOSC
Baud Rate = ─────────────────────
                     64

┌───────────┬─────────────────┬─────────────────┐
│           │ SMOD = 0        │ SMOD = 1        │
├───────────┼─────────────────┼─────────────────┤
│ Mode 1    │ Normal          │ 2× faster       │
│ Mode 2    │ fOSC/64         │ fOSC/32         │
│ Mode 3    │ Normal          │ 2× faster       │
└───────────┴─────────────────┴─────────────────┘

EXAMPLE: Set SMOD for higher baud rate
─────────────────────────────────────

    MOV A, PCON         ; Read PCON
    ORL A, #80H         ; Set SMOD bit
    MOV PCON, A         ; Write back
    
    ; Or more efficient:
    ORL PCON, #80H      ; Direct set (works on many 8051s)
```

---

## 4. Baud Rate Calculations

### 4.1 Complete Calculation Examples

```
BAUD RATE CALCULATION EXAMPLES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

EXAMPLE 1: 9600 baud @ 11.0592 MHz
──────────────────────────────────

Formula: TH1 = 256 - (fOSC / (384 × Baud))

TH1 = 256 - (11059200 / (384 × 9600))
TH1 = 256 - (11059200 / 3686400)
TH1 = 256 - 3
TH1 = 253 = FDH ✓

Verification:
Baud = 11059200 / (384 × 3) = 11059200 / 1152 = 9600 ✓


EXAMPLE 2: 9600 baud @ 12 MHz (Error Analysis)
─────────────────────────────────────────────

TH1 = 256 - (12000000 / (384 × 9600))
TH1 = 256 - (12000000 / 3686400)
TH1 = 256 - 3.255...
TH1 = 252.74 ≈ 253 = FDH

Actual Baud = 12000000 / (384 × 3) = 10416.67
Error = (10416.67 - 9600) / 9600 × 100% = 8.5% (Too high!)

With TH1 = 252 (FCH):
Actual Baud = 12000000 / (384 × 4) = 7812.5
Error = (9600 - 7812.5) / 9600 × 100% = 18.6% (Worse!)

⚠ 12 MHz is NOT suitable for 9600 baud!


EXAMPLE 3: Using SMOD for higher rates @ 11.0592 MHz
───────────────────────────────────────────────────

For 19200 baud with SMOD = 1:

TH1 = 256 - (2 × 11059200 / (384 × 19200))
TH1 = 256 - (22118400 / 7372800)
TH1 = 256 - 3
TH1 = 253 = FDH

Same TH1 as 9600 baud without SMOD!


EXAMPLE 4: Maximum baud rate
───────────────────────────

With TH1 = FFH (-1) and SMOD = 1:
Baud = (2 × 11059200) / (384 × 1) = 57600 baud

Maximum reliable baud @ 11.0592 MHz = 57600 baud
```

### 4.2 Quick Reference Table

```
BAUD RATE QUICK REFERENCE TABLE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Crystal: 11.0592 MHz (Recommended)
──────────────────────────────────

┌───────────┬─────┬───────────┬───────────┬─────────┐
│ Baud Rate │SMOD │ TH1 (Hex) │ TH1 (Dec) │ Error   │
├───────────┼─────┼───────────┼───────────┼─────────┤
│    1200   │  0  │    E8H    │   -24     │   0%    │
│    2400   │  0  │    F4H    │   -12     │   0%    │
│    4800   │  0  │    FAH    │    -6     │   0%    │
│    9600   │  0  │    FDH    │    -3     │   0%    │
│   19200   │  1  │    FDH    │    -3     │   0%    │
│   38400   │  1  │    FEH    │    -2     │  6.7%*  │
│   57600   │  1  │    FFH    │    -1     │   0%    │
└───────────┴─────┴───────────┴───────────┴─────────┘

Crystal: 12.000 MHz (Common but problematic)
────────────────────────────────────────────

┌───────────┬─────┬───────────┬─────────────────────┐
│ Baud Rate │SMOD │ TH1 (Hex) │ Error               │
├───────────┼─────┼───────────┼─────────────────────┤
│    9600   │  0  │    FDH    │  8.5% (Too high!)   │
│    4800   │  0  │    FAH    │  1.7%               │
│    2400   │  0  │    F4H    │  0.2%               │
└───────────┴─────┴───────────┴─────────────────────┘

Maximum acceptable error: ±3% for reliable communication
```

---

## 5. Programming Examples

### 5.1 Basic Transmission

```
BASIC SERIAL TRANSMISSION
━━━━━━━━━━━━━━━━━━━━━━━━━

; Configure serial port for 9600 baud @ 11.0592 MHz
SERIAL_INIT:
    MOV TMOD, #20H      ; Timer 1 Mode 2
    MOV TH1, #0FDH      ; 9600 baud
    MOV TL1, #0FDH
    MOV SCON, #50H      ; Mode 1, 8-bit, REN enabled
    SETB TR1            ; Start Timer 1
    RET

; Send single byte (in A)
SEND_BYTE:
    MOV SBUF, A         ; Load transmit buffer
    JNB TI, $           ; Wait for transmit complete
    CLR TI              ; Clear transmit flag
    RET

; Send string (DPTR points to null-terminated string)
SEND_STRING:
    CLR A
    MOVC A, @A+DPTR     ; Get character
    JZ SEND_DONE        ; If null, done
    CALL SEND_BYTE      ; Send character
    INC DPTR            ; Next character
    SJMP SEND_STRING
SEND_DONE:
    RET

; Example usage
MAIN:
    CALL SERIAL_INIT
    
    ; Send single character
    MOV A, #'H'
    CALL SEND_BYTE
    
    ; Send string
    MOV DPTR, #MSG
    CALL SEND_STRING
    
    SJMP $

MSG:
    DB 'Hello, World!', 0DH, 0AH, 0  ; CR, LF, NULL
```

### 5.2 Basic Reception

```
BASIC SERIAL RECEPTION
━━━━━━━━━━━━━━━━━━━━━━━

; Receive single byte (returned in A)
RECV_BYTE:
    JNB RI, $           ; Wait for receive complete
    MOV A, SBUF         ; Read received byte
    CLR RI              ; Clear receive flag
    RET

; Receive with timeout (uses Timer 0)
; Returns with C=1 if timeout, C=0 if byte received
RECV_TIMEOUT:
    MOV TMOD, #21H      ; Timer 0 Mode 1, Timer 1 Mode 2
    MOV TH0, #0         ; ~65ms timeout @ 12MHz
    MOV TL0, #0
    CLR TF0
    SETB TR0            ; Start timeout timer
    
WAIT_RECV:
    JB RI, GOT_BYTE     ; Check for received byte
    JNB TF0, WAIT_RECV  ; Check for timeout
    
    ; Timeout occurred
    CLR TR0
    SETB C              ; Set carry = timeout
    RET
    
GOT_BYTE:
    CLR TR0
    MOV A, SBUF
    CLR RI
    CLR C               ; Clear carry = success
    RET

; Echo received characters back
ECHO:
    CALL SERIAL_INIT
    
ECHO_LOOP:
    CALL RECV_BYTE      ; Wait for character
    CALL SEND_BYTE      ; Echo it back
    SJMP ECHO_LOOP
```

### 5.3 Complete Duplex Example

```
FULL DUPLEX COMMUNICATION
━━━━━━━━━━━━━━━━━━━━━━━━━

; Complete serial terminal example
; Receives characters, echoes back, displays on Port 1

    ORG 0000H
    LJMP MAIN

    ORG 0023H           ; Serial interrupt vector
    LJMP SERIAL_ISR

    ORG 0030H
MAIN:
    ; Initialize serial port
    MOV TMOD, #20H      ; Timer 1 Mode 2
    MOV TH1, #0FDH      ; 9600 baud
    MOV TL1, #0FDH
    MOV SCON, #50H      ; Mode 1, REN enabled
    SETB TR1
    
    ; Enable serial interrupt
    SETB ES
    SETB EA
    
    ; Send welcome message
    MOV DPTR, #WELCOME
    CALL SEND_STRING
    
MAIN_LOOP:
    ; Main program continues here
    ; Serial handled by interrupt
    SJMP MAIN_LOOP

SERIAL_ISR:
    PUSH ACC
    PUSH PSW
    
    JNB RI, CHECK_TX    ; Check receive flag
    
    ; Handle received byte
    CLR RI
    MOV A, SBUF         ; Get received byte
    MOV P1, A           ; Display on Port 1
    MOV SBUF, A         ; Echo back
    SJMP SERIAL_EXIT
    
CHECK_TX:
    JNB TI, SERIAL_EXIT
    CLR TI              ; Clear transmit flag
    
SERIAL_EXIT:
    POP PSW
    POP ACC
    RETI

SEND_STRING:
    CLR A
    MOVC A, @A+DPTR
    JZ SEND_DONE
WAIT_TI:
    JNB TI, WAIT_TI     ; Wait for previous TX
    CLR TI
    MOV SBUF, A
    INC DPTR
    SJMP SEND_STRING
SEND_DONE:
    RET

WELCOME:
    DB 'Serial Terminal Ready', 0DH, 0AH, 0
```

### 5.4 Number Transmission

```
SENDING NUMBERS
━━━━━━━━━━━━━━━

; Send hex byte as two ASCII characters
SEND_HEX:
    PUSH ACC
    SWAP A              ; High nibble first
    ANL A, #0FH
    CALL NIBBLE_TO_ASCII
    CALL SEND_BYTE
    POP ACC
    ANL A, #0FH         ; Low nibble
    CALL NIBBLE_TO_ASCII
    CALL SEND_BYTE
    RET

NIBBLE_TO_ASCII:
    ADD A, #'0'         ; Convert 0-9 to ASCII
    CJNE A, #':',  CHECK_LETTER
CHECK_LETTER:
    JC NIBBLE_DONE      ; If < ':', it's 0-9
    ADD A, #7           ; Adjust for A-F
NIBBLE_DONE:
    RET

; Send decimal number (0-255 in A)
SEND_DECIMAL:
    MOV B, #100
    DIV AB              ; A = hundreds
    ADD A, #'0'
    CALL SEND_BYTE
    
    MOV A, B
    MOV B, #10
    DIV AB              ; A = tens
    ADD A, #'0'
    CALL SEND_BYTE
    
    MOV A, B            ; A = ones
    ADD A, #'0'
    CALL SEND_BYTE
    RET

; Example: Send temperature reading
SEND_TEMP:
    MOV A, 30H          ; Get temperature from RAM
    CALL SEND_DECIMAL
    MOV A, #'C'
    CALL SEND_BYTE
    MOV A, #0DH         ; Carriage return
    CALL SEND_BYTE
    MOV A, #0AH         ; Line feed
    CALL SEND_BYTE
    RET
```

---

## 6. RS-232 Interface

```
RS-232 LEVEL CONVERSION
━━━━━━━━━━━━━━━━━━━━━━━

8051 uses TTL levels (0V, 5V)
RS-232 uses ±12V (or ±15V)

VOLTAGE LEVELS:
──────────────

             TTL (8051)       RS-232
    ┌────────────────────┬────────────────────┐
    │ Logic 1: +5V       │ Logic 1: -3 to -15V │
    │ Logic 0: 0V        │ Logic 0: +3 to +15V │
    └────────────────────┴────────────────────┘

    Note: RS-232 is INVERTED!

MAX232 LEVEL CONVERTER:
──────────────────────

                    MAX232
    ┌─────────────────────────────────────┐
    │                                     │
    │    ┌───────────────────────────┐   │
    │    │ C1+   C1-   C2+   C2-     │   │
    │    │  1     3     4     5      │   │
    │    │                           │   │
    │    │ V+    V-   T1out T1in     │   │
    │    │  2     6     14    11     │   │
    │    │                           │   │
    │    │ R1in  R1out T2out T2in    │   │
    │    │  13    12    7     10     │   │
    │    │                           │   │
    │    │      VCC   GND            │   │
    │    │       16    15            │   │
    │    └───────────────────────────┘   │
    │                                     │
    └─────────────────────────────────────┘

CONNECTION DIAGRAM:
─────────────────

    8051          MAX232            DB9 Connector
    ┌────┐        ┌─────┐           ┌─────────┐
    │    │ TXD ───│T1in │───T1out───│ Pin 3   │
    │    │ (P3.1) │     │    (14)   │         │
    │    │        │     │           │         │
    │    │ RXD ───│R1out│───R1in────│ Pin 2   │
    │    │ (P3.0) │ (12)│   (13)    │         │
    │    │        │     │           │         │
    │    │ GND ───│ GND │───────────│ Pin 5   │
    └────┘        └─────┘           └─────────┘

    Note: 4 × 1µF capacitors needed for charge pump
```

---

## 📋 Summary Table

| Mode | Bits | Baud Rate | Formula | Use |
|------|------|-----------|---------|-----|
| 0 | 8 | fOSC/12 | Fixed | Shift register |
| 1 | 8 | Variable | fOSC/(384×(256-TH1)) | Standard UART |
| 2 | 9 | fOSC/32 or /64 | Fixed (SMOD) | Multiprocessor |
| 3 | 9 | Variable | Same as Mode 1 | 9-bit UART |

---

## ❓ Quick Revision Questions

1. **Calculate TH1 for 9600 baud at 11.0592 MHz.**
   <details>
   <summary>Show Answer</summary>
   TH1 = 256 - (11059200 / (384 × 9600)) = 256 - 3 = 253 = FDH
   </details>

2. **Why is 11.0592 MHz preferred over 12 MHz for serial communication?**
   <details>
   <summary>Show Answer</summary>
   11.0592 MHz divides evenly to produce standard baud rates with 0% error. 12 MHz produces 8.5% error at 9600 baud, which exceeds the ±3% tolerance.
   </details>

3. **What is the purpose of the TI and RI flags?**
   <details>
   <summary>Show Answer</summary>
   TI (Transmit Interrupt) is set when byte transmission completes. RI (Receive Interrupt) is set when a byte is received. Both must be cleared by software and can trigger the serial interrupt.
   </details>

4. **How does SMOD affect baud rate?**
   <details>
   <summary>Show Answer</summary>
   SMOD (PCON.7) doubles the baud rate when set to 1. In Mode 1/3, it multiplies the baud by 2. In Mode 2, it changes baud from fOSC/64 to fOSC/32.
   </details>

5. **Why can't we directly connect 8051 to RS-232 port?**
   <details>
   <summary>Show Answer</summary>
   8051 uses TTL levels (0-5V) while RS-232 uses ±12V with inverted logic. Direct connection would damage the 8051. A level converter like MAX232 is needed.
   </details>

6. **What is the difference between Mode 1 and Mode 3?**
   <details>
   <summary>Show Answer</summary>
   Mode 1 transmits 8 data bits. Mode 3 transmits 9 data bits (8 data + TB8). Both use variable baud rate from Timer 1. The 9th bit in Mode 3 is used for parity or multiprocessor communication.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [8.3 Interrupt Programming](03-interrupt-programming.md) | [Unit 8 Index](README.md) | [8.5 Interfacing Examples](05-interfacing-examples.md) |

---

*[← Previous: Interrupt Programming](03-interrupt-programming.md) | [Next: Interfacing Examples →](05-interfacing-examples.md)*
