# Chapter 6.4: Display Interfacing

## Introduction

Displays are essential for user interfaces in embedded systems. This chapter covers common display technologies from simple 7-segment LEDs to graphical OLEDs.

---

## 7-Segment Display

```
┌─────────────────────────────────────────────────────────────────────┐
│                    7-SEGMENT DISPLAY                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SEGMENT LAYOUT:                PIN CONNECTIONS:                     │
│                                                                      │
│       ─────                     Common Cathode:                      │
│      │  a  │                    ┌─────────────────────────────┐     │
│       ─────                     │ a  b  c  d  e  f  g  dp CC │     │
│     │ f    │ b                  │ │  │  │  │  │  │  │  │  │  │     │
│       ─────                     └─┴──┴──┴──┴──┴──┴──┴──┴──┴──┘     │
│      │  g  │                     1  2  3  4  5  6  7  8  CC         │
│       ─────                                                          │
│     │ e    │ c                  HIGH on segment = LED ON             │
│       ─────   ●                 CC connected to GND                  │
│      │  d  │  dp                                                     │
│       ─────                     Common Anode:                        │
│                                 HIGH on segment = LED OFF            │
│                                 CA connected to VCC                  │
│                                                                      │
│  DIGIT ENCODING (Common Cathode):                                   │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Digit │  g   f   e   d   c   b   a  │ Hex   │              │     │
│  ├────────────────────────────────────────────────────────────┤     │
│  │   0   │  0   1   1   1   1   1   1  │ 0x3F  │   ─────      │     │
│  │   1   │  0   0   0   0   1   1   0  │ 0x06  │      │ │     │     │
│  │   2   │  1   0   1   1   0   1   1  │ 0x5B  │   ═════      │     │
│  │   3   │  1   0   0   1   1   1   1  │ 0x4F  │      ═══     │     │
│  │   4   │  1   1   0   0   1   1   0  │ 0x66  │   │ ═ │      │     │
│  │   5   │  1   1   0   1   1   0   1  │ 0x6D  │   ═══        │     │
│  │   6   │  1   1   1   1   1   0   1  │ 0x7D  │   ════       │     │
│  │   7   │  0   0   0   0   1   1   1  │ 0x07  │      │ │     │     │
│  │   8   │  1   1   1   1   1   1   1  │ 0x7F  │   ═════      │     │
│  │   9   │  1   1   0   1   1   1   1  │ 0x6F  │   ════ │     │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Multiplexed 4-Digit Display

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MULTIPLEXED DISPLAY                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Share segment lines, switch digits rapidly:                        │
│                                                                      │
│         MCU                    4-Digit Display                      │
│      ┌────────┐        ┌───┬───┬───┬───┐                           │
│      │  a-g   │───────▶│   │   │   │   │ Segments                  │
│      │ (7 pins)        │   │   │   │   │ (shared)                  │
│      │        │        └─┬─┴─┬─┴─┬─┴─┬─┘                           │
│      │  D1-D4 │──────────┘   │   │   │  Common pins                │
│      │ (4 pins)              │   │   │  (individual)               │
│      └────────┘              │   │   │                              │
│                                                                      │
│  TIMING (Persistence of Vision):                                    │
│                                                                      │
│  D1: ▓▓▓░░░░░░░░░░░░░░░░░░▓▓▓  Digit 1 on                          │
│  D2: ░░░▓▓▓░░░░░░░░░░░░░░░░░░  Digit 2 on                          │
│  D3: ░░░░░░▓▓▓░░░░░░░░░░░░░░░  Digit 3 on                          │
│  D4: ░░░░░░░░░▓▓▓░░░░░░░░░░░░  Digit 4 on                          │
│      ◄───────────────────────▶                                      │
│         One complete refresh (~20 ms)                               │
│         Each digit: ~5 ms                                           │
│                                                                      │
│  Total pins needed: 7 segments + 4 digits = 11                     │
│  Without multiplexing: 7 × 4 = 28 pins!                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

```c
// 7-segment lookup table (common cathode)
const uint8_t seg_digits[10] = {
    0x3F, 0x06, 0x5B, 0x4F, 0x66,  // 0-4
    0x6D, 0x7D, 0x07, 0x7F, 0x6F   // 5-9
};

// 4-digit display buffer
volatile uint8_t display_buffer[4] = {0, 0, 0, 0};
volatile uint8_t current_digit = 0;

// Set segments (PA0-PA6)
void set_segments(uint8_t pattern) {
    GPIOA->ODR = (GPIOA->ODR & 0xFF80) | pattern;
}

// Enable digit (PB0-PB3, active LOW for common cathode)
void enable_digit(uint8_t digit) {
    GPIOB->ODR |= 0x0F;           // All digits off
    GPIOB->ODR &= ~(1 << digit);  // Selected digit on
}

// Timer interrupt for multiplexing
void TIM3_IRQHandler(void) {
    if (TIM3->SR & TIM_SR_UIF) {
        TIM3->SR &= ~TIM_SR_UIF;
        
        enable_digit(current_digit);
        set_segments(display_buffer[current_digit]);
        
        current_digit = (current_digit + 1) & 0x03;
    }
}

// Display a 4-digit number
void display_number(uint16_t number) {
    display_buffer[0] = seg_digits[number / 1000 % 10];
    display_buffer[1] = seg_digits[number / 100 % 10];
    display_buffer[2] = seg_digits[number / 10 % 10];
    display_buffer[3] = seg_digits[number % 10];
}
```

---

## Character LCD (HD44780)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    HD44780 LCD INTERFACE                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PIN CONNECTIONS (16-pin):                                          │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────┐       │
│  │ Pin │ Name │ Function                                    │       │
│  ├──────────────────────────────────────────────────────────┤       │
│  │  1  │ VSS  │ Ground                                      │       │
│  │  2  │ VDD  │ +5V Power                                   │       │
│  │  3  │ VO   │ Contrast adjust (via potentiometer)         │       │
│  │  4  │ RS   │ Register Select (0=cmd, 1=data)             │       │
│  │  5  │ RW   │ Read/Write (0=write, 1=read) - often GND    │       │
│  │  6  │ E    │ Enable (data latched on falling edge)       │       │
│  │ 7-14│D0-D7 │ Data bus (4-bit mode uses D4-D7 only)       │       │
│  │ 15  │ A    │ Backlight anode (+5V)                       │       │
│  │ 16  │ K    │ Backlight cathode (GND)                     │       │
│  └──────────────────────────────────────────────────────────┘       │
│                                                                      │
│  4-BIT MODE (saves GPIO pins):                                      │
│                                                                      │
│     MCU                         LCD                                  │
│  ┌────────┐                  ┌───────────────────────────────┐      │
│  │   RS   │─────────────────▶│ RS                            │      │
│  │   E    │─────────────────▶│ E                             │      │
│  │   D4   │─────────────────▶│ D4    ┌─────────────────────┐ │      │
│  │   D5   │─────────────────▶│ D5    │ Hello World!        │ │      │
│  │   D6   │─────────────────▶│ D6    │ Line 2 Text         │ │      │
│  │   D7   │─────────────────▶│ D7    └─────────────────────┘ │      │
│  └────────┘                  └───────────────────────────────┘      │
│                                                                      │
│  6 GPIO pins needed (vs 11 for 8-bit mode)                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### HD44780 Timing

```
┌─────────────────────────────────────────────────────────────────────┐
│                    WRITE TIMING (4-BIT)                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  RS:    ═══════════════════════════════════════════════════         │
│         (0 for command, 1 for data - stable during E pulse)        │
│                                                                      │
│  E:     ────┐    ┌────────────────────────┐    ┌────                │
│             └────┘                        └────┘                    │
│              │ High nibble │              │ Low nibble │            │
│                                                                      │
│  D7-D4: ════╪═══════════════════════════════╪═══════════            │
│              High 4 bits                    Low 4 bits              │
│                                                                      │
│  Data latched on E falling edge                                     │
│  E pulse width: >450 ns                                             │
│  Data setup before E low: >80 ns                                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

```c
// LCD pins (example: all on GPIOB)
#define LCD_RS_PIN  0
#define LCD_E_PIN   1
#define LCD_D4_PIN  4
#define LCD_D5_PIN  5
#define LCD_D6_PIN  6
#define LCD_D7_PIN  7

#define LCD_RS_HIGH()  GPIO_SET(GPIOB, LCD_RS_PIN)
#define LCD_RS_LOW()   GPIO_CLEAR(GPIOB, LCD_RS_PIN)
#define LCD_E_HIGH()   GPIO_SET(GPIOB, LCD_E_PIN)
#define LCD_E_LOW()    GPIO_CLEAR(GPIOB, LCD_E_PIN)

// Send 4 bits
void lcd_send_nibble(uint8_t nibble) {
    GPIOB->ODR = (GPIOB->ODR & 0xFF0F) | ((nibble & 0x0F) << 4);
    
    LCD_E_HIGH();
    delay_us(1);
    LCD_E_LOW();
    delay_us(1);
}

// Send byte (as two nibbles)
void lcd_send_byte(uint8_t data, bool is_data) {
    if (is_data) LCD_RS_HIGH(); else LCD_RS_LOW();
    
    lcd_send_nibble(data >> 4);   // High nibble first
    lcd_send_nibble(data & 0x0F); // Low nibble
    
    delay_us(50);  // Most commands take 37 µs
}

// LCD commands
#define LCD_CMD_CLEAR       0x01
#define LCD_CMD_HOME        0x02
#define LCD_CMD_ENTRY_MODE  0x06
#define LCD_CMD_DISPLAY_ON  0x0C
#define LCD_CMD_FUNCTION    0x28  // 4-bit, 2 lines, 5x8 font
#define LCD_CMD_SET_DDRAM   0x80

void lcd_init(void) {
    delay_ms(50);  // Wait for LCD power-up
    
    // Initialize in 4-bit mode
    LCD_RS_LOW();
    lcd_send_nibble(0x03);  delay_ms(5);
    lcd_send_nibble(0x03);  delay_us(150);
    lcd_send_nibble(0x03);  delay_us(150);
    lcd_send_nibble(0x02);  delay_us(150);  // Switch to 4-bit
    
    // Configure display
    lcd_send_byte(LCD_CMD_FUNCTION, false);     // 4-bit, 2 lines
    lcd_send_byte(LCD_CMD_DISPLAY_ON, false);   // Display on
    lcd_send_byte(LCD_CMD_CLEAR, false);        // Clear display
    delay_ms(2);
    lcd_send_byte(LCD_CMD_ENTRY_MODE, false);   // Increment cursor
}

void lcd_clear(void) {
    lcd_send_byte(LCD_CMD_CLEAR, false);
    delay_ms(2);
}

void lcd_set_cursor(uint8_t row, uint8_t col) {
    uint8_t address = (row == 0) ? col : (0x40 + col);
    lcd_send_byte(LCD_CMD_SET_DDRAM | address, false);
}

void lcd_print(const char *str) {
    while (*str) {
        lcd_send_byte(*str++, true);
    }
}

// Usage
lcd_init();
lcd_set_cursor(0, 0);
lcd_print("Hello World!");
lcd_set_cursor(1, 0);
lcd_print("Temp: 25.5 C");
```

---

## OLED Display (SSD1306)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SSD1306 OLED (128x64)                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  INTERFACE OPTIONS:                                                  │
│  • I2C (2 wires) - most common for hobby use                       │
│  • SPI (4 wires) - faster                                          │
│                                                                      │
│  I2C PINOUT:                                                        │
│  ┌─────────────────────────────────────────────────────────┐        │
│  │ OLED Pin │ Connection                                   │        │
│  ├─────────────────────────────────────────────────────────┤        │
│  │ VCC      │ 3.3V or 5V                                   │        │
│  │ GND      │ Ground                                       │        │
│  │ SDA      │ I2C Data (with pull-up)                      │        │
│  │ SCL      │ I2C Clock (with pull-up)                     │        │
│  └─────────────────────────────────────────────────────────┘        │
│                                                                      │
│  I2C Address: 0x3C or 0x3D                                         │
│                                                                      │
│  MEMORY ORGANIZATION:                                                │
│                                                                      │
│  128 pixels wide × 64 pixels high = 8192 pixels                    │
│  Organized as 8 pages × 128 columns                                │
│                                                                      │
│  ┌──────────────────────────────────────────────┐                   │
│  │ Page 0  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│ 8 rows            │
│  │ Page 1  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│                   │
│  │ Page 2  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│                   │
│  │ Page 3  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│                   │
│  │ Page 4  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│                   │
│  │ Page 5  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│                   │
│  │ Page 6  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│                   │
│  │ Page 7  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│ 8 rows            │
│  └──────────────────────────────────────────────┘                   │
│            Column 0 ─────────────────▶ Column 127                   │
│                                                                      │
│  Each byte = 8 vertical pixels (1 column in 1 page)                │
│  Total buffer size = 128 × 8 = 1024 bytes                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### SSD1306 Driver

```c
#define SSD1306_ADDR  0x3C

// Display buffer (128 × 64 / 8 = 1024 bytes)
uint8_t display_buffer[1024];

// Send command
void ssd1306_command(uint8_t cmd) {
    uint8_t data[2] = {0x00, cmd};  // 0x00 = command
    i2c_write(SSD1306_ADDR, data, 2);
}

// Initialize display
void ssd1306_init(void) {
    delay_ms(100);  // Wait for power-up
    
    ssd1306_command(0xAE);  // Display off
    ssd1306_command(0xD5);  // Clock divide
    ssd1306_command(0x80);
    ssd1306_command(0xA8);  // Multiplex ratio
    ssd1306_command(0x3F);  // 1/64
    ssd1306_command(0xD3);  // Display offset
    ssd1306_command(0x00);
    ssd1306_command(0x40);  // Start line = 0
    ssd1306_command(0x8D);  // Charge pump
    ssd1306_command(0x14);  // Enable
    ssd1306_command(0x20);  // Memory mode
    ssd1306_command(0x00);  // Horizontal
    ssd1306_command(0xA1);  // Segment remap
    ssd1306_command(0xC8);  // COM scan direction
    ssd1306_command(0xDA);  // COM pins config
    ssd1306_command(0x12);
    ssd1306_command(0x81);  // Contrast
    ssd1306_command(0xCF);
    ssd1306_command(0xD9);  // Pre-charge
    ssd1306_command(0xF1);
    ssd1306_command(0xDB);  // VCOMH deselect
    ssd1306_command(0x40);
    ssd1306_command(0xA4);  // Output follows RAM
    ssd1306_command(0xA6);  // Normal display
    ssd1306_command(0xAF);  // Display on
    
    ssd1306_clear();
}

// Send buffer to display
void ssd1306_update(void) {
    ssd1306_command(0x21);  // Column range
    ssd1306_command(0);
    ssd1306_command(127);
    ssd1306_command(0x22);  // Page range
    ssd1306_command(0);
    ssd1306_command(7);
    
    // Send data
    for (int i = 0; i < 1024; i += 16) {
        uint8_t packet[17];
        packet[0] = 0x40;  // Data
        memcpy(&packet[1], &display_buffer[i], 16);
        i2c_write(SSD1306_ADDR, packet, 17);
    }
}

// Clear buffer
void ssd1306_clear(void) {
    memset(display_buffer, 0, 1024);
}

// Set pixel
void ssd1306_set_pixel(uint8_t x, uint8_t y, bool color) {
    if (x >= 128 || y >= 64) return;
    
    uint16_t index = x + (y / 8) * 128;
    uint8_t bit = y % 8;
    
    if (color) {
        display_buffer[index] |= (1 << bit);
    } else {
        display_buffer[index] &= ~(1 << bit);
    }
}

// Draw character (5x7 font)
extern const uint8_t font5x7[][5];  // Font data

void ssd1306_draw_char(uint8_t x, uint8_t y, char c) {
    if (c < 32 || c > 126) c = '?';
    c -= 32;
    
    for (int col = 0; col < 5; col++) {
        uint8_t line = font5x7[c][col];
        for (int row = 0; row < 7; row++) {
            ssd1306_set_pixel(x + col, y + row, line & (1 << row));
        }
    }
}

void ssd1306_print(uint8_t x, uint8_t y, const char *str) {
    while (*str) {
        ssd1306_draw_char(x, y, *str++);
        x += 6;  // 5 pixels + 1 space
    }
}
```

---

## Display Comparison

| Display Type | Resolution | Interface | Color | Power | Cost |
|--------------|------------|-----------|-------|-------|------|
| **7-Segment** | 1-8 digits | GPIO | Mono | Low | $ |
| **HD44780 LCD** | 16x2, 20x4 | GPIO/I2C | Mono | Medium | $ |
| **Graphic LCD** | 128x64 | SPI/I2C | Mono | Medium | $$ |
| **OLED SSD1306** | 128x64 | I2C/SPI | Mono | Low | $$ |
| **TFT LCD** | 320x240+ | SPI/Parallel | Color | High | $$$ |

---

## Summary Table

| Feature | 7-Segment | Character LCD | OLED |
|---------|-----------|---------------|------|
| **Pins** | 7-11 | 6-11 | 2-4 |
| **Content** | Digits only | Text + custom | Graphics |
| **Viewing Angle** | Wide | Limited | Wide |
| **Backlight** | No (LED) | Yes (optional) | Self-lit |
| **Power** | ~10mA/digit | ~1mA | ~20-40mA |

---

## Quick Revision Questions

1. **What is multiplexing in 7-segment displays? Calculate refresh rate for flicker-free 4-digit display.**

2. **Explain the difference between RS=0 and RS=1 on HD44780 LCD.**

3. **Why is 4-bit mode preferred for HD44780? What is the trade-off?**

4. **The SSD1306 organizes memory as pages. How many bytes represent one vertical column?**

5. **Design a display system for temperature (XX.X°C). Which display type would you choose and why?**

6. **What is the I2C address commonly used for SSD1306 OLEDs?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Chapter 6.3: ADC and DAC](03-adc-dac.md) | [Unit Index](README.md) | [Chapter 6.5: Motor Control](05-motor-control.md) |

---
