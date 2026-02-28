# Chapter 8.5: 8051 Interfacing Examples

## 📚 Chapter Overview

This chapter provides practical interfacing examples for the 8051 microcontroller, covering LEDs, switches, seven-segment displays, LCDs, keypads, ADC/DAC, and more.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Interface LEDs and switches with 8051
- Drive seven-segment displays (single and multiplexed)
- Interface character LCDs (16×2)
- Implement keypad scanning
- Interface ADC and DAC for analog signals

---

## 1. LED Interfacing

### 1.1 Single LED

```
SINGLE LED INTERFACING
━━━━━━━━━━━━━━━━━━━━━━

ACTIVE LOW CONNECTION (LED ON when pin = 0):
──────────────────────────────────────────

    +5V
     │
     ├──► LED (Anode)
     │    ┌─►├─┐
     │       │
     └───────┤  330Ω
             │
    P1.0 ────┘

    ; Turn LED ON
    CLR P1.0            ; LOW = LED ON
    
    ; Turn LED OFF
    SETB P1.0           ; HIGH = LED OFF

ACTIVE HIGH CONNECTION (LED ON when pin = 1):
────────────────────────────────────────────

    P1.0 ───┬──► 330Ω ──► LED ──► GND
            │              │
            │          ┌─►├─┘
            │ (Cathode)
            
    Note: May need buffer for higher current
    
    ; Turn LED ON
    SETB P1.0           ; HIGH = LED ON
    
    ; Turn LED OFF
    CLR P1.0            ; LOW = LED OFF

COMPLETE LED BLINK PROGRAM:
─────────────────────────

    ORG 0000H
    
MAIN:
    CPL P1.0            ; Toggle LED
    CALL DELAY_500MS
    SJMP MAIN
    
DELAY_500MS:
    MOV R6, #10         ; 10 × 50ms = 500ms
OUTER:
    MOV TMOD, #01H      ; Timer 0 Mode 1
    MOV TH0, #3CH       ; 50 ms
    MOV TL0, #0B0H
    SETB TR0
    JNB TF0, $
    CLR TR0
    CLR TF0
    DJNZ R6, OUTER
    RET
    
    END
```

### 1.2 LED Array (8 LEDs)

```
8-LED ARRAY INTERFACING
━━━━━━━━━━━━━━━━━━━━━━━

    8051          LED Array (Common Cathode)
    ┌────┐        ┌─────────────────────────────┐
    │    │        │  R    R    R    R    R    R    R    R   │
    │P1.0├────────┤  LED0 LED1 LED2 LED3 LED4 LED5 LED6 LED7│
    │P1.1├────────┤                                          │
    │P1.2├────────┤  (330Ω resistors for each LED)          │
    │P1.3├────────┤                                          │
    │P1.4├────────┤                        Common            │
    │P1.5├────────┤                          GND             │
    │P1.6├────────┤                           │              │
    │P1.7├────────┤───────────────────────────┴──────────────┘
    └────┘

RUNNING LED PATTERN:
───────────────────

    ORG 0000H
    
    MOV A, #01H         ; Start with LED 0
    
RUNNING:
    MOV P1, A           ; Output to LEDs
    CALL DELAY
    RL A                ; Rotate left
    SJMP RUNNING
    
DELAY:
    MOV R7, #200
DLY1:
    MOV R6, #250
DLY2:
    DJNZ R6, DLY2
    DJNZ R7, DLY1
    RET

LED PATTERNS COLLECTION:
───────────────────────

; Pattern 1: Knight Rider (back and forth)
KNIGHT:
    MOV A, #01H
LEFT:
    MOV P1, A
    CALL DELAY
    RL A
    CJNE A, #80H, LEFT
RIGHT:
    MOV P1, A
    CALL DELAY
    RR A
    CJNE A, #01H, RIGHT
    SJMP LEFT

; Pattern 2: Binary counter
BINARY:
    MOV A, #0
CNT_LOOP:
    MOV P1, A
    CALL DELAY
    INC A
    SJMP CNT_LOOP

; Pattern 3: Alternate blink
ALTERNATE:
    MOV P1, #55H        ; 01010101
    CALL DELAY
    MOV P1, #0AAH       ; 10101010
    CALL DELAY
    SJMP ALTERNATE
```

---

## 2. Switch Interfacing

### 2.1 Simple Push Button

```
PUSH BUTTON INTERFACING
━━━━━━━━━━━━━━━━━━━━━━━

ACTIVE LOW (Common configuration):
─────────────────────────────────

    +5V
     │
     ├───┬─── P1.0 (Input)
     │   │
    10K  ┴  Switch
     │   │
     ├───┴─── GND
     │
     
    Switch Open: P1.0 = HIGH (1)
    Switch Pressed: P1.0 = LOW (0)

READING A SWITCH:
────────────────

    ; Check if switch on P1.0 is pressed
    SETB P1.0           ; Configure as input
    JNB P1.0, PRESSED   ; Jump if LOW (pressed)
    ; Not pressed - continue
    SJMP CHECK_DONE
    
PRESSED:
    ; Switch is pressed
    ; Do something
    
CHECK_DONE:

WITH DEBOUNCING:
───────────────

; Mechanical switches bounce for 10-50ms
; Must debounce for reliable detection

READ_SWITCH:
    SETB P1.0           ; Input mode
    JB P1.0, NOT_PRESSED
    
    ; Debounce delay (20ms)
    MOV R7, #20
DEBOUNCE1:
    MOV R6, #250
    DJNZ R6, $
    DJNZ R7, DEBOUNCE1
    
    ; Check again
    JB P1.0, NOT_PRESSED
    
    ; Wait for release
WAIT_RELEASE:
    JNB P1.0, WAIT_RELEASE
    
    ; Debounce release
    MOV R7, #20
DEBOUNCE2:
    MOV R6, #250
    DJNZ R6, $
    DJNZ R7, DEBOUNCE2
    
    ; Valid press detected
    CPL P2.0            ; Toggle LED
    
NOT_PRESSED:
    RET
```

### 2.2 DIP Switch Array

```
DIP SWITCH ARRAY
━━━━━━━━━━━━━━━━

    +5V ──┬──┬──┬──┬──┬──┬──┬──┐
          │  │  │  │  │  │  │  │  10K each
          R  R  R  R  R  R  R  R
          │  │  │  │  │  │  │  │
    P2.0 ─┴──┴──┴──┴──┴──┴──┴──┴─┬─ DIP Switch
          SW0 SW1 ... SW7        │
                                 GND

READ DIP SWITCH:
───────────────

    ; Read all 8 switches
    MOV P2, #0FFH       ; All inputs
    MOV A, P2           ; Read switch states
    CPL A               ; Invert (pressed = 1)
    MOV P1, A           ; Display on LEDs
```

---

## 3. Seven-Segment Display

### 3.1 Single Digit

```
SEVEN-SEGMENT DISPLAY INTERFACING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SEGMENT LAYOUT:
──────────────
        a
      ┌───┐
    f │   │ b
      ├─g─┤
    e │   │ c
      └───┘ • dp
        d

    Common Cathode: Apply HIGH to turn ON segment
    Common Anode: Apply LOW to turn ON segment

CONNECTION (Common Cathode):
──────────────────────────

    8051        Segment
    P1.0 ──R──► a
    P1.1 ──R──► b
    P1.2 ──R──► c
    P1.3 ──R──► d
    P1.4 ──R──► e
    P1.5 ──R──► f
    P1.6 ──R──► g
    P1.7 ──R──► dp
                │
    GND ────────┴── Common

SEGMENT CODES (Common Cathode):
─────────────────────────────

┌───────┬─────────────────────────┬───────────┐
│ Digit │   dp g f e d c b a     │ Hex Code  │
├───────┼─────────────────────────┼───────────┤
│   0   │    0  0 1 1 1 1 1 1    │   3FH     │
│   1   │    0  0 0 0 0 1 1 0    │   06H     │
│   2   │    0  1 0 1 1 0 1 1    │   5BH     │
│   3   │    0  1 0 0 1 1 1 1    │   4FH     │
│   4   │    0  1 1 0 0 1 1 0    │   66H     │
│   5   │    0  1 1 0 1 1 0 1    │   6DH     │
│   6   │    0  1 1 1 1 1 0 1    │   7DH     │
│   7   │    0  0 0 0 0 1 1 1    │   07H     │
│   8   │    0  1 1 1 1 1 1 1    │   7FH     │
│   9   │    0  1 1 0 1 1 1 1    │   6FH     │
│   A   │    0  1 1 1 0 1 1 1    │   77H     │
│   b   │    0  1 1 1 1 1 0 0    │   7CH     │
│   C   │    0  0 1 1 1 0 0 1    │   39H     │
│   d   │    0  1 0 1 1 1 1 0    │   5EH     │
│   E   │    0  1 1 1 1 0 0 1    │   79H     │
│   F   │    0  1 1 1 0 0 0 1    │   71H     │
└───────┴─────────────────────────┴───────────┘

DISPLAY DIGIT PROGRAM:
────────────────────

    ORG 0000H
    
    MOV DPTR, #SEG_TABLE
    
DISPLAY_LOOP:
    MOV R0, #0          ; Start from 0
    
NEXT_DIGIT:
    MOV A, R0
    MOVC A, @A+DPTR     ; Get segment code
    MOV P1, A           ; Display
    CALL DELAY_1S
    INC R0
    CJNE R0, #10, NEXT_DIGIT
    SJMP DISPLAY_LOOP

SEG_TABLE:
    DB 3FH, 06H, 5BH, 4FH, 66H
    DB 6DH, 7DH, 07H, 7FH, 6FH
```

### 3.2 Multiplexed Display (4 Digits)

```
MULTIPLEXED 4-DIGIT DISPLAY
━━━━━━━━━━━━━━━━━━━━━━━━━━━

    8051           4-Digit 7-Segment (Common Cathode)
    ┌────┐         ┌─────────────────────────────┐
    │    │         │   ┌───┐ ┌───┐ ┌───┐ ┌───┐  │
    │P1.x├─────────┤   │ 1 │ │ 2 │ │ 3 │ │ 4 │  │ ◄─ Segments
    │    │         │   └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘  │    (P1)
    │    │         │     │     │     │     │    │
    │P2.0├─────────┤─────┼─────┼─────┼─────┘    │ ◄─ Digit 4
    │P2.1├─────────┤─────┼─────┼─────┘          │ ◄─ Digit 3
    │P2.2├─────────┤─────┼─────┘                │ ◄─ Digit 2
    │P2.3├─────────┤─────┘                      │ ◄─ Digit 1
    └────┘         └────────────────────────────┘

MULTIPLEXING CONCEPT:
───────────────────
- Only one digit ON at a time
- Switch between digits rapidly (>60 Hz)
- Persistence of vision creates solid display

DISPLAY BUFFER:
    30H = Digit 1 value
    31H = Digit 2 value
    32H = Digit 3 value
    33H = Digit 4 value

MULTIPLEXED DISPLAY PROGRAM:
─────────────────────────

    ORG 0000H
    LJMP MAIN

    ORG 000BH           ; Timer 0 ISR
    LJMP TIMER0_ISR

    ORG 0030H
MAIN:
    ; Initialize display values
    MOV 30H, #1         ; Digit 1
    MOV 31H, #2         ; Digit 2
    MOV 32H, #3         ; Digit 3
    MOV 33H, #4         ; Digit 4
    MOV R5, #0          ; Current digit
    
    ; Setup Timer 0 for 5ms refresh
    MOV TMOD, #01H
    MOV TH0, #0ECH
    MOV TL0, #78H
    SETB ET0
    SETB EA
    SETB TR0
    
    SJMP $

TIMER0_ISR:
    CLR TR0
    PUSH ACC
    PUSH PSW
    
    ; Turn off all digits first (prevent ghosting)
    MOV P2, #0FFH
    
    ; Get current digit value
    MOV A, R5
    ADD A, #30H         ; Point to buffer
    MOV R0, A
    MOV A, @R0          ; Get digit value
    
    ; Convert to segment pattern
    MOV DPTR, #SEG_TABLE
    MOVC A, @A+DPTR
    MOV P1, A           ; Output segments
    
    ; Enable current digit
    MOV A, R5
    MOV DPTR, #DIGIT_SELECT
    MOVC A, @A+DPTR
    MOV P2, A
    
    ; Next digit
    INC R5
    CJNE R5, #4, RELOAD_TIMER
    MOV R5, #0
    
RELOAD_TIMER:
    MOV TH0, #0ECH
    MOV TL0, #78H
    
    POP PSW
    POP ACC
    SETB TR0
    RETI

SEG_TABLE:
    DB 3FH, 06H, 5BH, 4FH, 66H
    DB 6DH, 7DH, 07H, 7FH, 6FH

DIGIT_SELECT:           ; Active LOW
    DB 0FEH             ; Digit 1 (P2.0 = 0)
    DB 0FDH             ; Digit 2 (P2.1 = 0)
    DB 0FBH             ; Digit 3 (P2.2 = 0)
    DB 0F7H             ; Digit 4 (P2.3 = 0)
```

---

## 4. LCD Interfacing

### 4.1 16×2 Character LCD (HD44780)

```
16×2 LCD INTERFACING
━━━━━━━━━━━━━━━━━━━━

LCD PIN CONNECTIONS:
───────────────────

    LCD Pin     Function         8051 Connection
    ─────────────────────────────────────────────
      1         GND              GND
      2         VCC              +5V
      3         VEE (Contrast)   Potentiometer → GND
      4         RS               P3.5
      5         R/W              P3.6 (or GND for write-only)
      6         E                P3.7
      7-14      D0-D7            P1.0-P1.7
      15        LED+             +5V (through 100Ω)
      16        LED-             GND

4-BIT MODE (Save pins):
    D4-D7 ─── P1.4-P1.7
    D0-D3 ─── Not connected

LCD COMMANDS:
────────────

┌────────┬───────────────────────────────────────┐
│ Command│ Description                           │
├────────┼───────────────────────────────────────┤
│  01H   │ Clear display                         │
│  02H   │ Return home                           │
│  04H   │ Decrement cursor                      │
│  06H   │ Increment cursor                      │
│  0CH   │ Display ON, cursor OFF                │
│  0EH   │ Display ON, cursor ON                 │
│  0FH   │ Display ON, cursor blink              │
│  10H   │ Shift cursor left                     │
│  14H   │ Shift cursor right                    │
│  18H   │ Shift display left                    │
│  1CH   │ Shift display right                   │
│  38H   │ 8-bit, 2-line, 5×7 font              │
│  80H   │ Cursor to beginning of line 1        │
│  C0H   │ Cursor to beginning of line 2        │
└────────┴───────────────────────────────────────┘

LCD TIMING:
──────────

           ┌────────────────┐
    RS ────┘                └─────────
           
           ┌────────────────┐
    E  ────┘                └─────────
           
           ├──►│      Data valid      │◄──│
           
    Minimum E pulse width: 450 ns
    Data setup time: 195 ns
```

### 4.2 8-Bit Mode LCD Program

```
8-BIT MODE LCD PROGRAMMING
━━━━━━━━━━━━━━━━━━━━━━━━━━

; Pin definitions
RS EQU P3.5
RW EQU P3.6
E  EQU P3.7
LCD_DATA EQU P1

    ORG 0000H
    
MAIN:
    CALL LCD_INIT
    
    ; Display message on line 1
    MOV A, #80H         ; Line 1, position 0
    CALL LCD_CMD
    MOV DPTR, #MSG1
    CALL LCD_STRING
    
    ; Display on line 2
    MOV A, #0C0H        ; Line 2, position 0
    CALL LCD_CMD
    MOV DPTR, #MSG2
    CALL LCD_STRING
    
    SJMP $

; Initialize LCD
LCD_INIT:
    CALL DELAY_20MS     ; Wait for LCD power-up
    
    MOV A, #38H         ; 8-bit, 2-line, 5×7
    CALL LCD_CMD
    
    MOV A, #0CH         ; Display ON, cursor OFF
    CALL LCD_CMD
    
    MOV A, #06H         ; Increment cursor
    CALL LCD_CMD
    
    MOV A, #01H         ; Clear display
    CALL LCD_CMD
    CALL DELAY_2MS      ; Clear needs 2ms
    RET

; Send command to LCD
LCD_CMD:
    MOV LCD_DATA, A
    CLR RS              ; RS = 0 for command
    CLR RW              ; RW = 0 for write
    SETB E              ; Enable high
    CALL DELAY_1MS
    CLR E               ; Enable low
    CALL DELAY_1MS
    RET

; Send data to LCD
LCD_DATA_WRITE:
    MOV LCD_DATA, A
    SETB RS             ; RS = 1 for data
    CLR RW              ; RW = 0 for write
    SETB E
    CALL DELAY_1MS
    CLR E
    CALL DELAY_1MS
    RET

; Display null-terminated string
LCD_STRING:
    CLR A
    MOVC A, @A+DPTR
    JZ STRING_DONE
    CALL LCD_DATA_WRITE
    INC DPTR
    SJMP LCD_STRING
STRING_DONE:
    RET

; Delay routines
DELAY_1MS:
    MOV R7, #250
    DJNZ R7, $
    RET

DELAY_2MS:
    CALL DELAY_1MS
    CALL DELAY_1MS
    RET

DELAY_20MS:
    MOV R6, #20
DLY20:
    CALL DELAY_1MS
    DJNZ R6, DLY20
    RET

; Messages
MSG1:
    DB 'Hello, World!', 0
MSG2:
    DB '8051 LCD Demo', 0

    END
```

### 4.3 4-Bit Mode LCD Program

```
4-BIT MODE LCD PROGRAMMING
━━━━━━━━━━━━━━━━━━━━━━━━━━

; Uses only P1.4-P1.7 for data (saves 4 pins)

; Pin definitions
RS EQU P3.5
RW EQU P3.6
E  EQU P3.7

    ORG 0000H
    
MAIN:
    CALL LCD_INIT_4BIT
    MOV A, #80H
    CALL LCD_CMD_4BIT
    MOV DPTR, #MSG
    CALL LCD_STRING_4BIT
    SJMP $

LCD_INIT_4BIT:
    CALL DELAY_20MS
    
    ; Special initialization sequence for 4-bit mode
    MOV A, #30H         ; Function set (8-bit)
    CALL SEND_NIBBLE
    CALL DELAY_5MS
    
    MOV A, #30H
    CALL SEND_NIBBLE
    CALL DELAY_1MS
    
    MOV A, #30H
    CALL SEND_NIBBLE
    CALL DELAY_1MS
    
    MOV A, #20H         ; Switch to 4-bit mode
    CALL SEND_NIBBLE
    CALL DELAY_1MS
    
    ; Now in 4-bit mode
    MOV A, #28H         ; 4-bit, 2-line, 5×7
    CALL LCD_CMD_4BIT
    
    MOV A, #0CH         ; Display ON
    CALL LCD_CMD_4BIT
    
    MOV A, #06H         ; Increment cursor
    CALL LCD_CMD_4BIT
    
    MOV A, #01H         ; Clear
    CALL LCD_CMD_4BIT
    CALL DELAY_2MS
    RET

; Send command in 4-bit mode
LCD_CMD_4BIT:
    PUSH ACC
    CLR RS
    CALL SEND_NIBBLE    ; High nibble first
    POP ACC
    SWAP A
    CLR RS
    CALL SEND_NIBBLE    ; Low nibble
    RET

; Send data in 4-bit mode
LCD_DATA_4BIT:
    PUSH ACC
    SETB RS
    CALL SEND_NIBBLE
    POP ACC
    SWAP A
    SETB RS
    CALL SEND_NIBBLE
    RET

; Send high nibble of A to LCD (P1.4-P1.7)
SEND_NIBBLE:
    ANL A, #0F0H        ; Keep high nibble
    ANL P1, #0FH        ; Clear high nibble of P1
    ORL P1, A           ; Set data bits
    CLR RW
    SETB E
    NOP
    NOP
    CLR E
    CALL DELAY_1MS
    RET

LCD_STRING_4BIT:
    CLR A
    MOVC A, @A+DPTR
    JZ DONE
    CALL LCD_DATA_4BIT
    INC DPTR
    SJMP LCD_STRING_4BIT
DONE:
    RET

MSG:
    DB '4-Bit LCD Mode', 0
```

---

## 5. Keypad Interfacing

### 5.1 4×4 Matrix Keypad

```
4×4 MATRIX KEYPAD INTERFACING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

KEYPAD LAYOUT:
─────────────

    ┌─────┬─────┬─────┬─────┐
    │  1  │  2  │  3  │  A  │ R0
    ├─────┼─────┼─────┼─────┤
    │  4  │  5  │  6  │  B  │ R1
    ├─────┼─────┼─────┼─────┤
    │  7  │  8  │  9  │  C  │ R2
    ├─────┼─────┼─────┼─────┤
    │  *  │  0  │  #  │  D  │ R3
    └─────┴─────┴─────┴─────┘
      C0    C1    C2    C3

CONNECTION:
──────────

    8051          Keypad
    P1.0 ───────► R0 (Row 0)
    P1.1 ───────► R1 (Row 1)
    P1.2 ───────► R2 (Row 2)
    P1.3 ───────► R3 (Row 3)
    P1.4 ◄─────── C0 (Column 0) ──┬── 10K ── +5V
    P1.5 ◄─────── C1 (Column 1) ──┤
    P1.6 ◄─────── C2 (Column 2) ──┤
    P1.7 ◄─────── C3 (Column 3) ──┘

    Rows: Output (drive LOW one at a time)
    Columns: Input (read for key press)

SCANNING ALGORITHM:
─────────────────

1. Ground one row at a time
2. Read columns
3. If any column LOW, key pressed at intersection
4. Move to next row
5. Repeat

KEYPAD SCAN PROGRAM:
───────────────────

    ORG 0000H

MAIN:
    MOV P2, #0          ; Clear display
SCAN_AGAIN:
    CALL KEY_SCAN       ; Get key
    JZ SCAN_AGAIN       ; No key pressed
    MOV P2, A           ; Display key code
    CALL DELAY_200MS    ; Debounce
    SJMP SCAN_AGAIN

; Returns key code in A, 0 if no key
KEY_SCAN:
    MOV R0, #0          ; Row counter
    MOV R1, #0FEH       ; Row select pattern
    
SCAN_ROW:
    MOV P1, R1          ; Ground current row
    NOP
    NOP
    MOV A, P1           ; Read columns
    ANL A, #0F0H        ; Mask rows
    CJNE A, #0F0H, KEY_FOUND
    
    ; No key in this row
    INC R0
    MOV A, R1
    RL A                ; Next row
    MOV R1, A
    CJNE R0, #4, SCAN_ROW
    
    ; No key pressed
    MOV A, #0
    RET

KEY_FOUND:
    ; Debounce
    CALL DELAY_20MS
    MOV A, P1
    ANL A, #0F0H
    CJNE A, #0F0H, VALID_KEY
    MOV A, #0           ; Bounce
    RET
    
VALID_KEY:
    ; Calculate key code
    ; Row × 4 + Column
    MOV A, R0
    MOV B, #4
    MUL AB              ; Row × 4
    
    ; Determine column
    MOV R2, A           ; Save row×4
    MOV A, P1
    ANL A, #0F0H
    
    CJNE A, #0E0H, NOT_C0
    MOV A, #0           ; Column 0
    SJMP GOT_COL
NOT_C0:
    CJNE A, #0D0H, NOT_C1
    MOV A, #1           ; Column 1
    SJMP GOT_COL
NOT_C1:
    CJNE A, #0B0H, NOT_C2
    MOV A, #2           ; Column 2
    SJMP GOT_COL
NOT_C2:
    MOV A, #3           ; Column 3
    
GOT_COL:
    ADD A, R2           ; Row×4 + Col
    
    ; Convert to ASCII using table
    MOV DPTR, #KEY_TABLE
    MOVC A, @A+DPTR
    
    ; Wait for key release
WAIT_REL:
    MOV P1, #0F0H       ; All rows LOW
    MOV B, P1
    ANL B, #0F0H
    CJNE B, #0F0H, WAIT_REL
    CALL DELAY_20MS
    
    RET

KEY_TABLE:
    DB '1', '2', '3', 'A'   ; Row 0
    DB '4', '5', '6', 'B'   ; Row 1
    DB '7', '8', '9', 'C'   ; Row 2
    DB '*', '0', '#', 'D'   ; Row 3
```

---

## 6. ADC Interfacing

### 6.1 ADC0804 (8-bit ADC)

```
ADC0804 INTERFACING
━━━━━━━━━━━━━━━━━━━

ADC0804 PINS:
────────────

    Pin  Name    Function
    ───────────────────────────
    1    CS      Chip Select (active LOW)
    2    RD      Read (active LOW)
    3    WR      Write/Start Conversion (active LOW)
    4    CLK IN  Clock Input
    5    INTR    Interrupt (active LOW, conversion done)
    6    Vin+    Analog Input (+)
    7    Vin-    Analog Input (-) usually GND
    8    AGND    Analog Ground
    9    Vref/2  Reference Voltage (optional)
    10   GND     Digital Ground
    11-18 D0-D7  Data Output
    19   CLK R   Clock Generator Resistor
    20   VCC     +5V

CONNECTION:
──────────

    8051          ADC0804          Analog
    ┌────┐        ┌──────┐         Source
    │P1  │◄───────│D0-D7 │         │
    │    │        │      │         │
    │P3.0├───────►│CS    │    Vin ─┤
    │P3.1├───────►│RD    │         │
    │P3.2├───────►│WR    │    GND ─┘
    │P3.3│◄───────│INTR  │
    └────┘        └──────┘

ADC READ PROGRAM:
────────────────

; Pin definitions
CS   EQU P3.0
RD   EQU P3.1
WR   EQU P3.2
INTR EQU P3.3

    ORG 0000H

MAIN:
    SETB INTR           ; Configure as input
    
READ_ADC:
    CLR CS              ; Select ADC
    
    ; Start conversion
    CLR WR
    NOP
    SETB WR             ; Rising edge starts conversion
    
    ; Wait for conversion complete
    JB INTR, $          ; Wait for INTR LOW
    
    ; Read data
    CLR RD              ; Enable output
    NOP
    MOV A, P1           ; Read ADC value
    SETB RD
    SETB CS             ; Deselect ADC
    
    ; Display result
    MOV P2, A           ; Show on Port 2
    
    CALL DELAY_100MS
    SJMP READ_ADC

; Alternative: Continuous conversion mode
CONTINUOUS_ADC:
    CLR CS
    CLR WR              ; Start conversion
    
WAIT_CONV:
    JB INTR, WAIT_CONV
    CLR RD
    MOV A, P1
    SETB RD
    
    MOV P2, A           ; Display
    
    ; Start next conversion immediately
    CLR WR
    NOP
    SETB WR
    
    SJMP WAIT_CONV
```

### 6.2 Temperature Sensor (LM35 + ADC)

```
LM35 TEMPERATURE SENSOR + ADC
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

LM35 OUTPUT: 10mV/°C
0°C = 0V, 100°C = 1V

For 0-100°C with 5V ADC reference:
    ADC Value = (Temperature × 10mV × 255) / 5000mV
    ADC Value = Temperature × 0.51
    
    Temperature = ADC Value / 0.51
    Temperature ≈ ADC Value × 2 (approximately)

For better resolution, use Vref = 2.56V:
    Temperature = ADC Value (direct!)

TEMPERATURE DISPLAY PROGRAM:
──────────────────────────

    ORG 0000H

MAIN:
    CALL LCD_INIT
    
TEMP_LOOP:
    CALL READ_ADC       ; Get temperature
    
    ; Convert to actual temperature
    ; A × 2 for approximate value
    MOV B, #2
    MUL AB              ; Temperature in A
    
    ; Display on LCD
    PUSH ACC
    MOV A, #80H
    CALL LCD_CMD
    MOV DPTR, #TEMP_MSG
    CALL LCD_STRING
    POP ACC
    
    ; Display temperature value
    CALL DISPLAY_DECIMAL
    
    ; Add °C symbol
    MOV A, #0DFH        ; Degree symbol
    CALL LCD_DATA_WRITE
    MOV A, #'C'
    CALL LCD_DATA_WRITE
    
    CALL DELAY_1S
    SJMP TEMP_LOOP

TEMP_MSG:
    DB 'Temp: ', 0

; Display A as 3-digit decimal
DISPLAY_DECIMAL:
    MOV B, #100
    DIV AB
    ADD A, #'0'
    CALL LCD_DATA_WRITE
    
    MOV A, B
    MOV B, #10
    DIV AB
    ADD A, #'0'
    CALL LCD_DATA_WRITE
    
    MOV A, B
    ADD A, #'0'
    CALL LCD_DATA_WRITE
    RET
```

---

## 7. DAC Interfacing

### 7.1 DAC0808 (8-bit DAC)

```
DAC0808 INTERFACING
━━━━━━━━━━━━━━━━━━━

OUTPUT EQUATION:
    Vout = Vref × (Digital Input) / 256
    
    For Vref = 5V:
    Vout = 5 × D / 256
    Vout = D × 19.53 mV (per step)

CONNECTION:
──────────

    8051          DAC0808          Op-Amp
    ┌────┐        ┌──────┐        ┌──────┐
    │P1.0├───────►│D0    │        │      │
    │P1.1├───────►│D1    │───Iout►│ 741  ├──► Vout
    │P1.2├───────►│D2    │        │      │
    │... │        │...   │        └──────┘
    │P1.7├───────►│D7    │
    └────┘        └──────┘

WAVEFORM GENERATION:
──────────────────

; Sawtooth wave
SAWTOOTH:
    MOV A, #0
SAW_LOOP:
    MOV P1, A
    INC A
    SJMP SAW_LOOP       ; 0→255→0...

; Triangle wave
TRIANGLE:
    MOV A, #0
TRI_UP:
    MOV P1, A
    INC A
    JNZ TRI_UP
    
TRI_DOWN:
    DEC A
    MOV P1, A
    JNZ TRI_DOWN
    SJMP TRI_UP

; Square wave
SQUARE:
    MOV P1, #0FFH       ; High
    CALL DELAY
    MOV P1, #00H        ; Low
    CALL DELAY
    SJMP SQUARE

; Sine wave (using lookup table)
SINE_WAVE:
    MOV DPTR, #SINE_TABLE
    MOV R0, #0
    
SINE_LOOP:
    MOV A, R0
    MOVC A, @A+DPTR
    MOV P1, A
    INC R0
    CJNE R0, #32, SINE_LOOP
    MOV R0, #0
    SJMP SINE_LOOP

; 32-point sine lookup (0-255 range)
SINE_TABLE:
    DB 128, 152, 176, 198, 217, 233, 245, 252
    DB 255, 252, 245, 233, 217, 198, 176, 152
    DB 128, 103, 79, 57, 38, 22, 10, 3
    DB 0, 3, 10, 22, 38, 57, 79, 103
```

---

## 📋 Summary Table

| Interface | Port Used | Key Points |
|-----------|-----------|------------|
| LED | P1 | 330Ω resistor, active LOW common |
| Switch | P1 (input) | Pull-up resistor, debounce 20ms |
| 7-Segment | P1 (segments), P2 (digits) | Multiplex at >60Hz |
| LCD 16×2 | P1 (data), P3 (control) | Initialize with 38H, 0CH, 06H, 01H |
| 4×4 Keypad | P1 (rows+cols) | Scan rows, read columns |
| ADC0804 | P1 (data), P3 (control) | Start WR pulse, wait INTR |
| DAC0808 | P1 (data) | Direct write, op-amp output |

---

## ❓ Quick Revision Questions

1. **Why use active-LOW for LED connection?**
   <details>
   <summary>Show Answer</summary>
   8051 ports can sink more current (up to 10mA) than source (limited). Active-LOW with pull-up to VCC through resistor provides reliable LED current.
   </details>

2. **What is the purpose of debouncing in switch interfacing?**
   <details>
   <summary>Show Answer</summary>
   Mechanical switches bounce (oscillate) for 10-50ms when pressed/released. Debouncing (20ms delay and re-check) ensures only one valid press is detected per actuation.
   </details>

3. **How does 7-segment multiplexing work?**
   <details>
   <summary>Show Answer</summary>
   Only one digit is ON at any time. Display switches between digits rapidly (>60Hz). Persistence of vision makes all digits appear lit simultaneously. Uses fewer I/O pins.
   </details>

4. **What is the initialization sequence for HD44780 LCD?**
   <details>
   <summary>Show Answer</summary>
   Wait 20ms, send 38H (8-bit, 2-line), 0CH (display ON, cursor OFF), 06H (increment cursor), 01H (clear display), wait 2ms. For 4-bit: send 30H three times, then 20H.
   </details>

5. **How to calculate ADC output from analog voltage?**
   <details>
   <summary>Show Answer</summary>
   ADC Value = (Vin / Vref) × 255. For 5V reference: ADC = Vin × 51. For example, 2.5V input gives ADC = 127 (half scale).
   </details>

6. **How to generate a 1 kHz sine wave using DAC?**
   <details>
   <summary>Show Answer</summary>
   Use 32-point sine lookup table. Output each point every 31.25µs (1ms/32 points). Period = 1ms = 1kHz. Adjust delay for different frequencies.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [8.4 Serial Communication](04-serial-communication.md) | [Unit 8 Index](README.md) | [8.6 Advanced Programs](06-advanced-programs.md) |

---

*[← Previous: Serial Communication](04-serial-communication.md) | [Next: Advanced Programs →](06-advanced-programs.md)*
