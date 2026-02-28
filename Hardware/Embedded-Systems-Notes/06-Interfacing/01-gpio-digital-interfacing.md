# Chapter 6.1: GPIO and Digital Interfacing

## Introduction

GPIO (General Purpose Input/Output) pins are the fundamental building blocks for digital interfacing. This chapter covers GPIO configuration, LED and switch interfacing, debouncing techniques, and matrix scanning.

---

## GPIO Modes and Configuration

```
┌─────────────────────────────────────────────────────────────────────┐
│                    GPIO PIN MODES                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  INPUT MODES:                                                        │
│                                                                      │
│  Floating (Hi-Z):           Pull-Up:               Pull-Down:       │
│                                                                      │
│  VCC         VCC            VCC                    VCC               │
│   │           │              │                      │                │
│   ×          ─┴─             │                      ×                │
│              │Rp│ weak       │                                       │
│              ─┬─ (30-50kΩ)   │                     ─┴─               │
│   │           │              │                     │Rp│              │
│   │      ─────┼──▶ GPIO ─────┼──▶ GPIO        ─────┼──▶ GPIO        │
│   ×           │              │                     ─┬─               │
│               │             ─┴─                     │                │
│  GND         GND            │Rp│                   GND               │
│                             ─┬─                                      │
│  Undefined                   │                                       │
│  level (noise)              GND                                      │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  OUTPUT MODES:                                                       │
│                                                                      │
│  Push-Pull:                 Open-Drain:                             │
│                                                                      │
│  VCC                        VCC                                      │
│   │                          │                                       │
│  ─┴─ PMOS                   ─┴─ External                            │
│  ─┼─                        │Rp│ Pull-up                            │
│   ├──▶ Output               ─┬─                                      │
│  ─┼─                         │                                       │
│  ─┬─ NMOS              ─────┼──▶ Output                             │
│   │                    │    │                                        │
│  GND                   │   ─┼─ NMOS                                  │
│                        │   ─┬─                                       │
│  Can drive HIGH or LOW │    │                                        │
│                       GND  GND                                       │
│                                                                      │
│                        Can only pull LOW                             │
│                        (needs external pull-up)                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### GPIO Configuration Code

```c
// GPIO configuration structure (conceptual)
typedef struct {
    uint32_t pin;
    uint32_t mode;       // INPUT, OUTPUT, AF, ANALOG
    uint32_t pull;       // NONE, PULLUP, PULLDOWN
    uint32_t speed;      // LOW, MEDIUM, HIGH, VERY_HIGH
    uint32_t otype;      // PUSH_PULL, OPEN_DRAIN
} GPIO_Config_t;

// STM32 GPIO initialization example
void gpio_init(void) {
    // Enable GPIOA clock
    RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;
    
    // PA5 as push-pull output (LED)
    GPIOA->MODER &= ~(3 << (5 * 2));   // Clear mode bits
    GPIOA->MODER |= (1 << (5 * 2));    // Set as output
    GPIOA->OTYPER &= ~(1 << 5);        // Push-pull
    GPIOA->OSPEEDR |= (2 << (5 * 2));  // High speed
    
    // PA0 as input with pull-up (button)
    GPIOA->MODER &= ~(3 << (0 * 2));   // Input mode
    GPIOA->PUPDR &= ~(3 << (0 * 2));   // Clear
    GPIOA->PUPDR |= (1 << (0 * 2));    // Pull-up
}

// Basic GPIO operations
#define GPIO_SET(port, pin)     ((port)->BSRR = (1 << (pin)))
#define GPIO_CLEAR(port, pin)   ((port)->BSRR = (1 << ((pin) + 16)))
#define GPIO_TOGGLE(port, pin)  ((port)->ODR ^= (1 << (pin)))
#define GPIO_READ(port, pin)    (((port)->IDR >> (pin)) & 1)
```

---

## LED Interfacing

```
┌─────────────────────────────────────────────────────────────────────┐
│                    LED INTERFACING                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  LED CURRENT LIMITING:                                              │
│                                                                      │
│  LED forward voltage: Vf ≈ 2V (red), 3.2V (blue/white)             │
│  LED current: If ≈ 5-20 mA (typical 10 mA)                         │
│                                                                      │
│  Resistor calculation:                                              │
│                                                                      │
│        Vcc - Vf                                                      │
│  R = ─────────────                                                   │
│          If                                                          │
│                                                                      │
│  Example: Vcc = 3.3V, Vf = 2V, If = 10mA                           │
│  R = (3.3 - 2) / 0.01 = 130Ω → use 150Ω or 220Ω                    │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  SINK vs SOURCE:                                                     │
│                                                                      │
│  Source (GPIO provides current):    Sink (GPIO absorbs current):   │
│                                                                      │
│  GPIO ──[R]──▶|──GND                VCC──▶|──[R]── GPIO            │
│                                                                      │
│  GPIO HIGH = LED ON                 GPIO LOW = LED ON               │
│                                                                      │
│  • MCUs often sink more current than source                        │
│  • Sink configuration preferred for multiple LEDs                  │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  RGB LED (Common Cathode):          RGB LED (Common Anode):        │
│                                                                      │
│      R   G   B                          R   G   B                   │
│      │   │   │                          │   │   │                   │
│     [R] [R] [R]                        [R] [R] [R]                  │
│      │   │   │                          │   │   │                   │
│      ▼   ▼   ▼                          │   │   │                   │
│      └───┴───┘                      VCC─┴───┴───┘                   │
│          │                                                          │
│         GND                         GPIO LOW = color ON             │
│                                                                      │
│  GPIO HIGH = color ON                                               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### LED Driver Code

```c
// LED pin definitions
#define LED_PORT      GPIOA
#define LED_RED       5
#define LED_GREEN     6
#define LED_BLUE      7

// Simple LED control
void led_on(uint8_t pin) {
    GPIO_SET(LED_PORT, pin);
}

void led_off(uint8_t pin) {
    GPIO_CLEAR(LED_PORT, pin);
}

void led_toggle(uint8_t pin) {
    GPIO_TOGGLE(LED_PORT, pin);
}

// LED pattern (blinking)
void led_blink(uint8_t pin, uint32_t on_ms, uint32_t off_ms) {
    led_on(pin);
    delay_ms(on_ms);
    led_off(pin);
    delay_ms(off_ms);
}

// RGB LED color mixing (0-255 each, using PWM for brightness)
typedef struct {
    uint8_t r, g, b;
} RGB_Color_t;

const RGB_Color_t COLOR_RED    = {255, 0, 0};
const RGB_Color_t COLOR_GREEN  = {0, 255, 0};
const RGB_Color_t COLOR_BLUE   = {0, 0, 255};
const RGB_Color_t COLOR_YELLOW = {255, 255, 0};
const RGB_Color_t COLOR_WHITE  = {255, 255, 255};

void rgb_set_color(RGB_Color_t color) {
    // Assuming PWM channels are configured
    pwm_set_duty(PWM_CH_RED, color.r);
    pwm_set_duty(PWM_CH_GREEN, color.g);
    pwm_set_duty(PWM_CH_BLUE, color.b);
}
```

---

## Switch/Button Interfacing

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SWITCH INTERFACING                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SWITCH TYPES:                                                       │
│                                                                      │
│  Momentary (Push Button):    Toggle/Slide:        Rotary:           │
│  ┌───┐                       ┌─────────┐          ┌───┐             │
│  │ ○ │ ← Press               │ ○───○   │          │ A │────         │
│  └───┘   releases            │    └─●  │          │ B │────         │
│                              └─────────┘          │ C │────         │
│  Normally Open (NO)          Maintains state      └───┘             │
│                                                   Encoder           │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  BOUNCE PROBLEM:                                                     │
│                                                                      │
│  Mechanical contacts don't make clean transitions:                  │
│                                                                      │
│  Voltage                                                             │
│  VCC ─────┐   ┌─┐ ┌─┐                                               │
│           │   │ │ │ │  Bouncing                                     │
│           └───┘ └─┘ └─────────────────  Final stable LOW           │
│  GND                                                                 │
│           ◄──────────────────▶                                      │
│            5-20 ms bounce time                                       │
│                                                                      │
│  MCU may see multiple transitions during one press!                │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  DEBOUNCING SOLUTIONS:                                               │
│                                                                      │
│  1. Hardware (RC filter):                                           │
│                                                                      │
│     VCC                                                              │
│      │                                                               │
│     ─┴─ 10kΩ                                                        │
│     ─┬─                                                              │
│      │                                                               │
│      ├───[100Ω]───┬── GPIO                                          │
│      │            │                                                  │
│     ─┴─ SW       ─┴─ 100nF                                          │
│      │           ─┬─                                                 │
│     GND          GND                                                 │
│                                                                      │
│     RC time constant smooths transitions                            │
│                                                                      │
│  2. Software (delay-based or state machine)                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Debouncing Implementation

```c
// Simple delay-based debounce
#define DEBOUNCE_DELAY_MS  20

bool button_read_debounced(uint8_t pin) {
    if (GPIO_READ(GPIOA, pin) == 0) {  // Button pressed (active low)
        delay_ms(DEBOUNCE_DELAY_MS);
        if (GPIO_READ(GPIOA, pin) == 0) {
            return true;  // Still pressed after delay
        }
    }
    return false;
}

// State machine debouncer (non-blocking)
typedef enum {
    BTN_RELEASED,
    BTN_PRESSING,
    BTN_PRESSED,
    BTN_RELEASING
} ButtonState_t;

typedef struct {
    GPIO_TypeDef *port;
    uint8_t pin;
    ButtonState_t state;
    uint32_t timestamp;
    bool pressed;
    bool event_press;   // Single-shot press event
    bool event_release; // Single-shot release event
} Button_t;

#define DEBOUNCE_TIME_MS  20

void button_init(Button_t *btn, GPIO_TypeDef *port, uint8_t pin) {
    btn->port = port;
    btn->pin = pin;
    btn->state = BTN_RELEASED;
    btn->pressed = false;
    btn->event_press = false;
    btn->event_release = false;
}

void button_update(Button_t *btn) {
    bool raw = (GPIO_READ(btn->port, btn->pin) == 0);  // Active low
    uint32_t now = millis();
    
    btn->event_press = false;
    btn->event_release = false;
    
    switch (btn->state) {
        case BTN_RELEASED:
            if (raw) {
                btn->state = BTN_PRESSING;
                btn->timestamp = now;
            }
            break;
            
        case BTN_PRESSING:
            if (!raw) {
                btn->state = BTN_RELEASED;
            } else if (now - btn->timestamp >= DEBOUNCE_TIME_MS) {
                btn->state = BTN_PRESSED;
                btn->pressed = true;
                btn->event_press = true;  // Single event
            }
            break;
            
        case BTN_PRESSED:
            if (!raw) {
                btn->state = BTN_RELEASING;
                btn->timestamp = now;
            }
            break;
            
        case BTN_RELEASING:
            if (raw) {
                btn->state = BTN_PRESSED;
            } else if (now - btn->timestamp >= DEBOUNCE_TIME_MS) {
                btn->state = BTN_RELEASED;
                btn->pressed = false;
                btn->event_release = true;  // Single event
            }
            break;
    }
}

// Usage
Button_t button1;
button_init(&button1, GPIOA, 0);

while (1) {
    button_update(&button1);
    
    if (button1.event_press) {
        led_on(LED_PIN);
    }
    if (button1.event_release) {
        led_off(LED_PIN);
    }
}
```

---

## Matrix Keypad

```
┌─────────────────────────────────────────────────────────────────────┐
│                    4x4 MATRIX KEYPAD                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PHYSICAL LAYOUT:           WIRING:                                  │
│                                                                      │
│  ┌───┬───┬───┬───┐          C1  C2  C3  C4                          │
│  │ 1 │ 2 │ 3 │ A │          │   │   │   │                           │
│  ├───┼───┼───┼───┤     R1 ──┼───┼───┼───┼── (scan row)             │
│  │ 4 │ 5 │ 6 │ B │          │   │   │   │                           │
│  ├───┼───┼───┼───┤     R2 ──┼───┼───┼───┼──                         │
│  │ 7 │ 8 │ 9 │ C │          │   │   │   │                           │
│  ├───┼───┼───┼───┤     R3 ──┼───┼───┼───┼──                         │
│  │ * │ 0 │ # │ D │          │   │   │   │                           │
│  └───┴───┴───┴───┘     R4 ──┼───┼───┼───┼──                         │
│                              │   │   │   │                           │
│                              ▼   ▼   ▼   ▼                           │
│                          (read columns with pull-ups)                │
│                                                                      │
│  SCANNING METHOD:                                                    │
│                                                                      │
│  1. Set all rows HIGH, columns as inputs with pull-ups             │
│  2. Set R1 LOW, keep others HIGH                                   │
│  3. Read all columns - LOW column = key in that row                │
│  4. Repeat for R2, R3, R4                                          │
│                                                                      │
│  TIMING DIAGRAM:                                                     │
│                                                                      │
│  R1: ─┐___________________________________┌────                     │
│  R2: ─────┐_______________________________┌────                     │
│  R3: ─────────┐___________________________┌────                     │
│  R4: ─────────────┐_______________________┌────                     │
│                    ◄──────────────────────▶                         │
│                      One complete scan                              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Keypad Driver

```c
#define KEYPAD_ROWS    4
#define KEYPAD_COLS    4

// GPIO pins for rows (outputs) and columns (inputs)
const uint8_t row_pins[KEYPAD_ROWS] = {0, 1, 2, 3};  // PA0-PA3
const uint8_t col_pins[KEYPAD_COLS] = {4, 5, 6, 7};  // PA4-PA7

const char keymap[KEYPAD_ROWS][KEYPAD_COLS] = {
    {'1', '2', '3', 'A'},
    {'4', '5', '6', 'B'},
    {'7', '8', '9', 'C'},
    {'*', '0', '#', 'D'}
};

void keypad_init(void) {
    // Rows as outputs (initially HIGH)
    for (int i = 0; i < KEYPAD_ROWS; i++) {
        // Set as output, push-pull
        gpio_mode_output(GPIOA, row_pins[i]);
        GPIO_SET(GPIOA, row_pins[i]);
    }
    
    // Columns as inputs with pull-up
    for (int i = 0; i < KEYPAD_COLS; i++) {
        gpio_mode_input_pullup(GPIOA, col_pins[i]);
    }
}

char keypad_scan(void) {
    for (int row = 0; row < KEYPAD_ROWS; row++) {
        // Set current row LOW, others HIGH
        for (int r = 0; r < KEYPAD_ROWS; r++) {
            if (r == row) {
                GPIO_CLEAR(GPIOA, row_pins[r]);
            } else {
                GPIO_SET(GPIOA, row_pins[r]);
            }
        }
        
        delay_us(10);  // Settling time
        
        // Check each column
        for (int col = 0; col < KEYPAD_COLS; col++) {
            if (GPIO_READ(GPIOA, col_pins[col]) == 0) {
                // Key found - wait for release
                while (GPIO_READ(GPIOA, col_pins[col]) == 0);
                delay_ms(20);  // Debounce
                return keymap[row][col];
            }
        }
    }
    
    return '\0';  // No key pressed
}
```

---

## LED Matrix

```
┌─────────────────────────────────────────────────────────────────────┐
│                    8x8 LED MATRIX                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  COMMON ANODE (rows = anodes):                                      │
│                                                                      │
│     C1  C2  C3  C4  C5  C6  C7  C8  (Cathodes - active LOW)        │
│      │   │   │   │   │   │   │   │                                  │
│  R1──╋───╋───╋───╋───╋───╋───╋───╋  (Anode - active HIGH)          │
│      │   │   │   │   │   │   │   │                                  │
│  R2──╋───╋───╋───╋───╋───╋───╋───╋                                  │
│      │   │   │   │   │   │   │   │                                  │
│     ... (8 rows total)                                              │
│                                                                      │
│  MULTIPLEXING (persistence of vision):                              │
│                                                                      │
│  Time ─▶                                                            │
│                                                                      │
│  R1: ▓▓▓░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  (pattern for row 1)       │
│  R2: ░░░▓▓▓░░░░░░░░░░░░░░░░░░░░░░░░░░░  (pattern for row 2)       │
│  R3: ░░░░░░▓▓▓░░░░░░░░░░░░░░░░░░░░░░░░                             │
│  R4: ░░░░░░░░░▓▓▓░░░░░░░░░░░░░░░░░░░░░                             │
│  R5: ░░░░░░░░░░░░▓▓▓░░░░░░░░░░░░░░░░░░                             │
│  R6: ░░░░░░░░░░░░░░░▓▓▓░░░░░░░░░░░░░░░                             │
│  R7: ░░░░░░░░░░░░░░░░░░▓▓▓░░░░░░░░░░░░                             │
│  R8: ░░░░░░░░░░░░░░░░░░░░░▓▓▓░░░░░░░░░  → Repeat                   │
│                                                                      │
│      ◄────────────────────────────────▶                             │
│        One frame (< 20ms for no flicker)                            │
│        Each row: ~2ms                                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Relay and Optocoupler

```
┌─────────────────────────────────────────────────────────────────────┐
│                    RELAY INTERFACING                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  RELAY DRIVER CIRCUIT:                                              │
│                                                                      │
│  VCC (5V or 12V)                                                    │
│   │                                                                  │
│   │    ┌───────────────┐                                            │
│   └────┤ RELAY COIL    │                                            │
│        │   (~100mA)    │                                            │
│        └───────┬───────┘                                            │
│           ─┬─  │                                                     │
│      Flyback│  │  D1 (1N4007)                                       │
│       Diode ▼  │                                                     │
│           ─┴─  │                                                     │
│                │                                                     │
│  GPIO ─[1kΩ]─┬─┴──|/                                                │
│              │     \│ NPN (2N2222)                                  │
│             GND     │                                                │
│                    GND                                               │
│                                                                      │
│  Flyback diode protects transistor from back-EMF when coil turns off│
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  OPTOCOUPLER (ELECTRICAL ISOLATION):                                │
│                                                                      │
│  MCU Side                 │      │            Load Side              │
│  (3.3V logic)             │ISOLA│            (120V AC)              │
│                           │TION │                                   │
│            ┌──────────────┼──────┼──────────────┐                   │
│  GPIO ─[R]─┤ LED    ▶◀    │      │   PHOTO-    ├── TRIAC ── Load   │
│            │        ◀▶    │      │   TRIAC     │                    │
│            └──────────────┼──────┼──────────────┘                   │
│  GND ──────────────────────      ────────────────── AC Neutral      │
│                                                                      │
│  No electrical connection - isolated by light                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Component | Interface | Considerations |
|-----------|-----------|----------------|
| **LED** | GPIO output | Current limiting resistor |
| **Button** | GPIO input + pull | Debouncing required |
| **Keypad** | Matrix scan | Row outputs, column inputs |
| **LED Matrix** | Multiplexed | Row drivers, refresh rate |
| **Relay** | Transistor driver | Flyback diode for coil |
| **Optocoupler** | LED + detector | Isolation, current limits |

---

## Quick Revision Questions

1. **Explain push-pull vs open-drain output modes. When would you use each?**

2. **Calculate the resistor value for a red LED (Vf=2V) at 15mA from a 5V GPIO.**

3. **Why do mechanical switches need debouncing? Describe a software debouncing algorithm.**

4. **A 4x4 keypad uses 8 GPIO pins. How would the connections change for a 5x5 keypad?**

5. **What is the purpose of a flyback diode in relay circuits? What happens without it?**

6. **Design a circuit to control a 12V relay from a 3.3V MCU output.**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Unit Index](README.md) | [Unit Index](README.md) | [Chapter 6.2: Timers and PWM](02-timers-pwm.md) |

---
