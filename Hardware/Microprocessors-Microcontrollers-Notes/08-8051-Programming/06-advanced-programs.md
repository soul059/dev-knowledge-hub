# Chapter 8.6: 8051 Advanced Programs

## 📚 Chapter Overview

This chapter presents complete advanced application programs for the 8051 microcontroller, including motor control, PWM generation, real-time systems, and integrated projects.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Implement PWM for speed control
- Interface and control DC and stepper motors
- Build complete integrated projects
- Develop efficient assembly coding techniques
- Apply 8051 to practical embedded applications

---

## 1. PWM Generation

### 1.1 Software PWM Basics

```
PULSE WIDTH MODULATION (PWM)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PWM CONCEPT:
───────────

Duty Cycle = TON / (TON + TOFF) × 100%

    ┌────┐    ┌────┐    ┌────┐    ┌────┐
    │    │    │    │    │    │    │    │
────┘    └────┘    └────┘    └────┘    └────
    │◄──►│     
      TON  TOFF
    │◄───────►│
       Period

    25% Duty:   ┌─┐     ┌─┐     ┌─┐     ┌─┐
              ──┘ └─────┘ └─────┘ └─────┘ └───

    50% Duty:   ┌───┐   ┌───┐   ┌───┐   ┌───┐
              ──┘   └───┘   └───┘   └───┘   └─

    75% Duty:   ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐
              ──┘     └─┘     └─┘     └─┘     └

APPLICATIONS:
• LED brightness control
• Motor speed control
• Audio generation
• Power control

BASIC SOFTWARE PWM:
─────────────────

; Generate PWM on P1.0
; Duty cycle in R7 (0-100)

    ORG 0000H

MAIN:
    MOV R7, #50         ; 50% duty cycle
    
PWM_LOOP:
    SETB P1.0           ; HIGH
    MOV R6, R7          ; ON time
    CALL DELAY_10US
    
    CLR P1.0            ; LOW
    MOV A, #100
    CLR C
    SUBB A, R7          ; OFF time = 100 - duty
    MOV R6, A
    CALL DELAY_10US
    
    SJMP PWM_LOOP

DELAY_10US:
    ; R6 × 10µs delay
    MOV A, R6
    JZ DELAY_DONE
DLY_LOOP:
    NOP
    NOP
    NOP
    NOP
    NOP
    NOP
    NOP
    DJNZ R6, DLY_LOOP
DELAY_DONE:
    RET
```

### 1.2 Timer-Based PWM

```
TIMER-BASED PWM (More Precise)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Using Timer 0 interrupt for precise PWM timing.

; 1 kHz PWM with variable duty cycle
; Duty cycle controlled by value in DUTY (0-100)

DUTY EQU 30H            ; Duty cycle storage

    ORG 0000H
    LJMP MAIN

    ORG 000BH           ; Timer 0 vector
    LJMP PWM_ISR

    ORG 0030H
MAIN:
    MOV DUTY, #75       ; 75% duty cycle
    MOV R7, #0          ; PWM counter (0-99)
    
    ; Timer 0: 10µs interrupt (100 per ms = 1kHz PWM)
    MOV TMOD, #02H      ; Mode 2 auto-reload
    MOV TH0, #246       ; 256-10 = 246 (10µs @ 12MHz)
    MOV TL0, #246
    
    SETB ET0
    SETB EA
    SETB TR0
    
    ; Main loop - adjust duty cycle as needed
    SJMP $

PWM_ISR:
    INC R7
    CJNE R7, #100, CHECK_DUTY
    MOV R7, #0          ; Reset counter
    
CHECK_DUTY:
    MOV A, R7
    CJNE A, DUTY, NOT_MATCH
    CLR P1.0            ; Turn OFF at duty point
    RETI
    
NOT_MATCH:
    JNC PWM_OFF
    SETB P1.0           ; Turn ON when counter < duty
    RETI
    
PWM_OFF:
    CLR P1.0            ; Keep OFF when counter > duty
    RETI

; Actually simpler version:
PWM_ISR_SIMPLE:
    INC R7
    CJNE R7, #100, COMPARE
    MOV R7, #0
    SETB P1.0           ; Start of cycle = ON
    RETI
    
COMPARE:
    MOV A, DUTY
    CJNE A, R7, ISR_EXIT
    CLR P1.0            ; At duty point = OFF
ISR_EXIT:
    RETI
```

### 1.3 Multi-Channel PWM

```
MULTI-CHANNEL PWM (LED RGB Control)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

; 3-channel PWM for RGB LED
; P1.0 = Red, P1.1 = Green, P1.2 = Blue

RED_DUTY   EQU 30H
GREEN_DUTY EQU 31H
BLUE_DUTY  EQU 32H
PWM_COUNT  EQU 33H

    ORG 0000H
    LJMP MAIN

    ORG 000BH
    LJMP PWM_ISR

    ORG 0030H
MAIN:
    ; Set RGB values (0-100 each)
    MOV RED_DUTY, #100      ; Red 100%
    MOV GREEN_DUTY, #50     ; Green 50%
    MOV BLUE_DUTY, #25      ; Blue 25%
    MOV PWM_COUNT, #0
    
    MOV TMOD, #02H
    MOV TH0, #246
    MOV TL0, #246
    SETB ET0
    SETB EA
    SETB TR0
    
    SJMP $

PWM_ISR:
    PUSH ACC
    PUSH PSW
    
    INC PWM_COUNT
    MOV A, PWM_COUNT
    CJNE A, #100, UPDATE_PWM
    MOV PWM_COUNT, #0
    MOV P1, #07H            ; All channels ON at start
    SJMP ISR_EXIT
    
UPDATE_PWM:
    ; Check Red
    MOV A, RED_DUTY
    CJNE A, PWM_COUNT, CHECK_GREEN
    CLR P1.0                ; Red OFF
    
CHECK_GREEN:
    MOV A, GREEN_DUTY
    CJNE A, PWM_COUNT, CHECK_BLUE
    CLR P1.1                ; Green OFF
    
CHECK_BLUE:
    MOV A, BLUE_DUTY
    CJNE A, PWM_COUNT, ISR_EXIT
    CLR P1.2                ; Blue OFF
    
ISR_EXIT:
    POP PSW
    POP ACC
    RETI
```

---

## 2. Motor Control

### 2.1 DC Motor with L293D

```
DC MOTOR CONTROL USING L293D
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

L293D H-BRIDGE CONNECTION:
─────────────────────────

    8051          L293D            DC Motor
    ┌────┐        ┌──────┐         ┌──────┐
    │P1.0├───────►│ IN1  │  OUT1 ──┤ M    │
    │P1.1├───────►│ IN2  │  OUT2 ──┤      │
    │    │        │      │         └──────┘
    │P1.2├───────►│ EN1  │ (Enable/PWM)
    └────┘        └──────┘

    IN1  IN2  EN   Motor Action
    ───────────────────────────
     0    0   1    Stop (brake)
     0    1   1    Reverse
     1    0   1    Forward
     1    1   1    Stop (brake)
     X    X   0    Coast (free run)

DC MOTOR CONTROL PROGRAM:
────────────────────────

; Pin definitions
IN1  EQU P1.0
IN2  EQU P1.1
EN   EQU P1.2

    ORG 0000H

MAIN:
    SETB EN             ; Enable motor driver
    
    ; Run forward for 3 seconds
    CALL MOTOR_FORWARD
    CALL DELAY_3S
    
    ; Stop for 1 second
    CALL MOTOR_STOP
    CALL DELAY_1S
    
    ; Run reverse for 3 seconds
    CALL MOTOR_REVERSE
    CALL DELAY_3S
    
    ; Stop
    CALL MOTOR_STOP
    
    SJMP $

MOTOR_FORWARD:
    SETB IN1
    CLR IN2
    RET

MOTOR_REVERSE:
    CLR IN1
    SETB IN2
    RET

MOTOR_STOP:
    CLR IN1
    CLR IN2
    RET

MOTOR_BRAKE:
    SETB IN1
    SETB IN2
    RET

; Speed control using PWM on EN pin
MOTOR_SPEED:
    ; R7 = speed (0-100)
    ; Use PWM routine on EN pin
    MOV DUTY, R7
    RET
```

### 2.2 Stepper Motor Control

```
STEPPER MOTOR CONTROL
━━━━━━━━━━━━━━━━━━━━━

STEPPER MOTOR TYPES:
──────────────────

Unipolar (5/6 wire):  Uses center-tapped coils
Bipolar (4 wire):     Requires H-bridge

STEP SEQUENCES:
──────────────

Full Step (Wave Drive - 1 coil at a time):
    Step  A   B   C   D
    1     1   0   0   0
    2     0   1   0   0
    3     0   0   1   0
    4     0   0   0   1
    
Full Step (2-Phase - 2 coils at a time, more torque):
    Step  A   B   C   D
    1     1   1   0   0
    2     0   1   1   0
    3     0   0   1   1
    4     1   0   0   1

Half Step (8 steps, smoother):
    Step  A   B   C   D
    1     1   0   0   0
    2     1   1   0   0
    3     0   1   0   0
    4     0   1   1   0
    5     0   0   1   0
    6     0   0   1   1
    7     0   0   0   1
    8     1   0   0   1

CONNECTION WITH ULN2003:
──────────────────────

    8051          ULN2003        Stepper
    ┌────┐        ┌──────┐       ┌──────┐
    │P1.0├───────►│ IN1  │ OUT1 ─┤ A    │
    │P1.1├───────►│ IN2  │ OUT2 ─┤ B    │
    │P1.2├───────►│ IN3  │ OUT3 ─┤ C    │
    │P1.3├───────►│ IN4  │ OUT4 ─┤ D    │
    └────┘        └──────┘       │      │
                    COM ──────────┤ +12V │
                                 └──────┘

STEPPER MOTOR PROGRAM:
────────────────────

; Pin definitions - P1.0-P1.3 for stepper

STEP_DELAY EQU 35H      ; Delay between steps (speed)

    ORG 0000H

MAIN:
    MOV STEP_DELAY, #10  ; Speed setting
    
    ; Rotate clockwise 200 steps (1 revolution for 200-step motor)
    MOV R5, #200
    CALL ROTATE_CW
    
    CALL DELAY_1S
    
    ; Rotate counter-clockwise 100 steps
    MOV R5, #100
    CALL ROTATE_CCW
    
    SJMP $

; Full step - wave drive clockwise
ROTATE_CW:
    MOV DPTR, #STEP_TABLE
CW_LOOP:
    MOV R0, #0          ; Step index
CW_STEP:
    MOV A, R0
    MOVC A, @A+DPTR
    MOV P1, A           ; Output step pattern
    CALL STEP_WAIT
    
    INC R0
    CJNE R0, #4, CW_STEP
    
    DJNZ R5, CW_LOOP
    RET

; Full step - wave drive counter-clockwise
ROTATE_CCW:
    MOV DPTR, #STEP_TABLE
CCW_LOOP:
    MOV R0, #3          ; Start from end
CCW_STEP:
    MOV A, R0
    MOVC A, @A+DPTR
    MOV P1, A
    CALL STEP_WAIT
    
    DEC R0
    CJNE R0, #0FFH, CCW_STEP
    
    DJNZ R5, CCW_LOOP
    RET

STEP_WAIT:
    MOV R7, STEP_DELAY
SW_LOOP:
    MOV R6, #250
    DJNZ R6, $
    DJNZ R7, SW_LOOP
    RET

; Wave drive step sequence
STEP_TABLE:
    DB 01H              ; 0001 - A
    DB 02H              ; 0010 - B
    DB 04H              ; 0100 - C
    DB 08H              ; 1000 - D

; Full step (2-phase) sequence
STEP_TABLE_2PHASE:
    DB 03H              ; 0011 - AB
    DB 06H              ; 0110 - BC
    DB 0CH              ; 1100 - CD
    DB 09H              ; 1001 - DA

; Half step sequence
HALF_STEP_TABLE:
    DB 01H, 03H, 02H, 06H, 04H, 0CH, 08H, 09H
```

### 2.3 Servo Motor Control

```
SERVO MOTOR CONTROL (Using PWM)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SERVO PWM SPECIFICATION:
──────────────────────

Period: 20ms (50 Hz)
Pulse Width: 1ms (0°) to 2ms (180°)

    Position    Pulse Width
    ─────────────────────────
    0°          1.0 ms
    45°         1.25 ms
    90°         1.5 ms
    135°        1.75 ms
    180°        2.0 ms

    ┌───┐                                          ┌───┐
    │   │                                          │   │
────┘   └──────────────────────────────────────────┘   └───
    │◄─►│
    1-2ms
    │◄────────────── 20ms ────────────────────────►│

SERVO CONTROL PROGRAM:
────────────────────

SERVO_PIN EQU P1.0
SERVO_POS EQU 30H       ; Position value (10-20 for 1-2ms)

    ORG 0000H
    LJMP MAIN

    ORG 000BH           ; Timer 0 for 100µs intervals
    LJMP TIMER0_ISR

    ORG 0030H
MAIN:
    MOV R5, #0          ; PWM counter (0-199 for 20ms)
    MOV SERVO_POS, #15  ; 1.5ms = 90°
    
    MOV TMOD, #02H
    MOV TH0, #156       ; 100µs @ 12MHz (256-100)
    MOV TL0, #156
    
    SETB ET0
    SETB EA
    SETB TR0
    
MAIN_LOOP:
    ; Sweep from 0° to 180° and back
    
    ; Move to 0°
    MOV SERVO_POS, #10
    CALL DELAY_1S
    
    ; Move to 90°
    MOV SERVO_POS, #15
    CALL DELAY_1S
    
    ; Move to 180°
    MOV SERVO_POS, #20
    CALL DELAY_1S
    
    SJMP MAIN_LOOP

TIMER0_ISR:
    ; 100µs interrupt - 200 counts = 20ms
    INC R5
    CJNE R5, #200, CHECK_PULSE
    MOV R5, #0
    SETB SERVO_PIN      ; Start pulse
    RETI
    
CHECK_PULSE:
    MOV A, R5
    CJNE A, SERVO_POS, ISR_EXIT
    CLR SERVO_PIN       ; End pulse at position
    
ISR_EXIT:
    RETI

; Convert angle (0-180) to pulse value (10-20)
ANGLE_TO_PULSE:
    ; Input: A = angle (0-180)
    ; Output: A = pulse value (10-20)
    MOV B, #18          ; 180/10 = 18
    DIV AB              ; A = angle/18
    ADD A, #10          ; Add base (1ms = 10 × 100µs)
    RET
```

---

## 3. Complete Projects

### 3.1 Digital Voltmeter

```
DIGITAL VOLTMETER PROJECT
━━━━━━━━━━━━━━━━━━━━━━━━━

; Measures 0-5V and displays on LCD
; Uses ADC0804 and 16×2 LCD

; Hardware connections:
; P1 - ADC data
; P2 - LCD data
; P3.0 - ADC CS
; P3.1 - ADC RD
; P3.2 - ADC WR
; P3.3 - ADC INTR
; P3.5 - LCD RS
; P3.6 - LCD RW
; P3.7 - LCD E

    ORG 0000H
    LJMP MAIN

    ORG 0030H
MAIN:
    CALL LCD_INIT
    
    ; Display title
    MOV A, #80H
    CALL LCD_CMD
    MOV DPTR, #TITLE
    CALL LCD_STRING
    
MEASURE_LOOP:
    ; Read ADC
    CALL READ_ADC       ; Result in A
    
    ; Convert to voltage
    ; Voltage = ADC × 5000 / 255 mV
    ; Simplified: V = ADC × 19.6 mV
    
    ; Store ADC value
    MOV R0, A
    
    ; Calculate voltage
    MOV B, #196         ; × 196 (for 0.0196V per step × 10000)
    MUL AB              ; Result in B:A
    ; Now divide by 1000 for voltage in 0.01V units
    ; Simplified: divide by 10 twice
    MOV R1, B           ; High byte
    MOV R2, A           ; Low byte
    
    ; Display on LCD
    MOV A, #0C0H        ; Line 2
    CALL LCD_CMD
    
    MOV DPTR, #VOLTAGE
    CALL LCD_STRING
    
    ; Convert and display voltage
    MOV A, R0
    CALL DISPLAY_VOLTAGE
    
    MOV A, #'V'
    CALL LCD_DATA_WRITE
    
    CALL DELAY_500MS
    SJMP MEASURE_LOOP

; Display voltage as X.XX format
DISPLAY_VOLTAGE:
    ; A = ADC value (0-255)
    ; Voltage = A × 5 / 255 ≈ A × 0.0196
    
    ; Calculate: voltage × 100 = A × 500 / 255
    MOV B, #2           ; Approximate: A × 2 = A × 510/255
    MUL AB
    
    ; Ones digit (volts)
    MOV B, #100
    DIV AB
    ADD A, #'0'
    CALL LCD_DATA_WRITE
    
    MOV A, #'.'
    CALL LCD_DATA_WRITE
    
    ; Tenths
    MOV A, B
    MOV B, #10
    DIV AB
    ADD A, #'0'
    CALL LCD_DATA_WRITE
    
    ; Hundredths
    MOV A, B
    ADD A, #'0'
    CALL LCD_DATA_WRITE
    RET

TITLE:
    DB 'Digital Voltmeter', 0
    
VOLTAGE:
    DB 'Voltage: ', 0
```

### 3.2 Digital Clock

```
DIGITAL CLOCK WITH ALARM
━━━━━━━━━━━━━━━━━━━━━━━━

; 24-hour digital clock with alarm feature
; Display on 16×2 LCD
; Buttons: Set, Hour+, Min+, Alarm On/Off

; Time storage
SECONDS EQU 30H
MINUTES EQU 31H
HOURS   EQU 32H
TICKS   EQU 33H

; Alarm storage
ALARM_MIN  EQU 34H
ALARM_HOUR EQU 35H
ALARM_ON   EQU 36H

; Buttons
BTN_SET    EQU P3.2
BTN_HOUR   EQU P3.3
BTN_MIN    EQU P3.4
BTN_ALARM  EQU P3.5

; Buzzer
BUZZER     EQU P3.7

    ORG 0000H
    LJMP MAIN

    ORG 000BH
    LJMP TIMER0_ISR

    ORG 0030H
MAIN:
    ; Initialize time
    MOV SECONDS, #0
    MOV MINUTES, #30
    MOV HOURS, #12
    MOV TICKS, #0
    
    MOV ALARM_MIN, #0
    MOV ALARM_HOUR, #7
    MOV ALARM_ON, #0
    
    CALL LCD_INIT
    
    ; Timer 0: 50ms interrupt (20 per second)
    MOV TMOD, #01H
    MOV TH0, #3CH
    MOV TL0, #0B0H
    SETB ET0
    SETB EA
    SETB TR0
    
MAIN_LOOP:
    CALL DISPLAY_TIME
    CALL CHECK_BUTTONS
    CALL CHECK_ALARM
    CALL DELAY_100MS
    SJMP MAIN_LOOP

TIMER0_ISR:
    PUSH ACC
    PUSH PSW
    
    ; Reload timer
    CLR TR0
    MOV TH0, #3CH
    MOV TL0, #0B0H
    SETB TR0
    
    INC TICKS
    MOV A, TICKS
    CJNE A, #20, T0_EXIT    ; 20 × 50ms = 1 second
    
    MOV TICKS, #0
    INC SECONDS
    MOV A, SECONDS
    CJNE A, #60, T0_EXIT
    
    MOV SECONDS, #0
    INC MINUTES
    MOV A, MINUTES
    CJNE A, #60, T0_EXIT
    
    MOV MINUTES, #0
    INC HOURS
    MOV A, HOURS
    CJNE A, #24, T0_EXIT
    
    MOV HOURS, #0
    
T0_EXIT:
    POP PSW
    POP ACC
    RETI

DISPLAY_TIME:
    MOV A, #80H
    CALL LCD_CMD
    
    ; Display HH:MM:SS
    MOV A, HOURS
    CALL DISPLAY_2DIGIT
    
    MOV A, #':'
    CALL LCD_DATA_WRITE
    
    MOV A, MINUTES
    CALL DISPLAY_2DIGIT
    
    MOV A, #':'
    CALL LCD_DATA_WRITE
    
    MOV A, SECONDS
    CALL DISPLAY_2DIGIT
    
    ; Display alarm status on line 2
    MOV A, #0C0H
    CALL LCD_CMD
    
    MOV DPTR, #ALARM_MSG
    CALL LCD_STRING
    
    MOV A, ALARM_HOUR
    CALL DISPLAY_2DIGIT
    MOV A, #':'
    CALL LCD_DATA_WRITE
    MOV A, ALARM_MIN
    CALL DISPLAY_2DIGIT
    
    ; Show if alarm is ON/OFF
    MOV A, #' '
    CALL LCD_DATA_WRITE
    MOV A, ALARM_ON
    JZ SHOW_OFF
    MOV A, #'*'         ; Alarm ON indicator
    SJMP SHOW_STATUS
SHOW_OFF:
    MOV A, #' '
SHOW_STATUS:
    CALL LCD_DATA_WRITE
    RET

DISPLAY_2DIGIT:
    ; Display A as 2-digit decimal
    MOV B, #10
    DIV AB
    ADD A, #'0'
    CALL LCD_DATA_WRITE
    MOV A, B
    ADD A, #'0'
    CALL LCD_DATA_WRITE
    RET

CHECK_ALARM:
    MOV A, ALARM_ON
    JZ ALARM_DONE
    
    MOV A, SECONDS
    JNZ ALARM_DONE      ; Only check at :00
    
    MOV A, MINUTES
    CJNE A, ALARM_MIN, ALARM_DONE
    
    MOV A, HOURS
    CJNE A, ALARM_HOUR, ALARM_DONE
    
    ; Alarm triggered!
    CALL SOUND_ALARM
    
ALARM_DONE:
    RET

SOUND_ALARM:
    MOV R7, #10         ; 10 beeps
BEEP_LOOP:
    SETB BUZZER
    CALL DELAY_100MS
    CLR BUZZER
    CALL DELAY_100MS
    DJNZ R7, BEEP_LOOP
    RET

CHECK_BUTTONS:
    ; Check set button
    JB BTN_SET, CHK_HOUR
    CALL DELAY_20MS
    JB BTN_SET, CHK_HOUR
    ; Enter set mode (implementation depends on UI design)
    
CHK_HOUR:
    JB BTN_HOUR, CHK_MIN
    CALL DELAY_20MS
    JB BTN_HOUR, CHK_MIN
    INC HOURS
    MOV A, HOURS
    CJNE A, #24, WAIT_H_REL
    MOV HOURS, #0
WAIT_H_REL:
    JNB BTN_HOUR, $
    
CHK_MIN:
    JB BTN_MIN, CHK_ALM
    CALL DELAY_20MS
    JB BTN_MIN, CHK_ALM
    INC MINUTES
    MOV A, MINUTES
    CJNE A, #60, WAIT_M_REL
    MOV MINUTES, #0
WAIT_M_REL:
    JNB BTN_MIN, $
    
CHK_ALM:
    JB BTN_ALARM, BTN_DONE
    CALL DELAY_20MS
    JB BTN_ALARM, BTN_DONE
    MOV A, ALARM_ON
    CPL ACC.0
    MOV ALARM_ON, A
    JNB BTN_ALARM, $
    
BTN_DONE:
    RET

ALARM_MSG:
    DB 'Alarm:', 0
```

### 3.3 Temperature Controller

```
TEMPERATURE CONTROLLER PROJECT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

; Temperature controller with:
; - LM35 sensor + ADC0804
; - LCD display
; - Set point adjustment
; - Heating element control (relay)
; - Cooling fan control

; Hardware:
; P1 - ADC data / LCD data (multiplexed or separate)
; P2.0 - Heater relay
; P2.1 - Fan relay
; P2.2 - Up button
; P2.3 - Down button

CURRENT_TEMP EQU 30H
SET_TEMP     EQU 31H
HYSTERESIS   EQU 32H     ; Dead band (typically 2°C)

HEATER EQU P2.0
FAN    EQU P2.1
BTN_UP EQU P2.2
BTN_DN EQU P2.3

    ORG 0000H
    LJMP MAIN

    ORG 0030H
MAIN:
    MOV SET_TEMP, #25    ; Default 25°C
    MOV HYSTERESIS, #2   ; 2°C hysteresis
    
    CALL LCD_INIT
    
    ; Display title
    MOV A, #80H
    CALL LCD_CMD
    MOV DPTR, #TITLE
    CALL LCD_STRING
    
CONTROL_LOOP:
    ; Read temperature
    CALL READ_ADC
    ; Convert ADC to temperature
    ; LM35: 10mV/°C, with 5V ref: Temp = ADC × 500 / 255 / 10
    ; Simplified: Temp ≈ ADC × 2 (for room temperature range)
    MOV B, #2
    MUL AB
    MOV CURRENT_TEMP, A
    
    ; Display current temperature
    CALL DISPLAY_TEMPS
    
    ; Temperature control logic
    CALL CONTROL_LOGIC
    
    ; Check buttons for set point adjustment
    CALL CHECK_SETPOINT
    
    CALL DELAY_500MS
    SJMP CONTROL_LOOP

CONTROL_LOGIC:
    ; Hysteresis control to prevent rapid switching
    
    MOV A, CURRENT_TEMP
    MOV R0, A
    
    ; Calculate lower threshold: SET - HYSTERESIS
    MOV A, SET_TEMP
    CLR C
    SUBB A, HYSTERESIS
    MOV R1, A               ; Lower threshold
    
    ; Calculate upper threshold: SET + HYSTERESIS
    MOV A, SET_TEMP
    ADD A, HYSTERESIS
    MOV R2, A               ; Upper threshold
    
    ; If temp < lower threshold, turn ON heater
    MOV A, R0
    CLR C
    SUBB A, R1              ; Compare with lower
    JC HEAT_ON              ; If temp < lower
    
    ; If temp > upper threshold, turn ON fan
    MOV A, R0
    CLR C
    SUBB A, R2
    JNC FAN_ON              ; If temp > upper
    
    ; Within dead band - maintain current state
    RET
    
HEAT_ON:
    SETB HEATER
    CLR FAN
    RET
    
FAN_ON:
    CLR HEATER
    SETB FAN
    RET

DISPLAY_TEMPS:
    MOV A, #0C0H            ; Line 2
    CALL LCD_CMD
    
    ; Current temp
    MOV A, #'C'
    CALL LCD_DATA_WRITE
    MOV A, #':'
    CALL LCD_DATA_WRITE
    MOV A, CURRENT_TEMP
    CALL DISPLAY_TEMP_VALUE
    
    ; Set temp
    MOV A, #' '
    CALL LCD_DATA_WRITE
    MOV A, #'S'
    CALL LCD_DATA_WRITE
    MOV A, #':'
    CALL LCD_DATA_WRITE
    MOV A, SET_TEMP
    CALL DISPLAY_TEMP_VALUE
    
    ; Status indicators
    MOV A, #' '
    CALL LCD_DATA_WRITE
    
    JNB HEATER, CHK_FAN_IND
    MOV A, #'H'             ; Heating
    SJMP SHOW_IND
CHK_FAN_IND:
    JNB FAN, NO_IND
    MOV A, #'C'             ; Cooling
    SJMP SHOW_IND
NO_IND:
    MOV A, #'-'             ; Idle
SHOW_IND:
    CALL LCD_DATA_WRITE
    RET

DISPLAY_TEMP_VALUE:
    ; Display temperature with °C
    MOV B, #10
    DIV AB
    ADD A, #'0'
    CALL LCD_DATA_WRITE
    MOV A, B
    ADD A, #'0'
    CALL LCD_DATA_WRITE
    MOV A, #0DFH            ; ° symbol
    CALL LCD_DATA_WRITE
    RET

CHECK_SETPOINT:
    JB BTN_UP, CHK_DOWN
    CALL DELAY_20MS
    JB BTN_UP, CHK_DOWN
    INC SET_TEMP
    MOV A, SET_TEMP
    CJNE A, #50, WAIT_UP    ; Max 50°C
    MOV SET_TEMP, #50
WAIT_UP:
    JNB BTN_UP, $
    
CHK_DOWN:
    JB BTN_DN, SP_DONE
    CALL DELAY_20MS
    JB BTN_DN, SP_DONE
    DEC SET_TEMP
    MOV A, SET_TEMP
    CJNE A, #10, WAIT_DN    ; Min 10°C
    MOV SET_TEMP, #10
WAIT_DN:
    JNB BTN_DN, $
    
SP_DONE:
    RET

TITLE:
    DB 'Temp Controller', 0
```

---

## 4. Code Optimization Tips

```
8051 CODE OPTIMIZATION TECHNIQUES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. USE REGISTER BANK SWITCHING:
   Instead of PUSH/POP in ISRs
   
   ; Slow:
   ISR:
       PUSH ACC
       PUSH PSW
       PUSH R0
       ; ... code ...
       POP R0
       POP PSW
       POP ACC
       RETI
   
   ; Fast:
   ISR:
       PUSH PSW
       MOV PSW, #08H    ; Switch to Bank 1
       ; Use R0-R7 of Bank 1
       POP PSW          ; Restore Bank 0
       RETI

2. USE DIRECT ADDRESSING:
   ; Slower:
   MOV A, R0
   ADD A, R1
   
   ; Faster:
   MOV A, 00H           ; Direct address of R0
   ADD A, 01H           ; Direct address of R1

3. AVOID UNNECESSARY JUMPS:
   ; Before:
   MOV A, 30H
   CJNE A, #10, SKIP
   SJMP NEXT
   SKIP:
       ; code
   NEXT:
   
   ; After:
   MOV A, 30H
   CJNE A, #10, SKIP
   ; code for equal
   SKIP:

4. USE DJNZ FOR LOOPS:
   ; Instead of:
   MOV R7, #10
   LOOP:
       DEC R7
       JNZ LOOP
   
   ; Use:
   MOV R7, #10
   LOOP:
       DJNZ R7, LOOP

5. UNROLL SMALL LOOPS:
   ; Instead of loop for 3 iterations:
   MOV P1, #01H
   MOV P1, #02H
   MOV P1, #04H
   ; Faster than looping

6. USE BIT OPERATIONS:
   ; Instead of:
   MOV A, P1
   ANL A, #01H
   JZ SKIP
   
   ; Use:
   JNB P1.0, SKIP
```

---

## 📋 Summary Table

| Application | Key Concepts | Typical Frequency/Rate |
|-------------|--------------|------------------------|
| PWM LED | Duty cycle 0-100% | 100 Hz - 1 kHz |
| DC Motor | H-bridge, PWM speed | 1-20 kHz PWM |
| Stepper | Step sequence table | 100-1000 steps/sec |
| Servo | 20ms period, 1-2ms pulse | 50 Hz |
| ADC Sampling | Conversion + read | 10-1000 samples/sec |

---

## ❓ Quick Revision Questions

1. **What is the relationship between PWM duty cycle and LED brightness?**
   <details>
   <summary>Show Answer</summary>
   LED brightness is proportional to duty cycle. 0% = OFF, 100% = full brightness. At 50% duty cycle, LED appears half as bright due to persistence of vision averaging the ON/OFF times.
   </details>

2. **Why is hysteresis used in temperature control?**
   <details>
   <summary>Show Answer</summary>
   Hysteresis (dead band) prevents rapid ON/OFF switching when temperature is near the set point. Without it, noise or small fluctuations would cause the heater/fan to toggle continuously, reducing lifespan and efficiency.
   </details>

3. **What step sequence gives maximum torque in stepper motors?**
   <details>
   <summary>Show Answer</summary>
   Full-step 2-phase drive (two coils energized at a time) provides approximately 40% more torque than wave drive (one coil at a time), as both coil pairs contribute to holding torque.
   </details>

4. **How do you calculate servo position from pulse width?**
   <details>
   <summary>Show Answer</summary>
   Position = (Pulse Width - 1ms) × 180° / 1ms. For 1.5ms pulse: (1.5-1) × 180 = 90°. Range is 1ms (0°) to 2ms (180°) within a 20ms period.
   </details>

5. **Why use timer interrupt for PWM instead of delay loops?**
   <details>
   <summary>Show Answer</summary>
   Timer interrupts allow the main program to continue executing while PWM runs in background. Delay loops block execution and can't maintain accurate timing during other operations. Timer-based PWM provides consistent frequency regardless of main program activities.
   </details>

6. **What is the advantage of using register bank switching in ISRs?**
   <details>
   <summary>Show Answer</summary>
   Register bank switching (changing RS1:RS0 in PSW) instantly gives access to a fresh set of R0-R7 registers without PUSH/POP overhead. Only PSW needs to be saved/restored, making ISR entry/exit much faster.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next Unit |
|----------|-----|-----------|
| [8.5 Interfacing Examples](05-interfacing-examples.md) | [Unit 8 Index](README.md) | [Unit 9: Advanced Processors](../09-Advanced-Processors/README.md) |

---

*[← Previous: Interfacing Examples](05-interfacing-examples.md) | [Back to Unit 8 Index](README.md)*
