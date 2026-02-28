# Chapter 8.1: Automotive Embedded Systems

## Introduction

Modern vehicles contain 50-150 Electronic Control Units (ECUs), each an embedded system managing everything from engine control to infotainment. Automotive embedded systems are characterized by harsh environments, long lifecycles (15+ years), stringent safety requirements, and complex networking. This chapter explores the architecture, protocols, and design considerations unique to automotive applications.

---

## Automotive System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│              VEHICLE ELECTRICAL ARCHITECTURE                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                     POWERTRAIN DOMAIN                          │ │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐          │ │
│  │  │ Engine  │  │ Trans-  │  │ Battery │  │ Motor   │          │ │
│  │  │ ECU     │  │ mission │  │ Mgmt    │  │ Control │          │ │
│  │  │ (EMS)   │  │ ECU     │  │ (BMS)   │  │ (EV/HEV)│          │ │
│  │  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘          │ │
│  │       └───────────┬┴───────────┴───────────┬─┘               │ │
│  └───────────────────┼────────────────────────┼─────────────────┘ │
│                      │  HIGH-SPEED CAN (500Kbps-1Mbps)           │ │
│  ════════════════════╪════════════════════════╪═════════════════  │
│                      │                        │                    │
│  ┌───────────────────┼────────────────────────┼─────────────────┐ │
│  │                   │    CHASSIS DOMAIN      │                  │ │
│  │  ┌─────────┐  ┌───┴─────┐  ┌─────────┐  ┌─┴───────┐         │ │
│  │  │   ABS   │  │   ESC   │  │Steering │  │Suspension│         │ │
│  │  │         │  │         │  │ (EPS)   │  │ (Active) │         │ │
│  │  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘         │ │
│  │       └───────────┬┴───────────┴───────────┬─┘               │ │
│  └───────────────────┼────────────────────────┼─────────────────┘ │
│                      │  CHASSIS CAN (500Kbps)                     │
│  ════════════════════╪════════════════════════════════════════   │
│                      │                                            │
│  ┌───────────────────┼──────────────────────────────────────────┐ │
│  │                   │     BODY DOMAIN                           │ │
│  │  ┌─────────┐  ┌───┴─────┐  ┌─────────┐  ┌─────────┐         │ │
│  │  │ Body   │  │ Door    │  │  HVAC   │  │ Lighting│         │ │
│  │  │ Control │  │ Modules │  │ Control │  │ Module  │         │ │
│  │  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘         │ │
│  │       └─────┬──────┴───────────┴───────────┬─┘               │ │
│  └─────────────┼──────────────────────────────┼─────────────────┘ │
│                │  LOW-SPEED CAN/LIN (125Kbps/20Kbps)             │
│  ══════════════╪══════════════════════════════╪═════════════════  │
│                │                              │                    │
│  ┌─────────────┼──────────────────────────────┼─────────────────┐ │
│  │             │  INFOTAINMENT DOMAIN         │                  │ │
│  │  ┌─────────┐│ ┌─────────┐  ┌─────────┐  ┌─┴───────┐         │ │
│  │  │  Head  ││ │ Cluster │  │ Audio   │  │ Telematics        │ │
│  │  │  Unit  ││ │(Digital)│  │ Amp     │  │ (TCU)   │         │ │
│  │  └────┬───┘│ └────┬────┘  └────┬────┘  └────┬────┘         │ │
│  │       └────┼──────┴───────────┴───────────┬─┘               │ │
│  │            │      ETHERNET / MOST / CAN                      │ │
│  └────────────┼─────────────────────────────────────────────────┘ │
│               │                                                    │
│  ┌────────────┼─────────────────────────────────────────────────┐ │
│  │            │         ADAS DOMAIN                              │ │
│  │  ┌─────────┴─┐  ┌─────────┐  ┌─────────┐  ┌─────────┐       │ │
│  │  │  Central  │  │ Camera  │  │  Radar  │  │  Lidar  │       │ │
│  │  │ Compute   │  │ Modules │  │ Sensors │  │ Sensors │       │ │
│  │  │  Unit     │  │         │  │         │  │         │       │ │
│  │  └───────────┘  └─────────┘  └─────────┘  └─────────┘       │ │
│  │                  AUTOMOTIVE ETHERNET (100Mbps-10Gbps)        │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    GATEWAY ECU                                 │ │
│  │            (Interconnects all domains)                         │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## CAN Bus Protocol

### CAN Frame Structure

```
┌─────────────────────────────────────────────────────────────────────┐
│              CAN 2.0B FRAME FORMAT                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  STANDARD FRAME (11-bit ID):                                        │
│                                                                      │
│  ┌─┬───────────┬─┬─┬─┬────┬──────────────────┬──────────┬─┬─┬─────┐│
│  │S│  11-bit   │R│I│r│DLC │   0-8 DATA       │   CRC    │A│ │ EOF ││
│  │O│    ID     │T│D│0│    │   BYTES          │  15-bit  │C│D│     ││
│  │F│           │R│E│ │    │                  │          │K│L│     ││
│  └─┴───────────┴─┴─┴─┴────┴──────────────────┴──────────┴─┴─┴─────┘│
│                                                                      │
│  SOF: Start of Frame (1 bit)                                        │
│  ID:  Identifier (11 bits) - also determines priority              │
│  RTR: Remote Transmission Request                                   │
│  IDE: Identifier Extension (0 for standard)                        │
│  r0:  Reserved                                                      │
│  DLC: Data Length Code (4 bits, 0-8 bytes)                         │
│  CRC: Cyclic Redundancy Check                                       │
│  ACK: Acknowledge slot                                              │
│  EOF: End of Frame (7 bits)                                         │
│                                                                      │
│  ═══════════════════════════════════════════════════════════════    │
│                                                                      │
│  EXTENDED FRAME (29-bit ID):                                        │
│                                                                      │
│  ┌─┬────────┬─┬─┬────────────────────┬─┬────┬──────────────┬─────┐ │
│  │S│ 11-bit │S│I│     18-bit         │R│DLC │  DATA        │ ... │ │
│  │O│ Base ID│R│D│   Extended ID      │T│    │              │     │ │
│  │F│        │R│E│                    │R│    │              │     │ │
│  └─┴────────┴─┴─┴────────────────────┴─┴────┴──────────────┴─────┘ │
│                                                                      │
│  SRR: Substitute Remote Request (always 1)                         │
│  IDE: Identifier Extension (1 for extended)                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### CAN Arbitration

```
┌─────────────────────────────────────────────────────────────────────┐
│              CAN BUS ARBITRATION                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  BUS STATES:                                                        │
│  • Dominant (0): Pulls bus low → WINS arbitration                  │
│  • Recessive (1): Released → Loses if another drives dominant      │
│                                                                      │
│  ARBITRATION EXAMPLE (3 nodes transmitting simultaneously):        │
│                                                                      │
│           Bit:   10  9   8   7   6   5   4   3   2   1   0         │
│  ────────────────────────────────────────────────────────────────   │
│  Node A (0x123): 0   0   1   0   0   1   0   0   0   1   1         │
│  Node B (0x120): 0   0   1   0   0   1   0   0   0   0   0         │
│  Node C (0x125): 0   0   1   0   0   1   0   0   1   0   1         │
│  ────────────────────────────────────────────────────────────────   │
│  BUS:            0   0   1   0   0   1   0   0   0   0   0         │
│                                              ↑       ↑              │
│                                              │       │              │
│                                         C loses  A loses            │
│                                                                      │
│  Node B wins (lowest ID = highest priority)                        │
│                                                                      │
│  TIMING DIAGRAM:                                                    │
│                                                                      │
│  CAN_H ────╲    ╱────╲    ╱────────────────╲    ╱────              │
│         2.5V╲  ╱      ╲  ╱                  ╲  ╱                    │
│              ╲╱        ╲╱                    ╲╱                      │
│               ┆         ┆                    ┆                       │
│  CAN_L ──────╱╲────────╱╲────────────────────╱╲──────               │
│         2.5V╱  ╲      ╱  ╲                  ╱  ╲                    │
│            ╱    ╲    ╱    ╲                ╱    ╲                   │
│                                                                      │
│       DOMINANT  RECESSIVE  DOMINANT      RECESSIVE                  │
│         (0)       (1)        (0)           (1)                      │
│                                                                      │
│  Differential voltage:                                              │
│  • Dominant: CAN_H - CAN_L ≈ 2V                                    │
│  • Recessive: CAN_H - CAN_L ≈ 0V                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### CAN Driver Implementation

```c
/* ================================================================
 * CAN Driver for STM32 - Automotive Application
 * ================================================================ */

#include "stm32f4xx.h"

/* CAN message structure */
typedef struct {
    uint32_t id;           /* 11-bit or 29-bit identifier */
    uint8_t  ide;          /* 0: Standard, 1: Extended */
    uint8_t  rtr;          /* 0: Data frame, 1: Remote frame */
    uint8_t  dlc;          /* Data length (0-8) */
    uint8_t  data[8];      /* Data payload */
    uint32_t timestamp;    /* Reception timestamp */
} can_msg_t;

/* Standard automotive message IDs */
#define CAN_ID_ENGINE_RPM       0x0C0
#define CAN_ID_VEHICLE_SPEED    0x0D0
#define CAN_ID_THROTTLE_POS     0x0E0
#define CAN_ID_COOLANT_TEMP     0x0F0
#define CAN_ID_BRAKE_PRESSURE   0x100

/* ================================================================
 * CAN Initialization (500kbps)
 * ================================================================ */
void can_init(void) {
    /* Enable CAN1 clock */
    RCC->APB1ENR |= RCC_APB1ENR_CAN1EN;
    
    /* Enable GPIOA clock (PA11=RX, PA12=TX) */
    RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;
    
    /* Configure GPIO for CAN (AF9) */
    GPIOA->MODER |= (2 << 22) | (2 << 24);  /* AF mode */
    GPIOA->AFR[1] |= (9 << 12) | (9 << 16); /* AF9 */
    
    /* Enter initialization mode */
    CAN1->MCR |= CAN_MCR_INRQ;
    while (!(CAN1->MSR & CAN_MSR_INAK));
    
    /* Exit sleep mode */
    CAN1->MCR &= ~CAN_MCR_SLEEP;
    
    /* Configure bit timing for 500kbps @ 42MHz APB1
     * Prescaler = 6, TS1 = 11, TS2 = 2, SJW = 1
     * Bit time = (1 + 11 + 2) / (42MHz / 6) = 2µs
     * Bit rate = 500kbps
     */
    CAN1->BTR = (1 << 24) |  /* SJW = 1 */
                (1 << 20) |  /* TS2 = 2 */
                (10 << 16) | /* TS1 = 11 */
                (5);         /* Prescaler = 6 */
    
    /* Configure filters - accept all messages */
    CAN1->FMR |= CAN_FMR_FINIT;  /* Filter init mode */
    CAN1->FA1R |= (1 << 0);      /* Activate filter 0 */
    CAN1->FM1R &= ~(1 << 0);     /* Mask mode */
    CAN1->FS1R |= (1 << 0);      /* 32-bit scale */
    CAN1->sFilterRegister[0].FR1 = 0;  /* Accept all */
    CAN1->sFilterRegister[0].FR2 = 0;
    CAN1->FMR &= ~CAN_FMR_FINIT;
    
    /* Leave initialization mode */
    CAN1->MCR &= ~CAN_MCR_INRQ;
    while (CAN1->MSR & CAN_MSR_INAK);
    
    /* Enable RX interrupt */
    CAN1->IER |= CAN_IER_FMPIE0;
    NVIC_EnableIRQ(CAN1_RX0_IRQn);
}

/* ================================================================
 * Transmit CAN Message
 * ================================================================ */
int can_transmit(can_msg_t *msg) {
    uint32_t mailbox;
    
    /* Find empty mailbox */
    if (CAN1->TSR & CAN_TSR_TME0) {
        mailbox = 0;
    } else if (CAN1->TSR & CAN_TSR_TME1) {
        mailbox = 1;
    } else if (CAN1->TSR & CAN_TSR_TME2) {
        mailbox = 2;
    } else {
        return -1;  /* All mailboxes full */
    }
    
    /* Set identifier */
    if (msg->ide) {
        /* Extended ID */
        CAN1->sTxMailBox[mailbox].TIR = 
            (msg->id << 3) | CAN_TI0R_IDE;
    } else {
        /* Standard ID */
        CAN1->sTxMailBox[mailbox].TIR = (msg->id << 21);
    }
    
    if (msg->rtr) {
        CAN1->sTxMailBox[mailbox].TIR |= CAN_TI0R_RTR;
    }
    
    /* Set data length */
    CAN1->sTxMailBox[mailbox].TDTR = msg->dlc;
    
    /* Set data bytes */
    CAN1->sTxMailBox[mailbox].TDLR = 
        msg->data[0] | (msg->data[1] << 8) | 
        (msg->data[2] << 16) | (msg->data[3] << 24);
    CAN1->sTxMailBox[mailbox].TDHR = 
        msg->data[4] | (msg->data[5] << 8) | 
        (msg->data[6] << 16) | (msg->data[7] << 24);
    
    /* Request transmission */
    CAN1->sTxMailBox[mailbox].TIR |= CAN_TI0R_TXRQ;
    
    return 0;
}

/* ================================================================
 * Receive CAN Message (called from ISR)
 * ================================================================ */
int can_receive(can_msg_t *msg) {
    if (!(CAN1->RF0R & CAN_RF0R_FMP0)) {
        return -1;  /* No message pending */
    }
    
    /* Get identifier */
    if (CAN1->sFIFOMailBox[0].RIR & CAN_RI0R_IDE) {
        msg->ide = 1;
        msg->id = CAN1->sFIFOMailBox[0].RIR >> 3;
    } else {
        msg->ide = 0;
        msg->id = CAN1->sFIFOMailBox[0].RIR >> 21;
    }
    
    msg->rtr = (CAN1->sFIFOMailBox[0].RIR & CAN_RI0R_RTR) ? 1 : 0;
    msg->dlc = CAN1->sFIFOMailBox[0].RDTR & 0x0F;
    
    /* Get timestamp */
    msg->timestamp = (CAN1->sFIFOMailBox[0].RDTR >> 16) & 0xFFFF;
    
    /* Get data */
    msg->data[0] = CAN1->sFIFOMailBox[0].RDLR;
    msg->data[1] = CAN1->sFIFOMailBox[0].RDLR >> 8;
    msg->data[2] = CAN1->sFIFOMailBox[0].RDLR >> 16;
    msg->data[3] = CAN1->sFIFOMailBox[0].RDLR >> 24;
    msg->data[4] = CAN1->sFIFOMailBox[0].RDHR;
    msg->data[5] = CAN1->sFIFOMailBox[0].RDHR >> 8;
    msg->data[6] = CAN1->sFIFOMailBox[0].RDHR >> 16;
    msg->data[7] = CAN1->sFIFOMailBox[0].RDHR >> 24;
    
    /* Release FIFO */
    CAN1->RF0R |= CAN_RF0R_RFOM0;
    
    return 0;
}

/* ================================================================
 * Application: Engine Data Broadcasting
 * ================================================================ */
void broadcast_engine_data(void) {
    can_msg_t msg;
    
    /* Send RPM (0x0C0) */
    msg.id = CAN_ID_ENGINE_RPM;
    msg.ide = 0;
    msg.rtr = 0;
    msg.dlc = 2;
    uint16_t rpm = engine_get_rpm();
    msg.data[0] = rpm >> 8;   /* High byte first */
    msg.data[1] = rpm & 0xFF;
    can_transmit(&msg);
    
    /* Send Coolant Temperature (0x0F0) */
    msg.id = CAN_ID_COOLANT_TEMP;
    msg.dlc = 1;
    msg.data[0] = engine_get_coolant_temp() + 40;  /* Offset encoding */
    can_transmit(&msg);
}
```

---

## Engine Management System

```
┌─────────────────────────────────────────────────────────────────────┐
│              ENGINE CONTROL UNIT (ECU) ARCHITECTURE                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                       SENSORS                                  │ │
│  │                                                                │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      │ │
│  │  │ Crank    │  │   MAP    │  │   MAF    │  │ Throttle │      │ │
│  │  │ Position │  │ (Manifold│  │(Mass Air │  │ Position │      │ │
│  │  │ Sensor   │  │ Pressure)│  │  Flow)   │  │ Sensor   │      │ │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘      │ │
│  │       │             │             │             │             │ │
│  │  ┌────┴─────┐  ┌────┴─────┐  ┌────┴─────┐  ┌────┴─────┐      │ │
│  │  │ Cam      │  │ Coolant  │  │ Intake   │  │   O2     │      │ │
│  │  │ Position │  │   Temp   │  │ Air Temp │  │  Sensor  │      │ │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘      │ │
│  └───────┼─────────────┼─────────────┼─────────────┼─────────────┘ │
│          │             │             │             │                │
│  ┌───────▼─────────────▼─────────────▼─────────────▼─────────────┐ │
│  │                    ENGINE CONTROL UNIT                         │ │
│  │                                                                │ │
│  │  ┌──────────────────────────────────────────────────────────┐ │ │
│  │  │                  MICROCONTROLLER                         │ │ │
│  │  │            (e.g., Infineon AURIX TC3xx)                  │ │ │
│  │  │                                                          │ │ │
│  │  │  ┌──────────────────────────────────────────────────┐   │ │ │
│  │  │  │                INPUT CONDITIONING                 │   │ │ │
│  │  │  │  • Signal filtering                              │   │ │ │
│  │  │  │  • Clamping/protection                           │   │ │ │
│  │  │  │  • A/D conversion                                │   │ │ │
│  │  │  └──────────────────────────────────────────────────┘   │ │ │
│  │  │                          │                               │ │ │
│  │  │                          ▼                               │ │ │
│  │  │  ┌──────────────────────────────────────────────────┐   │ │ │
│  │  │  │              ENGINE CONTROL ALGORITHMS            │   │ │ │
│  │  │  │                                                   │   │ │ │
│  │  │  │  • Fuel injection timing & duration              │   │ │ │
│  │  │  │  • Ignition timing (spark advance)               │   │ │ │
│  │  │  │  • Idle speed control                            │   │ │ │
│  │  │  │  • Knock detection & mitigation                  │   │ │ │
│  │  │  │  • Emissions control (O2 feedback)               │   │ │ │
│  │  │  │  • Variable valve timing                         │   │ │ │
│  │  │  │                                                   │   │ │ │
│  │  │  │  Uses: Lookup tables, PID control, state machines │   │ │ │
│  │  │  └──────────────────────────────────────────────────┘   │ │ │
│  │  │                          │                               │ │ │
│  │  │                          ▼                               │ │ │
│  │  │  ┌──────────────────────────────────────────────────┐   │ │ │
│  │  │  │               OUTPUT DRIVERS                      │   │ │ │
│  │  │  │  • High-current for injectors/coils              │   │ │ │
│  │  │  │  • PWM generation                                 │   │ │ │
│  │  │  │  • Diagnostics (open/short detection)            │   │ │ │
│  │  │  └──────────────────────────────────────────────────┘   │ │ │
│  │  └──────────────────────────────────────────────────────────┘ │ │
│  └────────────────────────────────────────────────────────────────┘ │
│          │             │             │             │                │
│  ┌───────▼─────────────▼─────────────▼─────────────▼─────────────┐ │
│  │                       ACTUATORS                                │ │
│  │                                                                │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      │ │
│  │  │   Fuel   │  │ Ignition │  │  Idle    │  │ Variable │      │ │
│  │  │ Injectors│  │  Coils   │  │ Control  │  │  Valve   │      │ │
│  │  │          │  │          │  │  Valve   │  │  Timing  │      │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘      │ │
│  │                                                                │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ADAS (Advanced Driver Assistance Systems)

```
┌─────────────────────────────────────────────────────────────────────┐
│              ADAS SENSOR FUSION SYSTEM                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SENSOR SUITE:                                                       │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                                                                │ │
│  │                    Front Camera                                │ │
│  │                         │                                      │ │
│  │     Front Radar ─────┬──┼──┬───── Front Radar                 │ │
│  │         (77GHz)       │  │  │       (77GHz)                    │ │
│  │                       │  │  │                                  │ │
│  │  ┌─────────────────────────────────────────────────┐          │ │
│  │  │                                                 │          │ │
│  │  │    ┌───────────────────────────────────────┐   │          │ │
│  │  │    │            VEHICLE                    │   │          │ │
│  │  │    │                                       │   │          │ │
│  │  │ ──▶│  Side     ┌─────────────┐     Side   │◀── │          │ │
│  │  │    │  Radar    │   CENTRAL   │     Radar  │   │          │ │
│  │  │    │           │   COMPUTE   │            │   │          │ │
│  │  │    │           │    UNIT     │            │   │          │ │
│  │  │    │           └─────────────┘            │   │          │ │
│  │  │    │  Ultrasonic sensors around perimeter │   │          │ │
│  │  │    └───────────────────────────────────────┘   │          │ │
│  │  │                                                 │          │ │
│  │  └─────────────────────────────────────────────────┘          │ │
│  │                       │  │  │                                  │ │
│  │     Rear Radar  ─────┴──┼──┴───── Rear Camera                 │ │
│  │                         │                                      │ │
│  │                    Rear Camera                                 │ │
│  │                                                                │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  ADAS FEATURES BY SENSOR:                                           │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │ Sensor     │ Range    │ Features Enabled                      │ │
│  ├────────────┼──────────┼────────────────────────────────────────┤ │
│  │ Camera     │ 0-200m   │ Lane detection, sign recognition,     │ │
│  │            │          │ pedestrian detection, traffic lights  │ │
│  ├────────────┼──────────┼────────────────────────────────────────┤ │
│  │ Radar      │ 0-250m   │ Adaptive cruise control, collision    │ │
│  │ (77GHz)    │          │ warning, blind spot detection         │ │
│  ├────────────┼──────────┼────────────────────────────────────────┤ │
│  │ Lidar      │ 0-200m   │ 3D mapping, precise distance,         │ │
│  │            │          │ autonomous navigation                 │ │
│  ├────────────┼──────────┼────────────────────────────────────────┤ │
│  │ Ultrasonic │ 0-5m     │ Parking assist, low-speed detection   │ │
│  │            │          │                                        │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  SENSOR FUSION PIPELINE:                                            │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                                                                │ │
│  │  ┌────────┐   ┌────────────┐   ┌────────────┐   ┌──────────┐ │ │
│  │  │Raw Data│──▶│Preprocessing│──▶│  Object    │──▶│ Tracking │ │ │
│  │  │Streams │   │& Detection │   │Classification│  │& Fusion  │ │ │
│  │  └────────┘   └────────────┘   └────────────┘   └────┬─────┘ │ │
│  │                                                       │       │ │
│  │                                                       ▼       │ │
│  │  ┌────────┐   ┌────────────┐   ┌────────────┐   ┌──────────┐ │ │
│  │  │Actuator│◀──│   Path     │◀──│ Decision   │◀──│ Situation│ │ │
│  │  │Control │   │ Planning   │   │  Making    │   │ Analysis │ │ │
│  │  └────────┘   └────────────┘   └────────────┘   └──────────┘ │ │
│  │                                                                │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Functional Safety (ISO 26262)

```
┌─────────────────────────────────────────────────────────────────────┐
│              ISO 26262 AUTOMOTIVE SAFETY                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ASIL (Automotive Safety Integrity Level):                          │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │ Level │ Severity │ Probability │ Example                      │ │
│  ├───────┼──────────┼─────────────┼──────────────────────────────┤ │
│  │ QM    │ -        │ -           │ Interior lighting            │ │
│  │ ASIL A│ Low      │ Low         │ Tail lights                  │ │
│  │ ASIL B│ Medium   │ Medium      │ Headlights, wipers           │ │
│  │ ASIL C│ High     │ High        │ Airbag deployment logic      │ │
│  │ ASIL D│ Very High│ Very High   │ Braking, steering            │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  SAFETY MECHANISMS:                                                  │
│                                                                      │
│  1. HARDWARE REDUNDANCY                                             │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                                                               │  │
│  │   Sensor A ────┐                                             │  │
│  │                ├────▶ Comparator ────▶ Safe Output           │  │
│  │   Sensor B ────┘        │                                    │  │
│  │                         │                                     │  │
│  │                    Mismatch? ────▶ Fault Handler             │  │
│  │                                                               │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  2. WATCHDOG TIMER                                                  │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                                                               │  │
│  │   Application ────▶ Kick WDT ────▶ WDT Counter Reset         │  │
│  │                        │                                      │  │
│  │                   Timeout?                                    │  │
│  │                        │                                      │  │
│  │                        ▼                                      │  │
│  │                  System Reset ────▶ Safe State               │  │
│  │                                                               │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  3. LOCKSTEP CPU (ASIL D)                                          │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                                                               │  │
│  │   ┌────────┐        ┌────────┐                              │  │
│  │   │ CPU A  │───────▶│Compare │                              │  │
│  │   └────────┘        │ Logic  │────▶ Output if match         │  │
│  │   ┌────────┐        │        │                              │  │
│  │   │ CPU B  │───────▶│        │────▶ Error if mismatch       │  │
│  │   │(shadow)│        └────────┘                              │  │
│  │   └────────┘                                                 │  │
│  │                                                               │  │
│  │   Both CPUs execute same code, results compared              │  │
│  │                                                               │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  4. MEMORY PROTECTION                                               │
│  • ECC (Error Correction Code) for RAM                             │
│  • CRC for Flash content verification                              │
│  • MPU (Memory Protection Unit) regions                            │
│                                                                      │
│  5. PROGRAM FLOW MONITORING                                         │
│  • Sequence checking (correct execution order)                     │
│  • Deadline monitoring (timing constraints)                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **ECU** | Electronic Control Unit - dedicated embedded computer for specific function |
| **CAN Bus** | Controller Area Network - robust, priority-based vehicle network |
| **Arbitration** | Lower CAN ID wins; non-destructive collision resolution |
| **Domains** | Powertrain, Chassis, Body, Infotainment, ADAS - separated by function |
| **Gateway** | Central ECU that routes messages between domains |
| **ADAS** | Advanced Driver Assistance - uses sensor fusion for safety features |
| **ISO 26262** | Automotive functional safety standard |
| **ASIL** | Automotive Safety Integrity Level (A-D, D highest) |
| **Lockstep** | Dual CPU architecture for safety-critical functions |

---

## Quick Revision Questions

1. **Explain the structure of a CAN message frame. What determines message priority on the bus?**

2. **What is the difference between CAN_H and CAN_L signals? Why is differential signaling used?**

3. **Describe the typical domains in a modern vehicle's electrical architecture. Why are they separated?**

4. **What sensors are used in ADAS systems? What is sensor fusion and why is it needed?**

5. **Explain ASIL levels in ISO 26262. Give an example of a system that would require ASIL D.**

6. **What safety mechanisms are used to achieve ASIL D compliance? Explain the lockstep CPU concept.**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Unit 8 Overview](README.md) | [Unit 8: Case Studies](README.md) | [Chapter 8.2: Medical Devices](02-medical-devices.md) |

---
