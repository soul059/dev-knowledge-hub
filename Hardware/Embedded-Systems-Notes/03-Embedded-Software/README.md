# Unit 3: Embedded Software

## Unit Overview

This unit covers the software aspects of embedded systems, including programming languages, development tools, firmware architecture, device drivers, and the boot process. Understanding embedded software development is crucial for creating reliable, efficient, and maintainable embedded systems.

---

## Learning Objectives

After completing this unit, you will be able to:

- Write efficient embedded C code with hardware awareness
- Understand cross-compilation and toolchain concepts
- Design modular firmware architectures
- Develop device drivers for common peripherals
- Understand the boot sequence from power-on to application

---

## Unit Contents

| Chapter | Topic | Key Concepts |
|---------|-------|--------------|
| 3.1 | [Embedded C Programming](01-embedded-c-programming.md) | C extensions, volatile, bit manipulation, memory-mapped I/O |
| 3.2 | [Cross-Compilation and Toolchains](02-cross-compilation.md) | GCC, linker scripts, startup code, debugging |
| 3.3 | [Firmware Architecture](03-firmware-architecture.md) | Main loop, state machines, layered design, HAL |
| 3.4 | [Device Drivers](04-device-drivers.md) | Driver models, abstraction, interrupt handling |
| 3.5 | [Boot Sequence](05-boot-sequence.md) | Reset vector, startup code, bootloaders |

---

## Embedded Software Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                 EMBEDDED SOFTWARE STACK                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    APPLICATION LAYER                         │    │
│  │  • Main application logic                                   │    │
│  │  • State machines, control algorithms                       │    │
│  │  • User interface handling                                  │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    MIDDLEWARE LAYER                          │    │
│  │  • Protocol stacks (TCP/IP, USB, CAN)                       │    │
│  │  • File systems, databases                                  │    │
│  │  • Graphics libraries                                       │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                RTOS / BARE-METAL SERVICES                    │    │
│  │  • Task scheduling                                          │    │
│  │  • Memory management                                        │    │
│  │  • Inter-task communication                                 │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              HARDWARE ABSTRACTION LAYER (HAL)                │    │
│  │  • Peripheral APIs                                          │    │
│  │  • Board Support Package (BSP)                              │    │
│  │  • Device drivers                                           │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    HARDWARE LAYER                            │    │
│  │  • Registers, Peripherals, Memory                           │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Development Environment

```
┌─────────────────────────────────────────────────────────────────────┐
│              EMBEDDED DEVELOPMENT WORKFLOW                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  HOST PC (Development)                TARGET (Embedded Device)       │
│  ┌──────────────────────┐            ┌──────────────────────┐       │
│  │                      │            │                      │       │
│  │  Source Code (.c/.h) │            │   Flash Memory       │       │
│  │        │             │            │   ┌──────────────┐   │       │
│  │        ▼             │            │   │   Firmware   │   │       │
│  │  ┌───────────────┐   │   JTAG/    │   │    (.bin)    │   │       │
│  │  │ Cross-Compiler│   │   SWD      │   └──────────────┘   │       │
│  │  │   (arm-gcc)   │   │  ─────────►│                      │       │
│  │  └───────┬───────┘   │            │   ┌──────────────┐   │       │
│  │          │           │            │   │     MCU      │   │       │
│  │          ▼           │            │   │   (Running)  │   │       │
│  │  ┌───────────────┐   │  Debug     │   └──────────────┘   │       │
│  │  │    Linker     │   │  Probe     │                      │       │
│  │  └───────┬───────┘   │  ◄────────►│   Debug Interface    │       │
│  │          │           │  (GDB)     │                      │       │
│  │          ▼           │            │                      │       │
│  │  Binary (.elf/.bin)  │            │                      │       │
│  │                      │            │                      │       │
│  └──────────────────────┘            └──────────────────────┘       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Key Concepts Preview

### Software Development Concepts

| Concept | Description |
|---------|-------------|
| **Cross-Compilation** | Compiling code on host for different target architecture |
| **Volatile** | Keyword preventing compiler optimization for hardware registers |
| **Memory-Mapped I/O** | Accessing peripherals through memory addresses |
| **Interrupt Service Routine** | Function called in response to hardware interrupt |
| **HAL (Hardware Abstraction)** | Layer isolating application from hardware specifics |
| **Bootloader** | Code that initializes system and loads main application |
| **Linker Script** | Defines memory layout for compiled code |

---

## Navigation

| Previous Unit | Course Index | Next Unit |
|---------------|--------------|-----------|
| [Unit 2: Embedded Hardware](../02-Embedded-Hardware/README.md) | [Main Index](../README.md) | [Unit 4: RTOS](../04-RTOS/README.md) |

---
