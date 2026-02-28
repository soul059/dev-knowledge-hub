# Chapter 5.2: UART Protocol

## Introduction

UART (Universal Asynchronous Receiver/Transmitter) is the most common serial communication interface in embedded systems. It provides simple, reliable point-to-point communication with minimal hardware requirements.

---

## UART Hardware Interface

```
┌─────────────────────────────────────────────────────────────────────┐
│                    UART CONNECTION                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  BASIC CONNECTION (2-wire):                                         │
│                                                                      │
│     Device A                              Device B                   │
│  ┌──────────────┐                     ┌──────────────┐              │
│  │              │  TX ──────────────▶ │ RX           │              │
│  │    UART      │                     │    UART      │              │
│  │              │  RX ◀────────────── │ TX           │              │
│  │              │                     │              │              │
│  │              │  GND ───────────── │ GND          │              │
│  └──────────────┘                     └──────────────┘              │
│                                                                      │
│  NOTE: TX connects to RX (crossover)                                │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  WITH HARDWARE FLOW CONTROL (4-wire):                               │
│                                                                      │
│     Device A                              Device B                   │
│  ┌──────────────┐                     ┌──────────────┐              │
│  │              │  TX ──────────────▶ │ RX           │              │
│  │              │  RX ◀────────────── │ TX           │              │
│  │    UART      │                     │    UART      │              │
│  │              │  RTS ─────────────▶ │ CTS          │              │
│  │              │  CTS ◀───────────── │ RTS          │              │
│  │              │                     │              │              │
│  │              │  GND ───────────── │ GND          │              │
│  └──────────────┘                     └──────────────┘              │
│                                                                      │
│  RTS = Request To Send (I'm ready to receive)                       │
│  CTS = Clear To Send (You can send to me)                           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## UART Frame Format

```
┌─────────────────────────────────────────────────────────────────────┐
│                    UART TIMING DIAGRAM                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Transmitting byte 0x55 (01010101) at 9600 baud, 8N1:              │
│                                                                      │
│  Bit time = 1/9600 = 104.17 μs                                      │
│                                                                      │
│  Voltage                                                             │
│     │                                                                │
│   H ┼────┐    ┌────┐    ┌────┐    ┌────┐    ┌────────────           │
│     │    │    │    │    │    │    │    │    │                        │
│   L ┼    └────┘    └────┘    └────┘    └────┘                        │
│     │                                                                │
│     └────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────▶ Time   │
│          │ S  │ D0 │ D1 │ D2 │ D3 │ D4 │ D5 │ D6 │ D7 │ ST │        │
│          │ 0  │ 1  │ 0  │ 1  │ 0  │ 1  │ 0  │ 1  │ 0  │ 1  │        │
│          │Start    ◀────── Data Bits ──────▶    │Stop│              │
│          │                                                           │
│     │◀──────────── 10 × 104.17μs = 1.042ms ─────────────▶│          │
│                                                                      │
│  Data sent LSB first: 10101010 (on wire) = 0x55                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Frame Configurations

| Configuration | Bits/Frame | Efficiency | Use Case |
|---------------|------------|------------|----------|
| 8N1 | 10 | 80% | Most common |
| 8E1 | 11 | 72.7% | Error detection |
| 8O1 | 11 | 72.7% | Error detection |
| 8N2 | 11 | 72.7% | Slow receivers |
| 7E1 | 10 | 70% | Legacy ASCII |

---

## UART Register Model

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TYPICAL UART REGISTERS                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Register        │ Purpose                                   │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │ UART_DR         │ Data Register (TX and RX)                │    │
│  │ UART_BRR        │ Baud Rate Register                       │    │
│  │ UART_CR1        │ Control Register 1 (enable, word length) │    │
│  │ UART_CR2        │ Control Register 2 (stop bits, etc.)     │    │
│  │ UART_CR3        │ Control Register 3 (flow control)        │    │
│  │ UART_SR         │ Status Register (flags)                  │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  STATUS REGISTER FLAGS:                                              │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Bit  │ Name │ Meaning                                      │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │ TXE  │      │ TX buffer Empty (ready for new data)        │    │
│  │ RXNE │      │ RX buffer Not Empty (data available)        │    │
│  │ TC   │      │ Transmission Complete                       │    │
│  │ ORE  │      │ Overrun Error (data lost)                   │    │
│  │ FE   │      │ Framing Error (invalid stop bit)            │    │
│  │ PE   │      │ Parity Error                                │    │
│  │ IDLE │      │ Idle line detected                          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## UART Implementation

### Polling-Based Driver

```c
// UART initialization
void uart_init(uint32_t baud_rate) {
    // Enable clock
    RCC->APB2ENR |= RCC_APB2ENR_USART1EN;
    
    // Configure baud rate
    // BRR = fck / baud_rate (assuming 16x oversampling)
    USART1->BRR = SystemCoreClock / baud_rate;
    
    // Configure: 8N1, TX and RX enabled
    USART1->CR1 = USART_CR1_TE |      // TX enable
                  USART_CR1_RE |      // RX enable
                  USART_CR1_UE;       // UART enable
    
    // Default: 8 data bits, no parity, 1 stop bit
}

// Transmit single byte (blocking)
void uart_tx_byte(uint8_t data) {
    // Wait for TX buffer empty
    while (!(USART1->SR & USART_SR_TXE));
    
    // Write data
    USART1->DR = data;
}

// Receive single byte (blocking)
uint8_t uart_rx_byte(void) {
    // Wait for data
    while (!(USART1->SR & USART_SR_RXNE));
    
    // Read and return
    return (uint8_t)USART1->DR;
}

// Transmit string
void uart_tx_string(const char *str) {
    while (*str) {
        uart_tx_byte(*str++);
    }
}
```

### Interrupt-Based Driver

```c
// Ring buffer for received data
#define RX_BUFFER_SIZE 128
volatile uint8_t rx_buffer[RX_BUFFER_SIZE];
volatile uint16_t rx_head = 0;
volatile uint16_t rx_tail = 0;

void uart_init_interrupt(uint32_t baud_rate) {
    // Basic init
    uart_init(baud_rate);
    
    // Enable RX interrupt
    USART1->CR1 |= USART_CR1_RXNEIE;
    
    // Enable NVIC interrupt
    NVIC_EnableIRQ(USART1_IRQn);
    NVIC_SetPriority(USART1_IRQn, 5);
}

// UART interrupt handler
void USART1_IRQHandler(void) {
    // RX not empty interrupt
    if (USART1->SR & USART_SR_RXNE) {
        uint8_t data = USART1->DR;  // Reading clears flag
        
        // Calculate next head position
        uint16_t next = (rx_head + 1) % RX_BUFFER_SIZE;
        
        // Store if buffer not full
        if (next != rx_tail) {
            rx_buffer[rx_head] = data;
            rx_head = next;
        }
        // else: buffer overflow, data lost
    }
    
    // Handle errors
    if (USART1->SR & (USART_SR_ORE | USART_SR_FE | USART_SR_PE)) {
        // Clear errors by reading SR then DR
        volatile uint32_t temp = USART1->SR;
        temp = USART1->DR;
        (void)temp;
    }
}

// Check if data available
bool uart_available(void) {
    return rx_head != rx_tail;
}

// Read byte from buffer (non-blocking)
int uart_read(void) {
    if (rx_head == rx_tail) {
        return -1;  // No data
    }
    
    uint8_t data = rx_buffer[rx_tail];
    rx_tail = (rx_tail + 1) % RX_BUFFER_SIZE;
    return data;
}
```

---

## Flow Control

```
┌─────────────────────────────────────────────────────────────────────┐
│                    HARDWARE FLOW CONTROL (RTS/CTS)                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SCENARIO: Device B's buffer filling up                             │
│                                                                      │
│  Device A                                  Device B                  │
│     │                                         │                      │
│     │ ──────────────── Data ────────────────▶ │ Buffer filling       │
│     │                                         │                      │
│     │ ──────────────── Data ────────────────▶ │ Buffer 80%          │
│     │                                         │                      │
│     │ ◀──────────── CTS=LOW ──────────────── │ Buffer full!         │
│     │                                         │ (Stop sending)       │
│     │ (A pauses)                              │                      │
│     │                                         │ (B processes data)   │
│     │                                         │                      │
│     │ ◀──────────── CTS=HIGH ─────────────── │ Buffer has space     │
│     │                                         │                      │
│     │ ──────────────── Data ────────────────▶ │ Resume               │
│     │                                         │                      │
│                                                                      │
│  TIMING:                                                             │
│                                                                      │
│  Data:  ════════════════════════════        ═══════════════         │
│                                                                      │
│  CTS:   ────────────────────────────┐       ┌───────────────        │
│                                     └───────┘                        │
│                                     Stop    Resume                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Software Flow Control (XON/XOFF)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SOFTWARE FLOW CONTROL                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Control characters embedded in data stream:                        │
│                                                                      │
│  XON  = 0x11 (DC1) = Resume transmission                           │
│  XOFF = 0x13 (DC3) = Pause transmission                            │
│                                                                      │
│  Device A                                  Device B                  │
│     │                                         │                      │
│     │ ──────── Data Data Data ─────────────▶ │                      │
│     │                                         │                      │
│     │ ◀──────────── XOFF (0x13) ───────────── │ Buffer full         │
│     │                                         │                      │
│     │ (A pauses transmission)                 │ (B drains buffer)   │
│     │                                         │                      │
│     │ ◀──────────── XON (0x11) ────────────── │ Ready again         │
│     │                                         │                      │
│     │ ──────── Data Data Data ─────────────▶ │                      │
│                                                                      │
│  LIMITATION: Cannot send 0x11 or 0x13 as data (binary data issue)  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## RS-232 and RS-485

```
┌─────────────────────────────────────────────────────────────────────┐
│                    RS-232 vs RS-485                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  RS-232 (Point-to-point):                                           │
│                                                                      │
│     DTE                 DCE                                          │
│  ┌───────┐           ┌───────┐                                      │
│  │  PC   │───────────│ Modem │                                      │
│  └───────┘           └───────┘                                      │
│                                                                      │
│  • Voltage: ±3V to ±15V                                             │
│  • Distance: Up to 15m (50ft)                                       │
│  • Nodes: 1 transmitter, 1 receiver                                 │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  RS-485 (Multi-drop):                                               │
│                                                                      │
│      ┌───────┐  ┌───────┐  ┌───────┐  ┌───────┐                    │
│      │Node 1 │  │Node 2 │  │Node 3 │  │Node 32│                    │
│      └───┬───┘  └───┬───┘  └───┬───┘  └───┬───┘                    │
│          │          │          │          │                         │
│      ────┴──────────┴──────────┴──────────┴────── A (Data+)        │
│      ────┬──────────┬──────────┬──────────┬────── B (Data-)        │
│          │          │          │          │                         │
│        120Ω                              120Ω  ◀─ Termination       │
│                                                                      │
│  • Voltage: Differential ±1.5V to ±6V                               │
│  • Distance: Up to 1200m (4000ft) at 100kbps                       │
│  • Nodes: 32 (or 256 with high-impedance receivers)                │
│  • Half-duplex (shared bus)                                         │
│                                                                      │
│  RS-485 TRANSCEIVER CONTROL:                                        │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  DE (Driver Enable): High = Transmit, Low = Receive        │     │
│  │  RE (Receiver Enable): Active low, usually tied to ~DE    │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### RS-485 Driver Control

```c
// RS-485 with direction control
#define RS485_TX_ENABLE()   GPIO_SET(DE_PIN)
#define RS485_RX_ENABLE()   GPIO_CLEAR(DE_PIN)

void rs485_tx_byte(uint8_t data) {
    RS485_TX_ENABLE();
    
    // Wait for TX buffer empty
    while (!(USART1->SR & USART_SR_TXE));
    USART1->DR = data;
    
    // Wait for transmission complete
    while (!(USART1->SR & USART_SR_TC));
    
    RS485_RX_ENABLE();  // Return to receive mode
}
```

---

## DMA-Based UART

For high-speed or large transfers, DMA offloads the CPU.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    UART WITH DMA                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────┐      ┌─────────┐      ┌─────────┐      ┌─────────┐    │
│  │  CPU    │      │   DMA   │      │  UART   │      │ Buffer  │    │
│  │         │      │         │      │         │      │         │    │
│  │ Setup ──┼─────▶│ Config  │      │         │      │ [Data]  │    │
│  │         │      │         │      │         │      │         │    │
│  │ (Free)  │      │ Transfer├─────▶│   TX  ──┼─────▶│         │    │
│  │         │      │ (auto)  │      │         │      │         │    │
│  │ ◀───────┼──────┤ Done IRQ│      │         │      │         │    │
│  └─────────┘      └─────────┘      └─────────┘      └─────────┘    │
│                                                                      │
│  CPU only involved at start (setup) and end (completion)           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

```c
// DMA TX configuration
void uart_dma_tx(uint8_t *buffer, uint16_t length) {
    // Disable DMA
    DMA1_Channel4->CCR &= ~DMA_CCR_EN;
    
    // Configure
    DMA1_Channel4->CPAR = (uint32_t)&USART1->DR;  // Peripheral address
    DMA1_Channel4->CMAR = (uint32_t)buffer;       // Memory address
    DMA1_Channel4->CNDTR = length;                 // Transfer count
    
    DMA1_Channel4->CCR = DMA_CCR_MINC |   // Memory increment
                         DMA_CCR_DIR |     // Memory to peripheral
                         DMA_CCR_TCIE;     // Transfer complete IRQ
    
    // Enable UART DMA request
    USART1->CR3 |= USART_CR3_DMAT;
    
    // Start DMA
    DMA1_Channel4->CCR |= DMA_CCR_EN;
}

// DMA complete interrupt
void DMA1_Channel4_IRQHandler(void) {
    if (DMA1->ISR & DMA_ISR_TCIF4) {
        DMA1->IFCR = DMA_IFCR_CTCIF4;  // Clear flag
        // Transfer complete - notify application
        tx_complete_flag = 1;
    }
}
```

---

## Common UART Applications

```
┌─────────────────────────────────────────────────────────────────────┐
│                    UART APPLICATION EXAMPLES                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. DEBUG CONSOLE                                                    │
│                                                                      │
│  ┌───────────┐   UART/USB    ┌──────────────────┐                   │
│  │    MCU    │──────────────▶│  PC Terminal     │                   │
│  │           │  115200 8N1   │ (PuTTY, minicom) │                   │
│  └───────────┘               └──────────────────┘                   │
│                                                                      │
│  printf() redirected to UART for debugging                          │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  2. GPS MODULE                                                       │
│                                                                      │
│  ┌───────────┐               ┌──────────────────┐                   │
│  │    MCU    │◀──────────────│   GPS Module     │                   │
│  │           │    9600 8N1   │  (NMEA output)   │                   │
│  └───────────┘               └──────────────────┘                   │
│                                                                      │
│  $GPGGA,123519,4807.038,N,01131.000,E,1,08,0.9,545.4,M,...         │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  3. BLUETOOTH MODULE (HC-05)                                        │
│                                                                      │
│  ┌───────────┐               ┌──────────────────┐                   │
│  │    MCU    │◀─────────────▶│  HC-05 Bluetooth │                   │
│  │           │   38400 8N1   │     Module       │                   │
│  └───────────┘               └──────────────────┘                   │
│                                                                      │
│  AT commands: AT+NAME=MyDevice, AT+BAUD=4                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Feature | Specification |
|---------|---------------|
| **Wires** | 2 (TX, RX) + optional (RTS, CTS) |
| **Speed** | 300 - 3,000,000 baud |
| **Distance** | 15m (RS-232), 1200m (RS-485) |
| **Frame** | Start + 5-9 data + parity + 1-2 stop |
| **Error Detection** | Parity, framing error, overrun |
| **Flow Control** | None, HW (RTS/CTS), SW (XON/XOFF) |
| **Topology** | Point-to-point (multi-drop with RS-485) |

---

## Quick Revision Questions

1. **Explain the UART frame structure. What is the purpose of start and stop bits?**

2. **What is the difference between TXE and TC flags? When would you check each?**

3. **Implement a function to calculate the baud rate register value given a 16 MHz clock and desired 115200 baud.**

4. **Explain hardware flow control (RTS/CTS). Why is it preferred over XON/XOFF for binary data?**

5. **What is the difference between RS-232 and RS-485? When would you use each?**

6. **Why is DMA useful for UART? Describe the flow of a DMA-based transmission.**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Chapter 5.1: Serial Communication Basics](01-serial-communication-basics.md) | [Unit Index](README.md) | [Chapter 5.3: SPI Protocol](03-spi-protocol.md) |

---
