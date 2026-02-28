# Chapter 1.3: Classification of Embedded Systems

## Chapter Overview

This chapter explores the various ways embedded systems can be classified. Understanding these classifications helps in system selection, design decisions, and identifying appropriate solutions for specific applications. We examine classifications based on complexity, performance requirements, functional characteristics, and application domains.

---

## 1.3.1 Classification Overview

### Multiple Dimensions of Classification

```
┌─────────────────────────────────────────────────────────────────────┐
│              EMBEDDED SYSTEM CLASSIFICATION DIMENSIONS               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                    ┌───────────────────┐                            │
│                    │    EMBEDDED       │                            │
│                    │    SYSTEMS        │                            │
│                    └─────────┬─────────┘                            │
│                              │                                       │
│      ┌───────────────────────┼───────────────────────┐              │
│      │           │           │           │           │              │
│      ▼           ▼           ▼           ▼           ▼              │
│  ┌───────┐  ┌───────┐  ┌───────┐  ┌───────┐  ┌───────┐            │
│  │  By   │  │  By   │  │  By   │  │  By   │  │  By   │            │
│  │ Scale │  │ Perf. │  │ Func. │  │Domain │  │  RT   │            │
│  │ /Size │  │Require│  │Require│  │       │  │Require│            │
│  └───────┘  └───────┘  └───────┘  └───────┘  └───────┘            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1.3.2 Classification by Scale and Complexity

### The Complexity Pyramid

```
┌─────────────────────────────────────────────────────────────────────┐
│                CLASSIFICATION BY SCALE/COMPLEXITY                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                            /\                                        │
│                           /  \                                       │
│                          /    \     SOPHISTICATED                   │
│                         / SOC  \    (System-on-Chip)                │
│                        /--------\                                    │
│                       /          \                                   │
│                      /   MEDIUM   \  MEDIUM SCALE                   │
│                     /    SCALE     \                                 │
│                    /--------------\                                 │
│                   /                \                                 │
│                  /      SMALL       \  SMALL SCALE                  │
│                 /       SCALE        \                               │
│                /──────────────────────\                              │
│                                                                      │
│  COMPLEXITY:  LOW ◄───────────────────────────────────► HIGH        │
│  COST:        LOW ◄───────────────────────────────────► HIGH        │
│  CAPABILITY:  LIMITED ◄───────────────────────────────► EXTENSIVE   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Detailed Classification

#### 1. Small-Scale Embedded Systems

```
┌─────────────────────────────────────────────────────────────────────┐
│                   SMALL-SCALE EMBEDDED SYSTEMS                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  CHARACTERISTICS:                                                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ • 8-bit or 16-bit microcontroller                           │   │
│  │ • Memory: 1KB - 64KB ROM, 64B - 2KB RAM                     │   │
│  │ • Simple functionality                                       │   │
│  │ • Battery or small power supply                              │   │
│  │ • Single developer can handle entire project                 │   │
│  │ • Programming: Assembly or Embedded C                        │   │
│  │ • Often bare-metal (no OS)                                   │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  TYPICAL ARCHITECTURE:                                               │
│  ┌─────────────────────────────────────────┐                        │
│  │          8-bit Microcontroller          │                        │
│  │  ┌────────┐ ┌────────┐ ┌────────────┐  │                        │
│  │  │  CPU   │ │ 2KB    │ │   8 GPIO   │  │                        │
│  │  │  8MHz  │ │ Flash  │ │   2 Timer  │  │                        │
│  │  └────────┘ └────────┘ │   1 UART   │  │                        │
│  │                        └────────────┘  │                        │
│  └─────────────────────────────────────────┘                        │
│                                                                      │
│  EXAMPLES:                                                          │
│  • Electronic toys           • Simple thermometers                  │
│  • Digital watches           • Basic remote controls                │
│  • Greeting cards            • LED controllers                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

#### 2. Medium-Scale Embedded Systems

```
┌─────────────────────────────────────────────────────────────────────┐
│                   MEDIUM-SCALE EMBEDDED SYSTEMS                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  CHARACTERISTICS:                                                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ • 16-bit or 32-bit microcontroller                          │   │
│  │ • Memory: 64KB - 512KB ROM, 2KB - 64KB RAM                  │   │
│  │ • Moderate complexity                                        │   │
│  │ • May use RTOS                                               │   │
│  │ • Team of 2-5 developers                                     │   │
│  │ • Programming: C, C++                                        │   │
│  │ • Multiple peripherals and communication interfaces         │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  TYPICAL ARCHITECTURE:                                               │
│  ┌───────────────────────────────────────────────────────────┐     │
│  │              32-bit ARM Cortex-M Microcontroller          │     │
│  │  ┌────────┐ ┌────────┐ ┌────────────────────────────┐    │     │
│  │  │  CPU   │ │ 256KB  │ │   16+ GPIO                 │    │     │
│  │  │ 48MHz  │ │ Flash  │ │   4+ Timers               │    │     │
│  │  └────────┘ └────────┘ │   2+ UART, SPI, I2C       │    │     │
│  │  ┌────────┐ ┌────────┐ │   ADC, DAC, PWM           │    │     │
│  │  │ 32KB   │ │  DMA   │ │   USB, CAN               │    │     │
│  │  │  RAM   │ │        │ └────────────────────────────┘    │     │
│  │  └────────┘ └────────┘                                   │     │
│  └───────────────────────────────────────────────────────────┘     │
│                                                                      │
│  EXAMPLES:                                                          │
│  • Industrial controllers    • Building automation                  │
│  • Medical instruments       • Point-of-sale terminals              │
│  • Automotive sensors        • Smart home devices                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

#### 3. Sophisticated/Complex Embedded Systems

```
┌─────────────────────────────────────────────────────────────────────┐
│              SOPHISTICATED/COMPLEX EMBEDDED SYSTEMS                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  CHARACTERISTICS:                                                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ • 32-bit or 64-bit processors (often multi-core)            │   │
│  │ • Memory: MBs to GBs                                         │   │
│  │ • High complexity, multiple functions                        │   │
│  │ • Full RTOS or embedded Linux                                │   │
│  │ • Large development team                                     │   │
│  │ • Hardware-software co-design                                │   │
│  │ • Advanced peripherals, networking                           │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  TYPICAL ARCHITECTURE:                                               │
│  ┌───────────────────────────────────────────────────────────┐     │
│  │           System-on-Chip (SoC) Architecture               │     │
│  │  ┌─────────────────────────────────────────────────────┐ │     │
│  │  │  Multi-Core CPU     │    GPU      │   DSP/NPU      │ │     │
│  │  │  (ARM Cortex-A)     │             │               │ │     │
│  │  └─────────────────────────────────────────────────────┘ │     │
│  │  ┌─────────────────────────────────────────────────────┐ │     │
│  │  │ DDR RAM  │ Flash/eMMC │ USB 3.0 │ PCIe │ Ethernet │ │     │
│  │  │ (1-8GB)  │ (8-128GB)  │         │      │ WiFi/BT  │ │     │
│  │  └─────────────────────────────────────────────────────┘ │     │
│  │  ┌─────────────────────────────────────────────────────┐ │     │
│  │  │ Camera I/F │ Display │ Audio │ Security │ Sensors  │ │     │
│  │  └─────────────────────────────────────────────────────┘ │     │
│  └───────────────────────────────────────────────────────────┘     │
│                                                                      │
│  EXAMPLES:                                                          │
│  • Smartphones              • Advanced driver assistance (ADAS)     │
│  • Smart TVs                • Industrial robots                     │
│  • Network routers          • Medical imaging equipment             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Scale Comparison Table

| Aspect | Small Scale | Medium Scale | Sophisticated |
|--------|-------------|--------------|---------------|
| **Processor** | 8/16-bit MCU | 16/32-bit MCU | 32/64-bit SoC |
| **Clock Speed** | 1-20 MHz | 20-200 MHz | 200 MHz - 2+ GHz |
| **Program Memory** | 1-64 KB | 64-512 KB | 1 MB - 128+ GB |
| **RAM** | 64B - 2 KB | 2-64 KB | 1 MB - 8+ GB |
| **OS** | None/Simple | RTOS optional | RTOS/Linux |
| **Development** | 1 person | 2-5 people | Large team |
| **Cost** | $0.10 - $5 | $5 - $50 | $20 - $500+ |
| **Power** | µW - mW | mW - 100mW | 100mW - 10W |

---

## 1.3.3 Classification by Performance Requirements

### Performance Categories

```
┌─────────────────────────────────────────────────────────────────────┐
│              CLASSIFICATION BY PERFORMANCE REQUIREMENTS              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────────┐                                             │
│  │ TRANSFORMATIVE     │ Continuous data processing                  │
│  │ (Stream-oriented)  │ Audio/Video processing                      │
│  │                    │ Signal filtering                            │
│  └────────────────────┘                                             │
│           │                                                          │
│           ▼                                                          │
│  ┌────────────────────┐                                             │
│  │ REACTIVE           │ Event-driven response                       │
│  │ (Event-oriented)   │ Interrupt handling                          │
│  │                    │ Sensor monitoring                           │
│  └────────────────────┘                                             │
│           │                                                          │
│           ▼                                                          │
│  ┌────────────────────┐                                             │
│  │ INTERACTIVE        │ Human-machine interface                     │
│  │ (User-oriented)    │ Display updates                             │
│  │                    │ Input processing                            │
│  └────────────────────┘                                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Performance Metrics

| Metric | Low Performance | Medium Performance | High Performance |
|--------|-----------------|-------------------|------------------|
| **MIPS** | < 1 | 1 - 100 | 100 - 10,000+ |
| **Latency** | > 10 ms | 1 - 10 ms | < 1 ms |
| **Throughput** | < 100 Kbps | 100 Kbps - 10 Mbps | > 10 Mbps |
| **Response** | Seconds | Milliseconds | Microseconds |

---

## 1.3.4 Classification by Real-Time Requirements

### Real-Time Spectrum

```
┌─────────────────────────────────────────────────────────────────────┐
│                CLASSIFICATION BY REAL-TIME REQUIREMENTS              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  DEADLINE STRICTNESS:                                               │
│                                                                      │
│       HARD                FIRM               SOFT           NON-    │
│    REAL-TIME          REAL-TIME          REAL-TIME      REAL-TIME   │
│        │                  │                  │              │       │
│        ▼                  ▼                  ▼              ▼       │
│  ┌──────────┐       ┌──────────┐       ┌──────────┐  ┌──────────┐  │
│  │ Deadline │       │ Deadline │       │ Deadline │  │    No    │  │
│  │ MUST be  │       │ SHOULD   │       │ PREFERRED│  │ Deadline │  │
│  │   met    │       │  be met  │       │  met     │  │          │  │
│  └──────────┘       └──────────┘       └──────────┘  └──────────┘  │
│                                                                      │
│  Miss =             Miss =              Miss =         Best effort  │
│  CATASTROPHE        USELESS RESULT      DEGRADED                    │
│                                                                      │
│  Examples:          Examples:           Examples:      Examples:    │
│  • Airbag           • Video frame       • Web server   • Batch      │
│  • Pacemaker        • Audio sample      • Email        • Background │
│  • ABS brake        • Packet drop       • Logging      • Updates    │
│  • Flight ctrl                                                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Real-Time Response Timing

```
TIMING DIAGRAM - REAL-TIME RESPONSE:

Event occurs
    │
    │ ┌─────────────────────────────────────────────────────────────┐
    │ │                    DEADLINE                                 │
    │ │                       │                                     │
    ▼ │                       ▼                                     │
────┬─┴───────────────────────┬─────────────────────────────────────►
    │   Processing Time       │                               Time
    │◄───────────────────────►│
    │                         │
    │    VALID RESPONSE       │    INVALID (for hard RT)
    │       ZONE              │
    │◄───────────────────────►│◄──────────────────────────────────►
    
For Hard RT:  Response after deadline = System failure
For Soft RT:  Response after deadline = Acceptable with penalty
```

### Real-Time Categories Table

| Category | Deadline | Miss Consequence | Latency | Examples |
|----------|----------|-----------------|---------|----------|
| **Hard RT** | Absolute | Catastrophic | µs - ms | Airbag, pacemaker |
| **Firm RT** | Important | Result worthless | ms | Video frame, VoIP |
| **Soft RT** | Flexible | Quality degrades | ms - s | GUI, networking |
| **Non-RT** | None | Acceptable | Any | Batch processing |

---

## 1.3.5 Classification by Functional Requirements

### Functional Categories

```
┌─────────────────────────────────────────────────────────────────────┐
│              CLASSIFICATION BY FUNCTIONAL REQUIREMENTS               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    STAND-ALONE SYSTEMS                       │   │
│  │  • Self-contained operation                                  │   │
│  │  • No external host required                                 │   │
│  │  • Examples: Digital cameras, MP3 players, washing machines  │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                             │                                        │
│                             ▼                                        │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    NETWORKED SYSTEMS                         │   │
│  │  • Connected to network (LAN/WAN/Internet)                   │   │
│  │  • Can be accessed remotely                                  │   │
│  │  • Examples: Web cameras, network printers, smart home       │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                             │                                        │
│                             ▼                                        │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    MOBILE SYSTEMS                            │   │
│  │  • Portable, battery-powered                                 │   │
│  │  • Size and weight constraints                               │   │
│  │  • Examples: Smartphones, fitness trackers, drones           │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Control System Classification

```
CONTROL-BASED CLASSIFICATION:

┌─────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  ┌──────────────────────┐        ┌──────────────────────┐          │
│  │   OPEN-LOOP CONTROL  │        │  CLOSED-LOOP CONTROL │          │
│  └──────────────────────┘        └──────────────────────┘          │
│                                                                      │
│  ┌────────┐    ┌──────────┐      ┌────────┐    ┌──────────┐        │
│  │ Input  │───►│Controller│──────│ Input  │───►│Controller│─┐      │
│  └────────┘    └────┬─────┘      └────────┘    └────┬─────┘ │      │
│                     │                               │       │      │
│                     ▼                               ▼       │      │
│               ┌──────────┐                    ┌──────────┐  │      │
│               │  Plant/  │                    │  Plant/  │  │      │
│               │  Process │                    │  Process │  │      │
│               └────┬─────┘                    └────┬─────┘  │      │
│                    │                               │        │      │
│                    ▼                               ▼        │      │
│               ┌──────────┐                    ┌──────────┐  │      │
│               │  Output  │                    │  Output  │◄─┘      │
│               └──────────┘                    └────┬─────┘  ▲      │
│                                                    │        │      │
│  No feedback                                       └────────┘      │
│  Examples:                                    Feedback loop        │
│  • Toaster                                   Examples:             │
│  • Microwave timer                           • Thermostat          │
│  • Clothes dryer                             • Cruise control      │
│                                              • Robot arm           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1.3.6 Classification by Application Domain

### Industry-wise Classification

```
┌─────────────────────────────────────────────────────────────────────┐
│                 CLASSIFICATION BY APPLICATION DOMAIN                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐      │
│  │CONSUMER │ │INDUSTRIAL│ │AUTOMOTIVE│ │ MEDICAL │ │AEROSPACE│      │
│  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘      │
│       │           │           │           │           │            │
│  ┌────┴────┐ ┌────┴────┐ ┌────┴────┐ ┌────┴────┐ ┌────┴────┐      │
│  │• Smart  │ │• PLC    │ │• ECU    │ │• Pace-  │ │• Flight │      │
│  │  TV     │ │• SCADA  │ │• ABS    │ │  maker  │ │  Control│      │
│  │• Washer │ │• Robots │ │• Airbag │ │• Monitor│ │• Avionics│     │
│  │• Camera │ │• CNC    │ │• Infotain│ │• Imaging│ │• Satellite│   │
│  │• Gaming │ │• Sensors│ │• ADAS   │ │• Pumps  │ │• Weapons│      │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘      │
│                                                                      │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐      │
│  │TELECOM  │ │  SMART  │ │   IoT   │ │ OFFICE  │ │ MILITARY│      │
│  │         │ │  HOME   │ │         │ │ AUTOMAT.│ │ DEFENSE │      │
│  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘      │
│       │           │           │           │           │            │
│  ┌────┴────┐ ┌────┴────┐ ┌────┴────┐ ┌────┴────┐ ┌────┴────┐      │
│  │• Routers│ │• Thermo-│ │• Sensors│ │• Printer│ │• Radar  │      │
│  │• Modems │ │  stat   │ │• Wearables│ │• Copier│ │• Guided │      │
│  │• Base   │ │• Locks  │ │• Track- │ │• Access │ │  Missile│      │
│  │ Stations│ │• Lights │ │  ers    │ │ Control│ │• Comm   │      │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Domain Requirements Matrix

| Domain | Safety Level | Real-Time | Reliability | Power | Cost Sensitivity |
|--------|-------------|-----------|-------------|-------|------------------|
| **Consumer** | Low | Soft | Moderate | Moderate | High |
| **Industrial** | Medium-High | Hard/Firm | High | Low | Medium |
| **Automotive** | Very High | Hard | Very High | Moderate | Medium |
| **Medical** | Critical | Hard | Critical | Varies | Low |
| **Aerospace** | Critical | Hard | Mission-Critical | Varies | Low |
| **IoT** | Low-Medium | Soft | Moderate | Very High | High |

---

## 1.3.7 Classification by Microcontroller vs Microprocessor

### Architecture-Based Classification

```
┌─────────────────────────────────────────────────────────────────────┐
│           MICROCONTROLLER vs MICROPROCESSOR BASED SYSTEMS            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  MICROCONTROLLER-BASED:              MICROPROCESSOR-BASED:          │
│  (Self-contained)                    (Requires external components) │
│                                                                      │
│  ┌─────────────────────────────┐    ┌─────────────────────────────┐│
│  │    Single Chip Solution     │    │      Multi-Chip Solution    ││
│  │  ┌───────────────────────┐  │    │                             ││
│  │  │ ┌─────┐ ┌─────┐       │  │    │  ┌───────┐                  ││
│  │  │ │ CPU │ │ RAM │       │  │    │  │  CPU  │                  ││
│  │  │ └─────┘ └─────┘       │  │    │  │(only) │                  ││
│  │  │ ┌─────┐ ┌─────┐       │  │    │  └───┬───┘                  ││
│  │  │ │Flash│ │Timer│       │  │    │      │                      ││
│  │  │ └─────┘ └─────┘       │  │    │  ┌───┴───────────────────┐  ││
│  │  │ ┌─────┐ ┌─────┐       │  │    │  │       SYSTEM BUS      │  ││
│  │  │ │GPIO │ │UART │       │  │    │  └───┬───────┬───────┬───┘  ││
│  │  │ └─────┘ └─────┘       │  │    │      │       │       │      ││
│  │  │ ┌─────┐ ┌─────┐       │  │    │  ┌───┴──┐┌───┴──┐┌───┴──┐  ││
│  │  │ │ ADC │ │ I2C │       │  │    │  │ RAM  ││ ROM  ││ I/O  │  ││
│  │  │ └─────┘ └─────┘       │  │    │  │(Ext) ││(Ext) ││(Ext) │  ││
│  │  └───────────────────────┘  │    │  └──────┘└──────┘└──────┘  ││
│  └─────────────────────────────┘    └─────────────────────────────┘│
│                                                                      │
│  Examples:                          Examples:                       │
│  • 8051, AVR, PIC, STM32            • x86-based, ARM Cortex-A      │
│  • ESP32, Arduino                   • Raspberry Pi, BeagleBone     │
│                                                                      │
│  Use Case:                          Use Case:                       │
│  • Simple to medium embedded        • Complex, Linux-based         │
│  • Cost-sensitive                   • High performance needed      │
│  • Low power applications           • Rich OS features needed      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1.3.8 Classification Summary Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│             COMPLETE EMBEDDED SYSTEM CLASSIFICATION                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                    ┌─────────────────────┐                          │
│                    │  EMBEDDED SYSTEMS   │                          │
│                    └──────────┬──────────┘                          │
│           ┌───────────────────┼───────────────────┐                 │
│           │                   │                   │                 │
│    BY COMPLEXITY       BY REAL-TIME        BY FUNCTION              │
│           │                   │                   │                 │
│    ┌──────┴──────┐     ┌──────┴──────┐    ┌──────┴──────┐          │
│    │• Small      │     │• Hard RT    │    │• Stand-alone│          │
│    │• Medium     │     │• Firm RT    │    │• Networked  │          │
│    │• Sophisticated   │• Soft RT    │    │• Mobile     │          │
│    └─────────────┘     │• Non-RT    │    └─────────────┘          │
│                        └─────────────┘                              │
│           │                   │                   │                 │
│    BY ARCHITECTURE     BY PERFORMANCE       BY DOMAIN               │
│           │                   │                   │                 │
│    ┌──────┴──────┐     ┌──────┴──────┐    ┌──────┴──────┐          │
│    │• MCU-based  │     │• Transform. │    │• Consumer   │          │
│    │• MPU-based  │     │• Reactive   │    │• Industrial │          │
│    │• SoC-based  │     │• Interactive│    │• Automotive │          │
│    │• DSP-based  │     └─────────────┘    │• Medical    │          │
│    └─────────────┘                        │• Aerospace  │          │
│                                           │• IoT        │          │
│                                           └─────────────┘          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Classification Basis | Categories | Key Distinguishing Factor |
|---------------------|------------|--------------------------|
| **By Scale/Complexity** | Small, Medium, Sophisticated | Resources and capabilities |
| **By Real-Time** | Hard, Firm, Soft, Non-RT | Deadline strictness |
| **By Function** | Stand-alone, Networked, Mobile | Connectivity and portability |
| **By Control** | Open-loop, Closed-loop | Feedback mechanism |
| **By Performance** | Transformative, Reactive, Interactive | Data processing pattern |
| **By Architecture** | MCU-based, MPU-based, SoC-based | Hardware integration level |
| **By Domain** | Consumer, Industrial, Automotive, Medical, etc. | Application area |

---

## Quick Revision Questions

1. **What are the three main categories in classification by scale/complexity? Give two examples of each.**

2. **Explain the difference between hard real-time and soft real-time systems with examples.**

3. **Compare microcontroller-based and microprocessor-based embedded systems.**

4. **What factors distinguish a small-scale embedded system from a sophisticated embedded system?**

5. **List five different application domains for embedded systems and give one example device for each.**

6. **What is the difference between stand-alone and networked embedded systems?**

---

## Navigation

| Previous | Up | Next |
|----------|------|------|
| [Embedded vs General-Purpose](02-embedded-vs-general-purpose.md) | [Unit 1 Index](README.md) | [Applications and Examples](04-applications-and-examples.md) |

---
