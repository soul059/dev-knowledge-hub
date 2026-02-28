# Chapter 2.4: Sensors and Actuators

## Chapter Overview

Sensors and actuators are the interface between embedded systems and the physical world. Sensors convert physical phenomena into electrical signals, while actuators convert electrical signals into physical actions. This chapter covers the principles, types, and interfacing techniques for common sensors and actuators.

---

## 2.4.1 Introduction to Sensors

### Sensor Classification

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SENSOR CLASSIFICATION                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  BY PHYSICAL PHENOMENON:                                             │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                                                              │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │    │
│  │  │  Temp    │  │ Pressure │  │  Light   │  │ Position │    │    │
│  │  │  Sensors │  │  Sensors │  │  Sensors │  │  Sensors │    │    │
│  │  ├──────────┤  ├──────────┤  ├──────────┤  ├──────────┤    │    │
│  │  │Thermistor│  │Piezo     │  │LDR       │  │Encoder   │    │    │
│  │  │RTD       │  │Strain    │  │Photodiode│  │Potentiom │    │    │
│  │  │Thermocpl │  │Capacitive│  │Phototran │  │LVDT      │    │    │
│  │  │DS18B20   │  │MEMS      │  │PIR       │  │Hall      │    │    │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘    │    │
│  │                                                              │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │    │
│  │  │  Motion  │  │ Chemical │  │  Flow    │  │ Humidity │    │    │
│  │  │  Sensors │  │  Sensors │  │  Sensors │  │  Sensors │    │    │
│  │  ├──────────┤  ├──────────┤  ├──────────┤  ├──────────┤    │    │
│  │  │Accel     │  │Gas (MQ)  │  │Turbine   │  │Capacitive│    │    │
│  │  │Gyroscope │  │pH        │  │Ultrasonic│  │Resistive │    │    │
│  │  │IMU       │  │Oxygen    │  │Mass flow │  │DHT11/22  │    │    │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘    │    │
│  │                                                              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  BY OUTPUT TYPE:                                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Type        │ Output      │ Examples                       │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Analog      │ Voltage/Curr│ Thermistor, LDR, Potentiometer │    │
│  │ Digital     │ Logic levels│ Encoder, Hall switch, PIR      │    │
│  │ Serial      │ I2C/SPI/UART│ DS18B20, BMP280, MPU6050       │    │
│  │ PWM         │ Duty cycle  │ Some ultrasonic sensors        │    │
│  │ Frequency   │ Freq output │ Some flow sensors, encoders    │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Sensor Characteristics

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SENSOR CHARACTERISTICS                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  TRANSFER FUNCTION:                                                  │
│                                                                      │
│  Output                                                              │
│    ▲                                                                 │
│    │                    ╱╱╱╱  Saturation                            │
│    │               ╱╱╱╱╱                                            │
│    │          ╱╱╱╱╱                                                 │
│    │     ╱╱╱╱╱        Linear range                                  │
│    │╱╱╱╱╱                                                           │
│    │                                                                 │
│    └────────────────────────────────────────────►  Input            │
│                                                    (Measurand)      │
│                                                                      │
│  KEY PARAMETERS:                                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Parameter       │ Definition                                │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Sensitivity     │ Output change per unit input change       │    │
│  │                 │ S = ΔOutput / ΔInput                       │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Range           │ Min to max measurable input values        │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Resolution      │ Smallest detectable input change          │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Accuracy        │ Closeness to true value                   │    │
│  │                 │ Error = |Measured - True|                  │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Precision       │ Repeatability of measurements             │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Linearity       │ Deviation from straight line response     │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Response Time   │ Time to reach 63.2% of final value        │    │
│  │                 │ (Time constant τ)                          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ACCURACY vs PRECISION:                                              │
│                                                                      │
│   High Accuracy     Low Accuracy      Low Accuracy     High Accuracy │
│   High Precision    High Precision    Low Precision    Low Precision│
│                                                                      │
│     ┌─────┐          ┌─────┐          ┌─────┐          ┌─────┐     │
│     │● ● ●│          │     │          │  ●  │          │     │     │
│     │● ⊕ ●│          │●●●  │          │●    │          │ ⊕   │     │
│     │● ● ●│          │●●●⊕ │          │ ⊕ ● │          │● ●● │     │
│     └─────┘          └─────┘          └─────┘          └─────┘     │
│     (Ideal)         (Systematic      (Random          (Worst case) │
│                       error)          error)                        │
│                     ⊕ = True value   ● = Measurements              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2.4.2 Common Sensor Types

### Temperature Sensors

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TEMPERATURE SENSORS                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  THERMISTOR (NTC):                                                   │
│                                                                      │
│  Resistance                                                          │
│    ▲                                                                 │
│    │\                                                                │
│    │ \                                                               │
│    │  \                                                              │
│    │   \                                                             │
│    │    \___                                                         │
│    │        \___                                                     │
│    │            \___                                                 │
│    └────────────────────────────────►  Temperature                  │
│                                                                      │
│  STEINHART-HART EQUATION:                                            │
│  1/T = A + B×ln(R) + C×(ln(R))³                                     │
│                                                                      │
│  SIMPLIFIED (β PARAMETER):                                          │
│  1/T = 1/T₀ + (1/β)×ln(R/R₀)                                       │
│                                                                      │
│  VOLTAGE DIVIDER CIRCUIT:                                            │
│                                                                      │
│      VCC (3.3V)                                                      │
│         │                                                            │
│    ┌────┴────┐                                                      │
│    │  R_fixed│ (10kΩ)                                               │
│    │         │                                                      │
│    └────┬────┘                                                      │
│         ├────────────►  V_out (to ADC)                              │
│    ┌────┴────┐                                                      │
│    │  NTC    │ (10kΩ @ 25°C)                                        │
│    │Thermistor                                                       │
│    └────┬────┘                                                      │
│         │                                                            │
│        GND                                                           │
│                                                                      │
│  V_out = VCC × R_NTC / (R_fixed + R_NTC)                            │
│                                                                      │
│  TEMPERATURE SENSOR COMPARISON:                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Type        │ Range       │ Accuracy  │ Interface │ Cost   │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Thermistor  │ -40 to 150°C│ ±0.5-1°C  │ Analog    │ Low    │    │
│  │ RTD (PT100) │ -200 to 850°│ ±0.1°C    │ Analog    │ High   │    │
│  │ Thermocouple│ -200 to 2000│ ±1-2°C    │ Analog    │ Medium │    │
│  │ LM35        │ -55 to 150°C│ ±0.5°C    │ Analog    │ Low    │    │
│  │ DS18B20     │ -55 to 125°C│ ±0.5°C    │ 1-Wire    │ Low    │    │
│  │ BMP280      │ -40 to 85°C │ ±1°C      │ I2C/SPI   │ Low    │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Motion Sensors (Accelerometer/Gyroscope)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MOTION SENSORS (IMU)                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ACCELEROMETER PRINCIPLE (MEMS):                                     │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              Fixed electrode                                │    │
│  │  ═══════════════════════════════════════════                │    │
│  │              ↑                                              │    │
│  │        Capacitance C1                                       │    │
│  │              ↓                                              │    │
│  │        ┌─────────────┐                                      │    │
│  │  ◄────►│  Proof Mass │◄────►   (Suspended by springs)      │    │
│  │ Spring │             │ Spring                               │    │
│  │        └─────────────┘                                      │    │
│  │              ↑                                              │    │
│  │        Capacitance C2                                       │    │
│  │              ↓                                              │    │
│  │  ═══════════════════════════════════════════                │    │
│  │              Fixed electrode                                │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Acceleration → Mass displacement → Capacitance change → Voltage   │
│                                                                      │
│  GYROSCOPE PRINCIPLE (MEMS - Coriolis Effect):                      │
│                                                                      │
│         Vibrating Mass                                               │
│              ▲                                                       │
│         ◄────┼────►  Driven vibration                               │
│              │                                                       │
│              │       When rotating:                                  │
│              │       Coriolis force causes                          │
│              ▼       perpendicular displacement                      │
│         ──────────                                                   │
│         Detection axis                                               │
│                                                                      │
│  F_coriolis = 2 × m × v × ω                                         │
│                                                                      │
│  6-AXIS IMU (MPU6050):                                               │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                                                              │    │
│  │        Z (Yaw)                                              │    │
│  │          ▲                                                  │    │
│  │          │    Y (Pitch)                                     │    │
│  │          │   ╱                                              │    │
│  │          │  ╱                                               │    │
│  │          │ ╱                                                │    │
│  │          │╱───────────►  X (Roll)                           │    │
│  │     ┌────────────┐                                          │    │
│  │     │            │  • 3-axis accelerometer (±2g to ±16g)   │    │
│  │     │  MPU6050   │  • 3-axis gyroscope (±250 to ±2000°/s)  │    │
│  │     │            │  • I2C interface (up to 400 kHz)        │    │
│  │     └────────────┘  • 16-bit resolution                    │    │
│  │                                                              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Proximity and Distance Sensors

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PROXIMITY AND DISTANCE SENSORS                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ULTRASONIC SENSOR (HC-SR04):                                        │
│                                                                      │
│      ┌───────────────────────────────────┐                          │
│      │        HC-SR04 Module             │                          │
│      │  ┌─────────┐   ┌─────────┐       │                          │
│      │  │ Speaker │   │  Mic    │       │                          │
│      │  │   TX    │   │   RX    │       │                          │
│      │  └────┬────┘   └────┬────┘       │                          │
│      └───────┼─────────────┼────────────┘                          │
│              │             │                                        │
│              │  )))))))))) │  (((((((((                             │
│              │             │       ◄──── Object                     │
│              │             │       │                                │
│              │◄────────────┼───────┘                                │
│                            │                                        │
│                        Reflected                                    │
│                          wave                                       │
│                                                                      │
│  TIMING DIAGRAM:                                                     │
│                                                                      │
│  Trigger ──┐    ┌──                                                 │
│            └────┘  (10µs pulse)                                     │
│                                                                      │
│  Echo     ─────────┐              ┌─────────                        │
│                    └──────────────┘                                  │
│                    │◄────────────►│                                  │
│                         t_echo                                       │
│                                                                      │
│  Distance = (t_echo × Speed_of_sound) / 2                           │
│  Distance = (t_echo × 343 m/s) / 2                                  │
│  Distance (cm) = t_echo (µs) / 58                                   │
│                                                                      │
│  IR PROXIMITY SENSOR:                                                │
│                                                                      │
│      ┌─────────────────────────────────────┐                        │
│      │   ┌─────┐             ┌─────┐      │                        │
│      │   │ LED │   )))       │Photo│      │                        │
│      │   │ IR  │   )))  ◄──  │Trans│      │                        │
│      │   │     │   )))  │    │     │      │                        │
│      │   └──┬──┘        │    └──┬──┘      │                        │
│      └──────┼───────────┼───────┼─────────┘                        │
│             │           │       │                                   │
│           Emit       Reflect  Detect                                │
│                                                                      │
│  SENSOR COMPARISON:                                                  │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Type        │ Range     │ Accuracy │ Speed    │ Cost       │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Ultrasonic  │ 2-400 cm  │ ±3 mm    │ ~60 ms   │ Very Low   │    │
│  │ IR Sharp    │ 10-80 cm  │ ±5%      │ ~40 ms   │ Low        │    │
│  │ ToF (VL53L0)│ 0-200 cm  │ ±3%      │ <30 ms   │ Medium     │    │
│  │ LIDAR      │ 0.5-40 m  │ ±2 cm    │ <1 ms    │ High       │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2.4.3 Actuators

### Actuator Classification

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ACTUATOR CLASSIFICATION                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                      ACTUATORS                              │    │
│  └────────────────────────┬────────────────────────────────────┘    │
│                           │                                         │
│     ┌─────────────┬───────┴───────┬─────────────┐                  │
│     │             │               │             │                   │
│  ┌──┴───┐     ┌───┴───┐      ┌────┴────┐   ┌───┴────┐             │
│  │Motors│     │Solenoids│     │ Displays│   │ Others │             │
│  └──┬───┘     └───┬───┘      └────┬────┘   └───┬────┘             │
│     │             │               │             │                   │
│  ┌──┴──────┐  ┌───┴───┐     ┌────┴────┐   ┌───┴────┐             │
│  │DC Motor │  │Linear │     │LED/OLED │   │Heaters │             │
│  │Stepper  │  │Rotary │     │LCD      │   │Speakers│             │
│  │Servo    │  │Valve  │     │7-Segment│   │Relays  │             │
│  │BLDC     │  │Relay  │     │E-Paper  │   │Piezo   │             │
│  └─────────┘  └───────┘     └─────────┘   └────────┘             │
│                                                                      │
│  ACTUATION PRINCIPLES:                                               │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Type           │ Principle          │ Control Method        │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ DC Motor       │ Electromagnetic    │ PWM voltage           │    │
│  │ Stepper Motor  │ Electromagnetic    │ Step pulses           │    │
│  │ Servo Motor    │ Closed-loop DC     │ PWM position signal   │    │
│  │ Solenoid       │ Electromagnetic    │ On/Off voltage        │    │
│  │ Piezoelectric  │ Piezo effect       │ High voltage AC       │    │
│  │ Thermal        │ Thermal expansion  │ Current/Power         │    │
│  │ Pneumatic      │ Gas pressure       │ Valve control         │    │
│  │ Hydraulic      │ Fluid pressure     │ Valve control         │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### DC Motor Control

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DC MOTOR CONTROL                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  H-BRIDGE CIRCUIT:                                                   │
│                                                                      │
│              VCC                                                     │
│               │                                                      │
│       ┌───────┼───────┐                                             │
│       │       │       │                                             │
│     ┌─┴─┐   ┌─┴─┐   ┌─┴─┐   ┌───┐                                 │
│     │Q1 │   │   │   │Q2 │   │   │                                 │
│     │PNP│   │ M │   │PNP│   │ M │                                 │
│  A──┤   ├───┤   ├───┤   ├───┤   │                                 │
│     └─┬─┘   │ O │   └─┬─┘   │ O │                                 │
│       │     │ T │     │     │ T │                                 │
│       │     │ O │     │     │ O │                                 │
│     ┌─┴─┐   │ R │   ┌─┴─┐   │ R │                                 │
│     │Q3 │   │   │   │Q4 │   │   │                                 │
│     │NPN│   │   │   │NPN│   │   │                                 │
│  B──┤   ├───┤   ├───┤   ├───┘   │                                 │
│     └─┬─┘   └───┘   └─┬─┘                                          │
│       │               │                                             │
│       └───────┬───────┘                                             │
│               │                                                      │
│              GND                                                     │
│                                                                      │
│  TRUTH TABLE:                                                        │
│  ┌───────────────────────────────────────────────────────────┐      │
│  │  A  │  B  │ Q1/Q4 │ Q2/Q3 │    Motor Action              │      │
│  │─────────────────────────────────────────────────────────── │     │
│  │  0  │  0  │  OFF  │  OFF  │    Coast (Free running)      │      │
│  │  1  │  0  │  ON   │  OFF  │    Forward rotation          │      │
│  │  0  │  1  │  OFF  │  ON   │    Reverse rotation          │      │
│  │  1  │  1  │  ON   │  ON   │    Brake (Short circuit)     │      │
│  └───────────────────────────────────────────────────────────┘      │
│                                                                      │
│  PWM SPEED CONTROL:                                                  │
│                                                                      │
│  PWM    ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐                               │
│  Signal │  │  │  │  │  │  │  │  │  │                               │
│         ┘  └──┘  └──┘  └──┘  └──┘  └──                              │
│                                                                      │
│         25% duty          75% duty                                   │
│         Low speed         High speed                                 │
│                                                                      │
│  Average Voltage = Duty Cycle × VCC                                 │
│  V_avg = 0.75 × 12V = 9V                                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Servo Motor Control

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SERVO MOTOR CONTROL                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SERVO INTERNAL STRUCTURE:                                           │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                         SERVO MOTOR                          │   │
│  │                                                              │   │
│  │   ┌─────────────────────────────────────────────────────┐   │   │
│  │   │  PWM ──► Control ──► Motor ──► Gearbox ──► Shaft   │   │   │
│  │   │  Input   Circuit     Driver              │         │   │   │
│  │   │                                          │         │   │   │
│  │   │   Position Feedback ◄── Potentiometer ◄──┘         │   │   │
│  │   └─────────────────────────────────────────────────────┘   │   │
│  │                                                              │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  PWM CONTROL SIGNAL (50 Hz / 20 ms period):                         │
│                                                                      │
│  0° Position:                                                        │
│  ─┐ ┌─────────────────────────────────────────────────────────      │
│   └─┘ (1.0 ms pulse)                                                │
│   │◄►│                                                              │
│   1ms     20ms total                                                 │
│                                                                      │
│  90° Position:                                                       │
│  ─┐   ┌───────────────────────────────────────────────────────      │
│   └───┘ (1.5 ms pulse)                                              │
│   │◄─►│                                                             │
│   1.5ms                                                              │
│                                                                      │
│  180° Position:                                                      │
│  ─┐     ┌─────────────────────────────────────────────────────      │
│   └─────┘ (2.0 ms pulse)                                            │
│   │◄───►│                                                           │
│    2ms                                                               │
│                                                                      │
│  POSITION CALCULATION:                                               │
│                                                                      │
│  Pulse Width (ms) = 1.0 + (Angle / 180) × 1.0                       │
│  Angle (degrees) = (Pulse Width - 1.0) × 180                        │
│                                                                      │
│  SERVO SPECIFICATIONS:                                               │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Parameter        │ SG90 (Micro) │ MG996R (Standard)        │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Torque           │ 1.8 kg-cm    │ 10 kg-cm                 │    │
│  │ Speed            │ 0.1 s/60°    │ 0.2 s/60°                │    │
│  │ Voltage          │ 4.8-6V       │ 4.8-7.2V                 │    │
│  │ Current (stall)  │ 650 mA       │ 2.5 A                    │    │
│  │ Rotation         │ 0-180°       │ 0-180°                   │    │
│  │ Weight           │ 9g           │ 55g                      │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Stepper Motor Control

```
┌─────────────────────────────────────────────────────────────────────┐
│                    STEPPER MOTOR CONTROL                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  STEPPER MOTOR TYPES:                                                │
│                                                                      │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐ │
│  │   UNIPOLAR      │    │    BIPOLAR      │    │    HYBRID       │ │
│  │                 │    │                 │    │                 │ │
│  │   ┌───┬───┐     │    │   ┌───────┐     │    │  ┌───────┐     │ │
│  │   │A1 │A2 │     │    │   │ A     │     │    │  │  ┌─┐  │     │ │
│  │   └─┬─┴─┬─┘     │    │   └───────┘     │    │  │  │N│  │     │ │
│  │     │ C │       │    │   ┌───────┐     │    │  │  │S│  │     │ │
│  │   ┌─┴─┬─┴─┐     │    │   │ B     │     │    │  │  └─┘  │     │ │
│  │   │B1 │B2 │     │    │   └───────┘     │    │  └───────┘     │ │
│  │   └───┴───┘     │    │                 │    │   Toothed       │ │
│  │   6 wires       │    │   4 wires       │    │   rotor         │ │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘ │
│                                                                      │
│  FULL STEP SEQUENCE (Bipolar):                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Step │ A+ │ A- │ B+ │ B- │ Coil State                      │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │  1   │ 1  │ 0  │ 1  │ 0  │ A positive, B positive         │    │
│  │  2   │ 0  │ 1  │ 1  │ 0  │ A negative, B positive         │    │
│  │  3   │ 0  │ 1  │ 0  │ 1  │ A negative, B negative         │    │
│  │  4   │ 1  │ 0  │ 0  │ 1  │ A positive, B negative         │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  MICROSTEPPING CONCEPT:                                              │
│                                                                      │
│  Full Step (1.8°)        Half Step (0.9°)      Microstep (0.225°)  │
│                                                                      │
│  Current A    ▲          Current A    ▲          Current A    ▲    │
│               │                       │                       │    │
│          ─────┼─────             ─────┼─────             ─────┼─────│
│               │                       │                       │    │
│               ├──────►            ┌───┼───┐             ∿∿∿∿∿│     │
│               │       Current B   │   │   │►    sine     │     │    │
│                                   └───┴───┘     wave     ▼     │    │
│                                                                      │
│  Step Angle = 360° / (Steps per Revolution × Microstep Divisor)    │
│  Example: 200 steps × 16 microsteps = 3200 steps/rev = 0.1125°     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2.4.4 Signal Conditioning

### Analog Signal Conditioning

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SIGNAL CONDITIONING                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SIGNAL CONDITIONING CHAIN:                                          │
│                                                                      │
│  ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐   │
│  │ Sensor │──►│ Amplif │──►│ Filter │──►│ Level  │──►│  ADC   │   │
│  │        │   │        │   │        │   │ Shift  │   │        │   │
│  └────────┘   └────────┘   └────────┘   └────────┘   └────────┘   │
│                                                                      │
│  AMPLIFICATION (Op-Amp Non-Inverting):                               │
│                                                                      │
│              ┌───────────┐                                          │
│              │     ┌─────┤                                          │
│              │    Rf     │                                          │
│              │     └──┬──┤                                          │
│              │        │  │                                          │
│  V_in ───────┼────┬───┴──┼───►  V_out                              │
│              │  ┌─┴─┐    │                                          │
│              │  │-  │    │                                          │
│              │  │Op │────┘                                          │
│              │  │Amp│                                               │
│              │  │+  │                                               │
│              │  └─┬─┘                                               │
│              │    │                                                 │
│              │   Rg                                                 │
│              │    │                                                 │
│              └────┴───── GND                                        │
│                                                                      │
│  Gain = 1 + (Rf / Rg)                                               │
│                                                                      │
│  LOW-PASS FILTER (RC):                                               │
│                                                                      │
│  V_in ───┬─────┬───────► V_out                                      │
│          │     │                                                     │
│          R     C                                                     │
│          │     │                                                     │
│          └──┬──┘                                                     │
│             │                                                        │
│            GND                                                       │
│                                                                      │
│  Cutoff Frequency: fc = 1 / (2π × R × C)                            │
│                                                                      │
│  LEVEL SHIFTING (0-5V to 0-3.3V):                                    │
│                                                                      │
│  V_in (0-5V) ───┬─────┬───► V_out (0-3.3V)                          │
│                 │     │                                              │
│                R1    R2                                              │
│               10kΩ   20kΩ                                           │
│                 │     │                                              │
│                 └──┬──┘                                              │
│                    │                                                 │
│                   GND                                                │
│                                                                      │
│  V_out = V_in × R2 / (R1 + R2) = 5V × 20k / 30k = 3.33V            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Component | Type | Interface | Typical Use |
|-----------|------|-----------|-------------|
| **NTC Thermistor** | Temperature sensor | Analog | HVAC, appliances |
| **DS18B20** | Temperature sensor | 1-Wire | Industrial, outdoor |
| **MPU6050** | IMU (Accel+Gyro) | I2C | Drones, robotics |
| **HC-SR04** | Ultrasonic distance | GPIO | Obstacle detection |
| **LDR** | Light sensor | Analog | Ambient light sensing |
| **DC Motor** | Rotary actuator | PWM + H-Bridge | Fans, wheels |
| **Servo Motor** | Rotary actuator | PWM (50Hz) | Arms, pan/tilt |
| **Stepper Motor** | Rotary actuator | Step/Direction | CNC, 3D printers |

---

## Quick Revision Questions

1. **What are the key differences between accuracy and precision in sensor measurements?**

2. **Calculate the temperature from an NTC thermistor using the β parameter equation given R = 8kΩ at unknown temperature, R₀ = 10kΩ at T₀ = 25°C, and β = 3950.**

3. **How does a MEMS accelerometer measure acceleration using the capacitive principle?**

4. **What pulse width is needed to position a servo motor at 45 degrees?**

5. **Explain the difference between full-step and microstepping in stepper motor control.**

6. **Design a voltage divider to convert a 0-5V sensor output to 0-3.3V for an ADC input.**

---

## Navigation

| Previous | Up | Next |
|----------|------|------|
| [I/O Devices](03-io-devices.md) | [Unit 2 Index](README.md) | [Power Supply](05-power-supply.md) |

---
