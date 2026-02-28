# Chapter 5.3: SPI Protocol

## Introduction

SPI (Serial Peripheral Interface) is a synchronous, full-duplex communication protocol. It offers high-speed data transfer and is commonly used for displays, SD cards, sensors, and flash memory.

---

## SPI Bus Structure

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SPI BUS ARCHITECTURE                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SINGLE SLAVE:                                                       │
│                                                                      │
│     MASTER                              SLAVE                        │
│  ┌──────────────┐                   ┌──────────────┐                │
│  │              │  SCK ───────────▶ │ SCK          │                │
│  │              │  MOSI ──────────▶ │ MOSI (DI)    │                │
│  │    MCU       │  MISO ◀────────── │ MISO (DO)    │                │
│  │              │  CS ────────────▶ │ CS (SS)      │                │
│  └──────────────┘                   └──────────────┘                │
│                                                                      │
│  Signal Names:                                                       │
│  • SCK/SCLK = Serial Clock (Master generates)                       │
│  • MOSI = Master Out, Slave In (SDI, DI)                           │
│  • MISO = Master In, Slave Out (SDO, DO)                           │
│  • CS/SS = Chip Select (active low)                                 │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  MULTIPLE SLAVES (Independent CS):                                  │
│                                                                      │
│     MASTER             SLAVE 1      SLAVE 2      SLAVE 3           │
│  ┌──────────┐         ┌──────┐     ┌──────┐     ┌──────┐           │
│  │          │ SCK ────┤──────┼─────┤──────┼─────┤──────│           │
│  │          │ MOSI ───┤──────┼─────┤──────┼─────┤──────│           │
│  │          │ MISO ◀──┤──────┼──◀──┤──────┼──◀──┤──────│           │
│  │          │         │      │     │      │     │      │           │
│  │          │ CS1 ────┤ CS   │     │      │     │      │           │
│  │          │ CS2 ─────────────────┤ CS   │     │      │           │
│  │          │ CS3 ──────────────────────────────┤ CS   │           │
│  └──────────┘         └──────┘     └──────┘     └──────┘           │
│                                                                      │
│  Only ONE CS active at a time!                                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## SPI Timing and Modes

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SPI CLOCK MODES                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Two parameters define SPI mode:                                    │
│  • CPOL (Clock Polarity): Idle state of clock (0=Low, 1=High)      │
│  • CPHA (Clock Phase): When data is sampled (0=First, 1=Second)    │
│                                                                      │
│  MODE 0 (CPOL=0, CPHA=0) - Most common:                            │
│  Clock idles LOW, data sampled on RISING edge                      │
│                                                                      │
│  CS:   ─────┐                                               ┌─────  │
│             └───────────────────────────────────────────────┘       │
│                                                                      │
│  SCK:  ─────────┐   ┌───┐   ┌───┐   ┌───┐   ┌───────────────────   │
│                 └───┘   └───┘   └───┘   └───┘                       │
│                   ↑       ↑       ↑       ↑  Sample on rising       │
│                                                                      │
│  MOSI: ═════════╪═══════╪═══════╪═══════╪══════════════════════    │
│                 D7      D6      D5      D4...                        │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  MODE 1 (CPOL=0, CPHA=1):                                           │
│  Clock idles LOW, data sampled on FALLING edge                     │
│                                                                      │
│  SCK:  ─────────┐   ┌───┐   ┌───┐   ┌───┐   ┌───────────────────   │
│                 └───┘   └───┘   └───┘   └───┘                       │
│                     ↓       ↓       ↓       ↓  Sample on falling   │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  MODE 2 (CPOL=1, CPHA=0):                                           │
│  Clock idles HIGH, data sampled on FALLING edge                    │
│                                                                      │
│  SCK:  ─────────────┐   ┌───┐   ┌───┐   ┌───┐   ┌───────────       │
│                     └───┘   └───┘   └───┘   └───┘                   │
│                     ↓       ↓       ↓       ↓  Sample on falling   │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  MODE 3 (CPOL=1, CPHA=1):                                           │
│  Clock idles HIGH, data sampled on RISING edge                     │
│                                                                      │
│  SCK:  ─────────────┐   ┌───┐   ┌───┐   ┌───┐   ┌───────────       │
│                     └───┘   └───┘   └───┘   └───┘                   │
│                       ↑       ↑       ↑       ↑  Sample on rising  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### SPI Mode Summary Table

| Mode | CPOL | CPHA | Clock Idle | Sample Edge | Used By |
|------|------|------|------------|-------------|---------|
| 0 | 0 | 0 | Low | Rising | Most devices |
| 1 | 0 | 1 | Low | Falling | Some ADCs |
| 2 | 1 | 0 | High | Falling | Some ADCs |
| 3 | 1 | 1 | High | Rising | Some sensors |

---

## SPI Data Transfer

```
┌─────────────────────────────────────────────────────────────────────┐
│                    FULL-DUPLEX TRANSFER                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SPI is ALWAYS full-duplex: data exchanged simultaneously          │
│                                                                      │
│     MASTER                              SLAVE                        │
│  ┌──────────────┐                   ┌──────────────┐                │
│  │   TX: 0xA5   │   MOSI ─────────▶ │  RX: 0xA5    │                │
│  │   RX: 0x3C   │   MISO ◀───────── │  TX: 0x3C    │                │
│  └──────────────┘                   └──────────────┘                │
│                                                                      │
│  Master sends 0xA5, receives 0x3C                                   │
│  Slave receives 0xA5, sends 0x3C                                    │
│                                                                      │
│  TIMING (Mode 0, MSB first):                                        │
│                                                                      │
│  CS:    ──┐                                               ┌──       │
│           └───────────────────────────────────────────────┘         │
│                                                                      │
│  SCK:   ────┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌────                    │
│             └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘                         │
│             1   2   3   4   5   6   7   8                           │
│                                                                      │
│  MOSI:  ────╪───╪───╪───╪───╪───╪───╪───╪────                       │
│  (0xA5)     1   0   1   0   0   1   0   1                           │
│             ▲   ▲   ▲   ▲   ▲   ▲   ▲   ▲                           │
│  MISO:  ────╪───╪───╪───╪───╪───╪───╪───╪────                       │
│  (0x3C)     0   0   1   1   1   1   0   0                           │
│                                                                      │
│  Both directions transfer simultaneously on each clock!            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Read vs Write Operations

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TYPICAL SPI OPERATIONS                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  WRITE TO SLAVE (master sends command + data):                      │
│                                                                      │
│  CS:    ──┐                                               ┌──       │
│           └───────────────────────────────────────────────┘         │
│                                                                      │
│  MOSI:  ────┤ CMD  ├─────┤ DATA ├───────────────────────────        │
│             │ 0x02 │     │ 0xFF │                                   │
│                                                                      │
│  MISO:  ────┤ xxxx ├─────┤ xxxx ├───────────────────────────        │
│             (ignored)    (ignored)                                  │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  READ FROM SLAVE (master sends command, slave responds):           │
│                                                                      │
│  CS:    ──┐                                               ┌──       │
│           └───────────────────────────────────────────────┘         │
│                                                                      │
│  MOSI:  ────┤ CMD  ├─────┤ 0x00 ├───────────────────────────        │
│             │ 0x03 │     │ dummy│                                   │
│                                                                      │
│  MISO:  ────┤ xxxx ├─────┤ DATA ├───────────────────────────        │
│             (garbage)    │ 0x42 │                                   │
│                                                                      │
│  Master sends dummy byte to clock out slave's response             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## SPI Implementation

### Polling-Based Driver

```c
// SPI GPIO configuration
#define SPI_CS_PIN    GPIO_PIN_4   // NSS
#define SPI_SCK_PIN   GPIO_PIN_5
#define SPI_MISO_PIN  GPIO_PIN_6
#define SPI_MOSI_PIN  GPIO_PIN_7

#define CS_LOW()      GPIO_CLEAR(GPIOA, SPI_CS_PIN)
#define CS_HIGH()     GPIO_SET(GPIOA, SPI_CS_PIN)

void spi_init(void) {
    // Enable clocks
    RCC->APB2ENR |= RCC_APB2ENR_SPI1EN;
    
    // Configure SPI
    SPI1->CR1 = SPI_CR1_MSTR |        // Master mode
                SPI_CR1_BR_1 |         // Baud = fPCLK/8
                SPI_CR1_SSM |          // Software CS management
                SPI_CR1_SSI;           // Internal CS high
    // Default: Mode 0 (CPOL=0, CPHA=0), MSB first, 8-bit
    
    // Enable SPI
    SPI1->CR1 |= SPI_CR1_SPE;
    
    // Configure CS as GPIO output, start high
    CS_HIGH();
}

// Transfer single byte (send and receive)
uint8_t spi_transfer(uint8_t tx_data) {
    // Wait for TX buffer empty
    while (!(SPI1->SR & SPI_SR_TXE));
    
    // Send data
    SPI1->DR = tx_data;
    
    // Wait for RX buffer not empty
    while (!(SPI1->SR & SPI_SR_RXNE));
    
    // Return received data
    return (uint8_t)SPI1->DR;
}

// Write multiple bytes
void spi_write(uint8_t *data, uint16_t length) {
    CS_LOW();
    
    for (uint16_t i = 0; i < length; i++) {
        spi_transfer(data[i]);
    }
    
    CS_HIGH();
}

// Read multiple bytes (send dummy 0x00)
void spi_read(uint8_t *buffer, uint16_t length) {
    CS_LOW();
    
    for (uint16_t i = 0; i < length; i++) {
        buffer[i] = spi_transfer(0x00);
    }
    
    CS_HIGH();
}
```

### Flash Memory Example (W25Q)

```c
// W25Q Flash commands
#define CMD_READ_ID       0x9F
#define CMD_READ_DATA     0x03
#define CMD_WRITE_ENABLE  0x06
#define CMD_PAGE_PROGRAM  0x02
#define CMD_SECTOR_ERASE  0x20
#define CMD_READ_STATUS   0x05

// Read manufacturer ID
uint32_t flash_read_id(void) {
    uint32_t id;
    
    CS_LOW();
    spi_transfer(CMD_READ_ID);
    id  = spi_transfer(0x00) << 16;  // Manufacturer
    id |= spi_transfer(0x00) << 8;   // Device ID high
    id |= spi_transfer(0x00);        // Device ID low
    CS_HIGH();
    
    return id;  // e.g., 0xEF4016 for W25Q32
}

// Read data from flash
void flash_read(uint32_t address, uint8_t *buffer, uint16_t length) {
    CS_LOW();
    
    // Send command + 24-bit address
    spi_transfer(CMD_READ_DATA);
    spi_transfer((address >> 16) & 0xFF);
    spi_transfer((address >> 8) & 0xFF);
    spi_transfer(address & 0xFF);
    
    // Read data
    for (uint16_t i = 0; i < length; i++) {
        buffer[i] = spi_transfer(0x00);
    }
    
    CS_HIGH();
}

// Wait for write/erase to complete
void flash_wait_busy(void) {
    CS_LOW();
    spi_transfer(CMD_READ_STATUS);
    
    while (spi_transfer(0x00) & 0x01) {
        // Bit 0 = busy
    }
    
    CS_HIGH();
}
```

---

## Daisy-Chain Configuration

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DAISY-CHAIN SPI                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Data flows through all devices in series:                          │
│                                                                      │
│     MASTER           SLAVE 1         SLAVE 2         SLAVE 3        │
│  ┌──────────┐      ┌──────────┐    ┌──────────┐    ┌──────────┐    │
│  │          │ SCK ─┤──────────┼────┤──────────┼────┤──────────│    │
│  │          │      │          │    │          │    │          │    │
│  │          │ MOSI─┤─▶DIN DOUT├───▶│ DIN DOUT├───▶│ DIN DOUT │    │
│  │          │      │          │    │          │    │     │    │    │
│  │          │ MISO◀─────────────────────────────────────────┘─│    │
│  │          │      │          │    │          │    │          │    │
│  │          │ CS ──┤──────────┼────┤──────────┼────┤──────────│    │
│  └──────────┘      └──────────┘    └──────────┘    └──────────┘    │
│                                                                      │
│  All devices share same CS - data shifts through chain              │
│                                                                      │
│  TIMING (3 devices, each 8-bit):                                    │
│                                                                      │
│  CS:    ──┐                                               ┌──       │
│           └───────────────────────────────────────────────┘         │
│                                                                      │
│  MOSI:  ───┤ D3 (8b)├───┤ D2 (8b) ├───┤ D1 (8b) ├───────           │
│           First out                    Last out (to dev 1)         │
│                                                                      │
│  After 24 clocks:                                                   │
│  • Device 1 has D1                                                  │
│  • Device 2 has D2                                                  │
│  • Device 3 has D3                                                  │
│                                                                      │
│  Common uses: LED drivers (WS2801), shift registers (74HC595)      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 74HC595 Shift Register Example

```c
// 74HC595: 8-bit shift register with latch
// Data in on MOSI, latch on CS rising edge

void shift_out_595(uint8_t data) {
    CS_LOW();  // RCLK low (don't latch yet)
    
    // Shift out 8 bits
    spi_transfer(data);
    
    CS_HIGH(); // RCLK rising edge - latch data to outputs
}

// Daisy-chain 3 x 74HC595 for 24 outputs
void shift_out_24bit(uint32_t data) {
    CS_LOW();
    
    spi_transfer((data >> 16) & 0xFF);  // First (goes to last device)
    spi_transfer((data >> 8) & 0xFF);
    spi_transfer(data & 0xFF);           // Last (goes to first device)
    
    CS_HIGH();  // Latch all
}
```

---

## SPI Speed Considerations

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SPI SPEED AND TIMING                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SPI CLOCK DIVIDERS (typical):                                      │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Prescaler │ fPCLK = 48 MHz │ Use Case                      │     │
│  ├────────────────────────────────────────────────────────────┤     │
│  │    /2     │    24 MHz      │ Fast flash, displays          │     │
│  │    /4     │    12 MHz      │ Most peripherals              │     │
│  │    /8     │     6 MHz      │ Default safe speed            │     │
│  │   /16     │     3 MHz      │ Long wires                    │     │
│  │   /32     │   1.5 MHz      │ Low-power devices             │     │
│  │   /64     │   750 kHz      │ Very long cables              │     │
│  │  /128     │   375 kHz      │ Slow peripherals              │     │
│  │  /256     │   187 kHz      │ Minimum speed                 │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  TIMING CONSTRAINTS:                                                 │
│                                                                      │
│  • CS setup time: Time from CS low to first clock                  │
│  • CS hold time: Time from last clock to CS high                   │
│  • Data setup/hold: Data valid before/after clock edge             │
│                                                                      │
│  WIRE LENGTH EFFECTS:                                                │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Speed     │ Max Wire Length │ Notes                        │     │
│  ├────────────────────────────────────────────────────────────┤     │
│  │ > 10 MHz  │ < 10 cm         │ Same PCB only                │     │
│  │ 1-10 MHz  │ < 1 m           │ Shielded cable recommended   │     │
│  │ < 1 MHz   │ < 3 m           │ With proper termination      │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Feature | SPI Specification |
|---------|-------------------|
| **Type** | Synchronous, full-duplex |
| **Wires** | 4 (SCK, MOSI, MISO, CS) + 1 per slave |
| **Speed** | Up to 50+ MHz |
| **Distance** | < 1m typically |
| **Topology** | Master-slave (single master) |
| **Modes** | 4 (CPOL/CPHA combinations) |
| **Addressing** | Hardware CS lines |
| **Bit Order** | MSB or LSB first (configurable) |

---

## Quick Revision Questions

1. **What are the four SPI signals? Describe the direction of each in master/slave configuration.**

2. **Explain CPOL and CPHA. What mode has clock idle high and samples on rising edge?**

3. **Why is SPI considered full-duplex? What data does the master receive during a "write-only" operation?**

4. **A system has 5 SPI slaves. How many GPIO pins are needed for chip selects? Can they share MISO?**

5. **Describe daisy-chain SPI. How do you send different data to 3 devices in a chain?**

6. **What limits SPI cable length? How would you extend range?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Chapter 5.2: UART Protocol](02-uart-protocol.md) | [Unit Index](README.md) | [Chapter 5.4: I2C Protocol](04-i2c-protocol.md) |

---
