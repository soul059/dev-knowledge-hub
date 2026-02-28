# Unit 6: Interfacing

## Overview

This unit covers practical hardware interfacing techniques for embedded systems. From basic GPIO control to advanced peripheral interfacing, these concepts bridge the gap between software and the physical world.

---

## Unit Structure

```
┌─────────────────────────────────────────────────────────────────────┐
│                    INTERFACING HIERARCHY                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                    ┌────────────────────┐                           │
│                    │     EMBEDDED       │                           │
│                    │  MICROCONTROLLER   │                           │
│                    └─────────┬──────────┘                           │
│                              │                                       │
│         ┌──────────┬─────────┼─────────┬──────────┐                 │
│         │          │         │         │          │                 │
│         ▼          ▼         ▼         ▼          ▼                 │
│    ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐          │
│    │  GPIO  │ │ Timers │ │ ADC/   │ │Display │ │ Motor  │          │
│    │        │ │  PWM   │ │ DAC    │ │        │ │ Drive  │          │
│    └────┬───┘ └────┬───┘ └────┬───┘ └────┬───┘ └────┬───┘          │
│         │          │         │         │          │                 │
│         ▼          ▼         ▼         ▼          ▼                 │
│    ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐          │
│    │LEDs    │ │Servo   │ │Sensors │ │LCD     │ │DC/     │          │
│    │Buttons │ │Buzzer  │ │Temp    │ │7-Seg   │ │Stepper │          │
│    │Matrix  │ │Timing  │ │Light   │ │OLED    │ │        │          │
│    └────────┘ └────────┘ └────────┘ └────────┘ └────────┘          │
│                                                                      │
│  SIGNAL TYPES:                                                       │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ Type        │ Examples              │ Interface              │   │
│  ├──────────────────────────────────────────────────────────────┤   │
│  │ Digital I/O │ LEDs, switches        │ GPIO                   │   │
│  │ Analog In   │ Temperature, light    │ ADC                    │   │
│  │ Analog Out  │ Audio, control        │ DAC, PWM               │   │
│  │ Timed       │ Servo, measurement    │ Timer/Counter          │   │
│  │ Serial      │ Sensors, displays     │ UART, SPI, I2C         │   │
│  │ Power       │ Motors, relays        │ Driver circuits        │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Chapter Overview

### Chapter 6.1: GPIO and Digital Interfacing
- GPIO modes and configurations
- LED interfacing (direct, current limiting)
- Switch/button interfacing (pull-up/down, debouncing)
- LED matrix and keypad scanning
- Relay and optocoupler interfacing

### Chapter 6.2: Timers and PWM
- Timer/counter fundamentals
- Timer modes (free-run, capture, compare)
- PWM generation and applications
- Input capture for measurement
- Servo motor control

### Chapter 6.3: ADC and DAC
- ADC architectures (SAR, sigma-delta)
- ADC configuration and sampling
- Sensor signal conditioning
- DAC fundamentals
- DMA with ADC/DAC

### Chapter 6.4: Display Interfacing
- 7-segment displays (multiplexing)
- Character LCD (HD44780)
- Graphic LCD basics
- OLED displays (SSD1306)
- Display drivers and buffers

### Chapter 6.5: Motor Control
- DC motor interfacing (H-bridge)
- Stepper motor basics
- Servo motor control
- Motor driver ICs
- Speed and position feedback

### Chapter 6.6: Sensor Interfacing
- Temperature sensors (NTC, digital)
- Light sensors (LDR, photodiode)
- Distance sensors (ultrasonic, IR)
- Inertial sensors (accelerometer, gyro)
- Environmental sensors

---

## Key Interfacing Concepts

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SIGNAL CONDITIONING                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Real-world signals often need conditioning before MCU input:      │
│                                                                      │
│  ANALOG SIGNALS:                                                     │
│                                                                      │
│  Sensor ──▶ Amplifier ──▶ Filter ──▶ Level Shift ──▶ ADC ──▶ MCU   │
│              │            │           │                              │
│              │ Gain       │ Remove    │ Scale to                    │
│              │ weak       │ noise     │ 0-3.3V                      │
│              │ signals    │           │                              │
│                                                                      │
│  DIGITAL SIGNALS:                                                    │
│                                                                      │
│  Switch ──▶ Pull-up/Down ──▶ Debounce ──▶ GPIO ──▶ MCU             │
│              │                │                                      │
│              │ Define         │ Remove                               │
│              │ voltage        │ bounce                               │
│                                                                      │
│  OUTPUT INTERFACING:                                                 │
│                                                                      │
│  MCU ──▶ GPIO ──▶ Buffer/Driver ──▶ Load                           │
│                     │                                                │
│                     │ Increase                                       │
│                     │ current                                        │
│                     │ capability                                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Common Interface Circuits

```
┌─────────────────────────────────────────────────────────────────────┐
│                    INTERFACE CIRCUIT BUILDING BLOCKS                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  LED (Active High):         LED (Active Low):                       │
│                                                                      │
│  GPIO ──┬──[R]──▶|──GND     VCC──▶|──[R]──┬── GPIO                 │
│         │ 330Ω   LED               LED    │                         │
│     HIGH = ON               HIGH = OFF                               │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  Switch with Pull-up:       Switch with Pull-down:                  │
│                                                                      │
│  VCC                        GPIO                                     │
│   │                          │                                       │
│  ─┴─ R(10kΩ)                ─┴─ R(10kΩ)                             │
│  ─┬─                        ─┬─                                      │
│   │                          │                                       │
│   ├─── GPIO                  ├─── VCC                               │
│   │                          │                                       │
│  ─┴─ SW                     ─┴─ SW                                  │
│   │                          │                                       │
│  GND                        GND                                      │
│                                                                      │
│  Released = HIGH            Released = LOW                          │
│  Pressed = LOW              Pressed = HIGH                          │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  High-Current Load (NPN):   High-Current Load (N-FET):             │
│                                                                      │
│  VCC ──▶ Load ──┬           VCC ──▶ Load ──┬                        │
│                 │                          │                         │
│  GPIO ─[R]─┬──|/            GPIO ─[R]─┬───┤├                        │
│            │   \│                     │   └┤                        │
│           GND   │                    GND   │                        │
│                GND                        GND                        │
│                                                                      │
│  GPIO HIGH = Load ON        GPIO HIGH = Load ON                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Hardware Abstraction

```
┌─────────────────────────────────────────────────────────────────────┐
│                    HAL ARCHITECTURE                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                     APPLICATION                              │    │
│  │  led_on(), read_temperature(), motor_set_speed()             │    │
│  └──────────────────────────────┬──────────────────────────────┘    │
│                                 │                                    │
│  ┌──────────────────────────────┴──────────────────────────────┐    │
│  │                    DEVICE DRIVERS                            │    │
│  │  led_driver, temp_sensor_driver, motor_driver                │    │
│  └──────────────────────────────┬──────────────────────────────┘    │
│                                 │                                    │
│  ┌──────────────────────────────┴──────────────────────────────┐    │
│  │          HARDWARE ABSTRACTION LAYER (HAL)                    │    │
│  │  gpio_write(), adc_read(), pwm_set_duty()                    │    │
│  └──────────────────────────────┬──────────────────────────────┘    │
│                                 │                                    │
│  ┌──────────────────────────────┴──────────────────────────────┐    │
│  │              LOW-LEVEL DRIVERS (CMSIS)                       │    │
│  │  Direct register access, interrupt handlers                  │    │
│  └──────────────────────────────┬──────────────────────────────┘    │
│                                 │                                    │
│  ┌──────────────────────────────┴──────────────────────────────┐    │
│  │                       HARDWARE                               │    │
│  │  GPIO, ADC, Timers, UART, SPI, I2C registers                │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Benefits:                                                           │
│  • Portability across MCU families                                  │
│  • Testability (mock hardware layer)                                │
│  • Clean separation of concerns                                     │
│  • Reusable driver code                                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Unit 5: Communication Protocols](../05-Communication-Protocols/README.md) | [Course Index](../README.md) | [Chapter 6.1: GPIO and Digital Interfacing](01-gpio-digital-interfacing.md) |

---
