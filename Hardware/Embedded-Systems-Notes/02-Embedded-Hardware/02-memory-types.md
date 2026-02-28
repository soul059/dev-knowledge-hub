# Chapter 2.2: Memory Types and Selection

## Chapter Overview

Memory is a fundamental component of embedded systems, storing both program code and data. This chapter explores the various types of memory used in embedded systems, their characteristics, organization, and selection criteria for different applications.

---

## 2.2.1 Memory Classification

### Memory Taxonomy

```
┌─────────────────────────────────────────────────────────────────────┐
│                       MEMORY CLASSIFICATION                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                          ┌─────────┐                                │
│                          │ MEMORY  │                                │
│                          └────┬────┘                                │
│                               │                                      │
│              ┌────────────────┴────────────────┐                    │
│              │                                 │                    │
│              ▼                                 ▼                    │
│     ┌─────────────────┐              ┌─────────────────┐           │
│     │    VOLATILE     │              │  NON-VOLATILE   │           │
│     │ (Loses data     │              │ (Retains data   │           │
│     │  when power off)│              │  without power) │           │
│     └────────┬────────┘              └────────┬────────┘           │
│              │                                │                     │
│     ┌────────┴────────┐             ┌────────┴────────┐            │
│     │                 │             │                 │            │
│     ▼                 ▼             ▼                 ▼            │
│  ┌──────┐        ┌──────┐      ┌──────┐          ┌──────┐         │
│  │ SRAM │        │ DRAM │      │ ROM  │          │ Flash│         │
│  │      │        │      │      │      │          │      │         │
│  │Static│        │Dynamic│     │Read  │          │Electr│         │
│  │Random│        │Random │     │Only  │          │Erase │         │
│  │Access│        │Access │     │Memory│          │      │         │
│  └──────┘        └──────┘      └──┬───┘          └──────┘         │
│                                   │                                 │
│                    ┌──────────────┼──────────────┐                 │
│                    │              │              │                 │
│                    ▼              ▼              ▼                 │
│                ┌──────┐      ┌──────┐       ┌──────┐              │
│                │Mask  │      │ PROM │       │EEPROM│              │
│                │ ROM  │      │      │       │      │              │
│                └──────┘      └──────┘       └──────┘              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2.2.2 Volatile Memory

### SRAM (Static RAM)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SRAM (STATIC RAM)                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  6-TRANSISTOR SRAM CELL:                                            │
│                                                                      │
│             VDD                      VDD                            │
│              │                        │                             │
│          ┌───┴───┐              ┌─────┴─────┐                      │
│          │       │              │           │                      │
│        ─┤├─    ─┤├─          ─┤├─        ─┤├─                    │
│          │       │              │           │                      │
│          ├───────┼──────────────┼───────────┤                      │
│          │       │      Q       │    Q̄      │                      │
│          │       └──────┬───────┘           │                      │
│          │              │                   │                      │
│        ─┤├─           ─┤├─               ─┤├─                    │
│          │              │                   │                      │
│          └──────────────┼───────────────────┘                      │
│                         │                                           │
│                        GND                                          │
│                                                                      │
│  CHARACTERISTICS:                                                    │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  • Fast access: 1-10 ns                                    │     │
│  │  • No refresh required (static)                            │     │
│  │  • Low power (only during access)                          │     │
│  │  • 6 transistors per bit (larger cell)                     │     │
│  │  • More expensive per bit than DRAM                        │     │
│  │  • Used for: Cache, registers, MCU internal RAM            │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  TIMING DIAGRAM:                                                     │
│                                                                      │
│  CLK     ─┐  ┌──┐  ┌──┐  ┌──┐  ┌──                                 │
│           └──┘  └──┘  └──┘  └──┘                                    │
│  ADDR    ─────┐     ┌─────────────                                  │
│               └─────┘ Valid Address                                 │
│  DATA    ───────────┐     ┌───────                                  │
│                     └─────┘ Valid Data                              │
│                    │◄────►│                                         │
│                    Access Time                                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### DRAM (Dynamic RAM)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DRAM (DYNAMIC RAM)                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1-TRANSISTOR DRAM CELL:                                            │
│                                                                      │
│      Word Line                                                       │
│          │                                                           │
│         ─┼─                                                         │
│        ─┤├─ Access Transistor                                      │
│          │                                                           │
│     ─────┼───── Bit Line                                            │
│          │                                                           │
│         ─┴─                                                         │
│         ─┬─ Storage Capacitor                                       │
│          │                                                           │
│         ─┴─                                                         │
│         GND                                                          │
│                                                                      │
│  CHARACTERISTICS:                                                    │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  • Higher density (1T + 1C per bit)                        │     │
│  │  • Requires periodic refresh (every ~64ms)                 │     │
│  │  • Slower than SRAM: 10-100 ns                             │     │
│  │  • Lower cost per bit                                      │     │
│  │  • Types: SDRAM, DDR, DDR2, DDR3, DDR4, DDR5, LPDDR       │     │
│  │  • Used for: Main memory in MPU-based systems              │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  REFRESH CYCLE:                                                      │
│                                                                      │
│  Charge │    ┌───────┐       ┌───────┐       ┌───────                │
│  Level  │   /         \     /         \     /                        │
│         │  /           \   /           \   /                         │
│         │ /             \ /             \ /                          │
│         └─────────────────────────────────────────►                 │
│              Refresh    Refresh    Refresh   Time                   │
│              (64ms interval for all rows)                           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### SRAM vs DRAM Comparison

| Feature | SRAM | DRAM |
|---------|------|------|
| **Cell Size** | 6 transistors | 1T + 1C |
| **Density** | Lower | Higher |
| **Speed** | 1-10 ns | 10-100 ns |
| **Refresh** | Not needed | Required (64ms) |
| **Power (idle)** | Low | Higher (refresh) |
| **Cost/bit** | Higher | Lower |
| **Typical Use** | Cache, MCU RAM | Main memory (MPU) |

---

## 2.2.3 Non-Volatile Memory

### ROM Family

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ROM FAMILY COMPARISON                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  MASK ROM:                                                           │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  • Programmed during manufacturing                          │     │
│  │  • Lowest cost at high volume                               │     │
│  │  • Cannot be modified after production                      │     │
│  │  • Used for: Mass production, mature products               │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  PROM (Programmable ROM):                                            │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  • One-time programmable (OTP)                              │     │
│  │  • Fuse-based (blown fuses = 0, intact = 1)                 │     │
│  │  • Low cost, no erasing                                     │     │
│  │  • Used for: Small production, prototyping                  │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  EPROM (Erasable PROM):                                              │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  • UV light erasable (15-20 minutes)                        │     │
│  │  • Quartz window for UV exposure                            │     │
│  │  • Reusable but inconvenient                                │     │
│  │  • Mostly obsolete now                                      │     │
│  │  ┌─────────────────────┐                                   │     │
│  │  │   ┌───────────┐     │ ← Quartz window                   │     │
│  │  │   │           │     │                                   │     │
│  │  │   │   EPROM   │     │                                   │     │
│  │  │   │    Die    │     │                                   │     │
│  │  └───┴───────────┴─────┘                                   │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  EEPROM (Electrically Erasable PROM):                                │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  • Byte-level erase/write                                   │     │
│  │  • Limited endurance: 100,000 - 1,000,000 cycles           │     │
│  │  • Slow write: 5-10 ms per byte                             │     │
│  │  • Used for: Configuration, calibration data               │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Flash Memory

```
┌─────────────────────────────────────────────────────────────────────┐
│                    FLASH MEMORY                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  FLOATING GATE TRANSISTOR:                                           │
│                                                                      │
│          Control Gate                                                │
│              │                                                       │
│         ═════════════ ← Control Gate                                │
│         ░░░░░░░░░░░░░ ← Oxide                                       │
│         ▓▓▓▓▓▓▓▓▓▓▓▓▓ ← Floating Gate (stores electrons)           │
│         ░░░░░░░░░░░░░ ← Tunnel Oxide                                │
│        ┌─────────────┐                                               │
│     ───┤   Source    ├─── Drain                                     │
│        └─────────────┘                                               │
│              │                                                       │
│          Substrate                                                   │
│                                                                      │
│  TYPES OF FLASH:                                                     │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │                                                            │     │
│  │  NOR FLASH:                     NAND FLASH:                │     │
│  │  ┌────────────────────┐        ┌────────────────────┐     │     │
│  │  │ • Random access    │        │ • Sequential access│     │     │
│  │  │ • Execute-in-place │        │ • Higher density   │     │     │
│  │  │ • Fast read        │        │ • Faster write     │     │     │
│  │  │ • Slower write     │        │ • Lower cost/bit   │     │     │
│  │  │ • Lower density    │        │ • Block erase only │     │     │
│  │  │ • Code storage     │        │ • Data storage     │     │     │
│  │  └────────────────────┘        └────────────────────┘     │     │
│  │                                                            │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  FLASH ARCHITECTURE:                                                 │
│                                                                      │
│  NOR (Parallel):          NAND (Series):                            │
│       BL                       BL                                    │
│       │                        │                                     │
│  WL0 ─┼─                  WL0 ─┼─                                   │
│       │                        │                                     │
│  WL1 ─┼─                  WL1 ─┼─                                   │
│       │                        │                                     │
│  WL2 ─┼─                  WL2 ─┼─                                   │
│       │                        │                                     │
│      GND                      GND                                    │
│                                                                      │
│  Each cell readable       String of cells,                          │
│  independently            read sequentially                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Flash Memory Specifications

| Parameter | NOR Flash | NAND Flash |
|-----------|-----------|------------|
| **Read Access** | Random, ~100 ns | Sequential, ~25 µs (page) |
| **Write Speed** | ~1 MB/s | ~10-40 MB/s |
| **Erase** | Sector (4-256 KB) | Block (128-512 KB) |
| **Endurance** | 10,000-100,000 cycles | 1,000-100,000 cycles |
| **Density** | Up to 256 MB typical | Up to TB range |
| **Cost/bit** | Higher | Lower |
| **XIP** | Yes | No (needs controller) |
| **Use Case** | Code execution | Data storage, SSD |

---

## 2.2.4 Memory Hierarchy in Embedded Systems

### Memory Organization

```
┌─────────────────────────────────────────────────────────────────────┐
│                    EMBEDDED MEMORY HIERARCHY                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SPEED          SIZE           TYPE           TYPICAL VALUES        │
│    ▲              │                                                  │
│    │              │                                                  │
│    │  ┌───────────┴───────────┐                                     │
│    │  │      REGISTERS        │  32-256 bytes    ~1 cycle          │
│    │  │   (CPU internal)      │                                     │
│    │  └───────────────────────┘                                     │
│    │              │                                                  │
│    │  ┌───────────┴───────────┐                                     │
│    │  │   L1 CACHE (optional) │  0-64 KB         1-3 cycles        │
│    │  │   (Cortex-M7 only)    │                                     │
│    │  └───────────────────────┘                                     │
│    │              │                                                  │
│    │  ┌───────────┴───────────┐                                     │
│    │  │   TIGHTLY-COUPLED     │  16-256 KB       1 cycle           │
│    │  │      MEMORY (TCM)     │  (SRAM)                            │
│    │  └───────────────────────┘                                     │
│    │              │                                                  │
│    │  ┌───────────┴───────────┐                                     │
│    │  │    INTERNAL SRAM      │  2 KB - 1 MB     1-2 cycles        │
│    │  │    (Main RAM)         │                                     │
│    │  └───────────────────────┘                                     │
│    │              │                                                  │
│    │  ┌───────────┴───────────┐                                     │
│    │  │   INTERNAL FLASH      │  16 KB - 2 MB    1-3 wait states   │
│    │  │    (Program)          │                                     │
│    │  └───────────────────────┘                                     │
│    │              │                                                  │
│    │  ┌───────────┴───────────┐                                     │
│    │  │   EXTERNAL MEMORY     │  MB - GB         10+ cycles        │
│    │  │  (SDRAM, Flash)       │                                     │
│    ▼  └───────────────────────┘                                     │
│  SLOW           LARGE                                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### MCU Memory Map Example (STM32F4)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    STM32F4 MEMORY MAP                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Address Range        │ Region              │ Size                  │
│  ─────────────────────────────────────────────────────────────────  │
│  0xFFFF_FFFF ─────────┤                     │                       │
│                       │ Cortex-M4 Internal  │ 0.5 GB                │
│  0xE000_0000 ─────────┤ (SysTick, NVIC)     │                       │
│                       │                     │                       │
│  0xA000_0000 ─────────┤ External Device     │ 0.5 GB                │
│                       │                     │                       │
│  0x6000_0000 ─────────┤ External RAM        │ 0.5 GB                │
│                       │ (FSMC/FMC)          │                       │
│  0x4000_0000 ─────────┤ Peripherals         │ 0.5 GB                │
│                       │ (APB1, APB2, AHB)   │                       │
│  0x2002_0000 ─────────┤                     │                       │
│                       │ SRAM2               │ 16 KB                 │
│  0x2001_C000 ─────────┤                     │                       │
│                       │ SRAM1               │ 112 KB                │
│  0x2000_0000 ─────────┤                     │                       │
│                       │ Reserved            │                       │
│  0x1FFF_C000 ─────────┤                     │                       │
│                       │ System Memory       │ 30 KB                 │
│  0x1FFF_0000 ─────────┤ (Bootloader)        │                       │
│                       │                     │                       │
│  0x0810_0000 ─────────┤ Flash Memory        │                       │
│                       │ (User Code)         │ 1 MB                  │
│  0x0800_0000 ─────────┤                     │                       │
│                       │                     │                       │
│  0x0000_0000 ─────────┤ Aliased to Flash    │ (boot region)         │
│                       │ or System Memory    │                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2.2.5 Memory Interface and Timing

### Parallel Memory Interface

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PARALLEL MEMORY INTERFACE                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│       MCU/CPU                              MEMORY                    │
│  ┌──────────────┐                    ┌──────────────┐               │
│  │              │  Address Bus       │              │               │
│  │              │  A[0:n] ──────────►│              │               │
│  │              │  (16-32 bits)      │              │               │
│  │              │                    │              │               │
│  │              │  Data Bus          │              │               │
│  │              │  D[0:n] ◄─────────►│              │               │
│  │              │  (8-32 bits)       │              │               │
│  │              │                    │              │               │
│  │              │  Control Signals   │              │               │
│  │              │  CS ──────────────►│              │               │
│  │              │  OE ──────────────►│              │               │
│  │              │  WE ──────────────►│              │               │
│  │              │                    │              │               │
│  └──────────────┘                    └──────────────┘               │
│                                                                      │
│  READ TIMING:                                                        │
│                                                                      │
│  CLK     ──┐  ┌──┐  ┌──┐  ┌──┐  ┌──                                │
│            └──┘  └──┘  └──┘  └──┘                                   │
│                                                                      │
│  ADDR    ────┐        ┌──────────────                               │
│              └────────┘ Valid                                        │
│                                                                      │
│  CS      ────┐        ┌──────────────                               │
│              └────────┘                                              │
│                                                                      │
│  OE      ──────┐      ┌──────────────                               │
│                └──────┘                                              │
│                                                                      │
│  DATA    ─────────────┐     ┌────────                               │
│                       └─────┘ Valid Data                            │
│                       │◄───►│                                       │
│                       Access Time                                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Serial Memory Interfaces

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SERIAL MEMORY INTERFACES                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SPI FLASH:                                                          │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │                                                            │     │
│  │    MCU                               SPI Flash             │     │
│  │  ┌──────┐                          ┌──────────┐           │     │
│  │  │      │ SCK  ──────────────────► │          │           │     │
│  │  │      │ MOSI ──────────────────► │          │           │     │
│  │  │      │ MISO ◄────────────────── │          │           │     │
│  │  │      │ CS   ──────────────────► │          │           │     │
│  │  └──────┘                          └──────────┘           │     │
│  │                                                            │     │
│  │  Standard SPI: 1-bit data, up to 50 MHz                    │     │
│  │  Dual SPI: 2-bit data                                      │     │
│  │  Quad SPI (QSPI): 4-bit data, up to 100+ MHz               │     │
│  │  Octal SPI: 8-bit data                                     │     │
│  │                                                            │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  I2C EEPROM:                                                         │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │                                                            │     │
│  │    MCU                               I2C EEPROM            │     │
│  │  ┌──────┐      ┌─────────────────┐  ┌──────────┐          │     │
│  │  │      │ SDA ─┤◄───────────────►├──┤          │          │     │
│  │  │      │ SCL ─┤─────────────────┼──┤          │          │     │
│  │  └──────┘      │     Pull-ups    │  └──────────┘          │     │
│  │                │       VCC       │                        │     │
│  │                └────────┬────────┘                        │     │
│  │                         │                                 │     │
│  │  Speed: 100 kHz, 400 kHz, 1 MHz, 3.4 MHz                 │     │
│  │  Multi-device on same bus (using addresses)              │     │
│  │                                                            │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2.2.6 Memory Selection Criteria

### Selection Decision Matrix

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MEMORY SELECTION CRITERIA                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  APPLICATION REQUIREMENTS → MEMORY TYPE SELECTION                    │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │                                                            │     │
│  │  Requirement              │ Best Memory Type               │     │
│  │  ─────────────────────────────────────────────────────────│     │
│  │  Program storage          │ Internal Flash (NOR)           │     │
│  │  Execution-in-place       │ NOR Flash                      │     │
│  │  Variable storage         │ SRAM                           │     │
│  │  Large data storage       │ NAND Flash, SD Card            │     │
│  │  Configuration data       │ EEPROM, Flash (data section)   │     │
│  │  High-speed buffer        │ SRAM, TCM                      │     │
│  │  Large RAM (Linux)        │ DDR SDRAM                      │     │
│  │  Battery-backed storage   │ FRAM, MRAM                     │     │
│  │  Frequent write cycles    │ FRAM (10^15 cycles)            │     │
│  │                                                            │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  SIZE ESTIMATION:                                                    │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │                                                            │     │
│  │  Code Size = Source Lines × ~10-50 bytes/line             │     │
│  │  RAM Size  = Stack + Heap + Static + Buffer               │     │
│  │                                                            │     │
│  │  Example:                                                   │     │
│  │  • 5000 lines C code → ~100 KB Flash                       │     │
│  │  • Stack: 4 KB, Heap: 8 KB, Static: 2 KB, Buffers: 10 KB  │     │
│  │  • RAM needed: ~24 KB                                      │     │
│  │  • Add 30-50% margin → 150 KB Flash, 32 KB RAM            │     │
│  │                                                            │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Memory Technology Comparison

| Technology | Density | Speed | Endurance | Power | Cost | Use Case |
|------------|---------|-------|-----------|-------|------|----------|
| **SRAM** | Low | Fastest | ∞ | Low (active) | High | Cache, MCU RAM |
| **DRAM** | High | Fast | ∞ | Medium | Low | Main memory |
| **NOR Flash** | Medium | Fast read | 100K | Low | Medium | Code storage |
| **NAND Flash** | Very High | Medium | 10K-100K | Low | Very Low | Mass storage |
| **EEPROM** | Low | Slow | 100K-1M | Low | Medium | Parameters |
| **FRAM** | Low | Fast | 10^15 | Very Low | High | Non-volatile RAM |
| **MRAM** | Low | Fast | 10^15 | Low | High | NV Cache |

---

## 2.2.7 Special Memory Technologies

### Emerging Non-Volatile Memories

```
┌─────────────────────────────────────────────────────────────────────┐
│                    EMERGING MEMORY TECHNOLOGIES                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  FRAM (Ferroelectric RAM):                                          │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  • Uses ferroelectric material instead of dielectric       │     │
│  │  • Non-volatile, fast like SRAM                            │     │
│  │  • Virtually unlimited endurance (10^15 cycles)            │     │
│  │  • Low power, no wear leveling needed                      │     │
│  │  • Higher cost, lower density than Flash                   │     │
│  │  • Vendors: TI (MSP430FR), Fujitsu, Cypress                │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  MRAM (Magnetoresistive RAM):                                        │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  • Uses magnetic tunnel junctions                          │     │
│  │  • Non-volatile, fast, unlimited endurance                 │     │
│  │  • Radiation tolerant                                      │     │
│  │  • Used in automotive, aerospace                           │     │
│  │  • Vendors: Everspin, Avalanche Technology                 │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  ReRAM (Resistive RAM):                                              │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  • Changes resistance of dielectric material               │     │
│  │  • Very high density potential                             │     │
│  │  • Fast switching, low power                               │     │
│  │  • Still emerging technology                               │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  COMPARISON WITH CONVENTIONAL:                                       │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │            │ Flash   │ FRAM    │ MRAM    │ SRAM    │      │     │
│  │  ──────────────────────────────────────────────────────── │     │
│  │  Speed     │ Slow W  │ Fast    │ Fast    │ Fastest │      │     │
│  │  Endurance │ 10^5    │ 10^15   │ 10^15   │ ∞       │      │     │
│  │  Density   │ High    │ Low     │ Low     │ Low     │      │     │
│  │  NV?       │ Yes     │ Yes     │ Yes     │ No      │      │     │
│  │  Cost      │ Low     │ High    │ High    │ Medium  │      │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2.2.8 Memory Protection and Management

### Memory Protection Unit (MPU)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MEMORY PROTECTION UNIT (MPU)                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PURPOSE:                                                            │
│  • Prevent code from accessing unauthorized memory regions          │
│  • Separate stack from heap (prevent stack overflow corruption)     │
│  • Protect OS from application code                                 │
│  • Enable sandboxing of tasks in RTOS                               │
│                                                                      │
│  MPU REGION CONFIGURATION:                                           │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                                                              │   │
│  │  Region │ Start Addr │ Size  │ Access      │ Permissions   │   │
│  │  ───────────────────────────────────────────────────────── │   │
│  │    0    │ 0x08000000 │ 256KB │ Flash       │ RO, Execute   │   │
│  │    1    │ 0x20000000 │ 32KB  │ RAM         │ RW, No-Execute│   │
│  │    2    │ 0x20008000 │ 4KB   │ Stack       │ RW, No-Execute│   │
│  │    3    │ 0x40000000 │ 512MB │ Peripherals │ RW, No-Execute│   │
│  │    4    │ 0x60000000 │ 1MB   │ External    │ RW, Execute   │   │
│  │                                                              │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  MEMORY MAP WITH MPU:                                                │
│                                                                      │
│  ┌──────────────────────────────┐                                   │
│  │         FLASH                │ ← Execute, Read-only              │
│  │    (Program Code)            │   Region 0                        │
│  ├──────────────────────────────┤                                   │
│  │         RAM                  │ ← Read-Write, No-Execute          │
│  │    (Variables)               │   Region 1                        │
│  ├──────────────────────────────┤                                   │
│  │         STACK                │ ← Read-Write, Protected           │
│  │    (with guard band)         │   Region 2                        │
│  ├──────────────────────────────┤                                   │
│  │      PERIPHERALS             │ ← Read-Write, Device              │
│  │    (Special access)          │   Region 3                        │
│  └──────────────────────────────┘                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Memory Type | Volatile | Speed | Density | Endurance | Typical Use |
|-------------|----------|-------|---------|-----------|-------------|
| **SRAM** | Yes | Fastest | Low | Infinite | MCU RAM, cache |
| **DRAM** | Yes | Fast | High | Infinite | Main memory |
| **Mask ROM** | No | Fast read | High | N/A | Mass production |
| **EEPROM** | No | Slow write | Low | 100K-1M | Configuration |
| **NOR Flash** | No | Fast read | Medium | 10K-100K | Code storage |
| **NAND Flash** | No | Sequential | Very high | 1K-100K | Data storage |
| **FRAM** | No | Fast | Low | 10^15 | NV RAM |
| **MRAM** | No | Fast | Low | 10^15 | Critical data |

---

## Quick Revision Questions

1. **What is the fundamental difference between SRAM and DRAM? Why does DRAM need refresh?**

2. **Compare NOR Flash and NAND Flash. When would you use each?**

3. **What is Execute-in-Place (XIP)? Which memory types support it?**

4. **Explain the concept of Flash memory endurance. Why is wear leveling important?**

5. **Draw a typical memory map for an ARM Cortex-M microcontroller and explain each region.**

6. **What is FRAM and what advantages does it offer over traditional Flash memory?**

---

## Navigation

| Previous | Up | Next |
|----------|------|------|
| [Processor Selection](01-processor-selection.md) | [Unit 2 Index](README.md) | [I/O Devices](03-io-devices.md) |

---
