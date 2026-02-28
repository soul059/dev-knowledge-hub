# Chapter 6.6: DMA and 8257 Controller

## 📚 Chapter Overview

Direct Memory Access (DMA) allows data transfer between memory and I/O devices without CPU intervention. This chapter covers DMA concepts, the 8257 DMA controller, and its programming for high-speed data transfers.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Understand DMA concept and benefits
- Know 8257 DMA controller architecture
- Program 8257 for data transfers
- Design DMA-based interfaces
- Compare DMA with other transfer methods

---

## 1. DMA Fundamentals

### 1.1 Why DMA?

```
DATA TRANSFER METHODS COMPARISON
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Programmed I/O:
┌──────────────────────────────────────────────────────────────┐
│  CPU reads byte → CPU writes byte → Repeat                   │
│                                                              │
│  Memory ◄──── CPU ────► I/O Device                          │
│          ←───────────────→                                   │
│          (CPU involved 100%)                                 │
│                                                              │
│  Speed: ~10 KB/s (CPU bottleneck)                           │
└──────────────────────────────────────────────────────────────┘

DMA Transfer:
┌──────────────────────────────────────────────────────────────┐
│  CPU sets up DMA → DMA transfers data → CPU continues        │
│                                                              │
│  Memory ◄─────────────────► I/O Device                      │
│              ↑                                               │
│          DMA Controller                                      │
│          (CPU free for other tasks)                          │
│                                                              │
│  Speed: 1-10 MB/s (Memory speed limited)                    │
└──────────────────────────────────────────────────────────────┘

CPU Involvement:
• Programmed I/O: 100% (completely busy during transfer)
• Interrupt I/O:  ~10% (handles each byte interrupt)
• DMA:           ~1% (setup and completion only)
```

### 1.2 DMA Transfer Types

```
DMA TRANSFER MODES
━━━━━━━━━━━━━━━━━━

1. BURST MODE (Block Transfer):
   • DMA takes bus control
   • Transfers entire block
   • CPU halted during transfer
   
   CPU ────────┐     ┌────────────────────
               └─────┘
   DMA ────────      ┌─────┐
               ──────┘     └─────────────
        │◄───────────────►│
           Block transfer

2. CYCLE STEALING:
   • DMA takes one bus cycle
   • CPU continues between transfers
   • Slower but CPU not halted
   
   CPU ──┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─────
         └─┘ └─┘ └─┘ └─┘ └─┘ └─┘
   DMA ──  ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─
         ──┘ └─┘ └─┘ └─┘ └─┘ └─┘ └────
           │   │   │   │   │   │
           Single byte transfers

3. TRANSPARENT DMA:
   • DMA uses only CPU's idle cycles
   • No CPU slow-down
   • Complex timing, limited throughput
```

### 1.3 DMA Operation Steps

```
DMA TRANSFER SEQUENCE
━━━━━━━━━━━━━━━━━━━━━

┌────────────────────────────────────────────────────────────┐
│ 1. CPU Programs DMA Controller:                            │
│    • Starting address                                      │
│    • Byte count                                            │
│    • Transfer direction (read/write)                       │
│    • DMA mode                                              │
└───────────────────────────────────────────────────────────┬┘
                                                             │
┌────────────────────────────────────────────────────────────▼┐
│ 2. I/O Device Requests DMA:                                 │
│    • Device asserts DREQ (DMA Request)                      │
│    • DMA controller receives request                        │
└───────────────────────────────────────────────────────────┬─┘
                                                             │
┌────────────────────────────────────────────────────────────▼┐
│ 3. DMA Controller Requests Bus:                             │
│    • DMA asserts HRQ (Hold Request) to CPU                  │
│    • CPU completes current cycle                            │
│    • CPU asserts HLDA (Hold Acknowledge)                    │
│    • CPU tri-states buses                                   │
└───────────────────────────────────────────────────────────┬─┘
                                                             │
┌────────────────────────────────────────────────────────────▼┐
│ 4. DMA Performs Transfer:                                   │
│    • DMA drives address bus                                 │
│    • DMA generates read/write signals                       │
│    • Data flows: Memory ↔ I/O directly                     │
│    • DMA asserts DACK (DMA Acknowledge) to device          │
└───────────────────────────────────────────────────────────┬─┘
                                                             │
┌────────────────────────────────────────────────────────────▼┐
│ 5. DMA Updates Counters:                                    │
│    • Increment/decrement address                            │
│    • Decrement byte count                                   │
│    • If count = 0, assert TC (Terminal Count)              │
└───────────────────────────────────────────────────────────┬─┘
                                                             │
┌────────────────────────────────────────────────────────────▼┐
│ 6. DMA Releases Bus:                                        │
│    • DMA de-asserts HRQ                                     │
│    • CPU de-asserts HLDA                                    │
│    • CPU resumes operation                                  │
└────────────────────────────────────────────────────────────┘
```

---

## 2. 8257 DMA Controller

### 2.1 Features

```
8257 DMA CONTROLLER FEATURES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
• 4 independent DMA channels
• Priority (fixed or rotating)
• 16-bit address capability (64KB)
• 14-bit transfer count (16KB max)
• Auto-reload capability
• Read, write, and verify modes
• Compatible with 8080/8085
• 40-pin DIP package
```

### 2.2 Block Diagram

```
                      8257 BLOCK DIAGRAM
    ═══════════════════════════════════════════════════════════
    
    ┌───────────────────────────────────────────────────────────┐
    │                        8257 DMA                           │
    │                                                           │
    │   ┌────────────────────────────────────────────────────┐ │
    │   │              4 DMA Channels                        │ │
    │   │                                                    │ │
    │   │  ┌─────────────────┐  ┌─────────────────┐         │ │
    │   │  │   Channel 0     │  │   Channel 1     │         │ │
    │   │  │ Address Register│  │ Address Register│         │ │
    │   │  │ Count Register  │  │ Count Register  │         │ │
    │   │  └────────┬────────┘  └────────┬────────┘         │ │
    │   │           │                    │                   │ │
    │   │  ┌─────────────────┐  ┌─────────────────┐         │ │
    │   │  │   Channel 2     │  │   Channel 3     │         │ │
    │   │  │ Address Register│  │ Address Register│         │ │
    │   │  │ Count Register  │  │ Count Register  │         │ │
    │   │  └────────┬────────┘  └────────┬────────┘         │ │
    │   │           │                    │                   │ │
    │   └───────────┴────────────────────┴───────────────────┘ │
    │                          │                                │
    │   ┌──────────────────────▼──────────────────────────────┐│
    │   │              Control Logic                          ││
    │   │  • Priority Encoder                                 ││
    │   │  • Mode Register                                    ││
    │   │  • Status Register                                  ││
    │   └─────────────────────────────────────────────────────┘│
    │                          │                                │
    │   ┌──────────────────────▼──────────────────────────────┐│
    │   │              Bus Interface                          ││
    │   │  • Address Buffer                                   ││
    │   │  • Data Buffer                                      ││
    │   │  • Control Signal Generation                        ││
    │   └─────────────────────────────────────────────────────┘│
    │                                                           │
    └───────────────────────────────────────────────────────────┘
    
    External Connections:
    ┌─────────────────────────────────────────────────────────┐
    │  DRQ0-3 ─────► DMA Requests from devices                │
    │  DACK0-3 ◄──── DMA Acknowledges to devices              │
    │  HRQ ────────► Hold Request to CPU                      │
    │  HLDA ◄─────── Hold Acknowledge from CPU                │
    │  A0-A3 ◄────── Register select                          │
    │  A4-A7 ◄─────► Address bus (low nibble out)             │
    │  DB0-7 ◄─────► Data bus                                 │
    │  MEMR̄, MEMW̄ ──► Memory control                          │
    │  IOR̄, IOW̄ ────► I/O control                             │
    └─────────────────────────────────────────────────────────┘
```

### 2.3 Pin Diagram

```
                    8257 PIN DIAGRAM
                   ┌────────┴────────┐
         IOR̄  ───┤ 1            40 ├─── A7
         IOW̄  ───┤ 2            39 ├─── A6
        MEMR̄  ───┤ 3            38 ├─── A5
        MEMW̄  ───┤ 4            37 ├─── A4
          ─── ───┤ 5            36 ├─── TC
        READY ───┤ 6            35 ├─── A3
         HLDA ───┤ 7            34 ├─── A2
        ADSTB ───┤ 8            33 ├─── A1
         AEN  ───┤ 9            32 ├─── A0
          HRQ ───┤ 10           31 ├─── Vcc
          C̄S̄  ───┤ 11           30 ├─── DB0
          CLK ───┤ 12           29 ├─── DB1
        RESET ───┤ 13           28 ├─── DB2
        DACK2 ───┤ 14           27 ├─── DB3
        DACK3 ───┤ 15           26 ├─── DB4
         DRQ3 ───┤ 16           25 ├─── DB5
         DRQ2 ───┤ 17           24 ├─── DB6
         DRQ1 ───┤ 18           23 ├─── DB7
         DRQ0 ───┤ 19           22 ├─── DACK0
          GND ───┤ 20           21 ├─── DACK1
                   └─────────────────┘

Pin Functions:
┌────────────┬────────────────────────────────────────────┐
│ Pin        │ Function                                   │
├────────────┼────────────────────────────────────────────┤
│ DRQ0-DRQ3  │ DMA request inputs from devices           │
│ DACK0-DACK3│ DMA acknowledge outputs to devices        │
│ HRQ        │ Hold request output to CPU                │
│ HLDA       │ Hold acknowledge input from CPU           │
│ A0-A3      │ Register select (from CPU)                │
│ A4-A7      │ Low address out / mid address in          │
│ DB0-DB7    │ Bidirectional data bus                    │
│ MEMR̄, MEMW̄ │ Memory read/write outputs                 │
│ IOR̄, IOW̄   │ I/O read/write outputs                    │
│ ADSTB      │ Address strobe (latch high address)       │
│ AEN        │ Address enable                            │
│ TC         │ Terminal count output                     │
│ READY      │ Ready input for wait states               │
└────────────┴────────────────────────────────────────────┘
```

---

## 3. 8257 Registers

### 3.1 Register Map

```
8257 REGISTER ADDRESSES
━━━━━━━━━━━━━━━━━━━━━━━

┌───────┬────────────────────────────────────────────────┐
│ A3-A0 │ Register (Read/Write)                          │
├───────┼────────────────────────────────────────────────┤
│ 0000  │ Channel 0 Address Register                     │
│ 0001  │ Channel 0 Count Register                       │
│ 0010  │ Channel 1 Address Register                     │
│ 0011  │ Channel 1 Count Register                       │
│ 0100  │ Channel 2 Address Register                     │
│ 0101  │ Channel 2 Count Register                       │
│ 0110  │ Channel 3 Address Register                     │
│ 0111  │ Channel 3 Count Register                       │
│ 1000  │ Mode Set Register (Write only)                 │
│ 1000  │ Status Register (Read only)                    │
└───────┴────────────────────────────────────────────────┘

Address/Count Registers are 16-bit (write LSB first, then MSB)
```

### 3.2 Mode Register

```
MODE SET REGISTER (Write Only, Address 8)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  D7   D6   D5   D4   D3   D2   D1   D0
┌────┬────┬────┬────┬────┬────┬────┬────┐
│AL/D│ TC │ EW │ RP │ EN3│ EN2│ EN1│ EN0│
└────┴────┴────┴────┴────┴────┴────┴────┘
  │    │    │    │    │    │    │    │
  │    │    │    │    │    │    │    └─── Enable Channel 0
  │    │    │    │    │    │    │         0 = Disabled
  │    │    │    │    │    │    │         1 = Enabled
  │    │    │    │    │    │    │
  │    │    │    │    │    │    └──────── Enable Channel 1
  │    │    │    │    │    │
  │    │    │    │    │    └───────────── Enable Channel 2
  │    │    │    │    │
  │    │    │    │    └────────────────── Enable Channel 3
  │    │    │    │
  │    │    │    └─────────────────────── Rotating Priority
  │    │    │                             0 = Fixed (CH0 highest)
  │    │    │                             1 = Rotating
  │    │    │
  │    │    └──────────────────────────── Extended Write
  │    │                                  0 = Normal
  │    │                                  1 = Extended
  │    │
  │    └───────────────────────────────── TC Stop
  │                                       0 = Continue after TC
  │                                       1 = Stop on TC
  │
  └────────────────────────────────────── Auto Load
                                          0 = Disabled
                                          1 = Auto-reload CH2/3
                                              from CH0/1
```

### 3.3 Status Register

```
STATUS REGISTER (Read Only, Address 8)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  D7   D6   D5   D4   D3   D2   D1   D0
┌────┬────┬────┬────┬────┬────┬────┬────┐
│UPDT│ 0  │ 0  │ 0  │TC3 │TC2 │TC1 │TC0 │
└────┴────┴────┴────┴────┴────┴────┴────┘
  │              │    │    │    │    │
  │              │    │    │    │    └─── Channel 0 TC reached
  │              │    │    │    └──────── Channel 1 TC reached
  │              │    │    └───────────── Channel 2 TC reached
  │              │    └────────────────── Channel 3 TC reached
  │              │
  │              └─────────────────────── Always 0
  │
  └────────────────────────────────────── Update flag
                                          1 = CH0 or CH1 in use

Note: Reading status clears TC flags
```

### 3.4 DMA Address and Count Registers

```
ADDRESS REGISTER FORMAT (16-bit)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

First Write (LSB):
  D7   D6   D5   D4   D3   D2   D1   D0
┌────┬────┬────┬────┬────┬────┬────┬────┐
│ A7 │ A6 │ A5 │ A4 │ A3 │ A2 │ A1 │ A0 │
└────┴────┴────┴────┴────┴────┴────┴────┘

Second Write (MSB):
  D7   D6   D5   D4   D3   D2   D1   D0
┌────┬────┬────┬────┬────┬────┬────┬────┐
│A15 │A14 │A13 │A12 │A11 │A10 │ A9 │ A8 │
└────┴────┴────┴────┴────┴────┴────┴────┘


COUNT REGISTER FORMAT (16-bit)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

First Write (LSB):
  D7   D6   D5   D4   D3   D2   D1   D0
┌────┬────┬────┬────┬────┬────┬────┬────┐
│ C7 │ C6 │ C5 │ C4 │ C3 │ C2 │ C1 │ C0 │
└────┴────┴────┴────┴────┴────┴────┴────┘

Second Write (MSB):
  D7   D6   D5   D4   D3   D2   D1   D0
┌────┬────┬────┬────┬────┬────┬────┬────┐
│ M1 │ M0 │C13 │C12 │C11 │C10 │ C9 │ C8 │
└────┴────┴────┴────┴────┴────┴────┴────┘
  └──┬──┘
     │
     └─── Mode bits (00=Verify, 01=Write, 10=Read)
          
          Write: Memory → I/O (DMA reads memory)
          Read:  I/O → Memory (DMA writes memory)
          Verify: No transfer (address generation only)

Count is 14 bits: Maximum 16,384 bytes (0000-3FFF)
Actual transfers = Count + 1
```

---

## 4. DMA Transfer Modes

### 4.1 Read Mode (I/O to Memory)

```
DMA READ MODE (I/O → Memory)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Device sends data TO memory

          ┌─────────┐        ┌─────────┐
          │   I/O   │        │ Memory  │
          │ Device  │        │         │
          └────┬────┘        └────┬────┘
               │                  │
               │                  │
               │    ┌────────────┐│
               │    │    8257    ││
               │    │    DMA     ││
               └───►│   Read     │┘
                    │            ├──► Data to Memory
            DRQ ───►│            │
           DACK ◄───│            │
                    │  IOR̄       │──► (Reads from I/O)
                    │  MEMW̄      │──► (Writes to Memory)
                    └────────────┘

Mode bits = 10 (Read)
Use case: Receiving data from disk, ADC, serial port
```

### 4.2 Write Mode (Memory to I/O)

```
DMA WRITE MODE (Memory → I/O)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Memory sends data TO device

          ┌─────────┐        ┌─────────┐
          │ Memory  │        │   I/O   │
          │         │        │ Device  │
          └────┬────┘        └────┬────┘
               │                  │
               │                  │
               │    ┌────────────┐│
               │    │    8257    ││
               │    │    DMA     ││
               └───►│   Write    │┘
                    │            ├──► Data to I/O
            DRQ ───►│            │
           DACK ◄───│            │
                    │  MEMR̄      │──► (Reads from Memory)
                    │  IOW̄       │──► (Writes to I/O)
                    └────────────┘

Mode bits = 01 (Write)
Use case: Sending data to disk, DAC, display
```

### 4.3 Verify Mode

```
DMA VERIFY MODE (No Transfer)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

• Generates addresses only
• No actual data transfer
• Used for testing or CRC checking

Mode bits = 00 (Verify)
```

---

## 5. Programming Examples

### 5.1 Initialize 8257 for Memory-to-I/O Transfer

```asm
;------------------------------------------------
; 8257 DMA Transfer: Memory to I/O (Write Mode)
; Channel 0: 256 bytes from 2000H
;------------------------------------------------

DMA_CH0_ADDR EQU 00H      ; Channel 0 address reg
DMA_CH0_CNT  EQU 01H      ; Channel 0 count reg
DMA_MODE     EQU 08H      ; Mode set register

; Source address = 2000H
; Count = 256 bytes (FFH, since actual = count+1)
; Mode = Write (01)

INIT_DMA:
        ; Disable DMA first
        MVI A, 00H
        OUT DMA_MODE
        
        ; Set Channel 0 address = 2000H
        MVI A, 00H        ; LSB of address
        OUT DMA_CH0_ADDR
        MVI A, 20H        ; MSB of address
        OUT DMA_CH0_ADDR
        
        ; Set Channel 0 count
        ; Count = FFH (256 bytes)
        ; Mode bits = 01 (Write mode)
        ; Full word: 01FF (mode 01, count 00FF)
        MVI A, 0FFH       ; LSB of count
        OUT DMA_CH0_CNT
        MVI A, 40H        ; MSB: 0100 0000 (Write mode + 00)
        OUT DMA_CH0_CNT
        
        ; Enable Channel 0
        MVI A, 01H        ; Enable CH0 only
        OUT DMA_MODE
        
        RET
```

### 5.2 DMA Read Transfer (I/O to Memory)

```asm
;------------------------------------------------
; 8257 DMA Transfer: I/O to Memory (Read Mode)
; Channel 1: 512 bytes to 3000H
;------------------------------------------------

DMA_CH1_ADDR EQU 02H
DMA_CH1_CNT  EQU 03H
DMA_MODE     EQU 08H
DMA_STATUS   EQU 08H

INIT_DMA_READ:
        ; Set Channel 1 address = 3000H
        MVI A, 00H        ; LSB
        OUT DMA_CH1_ADDR
        MVI A, 30H        ; MSB
        OUT DMA_CH1_ADDR
        
        ; Set Channel 1 count = 512 (1FFH)
        ; Mode = Read (10)
        ; Full word: 81FF (mode 10, count 01FF)
        MVI A, 0FFH       ; LSB
        OUT DMA_CH1_CNT
        MVI A, 81H        ; MSB: 1000 0001
        OUT DMA_CH1_CNT
        
        ; Enable Channel 1 with TC stop
        MVI A, 42H        ; TC stop + Enable CH1
        OUT DMA_MODE
        
        RET

; Check if transfer complete
CHECK_DMA:
        IN  DMA_STATUS
        ANI 02H           ; Check TC1 bit
        RZ                ; Not done, return
        ; Transfer complete
        RET
```

### 5.3 8086 DMA Programming

```asm
;------------------------------------------------
; 8257 with 8086: Setup floppy disk DMA
;------------------------------------------------
.MODEL SMALL
.CODE

DMA_CH2_ADDR EQU 04H
DMA_CH2_CNT  EQU 05H
DMA_MODE     EQU 08H

; Transfer 512 bytes (one sector) to 0000:8000
SETUP_FLOPPY_DMA PROC
        ; Channel 2 for floppy
        
        ; Set address = 8000H
        MOV AL, 00H
        OUT DMA_CH2_ADDR, AL
        MOV AL, 80H
        OUT DMA_CH2_ADDR, AL
        
        ; Set count = 511 (1FFH) for 512 bytes
        ; Read mode (I/O to memory) = 10
        MOV AL, 0FFH          ; LSB
        OUT DMA_CH2_CNT, AL
        MOV AL, 81H           ; MSB: mode 10, count 01
        OUT DMA_CH2_CNT, AL
        
        ; Enable CH2 with fixed priority
        MOV AL, 04H           ; Enable CH2
        OUT DMA_MODE, AL
        
        RET
SETUP_FLOPPY_DMA ENDP
```

### 5.4 Auto-reload Mode

```asm
;------------------------------------------------
; 8257 Auto-reload: Continuous transfers
; CH0/CH1 reload CH2/CH3 on TC
;------------------------------------------------

; Setup CH0 as template for CH2
SETUP_AUTOLOAD:
        ; CH0 = template (address 1000H, count 100H)
        MVI A, 00H
        OUT 00H           ; CH0 addr LSB
        MVI A, 10H
        OUT 00H           ; CH0 addr MSB
        
        MVI A, 0FFH
        OUT 01H           ; CH0 count LSB
        MVI A, 40H        ; Write mode
        OUT 01H           ; CH0 count MSB
        
        ; Copy to CH2
        MVI A, 00H
        OUT 04H           ; CH2 addr LSB
        MVI A, 10H
        OUT 04H           ; CH2 addr MSB
        
        MVI A, 0FFH
        OUT 05H           ; CH2 count LSB
        MVI A, 40H
        OUT 05H           ; CH2 count MSB
        
        ; Enable with auto-load
        MVI A, 84H        ; Auto-load + Enable CH2
        OUT 08H
        
        RET
```

---

## 6. 8257 Interfacing

```
8257 WITH 8085 INTERFACE
━━━━━━━━━━━━━━━━━━━━━━━━

     8085                           8257
    ┌─────┐                        ┌─────┐
    │     │  D0-D7                 │     │
    │ AD0 ├─────────────┬──────────┤ DB0 │
    │  .  │             │          │  .  │
    │ AD7 ├─────────────┤ 74LS373  │ DB7 │
    │     │             │ (Latch)  │     │
    │ ALE ├─────────────┤ G        │     │
    │     │             └──────────┤     │
    │     │                        │     │
    │ A8  ├────────────────────────┤ A0  │
    │ A9  ├────────────────────────┤ A1  │
    │ A10 ├────────────────────────┤ A2  │
    │ A11 ├────────────────────────┤ A3  │
    │     │                        │     │
    │ RD̄  ├────────────────────────┤ IOR̄ │
    │ WR̄  ├────────────────────────┤ IOW̄ │
    │     │                        │     │
    │HOLD │◄───────────────────────┤ HRQ │
    │HLDA ├────────────────────────┤HLDA │
    │     │                        │     │
    │     │     ┌─────────┐        │     │
    │ A15 ├─────┤ Address │        │     │
    │ A14 ├─────┤ Decoder ├────────┤ C̄S̄  │
    │IO/M̄ ├─────┤         │        │     │
    │     │     └─────────┘        │     │
    │     │                        │     │
    │RESET├────────────────────────┤RESET│
    │ OUT │                        │     │
    └─────┘                        └─────┘
    
    During DMA:
    • 8257 outputs A0-A7 on DB lines
    • 8257 outputs A8-A15 on A4-A7 (latched via ADSTB)
    • AEN disables CPU address buffers
```

---

## 📋 Summary Table

| Feature | 8257 | 8237 (Enhanced) |
|---------|------|-----------------|
| Channels | 4 | 4 |
| Address Bits | 16 (64KB) | 16 (64KB)* |
| Max Count | 16KB per channel | 64KB per channel |
| Modes | Read, Write, Verify | + Block, Cascade |
| Priority | Fixed, Rotating | Fixed, Rotating |
| Auto-reload | Yes (CH2/3) | Yes (all) |
| Used in | 8080/8085 systems | IBM PC/XT/AT |

*8237 uses page registers for 20-bit addressing in PC

---

## ❓ Quick Revision Questions

1. **What is the advantage of DMA over programmed I/O?**
   <details>
   <summary>Show Answer</summary>
   DMA transfers data directly between memory and I/O without CPU involvement. This frees the CPU for other tasks and achieves much higher transfer rates (memory speed vs. instruction speed). DMA is essential for high-speed devices like disks.
   </details>

2. **What is the difference between DMA read and write modes?**
   <details>
   <summary>Show Answer</summary>
   DMA Read: Data flows from I/O device TO memory. DMA generates MEMW̄ and IOR̄.
   DMA Write: Data flows from memory TO I/O device. DMA generates MEMR̄ and IOW̄.
   Note: Named from memory's perspective.
   </details>

3. **What signals are involved in DMA bus arbitration?**
   <details>
   <summary>Show Answer</summary>
   HRQ (Hold Request): DMA controller requests the bus from CPU.
   HLDA (Hold Acknowledge): CPU grants the bus and tri-states its outputs.
   DREQ (DMA Request): I/O device requests a DMA transfer.
   DACK (DMA Acknowledge): DMA acknowledges the requesting device.
   </details>

4. **Why is the count value one less than the actual transfer count?**
   <details>
   <summary>Show Answer</summary>
   The 8257 counts down from the loaded value to zero. A count of 0 means 1 transfer, count of 1 means 2 transfers, etc. This allows the full range 1-16384 with 14 bits (0000-3FFF). Terminal count occurs when counter reaches 0000.
   </details>

5. **What is auto-reload mode and when is it useful?**
   <details>
   <summary>Show Answer</summary>
   In auto-reload mode, when channel 2 or 3 reaches terminal count, it automatically reloads from channel 0 or 1 respectively. This is useful for continuous transfers (like display refresh or audio streaming) without CPU intervention.
   </details>

6. **What does the TC (Terminal Count) signal indicate?**
   <details>
   <summary>Show Answer</summary>
   TC goes active when a DMA channel's count register decrements to zero, indicating the transfer is complete. It can trigger an interrupt to notify the CPU, and optionally stop further DMA on that channel (if TC Stop is enabled in mode register).
   </details>

---

## 🧭 Navigation

| Previous | Up | Next Unit |
|----------|-----|-----------|
| [6.5 8259 PIC](05-8259-pic.md) | [Unit 6 Index](README.md) | [Unit 7: 8051 Architecture](../07-8051-Architecture/README.md) |

---

*[← Previous: 8259 PIC](05-8259-pic.md) | [Next Unit: 8051 Architecture →](../07-8051-Architecture/README.md)*
