# Chapter 3.4: Device Drivers

## Chapter Overview

Device drivers are software modules that provide a standardized interface between application code and hardware peripherals. This chapter covers driver architecture, common driver patterns, and implementation techniques for GPIO, UART, SPI, I2C, and timer peripherals.

---

## 3.4.1 Driver Architecture

### Driver Model Concepts

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DEVICE DRIVER ARCHITECTURE                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  DRIVER INTERFACE PATTERN:                                           │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                     APPLICATION                             │    │
│  │                          │                                  │    │
│  │              ┌───────────┴───────────┐                     │    │
│  │              │   Driver API (Public) │                     │    │
│  │              │  • init()             │                     │    │
│  │              │  • deinit()           │                     │    │
│  │              │  • read()             │                     │    │
│  │              │  • write()            │                     │    │
│  │              │  • ioctl()            │                     │    │
│  │              └───────────┬───────────┘                     │    │
│  │                          │                                  │    │
│  │              ┌───────────┴───────────┐                     │    │
│  │              │  Driver Core (Private)│                     │    │
│  │              │  • State management   │                     │    │
│  │              │  • Buffer handling    │                     │    │
│  │              │  • ISR handlers       │                     │    │
│  │              └───────────┬───────────┘                     │    │
│  │                          │                                  │    │
│  │              ┌───────────┴───────────┐                     │    │
│  │              │    HAL / Register     │                     │    │
│  │              │       Access          │                     │    │
│  │              └───────────────────────┘                     │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  STANDARD DRIVER API:                                                │
│                                                                      │
│  typedef struct {                                                    │
│      int  (*init)(void *config);        // Initialize driver        │
│      int  (*deinit)(void);              // Deinitialize             │
│      int  (*open)(uint32_t flags);      // Open for use             │
│      int  (*close)(void);               // Close/release            │
│      int  (*read)(void *buf, size_t n); // Read data               │
│      int  (*write)(const void *buf, size_t n); // Write data       │
│      int  (*ioctl)(uint32_t cmd, void *arg);   // Control          │
│  } driver_ops_t;                                                     │
│                                                                      │
│  DRIVER STATES:                                                      │
│                                                                      │
│       ┌──────────────────────────────────────────────┐              │
│       │                                              │              │
│       │    ┌──────────┐  init()   ┌──────────┐      │              │
│       │    │UNINITIALIZED│──────►│ READY    │       │              │
│       │    └──────────┘          └────┬─────┘       │              │
│       │          ▲                    │             │              │
│       │          │ deinit()    open() │             │              │
│       │          │                    ▼             │              │
│       │    ┌─────┴────┐          ┌──────────┐      │              │
│       │    │  READY   │◄─────────│  OPEN    │      │              │
│       │    │          │  close() │          │      │              │
│       │    └──────────┘          └────┬─────┘      │              │
│       │                               │             │              │
│       │                  read()/write()│            │              │
│       │                               ▼             │              │
│       │                          ┌──────────┐      │              │
│       │                          │  BUSY    │      │              │
│       │                          └──────────┘      │              │
│       │                                              │              │
│       └──────────────────────────────────────────────┘              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.4.2 GPIO Driver

### GPIO Driver Implementation

```
┌─────────────────────────────────────────────────────────────────────┐
│                    GPIO DRIVER                                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  GPIO DRIVER HEADER (gpio_driver.h):                                 │
│                                                                      │
│  typedef enum {                                                      │
│      GPIO_MODE_INPUT,                                                │
│      GPIO_MODE_OUTPUT_PP,    // Push-pull                           │
│      GPIO_MODE_OUTPUT_OD,    // Open-drain                          │
│      GPIO_MODE_AF_PP,        // Alternate function push-pull        │
│      GPIO_MODE_ANALOG                                                │
│  } gpio_mode_t;                                                      │
│                                                                      │
│  typedef enum {                                                      │
│      GPIO_PULL_NONE,                                                 │
│      GPIO_PULL_UP,                                                   │
│      GPIO_PULL_DOWN                                                  │
│  } gpio_pull_t;                                                      │
│                                                                      │
│  typedef struct {                                                    │
│      uint8_t     port;       // GPIO port (A, B, C, ...)            │
│      uint8_t     pin;        // Pin number (0-15)                   │
│      gpio_mode_t mode;       // Pin mode                            │
│      gpio_pull_t pull;       // Pull-up/down                        │
│      uint8_t     speed;      // Output speed                        │
│  } gpio_config_t;                                                    │
│                                                                      │
│  // API functions                                                    │
│  int  gpio_init(const gpio_config_t *config);                       │
│  void gpio_write(uint8_t port, uint8_t pin, uint8_t value);         │
│  int  gpio_read(uint8_t port, uint8_t pin);                         │
│  void gpio_toggle(uint8_t port, uint8_t pin);                       │
│                                                                      │
│  GPIO DRIVER IMPLEMENTATION (gpio_driver.c):                         │
│                                                                      │
│  static GPIO_TypeDef* get_gpio_port(uint8_t port) {                 │
│      GPIO_TypeDef* ports[] = {GPIOA, GPIOB, GPIOC, GPIOD};          │
│      return ports[port];                                             │
│  }                                                                   │
│                                                                      │
│  int gpio_init(const gpio_config_t *config) {                       │
│      GPIO_TypeDef *gpio = get_gpio_port(config->port);              │
│                                                                      │
│      // Enable clock                                                │
│      RCC->AHB1ENR |= (1 << config->port);                           │
│                                                                      │
│      // Set mode (2 bits per pin)                                   │
│      gpio->MODER &= ~(3 << (config->pin * 2));                      │
│      gpio->MODER |= (config->mode << (config->pin * 2));            │
│                                                                      │
│      // Set pull-up/down                                            │
│      gpio->PUPDR &= ~(3 << (config->pin * 2));                      │
│      gpio->PUPDR |= (config->pull << (config->pin * 2));            │
│                                                                      │
│      return 0;                                                       │
│  }                                                                   │
│                                                                      │
│  void gpio_write(uint8_t port, uint8_t pin, uint8_t value) {        │
│      GPIO_TypeDef *gpio = get_gpio_port(port);                      │
│      if (value) {                                                    │
│          gpio->BSRR = (1 << pin);        // Set (atomic)            │
│      } else {                                                        │
│          gpio->BSRR = (1 << (pin + 16)); // Reset (atomic)          │
│      }                                                               │
│  }                                                                   │
│                                                                      │
│  int gpio_read(uint8_t port, uint8_t pin) {                         │
│      GPIO_TypeDef *gpio = get_gpio_port(port);                      │
│      return (gpio->IDR >> pin) & 1;                                 │
│  }                                                                   │
│                                                                      │
│  USAGE EXAMPLE:                                                      │
│                                                                      │
│  gpio_config_t led_config = {                                       │
│      .port = 0,  // GPIOA                                           │
│      .pin  = 5,  // PA5                                             │
│      .mode = GPIO_MODE_OUTPUT_PP,                                   │
│      .pull = GPIO_PULL_NONE                                         │
│  };                                                                  │
│                                                                      │
│  gpio_init(&led_config);                                            │
│  gpio_write(0, 5, 1);  // LED ON                                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.4.3 UART Driver

### UART Driver with Interrupts

```
┌─────────────────────────────────────────────────────────────────────┐
│                    UART DRIVER                                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  UART DRIVER ARCHITECTURE:                                           │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                                                              │    │
│  │  Application:  uart_write(data, len);                       │    │
│  │                         │                                   │    │
│  │                         ▼                                   │    │
│  │  ┌───────────────────────────────────────────────────────┐ │    │
│  │  │  TX Ring Buffer     │     RX Ring Buffer              │ │    │
│  │  │  ┌───────────────┐  │  ┌───────────────┐             │ │    │
│  │  │  │ □ □ ■ ■ ■ □ □ │  │  │ ■ ■ ■ □ □ □ ■ │             │ │    │
│  │  │  └───────────────┘  │  └───────────────┘             │ │    │
│  │  │    ↑ head  ↑ tail   │    ↑ head  ↑ tail              │ │    │
│  │  └───────────────────────────────────────────────────────┘ │    │
│  │                │                       ▲                    │    │
│  │  TX Interrupt  │                       │  RX Interrupt     │    │
│  │                ▼                       │                    │    │
│  │  ┌────────────────────────────────────────────────────────┐│    │
│  │  │              UART Peripheral                           ││    │
│  │  │    TX Data Register ──► TX ──────────────► TX Pin      ││    │
│  │  │    RX Data Register ◄── RX ◄────────────── RX Pin      ││    │
│  │  └────────────────────────────────────────────────────────┘│    │
│  │                                                              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  UART DRIVER IMPLEMENTATION:                                         │
│                                                                      │
│  #define UART_BUF_SIZE 128                                           │
│                                                                      │
│  typedef struct {                                                    │
│      volatile uint8_t buffer[UART_BUF_SIZE];                        │
│      volatile uint16_t head;                                        │
│      volatile uint16_t tail;                                        │
│  } ring_buffer_t;                                                    │
│                                                                      │
│  static ring_buffer_t tx_buf, rx_buf;                               │
│  static USART_TypeDef *uart_periph;                                 │
│                                                                      │
│  int uart_init(uint32_t baudrate) {                                 │
│      uart_periph = USART1;                                          │
│                                                                      │
│      // Enable clocks                                               │
│      RCC->APB2ENR |= RCC_APB2ENR_USART1EN;                          │
│                                                                      │
│      // Configure baud rate                                         │
│      uart_periph->BRR = SystemCoreClock / baudrate;                 │
│                                                                      │
│      // Enable TX, RX, and RXNE interrupt                           │
│      uart_periph->CR1 = USART_CR1_TE | USART_CR1_RE |               │
│                         USART_CR1_RXNEIE | USART_CR1_UE;            │
│                                                                      │
│      // Enable NVIC interrupt                                       │
│      NVIC_EnableIRQ(USART1_IRQn);                                   │
│                                                                      │
│      return 0;                                                       │
│  }                                                                   │
│                                                                      │
│  int uart_write(const uint8_t *data, uint16_t len) {                │
│      for (int i = 0; i < len; i++) {                                │
│          uint16_t next = (tx_buf.head + 1) % UART_BUF_SIZE;         │
│          while (next == tx_buf.tail);  // Wait if full              │
│                                                                      │
│          tx_buf.buffer[tx_buf.head] = data[i];                      │
│          tx_buf.head = next;                                        │
│      }                                                               │
│      // Enable TX interrupt                                         │
│      uart_periph->CR1 |= USART_CR1_TXEIE;                           │
│      return len;                                                     │
│  }                                                                   │
│                                                                      │
│  // UART ISR                                                         │
│  void USART1_IRQHandler(void) {                                     │
│      // RX interrupt                                                │
│      if (uart_periph->SR & USART_SR_RXNE) {                         │
│          uint8_t data = uart_periph->DR;                            │
│          uint16_t next = (rx_buf.head + 1) % UART_BUF_SIZE;         │
│          if (next != rx_buf.tail) {                                 │
│              rx_buf.buffer[rx_buf.head] = data;                     │
│              rx_buf.head = next;                                    │
│          }                                                           │
│      }                                                               │
│                                                                      │
│      // TX interrupt                                                │
│      if (uart_periph->SR & USART_SR_TXE) {                          │
│          if (tx_buf.head != tx_buf.tail) {                          │
│              uart_periph->DR = tx_buf.buffer[tx_buf.tail];          │
│              tx_buf.tail = (tx_buf.tail + 1) % UART_BUF_SIZE;       │
│          } else {                                                    │
│              uart_periph->CR1 &= ~USART_CR1_TXEIE; // Disable TX int│
│          }                                                           │
│      }                                                               │
│  }                                                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.4.4 I2C Driver

### I2C Master Driver

```
┌─────────────────────────────────────────────────────────────────────┐
│                    I2C DRIVER                                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  I2C TRANSACTION FLOW:                                               │
│                                                                      │
│  Write Transaction:                                                  │
│  ┌─────┬─────────┬───┬─────┬─────────┬─────┬─────────┬─────┐       │
│  │START│ ADDRESS │ W │ ACK │ REG_ADDR│ ACK │  DATA   │STOP │       │
│  │     │ (7-bit) │   │     │         │     │         │     │       │
│  └─────┴─────────┴───┴─────┴─────────┴─────┴─────────┴─────┘       │
│                                                                      │
│  Read Transaction:                                                   │
│  ┌─────┬─────────┬───┬─────┬─────────┬─────┬──────┬─────────┬───┬──┐│
│  │START│ ADDRESS │ W │ ACK │ REG_ADDR│ ACK │RSTART│ ADDRESS │ R │  ││
│  └─────┴─────────┴───┴─────┴─────────┴─────┴──────┴─────────┴───┴──┘│
│                                                                      │
│  ┌─────┬─────────┬─────┬─────────┬─────┬─────┐                      │
│  │ ACK │  DATA   │ ACK │  DATA   │NACK │STOP │                      │
│  └─────┴─────────┴─────┴─────────┴─────┴─────┘                      │
│                                                                      │
│  I2C DRIVER API:                                                     │
│                                                                      │
│  typedef struct {                                                    │
│      I2C_TypeDef *periph;      // I2C peripheral                    │
│      uint32_t    clock_speed;  // 100kHz or 400kHz                  │
│  } i2c_config_t;                                                     │
│                                                                      │
│  int  i2c_init(const i2c_config_t *config);                         │
│  int  i2c_write(uint8_t addr, uint8_t reg, uint8_t *data, uint8_t len);│
│  int  i2c_read(uint8_t addr, uint8_t reg, uint8_t *data, uint8_t len); │
│                                                                      │
│  I2C DRIVER IMPLEMENTATION:                                          │
│                                                                      │
│  static I2C_TypeDef *i2c;                                           │
│                                                                      │
│  int i2c_write(uint8_t addr, uint8_t reg, uint8_t *data, uint8_t len) {│
│      // Generate START                                              │
│      i2c->CR1 |= I2C_CR1_START;                                     │
│      while(!(i2c->SR1 & I2C_SR1_SB));  // Wait for START            │
│                                                                      │
│      // Send address + write bit                                    │
│      i2c->DR = (addr << 1) | 0;  // Write mode                      │
│      while(!(i2c->SR1 & I2C_SR1_ADDR)); // Wait for ADDR            │
│      (void)i2c->SR2;  // Clear ADDR flag                            │
│                                                                      │
│      // Send register address                                       │
│      i2c->DR = reg;                                                 │
│      while(!(i2c->SR1 & I2C_SR1_TXE));                              │
│                                                                      │
│      // Send data                                                   │
│      for (int i = 0; i < len; i++) {                                │
│          i2c->DR = data[i];                                         │
│          while(!(i2c->SR1 & I2C_SR1_TXE));                          │
│      }                                                               │
│                                                                      │
│      // Generate STOP                                               │
│      i2c->CR1 |= I2C_CR1_STOP;                                      │
│                                                                      │
│      return len;                                                     │
│  }                                                                   │
│                                                                      │
│  USAGE EXAMPLE (Temperature Sensor):                                 │
│                                                                      │
│  #define TEMP_SENSOR_ADDR  0x48                                     │
│  #define TEMP_REG          0x00                                     │
│                                                                      │
│  int16_t read_temperature(void) {                                   │
│      uint8_t data[2];                                               │
│      i2c_read(TEMP_SENSOR_ADDR, TEMP_REG, data, 2);                 │
│      return (data[0] << 8) | data[1];  // 16-bit result             │
│  }                                                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.4.5 Timer Driver

### Timer and PWM Driver

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TIMER DRIVER                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  TIMER DRIVER API:                                                   │
│                                                                      │
│  typedef enum {                                                      │
│      TIMER_MODE_PERIODIC,     // Auto-reload mode                   │
│      TIMER_MODE_ONESHOT,      // Single shot mode                   │
│      TIMER_MODE_PWM           // PWM generation                     │
│  } timer_mode_t;                                                     │
│                                                                      │
│  typedef struct {                                                    │
│      TIM_TypeDef  *periph;    // Timer peripheral                   │
│      timer_mode_t  mode;      // Operating mode                     │
│      uint32_t      period_us; // Period in microseconds             │
│      void (*callback)(void);  // ISR callback function              │
│  } timer_config_t;                                                   │
│                                                                      │
│  int  timer_init(const timer_config_t *config);                     │
│  void timer_start(void);                                            │
│  void timer_stop(void);                                             │
│  void timer_set_period(uint32_t period_us);                         │
│  void timer_pwm_set_duty(uint8_t channel, uint8_t duty_percent);    │
│                                                                      │
│  TIMER DRIVER IMPLEMENTATION:                                        │
│                                                                      │
│  static TIM_TypeDef *timer;                                         │
│  static void (*timer_callback)(void);                               │
│                                                                      │
│  int timer_init(const timer_config_t *config) {                     │
│      timer = config->periph;                                        │
│      timer_callback = config->callback;                             │
│                                                                      │
│      // Calculate prescaler and period                              │
│      // Timer clock = 84 MHz (APB1 timers)                          │
│      uint32_t timer_clock = 84000000;                               │
│      uint32_t prescaler = (timer_clock / 1000000) - 1; // 1 MHz     │
│      uint32_t period = config->period_us - 1;                       │
│                                                                      │
│      timer->PSC = prescaler;                                        │
│      timer->ARR = period;                                           │
│      timer->CR1 = TIM_CR1_ARPE;  // Auto-reload preload             │
│                                                                      │
│      if (config->mode == TIMER_MODE_PWM) {                          │
│          // Configure PWM mode 1 on channel 1                       │
│          timer->CCMR1 = (6 << 4) | TIM_CCMR1_OC1PE;                 │
│          timer->CCER = TIM_CCER_CC1E;  // Enable output             │
│      } else {                                                        │
│          // Enable update interrupt                                 │
│          timer->DIER = TIM_DIER_UIE;                                │
│          NVIC_EnableIRQ(TIM2_IRQn);                                 │
│      }                                                               │
│                                                                      │
│      return 0;                                                       │
│  }                                                                   │
│                                                                      │
│  void timer_pwm_set_duty(uint8_t channel, uint8_t duty_percent) {   │
│      uint32_t compare_val = (timer->ARR * duty_percent) / 100;      │
│      switch(channel) {                                               │
│          case 1: timer->CCR1 = compare_val; break;                  │
│          case 2: timer->CCR2 = compare_val; break;                  │
│          case 3: timer->CCR3 = compare_val; break;                  │
│          case 4: timer->CCR4 = compare_val; break;                  │
│      }                                                               │
│  }                                                                   │
│                                                                      │
│  // Timer ISR                                                        │
│  void TIM2_IRQHandler(void) {                                       │
│      if (timer->SR & TIM_SR_UIF) {                                  │
│          timer->SR &= ~TIM_SR_UIF;  // Clear flag                   │
│          if (timer_callback) {                                      │
│              timer_callback();                                       │
│          }                                                           │
│      }                                                               │
│  }                                                                   │
│                                                                      │
│  USAGE EXAMPLE:                                                      │
│                                                                      │
│  void blink_callback(void) {                                        │
│      gpio_toggle(0, 5);  // Toggle LED                              │
│  }                                                                   │
│                                                                      │
│  timer_config_t config = {                                          │
│      .periph    = TIM2,                                             │
│      .mode      = TIMER_MODE_PERIODIC,                              │
│      .period_us = 500000,  // 500ms                                 │
│      .callback  = blink_callback                                    │
│  };                                                                  │
│                                                                      │
│  timer_init(&config);                                               │
│  timer_start();                                                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Driver | Key Functions | Interrupt Usage |
|--------|--------------|-----------------|
| **GPIO** | init, read, write, toggle | External interrupts (EXTI) |
| **UART** | init, read, write, flush | TX empty, RX not empty |
| **I2C** | init, read, write | Event, Error, DMA |
| **SPI** | init, transfer, read, write | TX empty, RX not empty |
| **Timer** | init, start, stop, set_period | Update, Capture/Compare |

---

## Quick Revision Questions

1. **What are the essential functions that every device driver should implement?**

2. **Why are ring buffers used in UART drivers? Draw a diagram showing head and tail pointers.**

3. **Explain the I2C read transaction sequence including START, RESTART, and STOP conditions.**

4. **What is the purpose of the BSRR register in GPIO drivers compared to ODR?**

5. **How do you calculate prescaler and period values for a timer to generate a specific frequency?**

6. **What precautions must be taken when accessing shared data between ISR and main code?**

---

## Navigation

| Previous | Up | Next |
|----------|------|------|
| [Firmware Architecture](03-firmware-architecture.md) | [Unit 3 Index](README.md) | [Boot Sequence](05-boot-sequence.md) |

---
