# Chapter 10.4: Communication Projects

## 📚 Chapter Overview

This chapter covers practical communication interface projects including UART serial communication, I2C multi-device systems, SPI interfaces, and wireless module integration.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Implement UART serial communication
- Design I2C multi-device networks
- Interface SPI devices
- Integrate wireless modules (Bluetooth, WiFi)
- Implement communication protocols

---

## 1. UART Serial Communication

### 1.1 RS-232 Serial Interface

```
RS-232 SERIAL COMMUNICATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━

8051 SERIAL COMMUNICATION:
─────────────────────────

    8051 MCU                    MAX232              PC/Terminal
    ┌───────┐                  ┌─────────┐          ┌──────────┐
    │       │   TTL Levels     │         │  RS-232  │          │
    │  TXD  │──────────────────┤ T1IN    │          │          │
    │ (P3.1)│                  │      T1OUT├─────────┤ RXD (Pin2)
    │       │                  │         │          │          │
    │  RXD  │──────────────────┤ R1OUT   │          │          │
    │ (P3.0)│                  │      R1IN├─────────┤ TXD (Pin3)
    │       │                  │         │          │          │
    └───────┘                  │   GND   │          │ GND (Pin5)
                               └────┬────┘          └──────────┘
                                    │
                                   GND

MAX232 PIN CONFIGURATION:
─────────────────────────

                ┌──────────────┐
        C1+ ───┤1           16├─── VCC (+5V)
        VS+ ───┤2           15├─── GND
        C1- ───┤3           14├─── T1OUT (to PC RX)
        C2+ ───┤4           13├─── R1IN (from PC TX)
        C2- ───┤5           12├─── R1OUT (to 8051 RXD)
        VS- ───┤6           11├─── T1IN (from 8051 TXD)
       T2OUT───┤7           10├─── T2IN
       R2IN ───┤8            9├─── R2OUT
                └──────────────┘

Capacitors: C1, C2, C3, C4 = 1µF to 10µF electrolytic

UART FRAME FORMAT:
─────────────────

Idle ─────┐   ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐   ┌─── Idle
          │   │   │   │   │   │   │   │   │   │   │   │   │
          └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
         Start D0  D1  D2  D3  D4  D5  D6  D7  P  Stop
         Bit                                   (opt)

• Start bit: Always 0 (marks beginning)
• Data bits: 8 bits (LSB first)
• Parity: Optional (even/odd)
• Stop bit: Always 1 (can be 1 or 2)
```

### 1.2 UART Implementation Code

```c
/*
 * 8051 UART SERIAL COMMUNICATION
 * 9600 baud, 8-N-1
 */

#include <reg51.h>
#include <stdio.h>

#define BAUD_9600   0xFD  // Timer reload for 9600 @ 11.0592 MHz

/*
 * UART INITIALIZATION
 */
void uart_init(void)
{
    SCON = 0x50;    // Mode 1, 8-bit, REN enabled
    TMOD |= 0x20;   // Timer 1 Mode 2 (auto-reload)
    TH1 = BAUD_9600;
    TL1 = BAUD_9600;
    TR1 = 1;        // Start Timer 1
    TI = 1;         // Enable first transmission
}

/*
 * SEND SINGLE CHARACTER
 */
void uart_tx_char(unsigned char ch)
{
    while(!TI);     // Wait for previous transmission
    TI = 0;         // Clear flag
    SBUF = ch;      // Load data
}

/*
 * RECEIVE SINGLE CHARACTER
 */
unsigned char uart_rx_char(void)
{
    while(!RI);     // Wait for reception
    RI = 0;         // Clear flag
    return SBUF;    // Return received data
}

/*
 * SEND STRING
 */
void uart_tx_string(char *str)
{
    while(*str)
    {
        uart_tx_char(*str++);
    }
}

/*
 * RECEIVE STRING (until newline)
 */
void uart_rx_string(char *buf, unsigned char max_len)
{
    unsigned char i = 0;
    char ch;
    
    while(i < max_len - 1)
    {
        ch = uart_rx_char();
        
        if(ch == '\r' || ch == '\n')
            break;
            
        buf[i++] = ch;
        uart_tx_char(ch);  // Echo
    }
    
    buf[i] = '\0';
}

/*
 * PRINTF REDIRECT (for standard printf)
 */
char putchar(char c)
{
    uart_tx_char(c);
    return c;
}

/*
 * SEND FORMATTED NUMBER
 */
void uart_tx_number(int num)
{
    char buf[16];
    sprintf(buf, "%d", num);
    uart_tx_string(buf);
}

void main(void)
{
    char buffer[64];
    int temp;
    
    uart_init();
    adc_init();
    
    uart_tx_string("\r\n=== 8051 Temperature Logger ===\r\n");
    uart_tx_string("Reading temperature every second...\r\n\n");
    
    while(1)
    {
        // Read temperature
        temp = read_temperature();
        
        // Send to PC
        uart_tx_string("Temperature: ");
        uart_tx_number(temp);
        uart_tx_string(" C\r\n");
        
        delay_ms(1000);
    }
}
```

### 1.3 Interrupt-Driven UART

```c
/*
 * INTERRUPT-DRIVEN UART
 * Non-blocking communication
 */

#include <reg51.h>

// Circular buffers
#define BUF_SIZE 64
volatile unsigned char tx_buf[BUF_SIZE];
volatile unsigned char rx_buf[BUF_SIZE];
volatile unsigned char tx_head = 0, tx_tail = 0;
volatile unsigned char rx_head = 0, rx_tail = 0;
volatile bit tx_busy = 0;

/*
 * UART ISR - Handles both TX and RX
 */
void uart_isr(void) interrupt 4
{
    // Reception complete
    if(RI)
    {
        RI = 0;
        unsigned char next = (rx_head + 1) % BUF_SIZE;
        
        if(next != rx_tail)  // Buffer not full
        {
            rx_buf[rx_head] = SBUF;
            rx_head = next;
        }
    }
    
    // Transmission complete
    if(TI)
    {
        TI = 0;
        
        if(tx_head != tx_tail)  // More data to send
        {
            SBUF = tx_buf[tx_tail];
            tx_tail = (tx_tail + 1) % BUF_SIZE;
        }
        else
        {
            tx_busy = 0;  // Transmission complete
        }
    }
}

/*
 * QUEUE CHARACTER FOR TRANSMISSION
 */
void uart_tx_queue(unsigned char ch)
{
    unsigned char next = (tx_head + 1) % BUF_SIZE;
    
    while(next == tx_tail);  // Wait if buffer full
    
    tx_buf[tx_head] = ch;
    tx_head = next;
    
    // Start transmission if not busy
    if(!tx_busy)
    {
        tx_busy = 1;
        TI = 1;  // Trigger first transmission
    }
}

/*
 * CHECK IF DATA AVAILABLE
 */
bit uart_rx_available(void)
{
    return (rx_head != rx_tail);
}

/*
 * READ RECEIVED CHARACTER
 */
unsigned char uart_rx_read(void)
{
    unsigned char ch;
    
    while(!uart_rx_available());  // Wait for data
    
    ch = rx_buf[rx_tail];
    rx_tail = (rx_tail + 1) % BUF_SIZE;
    
    return ch;
}

void uart_init(void)
{
    SCON = 0x50;
    TMOD |= 0x20;
    TH1 = 0xFD;  // 9600 baud
    TR1 = 1;
    ES = 1;      // Enable serial interrupt
    EA = 1;      // Enable global interrupts
}
```

---

## 2. I2C Multi-Device Communication

### 2.1 I2C Bus Overview

```
I2C BUS ARCHITECTURE
━━━━━━━━━━━━━━━━━━━

I2C MULTI-DEVICE CONNECTION:
──────────────────────────

         VCC (+3.3V/5V)
          │     │
          │     │
         ┌┴┐   ┌┴┐
         │R│   │R│  Pull-up resistors (4.7kΩ)
         │ │   │ │
         └┬┘   └┬┘
          │     │
    SDA ──┼─────┼──────┬────────┬────────┬────────
          │     │      │        │        │
    SCL ──┼─────┼──────┼────────┼────────┼────────
          │     │      │        │        │
       ┌──┴──┐  │   ┌──┴──┐  ┌──┴──┐  ┌──┴──┐
       │     │  │   │     │  │     │  │     │
       │8051 │  │   │EEPROM│  │ RTC │  │ LCD │
       │Master  │   │24C32│  │DS1307│  │PCF8574
       │     │  │   │0x50 │  │0x68 │  │0x27 │
       └─────┘  │   └─────┘  └─────┘  └─────┘
                │
               GND

I2C TIMING DIAGRAM:
──────────────────

         │←─ Start ─→│←──── Address + R/W ────→│←─ ACK─→│
         │           │                          │        │
SDA ─────┐    ┌───┬───┬───┬───┬───┬───┬───┬───┐   ┌───┐   
         │    │ A6│ A5│ A4│ A3│ A2│ A1│ A0│R/W│   │ 0 │  
         └────┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───

SCL ───────┐  ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───────
           │  │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │ 8 │ 9 │
           └──┴───┴───┴───┴───┴───┴───┴───┴───┴───┘

Start: SDA ↓ while SCL HIGH
Stop:  SDA ↑ while SCL HIGH
Data:  SDA stable while SCL HIGH

COMMON I2C ADDRESSES:
───────────────────

Device              │ Address (7-bit)
────────────────────┼────────────────
24C32 EEPROM        │ 0x50 - 0x57
DS1307 RTC          │ 0x68
PCF8574 I/O         │ 0x20 - 0x27
SSD1306 OLED        │ 0x3C or 0x3D
BMP180 Pressure     │ 0x77
MPU6050 IMU         │ 0x68 or 0x69
```

### 2.2 I2C Master Implementation

```c
/*
 * SOFTWARE I2C MASTER
 * For 8051 microcontroller
 */

#include <reg51.h>

sbit SDA = P2^0;
sbit SCL = P2^1;

#define I2C_DELAY  5  // Adjust for bus speed

void i2c_delay(void)
{
    unsigned char i;
    for(i = 0; i < I2C_DELAY; i++);
}

/*
 * I2C START CONDITION
 */
void i2c_start(void)
{
    SDA = 1;
    SCL = 1;
    i2c_delay();
    SDA = 0;    // SDA low while SCL high
    i2c_delay();
    SCL = 0;
    i2c_delay();
}

/*
 * I2C STOP CONDITION
 */
void i2c_stop(void)
{
    SDA = 0;
    SCL = 1;
    i2c_delay();
    SDA = 1;    // SDA high while SCL high
    i2c_delay();
}

/*
 * I2C WRITE BYTE
 * Returns ACK status (0 = ACK, 1 = NACK)
 */
bit i2c_write(unsigned char dat)
{
    unsigned char i;
    bit ack;
    
    for(i = 0; i < 8; i++)
    {
        SDA = (dat & 0x80) ? 1 : 0;
        dat <<= 1;
        
        SCL = 1;
        i2c_delay();
        SCL = 0;
        i2c_delay();
    }
    
    // Read ACK
    SDA = 1;    // Release SDA
    SCL = 1;
    i2c_delay();
    ack = SDA;  // Read ACK (0 = ACK)
    SCL = 0;
    i2c_delay();
    
    return ack;
}

/*
 * I2C READ BYTE
 * ack: 0 = send ACK, 1 = send NACK
 */
unsigned char i2c_read(bit ack)
{
    unsigned char i, dat = 0;
    
    SDA = 1;    // Release SDA
    
    for(i = 0; i < 8; i++)
    {
        dat <<= 1;
        SCL = 1;
        i2c_delay();
        dat |= SDA;
        SCL = 0;
        i2c_delay();
    }
    
    // Send ACK/NACK
    SDA = ack;
    SCL = 1;
    i2c_delay();
    SCL = 0;
    i2c_delay();
    SDA = 1;
    
    return dat;
}

/*
 * I2C DEVICE SCAN
 */
void i2c_scan(void)
{
    unsigned char addr;
    unsigned char count = 0;
    
    uart_tx_string("\r\nI2C Device Scanner\r\n");
    uart_tx_string("─────────────────────\r\n");
    
    for(addr = 0x08; addr < 0x78; addr++)
    {
        i2c_start();
        if(i2c_write(addr << 1) == 0)  // ACK received
        {
            uart_tx_string("Device found at 0x");
            uart_tx_hex(addr);
            uart_tx_string("\r\n");
            count++;
        }
        i2c_stop();
    }
    
    uart_tx_string("\r\nTotal devices: ");
    uart_tx_number(count);
    uart_tx_string("\r\n");
}
```

### 2.3 24C32 EEPROM Interface

```c
/*
 * 24C32 I2C EEPROM DRIVER
 * 32 Kbit (4K × 8) EEPROM
 */

#define EEPROM_ADDR  0x50  // A0, A1, A2 = 0

/*
 * WRITE SINGLE BYTE
 */
bit eeprom_write_byte(unsigned int addr, unsigned char dat)
{
    i2c_start();
    
    if(i2c_write(EEPROM_ADDR << 1))  // Device address + Write
    {
        i2c_stop();
        return 1;  // No ACK - device not responding
    }
    
    i2c_write(addr >> 8);    // Address high byte
    i2c_write(addr & 0xFF);  // Address low byte
    i2c_write(dat);          // Data
    
    i2c_stop();
    
    // Wait for internal write cycle (~5ms)
    delay_ms(5);
    
    return 0;
}

/*
 * READ SINGLE BYTE
 */
unsigned char eeprom_read_byte(unsigned int addr)
{
    unsigned char dat;
    
    // Set address
    i2c_start();
    i2c_write(EEPROM_ADDR << 1);      // Write mode
    i2c_write(addr >> 8);
    i2c_write(addr & 0xFF);
    
    // Read data
    i2c_start();                       // Repeated start
    i2c_write((EEPROM_ADDR << 1) | 1); // Read mode
    dat = i2c_read(1);                 // NACK (last byte)
    i2c_stop();
    
    return dat;
}

/*
 * WRITE PAGE (up to 32 bytes)
 */
bit eeprom_write_page(unsigned int addr, unsigned char *buf, 
                      unsigned char len)
{
    unsigned char i;
    
    // Page boundary check
    if((addr % 32) + len > 32)
        return 1;  // Would cross page boundary
    
    i2c_start();
    i2c_write(EEPROM_ADDR << 1);
    i2c_write(addr >> 8);
    i2c_write(addr & 0xFF);
    
    for(i = 0; i < len; i++)
        i2c_write(buf[i]);
    
    i2c_stop();
    delay_ms(5);  // Write cycle
    
    return 0;
}

/*
 * READ SEQUENTIAL (any length)
 */
void eeprom_read_sequential(unsigned int addr, unsigned char *buf,
                            unsigned int len)
{
    unsigned int i;
    
    i2c_start();
    i2c_write(EEPROM_ADDR << 1);
    i2c_write(addr >> 8);
    i2c_write(addr & 0xFF);
    
    i2c_start();
    i2c_write((EEPROM_ADDR << 1) | 1);
    
    for(i = 0; i < len - 1; i++)
        buf[i] = i2c_read(0);  // ACK
    
    buf[len-1] = i2c_read(1);  // NACK on last byte
    
    i2c_stop();
}

/*
 * EXAMPLE: Data Logger
 */
void data_logger_example(void)
{
    unsigned int log_addr = 0;
    unsigned char log_data[8];
    
    while(1)
    {
        // Read sensor data
        log_data[0] = read_temperature();
        log_data[1] = read_humidity();
        log_data[2] = read_light();
        log_data[3] = get_hours();
        log_data[4] = get_minutes();
        log_data[5] = get_seconds();
        
        // Store to EEPROM
        eeprom_write_page(log_addr, log_data, 6);
        log_addr += 8;  // Next record (8-byte aligned)
        
        if(log_addr >= 4096)  // 4K EEPROM full
            log_addr = 0;
        
        delay_ms(60000);  // Log every minute
    }
}
```

### 2.4 DS1307 RTC Interface

```c
/*
 * DS1307 I2C RTC DRIVER
 */

#define RTC_ADDR  0x68

typedef struct {
    unsigned char second;
    unsigned char minute;
    unsigned char hour;
    unsigned char day;
    unsigned char date;
    unsigned char month;
    unsigned char year;
} RTC_Time;

/*
 * BCD CONVERSION
 */
unsigned char bcd_to_dec(unsigned char bcd)
{
    return ((bcd >> 4) * 10) + (bcd & 0x0F);
}

unsigned char dec_to_bcd(unsigned char dec)
{
    return ((dec / 10) << 4) | (dec % 10);
}

/*
 * READ TIME FROM RTC
 */
void rtc_read_time(RTC_Time *t)
{
    i2c_start();
    i2c_write(RTC_ADDR << 1);  // Write mode
    i2c_write(0x00);           // Start register
    
    i2c_start();
    i2c_write((RTC_ADDR << 1) | 1);  // Read mode
    
    t->second = bcd_to_dec(i2c_read(0) & 0x7F);
    t->minute = bcd_to_dec(i2c_read(0));
    t->hour   = bcd_to_dec(i2c_read(0) & 0x3F);
    t->day    = bcd_to_dec(i2c_read(0));
    t->date   = bcd_to_dec(i2c_read(0));
    t->month  = bcd_to_dec(i2c_read(0));
    t->year   = bcd_to_dec(i2c_read(1));  // NACK last byte
    
    i2c_stop();
}

/*
 * SET TIME IN RTC
 */
void rtc_set_time(RTC_Time *t)
{
    i2c_start();
    i2c_write(RTC_ADDR << 1);
    i2c_write(0x00);
    
    i2c_write(dec_to_bcd(t->second) & 0x7F);  // Clear CH bit
    i2c_write(dec_to_bcd(t->minute));
    i2c_write(dec_to_bcd(t->hour));
    i2c_write(dec_to_bcd(t->day));
    i2c_write(dec_to_bcd(t->date));
    i2c_write(dec_to_bcd(t->month));
    i2c_write(dec_to_bcd(t->year));
    
    i2c_stop();
}

/*
 * DISPLAY TIME ON LCD
 */
void rtc_display_time(void)
{
    RTC_Time t;
    char buf[17];
    
    rtc_read_time(&t);
    
    lcd_goto(0, 0);
    sprintf(buf, "%02d:%02d:%02d", t.hour, t.minute, t.second);
    lcd_string(buf);
    
    lcd_goto(1, 0);
    sprintf(buf, "%02d/%02d/20%02d", t.date, t.month, t.year);
    lcd_string(buf);
}
```

---

## 3. SPI Communication

### 3.1 SPI Basics

```
SPI BUS COMMUNICATION
━━━━━━━━━━━━━━━━━━━━━

SPI CONNECTIONS:
──────────────

    Master (8051)              Slave (e.g., SD Card)
    ┌───────────┐              ┌───────────┐
    │           │    MOSI      │           │
    │    P1.0 ──┼──────────────┤►DI        │
    │    (MOSI) │              │           │
    │           │    MISO      │           │
    │    P1.1 ◄─┼──────────────┼─DO        │
    │    (MISO) │              │           │
    │           │    SCK       │           │
    │    P1.2 ──┼──────────────┤►CLK       │
    │    (SCK)  │              │           │
    │           │    CS        │           │
    │    P1.3 ──┼──────────────┤►CS        │
    │    (SS)   │              │           │
    └───────────┘              └───────────┘

SPI TIMING (Mode 0: CPOL=0, CPHA=0):
─────────────────────────────────

CS   ────┐                                    ┌────
         └────────────────────────────────────┘

SCK  ────┐   ┌───┐   ┌───┐   ┌───┐   ┌───┐   ┌────
         └───┘   └───┘   └───┘   └───┘   └───┘

MOSI ─────╔═══╗───╔═══╗───╔═══╗───╔═══╗───╔═══╗────
         ║ D7║   ║ D6║   ║ D5║   ║...║   ║ D0║
         ╚═══╝───╚═══╝───╚═══╝───╚═══╝───╚═══╝────

MISO ─────╔═══╗───╔═══╗───╔═══╗───╔═══╗───╔═══╗────
         ║ D7║   ║ D6║   ║ D5║   ║...║   ║ D0║
         ╚═══╝───╚═══╝───╚═══╝───╚═══╝───╚═══╝────
              ↑       ↑       ↑       ↑       ↑
              │       │       │       │       │
         Data sampled on rising edge

SPI MODES:
─────────

Mode │ CPOL │ CPHA │ Description
─────┼──────┼──────┼─────────────────────────────
  0  │   0  │   0  │ Clock idle low, sample rising
  1  │   0  │   1  │ Clock idle low, sample falling
  2  │   1  │   0  │ Clock idle high, sample falling
  3  │   1  │   1  │ Clock idle high, sample rising
```

### 3.2 SPI Implementation

```c
/*
 * SOFTWARE SPI MASTER
 * Mode 0 (CPOL=0, CPHA=0)
 */

#include <reg51.h>

sbit MOSI = P1^0;
sbit MISO = P1^1;
sbit SCK  = P1^2;
sbit CS   = P1^3;

void spi_delay(void)
{
    unsigned char i;
    for(i = 0; i < 2; i++);
}

/*
 * SPI TRANSFER BYTE
 * Returns received byte while sending
 */
unsigned char spi_transfer(unsigned char dat)
{
    unsigned char i, rx = 0;
    
    for(i = 0; i < 8; i++)
    {
        MOSI = (dat & 0x80) ? 1 : 0;  // MSB first
        dat <<= 1;
        
        SCK = 1;
        spi_delay();
        
        rx <<= 1;
        rx |= MISO;
        
        SCK = 0;
        spi_delay();
    }
    
    return rx;
}

/*
 * SPI WRITE ONLY
 */
void spi_write(unsigned char dat)
{
    spi_transfer(dat);
}

/*
 * SPI READ ONLY (send dummy byte)
 */
unsigned char spi_read(void)
{
    return spi_transfer(0xFF);
}

void spi_init(void)
{
    SCK = 0;
    CS = 1;   // Deselect
    MOSI = 1;
}
```

### 3.3 SPI SD Card Interface

```c
/*
 * SD CARD SPI INTERFACE
 * Basic read/write operations
 */

#define SD_CS  CS

// SD Commands
#define CMD0   0   // GO_IDLE_STATE
#define CMD8   8   // SEND_IF_COND
#define CMD17  17  // READ_SINGLE_BLOCK
#define CMD24  24  // WRITE_BLOCK
#define CMD55  55  // APP_CMD
#define ACMD41 41  // SD_SEND_OP_COND

/*
 * SEND SD COMMAND
 */
unsigned char sd_send_cmd(unsigned char cmd, unsigned long arg)
{
    unsigned char response;
    unsigned char retry = 0;
    
    SD_CS = 1;
    spi_transfer(0xFF);
    SD_CS = 0;
    
    spi_transfer(0x40 | cmd);        // Command
    spi_transfer((arg >> 24) & 0xFF);
    spi_transfer((arg >> 16) & 0xFF);
    spi_transfer((arg >> 8) & 0xFF);
    spi_transfer(arg & 0xFF);
    
    // CRC (only needed for CMD0 and CMD8)
    if(cmd == CMD0)
        spi_transfer(0x95);
    else if(cmd == CMD8)
        spi_transfer(0x87);
    else
        spi_transfer(0xFF);
    
    // Wait for response
    while((response = spi_transfer(0xFF)) == 0xFF)
    {
        if(++retry > 200)
            break;
    }
    
    return response;
}

/*
 * SD CARD INITIALIZATION
 */
bit sd_init(void)
{
    unsigned char i, response;
    unsigned int retry;
    
    // Send 80+ clock pulses with CS high
    SD_CS = 1;
    for(i = 0; i < 10; i++)
        spi_transfer(0xFF);
    
    // CMD0 - Go to idle state
    for(i = 0; i < 10; i++)
    {
        response = sd_send_cmd(CMD0, 0);
        if(response == 0x01)
            break;
    }
    
    if(response != 0x01)
        return 1;  // Error
    
    // CMD8 - Check voltage range (SD v2)
    response = sd_send_cmd(CMD8, 0x1AA);
    
    // ACMD41 - Initialize card
    retry = 0;
    do {
        sd_send_cmd(CMD55, 0);
        response = sd_send_cmd(ACMD41, 0x40000000);
        
        if(++retry > 1000)
            return 1;  // Timeout
            
    } while(response != 0x00);
    
    SD_CS = 1;
    return 0;  // Success
}

/*
 * READ 512-BYTE BLOCK
 */
bit sd_read_block(unsigned long block_addr, unsigned char *buf)
{
    unsigned int i;
    unsigned char response;
    
    response = sd_send_cmd(CMD17, block_addr);
    
    if(response != 0x00)
    {
        SD_CS = 1;
        return 1;
    }
    
    // Wait for data token (0xFE)
    while(spi_transfer(0xFF) != 0xFE);
    
    // Read 512 bytes
    for(i = 0; i < 512; i++)
        buf[i] = spi_transfer(0xFF);
    
    // Discard CRC
    spi_transfer(0xFF);
    spi_transfer(0xFF);
    
    SD_CS = 1;
    return 0;
}

/*
 * WRITE 512-BYTE BLOCK
 */
bit sd_write_block(unsigned long block_addr, unsigned char *buf)
{
    unsigned int i;
    unsigned char response;
    
    response = sd_send_cmd(CMD24, block_addr);
    
    if(response != 0x00)
    {
        SD_CS = 1;
        return 1;
    }
    
    spi_transfer(0xFF);  // Dummy byte
    spi_transfer(0xFE);  // Data token
    
    // Write 512 bytes
    for(i = 0; i < 512; i++)
        spi_transfer(buf[i]);
    
    // Dummy CRC
    spi_transfer(0xFF);
    spi_transfer(0xFF);
    
    // Check response
    response = spi_transfer(0xFF);
    if((response & 0x1F) != 0x05)
    {
        SD_CS = 1;
        return 1;  // Write error
    }
    
    // Wait for card to finish writing
    while(spi_transfer(0xFF) == 0x00);
    
    SD_CS = 1;
    return 0;
}
```

---

## 4. Wireless Communication

### 4.1 HC-05 Bluetooth Module

```
BLUETOOTH HC-05 MODULE
━━━━━━━━━━━━━━━━━━━━━━

HC-05 CONNECTIONS:
────────────────

    8051 MCU                   HC-05 Module
    ┌───────────┐              ┌───────────────┐
    │           │              │               │
    │    VCC ───┼──────────────┤ VCC (3.3-6V)  │
    │           │              │               │
    │    GND ───┼──────────────┤ GND           │
    │           │              │               │
    │    TXD ───┼───────┐      │               │
    │   (P3.1)  │       │      │               │
    │           │      [R]     │               │
    │           │       │      │               │
    │           │       └──────┤ RXD           │
    │           │              │               │
    │    RXD ◄──┼──────────────┤ TXD           │
    │   (P3.0)  │              │               │
    │           │              │               │
    │    P1.0 ──┼──────────────┤ KEY/EN        │
    │ (AT Mode) │              │ (for config)  │
    └───────────┘              └───────────────┘

Note: Use voltage divider on RXD if 3.3V logic
      R1=1kΩ, R2=2kΩ (5V to 3.3V)

AT COMMANDS:
───────────

Command          │ Response    │ Description
─────────────────┼─────────────┼─────────────────────
AT               │ OK          │ Test connection
AT+VERSION       │ +VERSION:.. │ Firmware version
AT+NAME=MyBT     │ OK          │ Set device name
AT+PIN=1234      │ OK          │ Set pairing PIN
AT+BAUD=4        │ OK          │ Set baud (4=9600)
AT+ROLE=0        │ OK          │ Slave mode (0)
AT+ROLE=1        │ OK          │ Master mode (1)
```

### 4.2 Bluetooth Communication Code

```c
/*
 * HC-05 BLUETOOTH COMMUNICATION
 */

#include <reg51.h>

sbit BT_KEY = P1^0;  // AT mode pin

/*
 * ENTER AT COMMAND MODE
 */
void bt_enter_at_mode(void)
{
    BT_KEY = 1;
    delay_ms(100);
    
    // AT mode uses 38400 baud
    TH1 = 0xF4;  // 38400 baud @ 11.0592 MHz
    TL1 = 0xF4;
}

/*
 * EXIT AT COMMAND MODE
 */
void bt_exit_at_mode(void)
{
    BT_KEY = 0;
    delay_ms(100);
    
    // Normal mode uses 9600 baud
    TH1 = 0xFD;
    TL1 = 0xFD;
}

/*
 * SEND AT COMMAND
 */
void bt_send_at(char *cmd)
{
    uart_tx_string(cmd);
    uart_tx_string("\r\n");
}

/*
 * CONFIGURE BLUETOOTH MODULE
 */
void bt_configure(void)
{
    bt_enter_at_mode();
    delay_ms(1000);
    
    bt_send_at("AT");
    delay_ms(500);
    
    bt_send_at("AT+NAME=8051_Sensor");
    delay_ms(500);
    
    bt_send_at("AT+PIN=1234");
    delay_ms(500);
    
    bt_send_at("AT+BAUD=4");  // 9600 baud
    delay_ms(500);
    
    bt_exit_at_mode();
}

/*
 * BLUETOOTH SENSOR TRANSMITTER
 */
void main(void)
{
    char buf[32];
    int temp, humidity;
    
    uart_init();
    dht11_init();
    lcd_init();
    
    lcd_string("Bluetooth Sensor");
    delay_ms(2000);
    
    while(1)
    {
        // Read sensors
        dht11_read(&humidity, &temp);
        
        // Send via Bluetooth
        sprintf(buf, "T:%d,H:%d\r\n", temp, humidity);
        uart_tx_string(buf);
        
        // Display locally
        lcd_goto(0, 0);
        sprintf(buf, "Temp: %dC   ", temp);
        lcd_string(buf);
        
        lcd_goto(1, 0);
        sprintf(buf, "Humid: %d%%  ", humidity);
        lcd_string(buf);
        
        delay_ms(2000);
    }
}
```

### 4.3 ESP8266 WiFi Module

```
ESP8266 WiFi MODULE
━━━━━━━━━━━━━━━━━━━

ESP8266 CONNECTIONS:
──────────────────

    8051 MCU                   ESP8266 (ESP-01)
    ┌───────────┐              ┌───────────────┐
    │           │              │ VCC (3.3V!)   ├── 3.3V
    │    VCC ───┼──┤3.3V├──────┤               │
    │           │              │ GND           ├── GND
    │    GND ───┼──────────────┤               │
    │           │              │ CH_PD/EN      ├── 3.3V
    │    TXD ───┼───[Divider]──┤ RXD           │
    │   (P3.1)  │              │               │
    │           │              │ TXD           ├── 8051 RXD
    │    RXD ◄──┼──────────────┤               │
    │   (P3.0)  │              │ GPIO0         ├── 3.3V (Run)
    │           │              │               │   GND (Flash)
    │    P1.0 ──┼──────────────┤ RST           │
    │ (Reset)   │              │               │
    └───────────┘              └───────────────┘

⚠️ IMPORTANT: ESP8266 is 3.3V only! Use level shifter!

BASIC AT COMMANDS:
────────────────

Command                      │ Description
─────────────────────────────┼───────────────────────────
AT                           │ Test connection
AT+RST                       │ Reset module
AT+CWMODE=1                  │ Station mode
AT+CWMODE=2                  │ AP mode
AT+CWMODE=3                  │ Station + AP mode
AT+CWJAP="SSID","password"   │ Connect to WiFi
AT+CWLAP                     │ List available APs
AT+CIFSR                     │ Get IP address
AT+CIPMUX=1                  │ Multiple connections
AT+CIPSERVER=1,80            │ Start server port 80
AT+CIPSEND=0,length          │ Send data on conn 0
```

### 4.4 ESP8266 Web Server Code

```c
/*
 * ESP8266 WEB SERVER
 * Remote sensor monitoring
 */

#include <reg51.h>
#include <stdio.h>

#define WIFI_SSID     "YourNetwork"
#define WIFI_PASSWORD "YourPassword"

unsigned char esp_buf[256];
unsigned char buf_idx = 0;

/*
 * SEND AT COMMAND AND WAIT FOR RESPONSE
 */
bit esp_send_cmd(char *cmd, char *response, unsigned int timeout)
{
    unsigned int time = 0;
    
    buf_idx = 0;
    uart_tx_string(cmd);
    uart_tx_string("\r\n");
    
    while(time < timeout)
    {
        if(uart_rx_available())
        {
            esp_buf[buf_idx++] = uart_rx_read();
            esp_buf[buf_idx] = '\0';
            
            if(strstr(esp_buf, response))
                return 0;  // Success
        }
        delay_ms(1);
        time++;
    }
    
    return 1;  // Timeout
}

/*
 * INITIALIZE ESP8266
 */
bit esp_init(void)
{
    // Reset
    if(esp_send_cmd("AT+RST", "ready", 5000))
        return 1;
    
    // Station mode
    if(esp_send_cmd("AT+CWMODE=1", "OK", 1000))
        return 1;
    
    // Connect to WiFi
    sprintf(esp_buf, "AT+CWJAP=\"%s\",\"%s\"", 
            WIFI_SSID, WIFI_PASSWORD);
    if(esp_send_cmd(esp_buf, "WIFI CONNECTED", 10000))
        return 1;
    
    delay_ms(3000);  // Wait for IP
    
    // Multiple connections
    if(esp_send_cmd("AT+CIPMUX=1", "OK", 1000))
        return 1;
    
    // Start server on port 80
    if(esp_send_cmd("AT+CIPSERVER=1,80", "OK", 1000))
        return 1;
    
    return 0;
}

/*
 * SEND HTTP RESPONSE
 */
void esp_send_http(unsigned char conn_id, char *content)
{
    char cmd[32];
    unsigned int len = strlen(content);
    
    sprintf(cmd, "AT+CIPSEND=%d,%d", conn_id, len);
    esp_send_cmd(cmd, ">", 1000);
    
    uart_tx_string(content);
    delay_ms(100);
    
    // Close connection
    sprintf(cmd, "AT+CIPCLOSE=%d", conn_id);
    esp_send_cmd(cmd, "OK", 1000);
}

/*
 * GENERATE HTML PAGE
 */
void generate_html(char *html, int temp, int humidity)
{
    sprintf(html,
        "HTTP/1.1 200 OK\r\n"
        "Content-Type: text/html\r\n\r\n"
        "<html><head>"
        "<meta http-equiv='refresh' content='5'>"
        "<style>body{font-family:Arial;text-align:center;}</style>"
        "</head><body>"
        "<h1>8051 Sensor Monitor</h1>"
        "<p>Temperature: <b>%d&deg;C</b></p>"
        "<p>Humidity: <b>%d%%</b></p>"
        "</body></html>",
        temp, humidity);
}

/*
 * MAIN WEB SERVER
 */
void main(void)
{
    char html[512];
    int temp, humidity;
    unsigned char conn_id;
    
    uart_init();
    lcd_init();
    dht11_init();
    
    lcd_string("Connecting...");
    
    if(esp_init())
    {
        lcd_goto(0, 0);
        lcd_string("WiFi Error!     ");
        while(1);
    }
    
    lcd_goto(0, 0);
    lcd_string("Server Ready    ");
    
    // Get and display IP
    esp_send_cmd("AT+CIFSR", "OK", 1000);
    lcd_goto(1, 0);
    // Parse IP from response and display
    
    while(1)
    {
        // Read sensors
        dht11_read(&humidity, &temp);
        
        // Check for incoming connection
        if(strstr(esp_buf, "+IPD,"))
        {
            // Extract connection ID
            conn_id = esp_buf[strstr(esp_buf, "+IPD,") - esp_buf + 5] - '0';
            
            // Generate and send response
            generate_html(html, temp, humidity);
            esp_send_http(conn_id, html);
            
            buf_idx = 0;
        }
        
        // Read any incoming data
        while(uart_rx_available())
        {
            esp_buf[buf_idx++] = uart_rx_read();
            if(buf_idx >= 250)
                buf_idx = 0;
            esp_buf[buf_idx] = '\0';
        }
        
        delay_ms(10);
    }
}
```

---

## 📋 Communication Protocol Comparison

| Feature | UART | I2C | SPI |
|---------|------|-----|-----|
| Wires | 2 (TX, RX) | 2 (SDA, SCL) | 4 (MOSI, MISO, SCK, CS) |
| Speed | Up to 1 Mbps | Up to 3.4 Mbps | Up to 50+ MHz |
| Devices | Point-to-point | Multi-master/slave | Multi-slave |
| Distance | Long (RS-485) | Short (<2m) | Short (<1m) |
| Duplex | Full | Half | Full |
| Addressing | None | 7/10-bit | CS per device |
| Complexity | Low | Medium | Low |

---

## ❓ Quick Revision Questions

1. **What is the purpose of MAX232 IC?**
   <details>
   <summary>Show Answer</summary>
   MAX232 converts TTL levels (0-5V) to RS-232 levels (±12V) and vice versa. Required because RS-232 uses inverted logic with higher voltage swing for noise immunity over longer cables.
   </details>

2. **How does I2C handle multiple slaves on the same bus?**
   <details>
   <summary>Show Answer</summary>
   Each I2C slave has unique 7-bit address. Master sends address after START condition. Only addressed slave responds with ACK and participates in data transfer. Others remain silent.
   </details>

3. **What is the difference between SPI Mode 0 and Mode 3?**
   <details>
   <summary>Show Answer</summary>
   Mode 0: Clock idle low (CPOL=0), sample on rising edge (CPHA=0). Mode 3: Clock idle high (CPOL=1), sample on rising edge (CPHA=1). Both sample on rising edge but idle states differ.
   </details>

4. **Why do Bluetooth and WiFi modules use AT commands?**
   <details>
   <summary>Show Answer</summary>
   AT commands (Hayes command set) provide simple ASCII-based configuration interface over serial port. Easy to implement, human-readable, no special protocol stack needed on host MCU. Just UART required.
   </details>

5. **What is the purpose of CIPSEND command in ESP8266?**
   <details>
   <summary>Show Answer</summary>
   AT+CIPSEND prepares ESP8266 to receive data for transmission. Syntax: AT+CIPSEND=conn_id,length. After receiving ">", host sends data bytes. Module transmits when length reached.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [10.3 Motor Control Systems](03-motor-control-systems.md) | [Unit 10 Index](README.md) | [10.5 Display and HMI Systems](05-display-hmi-systems.md) |

---

*[← Previous: Motor Control Systems](03-motor-control-systems.md) | [Next: Display and HMI Systems →](05-display-hmi-systems.md)*
