# Unit 2: Embedded Hardware

## Unit Overview

This unit provides a comprehensive understanding of the hardware components that form the foundation of embedded systems. We explore processor architectures, memory systems, input/output devices, sensors, actuators, and power supply considerations that are essential for embedded system design.

---

## Learning Objectives

After completing this unit, you will be able to:

1. Understand processor architectures and selection criteria for embedded applications
2. Identify different memory types and their appropriate applications
3. Analyze I/O device characteristics and interfaces
4. Select appropriate sensors and actuators for various applications
5. Design power supply systems for embedded devices

---

## Chapter Overview

| Chapter | Title | Description |
|---------|-------|-------------|
| 2.1 | [Processor Selection Criteria](01-processor-selection.md) | Choosing the right processor/microcontroller |
| 2.2 | [Memory Types and Selection](02-memory-types.md) | RAM, ROM, Flash, and memory hierarchies |
| 2.3 | [I/O Devices](03-io-devices.md) | Input/Output peripherals and interfaces |
| 2.4 | [Sensors and Actuators](04-sensors-and-actuators.md) | Transducers for real-world interaction |
| 2.5 | [Power Supply Considerations](05-power-supply.md) | Power management and distribution |

---

## Unit Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         EMBEDDED HARDWARE                                │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
            ┌───────────────────────┼───────────────────────┐
            │                       │                       │
            ▼                       ▼                       ▼
    ┌───────────────┐       ┌───────────────┐       ┌───────────────┐
    │   PROCESSING  │       │    MEMORY     │       │    INPUT/     │
    │     UNIT      │       │    SYSTEM     │       │    OUTPUT     │
    │               │       │               │       │               │
    │  • CPU/MCU    │       │  • RAM        │       │  • GPIO       │
    │  • DSP        │       │  • ROM/Flash  │       │  • Timers     │
    │  • FPGA       │       │  • EEPROM     │       │  • ADC/DAC    │
    │  • GPU        │       │  • Cache      │       │  • Comm I/F   │
    └───────────────┘       └───────────────┘       └───────────────┘
            │                       │                       │
            │                       │                       │
            ▼                       ▼                       ▼
    ┌───────────────┐       ┌───────────────┐       ┌───────────────┐
    │   SENSORS     │       │   ACTUATORS   │       │    POWER      │
    │               │       │               │       │   SUPPLY      │
    │  • Temp       │       │  • Motors     │       │  • Regulators │
    │  • Pressure   │       │  • Relays     │       │  • Batteries  │
    │  • Motion     │       │  • Displays   │       │  • PMIC       │
    │  • Light      │       │  • Speakers   │       │  • DC-DC      │
    └───────────────┘       └───────────────┘       └───────────────┘
```

---

## Key Hardware Components

### Processing Unit Types

| Type | Description | Use Case |
|------|-------------|----------|
| **Microcontroller (MCU)** | Integrated CPU + Memory + Peripherals | Simple to medium embedded |
| **Microprocessor (MPU)** | CPU only, external memory/peripherals | Complex embedded, Linux |
| **Digital Signal Processor (DSP)** | Optimized for signal processing | Audio, video, communications |
| **FPGA** | Reconfigurable hardware | Custom logic, prototyping |
| **System-on-Chip (SoC)** | Complete system integration | Smartphones, advanced IoT |

### Memory Hierarchy

```
                    ┌─────────────┐
                    │  Registers  │  ◄── Fastest, smallest
                    │   (Bytes)   │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │    Cache    │
                    │   (KB-MB)   │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  Main Memory│
                    │   (RAM)     │
                    │   (MB-GB)   │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │   Storage   │
                    │ (Flash/ROM) │  ◄── Slowest, largest
                    │   (MB-TB)   │
                    └─────────────┘
```

---

## Hardware Selection Criteria Summary

| Factor | Considerations |
|--------|---------------|
| **Performance** | MIPS, clock speed, architecture |
| **Power** | Active/sleep current, voltage range |
| **Memory** | Flash size, RAM size, external options |
| **Peripherals** | ADC, timers, communication interfaces |
| **Package** | Pin count, form factor, thermal |
| **Cost** | Unit price, volume discounts |
| **Ecosystem** | Tools, libraries, community support |
| **Availability** | Lead time, second sources |

---

## Prerequisites for This Unit

- Understanding of basic electronics (voltage, current, resistance)
- Digital logic fundamentals
- Basic computer architecture concepts

---

## Navigation

| Previous | Up | Next |
|----------|------|------|
| [Unit 1: Introduction](../01-Introduction-to-Embedded-Systems/README.md) | [Main Index](../README.md) | [Unit 3: Embedded Software](../03-Embedded-Software/README.md) |

---
