# Chapter 5.5: Advanced Communication Protocols

## Introduction

This chapter covers advanced protocols used in embedded systems: CAN (automotive), USB (universal connectivity), and an overview of wireless protocols. These provide higher data rates, longer distances, or wireless capability.

---

## CAN Protocol (Controller Area Network)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CAN BUS OVERVIEW                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  CAN: Robust multi-master protocol for automotive and industrial   │
│                                                                      │
│                    120Ω                           120Ω              │
│                  ┌─────┐                        ┌─────┐             │
│  ────────────────┤     ├────────────────────────┤     ├──────       │
│  CAN_H           └──┬──┘         BUS            └──┬──┘             │
│                     │                               │                │
│  ────────────────┬──┴──┬────────────────────────┬──┴──┬──────       │
│  CAN_L           │     │                        │     │             │
│                  │     │                        │     │             │
│              ┌───┴─────┴───┐              ┌─────┴─────┴───┐         │
│              │   NODE 1    │              │    NODE 2     │         │
│              │ Transceiver │              │  Transceiver  │         │
│              │     │       │              │      │        │         │
│              │ Controller  │              │  Controller   │         │
│              │     │       │              │      │        │         │
│              │    MCU      │              │     MCU       │         │
│              └─────────────┘              └───────────────┘         │
│                                                                      │
│  Key Features:                                                       │
│  • Differential signaling (noise immune)                            │
│  • Message-based (no addresses)                                     │
│  • Priority-based arbitration                                       │
│  • Error detection and fault confinement                           │
│  • Up to 1 Mbps, up to 40m                                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### CAN Signal Levels

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CAN DIFFERENTIAL SIGNALING                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Voltage                                                             │
│    5V ─┬─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─                  │
│        │                                                             │
│  3.5V ─┼─ ─ ─ ─ ┌────────┐ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  CAN_H          │
│        │        │        │                                          │
│  2.5V ─┼════════╪════════╪═════════════════════════  Recessive     │
│        │        │        │                                          │
│  1.5V ─┼─ ─ ─ ─ └────────┘ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  CAN_L          │
│        │                                                             │
│    0V ─┴─────────────────────────────────────────                   │
│        │ Recessive│Dominant│ Recessive                              │
│        │  (1)     │  (0)   │   (1)                                  │
│                                                                      │
│  RECESSIVE (Logic 1): CAN_H = CAN_L = 2.5V (differential = 0V)     │
│  DOMINANT (Logic 0):  CAN_H = 3.5V, CAN_L = 1.5V (diff = 2V)       │
│                                                                      │
│  Dominant overwrites recessive - enables arbitration               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### CAN Frame Format

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CAN 2.0A STANDARD FRAME                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌───┬─────────────┬─┬─┬─┬────┬─────────────────┬───────┬─┬───┬─┐  │
│  │SOF│ Identifier  │R│I│r│DLC │     DATA        │  CRC  │A│EOF│I│  │
│  │   │  (11 bits)  │T│D│0│    │   (0-8 bytes)   │(15+1) │C│   │F│  │
│  │   │             │R│E│ │    │                 │       │K│   │S│  │
│  └───┴─────────────┴─┴─┴─┴────┴─────────────────┴───────┴─┴───┴─┘  │
│   │        │        │ │ │  │          │            │     │  │   │   │
│   │        │        │ │ │  │          │            │     │  │   │   │
│   │        │        │ │ │  │          │            │     │  │   └─ Interframe Space
│   │        │        │ │ │  │          │            │     │  └─ End of Frame (7 bits)
│   │        │        │ │ │  │          │            │     └─ ACK slot + delimiter
│   │        │        │ │ │  │          │            └─ CRC sequence + delimiter
│   │        │        │ │ │  │          └─ 0 to 8 data bytes
│   │        │        │ │ │  └─ Data Length Code (4 bits)
│   │        │        │ │ └─ Reserved bit (dominant)
│   │        │        │ └─ IDE (Identifier Extension) = 0 for std
│   │        │        └─ RTR (Remote Transmission Request)
│   │        └─ Message identifier (priority)
│   └─ Start of Frame (dominant)                                      │
│                                                                      │
│  CAN 2.0B EXTENDED FRAME:                                           │
│  Uses 29-bit identifier (11 + 18 bits)                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### CAN Arbitration

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CAN ARBITRATION                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Lower identifier = Higher priority                                 │
│                                                                      │
│  Node A sends ID: 0x201 = 010 0000 0001                            │
│  Node B sends ID: 0x200 = 010 0000 0000                            │
│                                                                      │
│  Bit position:     10  9  8  7  6  5  4  3  2  1  0                │
│                                                                      │
│  Node A:           0   1  0  0  0  0  0  0  0  0  1                │
│  Node B:           0   1  0  0  0  0  0  0  0  0  0                │
│  Bus:              0   1  0  0  0  0  0  0  0  0  0                │
│                                                      ↑              │
│                          Node A sends 1 (recessive)  │              │
│                          Node B sends 0 (dominant)   │              │
│                          Bus = 0 (dominant wins)     │              │
│                                                                      │
│  Node A reads back 0, expected 1 → loses arbitration               │
│  Node B continues transmission                                      │
│  Node A retries after bus is free                                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### CAN Implementation

```c
// CAN initialization (STM32)
void can_init(void) {
    // Enable CAN clock
    RCC->APB1ENR |= RCC_APB1ENR_CAN1EN;
    
    // Enter initialization mode
    CAN1->MCR |= CAN_MCR_INRQ;
    while (!(CAN1->MSR & CAN_MSR_INAK));
    
    // Configure bit timing for 500 kbps @ 36 MHz APB1
    // Prescaler = 4, BS1 = 15, BS2 = 2 → 18 TQ
    CAN1->BTR = (3 << 0) |    // BRP = 4-1
                (14 << 16) |   // TS1 = 15-1
                (1 << 20);     // TS2 = 2-1
    
    // Leave initialization mode
    CAN1->MCR &= ~CAN_MCR_INRQ;
    while (CAN1->MSR & CAN_MSR_INAK);
}

// Send CAN message
bool can_send(uint32_t id, uint8_t *data, uint8_t len) {
    // Wait for empty mailbox
    uint32_t timeout = 10000;
    while (!(CAN1->TSR & CAN_TSR_TME0) && timeout--);
    if (timeout == 0) return false;
    
    // Set ID (standard 11-bit)
    CAN1->sTxMailBox[0].TIR = (id << 21);
    
    // Set data length
    CAN1->sTxMailBox[0].TDTR = len & 0x0F;
    
    // Set data
    CAN1->sTxMailBox[0].TDLR = 
        data[0] | (data[1] << 8) | (data[2] << 16) | (data[3] << 24);
    CAN1->sTxMailBox[0].TDHR = 
        data[4] | (data[5] << 8) | (data[6] << 16) | (data[7] << 24);
    
    // Request transmission
    CAN1->sTxMailBox[0].TIR |= CAN_TI0R_TXRQ;
    
    return true;
}
```

---

## USB Protocol

```
┌─────────────────────────────────────────────────────────────────────┐
│                    USB OVERVIEW                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  USB = Universal Serial Bus                                         │
│                                                                      │
│  TOPOLOGY (Tiered Star):                                            │
│                                                                      │
│                    ┌──────────┐                                     │
│                    │   HOST   │  (PC, MCU)                          │
│                    │   (Root) │                                     │
│                    └────┬─────┘                                     │
│                         │                                           │
│                    ┌────┴─────┐                                     │
│                    │   HUB    │                                     │
│                    └┬───┬───┬─┘                                     │
│                     │   │   │                                       │
│              ┌──────┘   │   └──────┐                                │
│              │          │          │                                │
│         ┌────┴───┐ ┌────┴───┐ ┌────┴───┐                           │
│         │Device 1│ │Device 2│ │  HUB   │                           │
│         └────────┘ └────────┘ └───┬────┘                           │
│                                   │                                 │
│                              ┌────┴───┐                             │
│                              │Device 3│                             │
│                              └────────┘                             │
│                                                                      │
│  Key Features:                                                       │
│  • Host-controlled (polled)                                         │
│  • Hot-pluggable                                                    │
│  • Self-describing (descriptors)                                    │
│  • Up to 127 devices                                                │
│  • Multiple speed classes                                           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### USB Speeds and Connectors

```
┌─────────────────────────────────────────────────────────────────────┐
│                    USB VERSIONS                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Version    │ Speed         │ Data Rate │ Common Use        │     │
│  ├────────────────────────────────────────────────────────────┤     │
│  │ USB 1.0    │ Low Speed     │ 1.5 Mbps  │ Keyboards, mice   │     │
│  │ USB 1.1    │ Full Speed    │ 12 Mbps   │ Audio, HID        │     │
│  │ USB 2.0    │ High Speed    │ 480 Mbps  │ Mass storage      │     │
│  │ USB 3.0    │ SuperSpeed    │ 5 Gbps    │ External drives   │     │
│  │ USB 3.1    │ SuperSpeed+   │ 10 Gbps   │ Video, fast SSDs  │     │
│  │ USB 3.2    │ SuperSpeed+   │ 20 Gbps   │ Docking stations  │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  CONNECTOR PINOUTS:                                                  │
│                                                                      │
│  USB Type-A:              USB Type-C (simplified):                  │
│  ┌─────────────┐          ┌───────────────────────┐                 │
│  │ 1  2  3  4  │          │ A1 ... A6   A7 ... A12│                 │
│  └─────────────┘          │ B12... B7   B6 ... B1 │                 │
│   V  D- D+ G              └───────────────────────┘                 │
│                            Reversible, 24 pins                      │
│                                                                      │
│  USB 2.0 Signals:                                                   │
│  • VBUS: 5V power (500mA USB 2.0, 900mA USB 3.0)                   │
│  • D+/D-: Differential data pair                                    │
│  • GND: Ground                                                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### USB Device Classes

| Class | Code | Description | Examples |
|-------|------|-------------|----------|
| **HID** | 0x03 | Human Interface Device | Keyboard, mouse, gamepad |
| **CDC** | 0x02 | Communication Device | Virtual COM port |
| **MSC** | 0x08 | Mass Storage | USB drives, card readers |
| **Audio** | 0x01 | Audio streaming | Microphones, speakers |
| **Video** | 0x0E | Video streaming | Webcams |
| **Vendor** | 0xFF | Custom protocol | Programmers, debuggers |

### USB Descriptors

```
┌─────────────────────────────────────────────────────────────────────┐
│                    USB DESCRIPTOR HIERARCHY                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────┐        │
│  │               DEVICE DESCRIPTOR                          │        │
│  │  • VID (Vendor ID), PID (Product ID)                     │        │
│  │  • Device class, USB version                             │        │
│  │  • Max packet size, number of configurations             │        │
│  └────────────────────────────┬────────────────────────────┘        │
│                               │                                      │
│               ┌───────────────┴───────────────┐                     │
│               │                               │                      │
│  ┌────────────┴────────────┐    ┌─────────────┴────────────┐        │
│  │ CONFIGURATION DESC #1   │    │  CONFIGURATION DESC #2   │        │
│  │ • Power requirements    │    │  (alternate config)      │        │
│  │ • Number of interfaces  │    │                          │        │
│  └────────────┬────────────┘    └──────────────────────────┘        │
│               │                                                      │
│       ┌───────┴───────┐                                             │
│       │               │                                              │
│  ┌────┴─────┐    ┌────┴─────┐                                       │
│  │INTERFACE │    │INTERFACE │                                       │
│  │ DESC #0  │    │ DESC #1  │                                       │
│  │ CDC Ctrl │    │ CDC Data │                                       │
│  └────┬─────┘    └────┬─────┘                                       │
│       │               │                                              │
│  ┌────┴─────┐    ┌────┴─────┐                                       │
│  │ENDPOINT  │    │ENDPOINT  │                                       │
│  │ DESC     │    │ DESC     │                                       │
│  │ INT IN   │    │ BULK IN  │                                       │
│  └──────────┘    │ BULK OUT │                                       │
│                  └──────────┘                                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### USB CDC (Virtual COM Port)

```c
// USB CDC Device Descriptor
const uint8_t device_descriptor[] = {
    18,                     // bLength
    0x01,                   // bDescriptorType (Device)
    0x00, 0x02,             // bcdUSB (USB 2.0)
    0x02,                   // bDeviceClass (CDC)
    0x02,                   // bDeviceSubClass
    0x00,                   // bDeviceProtocol
    64,                     // bMaxPacketSize0
    0x83, 0x04,             // idVendor (0x0483 = ST)
    0x40, 0x57,             // idProduct
    0x00, 0x02,             // bcdDevice
    1,                      // iManufacturer (string index)
    2,                      // iProduct (string index)
    3,                      // iSerialNumber (string index)
    1                       // bNumConfigurations
};

// USB CDC TX (send data to host)
void usb_cdc_send(uint8_t *data, uint16_t len) {
    // Wait for TX buffer ready
    while (CDC_Transmit_FS(data, len) == USBD_BUSY);
}

// USB CDC RX callback (called when data received)
void CDC_Receive_Callback(uint8_t *buf, uint32_t len) {
    // Process received data
    for (uint32_t i = 0; i < len; i++) {
        process_byte(buf[i]);
    }
}
```

---

## Wireless Protocols Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    WIRELESS PROTOCOL COMPARISON                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │ Protocol   │ Range    │ Data Rate  │ Power   │ Application   │  │
│  ├───────────────────────────────────────────────────────────────┤  │
│  │ Bluetooth  │ 10-100m  │ 1-3 Mbps   │ Medium  │ Audio, HID    │  │
│  │ BLE        │ 10-50m   │ 1-2 Mbps   │ Low     │ IoT sensors   │  │
│  │ WiFi       │ 50-100m  │ 1+ Gbps    │ High    │ Internet      │  │
│  │ Zigbee     │ 10-100m  │ 250 kbps   │ Low     │ Home auto     │  │
│  │ LoRa       │ 2-15 km  │ 0.3-50kbps │ Low     │ LPWAN IoT     │  │
│  │ NFC        │ <10 cm   │ 424 kbps   │ Low     │ Payments      │  │
│  │ 433 MHz    │ 100-500m │ 1-100 kbps │ Low     │ Remote ctrl   │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  RANGE vs DATA RATE vs POWER:                                       │
│                                                                      │
│  Data Rate                                                           │
│      ▲                                                               │
│      │   ┌─────┐                                                    │
│  1Gbps   │WiFi │                                                    │
│      │   └─────┘                                                    │
│      │                                                               │
│  10Mbps   ┌───────────┐                                             │
│      │    │ Bluetooth │                                             │
│      │    └───────────┘                                             │
│      │      ┌───┐                                                   │
│  1Mbps     │BLE│                                                    │
│      │      └───┘                                                   │
│      │           ┌──────┐                                           │
│  100kbps         │Zigbee│                                           │
│      │           └──────┘                                           │
│      │                            ┌────┐                            │
│  10kbps                           │LoRa│                            │
│      │                            └────┘                            │
│      └────────────┬────────────┬──────────┬──────▶ Range            │
│                  10m         100m        1km                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Bluetooth Low Energy (BLE)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    BLE ARCHITECTURE                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                     BLE PROTOCOL STACK                       │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │  APPLICATION (User code)                                     │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │  GAP (Generic Access Profile) - Discovery, connection       │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │  GATT (Generic Attribute Profile) - Data organization       │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │  ATT (Attribute Protocol) - Client-server data exchange     │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │  L2CAP (Logical Link Control) - Packet fragmentation        │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │  HCI (Host Controller Interface)                            │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │  Link Layer - Advertising, connection, encryption           │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │  Physical Layer - 2.4 GHz radio, 40 channels               │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  GATT Data Model:                                                   │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                      PROFILE                                 │    │
│  │  ┌─────────────────────┐  ┌─────────────────────┐           │    │
│  │  │     SERVICE         │  │     SERVICE         │           │    │
│  │  │  (UUID: 0x180F)     │  │  (UUID: 0x1809)     │           │    │
│  │  │  Battery Service    │  │  Health Thermo      │           │    │
│  │  │ ┌─────────────────┐ │  │ ┌─────────────────┐ │           │    │
│  │  │ │ CHARACTERISTIC  │ │  │ │ CHARACTERISTIC  │ │           │    │
│  │  │ │ Battery Level   │ │  │ │ Temperature     │ │           │    │
│  │  │ │ ┌─────────────┐ │ │  │ │ ┌─────────────┐ │ │           │    │
│  │  │ │ │  VALUE      │ │ │  │ │ │  VALUE      │ │ │           │    │
│  │  │ │ │   85%       │ │ │  │ │ │  36.5°C     │ │ │           │    │
│  │  │ │ └─────────────┘ │ │  │ │ └─────────────┘ │ │           │    │
│  │  │ │ ┌─────────────┐ │ │  │ └─────────────────┘ │           │    │
│  │  │ │ │ DESCRIPTOR  │ │ │  └─────────────────────┘           │    │
│  │  │ │ └─────────────┘ │ │                                    │    │
│  │  │ └─────────────────┘ │                                    │    │
│  │  └─────────────────────┘                                    │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### LoRa/LoRaWAN

```
┌─────────────────────────────────────────────────────────────────────┐
│                    LoRa NETWORK ARCHITECTURE                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                                                                      │
│  ┌───────┐  LoRa  ┌──────────┐  IP    ┌──────────┐   ┌──────────┐  │
│  │Sensor │───────▶│ Gateway  │───────▶│ Network  │──▶│Application│  │
│  │ Node  │  Radio │          │        │  Server  │   │  Server   │  │
│  └───────┘        └──────────┘        └──────────┘   └──────────┘  │
│                                                                      │
│  ┌───────┐        ┌──────────┐                                      │
│  │Sensor │───────▶│ Gateway  │────────────┘                        │
│  │ Node  │        │    2     │                                      │
│  └───────┘        └──────────┘                                      │
│                   Star-of-stars topology                            │
│                                                                      │
│  LoRa PARAMETERS:                                                    │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Parameter          │ Typical Values                        │     │
│  ├────────────────────────────────────────────────────────────┤     │
│  │ Frequency          │ 868 MHz (EU), 915 MHz (US)            │     │
│  │ Spreading Factor   │ SF7-SF12 (higher = longer range)     │     │
│  │ Bandwidth          │ 125-500 kHz                          │     │
│  │ Coding Rate        │ 4/5, 4/6, 4/7, 4/8                   │     │
│  │ Max Payload        │ 51-222 bytes (depends on SF)         │     │
│  │ Range              │ 2-15 km (urban to rural)             │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  Higher Spreading Factor = Longer range but lower data rate        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Ethernet (Brief Overview)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    EMBEDDED ETHERNET                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  For embedded systems with network connectivity:                    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  APPLICATION (HTTP, MQTT, CoAP, Modbus TCP)                 │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │  TCP / UDP                                                   │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │  IP (IPv4 / IPv6)                                           │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │  MAC (Ethernet - 10/100 Mbps)                               │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │  PHY (Physical Layer Transceiver)                           │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │  RJ45 Connector + Magnetics                                 │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  COMMON STACKS:                                                      │
│  • lwIP (Lightweight IP) - Open source, minimal footprint          │
│  • uIP - Ultra-lightweight (8-bit friendly)                        │
│  • Vendor stacks (ST, NXP, Microchip)                              │
│                                                                      │
│  W5500 (SPI Ethernet):                                              │
│  ┌─────────┐  SPI   ┌──────────────────────┐                       │
│  │   MCU   │───────▶│ W5500 (HW TCP/IP)    │──▶ RJ45               │
│  └─────────┘        └──────────────────────┘                       │
│  Hardware TCP/IP offloading - simple for small MCUs                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Protocol Selection Guide

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHOOSING THE RIGHT PROTOCOL                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                           START                                      │
│                             │                                        │
│                             ▼                                        │
│                    ┌────────────────┐                               │
│                    │ Wired or       │                               │
│                    │ Wireless?      │                               │
│                    └───────┬────────┘                               │
│                   Wired    │    Wireless                            │
│           ┌────────────────┴────────────────┐                       │
│           ▼                                 ▼                        │
│  ┌──────────────┐                  ┌──────────────┐                 │
│  │ Speed need?  │                  │ Range need?  │                 │
│  └──────┬───────┘                  └──────┬───────┘                 │
│         │                                 │                          │
│    ┌────┴────┐                      ┌─────┴─────┐                   │
│    ▼         ▼                      ▼           ▼                   │
│  Low       High                  Short       Long                   │
│   │          │                  (<100m)      (>1km)                 │
│   ▼          ▼                     │           │                    │
│ I2C/SPI    USB/                    ▼           ▼                    │
│ UART      Ethernet             BLE/WiFi/     LoRa                   │
│                                 Zigbee                              │
│                                                                      │
│  SUMMARY DECISION MATRIX:                                            │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Requirement           │ Best Protocol                      │     │
│  ├────────────────────────────────────────────────────────────┤     │
│  │ Simple debug output   │ UART (printf)                      │     │
│  │ On-board sensors      │ I2C or SPI                         │     │
│  │ Multiple slaves, low  │ I2C (2 wires shared)               │     │
│  │ Fast display/SD card  │ SPI (dedicated CS)                 │     │
│  │ Automotive/industrial │ CAN (robust, deterministic)        │     │
│  │ PC connectivity       │ USB CDC (virtual COM)              │     │
│  │ Internet/cloud        │ Ethernet or WiFi                   │     │
│  │ Smartphone pairing    │ BLE                                │     │
│  │ Long-range IoT        │ LoRa                               │     │
│  │ Smart home mesh       │ Zigbee or Thread                   │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Protocol | Type | Speed | Distance | Topology | Best For |
|----------|------|-------|----------|----------|----------|
| **UART** | Async | 115 kbps | 15m | P2P | Debug, GPS |
| **SPI** | Sync | 50+ MHz | 0.1m | Master-Slave | Flash, displays |
| **I2C** | Sync | 400 kHz | 1m | Multi-drop | Sensors, EEPROMs |
| **CAN** | Async | 1 Mbps | 40m | Multi-master | Automotive |
| **USB** | Async | 480 Mbps | 5m | Host-Device | PC connection |
| **BLE** | Radio | 2 Mbps | 50m | Star | Wearables, IoT |
| **LoRa** | Radio | 50 kbps | 15 km | Star | LPWAN, sensors |
| **Ethernet** | Packet | 100+ Mbps | 100m | Star | Internet, servers |

---

## Quick Revision Questions

1. **What makes CAN suitable for automotive applications? Explain dominant and recessive states.**

2. **Describe USB enumeration. What is the role of descriptors?**

3. **Compare BLE to classic Bluetooth. Why is BLE preferred for IoT?**

4. **What is the trade-off with LoRa spreading factor? How does it affect range and data rate?**

5. **When would you choose Ethernet over WiFi in an embedded system?**

6. **A design needs: 50m range, 10 sensors, battery powered, smartphone control. Which protocols would you consider?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Chapter 5.4: I2C Protocol](04-i2c-protocol.md) | [Unit Index](README.md) | [Unit 6: Interfacing](../06-Interfacing/README.md) |

---
