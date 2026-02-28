# Chapter 6.2: Timers and PWM

## Introduction

Hardware timers are essential peripherals in microcontrollers. They provide precise timing for delays, periodic interrupts, pulse generation (PWM), and input measurement without CPU intervention.

---

## Timer Fundamentals

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TIMER BLOCK DIAGRAM                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Clock Source                                                        │
│      │                                                               │
│      ▼                                                               │
│  ┌───────────┐    ┌─────────────┐    ┌──────────────┐              │
│  │ Prescaler │───▶│  Counter    │───▶│ Compare/     │──▶ Output    │
│  │  (÷ N)    │    │  (CNT)      │    │ Capture Reg  │              │
│  └───────────┘    └──────┬──────┘    └──────────────┘              │
│                          │                                          │
│                          │ Overflow/Match                           │
│                          ▼                                          │
│                    ┌───────────┐                                    │
│                    │ Interrupt │                                    │
│                    │  (IRQ)    │                                    │
│                    └───────────┘                                    │
│                                                                      │
│  Timer Frequency = Clock / Prescaler                                │
│  Timer Period = (Reload + 1) × (1 / Timer_Frequency)               │
│                                                                      │
│  EXAMPLE:                                                            │
│  • System clock: 72 MHz                                             │
│  • Prescaler: 72                                                    │
│  • Timer frequency: 72 MHz / 72 = 1 MHz (1 µs per tick)            │
│  • Reload value: 999                                                │
│  • Period: (999 + 1) × 1 µs = 1 ms                                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Timer Modes

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TIMER COUNTING MODES                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  UP COUNTING (most common):                                         │
│                                                                      │
│  CNT  ARR ─ ─ ─ ─┬─ ─ ─ ─ ─┬─ ─ ─ ─ ─┬─ ─ ─                        │
│       │        ╱│        ╱│        ╱│                               │
│       │      ╱  │      ╱  │      ╱  │                               │
│       │    ╱    │    ╱    │    ╱    │                               │
│       │  ╱      │  ╱      │  ╱      │                               │
│     0 └╱────────└╱────────└╱────────▶ Time                          │
│         overflow  overflow                                          │
│                                                                      │
│  DOWN COUNTING:                                                      │
│                                                                      │
│  CNT  ARR ─┬─ ─ ─ ─ ─┬─ ─ ─ ─ ─┬─ ─ ─                              │
│       │    ╲        │╲        │╲                                    │
│       │      ╲      │  ╲      │  ╲                                  │
│       │        ╲    │    ╲    │    ╲                                │
│       │          ╲  │      ╲  │      ╲                              │
│     0 └───────────╲─└────────╲└────────▶ Time                       │
│              underflow                                               │
│                                                                      │
│  CENTER-ALIGNED (up-down):                                          │
│                                                                      │
│  CNT  ARR ─ ─ ─ ─╲╱─ ─ ─ ─ ─╲╱─ ─ ─ ─ ─╲╱─                         │
│       │        ╱  ╲        ╱  ╲        ╱                            │
│       │      ╱      ╲    ╱      ╲    ╱                              │
│       │    ╱          ╲╱          ╲╱                                │
│     0 └──╱──────────────────────────────▶ Time                      │
│                                                                      │
│  Used for symmetric PWM (motor control)                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Timer Configuration

```c
// Timer configuration for 1 kHz periodic interrupt (STM32)
void timer_init_1khz(void) {
    // Enable TIM2 clock
    RCC->APB1ENR |= RCC_APB1ENR_TIM2EN;
    
    // Configure timer
    // Assuming APB1 clock = 36 MHz
    TIM2->PSC = 36 - 1;      // Prescaler: 36 MHz / 36 = 1 MHz
    TIM2->ARR = 1000 - 1;    // Auto-reload: 1 MHz / 1000 = 1 kHz
    TIM2->CR1 = 0;           // Up counting, no other options
    
    // Enable update interrupt
    TIM2->DIER |= TIM_DIER_UIE;
    
    // Enable interrupt in NVIC
    NVIC_EnableIRQ(TIM2_IRQn);
    NVIC_SetPriority(TIM2_IRQn, 1);
    
    // Start timer
    TIM2->CR1 |= TIM_CR1_CEN;
}

// Timer interrupt handler
volatile uint32_t tick_count = 0;

void TIM2_IRQHandler(void) {
    if (TIM2->SR & TIM_SR_UIF) {
        TIM2->SR &= ~TIM_SR_UIF;  // Clear interrupt flag
        tick_count++;
    }
}

// Delay function using timer
void delay_ms(uint32_t ms) {
    uint32_t start = tick_count;
    while ((tick_count - start) < ms);
}
```

---

## PWM Generation

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PWM FUNDAMENTALS                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PWM = Pulse Width Modulation                                       │
│                                                                      │
│  ┌────┐    ┌────┐    ┌────┐    ┌────┐                              │
│  │    │    │    │    │    │    │    │                              │
│  │    │    │    │    │    │    │    │                              │
│  ┘    └────┘    └────┘    └────┘    └────                          │
│  ◄────────▶                                                         │
│    Period (T)                                                        │
│  ◄──▶                                                               │
│  Pulse Width (ton)                                                  │
│                                                                      │
│  Duty Cycle = ton / T × 100%                                        │
│  Frequency = 1 / T                                                  │
│                                                                      │
│  DUTY CYCLE EXAMPLES:                                                │
│                                                                      │
│  0% ─────────────────────────────────  (always LOW)                │
│                                                                      │
│  25% ┌─┐   ┌─┐   ┌─┐   ┌─┐   ┌─┐                                  │
│      ┘ └───┘ └───┘ └───┘ └───┘ └───                                │
│                                                                      │
│  50% ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐                                 │
│      ┘  └──┘  └──┘  └──┘  └──└──                                   │
│                                                                      │
│  75% ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐                                │
│      ┘   └─┘   └─┘   └─┘   └─┘   └                                 │
│                                                                      │
│  100% ───────────────────────────────  (always HIGH)               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### PWM Timer Configuration

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PWM USING OUTPUT COMPARE                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Counter counts from 0 to ARR (auto-reload register)               │
│  CCR (capture/compare register) sets duty cycle                    │
│                                                                      │
│  CNT                                                                 │
│  ARR ─ ─ ─╱│╲─ ─ ─ ─╱│╲─ ─ ─ ─╱│╲─                                 │
│          ╱ │ ╲      ╱ │ ╲      ╱ │                                  │
│  CCR ─ ─╱──│──╲─ ─╱──│──╲─ ─╱──│──                                 │
│        ╱   │   ╲  ╱   │   ╲  ╱   │                                  │
│       ╱    │    ╲╱    │    ╲╱    │                                  │
│     0 ─────┴─────────┴─────────┴───▶ Time                           │
│                                                                      │
│  PWM (Mode 1):                                                       │
│      CNT < CCR → HIGH                                               │
│      CNT ≥ CCR → LOW                                                │
│                                                                      │
│  Output:                                                             │
│      ┌────┐    ┌────┐    ┌────┐                                    │
│      │    │    │    │    │    │                                    │
│      ┘    └────┘    └────┘    └──                                  │
│                                                                      │
│  Duty Cycle = CCR / (ARR + 1) × 100%                               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### PWM Implementation

```c
// PWM on TIM3 Channel 1 (PA6)
void pwm_init(uint32_t freq_hz) {
    // Enable clocks
    RCC->APB1ENR |= RCC_APB1ENR_TIM3EN;
    RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;
    
    // Configure PA6 as alternate function (TIM3_CH1)
    GPIOA->MODER &= ~(3 << (6 * 2));
    GPIOA->MODER |= (2 << (6 * 2));    // Alternate function
    GPIOA->AFR[0] |= (2 << (6 * 4));   // AF2 = TIM3
    
    // Calculate prescaler and ARR
    // For 1 kHz PWM with 72 MHz clock:
    // 72 MHz / 72 / 1000 = 1 kHz
    uint32_t timer_clock = 72000000;
    uint32_t prescaler = 72;
    uint32_t period = timer_clock / prescaler / freq_hz;
    
    TIM3->PSC = prescaler - 1;
    TIM3->ARR = period - 1;
    
    // Configure channel 1 as PWM mode 1
    TIM3->CCMR1 &= ~TIM_CCMR1_OC1M;
    TIM3->CCMR1 |= (6 << 4);           // PWM mode 1
    TIM3->CCMR1 |= TIM_CCMR1_OC1PE;    // Preload enable
    
    // Enable output
    TIM3->CCER |= TIM_CCER_CC1E;
    
    // Enable timer
    TIM3->CR1 |= TIM_CR1_CEN;
}

// Set duty cycle (0-100%)
void pwm_set_duty(uint8_t duty_percent) {
    if (duty_percent > 100) duty_percent = 100;
    
    uint32_t compare = ((TIM3->ARR + 1) * duty_percent) / 100;
    TIM3->CCR1 = compare;
}

// Set duty cycle (0-255 for 8-bit resolution)
void pwm_set_duty_8bit(uint8_t duty) {
    uint32_t compare = ((TIM3->ARR + 1) * duty) / 256;
    TIM3->CCR1 = compare;
}
```

---

## PWM Applications

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PWM APPLICATIONS                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  LED DIMMING:                                                        │
│                                                                      │
│  PWM ──[R]──▶|── GND                                                │
│                                                                      │
│  • High frequency (>1 kHz) - no visible flicker                    │
│  • Duty cycle controls brightness                                   │
│  • Non-linear perception: use gamma correction                     │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  DC MOTOR SPEED CONTROL:                                            │
│                                                                      │
│        VCC                                                           │
│         │                                                            │
│         │───┬── Motor ──┬───│                                       │
│         │   │           │   │                                       │
│  PWM ──┤├───┘           └───┤├── PWM                               │
│        └┤                   └┤                                      │
│         │                   │                                       │
│        GND                 GND                                       │
│      (H-Bridge simplified)                                          │
│                                                                      │
│  • Motor speed ∝ average voltage ∝ duty cycle                      │
│  • Frequency: 1-20 kHz (audible range consideration)               │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  ANALOG OUTPUT (DAC emulation):                                     │
│                                                                      │
│  PWM ──[R]──┬── Vout                                                │
│             │                                                        │
│            ─┴─ C                                                     │
│            ─┬─                                                       │
│             │                                                        │
│            GND                                                       │
│                                                                      │
│  RC filter smooths PWM to DC                                        │
│  Vout = Duty × VCC                                                  │
│  fc = 1 / (2π × R × C) << PWM frequency                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Servo Motor Control

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SERVO MOTOR PWM                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Standard hobby servo: 50 Hz (20 ms period)                        │
│  Position determined by pulse width:                                │
│                                                                      │
│  ┌─┐                                 0° (fully left)               │
│  │ │ 1 ms                                                           │
│  ┘ └───────────────────────────────                                │
│  ◄──────── 20 ms ─────────▶                                        │
│                                                                      │
│  ┌──┐                                90° (center)                   │
│  │  │ 1.5 ms                                                        │
│  ┘  └──────────────────────────────                                │
│                                                                      │
│  ┌───┐                               180° (fully right)            │
│  │   │ 2 ms                                                         │
│  ┘   └─────────────────────────────                                │
│                                                                      │
│  Pulse Width:                                                        │
│  • 1.0 ms = 0° (or -90°)                                           │
│  • 1.5 ms = 90° (or 0° center)                                     │
│  • 2.0 ms = 180° (or +90°)                                         │
│                                                                      │
│  SERVO WIRING:                                                       │
│                                                                      │
│  ┌─────────────────────────────────────────────┐                    │
│  │ Wire Color  │ Function                      │                    │
│  ├─────────────────────────────────────────────┤                    │
│  │ Brown/Black │ GND                           │                    │
│  │ Red         │ VCC (4.8-6V)                  │                    │
│  │ Orange/White│ Signal (PWM from MCU)         │                    │
│  └─────────────────────────────────────────────┘                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Servo Driver Code

```c
// Servo on TIM2 Channel 1
// PWM frequency: 50 Hz (20 ms period)

void servo_init(void) {
    // 72 MHz / 72 = 1 MHz timer clock
    // 1 MHz / 20000 = 50 Hz
    TIM2->PSC = 72 - 1;
    TIM2->ARR = 20000 - 1;  // 20 ms period
    
    // PWM mode 1
    TIM2->CCMR1 |= (6 << 4);
    TIM2->CCER |= TIM_CCER_CC1E;
    TIM2->CR1 |= TIM_CR1_CEN;
}

// Set servo position (0-180 degrees)
void servo_set_angle(uint8_t angle) {
    if (angle > 180) angle = 180;
    
    // Map 0-180° to 1000-2000 µs
    // At 1 MHz timer clock: 1000-2000 ticks
    uint32_t pulse_us = 1000 + (angle * 1000 / 180);
    TIM2->CCR1 = pulse_us;
}

// Sweep servo from min to max and back
void servo_sweep(void) {
    for (int angle = 0; angle <= 180; angle += 5) {
        servo_set_angle(angle);
        delay_ms(50);
    }
    for (int angle = 180; angle >= 0; angle -= 5) {
        servo_set_angle(angle);
        delay_ms(50);
    }
}
```

---

## Input Capture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    INPUT CAPTURE MODE                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Measures time between input signal edges:                          │
│                                                                      │
│  Input:   ───┐     ┌─────────────┐     ┌──────                     │
│              │     │             │     │                            │
│              └─────┘             └─────┘                            │
│              ↑     ↑             ↑                                  │
│           Capture Capture     Capture                               │
│           CCR=100 CCR=350     CCR=800                               │
│                                                                      │
│  Pulse width = 350 - 100 = 250 ticks                               │
│  Period = 800 - 100 = 700 ticks                                    │
│                                                                      │
│  APPLICATIONS:                                                       │
│  • Frequency measurement                                            │
│  • Pulse width measurement                                          │
│  • Ultrasonic distance sensing                                      │
│  • Encoder reading                                                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Ultrasonic Sensor (HC-SR04)

```c
// HC-SR04: Trigger pulse, measure echo duration
// Distance = (echo_time_us × 343) / 2 / 1000000  [in meters]
// Distance_cm = echo_time_us / 58

#define TRIG_PIN  0  // PA0
#define ECHO_PIN  1  // PA1 (TIM2 CH2)

volatile uint32_t echo_start = 0;
volatile uint32_t echo_end = 0;
volatile bool measurement_done = false;

void ultrasonic_init(void) {
    // Configure TRIG as output
    gpio_mode_output(GPIOA, TRIG_PIN);
    GPIO_CLEAR(GPIOA, TRIG_PIN);
    
    // Configure TIM2 CH2 for input capture (1 µs resolution)
    TIM2->PSC = 72 - 1;  // 1 MHz
    TIM2->ARR = 0xFFFF;
    
    // CH2 input capture on both edges
    TIM2->CCMR1 |= TIM_CCMR1_CC2S_0;  // IC2 mapped to TI2
    TIM2->CCER |= TIM_CCER_CC2E;       // Enable capture
    TIM2->CCER |= TIM_CCER_CC2P | TIM_CCER_CC2NP;  // Both edges
    
    // Enable interrupt
    TIM2->DIER |= TIM_DIER_CC2IE;
    NVIC_EnableIRQ(TIM2_IRQn);
    
    TIM2->CR1 |= TIM_CR1_CEN;
}

void TIM2_IRQHandler(void) {
    if (TIM2->SR & TIM_SR_CC2IF) {
        TIM2->SR &= ~TIM_SR_CC2IF;
        
        if (GPIO_READ(GPIOA, ECHO_PIN)) {
            // Rising edge - echo start
            echo_start = TIM2->CCR2;
        } else {
            // Falling edge - echo end
            echo_end = TIM2->CCR2;
            measurement_done = true;
        }
    }
}

uint32_t ultrasonic_measure_cm(void) {
    measurement_done = false;
    
    // Send 10 µs trigger pulse
    GPIO_SET(GPIOA, TRIG_PIN);
    delay_us(10);
    GPIO_CLEAR(GPIOA, TRIG_PIN);
    
    // Wait for measurement (timeout 30 ms)
    uint32_t timeout = 30000;
    while (!measurement_done && timeout--) {
        delay_us(1);
    }
    
    if (!measurement_done) return 0;  // Timeout
    
    uint32_t echo_time = echo_end - echo_start;
    return echo_time / 58;  // Convert to cm
}
```

---

## Summary Table

| Feature | Description | Application |
|---------|-------------|-------------|
| **Timer Interrupt** | Periodic interrupt at fixed rate | Scheduling, delays |
| **PWM Output** | Variable duty cycle waveform | LED dimming, motor speed |
| **Servo Control** | 50 Hz PWM, 1-2 ms pulses | Position control |
| **Input Capture** | Measure edge timestamps | Frequency, pulse width |
| **Output Compare** | Toggle output at match | Waveform generation |

---

## Quick Revision Questions

1. **A 48 MHz timer with prescaler 48 and ARR 999. Calculate the timer frequency and period.**

2. **Explain the difference between PWM mode 1 and PWM mode 2.**

3. **What PWM frequency is suitable for LED dimming? For motor control? Why?**

4. **A servo needs 1.5 ms pulse at 50 Hz. With 1 MHz timer clock, what should CCR be?**

5. **How does input capture measure frequency? What timer settings are needed?**

6. **Design a system to measure an unknown signal frequency from 10 Hz to 10 kHz.**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Chapter 6.1: GPIO and Digital Interfacing](01-gpio-digital-interfacing.md) | [Unit Index](README.md) | [Chapter 6.3: ADC and DAC](03-adc-dac.md) |

---
