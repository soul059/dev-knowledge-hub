# Chapter 2.1: Processor Selection Criteria

## Chapter Overview

Selecting the right processor is one of the most critical decisions in embedded system design. This chapter explores the various processor types available, their architectures, and the key criteria used to select the optimal processor for a specific application.

---

## 2.1.1 Types of Embedded Processors

### Processor Categories

```
┌─────────────────────────────────────────────────────────────────────┐
│                    EMBEDDED PROCESSOR TYPES                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                     MICROCONTROLLER (MCU)                      │  │
│  │  ┌────────────────────────────────────────────────────────┐   │  │
│  │  │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐     │   │  │
│  │  │  │ CPU │ │Flash│ │ RAM │ │Timer│ │ ADC │ │UART │     │   │  │
│  │  │  └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘     │   │  │
│  │  │              ALL ON SINGLE CHIP                        │   │  │
│  │  └────────────────────────────────────────────────────────┘   │  │
│  │  Examples: ATmega328, STM32F4, PIC18, ESP32                   │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                    MICROPROCESSOR (MPU)                        │  │
│  │  ┌─────┐     ┌─────┐     ┌─────┐     ┌─────┐                 │  │
│  │  │ CPU │◄───►│ RAM │◄───►│Flash│◄───►│ I/O │                 │  │
│  │  └─────┘     └─────┘     └─────┘     └─────┘                 │  │
│  │  (chip)     (external)  (external)  (external)                │  │
│  │  Examples: ARM Cortex-A, Intel Atom, AMD Embedded             │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │               DIGITAL SIGNAL PROCESSOR (DSP)                   │  │
│  │  ┌────────────────────────────────────────────────────────┐   │  │
│  │  │  • Hardware MAC (Multiply-Accumulate)                   │   │  │
│  │  │  • Parallel execution units                             │   │  │
│  │  │  • Specialized addressing modes                         │   │  │
│  │  │  • Optimized for filters, FFT, convolution              │   │  │
│  │  └────────────────────────────────────────────────────────┘   │  │
│  │  Examples: TI C6000, Analog Devices SHARC, Qualcomm Hexagon   │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │              SYSTEM-ON-CHIP (SoC)                              │  │
│  │  ┌────────────────────────────────────────────────────────┐   │  │
│  │  │ CPU │ GPU │ DSP │ ISP │ NPU │ Memory │ I/O │ Security │   │  │
│  │  └────────────────────────────────────────────────────────┘   │  │
│  │  Complete system on single die                                │  │
│  │  Examples: Qualcomm Snapdragon, Apple A-series, NXP i.MX      │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### MCU vs MPU Comparison

| Feature | Microcontroller (MCU) | Microprocessor (MPU) |
|---------|----------------------|---------------------|
| **Integration** | High (all-in-one) | Low (external memory/IO) |
| **Memory** | On-chip (KB-MB) | External (MB-GB) |
| **Clock Speed** | 1 MHz - 200 MHz | 200 MHz - 3+ GHz |
| **Power** | µW - mW | 100mW - 10W |
| **Cost** | $0.10 - $20 | $10 - $200 |
| **OS Support** | Bare-metal, RTOS | Full OS (Linux, Android) |
| **Boot Time** | Instant - ms | Seconds |
| **Typical Use** | Control, sensing | Multimedia, networking |

---

## 2.1.2 Processor Architectures

### RISC vs CISC

```
┌─────────────────────────────────────────────────────────────────────┐
│                      RISC vs CISC COMPARISON                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  CISC (Complex Instruction Set Computing):                          │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │                                                            │     │
│  │  • Many instructions (hundreds)                            │     │
│  │  • Variable instruction length (1-15 bytes)                │     │
│  │  • Memory-to-memory operations                             │     │
│  │  • Complex addressing modes                                │     │
│  │  • Multi-cycle instructions                                │     │
│  │                                                            │     │
│  │  Example: x86                                              │     │
│  │  ADD EAX, [EBX+ECX*4+offset]  ; Complex addressing        │     │
│  │                                                            │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  RISC (Reduced Instruction Set Computing):                          │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │                                                            │     │
│  │  • Fewer instructions (dozens)                             │     │
│  │  • Fixed instruction length (32 bits)                      │     │
│  │  • Load-store architecture                                 │     │
│  │  • Simple addressing modes                                 │     │
│  │  • Single-cycle execution (ideal)                          │     │
│  │                                                            │     │
│  │  Example: ARM                                              │     │
│  │  LDR R0, [R1]    ; Load from memory                       │     │
│  │  ADD R0, R0, R2  ; Add registers                          │     │
│  │  STR R0, [R1]    ; Store to memory                        │     │
│  │                                                            │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Popular Embedded Architectures

```
┌─────────────────────────────────────────────────────────────────────┐
│                    POPULAR EMBEDDED ARCHITECTURES                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ARM ARCHITECTURE FAMILY:                                            │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │                                                            │     │
│  │  Cortex-M Series (Microcontroller):                        │     │
│  │  ┌──────────────────────────────────────────────────────┐ │     │
│  │  │ M0/M0+ │  M3   │  M4   │  M7   │  M23  │ M33  │ M55 │ │     │
│  │  │ Ultra  │ Main- │ +DSP  │ High  │ Secure│ Secure│+NPU│ │     │
│  │  │ Low Pwr│ stream│ +FPU  │ Perf  │ IoT   │ +DSP  │    │ │     │
│  │  └──────────────────────────────────────────────────────┘ │     │
│  │                                                            │     │
│  │  Cortex-A Series (Application):                            │     │
│  │  ┌──────────────────────────────────────────────────────┐ │     │
│  │  │ A5/A7 │ A53/A55 │ A72/A75 │ A76/A78 │ X1/X2/X3      │ │     │
│  │  │ Entry │ Efficient│ High    │ Premium │ Ultra High    │ │     │
│  │  │ Level │ (Little) │ Perform │ Mobile  │ Performance   │ │     │
│  │  └──────────────────────────────────────────────────────┘ │     │
│  │                                                            │     │
│  │  Cortex-R Series (Real-time):                              │     │
│  │  ┌──────────────────────────────────────────────────────┐ │     │
│  │  │ R4/R5 │ R7  │ R8  │ R52 │ R82                        │ │     │
│  │  │ Deterministic, Safety-critical applications          │ │     │
│  │  └──────────────────────────────────────────────────────┘ │     │
│  │                                                            │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  OTHER ARCHITECTURES:                                                │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  • AVR (Atmel/Microchip): 8-bit, Arduino                  │     │
│  │  • PIC (Microchip): 8/16/32-bit, wide variety             │     │
│  │  • 8051 (Various): 8-bit, legacy, still popular           │     │
│  │  • MIPS: 32/64-bit, networking, IoT                       │     │
│  │  • RISC-V: Open-source ISA, growing adoption              │     │
│  │  • Xtensa (Tensilica): ESP32, configurable                │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2.1.3 Selection Criteria

### Primary Selection Factors

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PROCESSOR SELECTION CRITERIA                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                    ┌─────────────────────┐                          │
│                    │ APPLICATION         │                          │
│                    │ REQUIREMENTS        │                          │
│                    └──────────┬──────────┘                          │
│                               │                                      │
│    ┌──────────────────────────┼──────────────────────────┐          │
│    │          │          │    │    │          │          │          │
│    ▼          ▼          ▼    │    ▼          ▼          ▼          │
│ ┌──────┐  ┌──────┐  ┌──────┐  │ ┌──────┐  ┌──────┐  ┌──────┐      │
│ │Perfor│  │Power │  │Memory│  │ │Peri- │  │ Cost │  │Ecosys│      │
│ │mance │  │      │  │      │  │ │pherals│ │      │  │ tem  │      │
│ └──────┘  └──────┘  └──────┘  │ └──────┘  └──────┘  └──────┘      │
│                               │                                      │
│                    ┌──────────┴──────────┐                          │
│                    │  PROCESSOR          │                          │
│                    │  SELECTION          │                          │
│                    └─────────────────────┘                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Detailed Criteria Table

| Criterion | Questions to Ask | Metrics |
|-----------|-----------------|---------|
| **Performance** | How much processing power? | MIPS, DMIPS, CoreMark |
| **Power** | Battery or mains? Sleep modes? | Active/sleep current (µA, mA) |
| **Memory** | Code size? Data storage? | Flash (KB), RAM (KB) |
| **Peripherals** | What interfaces needed? | ADC, timers, UART, SPI, I2C, USB |
| **Real-time** | Hard deadlines? | Interrupt latency, determinism |
| **Cost** | Volume? Target price? | Unit price at volume |
| **Package** | Size constraints? | Pin count, package type |
| **Temperature** | Operating environment? | Industrial, automotive range |
| **Ecosystem** | Tool support? Libraries? | IDE, compiler, community |
| **Longevity** | Product lifetime? | Availability guarantees |

---

## 2.1.4 Performance Metrics

### Understanding Performance Specifications

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PERFORMANCE METRICS                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  CLOCK SPEED:                                                        │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  Frequency (MHz) = 1 / Clock Period                        │     │
│  │                                                            │     │
│  │  WARNING: Higher clock ≠ Better performance                │     │
│  │           Architecture and efficiency matter more          │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  MIPS (Million Instructions Per Second):                             │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │                                                            │     │
│  │  MIPS = Clock Frequency (MHz) / CPI                        │     │
│  │                                                            │     │
│  │  Where CPI = Cycles Per Instruction                        │     │
│  │                                                            │     │
│  │  Example: 100 MHz clock, CPI = 1.2                         │     │
│  │  MIPS = 100 / 1.2 = 83.3 MIPS                              │     │
│  │                                                            │     │
│  │  Note: MIPS varies by instruction mix                      │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  DMIPS (Dhrystone MIPS):                                             │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │                                                            │     │
│  │  Standardized benchmark for integer performance            │     │
│  │  DMIPS/MHz = Dhrystone score per MHz                       │     │
│  │                                                            │     │
│  │  Examples:                                                  │     │
│  │  • ARM Cortex-M0: 0.9 DMIPS/MHz                            │     │
│  │  • ARM Cortex-M3: 1.25 DMIPS/MHz                           │     │
│  │  • ARM Cortex-M4: 1.25 DMIPS/MHz                           │     │
│  │  • ARM Cortex-M7: 2.14 DMIPS/MHz                           │     │
│  │                                                            │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  COREMARK:                                                           │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │                                                            │     │
│  │  Industry-standard benchmark (EEMBC)                       │     │
│  │  Tests: list processing, matrix manipulation, state machine│     │
│  │                                                            │     │
│  │  CoreMark/MHz allows fair comparison across devices        │     │
│  │                                                            │     │
│  │  Examples:                                                  │     │
│  │  • ARM Cortex-M0+: 2.46 CoreMark/MHz                       │     │
│  │  • ARM Cortex-M4: 3.40 CoreMark/MHz                        │     │
│  │  • ARM Cortex-M7: 5.01 CoreMark/MHz                        │     │
│  │                                                            │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Performance Calculation Example

```
EXAMPLE: Calculate processing requirements for audio processing

Given:
  • Audio sample rate: 48 kHz
  • Bits per sample: 16
  • Processing per sample: 100 operations (filter, etc.)
  • Safety margin: 2x

Required MIPS:
  Operations/second = 48,000 × 100 = 4,800,000
  With margin = 4,800,000 × 2 = 9,600,000 = 9.6 MIPS

Processor selection:
  For ARM Cortex-M4 at 1.25 DMIPS/MHz:
  Required frequency = 9.6 / 1.25 = 7.7 MHz minimum
  
  Typical choice: 48 MHz Cortex-M4 (provides 60 DMIPS)
  Headroom: 60 / 9.6 = 6.25x margin ✓
```

---

## 2.1.5 Power Considerations

### Power Modes and Consumption

```
┌─────────────────────────────────────────────────────────────────────┐
│                    POWER CONSUMPTION ANALYSIS                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  TYPICAL MCU POWER MODES:                                            │
│                                                                      │
│  Current                                                             │
│    (mA)                                                              │
│      │                                                               │
│  100 ┤  ████                                                        │
│      │  ████                                                        │
│   10 ┤  ████  ████                                                  │
│      │  ████  ████                                                  │
│    1 ┤  ████  ████  ████                                            │
│      │  ████  ████  ████                                            │
│  0.1 ┤  ████  ████  ████  ████                                      │
│      │  ████  ████  ████  ████                                      │
│ 0.01 ┤  ████  ████  ████  ████  ████                                │
│      │  ████  ████  ████  ████  ████                                │
│      └──────────────────────────────────────────────────────────    │
│         RUN   SLEEP  STOP  STNDBY  OFF                              │
│        (100%) (10%)  (1%)  (0.1%) (0.01%)                           │
│                                                                      │
│  MODE CHARACTERISTICS:                                               │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Mode    │ CPU │ Periph │ RAM  │ Wake Time │ Wake Source   │     │
│  │─────────────────────────────────────────────────────────── │     │
│  │ Run     │ ON  │   ON   │ ON   │    -      │      -        │     │
│  │ Sleep   │ OFF │   ON   │ ON   │   ~1µs    │ Any interrupt │     │
│  │ Stop    │ OFF │  OFF   │ ON   │  ~10µs    │ RTC, GPIO     │     │
│  │ Standby │ OFF │  OFF   │ Ret  │  ~100µs   │ RTC, WKUP pin │     │
│  │ Off     │ OFF │  OFF   │ OFF  │   ~ms     │ Reset, WKUP   │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Power Budget Calculation

```
EXAMPLE: Battery Life Calculation

Device parameters:
  • Battery: 1000 mAh
  • Active current: 10 mA
  • Sleep current: 10 µA
  • Active time: 1 second every 60 seconds (1.67% duty cycle)
  
Average current:
  I_avg = (I_active × t_active + I_sleep × t_sleep) / t_total
  I_avg = (10mA × 1s + 0.01mA × 59s) / 60s
  I_avg = (10 + 0.59) / 60
  I_avg = 0.177 mA = 177 µA

Battery life:
  Life = Capacity / I_avg
  Life = 1000 mAh / 0.177 mA
  Life = 5650 hours ≈ 235 days

With 80% usable capacity: 188 days
```

---

## 2.1.6 Peripheral Requirements

### Common MCU Peripherals

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MCU PERIPHERAL OVERVIEW                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │                        MCU CORE                             │     │
│  │                          CPU                                │     │
│  └──────────────────────────┬─────────────────────────────────┘     │
│                             │                                        │
│         ┌───────────────────┼───────────────────┐                   │
│         │         │         │         │         │                   │
│         ▼         ▼         ▼         ▼         ▼                   │
│  ┌──────────┬──────────┬──────────┬──────────┬──────────┐          │
│  │   GPIO   │  Timers  │   ADC    │   DAC    │   DMA    │          │
│  │          │          │          │          │          │          │
│  │ Digital  │ Counting │ Analog   │ Analog   │ Memory   │          │
│  │ I/O pins │ PWM, Cap │ to Dig   │ to Dig   │ Transfer │          │
│  └──────────┴──────────┴──────────┴──────────┴──────────┘          │
│                                                                      │
│  ┌──────────┬──────────┬──────────┬──────────┬──────────┐          │
│  │   UART   │   SPI    │   I2C    │   USB    │   CAN    │          │
│  │          │          │          │          │          │          │
│  │ Serial   │ High-spd │ Multi-   │ Universal│ Automo-  │          │
│  │ Async    │ Serial   │ device   │ Serial   │ tive bus │          │
│  └──────────┴──────────┴──────────┴──────────┴──────────┘          │
│                                                                      │
│  ┌──────────┬──────────┬──────────┬──────────┬──────────┐          │
│  │   RTC    │ Watchdog │  Crypto  │ Ethernet │   LCD    │          │
│  │          │          │          │          │          │          │
│  │ Real-time│ Safety   │ Security │ Network  │ Display  │          │
│  │ Clock    │ Timer    │ Engine   │ Connect  │ Driver   │          │
│  └──────────┴──────────┴──────────┴──────────┴──────────┘          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Peripheral Selection Checklist

| Application | Required Peripherals |
|-------------|---------------------|
| **Sensor Node** | ADC, GPIO, UART/I2C, RTC, Low-power modes |
| **Motor Control** | PWM (multiple), ADC, GPIO, Timer |
| **Communication Hub** | UART, SPI, I2C, USB, Ethernet, WiFi |
| **Display Application** | LCD controller, DMA, SPI, I2C |
| **Audio Processing** | I2S, DMA, ADC/DAC, DSP instructions |
| **Security Device** | Crypto engine, RNG, secure storage |

---

## 2.1.7 Development Ecosystem

### Ecosystem Considerations

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DEVELOPMENT ECOSYSTEM                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  DEVELOPMENT TOOLS:                                                  │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │                                                            │     │
│  │  IDE & Compiler:                                           │     │
│  │  ├── Vendor IDEs (STM32CubeIDE, MPLAB X, Keil)            │     │
│  │  ├── Commercial (IAR, Keil MDK, CrossWorks)               │     │
│  │  └── Open source (GCC, Eclipse, VS Code + extensions)     │     │
│  │                                                            │     │
│  │  Debug Hardware:                                           │     │
│  │  ├── JTAG/SWD probes (J-Link, ST-Link, CMSIS-DAP)         │     │
│  │  └── In-circuit emulators                                  │     │
│  │                                                            │     │
│  │  Evaluation Boards:                                        │     │
│  │  ├── Vendor dev boards (Nucleo, LaunchPad, Curiosity)     │     │
│  │  └── Arduino/maker boards                                  │     │
│  │                                                            │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  SOFTWARE SUPPORT:                                                   │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │                                                            │     │
│  │  Vendor Libraries:                                         │     │
│  │  ├── HAL (Hardware Abstraction Layer)                      │     │
│  │  ├── LL (Low-Level) drivers                                │     │
│  │  └── Middleware (USB, TCP/IP, File system)                 │     │
│  │                                                            │     │
│  │  RTOS Options:                                             │     │
│  │  ├── FreeRTOS (most popular, free)                         │     │
│  │  ├── Zephyr (Linux Foundation, growing)                    │     │
│  │  ├── ThreadX (Azure RTOS, Microsoft)                       │     │
│  │  └── Vendor RTOS (embOS, µC/OS)                            │     │
│  │                                                            │     │
│  │  Community:                                                 │     │
│  │  ├── Forums, Stack Overflow presence                       │     │
│  │  ├── GitHub projects and examples                          │     │
│  │  └── YouTube tutorials, courses                            │     │
│  │                                                            │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2.1.8 Processor Selection Flowchart

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PROCESSOR SELECTION FLOWCHART                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                    ┌─────────────────────┐                          │
│                    │  Start: Define      │                          │
│                    │  Requirements       │                          │
│                    └──────────┬──────────┘                          │
│                               │                                      │
│                    ┌──────────▼──────────┐                          │
│              No    │  Need full OS       │    Yes                   │
│          ┌─────────┤  (Linux/Android)?   ├─────────┐                │
│          │         └─────────────────────┘         │                │
│          │                                         │                │
│          ▼                                         ▼                │
│  ┌───────────────┐                        ┌───────────────┐         │
│  │ MCU Candidate │                        │ MPU/SoC       │         │
│  └───────┬───────┘                        │ Candidate     │         │
│          │                                └───────────────┘         │
│          │                                                          │
│  ┌───────▼───────┐                                                  │
│  │ Check Memory  │                                                  │
│  │ Requirements  │                                                  │
│  └───────┬───────┘                                                  │
│          │                                                          │
│  ┌───────▼───────┐                                                  │
│  │ Check Periph  │                                                  │
│  │ Requirements  │                                                  │
│  └───────┬───────┘                                                  │
│          │                                                          │
│  ┌───────▼───────┐                                                  │
│  │ Check Power   │                                                  │
│  │ Requirements  │                                                  │
│  └───────┬───────┘                                                  │
│          │                                                          │
│  ┌───────▼───────┐                                                  │
│  │ Check Cost    │                                                  │
│  │ Constraints   │                                                  │
│  └───────┬───────┘                                                  │
│          │                                                          │
│  ┌───────▼───────┐                                                  │
│  │ Evaluate      │                                                  │
│  │ Ecosystem     │                                                  │
│  └───────┬───────┘                                                  │
│          │                                                          │
│  ┌───────▼───────┐                                                  │
│  │ Select Top    │                                                  │
│  │ 2-3 Options   │                                                  │
│  └───────┬───────┘                                                  │
│          │                                                          │
│  ┌───────▼───────┐                                                  │
│  │ Prototype &   │                                                  │
│  │ Validate      │                                                  │
│  └───────────────┘                                                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2.1.9 Popular Processors Comparison

### MCU Comparison Table

| Processor | Architecture | Speed | Flash | RAM | Power | Price |
|-----------|--------------|-------|-------|-----|-------|-------|
| ATmega328P | 8-bit AVR | 20 MHz | 32 KB | 2 KB | Low | $2 |
| STM32F103 | ARM Cortex-M3 | 72 MHz | 64-512 KB | 20-64 KB | Low | $2-5 |
| STM32F407 | ARM Cortex-M4 | 168 MHz | 512 KB-1 MB | 192 KB | Med | $8-12 |
| STM32H743 | ARM Cortex-M7 | 480 MHz | 2 MB | 1 MB | Med | $12-20 |
| ESP32 | Xtensa Dual | 240 MHz | 4 MB | 520 KB | Med | $3-5 |
| nRF52840 | ARM Cortex-M4 | 64 MHz | 1 MB | 256 KB | V.Low | $4-6 |

---

## Summary Table

| Selection Factor | Key Considerations | Typical Trade-offs |
|-----------------|-------------------|-------------------|
| **Architecture** | 8/16/32-bit, RISC/CISC | Simplicity vs capability |
| **Performance** | MIPS, CoreMark, Clock | Power, cost |
| **Power** | Active, sleep, modes | Performance, features |
| **Memory** | Flash, RAM, cache | Cost, package size |
| **Peripherals** | ADC, timers, comm | Integration vs flexibility |
| **Ecosystem** | Tools, libraries, support | Vendor lock-in |
| **Cost** | Unit price, NRE | Features, support |
| **Package** | Pin count, size | I/O count, thermals |

---

## Quick Revision Questions

1. **What are the main differences between a microcontroller (MCU) and a microprocessor (MPU)?**

2. **Explain DMIPS and CoreMark. Why are they better metrics than clock frequency alone?**

3. **List the ARM Cortex-M series processors in order of increasing performance. What are typical applications for each?**

4. **How do you calculate the average power consumption for a duty-cycled embedded system?**

5. **What factors would you consider when selecting between an 8-bit and 32-bit processor for a new design?**

6. **Why is the development ecosystem important in processor selection?**

---

## Navigation

| Previous | Up | Next |
|----------|------|------|
| [Unit 2 Overview](README.md) | [Unit 2 Index](README.md) | [Memory Types and Selection](02-memory-types.md) |

---
