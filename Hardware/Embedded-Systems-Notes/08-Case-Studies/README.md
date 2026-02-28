# Unit 8: Case Studies

## Overview

This unit presents real-world applications of embedded systems across different industries. Each case study demonstrates how the concepts from previous units—hardware, software, RTOS, communication protocols, interfacing, and system design—come together in practical implementations.

---

## Unit Structure

```
┌─────────────────────────────────────────────────────────────────────┐
│                  EMBEDDED SYSTEMS DOMAINS                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                                                                │ │
│  │            ┌─────────────┐         ┌─────────────┐            │ │
│  │            │ AUTOMOTIVE  │         │  MEDICAL    │            │ │
│  │            │ • ECUs      │         │ • Monitors  │            │ │
│  │            │ • CAN Bus   │         │ • Implants  │            │ │
│  │            │ • ADAS      │         │ • Regulation│            │ │
│  │            └─────────────┘         └─────────────┘            │ │
│  │                                                                │ │
│  │     ┌──────────────────────────────────────────────────┐      │ │
│  │     │              EMBEDDED SYSTEMS                     │      │ │
│  │     │         Real-time | Resource-constrained          │      │ │
│  │     │         Reliable | Application-specific          │      │ │
│  │     └──────────────────────────────────────────────────┘      │ │
│  │                                                                │ │
│  │            ┌─────────────┐         ┌─────────────┐            │ │
│  │            │     IoT     │         │  CONSUMER   │            │ │
│  │            │ • Sensors   │         │ • Wearables │            │ │
│  │            │ • Cloud     │         │ • Smart Home│            │ │
│  │            │ • Edge      │         │ • Low Power │            │ │
│  │            └─────────────┘         └─────────────┘            │ │
│  │                                                                │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Chapter Overview

### Chapter 8.1: Automotive Embedded Systems
- Electronic Control Units (ECUs)
- CAN bus networking
- Engine management systems
- Advanced Driver Assistance Systems (ADAS)
- Functional safety (ISO 26262)

### Chapter 8.2: Medical Devices
- Patient monitoring systems
- Implantable devices
- Safety-critical design
- Regulatory requirements (IEC 62304)
- Power and reliability considerations

### Chapter 8.3: IoT and Smart Devices
- Sensor networks
- Cloud connectivity
- Edge computing
- Smart home applications
- Industrial IoT (IIoT)

### Chapter 8.4: Consumer Electronics
- Wearable devices
- Smart appliances
- Battery management
- User experience design
- Cost optimization

---

## Industry Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│              EMBEDDED SYSTEMS BY INDUSTRY                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────┬────────────┬────────────┬────────────┬───────────┐ │
│  │ Aspect     │ Automotive │ Medical    │ IoT        │ Consumer  │ │
│  ├────────────┼────────────┼────────────┼────────────┼───────────┤ │
│  │ Safety     │ ASIL A-D   │ Class A-C  │ Low        │ Low-Med   │ │
│  │ Criticality│ (ISO 26262)│ (IEC 62304)│            │           │ │
│  ├────────────┼────────────┼────────────┼────────────┼───────────┤ │
│  │ Real-time  │ Hard       │ Hard/Soft  │ Soft       │ Soft      │ │
│  │ Requirement│ (ms level) │ (varies)   │ (flexible) │ (UX)      │ │
│  ├────────────┼────────────┼────────────┼────────────┼───────────┤ │
│  │ Lifetime   │ 15+ years  │ 5-20 years │ 3-10 years │ 2-5 years │ │
│  ├────────────┼────────────┼────────────┼────────────┼───────────┤ │
│  │ Volume     │ High       │ Med-Low    │ Very High  │ Very High │ │
│  │            │ millions   │ thousands  │ billions   │ millions  │ │
│  ├────────────┼────────────┼────────────┼────────────┼───────────┤ │
│  │ Cert Cost  │ Very High  │ Very High  │ Low-Med    │ Low       │ │
│  ├────────────┼────────────┼────────────┼────────────┼───────────┤ │
│  │ Update     │ Complex    │ Regulated  │ OTA common │ OTA       │ │
│  │ Mechanism  │ (recalls)  │ (FDA)      │            │ expected  │ │
│  ├────────────┼────────────┼────────────┼────────────┼───────────┤ │
│  │ Power      │ 12V/48V    │ Battery/   │ Battery    │ Battery/  │ │
│  │ Source     │ vehicle    │ mains      │ harvesting │ USB       │ │
│  ├────────────┼────────────┼────────────┼────────────┼───────────┤ │
│  │ Key        │ CAN, LIN   │ Wireless   │ WiFi, BLE  │ BLE, WiFi │ │
│  │ Protocols  │ Ethernet   │ proprietary│ LoRa, MQTT │ Zigbee    │ │
│  └────────────┴────────────┴────────────┴────────────┴───────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Common Design Patterns

```
┌─────────────────────────────────────────────────────────────────────┐
│              ARCHITECTURAL PATTERNS ACROSS DOMAINS                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. SENSE-PROCESS-ACTUATE LOOP                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                                                               │  │
│  │    ┌─────────┐      ┌─────────┐      ┌─────────┐            │  │
│  │    │ SENSORS │─────▶│ PROCESS │─────▶│ACTUATORS│            │  │
│  │    └─────────┘      └─────────┘      └─────────┘            │  │
│  │         │                                  │                 │  │
│  │         └───────── FEEDBACK ◀──────────────┘                 │  │
│  │                                                               │  │
│  │  Examples:                                                    │  │
│  │  • Automotive: Throttle sensor → ECU → Fuel injector        │  │
│  │  • Medical: Heart rate → MCU → Alert/pacing                 │  │
│  │  • IoT: Temperature → Gateway → HVAC control                │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  2. LAYERED ARCHITECTURE                                            │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                                                               │  │
│  │  ┌────────────────────────────────────────────────────────┐ │  │
│  │  │                  APPLICATION LAYER                      │ │  │
│  │  │           (Business logic, user interface)              │ │  │
│  │  └─────────────────────────┬──────────────────────────────┘ │  │
│  │  ┌─────────────────────────▼──────────────────────────────┐ │  │
│  │  │                  MIDDLEWARE LAYER                       │ │  │
│  │  │        (RTOS, communication stacks, security)           │ │  │
│  │  └─────────────────────────┬──────────────────────────────┘ │  │
│  │  ┌─────────────────────────▼──────────────────────────────┐ │  │
│  │  │                      HAL LAYER                          │ │  │
│  │  │        (Hardware abstraction, drivers)                  │ │  │
│  │  └─────────────────────────┬──────────────────────────────┘ │  │
│  │  ┌─────────────────────────▼──────────────────────────────┐ │  │
│  │  │                     HARDWARE                            │ │  │
│  │  │        (MCU, peripherals, sensors, actuators)           │ │  │
│  │  └────────────────────────────────────────────────────────┘ │  │
│  │                                                               │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  3. EVENT-DRIVEN ARCHITECTURE                                       │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                                                               │  │
│  │   Event         Event         State                          │  │
│  │   Sources       Queue         Machine                        │  │
│  │                                                               │  │
│  │  ┌─────┐      ┌─────────┐    ┌─────────────────────┐        │  │
│  │  │Timer│─────▶│         │    │  ┌─────┐            │        │  │
│  │  └─────┘      │         │    │  │IDLE │────────┐   │        │  │
│  │  ┌─────┐      │  Event  │───▶│  └──┬──┘        │   │        │  │
│  │  │GPIO │─────▶│  Queue  │    │     │     ┌────▼┐  │        │  │
│  │  └─────┘      │         │    │     └────▶│ACTIVE  │        │  │
│  │  ┌─────┐      │         │    │           └────┬┘  │        │  │
│  │  │UART │─────▶│         │    │                │   │        │  │
│  │  └─────┘      └─────────┘    │           ┌────▼┐  │        │  │
│  │                              │           │SLEEP│◀─┘        │  │
│  │                              │           └─────┘           │  │
│  │                              └─────────────────────┘        │  │
│  │                                                               │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Cross-Cutting Concerns

```
┌─────────────────────────────────────────────────────────────────────┐
│              CONCERNS ACROSS ALL DOMAINS                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SECURITY:                                                           │
│  • Secure boot and firmware updates                                 │
│  • Encrypted communication                                          │
│  • Authentication and access control                                │
│  • Protection against tampering                                     │
│                                                                      │
│  RELIABILITY:                                                        │
│  • Watchdog timers                                                  │
│  • Error detection and recovery                                     │
│  • Redundancy where critical                                        │
│  • Graceful degradation                                             │
│                                                                      │
│  TESTABILITY:                                                        │
│  • Design for test (DFT)                                            │
│  • Built-in self-test (BIST)                                        │
│  • Debug interfaces                                                  │
│  • Logging and diagnostics                                          │
│                                                                      │
│  MAINTAINABILITY:                                                    │
│  • Field update capability                                          │
│  • Configuration management                                         │
│  • Diagnostic data collection                                       │
│  • Long-term support planning                                       │
│                                                                      │
│  POWER EFFICIENCY:                                                   │
│  • Appropriate power modes                                          │
│  • Duty cycling                                                      │
│  • Efficient algorithms                                             │
│  • Hardware acceleration where needed                               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Learning Objectives

After completing this unit, you should be able to:

1. **Apply embedded system concepts** to specific industry domains
2. **Understand industry-specific requirements** for safety, reliability, and regulation
3. **Recognize common architectural patterns** across different applications
4. **Evaluate design trade-offs** for different use cases
5. **Design complete embedded systems** from requirements to implementation

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Unit 7: System Design](../07-System-Design/README.md) | [Course Index](../README.md) | [Chapter 8.1: Automotive Systems](01-automotive-systems.md) |

---
