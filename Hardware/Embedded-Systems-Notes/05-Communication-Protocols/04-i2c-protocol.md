# Chapter 5.4: I2C Protocol

## Introduction

I2C (Inter-Integrated Circuit), also called IIC or Two-Wire Interface (TWI), is a synchronous, half-duplex, multi-master bus protocol. It uses only two wires and supports addressing for multiple devices.

---

## I2C Bus Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    I2C BUS STRUCTURE                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                 VCC (3.3V or 5V)                                    │
│                    │       │                                        │
│                   ─┴─     ─┴─                                       │
│                   │Rp│    │Rp│  Pull-up resistors (4.7kΩ typical)  │
│                   ─┬─     ─┬─                                       │
│                    │       │                                        │
│  ──────────────────┴───────┴────────────────────────── SCL         │
│        │           │       │           │           │                │
│  ──────┴───────────┴───────┴───────────┴───────────┴── SDA         │
│        │           │       │           │           │                │
│   ┌────┴────┐ ┌────┴────┐ ┌────┴────┐ ┌────┴────┐                  │
│   │ MASTER  │ │ SLAVE   │ │ SLAVE   │ │ SLAVE   │                  │
│   │ (MCU)   │ │ 0x48    │ │ 0x50    │ │ 0x68    │                  │
│   │         │ │ Sensor  │ │ EEPROM  │ │ RTC     │                  │
│   └─────────┘ └─────────┘ └─────────┘ └─────────┘                  │
│                                                                      │
│  Key Features:                                                       │
│  • Only 2 wires: SCL (clock) and SDA (data)                        │
│  • Open-drain outputs with pull-ups                                 │
│  • Multi-master capable                                             │
│  • Each device has unique address                                   │
│  • Speeds: 100 kHz (Standard), 400 kHz (Fast), 1 MHz (Fast+)       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## I2C Signal Levels

```
┌─────────────────────────────────────────────────────────────────────┐
│                    OPEN-DRAIN CONFIGURATION                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  VCC ────────────┬───────────────────────────────────── BUS         │
│                  │                                                   │
│                 ─┴─                                                  │
│                 │Rp│ Pull-up resistor                               │
│                 ─┬─                                                  │
│                  │                                                   │
│     Device 1     │     Device 2                                     │
│  ┌─────────────┐ │  ┌─────────────┐                                 │
│  │     ┌───┐   │ │  │     ┌───┐   │                                 │
│  │     │ Q │───┼─┴──┼─────│ Q │   │  Open-drain outputs            │
│  │     └─┬─┘   │    │     └─┬─┘   │                                 │
│  │       │     │    │       │     │                                 │
│  │      ─┴─    │    │      ─┴─    │                                 │
│  │     GND     │    │     GND     │                                 │
│  └─────────────┘    └─────────────┘                                 │
│                                                                      │
│  Open-drain: Transistor can only pull LOW                          │
│  Pull-up resistor pulls HIGH when all transistors are off          │
│                                                                      │
│  WIRED-AND LOGIC:                                                   │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Device 1 │ Device 2 │ Bus Level │ Result                  │     │
│  ├────────────────────────────────────────────────────────────┤     │
│  │ Release  │ Release  │   HIGH    │ Idle state              │     │
│  │ Release  │   Pull   │   LOW     │ Device 2 drives low     │     │
│  │   Pull   │ Release  │   LOW     │ Device 1 drives low     │     │
│  │   Pull   │   Pull   │   LOW     │ Both drive low          │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  Any device can pull line LOW (dominant)                            │
│  Line goes HIGH only when ALL devices release                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## I2C Protocol Timing

```
┌─────────────────────────────────────────────────────────────────────┐
│                    START AND STOP CONDITIONS                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  START Condition: SDA goes LOW while SCL is HIGH                    │
│                                                                      │
│  SCL:  ─────────────────┐             ┌─────────────                │
│                         └─────────────┘                             │
│                         │                                           │
│  SDA:  ────────────┐    │                                           │
│                    └────┼──────────────────────────                 │
│                    ↑    │                                           │
│                  START  │                                           │
│                         │                                           │
│                    First data bit after START                       │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  STOP Condition: SDA goes HIGH while SCL is HIGH                    │
│                                                                      │
│  SCL:          ┌────────────────────────────────────                │
│       ─────────┘                                                    │
│                │                                                    │
│  SDA:          │    ┌───────────────────────────────                │
│       ─────────┴────┘                                               │
│                     ↑                                               │
│                   STOP (bus released)                               │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  REPEATED START: New transaction without releasing bus             │
│                                                                      │
│  SCL:   ───┘ └───┐     ┌───┘ └───                                  │
│              └───┘                                                  │
│                   │                                                 │
│  SDA:   ─────────┐│   ┌──────────                                   │
│                  └┴───┘                                             │
│                   ↑                                                 │
│               REPEATED START                                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## I2C Data Transfer

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DATA BYTE FORMAT                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Each byte followed by ACK/NAK bit (9 clocks per byte)             │
│                                                                      │
│  SCL:   ─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌─                      │
│          └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘ └─┘                        │
│          1   2   3   4   5   6   7   8   9                          │
│                                                                      │
│  SDA:   ─╪───╪───╪───╪───╪───╪───╪───╪───╪───                       │
│         D7  D6  D5  D4  D3  D2  D1  D0  ACK                         │
│         MSB first                       │                           │
│                                         └── Receiver drives:       │
│                                             LOW = ACK               │
│                                             HIGH = NAK              │
│                                                                      │
│  ACK/NAK DETAIL:                                                    │
│                                                                      │
│  After 8 data bits, transmitter releases SDA                       │
│  Receiver pulls SDA LOW for ACK, or leaves HIGH for NAK            │
│                                                                      │
│  SCL:        ┌─────────┐                                            │
│  ────────────┘         └────────                                    │
│              ↑ sample                                               │
│  SDA:    ────┴───┐                                                  │
│       (TX)    (RX└──────  ACK: receiver pulls low                   │
│                                                                      │
│  SDA:    ────┴───────────                                           │
│       (TX)      (RX)      NAK: receiver leaves high                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## I2C Addressing

```
┌─────────────────────────────────────────────────────────────────────┐
│                    7-BIT ADDRESSING                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  First byte after START contains 7-bit address + R/W bit           │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Bit 7 │ Bit 6 │ Bit 5 │ Bit 4 │ Bit 3 │ Bit 2 │ Bit 1 │ Bit 0│   │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │  A6   │  A5   │  A4   │  A3   │  A2   │  A1   │  A0   │ R/W │    │
│  │       7-bit slave address                     │ 0=W  │      │    │
│  │                                               │ 1=R  │      │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  EXAMPLE: EEPROM at address 0x50                                    │
│                                                                      │
│  Write: (0x50 << 1) | 0 = 0xA0 = 1010 0000                         │
│  Read:  (0x50 << 1) | 1 = 0xA1 = 1010 0001                         │
│                                                                      │
│  COMPLETE WRITE TRANSACTION:                                        │
│                                                                      │
│       S    Address + W  ACK   Data     ACK   Data    ACK  P        │
│       ┌─┬─────────────┬───┬──────────┬───┬─────────┬───┬─┐         │
│  SDA: │ │ 1010000  0  │ 0 │xxxxxxxx  │ 0 │xxxxxxxx │ 0 │ │         │
│       └─┴─────────────┴───┴──────────┴───┴─────────┴───┴─┘         │
│       │       │         │      │       │      │      │  │          │
│     START  address    slave  data    slave  data   slave STOP      │
│            +write     ACK    byte    ACK    byte   ACK             │
│                                                                      │
│  RESERVED ADDRESSES:                                                 │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Address  │ Purpose                                         │     │
│  ├────────────────────────────────────────────────────────────┤     │
│  │ 0x00     │ General call (broadcast)                        │     │
│  │ 0x01     │ CBUS address                                    │     │
│  │ 0x02     │ Reserved for different bus format              │     │
│  │ 0x03     │ Reserved for future purposes                   │     │
│  │ 0x04-0x07│ High-speed mode master code                    │     │
│  │ 0x78-0x7B│ 10-bit addressing                              │     │
│  │ 0x7C-0x7F│ Reserved for future purposes                   │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## I2C Read/Write Operations

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TRANSACTION PATTERNS                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SIMPLE WRITE (e.g., set LED brightness):                          │
│                                                                      │
│  S │ ADDR+W │ A │ CMD  │ A │ DATA │ A │ P                          │
│    │  1byte │   │ 1byte│   │1byte │   │                            │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  SIMPLE READ (e.g., read temperature):                             │
│                                                                      │
│  S │ ADDR+R │ A │ DATA │ A │ DATA │ N │ P                          │
│    │  1byte │   │1byte │   │ 1byte│   │                            │
│                                         └── Master sends NAK       │
│                                             before last byte        │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  REGISTER READ (common pattern):                                    │
│  First write register address, then read data                      │
│                                                                      │
│  S │ ADDR+W │ A │ REG  │ A │ Sr │ ADDR+R │ A │ DATA │ N │ P        │
│    │        │   │ addr │   │    │        │   │      │   │          │
│                           └────────────────────────────────         │
│                              REPEATED START                         │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  EEPROM PAGE WRITE (AT24C02):                                       │
│                                                                      │
│  S │ ADDR+W │ A │ MEM  │ A │ D0  │ A │ D1  │ A │...│ Dn │ A │ P    │
│    │  0x50  │   │ addr │   │     │   │     │   │   │    │   │      │
│              (write up to 8 bytes in one page)                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## I2C Implementation

### Basic I2C Driver

```c
#define I2C_TIMEOUT  10000

// I2C initialization
void i2c_init(void) {
    // Enable clocks
    RCC->APB1ENR |= RCC_APB1ENR_I2C1EN;
    
    // Configure for 100 kHz (standard mode)
    // Assuming PCLK = 36 MHz
    I2C1->CR2 = 36;  // FREQ = PCLK in MHz
    
    // CCR = PCLK / (2 * I2C_speed)
    // For 100 kHz: CCR = 36,000,000 / (2 * 100,000) = 180
    I2C1->CCR = 180;
    
    // TRISE = (max_rise_time / PCLK_period) + 1
    // For standard mode: 1000ns / 27.8ns + 1 = 37
    I2C1->TRISE = 37;
    
    // Enable I2C
    I2C1->CR1 |= I2C_CR1_PE;
}

// Generate START condition
bool i2c_start(void) {
    I2C1->CR1 |= I2C_CR1_START;
    
    uint32_t timeout = I2C_TIMEOUT;
    while (!(I2C1->SR1 & I2C_SR1_SB) && timeout--);
    
    return timeout > 0;
}

// Send slave address
bool i2c_send_address(uint8_t address, bool read) {
    uint8_t addr_byte = (address << 1) | (read ? 1 : 0);
    I2C1->DR = addr_byte;
    
    uint32_t timeout = I2C_TIMEOUT;
    while (!(I2C1->SR1 & I2C_SR1_ADDR) && timeout--) {
        if (I2C1->SR1 & I2C_SR1_AF) {
            // Address not acknowledged
            I2C1->SR1 &= ~I2C_SR1_AF;
            return false;
        }
    }
    
    // Clear ADDR flag by reading SR1 and SR2
    (void)I2C1->SR1;
    (void)I2C1->SR2;
    
    return timeout > 0;
}

// Write single byte
bool i2c_write_byte(uint8_t data) {
    I2C1->DR = data;
    
    uint32_t timeout = I2C_TIMEOUT;
    while (!(I2C1->SR1 & I2C_SR1_BTF) && timeout--);
    
    return timeout > 0;
}

// Read single byte with ACK
uint8_t i2c_read_byte_ack(void) {
    I2C1->CR1 |= I2C_CR1_ACK;  // Enable ACK
    
    uint32_t timeout = I2C_TIMEOUT;
    while (!(I2C1->SR1 & I2C_SR1_RXNE) && timeout--);
    
    return I2C1->DR;
}

// Read single byte with NAK (last byte)
uint8_t i2c_read_byte_nak(void) {
    I2C1->CR1 &= ~I2C_CR1_ACK;  // Disable ACK
    
    uint32_t timeout = I2C_TIMEOUT;
    while (!(I2C1->SR1 & I2C_SR1_RXNE) && timeout--);
    
    return I2C1->DR;
}

// Generate STOP condition
void i2c_stop(void) {
    I2C1->CR1 |= I2C_CR1_STOP;
}
```

### Sensor Read Example

```c
// Read temperature from LM75 sensor (address 0x48)
#define LM75_ADDR     0x48
#define LM75_TEMP_REG 0x00

int16_t lm75_read_temperature(void) {
    int16_t temp;
    uint8_t data[2];
    
    // Start + address + register
    i2c_start();
    i2c_send_address(LM75_ADDR, false);  // Write mode
    i2c_write_byte(LM75_TEMP_REG);
    
    // Repeated start + read
    i2c_start();
    i2c_send_address(LM75_ADDR, true);   // Read mode
    
    data[0] = i2c_read_byte_ack();   // MSB
    data[1] = i2c_read_byte_nak();   // LSB (NAK for last byte)
    i2c_stop();
    
    // Temperature in 0.5°C units (11-bit, left-justified)
    temp = (data[0] << 8) | data[1];
    temp >>= 5;  // Right-align 11 bits
    
    // Convert to 0.1°C units
    return (temp * 10) / 2;  // Returns 250 for 25.0°C
}
```

---

## Clock Stretching

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CLOCK STRETCHING                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Slave can hold SCL LOW to pause master while processing           │
│                                                                      │
│  NORMAL:           Master releases SCL, slave responds immediately │
│                                                                      │
│  SCL (M):  ────┘   └─────┘   └─────┘   └──                         │
│  SCL (S):  ────────────────────────────────  (no stretching)       │
│  SCL (bus):────┘   └─────┘   └─────┘   └──                         │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  WITH STRETCH:     Slave holds SCL low while processing            │
│                                                                      │
│  SCL (M):  ────┘  └──────┘  └──────┘  └──                           │
│                      │                                              │
│  SCL (S):  ─────────┐└──────────────────────                       │
│                     └────┐                                          │
│                          │ Slave holds low                          │
│  SCL (bus):────┘  └──────┴───┘  └──────┘  └──                      │
│                     ▲                                               │
│                     │ Stretched                                     │
│                                                                      │
│  Master detects stretched clock by:                                 │
│  1. Releasing SCL (trying to go high)                              │
│  2. Reading SCL pin - if still low, slave is stretching           │
│  3. Waiting until SCL goes high                                    │
│                                                                      │
│  Some masters have timeout for clock stretch detection             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Multi-Master and Arbitration

```
┌─────────────────────────────────────────────────────────────────────┐
│                    I2C ARBITRATION                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  When multiple masters try to transmit simultaneously:             │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ MASTER A │ MASTER B │ BUS     │ Result                      │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │   1 (H)  │   1 (H)  │   1     │ Both continue              │    │
│  │   1 (H)  │   0 (L)  │   0     │ A loses (detects mismatch) │    │
│  │   0 (L)  │   1 (H)  │   0     │ B loses (detects mismatch) │    │
│  │   0 (L)  │   0 (L)  │   0     │ Both continue              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ARBITRATION EXAMPLE:                                                │
│                                                                      │
│  Master A sends: 1 0 1 0 0 1 0 0  (address 0x52)                   │
│  Master B sends: 1 0 1 0 0 1 1 0  (address 0x53)                   │
│                          ↑   ↑                                      │
│                    Same  │   │  A sends 0, B sends 1               │
│                          │   │  Bus = 0 (A wins)                   │
│                          │   │  B reads 0, expected 1              │
│                          │   │  B backs off                        │
│                          │   │                                      │
│  Bus result:       1 0 1 0 0 1 0 0                                 │
│                                 │                                   │
│                         Master A continues                          │
│                         Master B retries later                      │
│                                                                      │
│  Key point: Lower address wins arbitration!                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## I2C Speed Modes

| Mode | Max Speed | Rise Time | Use Case |
|------|-----------|-----------|----------|
| **Standard** | 100 kHz | 1000 ns | Legacy devices |
| **Fast** | 400 kHz | 300 ns | Most sensors |
| **Fast+** | 1 MHz | 120 ns | Fast peripherals |
| **High-Speed** | 3.4 MHz | Variable | Memory, displays |

### Pull-up Resistor Selection

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PULL-UP RESISTOR CALCULATION                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Constraints:                                                        │
│  • Minimum Rp: Limited by I/O sink current (typically 3mA)         │
│  • Maximum Rp: Limited by rise time requirement                    │
│                                                                      │
│  Rp_min = (VCC - VOL) / IOL                                        │
│  For 3.3V: Rp_min = (3.3 - 0.4) / 0.003 = 967Ω                    │
│                                                                      │
│  Rp_max = tr / (0.8473 × Cb)                                       │
│  Where: tr = rise time, Cb = bus capacitance                       │
│                                                                      │
│  TYPICAL VALUES:                                                     │
│  ┌──────────────────────────────────────────────────────────┐       │
│  │ Speed      │ VCC   │ Cb = 100pF │ Cb = 400pF │          │       │
│  ├──────────────────────────────────────────────────────────┤       │
│  │ 100 kHz    │ 3.3V  │  4.7 kΩ    │   2.2 kΩ   │          │       │
│  │ 400 kHz    │ 3.3V  │  2.2 kΩ    │   1.0 kΩ   │          │       │
│  │ 1 MHz      │ 3.3V  │  1.0 kΩ    │   470 Ω    │          │       │
│  └──────────────────────────────────────────────────────────┘       │
│                                                                      │
│  Higher speed → Lower resistance → Faster rise time                │
│  More devices → Higher capacitance → Lower resistance needed       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Common I2C Devices

| Device | Address | Description |
|--------|---------|-------------|
| **EEPROM AT24C02** | 0x50-0x57 | 256 bytes storage |
| **RTC DS3231** | 0x68 | Real-time clock |
| **Temp LM75** | 0x48-0x4F | Temperature sensor |
| **Accel MPU6050** | 0x68/0x69 | 6-axis IMU |
| **OLED SSD1306** | 0x3C/0x3D | 128x64 display |
| **ADC ADS1115** | 0x48-0x4B | 16-bit ADC |
| **I/O MCP23017** | 0x20-0x27 | 16 GPIO expander |

---

## Summary Table

| Feature | I2C Specification |
|---------|-------------------|
| **Type** | Synchronous, half-duplex |
| **Wires** | 2 (SDA, SCL) |
| **Speed** | 100 kHz to 3.4 MHz |
| **Distance** | < 1m (standard) |
| **Topology** | Multi-master, multi-slave |
| **Addressing** | 7-bit (127 devices) or 10-bit |
| **Acknowledgment** | Hardware ACK/NAK |
| **Output** | Open-drain with pull-ups |

---

## Quick Revision Questions

1. **Why does I2C use open-drain outputs with pull-up resistors? What is wired-AND logic?**

2. **Draw the timing for START and STOP conditions. What makes them unique on the I2C bus?**

3. **Explain the 9-clock byte transfer. Who drives SDA during the 9th clock?**

4. **A device address is 0x48. What byte is sent for a read operation? For write?**

5. **Describe clock stretching. Why is it useful? What problems can it cause?**

6. **How does I2C handle arbitration when two masters transmit simultaneously?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Chapter 5.3: SPI Protocol](03-spi-protocol.md) | [Unit Index](README.md) | [Chapter 5.5: Advanced Protocols](05-advanced-protocols.md) |

---
