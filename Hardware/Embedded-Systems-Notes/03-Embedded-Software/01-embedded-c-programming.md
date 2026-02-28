# Chapter 3.1: Embedded C Programming

## Chapter Overview

Embedded C is an extension of standard C with additional features for low-level hardware access. This chapter covers embedded-specific C programming techniques including volatile usage, bit manipulation, memory-mapped I/O, and efficient coding practices for resource-constrained systems.

---

## 3.1.1 Embedded C vs Standard C

### Key Differences

```
┌─────────────────────────────────────────────────────────────────────┐
│                EMBEDDED C vs STANDARD C                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Feature          │ Standard C      │ Embedded C            │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Runtime library  │ Full libc       │ Minimal/custom        │    │
│  │ Heap usage       │ Common (malloc) │ Often avoided         │    │
│  │ Stack size       │ Large           │ Limited, fixed        │    │
│  │ I/O operations   │ printf/scanf    │ Direct register access│    │
│  │ Floating point   │ Native support  │ Often emulated/avoid  │    │
│  │ Memory model     │ Virtual memory  │ Physical addresses    │    │
│  │ Interrupts       │ Not applicable  │ Critical feature      │    │
│  │ volatile keyword │ Rarely used     │ Essential             │    │
│  │ Optimization     │ Speed focus     │ Size + speed          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  EMBEDDED C EXTENSIONS:                                              │
│                                                                      │
│  1. Direct Hardware Access                                           │
│     • Memory-mapped I/O                                             │
│     • Bit manipulation macros                                       │
│     • Interrupt service routines                                    │
│                                                                      │
│  2. Compiler-Specific Features                                       │
│     • Inline assembly                                               │
│     • Section attributes                                            │
│     • Packed structures                                             │
│                                                                      │
│  3. Fixed-Point Arithmetic                                          │
│     • When FPU is not available                                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.1.2 The volatile Keyword

### Understanding volatile

```
┌─────────────────────────────────────────────────────────────────────┐
│                    THE VOLATILE KEYWORD                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PURPOSE: Prevents compiler from optimizing away memory accesses    │
│                                                                      │
│  WITHOUT VOLATILE (Compiler may optimize):                          │
│                                                                      │
│  Code:                        Compiler may generate:                │
│  ┌──────────────────────┐     ┌──────────────────────┐             │
│  │ int *status = 0x4000;│     │ // Read once, cache  │             │
│  │ while(*status == 0) {│     │ int temp = *status;  │             │
│  │     // wait          │     │ while(temp == 0) {   │             │
│  │ }                    │     │     // infinite loop!│             │
│  └──────────────────────┘     └──────────────────────┘             │
│                                                                      │
│  WITH VOLATILE (Correct behavior):                                   │
│                                                                      │
│  Code:                        Compiler generates:                   │
│  ┌───────────────────────────┐ ┌──────────────────────┐            │
│  │ volatile int *status = ..;│ │ // Read every time   │            │
│  │ while(*status == 0) {     │ │ loop:                │            │
│  │     // wait               │ │   LDR R0, [status]   │            │
│  │ }                         │ │   CMP R0, #0         │            │
│  └───────────────────────────┘ │   BEQ loop           │            │
│                                 └──────────────────────┘            │
│                                                                      │
│  WHEN TO USE VOLATILE:                                               │
│                                                                      │
│  1. Hardware Registers                                               │
│     volatile uint32_t *GPIO = (volatile uint32_t *)0x40020000;      │
│                                                                      │
│  2. Interrupt Shared Variables                                       │
│     volatile int flag = 0;  // Modified in ISR                      │
│                                                                      │
│  3. Memory-Mapped Peripherals                                        │
│     #define UART_DR  (*(volatile uint32_t *)0x40011004)             │
│                                                                      │
│  4. Status Registers (may change without CPU action)                │
│     while(!(*(volatile uint32_t *)0x40011000 & 0x80));              │
│                                                                      │
│  VOLATILE DOES NOT:                                                  │
│  • Provide atomicity (use critical sections)                        │
│  • Act as a memory barrier (use barriers if needed)                 │
│  • Make variable thread-safe (still need synchronization)           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.1.3 Memory-Mapped I/O

### Register Access Patterns

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MEMORY-MAPPED I/O                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  MEMORY MAP CONCEPT:                                                 │
│                                                                      │
│  Address Space                                                       │
│  ┌─────────────────────┐  0xFFFFFFFF                               │
│  │   System Control    │                                            │
│  ├─────────────────────┤  0xE0000000                               │
│  │    Peripherals      │  ◄─── GPIO, UART, SPI, I2C, Timers        │
│  │   (Memory-Mapped)   │                                            │
│  ├─────────────────────┤  0x40000000                               │
│  │    SRAM (Data)      │                                            │
│  ├─────────────────────┤  0x20000000                               │
│  │    Flash (Code)     │                                            │
│  └─────────────────────┘  0x00000000                               │
│                                                                      │
│  REGISTER ACCESS METHODS:                                            │
│                                                                      │
│  Method 1: Direct Pointer                                            │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ #define GPIO_BASE  0x40020000                                │   │
│  │ #define GPIO_ODR   (*(volatile uint32_t *)(GPIO_BASE + 0x14))│   │
│  │                                                              │   │
│  │ GPIO_ODR = 0x0001;  // Set pin 0                             │   │
│  │ GPIO_ODR |= 0x0002; // Set pin 1 (read-modify-write)         │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  Method 2: Struct Overlay                                            │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ typedef struct {                                             │   │
│  │     volatile uint32_t MODER;    // Offset 0x00              │   │
│  │     volatile uint32_t OTYPER;   // Offset 0x04              │   │
│  │     volatile uint32_t OSPEEDR;  // Offset 0x08              │   │
│  │     volatile uint32_t PUPDR;    // Offset 0x0C              │   │
│  │     volatile uint32_t IDR;      // Offset 0x10              │   │
│  │     volatile uint32_t ODR;      // Offset 0x14              │   │
│  │     volatile uint32_t BSRR;     // Offset 0x18              │   │
│  │ } GPIO_TypeDef;                                              │   │
│  │                                                              │   │
│  │ #define GPIOA ((GPIO_TypeDef *)0x40020000)                   │   │
│  │                                                              │   │
│  │ GPIOA->ODR = 0x0001;  // Much cleaner!                       │   │
│  │ GPIOA->BSRR = 0x0002; // Atomic set                          │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  STRUCT PADDING AND PACKING:                                         │
│                                                                      │
│  // Ensure no padding between members                               │
│  typedef struct __attribute__((packed)) {                           │
│      uint8_t  status;                                               │
│      uint32_t data;   // Would normally be at offset 4              │
│  } PackedReg;         // With packed: offset 1                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.1.4 Bit Manipulation

### Bit Operation Techniques

```
┌─────────────────────────────────────────────────────────────────────┐
│                    BIT MANIPULATION                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  BASIC BIT OPERATIONS:                                               │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Operation     │ Code                │ Binary Example        │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Set bit n     │ reg |= (1 << n)     │ 0000 | 0100 = 0100   │    │
│  │ Clear bit n   │ reg &= ~(1 << n)    │ 0111 & 1011 = 0011   │    │
│  │ Toggle bit n  │ reg ^= (1 << n)     │ 0100 ^ 0100 = 0000   │    │
│  │ Check bit n   │ (reg & (1 << n))    │ 0100 & 0100 = 0100   │    │
│  │ Set bits      │ reg |= mask         │ 0001 | 1100 = 1101   │    │
│  │ Clear bits    │ reg &= ~mask        │ 1111 & 0011 = 0011   │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  COMMON BIT MACROS:                                                  │
│                                                                      │
│  #define BIT(n)           (1UL << (n))                              │
│  #define SET_BIT(reg,n)   ((reg) |= BIT(n))                         │
│  #define CLR_BIT(reg,n)   ((reg) &= ~BIT(n))                        │
│  #define TGL_BIT(reg,n)   ((reg) ^= BIT(n))                         │
│  #define GET_BIT(reg,n)   (((reg) >> (n)) & 1)                      │
│                                                                      │
│  BIT FIELD EXTRACTION:                                               │
│                                                                      │
│  // Extract bits [high:low] from value                              │
│  #define BITS(val,high,low) \                                       │
│      (((val) >> (low)) & ((1UL << ((high)-(low)+1)) - 1))           │
│                                                                      │
│  Example: Extract bits [7:4] from 0xAB                              │
│  BITS(0xAB, 7, 4) = (0xAB >> 4) & 0x0F = 0x0A                       │
│                                                                      │
│  BIT FIELD MODIFICATION:                                             │
│                                                                      │
│  // Modify bits [high:low] to new_val                               │
│  #define MODIFY_BITS(reg, high, low, val) \                         │
│      do { \                                                          │
│          uint32_t mask = ((1UL << ((high)-(low)+1)) - 1) << (low); \│
│          (reg) = ((reg) & ~mask) | (((val) << (low)) & mask); \     │
│      } while(0)                                                      │
│                                                                      │
│  VISUAL BIT MANIPULATION:                                            │
│                                                                      │
│  Register: 0b 0 0 0 0  1 0 1 1   (0x0B)                             │
│               7 6 5 4  3 2 1 0                                       │
│                                                                      │
│  SET_BIT(reg, 5):                                                    │
│    Before: 0 0 0 0  1 0 1 1                                         │
│    Mask:   0 0 1 0  0 0 0 0    (1 << 5)                             │
│    OR:     ─────────────────                                         │
│    After:  0 0 1 0  1 0 1 1   (0x2B)                                │
│                                                                      │
│  CLR_BIT(reg, 1):                                                    │
│    Before: 0 0 1 0  1 0 1 1                                         │
│    ~Mask:  1 1 1 1  1 1 0 1   ~(1 << 1)                             │
│    AND:    ─────────────────                                         │
│    After:  0 0 1 0  1 0 0 1   (0x29)                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.1.5 Data Types and Fixed-Point Arithmetic

### Fixed-Width Integer Types

```
┌─────────────────────────────────────────────────────────────────────┐
│                    EMBEDDED DATA TYPES                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  STANDARD FIXED-WIDTH TYPES (stdint.h):                             │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Type       │ Size    │ Range                    │ Use Case  │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ uint8_t    │ 1 byte  │ 0 to 255                │ Registers │    │
│  │ int8_t     │ 1 byte  │ -128 to 127             │ Signed    │    │
│  │ uint16_t   │ 2 bytes │ 0 to 65535              │ Peripherals│   │
│  │ int16_t    │ 2 bytes │ -32768 to 32767         │ ADC values│    │
│  │ uint32_t   │ 4 bytes │ 0 to 4294967295         │ Addresses │    │
│  │ int32_t    │ 4 bytes │ -2³¹ to 2³¹-1           │ Counters  │    │
│  │ uint64_t   │ 8 bytes │ 0 to 2⁶⁴-1              │ Timestamps│    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  WHY USE FIXED-WIDTH TYPES:                                          │
│  • int size varies (16-bit on AVR, 32-bit on ARM)                   │
│  • Portable across architectures                                    │
│  • Clear memory requirements                                        │
│  • Essential for hardware register mapping                          │
│                                                                      │
│  FIXED-POINT ARITHMETIC (when no FPU):                              │
│                                                                      │
│  Q-format Representation:                                            │
│  Q8.8 = 8 integer bits + 8 fractional bits (16-bit total)          │
│                                                                      │
│  Binary:  0 0 0 0 0 0 1 0 . 1 0 0 0 0 0 0 0                         │
│           ───────────────   ───────────────                          │
│           Integer (2)       Fraction (0.5)                          │
│           = 2.5                                                      │
│                                                                      │
│  FIXED-POINT OPERATIONS:                                             │
│                                                                      │
│  typedef int16_t fixed_t;  // Q8.8 format                           │
│  #define FIXED_SHIFT 8                                               │
│  #define FLOAT_TO_FIXED(f) ((fixed_t)((f) * (1 << FIXED_SHIFT)))    │
│  #define FIXED_TO_FLOAT(x) ((float)(x) / (1 << FIXED_SHIFT))        │
│  #define FIXED_MUL(a,b)    (((int32_t)(a) * (b)) >> FIXED_SHIFT)    │
│  #define FIXED_DIV(a,b)    (((int32_t)(a) << FIXED_SHIFT) / (b))    │
│                                                                      │
│  Example: 2.5 * 1.5 = 3.75                                          │
│  fixed_t a = FLOAT_TO_FIXED(2.5);  // 0x0280 (640)                  │
│  fixed_t b = FLOAT_TO_FIXED(1.5);  // 0x0180 (384)                  │
│  fixed_t c = FIXED_MUL(a, b);      // (640*384)>>8 = 960 = 3.75     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.1.6 Interrupt Handling in C

### ISR Implementation

```
┌─────────────────────────────────────────────────────────────────────┐
│                    INTERRUPT HANDLING IN C                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  INTERRUPT SERVICE ROUTINE (ISR) STRUCTURE:                          │
│                                                                      │
│  // ARM Cortex-M style                                               │
│  void TIM2_IRQHandler(void) {                                       │
│      if (TIM2->SR & TIM_SR_UIF) {     // Check interrupt flag       │
│          TIM2->SR &= ~TIM_SR_UIF;     // Clear flag                 │
│          // Handle interrupt                                         │
│          timer_tick_count++;                                         │
│      }                                                               │
│  }                                                                   │
│                                                                      │
│  // AVR style (with attribute)                                       │
│  ISR(TIMER1_OVF_vect) {                                             │
│      // Handle timer overflow                                        │
│      TCNT1 = preload_value;                                         │
│  }                                                                   │
│                                                                      │
│  ISR BEST PRACTICES:                                                 │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                          DO                                 │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ ✓ Keep ISRs short and fast                                  │    │
│  │ ✓ Use volatile for shared variables                         │    │
│  │ ✓ Clear interrupt flags                                     │    │
│  │ ✓ Use flags to signal main loop                             │    │
│  │ ✓ Save/restore context if needed                            │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                         DON'T                               │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ ✗ Call blocking functions (delay, printf)                   │    │
│  │ ✗ Use malloc/free                                           │    │
│  │ ✗ Perform long computations                                 │    │
│  │ ✗ Access non-reentrant functions                            │    │
│  │ ✗ Forget to clear interrupt flags (causes re-entry)         │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  SHARED VARIABLE PROTECTION:                                         │
│                                                                      │
│  volatile uint32_t shared_counter;                                   │
│                                                                      │
│  // In ISR:                                                          │
│  void SysTick_Handler(void) {                                       │
│      shared_counter++;                                               │
│  }                                                                   │
│                                                                      │
│  // In main (atomic read for multi-byte):                           │
│  uint32_t get_counter(void) {                                       │
│      uint32_t temp;                                                  │
│      __disable_irq();        // Critical section start              │
│      temp = shared_counter;                                          │
│      __enable_irq();         // Critical section end                │
│      return temp;                                                    │
│  }                                                                   │
│                                                                      │
│  ISR EXECUTION FLOW:                                                 │
│                                                                      │
│  Main ───────────────┐                 ┌───────────────────         │
│  Code                │                 │                            │
│                      │   ┌─────────────┘                            │
│                      │   │                                          │
│               IRQ ──►│   │◄── Return from ISR                       │
│                      │   │                                          │
│                      └───┼───┐                                      │
│                          │   │                                      │
│  ISR Code  ──────────────┴───┴──────────                            │
│            Save    Execute   Restore                                │
│            Context   ISR     Context                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.1.7 Code Optimization Techniques

### Embedded-Specific Optimizations

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CODE OPTIMIZATION                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SIZE vs SPEED OPTIMIZATION:                                         │
│                                                                      │
│  Compiler Flags:                                                     │
│  -Os  → Optimize for size (preferred for embedded)                  │
│  -O2  → Optimize for speed                                          │
│  -O3  → Aggressive optimization (may increase size)                 │
│                                                                      │
│  LOOP OPTIMIZATION:                                                  │
│                                                                      │
│  // Inefficient                    // Optimized                     │
│  for(i = 0; i < len; i++) {       for(i = len; i > 0; i--) {       │
│      // compare each iteration        // decrement + compare = 1   │
│  }                                 }                                │
│                                                                      │
│  // Loop unrolling                                                   │
│  for(i = 0; i < 16; i += 4) {                                       │
│      arr[i]   = val;                                                │
│      arr[i+1] = val;                                                │
│      arr[i+2] = val;                                                │
│      arr[i+3] = val;                                                │
│  }                                                                   │
│                                                                      │
│  LOOKUP TABLES vs COMPUTATION:                                       │
│                                                                      │
│  // Computation (slow)                                               │
│  uint8_t sin_val = (uint8_t)(127 * sin(angle * 3.14159 / 180));     │
│                                                                      │
│  // Lookup table (fast)                                              │
│  const uint8_t sin_table[360] = {0, 2, 4, 7, 9, ...};               │
│  uint8_t sin_val = sin_table[angle];                                │
│                                                                      │
│  INLINE FUNCTIONS:                                                   │
│                                                                      │
│  // Avoid function call overhead for small functions                │
│  static inline void gpio_set_high(uint8_t pin) {                    │
│      GPIOA->ODR |= (1 << pin);                                      │
│  }                                                                   │
│                                                                      │
│  REGISTER VARIABLES:                                                 │
│                                                                      │
│  // Hint to keep in CPU register                                    │
│  register uint8_t loop_count = 0;                                   │
│                                                                      │
│  CONST AND STATIC:                                                   │
│                                                                      │
│  // const → placed in Flash (saves RAM)                             │
│  const uint8_t lookup[256] = {...};                                 │
│                                                                      │
│  // static → no re-initialization overhead                          │
│  static uint8_t buffer[64];                                         │
│                                                                      │
│  MEMORY SECTIONS:                                                    │
│                                                                      │
│  // Place critical code in RAM for faster execution                 │
│  __attribute__((section(".ram_func")))                              │
│  void fast_function(void) {                                         │
│      // Time-critical code                                          │
│  }                                                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Description | Example |
|---------|-------------|---------|
| **volatile** | Prevents optimization, forces memory access | `volatile uint32_t *reg` |
| **Memory-mapped I/O** | Peripheral access via pointers | `*(volatile uint32_t*)0x40000` |
| **Bit manipulation** | Set/clear/toggle individual bits | `reg |= (1 << n)` |
| **Fixed-width types** | Guaranteed size across platforms | `uint8_t`, `uint32_t` |
| **Fixed-point** | Fractional math without FPU | Q8.8 format |
| **ISR** | Interrupt handler function | Keep short, use flags |
| **inline** | Eliminate function call overhead | `static inline void f()` |

---

## Quick Revision Questions

1. **Why is the `volatile` keyword essential in embedded C? Give two examples where it must be used.**

2. **Write a macro to toggle bit 5 of a register called `PORTA`.**

3. **What is the difference between `uint16_t` and `unsigned int` in embedded systems?**

4. **Convert the value 3.25 to Q8.8 fixed-point representation (show binary).**

5. **What are the key rules to follow when writing an interrupt service routine?**

6. **Explain why struct packing is important when mapping hardware registers.**

---

## Navigation

| Previous | Up | Next |
|----------|------|------|
| [Unit 3 Index](README.md) | [Unit 3 Index](README.md) | [Cross-Compilation](02-cross-compilation.md) |

---
