# Chapter 6.5: Motor Control

## Introduction

Motors are fundamental actuators in embedded systems. This chapter covers DC motors, stepper motors, and servo motors, along with the driver circuits and control techniques needed to interface them with microcontrollers.

---

## DC Motor Fundamentals

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DC MOTOR BASICS                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  DC Motor: Converts electrical energy to rotational motion         │
│                                                                      │
│  CHARACTERISTICS:                                                    │
│  • Speed proportional to voltage                                    │
│  • Torque proportional to current                                   │
│  • Direction depends on polarity                                    │
│  • Typical: 3V-24V, 100mA-several amps                             │
│                                                                      │
│  CONTROL REQUIREMENTS:                                               │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Control    │ Method                                        │     │
│  ├────────────────────────────────────────────────────────────┤     │
│  │ Speed      │ PWM (vary average voltage)                    │     │
│  │ Direction  │ H-Bridge (reverse polarity)                   │     │
│  │ Braking    │ Short motor terminals                         │     │
│  │ Position   │ Encoder feedback + control loop               │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  WHY CAN'T MCU DRIVE MOTOR DIRECTLY?                                │
│                                                                      │
│  MCU GPIO: 3.3V, ~20mA max                                         │
│  DC Motor: 6-12V, 500mA-2A typical                                 │
│                                                                      │
│  Need driver circuit to:                                            │
│  • Provide higher voltage                                           │
│  • Supply more current                                              │
│  • Protect MCU from back-EMF                                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## H-Bridge Motor Driver

```
┌─────────────────────────────────────────────────────────────────────┐
│                    H-BRIDGE CIRCUIT                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│             VCC                                                      │
│              │                                                       │
│        ┌─────┴─────┐                                                │
│        │           │                                                │
│       ─┴─ Q1     ─┴─ Q3                                             │
│        │           │                                                │
│    IN1─┤           ├─IN3                                            │
│        │     M     │                                                │
│        ├───┤   ├───┤                                                │
│        │   Motor   │                                                │
│    IN2─┤           ├─IN4                                            │
│        │           │                                                │
│       ─┴─ Q2     ─┴─ Q4                                             │
│        │           │                                                │
│        └─────┬─────┘                                                │
│              │                                                       │
│             GND                                                      │
│                                                                      │
│  OPERATION MODES:                                                    │
│  ┌──────────────────────────────────────────────────────────┐       │
│  │ Q1  │ Q2  │ Q3  │ Q4  │ Motor Action                    │       │
│  ├──────────────────────────────────────────────────────────┤       │
│  │ ON  │ OFF │ OFF │ ON  │ Forward (current L→R)           │       │
│  │ OFF │ ON  │ ON  │ OFF │ Reverse (current R→L)           │       │
│  │ OFF │ OFF │ OFF │ OFF │ Coast (free running)            │       │
│  │ ON  │ OFF │ ON  │ OFF │ Brake (motor shorted)           │       │
│  │ OFF │ ON  │ OFF │ ON  │ Brake (motor shorted)           │       │
│  │ ON  │ ON  │ x   │ x   │ SHOOT-THROUGH (DAMAGE!)         │       │
│  └──────────────────────────────────────────────────────────┘       │
│                                                                      │
│  NEVER turn on Q1 and Q2 (or Q3 and Q4) simultaneously!            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### L298N Motor Driver

```
┌─────────────────────────────────────────────────────────────────────┐
│                    L298N DUAL H-BRIDGE                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│              ┌───────────────────────┐                              │
│              │        L298N          │                              │
│              │                       │                              │
│  Motor A+ ◀──┤ OUT1           OUT3  ├──▶ Motor B+                   │
│  Motor A- ◀──┤ OUT2           OUT4  ├──▶ Motor B-                   │
│              │                       │                              │
│  IN1 ───────▶│ IN1             IN3  │◀─────── IN3                   │
│  IN2 ───────▶│ IN2             IN4  │◀─────── IN4                   │
│              │                       │                              │
│  ENA ───────▶│ ENA             ENB  │◀─────── ENB                   │
│  (PWM)       │                       │         (PWM)                │
│              │      VCC   GND        │                              │
│              └───────┬─────┬─────────┘                              │
│                      │     │                                        │
│                     +5-35V GND                                      │
│                                                                      │
│  CONTROL TABLE (per motor):                                         │
│  ┌────────────────────────────────────────────────────────┐         │
│  │ EN  │ IN1 │ IN2 │ Action                               │         │
│  ├────────────────────────────────────────────────────────┤         │
│  │  0  │  X  │  X  │ Coast (free spin)                    │         │
│  │ PWM │  1  │  0  │ Forward at PWM speed                 │         │
│  │ PWM │  0  │  1  │ Reverse at PWM speed                 │         │
│  │  1  │  1  │  1  │ Brake                                │         │
│  │  1  │  0  │  0  │ Brake                                │         │
│  └────────────────────────────────────────────────────────┘         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### DC Motor Driver Code

```c
// L298N pin definitions
#define MOTOR_ENA   0   // PA0 (PWM)
#define MOTOR_IN1   1   // PA1
#define MOTOR_IN2   2   // PA2

typedef enum {
    MOTOR_STOP,
    MOTOR_FORWARD,
    MOTOR_REVERSE,
    MOTOR_BRAKE
} MotorDirection_t;

void motor_init(void) {
    // Configure IN1, IN2 as outputs
    gpio_mode_output(GPIOA, MOTOR_IN1);
    gpio_mode_output(GPIOA, MOTOR_IN2);
    
    // Configure ENA for PWM (TIM2 CH1)
    pwm_init(MOTOR_ENA, 20000);  // 20 kHz PWM
    
    motor_stop();
}

void motor_set_direction(MotorDirection_t dir) {
    switch (dir) {
        case MOTOR_STOP:
            GPIO_CLEAR(GPIOA, MOTOR_IN1);
            GPIO_CLEAR(GPIOA, MOTOR_IN2);
            break;
        case MOTOR_FORWARD:
            GPIO_SET(GPIOA, MOTOR_IN1);
            GPIO_CLEAR(GPIOA, MOTOR_IN2);
            break;
        case MOTOR_REVERSE:
            GPIO_CLEAR(GPIOA, MOTOR_IN1);
            GPIO_SET(GPIOA, MOTOR_IN2);
            break;
        case MOTOR_BRAKE:
            GPIO_SET(GPIOA, MOTOR_IN1);
            GPIO_SET(GPIOA, MOTOR_IN2);
            break;
    }
}

void motor_set_speed(uint8_t speed_percent) {
    pwm_set_duty(MOTOR_ENA, speed_percent);
}

void motor_stop(void) {
    motor_set_direction(MOTOR_STOP);
    motor_set_speed(0);
}

// Combined control
void motor_run(int8_t speed) {
    // speed: -100 to +100 (negative = reverse)
    if (speed > 0) {
        motor_set_direction(MOTOR_FORWARD);
        motor_set_speed(speed);
    } else if (speed < 0) {
        motor_set_direction(MOTOR_REVERSE);
        motor_set_speed(-speed);
    } else {
        motor_stop();
    }
}
```

---

## Stepper Motor

```
┌─────────────────────────────────────────────────────────────────────┐
│                    STEPPER MOTOR BASICS                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Stepper: Precise position control, moves in discrete steps        │
│                                                                      │
│  TYPES:                                                              │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Type       │ Wires │ Common Use                           │     │
│  ├────────────────────────────────────────────────────────────┤     │
│  │ Unipolar   │ 5,6   │ Simple circuits, center-tap coils   │     │
│  │ Bipolar    │ 4     │ Higher torque, needs H-bridge        │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  BIPOLAR STEPPER (4-wire):                                          │
│                                                                      │
│           ┌───────────────────────────────────────┐                 │
│           │              Stepper                  │                 │
│           │                                       │                 │
│    A+ ────┤  ┌─────┐      ┌─────┐                │                 │
│    A- ────┤  │Coil │      │Coil │                │                 │
│           │  │  A  │ Rotor│  B  │                │                 │
│    B+ ────┤  └─────┘   ●  └─────┘                │                 │
│    B- ────┤                                       │                 │
│           └───────────────────────────────────────┘                 │
│                                                                      │
│  FULL-STEP SEQUENCE:                                                 │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Step │ A+  │ A-  │ B+  │ B-  │ Coil A │ Coil B            │     │
│  ├────────────────────────────────────────────────────────────┤     │
│  │  1   │ +   │ -   │ +   │ -   │  +1    │  +1               │     │
│  │  2   │ -   │ +   │ +   │ -   │  -1    │  +1               │     │
│  │  3   │ -   │ +   │ -   │ +   │  -1    │  -1               │     │
│  │  4   │ +   │ -   │ -   │ +   │  +1    │  -1               │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  STEP ANGLE:                                                         │
│  • 1.8° per step = 200 steps per revolution (common)               │
│  • 0.9° per step = 400 steps per revolution                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Stepper Driving Modes

```
┌─────────────────────────────────────────────────────────────────────┐
│                    STEPPER DRIVE MODES                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  FULL-STEP (Wave Drive):          FULL-STEP (Two-Phase):           │
│  One coil at a time               Two coils at a time              │
│                                                                      │
│  Step 1: A          Step 1: A+B                                     │
│  Step 2: B          Step 2: B+A'                                    │
│  Step 3: A'         Step 3: A'+B'                                   │
│  Step 4: B'         Step 4: B'+A                                    │
│                                                                      │
│  Less torque        More torque, more current                       │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  HALF-STEP:                       MICROSTEPPING:                    │
│  8 steps per cycle                256+ steps per cycle              │
│                                                                      │
│  Combines wave + two-phase        PWM controls current ratio        │
│  1: A                             Sine/cosine current profiles      │
│  2: A+B                           Very smooth motion                │
│  3: B                             Requires current control          │
│  4: B+A'                                                            │
│  5: A'                            Position                          │
│  6: A'+B'                          ●                                │
│  7: B'                            ╱ ╲  Micro-                       │
│  8: B'+A                         ╱   ╲  steps                       │
│                                 ●     ●                             │
│  Half the step angle            Full steps                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Stepper Driver Code

```c
// A4988 stepper driver interface
#define STEP_PIN    0   // PA0
#define DIR_PIN     1   // PA1
#define ENABLE_PIN  2   // PA2

#define STEPS_PER_REV  200  // 1.8° stepper

void stepper_init(void) {
    gpio_mode_output(GPIOA, STEP_PIN);
    gpio_mode_output(GPIOA, DIR_PIN);
    gpio_mode_output(GPIOA, ENABLE_PIN);
    
    GPIO_CLEAR(GPIOA, STEP_PIN);
    GPIO_SET(GPIOA, ENABLE_PIN);  // Disable (active low)
}

void stepper_enable(bool enable) {
    if (enable) {
        GPIO_CLEAR(GPIOA, ENABLE_PIN);  // Active low
    } else {
        GPIO_SET(GPIOA, ENABLE_PIN);
    }
}

void stepper_set_direction(bool clockwise) {
    if (clockwise) {
        GPIO_SET(GPIOA, DIR_PIN);
    } else {
        GPIO_CLEAR(GPIOA, DIR_PIN);
    }
}

void stepper_step(void) {
    GPIO_SET(GPIOA, STEP_PIN);
    delay_us(5);  // Minimum pulse width
    GPIO_CLEAR(GPIOA, STEP_PIN);
}

void stepper_move_steps(int32_t steps, uint16_t delay_us) {
    stepper_enable(true);
    
    if (steps < 0) {
        stepper_set_direction(false);
        steps = -steps;
    } else {
        stepper_set_direction(true);
    }
    
    for (int32_t i = 0; i < steps; i++) {
        stepper_step();
        delay_us(delay_us);
    }
}

void stepper_rotate_degrees(float degrees, uint16_t delay_us) {
    int32_t steps = (int32_t)((degrees / 360.0f) * STEPS_PER_REV);
    stepper_move_steps(steps, delay_us);
}

// Example: Rotate 90° clockwise
stepper_rotate_degrees(90.0f, 1000);  // 1ms between steps
```

---

## Servo Motor (Recap)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SERVO MOTOR CONTROL                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Standard Hobby Servo:                                              │
│  • PWM frequency: 50 Hz (20 ms period)                             │
│  • Pulse width: 1-2 ms → 0-180° position                           │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────┐        │
│  │ Pulse Width │ Position │ Application                    │        │
│  ├─────────────────────────────────────────────────────────┤        │
│  │   1.0 ms    │   0°     │ Full left                      │        │
│  │   1.5 ms    │   90°    │ Center                         │        │
│  │   2.0 ms    │   180°   │ Full right                     │        │
│  └─────────────────────────────────────────────────────────┘        │
│                                                                      │
│  CONTINUOUS ROTATION SERVO:                                         │
│  • Modified servo for continuous rotation                          │
│  • 1.5 ms = stop                                                   │
│  • < 1.5 ms = one direction                                        │
│  • > 1.5 ms = other direction                                      │
│  • Pulse width controls speed, not position                        │
│                                                                      │
│  DIGITAL SERVO:                                                      │
│  • Faster response                                                  │
│  • Higher holding torque                                            │
│  • Same PWM interface                                               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Motor Driver ICs Comparison

| Driver | Type | Voltage | Current | Features |
|--------|------|---------|---------|----------|
| **L293D** | Dual H-bridge | 4.5-36V | 600mA | Built-in diodes |
| **L298N** | Dual H-bridge | 5-35V | 2A | No built-in diodes |
| **TB6612** | Dual H-bridge | 2.5-13.5V | 1.2A | Low Rds(on), efficient |
| **A4988** | Stepper | 8-35V | 2A | Microstepping |
| **DRV8825** | Stepper | 8.2-45V | 2.5A | 32 microsteps |
| **TMC2209** | Stepper | 4.75-28V | 2A | Silent, sensorless |

---

## Summary Table

| Motor Type | Control | Driver | Application |
|------------|---------|--------|-------------|
| **DC Brushed** | PWM speed, H-bridge direction | L298N, TB6612 | Wheels, fans |
| **Stepper** | Step/direction pulses | A4988, DRV8825 | CNC, 3D printer |
| **Servo** | 50Hz PWM, 1-2ms | Direct GPIO | Robotic arms |
| **BLDC** | 3-phase, Hall sensors | ESC | Drones, vehicles |

---

## Quick Revision Questions

1. **Why can't an MCU GPIO pin drive a DC motor directly?**

2. **Explain shoot-through in an H-bridge. How is it prevented?**

3. **What is the relationship between PWM duty cycle and DC motor speed?**

4. **A stepper motor has 1.8° step angle. How many steps for one revolution? For 45°?**

5. **Compare full-step, half-step, and microstepping modes.**

6. **What PWM signal would position a servo at 45° assuming 1ms=0° and 2ms=180°?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Chapter 6.4: Display Interfacing](04-display-interfacing.md) | [Unit Index](README.md) | [Chapter 6.6: Sensor Interfacing](06-sensor-interfacing.md) |

---
