# Chapter 1.4: Applications and Examples of Embedded Systems

## Chapter Overview

This chapter explores the vast landscape of embedded system applications across various industries. We examine real-world examples that demonstrate how embedded systems have become integral to modern life, from household appliances to critical infrastructure, transportation, healthcare, and beyond.

---

## 1.4.1 Embedded Systems in Daily Life

### The Hidden Computing Revolution

```
┌─────────────────────────────────────────────────────────────────────┐
│                EMBEDDED SYSTEMS IN YOUR DAILY LIFE                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│    MORNING ROUTINE:                                                  │
│    ┌─────────────────────────────────────────────────────────────┐  │
│    │ 06:00 - Alarm clock ━━━━━━━━━━━━━━━━━━━━━━━━━━► [MCU]       │  │
│    │ 06:05 - Coffee maker ━━━━━━━━━━━━━━━━━━━━━━━━━► [MCU]       │  │
│    │ 06:15 - Smart thermostat adjusts ━━━━━━━━━━━━━► [MCU+WiFi] │  │
│    │ 06:20 - Microwave heats breakfast ━━━━━━━━━━━━► [MCU]       │  │
│    │ 06:30 - Fitness tracker syncs ━━━━━━━━━━━━━━━━► [MCU+BT]   │  │
│    │ 06:45 - Car engine starts ━━━━━━━━━━━━━━━━━━━━► [50+ ECUs] │  │
│    │ 07:00 - Traffic lights sequence ━━━━━━━━━━━━━━► [MCU]       │  │
│    │ 07:15 - Office elevator ━━━━━━━━━━━━━━━━━━━━━━► [MCU]       │  │
│    │ 07:20 - Badge access control ━━━━━━━━━━━━━━━━━► [MCU+RFID] │  │
│    └─────────────────────────────────────────────────────────────┘  │
│                                                                      │
│    By the time you reach work, you've interacted with 50+           │
│    embedded systems!                                                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Embedded Systems Count in Modern Life

| Location/Activity | Approximate Embedded Systems Count |
|-------------------|-----------------------------------|
| Modern home | 50-100 |
| Modern car | 70-150 |
| Office building | 500-1000 |
| Hospital | 10,000+ |
| Commercial aircraft | 100+ critical systems |
| Smartphone | 10+ embedded subsystems |

---

## 1.4.2 Consumer Electronics

### Home Appliances

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SMART WASHING MACHINE                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│    ┌─────────────────────────────────────────────────────────────┐  │
│    │                    CONTROL PANEL                             │  │
│    │   [START] [PAUSE] [PROGRAM] [TEMP] [SPIN] [DELAY]           │  │
│    │                    ┌──────────┐                              │  │
│    │                    │  LCD     │                              │  │
│    │                    │ Display  │                              │  │
│    │                    └──────────┘                              │  │
│    └─────────────────────────────────────────────────────────────┘  │
│                              │                                       │
│    ┌─────────────────────────┴─────────────────────────┐            │
│    │                MAIN CONTROLLER                     │            │
│    │    ┌─────────────────────────────────────────┐    │            │
│    │    │         MICROCONTROLLER                  │    │            │
│    │    │   • Program storage (wash cycles)        │    │            │
│    │    │   • Timing control                       │    │            │
│    │    │   • Sensor processing                    │    │            │
│    │    │   • Motor control                        │    │            │
│    │    │   • User interface                       │    │            │
│    │    └─────────────────────────────────────────┘    │            │
│    └───────────────────────────────────────────────────┘            │
│                              │                                       │
│         ┌────────────────────┼────────────────────┐                 │
│         │                    │                    │                 │
│         ▼                    ▼                    ▼                 │
│    ┌─────────┐         ┌─────────┐         ┌─────────┐             │
│    │ SENSORS │         │ MOTORS  │         │ VALVES  │             │
│    │         │         │         │         │         │             │
│    │•Water   │         │•Drum    │         │•Water   │             │
│    │ level   │         │ motor   │         │ inlet   │             │
│    │•Temp    │         │•Drain   │         │•Detergent│            │
│    │•Door    │         │ pump    │         │ dispenser│            │
│    │•Vibration│        │         │         │         │             │
│    └─────────┘         └─────────┘         └─────────┘             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Entertainment Devices

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SMART TV ARCHITECTURE                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│    ┌──────────────────────────────────────────────────────────┐     │
│    │                     MAIN SoC                              │     │
│    │  ┌─────────────────────────────────────────────────────┐ │     │
│    │  │ CPU    │ GPU     │ Video   │ Audio   │ Security   │ │     │
│    │  │(Quad)  │         │ Decoder │ DSP     │ Module     │ │     │
│    │  └─────────────────────────────────────────────────────┘ │     │
│    │  ┌─────────────────────────────────────────────────────┐ │     │
│    │  │ DDR4 RAM (2-4GB)  │  eMMC Storage (8-64GB)        │ │     │
│    │  └─────────────────────────────────────────────────────┘ │     │
│    └──────────────────────────────────────────────────────────┘     │
│                              │                                       │
│    ┌─────────────────────────┴─────────────────────────────┐        │
│    │                   PERIPHERALS                          │        │
│    │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐       │        │
│    │  │WiFi  │ │ BT   │ │HDMI  │ │ USB  │ │Tuner │       │        │
│    │  │Module│ │Module│ │ x4   │ │ x2   │ │DVB-T │       │        │
│    │  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘       │        │
│    │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────────────────┐    │        │
│    │  │ IR   │ │Audio │ │Display│ │  Smart Features  │    │        │
│    │  │Recvr │ │Output│ │Panel │ │ Voice, Streaming │    │        │
│    │  └──────┘ └──────┘ └──────┘ └──────────────────┘    │        │
│    └─────────────────────────────────────────────────────────┘      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Consumer Electronics Examples Table

| Device | Processor Type | Key Features | Typical OS |
|--------|---------------|--------------|------------|
| Smart TV | ARM SoC | Video decode, streaming | Android TV, webOS |
| Game Console | Custom SoC | High GPU, connectivity | Custom OS |
| Digital Camera | ARM + DSP | Image processing | Proprietary RTOS |
| Smart Speaker | ARM Cortex | Voice AI, WiFi/BT | Android Things |
| Wireless Earbuds | ARM Cortex-M | BT audio, ANC | Bare-metal/RTOS |
| E-Reader | ARM + E-ink | Low power display | Linux/Android |

---

## 1.4.3 Automotive Applications

### Modern Vehicle Electronics

```
┌─────────────────────────────────────────────────────────────────────┐
│                    AUTOMOTIVE EMBEDDED SYSTEMS                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                         ┌─────────────┐                             │
│                         │   VEHICLE   │                             │
│                         │   NETWORK   │                             │
│                         │    (CAN)    │                             │
│                         └──────┬──────┘                             │
│                                │                                     │
│    ┌───────────────────────────┼───────────────────────────┐        │
│    │           │               │               │           │        │
│    ▼           ▼               ▼               ▼           ▼        │
│ ┌──────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────┐      │
│ │ENGINE│  │  BODY    │  │  SAFETY  │  │INFOTAIN- │  │ ADAS │      │
│ │CONTROL│ │ CONTROL  │  │ SYSTEMS  │  │  MENT    │  │      │      │
│ └──┬───┘  └────┬─────┘  └────┬─────┘  └────┬─────┘  └──┬───┘      │
│    │           │             │             │           │           │
│ •Engine     •Lighting     •ABS          •Audio      •Cameras      │
│  timing     •Wipers       •Airbags      •Navigation •Radar        │
│ •Fuel       •Windows      •ESP          •Bluetooth  •LIDAR        │
│  injection  •HVAC         •Seatbelts    •Display    •Parking      │
│ •Trans-     •Central      •Tire         •Voice      •Lane         │
│  mission     locking       pressure      assistant   assist       │
│                                                                      │
│  ECU Count in Modern Cars: 70-150 units                            │
│  Code Lines: 100+ million                                           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Engine Control Unit (ECU) Block Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                       ENGINE CONTROL UNIT                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  INPUTS:                          OUTPUTS:                          │
│  ┌────────────────────┐          ┌────────────────────┐            │
│  │ • Throttle position│          │ • Fuel injectors   │            │
│  │ • Engine speed     │          │ • Ignition timing  │            │
│  │ • Coolant temp     │          │ • Idle air control │            │
│  │ • Air flow         │          │ • EGR valve        │            │
│  │ • Oxygen sensors   │ ───┐ ┌───│ • Turbo wastegate  │            │
│  │ • Knock sensor     │    │ │   │ • Cooling fan      │            │
│  │ • Cam/crank position│   │ │   │ • Warning lights   │            │
│  │ • Barometric pressure│  │ │   │ • CAN messages     │            │
│  └────────────────────┘    │ │   └────────────────────┘            │
│                            │ │                                       │
│                            ▼ ▼                                       │
│              ┌─────────────────────────────┐                        │
│              │     32-bit MCU (e.g.,       │                        │
│              │     ARM Cortex-R/M or       │                        │
│              │     Infineon TriCore)       │                        │
│              │                             │                        │
│              │ • Real-time calculations    │                        │
│              │ • Fuel maps lookup          │                        │
│              │ • PID control loops         │                        │
│              │ • Diagnostics (OBD-II)      │                        │
│              │ • Safety monitoring         │                        │
│              └─────────────────────────────┘                        │
│                                                                      │
│  TIMING CONSTRAINTS: Ignition timing calculated in < 50µs          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1.4.4 Medical Devices

### Life-Critical Medical Systems

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CARDIAC PACEMAKER SYSTEM                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│    IMPLANTED DEVICE:                                                 │
│    ┌─────────────────────────────────────────────────────────────┐  │
│    │  ┌─────────────────────────────────────────────────────┐   │  │
│    │  │              PACEMAKER UNIT                          │   │  │
│    │  │  ┌────────────┬────────────┬────────────────────┐   │   │  │
│    │  │  │ Sensing    │ Pulse      │ Communication      │   │   │  │
│    │  │  │ Circuits   │ Generator  │ Module (RF)        │   │   │  │
│    │  │  └────────────┴────────────┴────────────────────┘   │   │  │
│    │  │  ┌────────────┬────────────┬────────────────────┐   │   │  │
│    │  │  │ Ultra-Low  │ Program    │ Battery            │   │   │  │
│    │  │  │ Power MCU  │ Memory     │ (10+ years)        │   │   │  │
│    │  │  └────────────┴────────────┴────────────────────┘   │   │  │
│    │  └─────────────────────────────────────────────────────┘   │  │
│    │                         │                                   │  │
│    │              ┌──────────┴──────────┐                       │  │
│    │              │                     │                       │  │
│    │        ┌─────┴─────┐         ┌─────┴─────┐                │  │
│    │        │   Lead    │         │   Lead    │                │  │
│    │        │ (Atrium)  │         │(Ventricle)│                │  │
│    │        └───────────┘         └───────────┘                │  │
│    └─────────────────────────────────────────────────────────────┘  │
│                              │                                       │
│    REQUIREMENTS:              │  EXTERNAL PROGRAMMER:               │
│    • 99.9999% reliability    │  ┌──────────────────────┐           │
│    • 10+ year battery life   │  │ • Device programming │           │
│    • Hermetically sealed     │  │ • Data download      │           │
│    • MRI compatibility       │  │ • Diagnostics        │           │
│    • Sub-µW power modes      │  └──────────────────────┘           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Medical Device Categories

| Category | Examples | Safety Level | Typical MCU |
|----------|----------|--------------|-------------|
| **Life Support** | Pacemaker, Insulin pump | Critical | Ultra-low power ARM |
| **Diagnostic** | Blood glucose, ECG | High | ARM Cortex-M |
| **Imaging** | MRI, CT, Ultrasound | High | Multi-core SoC |
| **Monitoring** | Pulse oximeter, BP monitor | Medium | Low-power MCU |
| **Therapeutic** | Infusion pump, Dialysis | High | Safety-certified MCU |

---

## 1.4.5 Industrial Automation

### Programmable Logic Controller (PLC)

```
┌─────────────────────────────────────────────────────────────────────┐
│                       PLC ARCHITECTURE                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│    ┌────────────────────────────────────────────────────────────┐   │
│    │                      PLC RACK                               │   │
│    │  ┌─────────┬─────────┬─────────┬─────────┬─────────┐      │   │
│    │  │  POWER  │   CPU   │  INPUT  │ OUTPUT  │  COMM   │      │   │
│    │  │ SUPPLY  │ MODULE  │ MODULE  │ MODULE  │ MODULE  │      │   │
│    │  │         │         │         │         │         │      │   │
│    │  │ •24VDC  │•Processor│•Digital │•Digital │•Ethernet│      │   │
│    │  │ •AC/DC  │•Program │•Analog  │•Analog  │•Modbus  │      │   │
│    │  │  Conv.  │ Memory  │ inputs  │ outputs │•Profibus│      │   │
│    │  │         │•Timer   │         │         │         │      │   │
│    │  │         │•Counter │         │         │         │      │   │
│    │  └─────────┴─────────┴─────────┴─────────┴─────────┘      │   │
│    └────────────────────────────────────────────────────────────┘   │
│                              │                                       │
│    ┌─────────────────────────┴─────────────────────────────┐        │
│    │                     FIELD DEVICES                      │        │
│    │                                                        │        │
│    │  INPUTS:                    OUTPUTS:                   │        │
│    │  ┌────────────────────┐    ┌────────────────────┐     │        │
│    │  │ • Push buttons     │    │ • Motors           │     │        │
│    │  │ • Limit switches   │    │ • Solenoid valves  │     │        │
│    │  │ • Proximity sensors│    │ • Indicator lights │     │        │
│    │  │ • Temperature      │    │ • Variable speed   │     │        │
│    │  │ • Pressure         │    │   drives           │     │        │
│    │  │ • Flow meters      │    │ • Alarms           │     │        │
│    │  └────────────────────┘    └────────────────────┘     │        │
│    └────────────────────────────────────────────────────────┘        │
│                                                                      │
│  SCAN CYCLE: Input Scan → Program Execution → Output Update         │
│  TYPICAL CYCLE TIME: 1-50 ms                                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Industrial Robot Controller

```
┌─────────────────────────────────────────────────────────────────────┐
│                   INDUSTRIAL ROBOT SYSTEM                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│    ┌──────────────────────────────────────────────────────┐         │
│    │                  ROBOT CONTROLLER                     │         │
│    │  ┌────────────────────────────────────────────────┐  │         │
│    │  │ Main CPU │ Motion CPU │ I/O CPU │ Safety CPU  │  │         │
│    │  └────────────────────────────────────────────────┘  │         │
│    │                          │                           │         │
│    │  ┌────────────────────────────────────────────────┐  │         │
│    │  │ Path Planning │ Kinematics │ Servo Control    │  │         │
│    │  └────────────────────────────────────────────────┘  │         │
│    └────────────────────────────┬─────────────────────────┘         │
│                                 │                                    │
│                        ┌────────┴────────┐                          │
│                        │  SERVO DRIVES   │                          │
│                        │ (6-axis control)│                          │
│                        └────────┬────────┘                          │
│                                 │                                    │
│    ┌────────────────────────────┴────────────────────────┐          │
│    │                     ROBOT ARM                        │          │
│    │  ┌────────┬────────┬────────┬────────┬────────┐    │          │
│    │  │ Joint 1│ Joint 2│ Joint 3│ Joint 4│ Joint 5│    │          │
│    │  │(Base)  │(Shoulder)(Elbow) │(Wrist) │(Wrist) │    │          │
│    │  └────────┴────────┴────────┴────────┴────────┘    │          │
│    │  Each joint: Servo motor + Encoder + Brake          │          │
│    │                                                      │          │
│    │  ┌────────────────────────────────────────────┐    │          │
│    │  │              END EFFECTOR                   │    │          │
│    │  │  Gripper / Welding torch / Spray gun       │    │          │
│    │  └────────────────────────────────────────────┘    │          │
│    └──────────────────────────────────────────────────────┘          │
│                                                                      │
│  PERFORMANCE: Position accuracy ±0.01mm, Cycle time: ms level       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1.4.6 Aerospace and Defense

### Aircraft Flight Control System

```
┌─────────────────────────────────────────────────────────────────────┐
│                 FLY-BY-WIRE FLIGHT CONTROL SYSTEM                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│    ┌────────────────────────────────────────────────────────────┐   │
│    │                      PILOT INPUTS                           │   │
│    │   ┌──────────┐  ┌──────────┐  ┌──────────────────────┐    │   │
│    │   │  Stick   │  │  Pedals  │  │  Throttle Lever      │    │   │
│    │   │ (Roll/   │  │  (Yaw)   │  │                      │    │   │
│    │   │  Pitch)  │  │          │  │                      │    │   │
│    │   └────┬─────┘  └────┬─────┘  └──────────┬───────────┘    │   │
│    └────────┼─────────────┼───────────────────┼────────────────┘   │
│             │             │                   │                     │
│             └─────────────┴─────────┬─────────┘                     │
│                                     │                                │
│    ┌────────────────────────────────▼───────────────────────────┐   │
│    │              FLIGHT CONTROL COMPUTERS (FCC)                 │   │
│    │   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │   │
│    │   │   FCC #1    │  │   FCC #2    │  │   FCC #3    │       │   │
│    │   │  (Primary)  │  │ (Secondary) │  │  (Backup)   │       │   │
│    │   │             │  │             │  │             │       │   │
│    │   │ Voting logic for redundancy - 2 of 3 agreement       │   │
│    │   └─────────────┴──┴─────────────┴──┴─────────────┘       │   │
│    └────────────────────────────┬───────────────────────────────┘   │
│                                 │                                    │
│    ┌────────────────────────────▼───────────────────────────────┐   │
│    │                      CONTROL SURFACES                       │   │
│    │   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │   │
│    │   │ Ailerons │  │ Elevator │  │  Rudder  │  │  Flaps   │  │   │
│    │   │          │  │          │  │          │  │          │  │   │
│    │   │ Hydraulic actuators with position feedback            │   │
│    │   └──────────┘  └──────────┘  └──────────┘  └──────────┘  │   │
│    └────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  REDUNDANCY: Triple-redundant FCCs, Dual hydraulic systems          │
│  CERTIFICATION: DO-178C (DAL A - Catastrophic failure)              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Aerospace Requirements Table

| Requirement | Standard | Description |
|-------------|----------|-------------|
| Software | DO-178C | Software certification (DAL A-E) |
| Hardware | DO-254 | Electronic hardware certification |
| Systems | ARP 4754A | System development process |
| Safety | ARP 4761 | Safety assessment process |
| Environment | DO-160G | Environmental conditions |

---

## 1.4.7 Internet of Things (IoT)

### Smart Home Ecosystem

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SMART HOME IoT ECOSYSTEM                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                        ┌─────────────┐                              │
│                        │   CLOUD     │                              │
│                        │  PLATFORM   │                              │
│                        │ (AWS/Azure) │                              │
│                        └──────┬──────┘                              │
│                               │                                      │
│                        ┌──────┴──────┐                              │
│                        │   HOME      │                              │
│                        │   GATEWAY   │                              │
│                        │ (WiFi Router│                              │
│                        │  + Hub)     │                              │
│                        └──────┬──────┘                              │
│                               │                                      │
│    ┌──────────────────────────┼──────────────────────────┐          │
│    │         │         │      │      │         │         │          │
│    ▼         ▼         ▼      │      ▼         ▼         ▼          │
│ ┌─────┐  ┌─────┐  ┌─────┐    │   ┌─────┐  ┌─────┐  ┌─────┐        │
│ │Smart│  │Smart│  │Smart│    │   │Smart│  │Motion│  │Door │        │
│ │Light│  │Therm│  │Lock │    │   │Plug │  │Sensor│  │Bell │        │
│ │     │  │     │  │     │    │   │     │  │     │  │     │        │
│ │WiFi │  │WiFi │  │Z-Wave│   │   │WiFi │  │Zigbee│  │WiFi │        │
│ └─────┘  └─────┘  └─────┘    │   └─────┘  └─────┘  └─────┘        │
│                              │                                       │
│    PROTOCOLS:                │                                       │
│    ┌─────────────────────────┴───────────────────────────┐          │
│    │  WiFi  │  Zigbee  │  Z-Wave  │  BLE  │  Thread     │          │
│    └─────────────────────────────────────────────────────┘          │
│                                                                      │
│  TYPICAL IoT NODE:                                                   │
│  ┌───────────────────────────────────────────────────────┐          │
│  │  MCU (ESP32, nRF52)  │  Sensors  │  Radio  │ Battery │          │
│  │  RTOS/Bare-metal     │  (varies) │ (varies)│ 2-5 yrs │          │
│  └───────────────────────────────────────────────────────┘          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### IoT Device Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                   GENERIC IoT SENSOR NODE                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│    ┌────────────────────────────────────────────────────────────┐   │
│    │                        SENSOR NODE                          │   │
│    │                                                             │   │
│    │   ┌─────────────────────────────────────────────────────┐  │   │
│    │   │                 MICROCONTROLLER                      │  │   │
│    │   │  ┌─────────┐  ┌─────────┐  ┌─────────────────────┐  │  │   │
│    │   │  │ ARM     │  │ Flash   │  │  Peripherals        │  │  │   │
│    │   │  │ Cortex-M│  │ 256KB   │  │  GPIO, ADC, I2C     │  │  │   │
│    │   │  │ 48MHz   │  │ RAM 64KB│  │  SPI, UART, Timer   │  │  │   │
│    │   │  └─────────┘  └─────────┘  └─────────────────────┘  │  │   │
│    │   └─────────────────────────────────────────────────────┘  │   │
│    │                         │                                   │   │
│    │   ┌─────────────────────┼─────────────────────────┐        │   │
│    │   │                     │                         │        │   │
│    │   ▼                     ▼                         ▼        │   │
│    │ ┌─────────┐       ┌─────────┐              ┌─────────┐    │   │
│    │ │ SENSORS │       │ RADIO   │              │  POWER  │    │   │
│    │ │         │       │         │              │         │    │   │
│    │ │•Temp/Hum│       │•WiFi    │              │•Battery │    │   │
│    │ │•Light   │       │•BLE     │              │•LDO Reg │    │   │
│    │ │•Motion  │       │•LoRa    │              │•Solar   │    │   │
│    │ │•Air qual│       │•Zigbee  │              │ (option)│    │   │
│    │ └─────────┘       └─────────┘              └─────────┘    │   │
│    └────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  KEY METRICS:                                                        │
│  • Power: 10µA sleep, 20mA active                                   │
│  • Battery life: 2-5 years on coin cell                             │
│  • Range: 10m (BLE) to 10km (LoRa)                                  │
│  • Data rate: 250kbps (Zigbee) to 2Mbps (BLE 5)                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1.4.8 Telecommunications

### Network Router/Switch

```
┌─────────────────────────────────────────────────────────────────────┐
│                    NETWORK ROUTER ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│    ┌────────────────────────────────────────────────────────────┐   │
│    │                     ROUTER SYSTEM                           │   │
│    │                                                             │   │
│    │   ┌─────────────────────────────────────────────────────┐  │   │
│    │   │              CONTROL PLANE                           │  │   │
│    │   │  ┌───────────────┐  ┌───────────────────────────┐   │  │   │
│    │   │  │ Management CPU│  │ Routing Protocols         │   │  │   │
│    │   │  │ (ARM/MIPS)    │  │ OSPF, BGP, RIP            │   │  │   │
│    │   │  └───────────────┘  └───────────────────────────┘   │  │   │
│    │   └─────────────────────────────────────────────────────┘  │   │
│    │                         │                                   │   │
│    │   ┌─────────────────────┼───────────────────────────────┐  │   │
│    │   │              DATA PLANE                              │  │   │
│    │   │  ┌───────────────┐  ┌───────────────────────────┐   │  │   │
│    │   │  │ Packet        │  │ Forwarding Engine         │   │  │   │
│    │   │  │ Processor     │  │ (ASIC/FPGA)               │   │  │   │
│    │   │  └───────────────┘  └───────────────────────────┘   │  │   │
│    │   └─────────────────────────────────────────────────────┘  │   │
│    │                         │                                   │   │
│    │   ┌─────────────────────┴───────────────────────────────┐  │   │
│    │   │                  INTERFACES                          │  │   │
│    │   │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐     │  │   │
│    │   │  │GbE x4│ │WiFi 6│ │USB   │ │WAN   │ │Console│     │  │   │
│    │   │  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘     │  │   │
│    │   └─────────────────────────────────────────────────────┘  │   │
│    └────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  PERFORMANCE: Line-rate forwarding, <1µs latency                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1.4.9 Applications Summary by Domain

| Domain | Example Applications | Key Technologies | Criticality |
|--------|---------------------|------------------|-------------|
| **Consumer** | TV, Appliances, Cameras | ARM SoC, WiFi, BT | Low-Medium |
| **Automotive** | ECU, ADAS, Infotainment | CAN, Ethernet, ARM | High-Critical |
| **Medical** | Pacemaker, Monitors, Imaging | Ultra-low power, Safety MCU | Critical |
| **Industrial** | PLC, Robots, SCADA | Real-time, Fieldbus | High |
| **Aerospace** | Flight control, Avionics | DO-178C, Redundancy | Critical |
| **IoT** | Sensors, Smart home | WiFi, BLE, Zigbee | Low-Medium |
| **Telecom** | Routers, Base stations | Network processors | High |
| **Military** | Radar, Weapons, Comm | Radiation-hardened | Critical |

---

## Summary Table

| Application Area | Embedded Examples | Key Requirements |
|-----------------|-------------------|------------------|
| **Home** | Thermostat, appliances, entertainment | Cost, ease of use |
| **Transportation** | Engine control, safety systems, navigation | Reliability, safety |
| **Healthcare** | Monitoring, life support, diagnostics | Accuracy, certification |
| **Industry** | Automation, robotics, process control | Determinism, reliability |
| **Communication** | Routers, phones, base stations | Throughput, latency |
| **Energy** | Smart grid, meters, controllers | Efficiency, security |
| **Agriculture** | Irrigation, monitoring, drones | Ruggedness, low power |

---

## Quick Revision Questions

1. **List five embedded systems you interact with in your daily morning routine.**

2. **Describe the major embedded systems found in a modern automobile and their functions.**

3. **What are the key requirements for medical embedded devices? Why is reliability critical?**

4. **Explain the architecture of a PLC-based industrial control system.**

5. **How does a fly-by-wire flight control system ensure safety through redundancy?**

6. **Compare the embedded system requirements for consumer electronics versus aerospace applications.**

---

## Navigation

| Previous | Up | Next |
|----------|------|------|
| [Classification of Embedded Systems](03-classification.md) | [Unit 1 Index](README.md) | [Design Challenges](05-design-challenges.md) |

---
