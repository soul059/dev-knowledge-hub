# Chapter 7.2: Hardware-Software Co-Design

## Introduction

Hardware-software co-design is the concurrent development of hardware and software components in an embedded system. Unlike traditional sequential approaches, co-design optimizes the entire system by making partitioning decisions early and considering trade-offs between hardware and software implementations throughout the design process.

---

## The Co-Design Philosophy

```
┌─────────────────────────────────────────────────────────────────────┐
│              TRADITIONAL vs CO-DESIGN APPROACH                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  TRADITIONAL (Sequential):                                          │
│                                                                      │
│  ┌────────────┐                                                     │
│  │ HARDWARE   │──▶ Design ──▶ Build ──▶ Done                       │
│  │ DESIGN     │                                                     │
│  └────────────┘                                                     │
│                    └────────────────────┐                           │
│                                         ▼                           │
│  ┌────────────┐                    ┌─────────────┐                 │
│  │ SOFTWARE   │──▶ Wait... ───────▶│ Start SW    │──▶ Done        │
│  │ DESIGN     │    (blocked)       └─────────────┘                 │
│  └────────────┘                                                     │
│                                                                      │
│  Problem: Late integration, limited optimization, schedule risk     │
│                                                                      │
│  ════════════════════════════════════════════════════════════════   │
│                                                                      │
│  CO-DESIGN (Concurrent):                                            │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │              SYSTEM SPECIFICATION                             │  │
│  └─────────────────────────┬────────────────────────────────────┘  │
│                            │                                        │
│                            ▼                                        │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │           HW/SW PARTITIONING DECISION                         │  │
│  └──────────┬───────────────────────────────────┬───────────────┘  │
│             │                                   │                   │
│             ▼                                   ▼                   │
│  ┌─────────────────────┐           ┌─────────────────────┐        │
│  │   HARDWARE DESIGN   │◀═════════▶│   SOFTWARE DESIGN   │        │
│  │                     │  Feedback  │                     │        │
│  │   • Schematic       │  & Sync    │   • Architecture    │        │
│  │   • PCB             │           │   • Drivers          │        │
│  │   • FPGA            │           │   • Application      │        │
│  └──────────┬──────────┘           └──────────┬──────────┘        │
│             │                                   │                   │
│             └──────────────┬────────────────────┘                   │
│                            ▼                                        │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │              SYSTEM INTEGRATION & TEST                        │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  Benefits: Early validation, optimal partitioning, faster TTM       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Hardware/Software Partitioning

### Partitioning Decision Factors

```
┌─────────────────────────────────────────────────────────────────────┐
│              HARDWARE vs SOFTWARE IMPLEMENTATION                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  FACTOR        │ HARDWARE (HW)          │ SOFTWARE (SW)             │
│  ══════════════╪════════════════════════╪═════════════════════════  │
│  Speed         │ Very fast (ns-μs)      │ Slower (μs-ms)           │
│  Flexibility   │ Fixed after design     │ Easy to modify           │
│  Power         │ Can be very efficient  │ Generally higher         │
│  Cost (NRE)    │ High development cost  │ Lower development cost   │
│  Cost (unit)   │ Higher for custom HW   │ Lower marginal cost      │
│  Parallelism   │ True parallel exec     │ Sequential (mostly)      │
│  Development   │ Longer, specialized    │ Faster, more engineers   │
│  Debugging     │ Harder, special tools  │ Easier, simulators       │
│                                                                      │
│  DECISION MATRIX:                                                    │
│                                                                      │
│                    ┌────────────────────────────────────┐           │
│                    │          IMPLEMENTATION            │           │
│                    ├─────────────┬──────────────────────┤           │
│                    │  SOFTWARE   │      HARDWARE        │           │
│  ┌────────────────┼─────────────┼──────────────────────┤           │
│  │ Timing         │ Flexible    │ Hard real-time       │           │
│  │ requirements   │ (ms range)  │ (ns-μs range)        │           │
│  ├────────────────┼─────────────┼──────────────────────┤           │
│  │ Algorithm      │ Complex,    │ Simple, fixed        │           │
│  │ complexity     │ evolving    │ parallel data        │           │
│  ├────────────────┼─────────────┼──────────────────────┤           │
│  │ Production     │ Low-medium  │ High volume          │           │
│  │ volume         │ volume      │ (amortize NRE)       │           │
│  ├────────────────┼─────────────┼──────────────────────┤           │
│  │ Power budget   │ More        │ Tight power          │           │
│  │                │ flexible    │ constraints          │           │
│  └────────────────┴─────────────┴──────────────────────┘           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Partitioning Examples

```
┌─────────────────────────────────────────────────────────────────────┐
│              PARTITIONING EXAMPLE: VIDEO PROCESSOR                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  VIDEO INPUT ──▶ DECODE ──▶ PROCESS ──▶ ENCODE ──▶ OUTPUT          │
│                                                                      │
│  OPTION A: All Software (DSP/CPU)                                   │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                  POWERFUL CPU/DSP                             │  │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐            │  │
│  │  │ Decode  │→│ Process │→│ Encode  │→│ Output  │            │  │
│  │  │  (SW)   │ │  (SW)   │ │  (SW)   │ │  (SW)   │            │  │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘            │  │
│  └──────────────────────────────────────────────────────────────┘  │
│  ✓ Flexible    ✗ High power    ✗ Expensive CPU                    │
│                                                                      │
│  OPTION B: Hardware Accelerators                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  ┌─────────┐              ┌─────────┐                        │  │
│  │  │ Decode  │   ┌──────┐   │ Encode  │                        │  │
│  │  │  (HW)   │──▶│ CPU  │──▶│  (HW)   │                        │  │
│  │  │ ASIC    │   │ (SW) │   │ ASIC    │                        │  │
│  │  └─────────┘   │Process│   └─────────┘                        │  │
│  │                └──────┘                                       │  │
│  └──────────────────────────────────────────────────────────────┘  │
│  ✓ Lower power   ✓ Smaller CPU   ✗ Less flexible                  │
│                                                                      │
│  OPTION C: FPGA + CPU                                               │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │        FPGA                              CPU                  │  │
│  │  ┌─────────────────────┐           ┌──────────┐             │  │
│  │  │ Decode │ Preprocess │◀─────────▶│ Control  │             │  │
│  │  │ Encode │ Filters    │           │ Config   │             │  │
│  │  └─────────────────────┘           └──────────┘             │  │
│  └──────────────────────────────────────────────────────────────┘  │
│  ✓ Field updateable   ✓ Good balance   ✗ FPGA cost               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Hardware Abstraction Layer (HAL)

### HAL Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│              HARDWARE ABSTRACTION LAYER                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                     APPLICATION                               │  │
│  │         (Business logic, algorithms, UI)                      │  │
│  └──────────────────────────┬───────────────────────────────────┘  │
│                             │                                       │
│  ┌──────────────────────────▼───────────────────────────────────┐  │
│  │                    MIDDLEWARE/RTOS                            │  │
│  │         (OS services, communication stacks)                   │  │
│  └──────────────────────────┬───────────────────────────────────┘  │
│                             │                                       │
│  ┌──────────────────────────▼───────────────────────────────────┐  │
│  │                HARDWARE ABSTRACTION LAYER                     │  │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐     │  │
│  │  │ GPIO   │ │ UART   │ │ SPI    │ │ Timer  │ │  ADC   │     │  │
│  │  │ HAL    │ │ HAL    │ │ HAL    │ │ HAL    │ │  HAL   │     │  │
│  │  └────────┘ └────────┘ └────────┘ └────────┘ └────────┘     │  │
│  └──────────────────────────┬───────────────────────────────────┘  │
│  ═══════════════════════════│═══════════════════════════════════   │
│                             │  HARDWARE INTERFACE                   │
│  ┌──────────────────────────▼───────────────────────────────────┐  │
│  │                      HARDWARE                                 │  │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐     │  │
│  │  │  I/O   │ │ USART  │ │  SPI   │ │ TIMx   │ │  ADC   │     │  │
│  │  │ Ports  │ │ Periph │ │ Periph │ │ Periph │ │ Periph │     │  │
│  │  └────────┘ └────────┘ └────────┘ └────────┘ └────────┘     │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  BENEFITS OF HAL:                                                    │
│  • Portability: Same app code works on different MCUs              │
│  • Testability: Mock hardware for unit testing                     │
│  • Maintainability: Hardware changes isolated to HAL               │
│  • Parallel development: SW team can work with HAL stubs           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### HAL Implementation Example

```c
/* ================================================================
 * GPIO HAL Interface (platform-independent header)
 * ================================================================ */

/* hal_gpio.h */
#ifndef HAL_GPIO_H
#define HAL_GPIO_H

#include <stdint.h>
#include <stdbool.h>

/* Port and pin abstraction */
typedef uint8_t gpio_port_t;
typedef uint8_t gpio_pin_t;

/* Pin mode options */
typedef enum {
    GPIO_MODE_INPUT,
    GPIO_MODE_INPUT_PULLUP,
    GPIO_MODE_INPUT_PULLDOWN,
    GPIO_MODE_OUTPUT_PP,      /* Push-pull output */
    GPIO_MODE_OUTPUT_OD,      /* Open-drain output */
    GPIO_MODE_ANALOG
} gpio_mode_t;

/* HAL function prototypes */
void hal_gpio_init(gpio_port_t port, gpio_pin_t pin, gpio_mode_t mode);
void hal_gpio_write(gpio_port_t port, gpio_pin_t pin, bool value);
bool hal_gpio_read(gpio_port_t port, gpio_pin_t pin);
void hal_gpio_toggle(gpio_port_t port, gpio_pin_t pin);

#endif /* HAL_GPIO_H */


/* ================================================================
 * STM32F4 HAL Implementation
 * ================================================================ */

/* hal_gpio_stm32f4.c */
#include "hal_gpio.h"
#include "stm32f4xx.h"

/* Port mapping table */
static GPIO_TypeDef* const port_map[] = {
    GPIOA, GPIOB, GPIOC, GPIOD, GPIOE, GPIOF
};

void hal_gpio_init(gpio_port_t port, gpio_pin_t pin, gpio_mode_t mode) {
    GPIO_TypeDef *gpio = port_map[port];
    
    /* Enable port clock */
    RCC->AHB1ENR |= (1 << port);
    
    /* Configure mode */
    gpio->MODER &= ~(3U << (pin * 2));  /* Clear mode bits */
    
    switch (mode) {
        case GPIO_MODE_INPUT:
        case GPIO_MODE_INPUT_PULLUP:
        case GPIO_MODE_INPUT_PULLDOWN:
            /* MODER = 00 (input) */
            break;
            
        case GPIO_MODE_OUTPUT_PP:
        case GPIO_MODE_OUTPUT_OD:
            gpio->MODER |= (1U << (pin * 2));  /* MODER = 01 */
            break;
            
        case GPIO_MODE_ANALOG:
            gpio->MODER |= (3U << (pin * 2));  /* MODER = 11 */
            break;
    }
    
    /* Configure pull-up/down */
    gpio->PUPDR &= ~(3U << (pin * 2));
    if (mode == GPIO_MODE_INPUT_PULLUP) {
        gpio->PUPDR |= (1U << (pin * 2));
    } else if (mode == GPIO_MODE_INPUT_PULLDOWN) {
        gpio->PUPDR |= (2U << (pin * 2));
    }
    
    /* Configure output type */
    if (mode == GPIO_MODE_OUTPUT_OD) {
        gpio->OTYPER |= (1U << pin);
    } else {
        gpio->OTYPER &= ~(1U << pin);
    }
}

void hal_gpio_write(gpio_port_t port, gpio_pin_t pin, bool value) {
    GPIO_TypeDef *gpio = port_map[port];
    if (value) {
        gpio->BSRR = (1U << pin);        /* Set bit */
    } else {
        gpio->BSRR = (1U << (pin + 16)); /* Reset bit */
    }
}

bool hal_gpio_read(gpio_port_t port, gpio_pin_t pin) {
    GPIO_TypeDef *gpio = port_map[port];
    return (gpio->IDR & (1U << pin)) != 0;
}

void hal_gpio_toggle(gpio_port_t port, gpio_pin_t pin) {
    GPIO_TypeDef *gpio = port_map[port];
    gpio->ODR ^= (1U << pin);
}


/* ================================================================
 * Application code (platform-independent)
 * ================================================================ */

#include "hal_gpio.h"

/* Board-specific pin definitions */
#define LED_PORT        0  /* Port A */
#define LED_PIN         5  /* Pin 5 */

#define BUTTON_PORT     2  /* Port C */
#define BUTTON_PIN      13

void app_init(void) {
    hal_gpio_init(LED_PORT, LED_PIN, GPIO_MODE_OUTPUT_PP);
    hal_gpio_init(BUTTON_PORT, BUTTON_PIN, GPIO_MODE_INPUT_PULLUP);
}

void app_run(void) {
    if (!hal_gpio_read(BUTTON_PORT, BUTTON_PIN)) {
        hal_gpio_write(LED_PORT, LED_PIN, true);
    } else {
        hal_gpio_write(LED_PORT, LED_PIN, false);
    }
}
```

---

## Interface Definition

### Communication Between HW and SW

```
┌─────────────────────────────────────────────────────────────────────┐
│              HW/SW INTERFACE TYPES                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. MEMORY-MAPPED REGISTERS                                         │
│     ┌────────────────────────────────────────────────────────────┐ │
│     │  CPU ADDRESS SPACE                                         │ │
│     │  ┌──────────────┐                                          │ │
│     │  │ Flash Memory │ 0x08000000 - 0x080FFFFF                 │ │
│     │  ├──────────────┤                                          │ │
│     │  │ SRAM         │ 0x20000000 - 0x2001FFFF                 │ │
│     │  ├──────────────┤                                          │ │
│     │  │ Peripherals  │ 0x40000000 - 0x5FFFFFFF                 │ │
│     │  │  ├─ GPIO     │  0x40020000                             │ │
│     │  │  ├─ USART    │  0x40011000                             │ │
│     │  │  └─ SPI      │  0x40013000                             │ │
│     │  └──────────────┘                                          │ │
│     └────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  2. INTERRUPTS                                                       │
│     ┌────────────────────────────────────────────────────────────┐ │
│     │                                                            │ │
│     │   Hardware ─────▶ IRQ ─────▶ ISR ─────▶ Software          │ │
│     │    Event        Signal     Handler      Action            │ │
│     │                                                            │ │
│     │   Examples:                                                │ │
│     │   • Timer overflow → Update system tick                   │ │
│     │   • UART RX complete → Read received byte                 │ │
│     │   • ADC conversion done → Process sample                  │ │
│     │   • External interrupt → Handle button press              │ │
│     │                                                            │ │
│     └────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  3. DMA (Direct Memory Access)                                      │
│     ┌────────────────────────────────────────────────────────────┐ │
│     │                                                            │ │
│     │  ┌────────────┐        ┌───────────┐        ┌──────────┐ │ │
│     │  │ Peripheral │◀──────▶│    DMA    │◀──────▶│  Memory  │ │ │
│     │  │ (ADC, SPI) │ Data   │ Controller│ Data   │  Buffer  │ │ │
│     │  └────────────┘        └─────┬─────┘        └──────────┘ │ │
│     │                              │                            │ │
│     │                         IRQ on complete                   │ │
│     │                              │                            │ │
│     │                              ▼                            │ │
│     │                         ┌────────┐                       │ │
│     │                         │  CPU   │ Process data          │ │
│     │                         └────────┘                       │ │
│     │                                                            │ │
│     │  Benefits: CPU freed during transfer, high throughput     │ │
│     └────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Resource Allocation

### Memory Partitioning

```
┌─────────────────────────────────────────────────────────────────────┐
│              MEMORY MAP DESIGN                                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  FLASH MEMORY LAYOUT (256KB example):                               │
│                                                                      │
│  Address        Size     Content                                     │
│  ─────────────────────────────────────────────────────────────────  │
│  0x0800 0000    16KB     Bootloader (protected)                     │
│  0x0800 4000     4KB     Configuration data                         │
│  0x0800 5000     4KB     Factory calibration (read-only)            │
│  0x0800 6000   104KB     Application firmware                       │
│  0x0802 0000   104KB     OTA update staging area                    │
│  0x0803 A000    24KB     Reserved for future use                    │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │░░░░░░░░│▓▓▓▓│████│████████████████████│▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒│   │ │
│  │ BOOT   │CFG │CAL │   APPLICATION      │    OTA UPDATE      │   │ │
│  │░░░░░░░░│▓▓▓▓│████│████████████████████│▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒│   │ │
│  └───────────────────────────────────────────────────────────────┘ │
│  0x0800_0000                                           0x0803_FFFF │
│                                                                      │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                      │
│  SRAM MEMORY LAYOUT (64KB example):                                 │
│                                                                      │
│  High Address ───────────────────────────────                       │
│      │    ┌──────────────────┐                                      │
│      │    │      STACK       │ ← SP (grows down)                   │
│      │    │    (8KB)         │                                      │
│      │    ├──────────────────┤ ← Stack limit                       │
│      │    │     GUARD        │   (protect from overflow)           │
│      │    ├──────────────────┤                                      │
│      │    │      HEAP        │                                      │
│      │    │    (16KB)        │ ← Dynamic allocation                │
│      │    ├──────────────────┤                                      │
│      │    │   BSS (zero)     │ ← Uninitialized globals            │
│      │    │    (8KB)         │                                      │
│      │    ├──────────────────┤                                      │
│      │    │   DATA (init)    │ ← Initialized globals              │
│      │    │    (4KB)         │                                      │
│      │    ├──────────────────┤                                      │
│      │    │  DMA BUFFERS     │ ← Must be aligned                   │
│      │    │    (8KB)         │                                      │
│      │    ├──────────────────┤                                      │
│      │    │  RTOS RESOURCES  │ ← Task stacks, queues              │
│      │    │   (20KB)         │                                      │
│      ▼    └──────────────────┘                                      │
│  Low Address ────────────────────────────────                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Timing Budget Analysis

```
┌─────────────────────────────────────────────────────────────────────┐
│              TIMING BUDGET ANALYSIS                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  EXAMPLE: 10ms Control Loop                                         │
│                                                                      │
│  Total Period: 10,000 μs                                            │
│                                                                      │
│  ACTIVITY              │ TIME (μs) │ % BUDGET │ CUMULATIVE          │
│  ──────────────────────┼───────────┼──────────┼─────────────────────│
│  Read ADC (8 channels) │    200    │   2.0%   │    200              │
│  Read digital inputs   │     20    │   0.2%   │    220              │
│  Sensor processing     │    800    │   8.0%   │   1020              │
│  Control algorithm     │   1500    │  15.0%   │   2520              │
│  PWM output update     │     50    │   0.5%   │   2570              │
│  Communication (CAN)   │    500    │   5.0%   │   3070              │
│  Logging/diagnostics   │    200    │   2.0%   │   3270              │
│  ──────────────────────┼───────────┼──────────┼─────────────────────│
│  TOTAL ACTIVE          │   3270    │  32.7%   │                     │
│  MARGIN/IDLE           │   6730    │  67.3%   │   10000             │
│                                                                      │
│  TIMING DIAGRAM:                                                     │
│                                                                      │
│  ├────┬──┬────────┬──────────────┬─┬─────┬──┬────────────────────┤ │
│  │ADC │IN│PROCESS │  CONTROL     │O│ CAN │L │     IDLE           │ │
│  ├────┴──┴────────┴──────────────┴─┴─────┴──┴────────────────────┤ │
│  0                                                              10ms │
│                                                                      │
│  DESIGN RULES:                                                       │
│  • Keep margin > 30% for worst-case variations                      │
│  • Account for interrupt latency                                    │
│  • Include cache misses, wait states                                │
│  • Profile on actual hardware                                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Trade-off Analysis

### Performance vs Resources

```
┌─────────────────────────────────────────────────────────────────────┐
│              IMPLEMENTATION TRADE-OFFS                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  EXAMPLE: FIR Filter Implementation                                 │
│                                                                      │
│  y[n] = Σ(k=0 to N-1) h[k] × x[n-k]                                │
│                                                                      │
│  OPTION 1: Pure Software (General CPU)                              │
│  ┌────────────────────────────────────────────────────────────────┐│
│  │ for (k = 0; k < N; k++) {                                      ││
│  │     y += h[k] * x[n - k];   // Many cycles per tap             ││
│  │ }                                                               ││
│  └────────────────────────────────────────────────────────────────┘│
│  • Execution: ~50 cycles/tap                                        │
│  • 64-tap @ 48MHz: 67μs                                            │
│  • Flexibility: High (change coefficients anytime)                 │
│  • Cost: Low (just CPU time)                                        │
│                                                                      │
│  OPTION 2: DSP Instructions (Cortex-M4/M7)                          │
│  ┌────────────────────────────────────────────────────────────────┐│
│  │ // Uses SIMD MAC instruction                                    ││
│  │ arm_fir_f32(&fir_instance, input, output, block_size);         ││
│  └────────────────────────────────────────────────────────────────┘│
│  • Execution: ~2 cycles/tap (with pipelining)                      │
│  • 64-tap @ 48MHz: 2.7μs                                           │
│  • Flexibility: High                                                │
│  • Cost: Slightly more expensive MCU                                │
│                                                                      │
│  OPTION 3: Hardware Accelerator (FPGA/ASIC)                         │
│  ┌────────────────────────────────────────────────────────────────┐│
│  │ Input ──▶ [z⁻¹]──▶[z⁻¹]──▶[z⁻¹]──▶ ...                        ││
│  │             │        │        │                                 ││
│  │             ×h[0]    ×h[1]    ×h[2]                             ││
│  │             │        │        │                                 ││
│  │             └────────┴────────┴────▶ Σ ──▶ Output              ││
│  └────────────────────────────────────────────────────────────────┘│
│  • Execution: 1 sample per clock cycle                             │
│  • 64-tap @ 48MHz: 21ns (3000x faster!)                            │
│  • Flexibility: Low (fixed at design time)                         │
│  • Cost: High (FPGA or custom silicon)                             │
│                                                                      │
│  SELECTION CRITERIA:                                                 │
│  ├─ Sample rate < 10kHz → Software OK                              │
│  ├─ Sample rate 10kHz-1MHz → DSP instructions                      │
│  └─ Sample rate > 1MHz → Hardware accelerator                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Make vs Buy Decision

```
┌─────────────────────────────────────────────────────────────────────┐
│              MAKE vs BUY ANALYSIS                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                         MAKE                    BUY                  │
│                    (Custom Design)         (Off-the-shelf)           │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │  COST ANALYSIS (Example: Motor Driver Module)                 │ │
│  │                                                                │ │
│  │  MAKE:                          BUY:                          │ │
│  │  • NRE: $15,000                 • Module: $25/unit            │ │
│  │  • Per unit: $12                • No NRE                      │ │
│  │  • Lead time: 12 weeks          • Lead time: 2 weeks          │ │
│  │                                                                │ │
│  │  BREAK-EVEN CALCULATION:                                      │ │
│  │  $15,000 + $12×N = $25×N                                     │ │
│  │  $15,000 = $13×N                                              │ │
│  │  N = 1,154 units                                              │ │
│  │                                                                │ │
│  │  DECISION:                                                    │ │
│  │  • < 1000 units → BUY                                        │ │
│  │  • > 1500 units → MAKE                                       │ │
│  │  • 1000-1500 → Consider other factors                        │ │
│  │                                                                │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  OTHER CONSIDERATIONS:                                               │
│                                                                      │
│  MAKE IF:                          BUY IF:                          │
│  ├─ Core competency                ├─ Commodity function            │
│  ├─ Competitive advantage          ├─ Time-to-market critical       │
│  ├─ High volume production         ├─ Low volume                    │
│  ├─ Special requirements           ├─ Standard requirements         │
│  ├─ Long product lifetime          ├─ Short lifetime                │
│  └─ Supply chain control           └─ Multiple suppliers            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Co-Design Workflow

```
┌─────────────────────────────────────────────────────────────────────┐
│              PRACTICAL CO-DESIGN WORKFLOW                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PHASE 1: SPECIFICATION (Week 1-2)                                  │
│  ├─ Define system requirements                                      │
│  ├─ Create initial architecture                                     │
│  ├─ Identify critical functions                                     │
│  └─ Estimate HW/SW split                                            │
│      │                                                               │
│      ▼                                                               │
│  PHASE 2: EXPLORATION (Week 3-4)                                    │
│  ├─ Evaluate MCU/FPGA options                                       │
│  ├─ Create behavioral models                                        │
│  ├─ Simulate timing/performance                                     │
│  └─ Refine partitioning                                             │
│      │                                                               │
│      ▼                                                               │
│  PHASE 3: PARALLEL DEVELOPMENT (Week 5-12)                          │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │  HARDWARE TEAM:           │  SOFTWARE TEAM:                     ││
│  │  • Schematic design       │  • HAL development                  ││
│  │  • Component selection    │  • Driver implementation            ││
│  │  • PCB layout             │  • Unit tests with stubs            ││
│  │  • Prototype build        │  • Simulation testing               ││
│  │                           │                                      ││
│  │  SYNC POINTS:             │                                      ││
│  │  ◆ Pin allocation review  │                                      ││
│  │  ◆ Interface definition   │                                      ││
│  │  ◆ Timing specification   │                                      ││
│  └─────────────────────────────────────────────────────────────────┘│
│      │                                                               │
│      ▼                                                               │
│  PHASE 4: INTEGRATION (Week 13-16)                                  │
│  ├─ First power-on testing                                          │
│  ├─ Driver validation on hardware                                   │
│  ├─ System integration testing                                      │
│  └─ Performance optimization                                        │
│      │                                                               │
│      ▼                                                               │
│  PHASE 5: VALIDATION (Week 17-20)                                   │
│  ├─ Requirements verification                                       │
│  ├─ Environmental testing                                           │
│  ├─ Compliance testing                                              │
│  └─ Production preparation                                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **Co-Design** | Concurrent HW/SW development with continuous optimization |
| **Partitioning** | Deciding what functions go in HW vs SW |
| **HAL** | Hardware Abstraction Layer for portability and testability |
| **Trade-offs** | Speed/power/cost/flexibility considerations |
| **Memory Map** | Organized allocation of Flash and SRAM |
| **Timing Budget** | Analysis of execution time vs available period |
| **Make vs Buy** | Economic analysis of custom vs off-the-shelf |
| **Interface Definition** | Registers, interrupts, DMA between HW/SW |

---

## Quick Revision Questions

1. **What are the main benefits of hardware-software co-design compared to sequential development?**

2. **List three factors that favor implementing a function in hardware rather than software.**

3. **What is a Hardware Abstraction Layer (HAL)? How does it enable parallel development between hardware and software teams?**

4. **Explain how you would perform a timing budget analysis for a 1ms control loop. What margin should be maintained?**

5. **A motor driver module costs $15,000 NRE + $12/unit to make, or $25/unit to buy. Calculate the break-even volume and recommend a decision for 500 units vs 5000 units.**

6. **What are the key synchronization points between hardware and software teams during co-design?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Chapter 7.1: Requirements and Specification](01-requirements-specification.md) | [Unit 7: System Design](README.md) | [Chapter 7.3: Prototyping and Development](03-prototyping-development.md) |

---
