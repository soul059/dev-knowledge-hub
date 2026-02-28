# Chapter 10.3: Motor Control Systems

## 📚 Chapter Overview

This chapter covers practical motor control projects, including DC motors, stepper motors, and servo motors, with emphasis on PWM speed control and position control.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Control DC motor speed using PWM
- Implement stepper motor sequencing
- Position servo motors precisely
- Design H-bridge motor driver circuits
- Implement PID control for motors

---

## 1. DC Motor Control

### 1.1 DC Motor Basics and H-Bridge

```
DC MOTOR CONTROL FUNDAMENTALS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

DC MOTOR CHARACTERISTICS:
───────────────────────
• Speed proportional to applied voltage
• Direction depends on voltage polarity
• Requires current capability (100mA - several amps)
• Generates back-EMF when spinning

H-BRIDGE OPERATION:
─────────────────

Forward (S1,S4 ON):         Reverse (S2,S3 ON):
         +V                          +V
          │                           │
    ┌─────┼─────┐               ┌─────┼─────┐
    │     │     │               │     │     │
   [S1]  ───   [S2]            [S1]  ───   [S2]
    │ON   │     │               │     │    ON│
    ├─────┴─────┤               ├─────┴─────┤
    │           │               │           │
    │   ┌───┐   │               │   ┌───┐   │
    │   │ M │   │               │   │ M │   │
    │   └───┘   │               │   └───┘   │
    │           │               │           │
    ├─────┬─────┤               ├─────┬─────┤
   [S3]   │    [S4]            [S3]   │    [S4]
    │     │    ON│              ON│   │     │
    └─────┼─────┘               └─────┼─────┘
          │                           │
         GND                         GND

    Current: → M →                Current: ← M ←

BRAKE (S1,S2 OFF, S3,S4 ON or vice versa):
    Motor windings shorted - rapid stop
    
COAST (All switches OFF):
    Motor freewheels - slow stop
```

### 1.2 L293D Motor Driver

```
L293D H-BRIDGE DRIVER IC
━━━━━━━━━━━━━━━━━━━━━━━━

L293D PIN CONFIGURATION:
─────────────────────

            ┌───────────┐
    1,2EN ─┤1        16├─ VCC1 (5V Logic)
      1A  ─┤2        15├─ 4A
      1Y  ─┤3        14├─ 4Y
     GND  ─┤4        13├─ GND
     GND  ─┤5        12├─ GND
      2Y  ─┤6        11├─ 3Y
      2A  ─┤7        10├─ 3A
VCC2(Motor)┤8         9├─ 3,4EN
            └───────────┘

CIRCUIT CONNECTION:
─────────────────

                    +5V                    +12V (Motor)
                     │                        │
                     │                        │
    ┌────────────────┼────────────────────────┼────────────────┐
    │                │                        │                │
    │          ┌─────┴──────────────────┬─────┴────────┐      │
    │          │                        │    L293D     │      │
    │          │   ┌──────────────────┐ │              │      │
    │   P1.0 ──┼──►│ 1,2EN       VCC2 │─┘              │      │
    │          │   │                  │                │      │
    │   P1.1 ──┼──►│ 1A          VCC1 │─── +5V        │      │
    │          │   │                  │                │      │
    │   P1.2 ──┼──►│ 2A          1Y   │───────┐       │      │
    │          │   │                  │       │       │      │
    │          │   │             2Y   │───┐   │       │      │
    │          │   │                  │   │   │       │      │
    │          │   │            GND   │   │   │       │      │
    │          │   └──────────────────┘   │   │       │      │
    │          │                          │   │       │      │
    │          └──────────────────────────┼───┼───────┘      │
    │                                     │   │              │
    │                                   ┌─┴───┴─┐            │
    │                                   │   M   │ DC Motor   │
    │                                   └───────┘            │
    │                                                        │
    └────────────────────────────────────────────────────────┘

CONTROL LOGIC:
────────────

EN   1A   2A   │ Motor Action
────────────────────────────
0    X    X    │ Stop (coast)
1    0    0    │ Brake
1    0    1    │ Reverse
1    1    0    │ Forward
1    1    1    │ Brake
```

### 1.3 PWM Speed Control

```c
/*
 * DC MOTOR PWM SPEED CONTROL
 * Using Timer for PWM generation
 */

#include <reg51.h>

sbit MOTOR_EN = P1^0;    // L293D Enable (PWM)
sbit MOTOR_A  = P1^1;    // Direction A
sbit MOTOR_B  = P1^2;    // Direction B

unsigned char pwm_duty = 0;     // 0-100%
volatile unsigned char pwm_counter = 0;

/*
 * TIMER 0 ISR - PWM Generation
 * ~10 kHz PWM frequency
 */
void timer0_isr(void) interrupt 1
{
    TH0 = 0xFF;    // ~100µs at 11.0592 MHz
    TL0 = 0xA4;
    
    pwm_counter++;
    if(pwm_counter >= 100)
        pwm_counter = 0;
    
    // PWM output
    if(pwm_counter < pwm_duty)
        MOTOR_EN = 1;
    else
        MOTOR_EN = 0;
}

/*
 * SET MOTOR SPEED AND DIRECTION
 * speed: -100 to +100 (negative = reverse)
 */
void motor_control(signed char speed)
{
    if(speed > 0)
    {
        // Forward
        MOTOR_A = 1;
        MOTOR_B = 0;
        pwm_duty = speed;
    }
    else if(speed < 0)
    {
        // Reverse
        MOTOR_A = 0;
        MOTOR_B = 1;
        pwm_duty = -speed;
    }
    else
    {
        // Stop
        MOTOR_A = 0;
        MOTOR_B = 0;
        pwm_duty = 0;
    }
}

/*
 * BRAKE MOTOR
 */
void motor_brake(void)
{
    MOTOR_A = 1;
    MOTOR_B = 1;
    MOTOR_EN = 1;
}

void main(void)
{
    unsigned char adc_value;
    signed char speed;
    
    // Initialize Timer 0 for PWM
    TMOD = 0x01;    // Mode 1
    TH0 = 0xFF;
    TL0 = 0xA4;
    TR0 = 1;
    ET0 = 1;
    EA = 1;
    
    lcd_init();
    adc_init();
    
    while(1)
    {
        // Read potentiometer for speed control
        adc_value = adc_read();
        
        // Convert to -100 to +100 range
        // Center position (128) = stop
        speed = (signed char)((adc_value - 128) * 100 / 128);
        
        motor_control(speed);
        
        // Display
        lcd_goto(0, 0);
        lcd_string("Speed: ");
        lcd_number(speed);
        lcd_string("%  ");
        
        delay_ms(50);
    }
}
```

---

## 2. Stepper Motor Control

### 2.1 Stepper Motor Basics

```
STEPPER MOTOR FUNDAMENTALS
━━━━━━━━━━━━━━━━━━━━━━━━━━

TYPES:
─────
• Unipolar: 5 or 6 wires, simpler drive
• Bipolar: 4 wires, more torque

COMMON SPECIFICATIONS (28BYJ-48):
──────────────────────────────
• Type: Unipolar
• Steps: 32 per revolution (motor)
• Gear ratio: 64:1
• Total steps: 2048 per output revolution
• Step angle: 5.625°/64 = 0.176° per step
• Rated voltage: 5V DC
• Phases: 4

WINDING CONFIGURATION:
────────────────────

Unipolar (28BYJ-48):
                   Common (+5V)
                       │
           ┌───────────┼───────────┐
           │           │           │
        ┌──┴──┐     ┌──┴──┐     ┌──┴──┐
        │Coil │     │Coil │     │Center
        │  A  │     │  B  │     │ Tap │
        └──┬──┘     └──┬──┘     └─────┘
           │           │
           │  ┌────────┘
           │  │     ┌───────────┐
           │  │     │           │
        ┌──┴──┴─────┴───────────┴──┐
        │ A    B    C    D         │
        │ Coil Coil Coil Coil      │
        │ End  End  End  End       │
        └──────────────────────────┘
```

### 2.2 Stepping Sequences

```
STEPPER MOTOR DRIVE SEQUENCES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. FULL STEP (Wave Drive) - One phase at a time:
─────────────────────────────────────────────

Step  │  A   B   C   D  │ Angle
──────┼─────────────────┼──────
  1   │  1   0   0   0  │   0°
  2   │  0   1   0   0  │  90°
  3   │  0   0   1   0  │ 180°
  4   │  0   0   0   1  │ 270°
  1   │  1   0   0   0  │ 360°

2. FULL STEP (Two Phase) - Two phases at a time:
───────────────────────────────────────────────

Step  │  A   B   C   D  │ Angle
──────┼─────────────────┼──────
  1   │  1   1   0   0  │  45°
  2   │  0   1   1   0  │ 135°
  3   │  0   0   1   1  │ 225°
  4   │  1   0   0   1  │ 315°

Higher torque than wave drive

3. HALF STEP - Alternating one and two phases:
─────────────────────────────────────────────

Step  │  A   B   C   D  │ Angle
──────┼─────────────────┼──────
  1   │  1   0   0   0  │   0°
  2   │  1   1   0   0  │  45°
  3   │  0   1   0   0  │  90°
  4   │  0   1   1   0  │ 135°
  5   │  0   0   1   0  │ 180°
  6   │  0   0   1   1  │ 225°
  7   │  0   0   0   1  │ 270°
  8   │  1   0   0   1  │ 315°

Double resolution (8 steps per cycle)
```

### 2.3 Stepper Motor Driver Code

```c
/*
 * STEPPER MOTOR CONTROL (28BYJ-48 with ULN2003)
 */

#include <reg51.h>

// Stepper motor connections
#define STEPPER_PORT P2

// Step sequences
code unsigned char wave_drive[4] = {0x01, 0x02, 0x04, 0x08};
code unsigned char full_step[4]  = {0x03, 0x06, 0x0C, 0x09};
code unsigned char half_step[8]  = {0x01, 0x03, 0x02, 0x06, 
                                     0x04, 0x0C, 0x08, 0x09};

unsigned char current_step = 0;
unsigned int step_delay = 2;  // ms between steps

/*
 * MOVE STEPPER - Single step
 * direction: 1 = CW, 0 = CCW
 */
void stepper_step(bit direction)
{
    if(direction)
    {
        current_step++;
        if(current_step >= 8)
            current_step = 0;
    }
    else
    {
        if(current_step == 0)
            current_step = 7;
        else
            current_step--;
    }
    
    STEPPER_PORT = half_step[current_step];
}

/*
 * MOVE STEPPER - Multiple steps
 * steps: Number of steps to move
 * direction: 1 = CW, 0 = CCW
 */
void stepper_move(unsigned int steps, bit direction)
{
    unsigned int i;
    
    for(i = 0; i < steps; i++)
    {
        stepper_step(direction);
        delay_ms(step_delay);
    }
}

/*
 * ROTATE STEPPER - By degrees
 * degrees: Angle to rotate
 * direction: 1 = CW, 0 = CCW
 */
void stepper_rotate(unsigned int degrees, bit direction)
{
    unsigned int steps;
    
    // 28BYJ-48: 4096 steps = 360°
    // Steps = degrees × 4096 / 360 ≈ degrees × 11.38
    steps = (unsigned long)degrees * 4096 / 360;
    
    stepper_move(steps, direction);
}

/*
 * SET STEPPER SPEED
 * rpm: Revolutions per minute (max ~15 for 28BYJ-48)
 */
void stepper_set_speed(unsigned char rpm)
{
    // Time for one step = 60s / (rpm × steps_per_rev)
    // 28BYJ-48: 4096 steps per revolution
    // step_delay (ms) = 60000 / (rpm × 4096)
    step_delay = 60000UL / ((unsigned long)rpm * 4096);
    
    if(step_delay < 1)
        step_delay = 1;  // Minimum delay
}

/*
 * STOP STEPPER - Turn off all coils
 */
void stepper_stop(void)
{
    STEPPER_PORT = 0x00;
}

void main(void)
{
    unsigned char key;
    
    lcd_init();
    keypad_init();
    
    lcd_string("Stepper Control");
    
    while(1)
    {
        key = keypad_scan();
        
        switch(key)
        {
            case '1':
                // Rotate 90° CW
                lcd_goto(1, 0);
                lcd_string("90 CW      ");
                stepper_rotate(90, 1);
                break;
                
            case '2':
                // Rotate 90° CCW
                lcd_goto(1, 0);
                lcd_string("90 CCW     ");
                stepper_rotate(90, 0);
                break;
                
            case '3':
                // Full rotation CW
                lcd_goto(1, 0);
                lcd_string("360 CW     ");
                stepper_rotate(360, 1);
                break;
                
            case '4':
                // Full rotation CCW
                lcd_goto(1, 0);
                lcd_string("360 CCW    ");
                stepper_rotate(360, 0);
                break;
                
            case '*':
                stepper_stop();
                lcd_goto(1, 0);
                lcd_string("Stopped    ");
                break;
        }
    }
}
```

---

## 3. Servo Motor Control

### 3.1 Servo Motor Basics

```
SERVO MOTOR CONTROL
━━━━━━━━━━━━━━━━━━━

SERVO CHARACTERISTICS:
────────────────────
• Position controlled by PWM pulse width
• Internal feedback (potentiometer)
• Typical range: 0° to 180°
• PWM frequency: 50 Hz (20ms period)
• Pulse width: 1ms (0°) to 2ms (180°)

PWM SIGNAL TIMING:
────────────────

                    20ms period
    ├────────────────────────────────────────┤
    
    ┌──┐                                     ┌──┐
    │  │                                     │  │
────┘  └─────────────────────────────────────┘  └─────

    │←→│
    1-2ms
    pulse

Position vs Pulse Width:
    0°:   1.0 ms
    45°:  1.25 ms
    90°:  1.5 ms
    135°: 1.75 ms
    180°: 2.0 ms

CIRCUIT:
───────

         +5V (from external source, NOT from MCU)
          │
          │    ┌─────────────────────┐
          └────┤ VCC (Red)           │
               │                     │
    MCU P1.0 ──┤ Signal (Orange/White)│  SG90 Servo
               │                     │
          ┌────┤ GND (Brown/Black)   │
          │    └─────────────────────┘
          │
         GND ─────────────────────────── MCU GND

Note: Connect servo GND to MCU GND for common reference!
```

### 3.2 Servo Control Code

```c
/*
 * SERVO MOTOR CONTROL
 * Using Timer for precise PWM
 */

#include <reg51.h>

sbit SERVO = P1^0;

unsigned int servo_pulse = 1500;  // µs (center position)

/*
 * TIMER 0 ISR - PWM Generation
 * Called every 100µs
 */
volatile unsigned int pwm_time = 0;

void timer0_isr(void) interrupt 1
{
    TH0 = 0xFF;    // ~100µs reload
    TL0 = 0xA4;
    
    pwm_time += 100;
    
    if(pwm_time >= 20000)  // 20ms period
    {
        pwm_time = 0;
        SERVO = 1;  // Start pulse
    }
    
    if(pwm_time >= servo_pulse)
    {
        SERVO = 0;  // End pulse
    }
}

/*
 * SET SERVO ANGLE
 * angle: 0 to 180 degrees
 */
void servo_set_angle(unsigned char angle)
{
    // Map 0-180° to 1000-2000µs
    // pulse = 1000 + (angle × 1000 / 180)
    servo_pulse = 1000 + ((unsigned int)angle * 1000 / 180);
}

/*
 * SET SERVO PULSE WIDTH DIRECTLY
 * pulse_us: 500 to 2500 µs (extended range)
 */
void servo_set_pulse(unsigned int pulse_us)
{
    if(pulse_us < 500)
        pulse_us = 500;
    if(pulse_us > 2500)
        pulse_us = 2500;
    
    servo_pulse = pulse_us;
}

/*
 * SWEEP SERVO - Smooth movement
 * from_angle, to_angle: Start and end positions
 * speed: Delay between steps (ms)
 */
void servo_sweep(unsigned char from_angle, 
                 unsigned char to_angle, 
                 unsigned char speed)
{
    signed char step = (to_angle > from_angle) ? 1 : -1;
    unsigned char current = from_angle;
    
    while(current != to_angle)
    {
        servo_set_angle(current);
        delay_ms(speed);
        current += step;
    }
    servo_set_angle(to_angle);
}

void main(void)
{
    unsigned char adc_value;
    unsigned char angle;
    char buf[16];
    
    // Initialize Timer 0
    TMOD = 0x01;
    TH0 = 0xFF;
    TL0 = 0xA4;
    TR0 = 1;
    ET0 = 1;
    EA = 1;
    
    lcd_init();
    adc_init();
    
    // Center servo
    servo_set_angle(90);
    
    lcd_string("Servo Control");
    delay_ms(1000);
    
    while(1)
    {
        // Read potentiometer
        adc_value = adc_read();
        
        // Map to angle
        angle = (unsigned int)adc_value * 180 / 255;
        
        // Set servo position
        servo_set_angle(angle);
        
        // Display
        lcd_goto(0, 0);
        sprintf(buf, "Angle: %3d", angle);
        lcd_string(buf);
        
        lcd_goto(1, 0);
        sprintf(buf, "Pulse: %4dus", servo_pulse);
        lcd_string(buf);
        
        delay_ms(20);
    }
}
```

---

## 4. PID Motor Control

### 4.1 PID Controller Theory

```
PID CONTROL FOR MOTORS
━━━━━━━━━━━━━━━━━━━━━━

PID FORMULA:
───────────

Output = Kp × Error + Ki × ∫Error dt + Kd × dError/dt

Where:
• Error = Setpoint - Measured Value
• Kp = Proportional gain
• Ki = Integral gain  
• Kd = Derivative gain

BLOCK DIAGRAM:
────────────

         Error           ┌─────────────┐
Setpoint ──┬───────────►│      P      │─────┐
  (SP)     │             │   Kp × e    │     │
           │             └─────────────┘     │
           │                                 │
           │             ┌─────────────┐     │         ┌──────────┐
           ├───────────►│      I      │─────┼────────►│  Motor   │─────► Speed
           │             │  Ki × ∫e dt │     │ Output  │          │
           │             └─────────────┘     │ (PWM)   └──────────┘
           │                                 │              │
           │             ┌─────────────┐     │              │
           └───────────►│      D      │─────┘              │
                        │  Kd × de/dt │                    │
                        └─────────────┘                    │
                                                           │
           ┌───────────────────────────────────────────────┘
           │
           │        ┌──────────────┐
Measured ◄─┴───────┤   Encoder    │
  Value            │   Sensor     │
  (PV)             └──────────────┘

TUNING EFFECTS:
─────────────

┌──────────────┬───────────────────────────────────────────────────┐
│ Parameter    │ Effect                                            │
├──────────────┼───────────────────────────────────────────────────┤
│ Increase Kp  │ Faster response, may overshoot, can oscillate    │
│ Increase Ki  │ Eliminates steady-state error, may overshoot     │
│ Increase Kd  │ Reduces overshoot, dampens oscillations          │
└──────────────┴───────────────────────────────────────────────────┘
```

### 4.2 PID Implementation

```c
/*
 * PID MOTOR SPEED CONTROLLER
 * With encoder feedback
 */

#include <reg51.h>

// PID Parameters (tune for your motor)
float Kp = 2.0;
float Ki = 0.5;
float Kd = 0.1;

// PID State
float setpoint = 0;
float error = 0;
float last_error = 0;
float integral = 0;
float derivative = 0;
float output = 0;

// Limits
#define OUTPUT_MAX  100
#define OUTPUT_MIN -100
#define INTEGRAL_MAX 1000

// Encoder
volatile unsigned int encoder_count = 0;
volatile unsigned int last_encoder = 0;
volatile int motor_speed = 0;  // RPM

/*
 * EXTERNAL INTERRUPT 0 - Encoder pulse
 */
void ext0_isr(void) interrupt 0
{
    encoder_count++;
}

/*
 * TIMER 1 ISR - Speed calculation (every 100ms)
 */
void timer1_isr(void) interrupt 3
{
    unsigned int pulses;
    
    // Reload timer
    TH1 = 0x3C;
    TL1 = 0xB0;  // ~100ms at 11.0592 MHz
    
    // Calculate speed
    pulses = encoder_count - last_encoder;
    last_encoder = encoder_count;
    
    // Convert to RPM
    // Example: 20 pulses/rev, 100ms sample
    // RPM = pulses × 10 (samples/sec) × 60 / 20 (pulses/rev)
    motor_speed = pulses * 30;  // Simplified for 20 p/r
}

/*
 * PID COMPUTE
 * Call every control loop iteration
 */
void pid_compute(float measured)
{
    // Calculate error
    error = setpoint - measured;
    
    // Proportional term
    float P = Kp * error;
    
    // Integral term with anti-windup
    integral += error;
    if(integral > INTEGRAL_MAX) integral = INTEGRAL_MAX;
    if(integral < -INTEGRAL_MAX) integral = -INTEGRAL_MAX;
    float I = Ki * integral;
    
    // Derivative term
    derivative = error - last_error;
    float D = Kd * derivative;
    
    // Calculate output
    output = P + I + D;
    
    // Limit output
    if(output > OUTPUT_MAX) output = OUTPUT_MAX;
    if(output < OUTPUT_MIN) output = OUTPUT_MIN;
    
    // Save error for next iteration
    last_error = error;
}

/*
 * SET MOTOR OUTPUT
 */
void set_motor(float out)
{
    if(out >= 0)
    {
        // Forward
        MOTOR_DIR = 1;
        pwm_duty = (unsigned char)out;
    }
    else
    {
        // Reverse
        MOTOR_DIR = 0;
        pwm_duty = (unsigned char)(-out);
    }
}

void main(void)
{
    char buf[16];
    
    // Initialize
    timer_init();
    encoder_init();
    motor_init();
    lcd_init();
    
    // Set target speed
    setpoint = 100;  // 100 RPM
    
    while(1)
    {
        // Run PID
        pid_compute((float)motor_speed);
        
        // Apply output
        set_motor(output);
        
        // Display
        lcd_goto(0, 0);
        sprintf(buf, "Set:%3d RPM", (int)setpoint);
        lcd_string(buf);
        
        lcd_goto(1, 0);
        sprintf(buf, "Act:%3d RPM", motor_speed);
        lcd_string(buf);
        
        delay_ms(100);  // Control loop period
    }
}
```

---

## 5. Complete Motor Control Project

### 5.1 Robot Motor Controller

```c
/*
 * DUAL MOTOR ROBOT CONTROLLER
 * 2-wheel differential drive
 */

#include <reg51.h>

// Motor A (Left)
sbit MOTOR_A_EN = P1^0;
sbit MOTOR_A_IN1 = P1^1;
sbit MOTOR_A_IN2 = P1^2;

// Motor B (Right)  
sbit MOTOR_B_EN = P1^3;
sbit MOTOR_B_IN1 = P1^4;
sbit MOTOR_B_IN2 = P1^5;

// PWM duty cycles
unsigned char pwm_a = 0;
unsigned char pwm_b = 0;
volatile unsigned char pwm_counter = 0;

typedef enum {
    STOP,
    FORWARD,
    BACKWARD,
    LEFT,
    RIGHT,
    PIVOT_LEFT,
    PIVOT_RIGHT
} RobotDirection;

/*
 * TIMER ISR - Dual PWM Generation
 */
void timer0_isr(void) interrupt 1
{
    TH0 = 0xFF;
    TL0 = 0xA4;
    
    pwm_counter++;
    if(pwm_counter >= 100)
        pwm_counter = 0;
    
    MOTOR_A_EN = (pwm_counter < pwm_a) ? 1 : 0;
    MOTOR_B_EN = (pwm_counter < pwm_b) ? 1 : 0;
}

/*
 * SET MOTOR SPEEDS AND DIRECTIONS
 */
void motor_a_set(signed char speed)
{
    if(speed > 0) {
        MOTOR_A_IN1 = 1;
        MOTOR_A_IN2 = 0;
        pwm_a = speed;
    } else if(speed < 0) {
        MOTOR_A_IN1 = 0;
        MOTOR_A_IN2 = 1;
        pwm_a = -speed;
    } else {
        MOTOR_A_IN1 = 0;
        MOTOR_A_IN2 = 0;
        pwm_a = 0;
    }
}

void motor_b_set(signed char speed)
{
    if(speed > 0) {
        MOTOR_B_IN1 = 1;
        MOTOR_B_IN2 = 0;
        pwm_b = speed;
    } else if(speed < 0) {
        MOTOR_B_IN1 = 0;
        MOTOR_B_IN2 = 1;
        pwm_b = -speed;
    } else {
        MOTOR_B_IN1 = 0;
        MOTOR_B_IN2 = 0;
        pwm_b = 0;
    }
}

/*
 * ROBOT MOVEMENT FUNCTIONS
 */
void robot_move(RobotDirection dir, unsigned char speed)
{
    switch(dir)
    {
        case STOP:
            motor_a_set(0);
            motor_b_set(0);
            break;
            
        case FORWARD:
            motor_a_set(speed);
            motor_b_set(speed);
            break;
            
        case BACKWARD:
            motor_a_set(-speed);
            motor_b_set(-speed);
            break;
            
        case LEFT:  // Gradual left turn
            motor_a_set(speed / 2);
            motor_b_set(speed);
            break;
            
        case RIGHT:  // Gradual right turn
            motor_a_set(speed);
            motor_b_set(speed / 2);
            break;
            
        case PIVOT_LEFT:  // Spin in place left
            motor_a_set(-speed);
            motor_b_set(speed);
            break;
            
        case PIVOT_RIGHT:  // Spin in place right
            motor_a_set(speed);
            motor_b_set(-speed);
            break;
    }
}

void main(void)
{
    unsigned char ir_left, ir_right;
    
    // Initialize
    timer_init();
    lcd_init();
    
    lcd_string("Line Follower");
    delay_ms(1000);
    
    while(1)
    {
        // Read IR sensors
        ir_left = IR_LEFT;
        ir_right = IR_RIGHT;
        
        // Line following logic
        if(ir_left == 0 && ir_right == 0)
        {
            // Both on line - go forward
            robot_move(FORWARD, 70);
        }
        else if(ir_left == 1 && ir_right == 0)
        {
            // Left off line - turn left
            robot_move(LEFT, 60);
        }
        else if(ir_left == 0 && ir_right == 1)
        {
            // Right off line - turn right
            robot_move(RIGHT, 60);
        }
        else
        {
            // Both off line - lost, search
            robot_move(PIVOT_LEFT, 50);
        }
        
        delay_ms(10);
    }
}
```

---

## 📋 Motor Comparison Table

| Feature | DC Motor | Stepper Motor | Servo Motor |
|---------|----------|---------------|-------------|
| Control | Voltage (PWM) | Pulse sequence | Pulse width |
| Position | Open loop | Open loop | Closed loop |
| Speed | Variable | Fixed steps | Fixed |
| Torque | Varies with speed | Constant | High at stall |
| Precision | Low | High | Very high |
| Cost | Low | Medium | Medium |
| Application | Wheels, fans | CNC, 3D printers | Robotics, RC |

---

## ❓ Quick Revision Questions

1. **How does an H-bridge control motor direction?**
   <details>
   <summary>Show Answer</summary>
   H-bridge has 4 switches arranged in H pattern. Activating diagonal pairs (S1+S4 or S2+S3) allows current to flow through motor in opposite directions. Both high = brake, both low = coast.
   </details>

2. **What is the difference between half-step and full-step modes?**
   <details>
   <summary>Show Answer</summary>
   Full-step energizes one or two phases per step (4 steps/cycle). Half-step alternates between one and two phases (8 steps/cycle). Half-step provides double resolution but lower torque.
   </details>

3. **How does a servo motor maintain position?**
   <details>
   <summary>Show Answer</summary>
   Servo has internal feedback (potentiometer). Controller compares pot position to input pulse width and drives motor to reduce error. Continuous feedback maintains position against external forces.
   </details>

4. **What causes motor oscillation and how does PID prevent it?**
   <details>
   <summary>Show Answer</summary>
   Oscillation from too aggressive proportional control. Derivative term (Kd) anticipates change by responding to error rate, providing damping before overshoot occurs.
   </details>

5. **Why use PWM for motor speed control?**
   <details>
   <summary>Show Answer</summary>
   PWM provides efficient speed control by rapidly switching full voltage on/off. Motor's inductance smooths current. More efficient than variable voltage (resistive losses). Digital control is easy.
   </details>

6. **What is back-EMF and why does it matter?**
   <details>
   <summary>Show Answer</summary>
   Back-EMF is voltage generated by spinning motor acting as generator. Opposes applied voltage. Limits current naturally at speed. Can damage driver if motor suddenly stops (voltage spike). Add flyback diodes.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [10.2 Sensor Interfacing Projects](02-sensor-interfacing-projects.md) | [Unit 10 Index](README.md) | [10.4 Communication Projects](04-communication-projects.md) |

---

*[← Previous: Sensor Interfacing Projects](02-sensor-interfacing-projects.md) | [Next: Communication Projects →](04-communication-projects.md)*
