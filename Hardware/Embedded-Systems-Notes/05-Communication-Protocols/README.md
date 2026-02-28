# Unit 5: Communication Protocols

## Unit Overview

Embedded systems must communicate with sensors, actuators, other microcontrollers, and external systems. This unit covers serial and parallel communication protocols, their timing characteristics, and practical implementation.

---

## Learning Objectives

After completing this unit, you will be able to:

- Understand serial vs parallel communication trade-offs
- Implement UART, SPI, and I2C communication
- Analyze protocol timing diagrams
- Choose appropriate protocols for different applications
- Interface with USB, CAN, and wireless modules

---

## Unit Contents

| Chapter | Topic | Key Concepts |
|---------|-------|--------------|
| 5.1 | [Serial Communication Basics](01-serial-communication-basics.md) | Sync/async, baud rate, data framing |
| 5.2 | [UART Protocol](02-uart-protocol.md) | RS-232, flow control, implementation |
| 5.3 | [SPI Protocol](03-spi-protocol.md) | Master/slave, modes, daisy-chain |
| 5.4 | [I2C Protocol](04-i2c-protocol.md) | Addressing, ACK/NAK, multi-master |
| 5.5 | [Advanced Protocols](05-advanced-protocols.md) | USB, CAN, Ethernet, wireless |

---

## Protocol Comparison Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    COMMUNICATION PROTOCOL COMPARISON                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │ Protocol │ Type   │ Wires │ Speed      │ Distance │ Use Case  │ │
│  ├────────────────────────────────────────────────────────────────┤ │
│  │ UART     │ Async  │ 2-3   │ 115kbps-   │ 15m      │ Debug,    │ │
│  │          │        │       │ 1Mbps      │          │ GPS, BT   │ │
│  ├────────────────────────────────────────────────────────────────┤ │
│  │ SPI      │ Sync   │ 4+    │ 10-50Mbps  │ <1m      │ Displays, │ │
│  │          │        │       │            │          │ SD cards  │ │
│  ├────────────────────────────────────────────────────────────────┤ │
│  │ I2C      │ Sync   │ 2     │ 100k-3.4M  │ <1m      │ Sensors,  │ │
│  │          │        │       │            │          │ EEPROM    │ │
│  ├────────────────────────────────────────────────────────────────┤ │
│  │ CAN      │ Diff.  │ 2     │ 1Mbps      │ 40m-1km  │ Automotive│ │
│  ├────────────────────────────────────────────────────────────────┤ │
│  │ USB      │ Diff.  │ 4     │ 480Mbps    │ 5m       │ PC comm,  │ │
│  │          │        │       │            │          │ storage   │ │
│  ├────────────────────────────────────────────────────────────────┤ │
│  │ Ethernet │ Diff.  │ 4     │ 100Mbps+   │ 100m     │ IoT, net  │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Serial vs Parallel Communication

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SERIAL vs PARALLEL                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PARALLEL COMMUNICATION:                                             │
│  • Multiple bits sent simultaneously                                │
│  • Faster for short distances                                       │
│  • More wires = more cost, crosstalk                               │
│                                                                      │
│     Device A                              Device B                   │
│  ┌──────────┐   D0  ─────────────────   ┌──────────┐               │
│  │          │   D1  ─────────────────   │          │               │
│  │          │   D2  ─────────────────   │          │               │
│  │  DATA    │   D3  ─────────────────   │  DATA    │               │
│  │  PORT    │   D4  ─────────────────   │  PORT    │               │
│  │          │   D5  ─────────────────   │          │               │
│  │          │   D6  ─────────────────   │          │               │
│  │          │   D7  ─────────────────   │          │               │
│  └──────────┘   CLK ─────────────────   └──────────┘               │
│                                                                      │
│  SERIAL COMMUNICATION:                                               │
│  • Bits sent one at a time                                          │
│  • Fewer wires, longer distances                                    │
│  • Higher speed with modern techniques                              │
│                                                                      │
│     Device A                              Device B                   │
│  ┌──────────┐                           ┌──────────┐               │
│  │  SERIAL  │   TX ─────────────────▶   │  SERIAL  │               │
│  │   PORT   │   RX ◀─────────────────   │   PORT   │               │
│  │          │   GND ─────────────────   │          │               │
│  └──────────┘                           └──────────┘               │
│                                                                      │
│  Data: 0x5A (01011010)                                              │
│                                                                      │
│  Serial:  ─┐ ┌─┐ ┌───┐ ┌─┐ ┌─┐                                     │
│            └─┘ └─┘   └─┘ └─┘ └─                                     │
│            0 1 0 1 1 0 1 0                                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Synchronous vs Asynchronous

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SYNC vs ASYNC COMMUNICATION                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ASYNCHRONOUS (UART):                                                │
│  • No shared clock signal                                           │
│  • Sender and receiver must agree on baud rate                      │
│  • Start/stop bits for synchronization                              │
│                                                                      │
│  Data: ──┐ ┌─┐ ┌───┐ ┌─┐ ┌─┐ ┌───────                              │
│          └─┘ └─┘   └─┘ └─┘ └─┘                                      │
│          │S│0│1│0│1│1│0│1│0│P│                                      │
│          Start    8 data bits   Stop                                │
│                                                                      │
│                                                                      │
│  SYNCHRONOUS (SPI, I2C):                                            │
│  • Shared clock signal                                              │
│  • Data sampled on clock edge                                       │
│  • Higher speeds possible                                           │
│                                                                      │
│   CLK: ──┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌──                         │
│          └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘                            │
│           ↑   ↑   ↑   ↑   ↑   ↑   ↑   ↑                             │
│  Data: ═══╪═══╪═══╪═══╪═══╪═══╪═══╪═══╪═══                          │
│          D7  D6  D5  D4  D3  D2  D1  D0                              │
│          Sample on each rising edge                                  │
│                                                                      │
│  COMPARISON:                                                         │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Feature       │ Asynchronous     │ Synchronous             │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │ Clock         │ No (embedded)    │ Yes (separate wire)     │    │
│  │ Overhead      │ ~20% (start/stop)│ Low (no framing)        │    │
│  │ Speed         │ Lower            │ Higher                  │    │
│  │ Wires         │ 2 (TX/RX)        │ 3+ (CLK, data, CS)      │    │
│  │ Complexity    │ Simple           │ More complex            │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Protocol Selection Guide

```
┌─────────────────────────────────────────────────────────────────────┐
│                    WHICH PROTOCOL TO USE?                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Start here:                                                         │
│       │                                                              │
│       ▼                                                              │
│  ┌─────────────────┐    No     ┌─────────────────────────────┐      │
│  │ Need multiple   │─────────▶ │ Single peripheral?          │      │
│  │ devices on bus? │           │ High speed needed?          │      │
│  └────────┬────────┘           └──────────────┬──────────────┘      │
│           │ Yes                               │                      │
│           ▼                                   ▼                      │
│  ┌─────────────────┐           ┌─────────────────────────────┐      │
│  │ Use I2C         │           │ Yes: Use SPI                │      │
│  │ (shared 2 wires)│           │ No:  Use UART               │      │
│  └─────────────────┘           └─────────────────────────────┘      │
│                                                                      │
│                                                                      │
│  DECISION MATRIX:                                                    │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │ Requirement              │ Best Protocol                      │ │
│  ├────────────────────────────────────────────────────────────────┤ │
│  │ Debug console            │ UART                               │ │
│  │ GPS module               │ UART                               │ │
│  │ LCD display (fast)       │ SPI                                │ │
│  │ SD card                  │ SPI (or SDIO)                      │ │
│  │ Multiple sensors         │ I2C                                │ │
│  │ EEPROM                   │ I2C or SPI                         │ │
│  │ Automotive network       │ CAN                                │ │
│  │ PC connectivity          │ USB or UART                        │ │
│  │ IoT connectivity         │ Ethernet, WiFi, BLE                │ │
│  │ Long distance (>15m)     │ RS-485, CAN                        │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Key Concepts Preview

| Concept | Description |
|---------|-------------|
| **Baud Rate** | Symbols per second (UART) |
| **Bit Rate** | Actual data bits per second |
| **Full Duplex** | Simultaneous bidirectional |
| **Half Duplex** | Bidirectional, one at a time |
| **Master/Slave** | Controller initiates transfers |
| **Multi-Master** | Multiple controllers (I2C) |
| **Pull-up/Pull-down** | Resistors for line states |
| **Open Drain** | Output can sink but not source |

---

## Navigation

| Previous Unit | Course Index | Next Unit |
|---------------|--------------|-----------|
| [Unit 4: RTOS](../04-RTOS/README.md) | [Main Index](../README.md) | [Unit 6: Interfacing](../06-Interfacing/README.md) |

---
