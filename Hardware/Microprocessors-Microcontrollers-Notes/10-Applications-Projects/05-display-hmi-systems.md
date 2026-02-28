# Chapter 10.5: Display and HMI Systems

## 📚 Chapter Overview

This chapter covers display interfaces and Human-Machine Interface (HMI) systems including character LCDs, graphics displays, OLED modules, and touchscreen integration.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Interface character and graphics LCDs
- Design menu systems for embedded devices
- Interface OLED displays using I2C/SPI
- Implement touch input systems
- Create custom graphics and fonts

---

## 1. Character LCD (16×2 HD44780)

### 1.1 HD44780 LCD Basics

```
16×2 CHARACTER LCD INTERFACE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

HD44780 PIN CONFIGURATION:
────────────────────────

Pin │ Symbol │ Function
────┼────────┼───────────────────────────
 1  │  VSS   │ Ground
 2  │  VCC   │ +5V Power supply
 3  │  VEE   │ Contrast adjustment
 4  │  RS    │ Register Select (0=Cmd, 1=Data)
 5  │  RW    │ Read/Write (0=Write, 1=Read)
 6  │  E     │ Enable (falling edge trigger)
 7  │  D0    │ Data bit 0 (not used in 4-bit)
 8  │  D1    │ Data bit 1 (not used in 4-bit)
 9  │  D2    │ Data bit 2 (not used in 4-bit)
10  │  D3    │ Data bit 3 (not used in 4-bit)
11  │  D4    │ Data bit 4
12  │  D5    │ Data bit 5
13  │  D6    │ Data bit 6
14  │  D7    │ Data bit 7
15  │  LED+  │ Backlight anode (+5V)
16  │  LED-  │ Backlight cathode (GND)

4-BIT MODE CONNECTION:
────────────────────

         +5V     +5V
          │       │
          │    ┌──┴──┐
         ┌┴┐   │10kΩ │ Contrast
         │R│   │ POT │ Control
         │ │   └──┬──┘
         └┬┘      │
          │       │
    ┌─────┴───────┴───────────────────────────────────────┐
    │ VSS VCC VEE RS  RW  E   D4  D5  D6  D7  LED+ LED-   │
    │  1   2   3   4   5   6   11  12  13  14   15   16   │
    │                                                      │
    │                  16×2 LCD                           │
    │                                                      │
    │  ┌──────────────────────────────────────────────┐   │
    │  │ █ █ █ █ █ █ █ █ █ █ █ █ █ █ █ █             │   │
    │  │ █ █ █ █ █ █ █ █ █ █ █ █ █ █ █ █             │   │
    │  └──────────────────────────────────────────────┘   │
    └─────────────────────────────────────────────────────┘
          │       │   │   │   │   │   │   │
          │       │   │   │   │   │   │   │
         GND     +5V P1.0 GND P1.1 P1.4 P1.5 P1.6 P1.7
                      RS      E    D4   D5   D6   D7

MEMORY MAP:
──────────

Line 1: DDRAM Address 0x00 - 0x0F (16 characters)
Line 2: DDRAM Address 0x40 - 0x4F (16 characters)

Position: │ 0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15
──────────┼─────────────────────────────────────────────────
Line 1:   │00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F
Line 2:   │40 41 42 43 44 45 46 47 48 49 4A 4B 4C 4D 4E 4F
```

### 1.2 LCD Driver Code

```c
/*
 * HD44780 16×2 LCD DRIVER
 * 4-bit mode implementation
 */

#include <reg51.h>

// LCD connections
#define LCD_DATA  P1  // D4-D7 on P1.4-P1.7
sbit LCD_RS = P1^0;
sbit LCD_EN = P1^1;

// LCD Commands
#define LCD_CLEAR       0x01
#define LCD_HOME        0x02
#define LCD_ENTRY_MODE  0x06
#define LCD_DISPLAY_ON  0x0C
#define LCD_DISPLAY_OFF 0x08
#define LCD_CURSOR_ON   0x0E
#define LCD_BLINK_ON    0x0F
#define LCD_4BIT_2LINE  0x28
#define LCD_LINE1       0x80
#define LCD_LINE2       0xC0

void lcd_delay(unsigned int us)
{
    unsigned int i;
    for(i = 0; i < us; i++);
}

/*
 * SEND 4 BITS TO LCD
 */
void lcd_nibble(unsigned char nibble)
{
    LCD_DATA = (LCD_DATA & 0x0F) | (nibble & 0xF0);
    LCD_EN = 1;
    lcd_delay(1);
    LCD_EN = 0;
    lcd_delay(50);
}

/*
 * SEND COMMAND TO LCD
 */
void lcd_cmd(unsigned char cmd)
{
    LCD_RS = 0;
    lcd_nibble(cmd);        // High nibble
    lcd_nibble(cmd << 4);   // Low nibble
    
    if(cmd == LCD_CLEAR || cmd == LCD_HOME)
        lcd_delay(2000);    // Clear needs more time
}

/*
 * SEND DATA (CHARACTER) TO LCD
 */
void lcd_data(unsigned char dat)
{
    LCD_RS = 1;
    lcd_nibble(dat);
    lcd_nibble(dat << 4);
}

/*
 * INITIALIZE LCD
 */
void lcd_init(void)
{
    lcd_delay(15000);   // Wait 15ms after power on
    
    LCD_RS = 0;
    
    // Special initialization sequence
    lcd_nibble(0x30);   // Function set
    lcd_delay(5000);
    lcd_nibble(0x30);
    lcd_delay(150);
    lcd_nibble(0x30);
    lcd_delay(150);
    
    lcd_nibble(0x20);   // 4-bit mode
    lcd_delay(150);
    
    lcd_cmd(LCD_4BIT_2LINE);   // 4-bit, 2 lines, 5×8 font
    lcd_cmd(LCD_DISPLAY_ON);   // Display ON, cursor OFF
    lcd_cmd(LCD_CLEAR);        // Clear display
    lcd_cmd(LCD_ENTRY_MODE);   // Entry mode: increment, no shift
}

/*
 * SET CURSOR POSITION
 * row: 0 or 1
 * col: 0-15
 */
void lcd_goto(unsigned char row, unsigned char col)
{
    unsigned char addr;
    
    addr = (row == 0) ? LCD_LINE1 : LCD_LINE2;
    addr += col;
    
    lcd_cmd(addr);
}

/*
 * DISPLAY STRING
 */
void lcd_string(char *str)
{
    while(*str)
        lcd_data(*str++);
}

/*
 * DISPLAY INTEGER
 */
void lcd_number(int num)
{
    char buf[8];
    int i = 0;
    bit negative = 0;
    
    if(num < 0)
    {
        negative = 1;
        num = -num;
    }
    
    if(num == 0)
    {
        lcd_data('0');
        return;
    }
    
    while(num > 0)
    {
        buf[i++] = (num % 10) + '0';
        num /= 10;
    }
    
    if(negative)
        lcd_data('-');
    
    while(i--)
        lcd_data(buf[i]);
}

/*
 * CLEAR DISPLAY
 */
void lcd_clear(void)
{
    lcd_cmd(LCD_CLEAR);
}
```

### 1.3 Custom Characters

```c
/*
 * CUSTOM CHARACTER CREATION
 * CGRAM addresses: 0-7 (8 custom chars)
 */

// Custom character patterns (5×8 pixels)
code unsigned char battery_full[8] = {
    0b01110,
    0b11111,
    0b11111,
    0b11111,
    0b11111,
    0b11111,
    0b11111,
    0b11111
};

code unsigned char battery_empty[8] = {
    0b01110,
    0b10001,
    0b10001,
    0b10001,
    0b10001,
    0b10001,
    0b10001,
    0b11111
};

code unsigned char heart[8] = {
    0b00000,
    0b01010,
    0b11111,
    0b11111,
    0b11111,
    0b01110,
    0b00100,
    0b00000
};

code unsigned char thermometer[8] = {
    0b00100,
    0b01010,
    0b01010,
    0b01010,
    0b01110,
    0b11111,
    0b11111,
    0b01110
};

code unsigned char degree[8] = {
    0b00110,
    0b01001,
    0b01001,
    0b00110,
    0b00000,
    0b00000,
    0b00000,
    0b00000
};

/*
 * CREATE CUSTOM CHARACTER
 * location: 0-7
 * pattern: 8-byte pattern array
 */
void lcd_create_char(unsigned char location, unsigned char *pattern)
{
    unsigned char i;
    
    lcd_cmd(0x40 + (location * 8));  // CGRAM address
    
    for(i = 0; i < 8; i++)
        lcd_data(pattern[i]);
    
    lcd_cmd(LCD_LINE1);  // Return to DDRAM
}

/*
 * DISPLAY CUSTOM CHARACTER
 */
void lcd_custom(unsigned char location)
{
    lcd_data(location);  // 0-7 for custom chars
}

/*
 * INITIALIZE CUSTOM CHARACTERS
 */
void lcd_init_custom(void)
{
    lcd_create_char(0, battery_full);
    lcd_create_char(1, battery_empty);
    lcd_create_char(2, heart);
    lcd_create_char(3, thermometer);
    lcd_create_char(4, degree);
}

void main(void)
{
    lcd_init();
    lcd_init_custom();
    
    lcd_string("Temp: 25");
    lcd_custom(4);  // Degree symbol
    lcd_string("C ");
    lcd_custom(3);  // Thermometer
    
    lcd_goto(1, 0);
    lcd_string("Battery: ");
    lcd_custom(0);  // Full battery
    
    while(1);
}
```

---

## 2. Graphics LCD (128×64 GLCD)

### 2.1 GLCD Basics (KS0108)

```
128×64 GRAPHICS LCD (KS0108)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

DISPLAY ORGANIZATION:
───────────────────

The 128×64 display is divided into two halves:
• Left half: Columns 0-63 (CS1)
• Right half: Columns 64-127 (CS2)

Each half has 8 pages (8 rows of 8 pixels each):

    Col: 0  1  2 ....... 63 64 65 ...... 127
    ┌─────────────────────┬──────────────────┐
    │                     │                  │ Page 0
    │     CS1 = 1         │    CS2 = 1       │ (Y=0-7)
    │                     │                  │
    ├─────────────────────┼──────────────────┤
    │                     │                  │ Page 1
    │                     │                  │ (Y=8-15)
    ├─────────────────────┼──────────────────┤
    │        ...          │       ...        │ ...
    ├─────────────────────┼──────────────────┤
    │                     │                  │ Page 7
    │                     │                  │ (Y=56-63)
    └─────────────────────┴──────────────────┘

PAGE DATA FORMAT:
───────────────

Each byte controls 8 vertical pixels:

     Bit 7 ─────► ●     LSB at top
     Bit 6 ─────► ●
     Bit 5 ─────► ●
     Bit 4 ─────► ●
     Bit 3 ─────► ●
     Bit 2 ─────► ●
     Bit 1 ─────► ●
     Bit 0 ─────► ●     MSB at bottom

PIN CONNECTIONS:
──────────────

Pin │ Symbol │ Function
────┼────────┼─────────────────────
 1  │  VSS   │ Ground
 2  │  VDD   │ +5V
 3  │  VO    │ Contrast
 4  │  D/I   │ Data/Instruction
 5  │  R/W   │ Read/Write
 6  │  E     │ Enable
 7  │  DB0   │ Data bit 0
... │  ...   │ ...
14  │  DB7   │ Data bit 7
15  │  CS1   │ Chip Select 1 (left)
16  │  CS2   │ Chip Select 2 (right)
17  │  RST   │ Reset
18  │  VEE   │ Negative voltage
19  │  A     │ Backlight +
20  │  K     │ Backlight -
```

### 2.2 GLCD Driver Code

```c
/*
 * KS0108 128×64 GLCD DRIVER
 */

#include <reg51.h>

// GLCD connections
#define GLCD_DATA  P0

sbit GLCD_DI  = P2^0;   // D/I (RS)
sbit GLCD_RW  = P2^1;   // R/W
sbit GLCD_EN  = P2^2;   // Enable
sbit GLCD_CS1 = P2^3;   // Chip Select 1
sbit GLCD_CS2 = P2^4;   // Chip Select 2
sbit GLCD_RST = P2^5;   // Reset

// GLCD Commands
#define GLCD_ON         0x3F
#define GLCD_OFF        0x3E
#define GLCD_SET_PAGE   0xB8
#define GLCD_SET_Y      0x40
#define GLCD_START_LINE 0xC0

// Display buffer (optional, for bitmap operations)
unsigned char xdata glcd_buffer[1024];  // 128×64/8

void glcd_delay(void)
{
    unsigned char i;
    for(i = 0; i < 10; i++);
}

/*
 * WRITE COMMAND TO GLCD
 */
void glcd_cmd(unsigned char cmd, bit chip)
{
    GLCD_CS1 = !chip;
    GLCD_CS2 = chip;
    
    GLCD_DI = 0;    // Command
    GLCD_RW = 0;    // Write
    GLCD_DATA = cmd;
    
    GLCD_EN = 1;
    glcd_delay();
    GLCD_EN = 0;
    glcd_delay();
}

/*
 * WRITE DATA TO GLCD
 */
void glcd_data(unsigned char dat, bit chip)
{
    GLCD_CS1 = !chip;
    GLCD_CS2 = chip;
    
    GLCD_DI = 1;    // Data
    GLCD_RW = 0;    // Write
    GLCD_DATA = dat;
    
    GLCD_EN = 1;
    glcd_delay();
    GLCD_EN = 0;
    glcd_delay();
}

/*
 * INITIALIZE GLCD
 */
void glcd_init(void)
{
    GLCD_RST = 0;
    glcd_delay();
    GLCD_RST = 1;
    glcd_delay();
    
    // Initialize both chips
    glcd_cmd(GLCD_OFF, 0);
    glcd_cmd(GLCD_OFF, 1);
    
    glcd_cmd(GLCD_START_LINE, 0);
    glcd_cmd(GLCD_START_LINE, 1);
    
    glcd_cmd(GLCD_ON, 0);
    glcd_cmd(GLCD_ON, 1);
}

/*
 * CLEAR DISPLAY
 */
void glcd_clear(void)
{
    unsigned char page, col;
    
    for(page = 0; page < 8; page++)
    {
        glcd_cmd(GLCD_SET_PAGE | page, 0);
        glcd_cmd(GLCD_SET_Y, 0);
        for(col = 0; col < 64; col++)
            glcd_data(0x00, 0);
        
        glcd_cmd(GLCD_SET_PAGE | page, 1);
        glcd_cmd(GLCD_SET_Y, 1);
        for(col = 0; col < 64; col++)
            glcd_data(0x00, 1);
    }
}

/*
 * SET PIXEL
 * x: 0-127
 * y: 0-63
 */
void glcd_set_pixel(unsigned char x, unsigned char y)
{
    unsigned char page, bit_pos, dat;
    bit chip;
    
    if(x >= 128 || y >= 64) return;
    
    chip = (x >= 64);
    if(chip) x -= 64;
    
    page = y / 8;
    bit_pos = y % 8;
    
    glcd_cmd(GLCD_SET_PAGE | page, chip);
    glcd_cmd(GLCD_SET_Y | x, chip);
    
    // Set single bit (simplified - real implementation needs read-modify-write)
    glcd_data(1 << bit_pos, chip);
}

/*
 * DRAW LINE (Bresenham's algorithm)
 */
void glcd_line(int x0, int y0, int x1, int y1)
{
    int dx = abs(x1 - x0);
    int dy = abs(y1 - y0);
    int sx = (x0 < x1) ? 1 : -1;
    int sy = (y0 < y1) ? 1 : -1;
    int err = dx - dy;
    int e2;
    
    while(1)
    {
        glcd_set_pixel(x0, y0);
        
        if(x0 == x1 && y0 == y1) break;
        
        e2 = 2 * err;
        
        if(e2 > -dy)
        {
            err -= dy;
            x0 += sx;
        }
        
        if(e2 < dx)
        {
            err += dx;
            y0 += sy;
        }
    }
}

/*
 * DRAW RECTANGLE
 */
void glcd_rect(unsigned char x, unsigned char y, 
               unsigned char w, unsigned char h)
{
    glcd_line(x, y, x + w, y);          // Top
    glcd_line(x, y + h, x + w, y + h);  // Bottom
    glcd_line(x, y, x, y + h);          // Left
    glcd_line(x + w, y, x + w, y + h);  // Right
}

/*
 * DRAW FILLED RECTANGLE
 */
void glcd_fill_rect(unsigned char x, unsigned char y,
                    unsigned char w, unsigned char h)
{
    unsigned char i, j;
    
    for(i = 0; i < h; i++)
        for(j = 0; j < w; j++)
            glcd_set_pixel(x + j, y + i);
}

/*
 * DRAW CIRCLE (Midpoint algorithm)
 */
void glcd_circle(unsigned char cx, unsigned char cy, unsigned char r)
{
    int x = r;
    int y = 0;
    int err = 0;
    
    while(x >= y)
    {
        glcd_set_pixel(cx + x, cy + y);
        glcd_set_pixel(cx + y, cy + x);
        glcd_set_pixel(cx - y, cy + x);
        glcd_set_pixel(cx - x, cy + y);
        glcd_set_pixel(cx - x, cy - y);
        glcd_set_pixel(cx - y, cy - x);
        glcd_set_pixel(cx + y, cy - x);
        glcd_set_pixel(cx + x, cy - y);
        
        y++;
        err += 2*y + 1;
        
        if(2*err + 1 > 2*x)
        {
            x--;
            err -= 2*x + 1;
        }
    }
}
```

### 2.3 GLCD Text and Fonts

```c
/*
 * 5×7 FONT FOR GLCD
 */

code unsigned char font_5x7[96][5] = {
    {0x00, 0x00, 0x00, 0x00, 0x00},  // Space (32)
    {0x00, 0x00, 0x5F, 0x00, 0x00},  // !
    {0x00, 0x07, 0x00, 0x07, 0x00},  // "
    {0x14, 0x7F, 0x14, 0x7F, 0x14},  // #
    {0x24, 0x2A, 0x7F, 0x2A, 0x12},  // $
    {0x23, 0x13, 0x08, 0x64, 0x62},  // %
    // ... (complete alphabet)
    {0x7F, 0x08, 0x08, 0x08, 0x7F},  // H
    {0x00, 0x41, 0x7F, 0x41, 0x00},  // I
    // ... more characters
    {0x7C, 0x12, 0x11, 0x12, 0x7C},  // A (65)
    {0x7F, 0x49, 0x49, 0x49, 0x36},  // B
    {0x3E, 0x41, 0x41, 0x41, 0x22},  // C
    // ... complete font table
};

// Current cursor position
unsigned char text_x = 0;
unsigned char text_y = 0;

/*
 * DISPLAY CHARACTER AT POSITION
 */
void glcd_putchar(unsigned char x, unsigned char page, char ch)
{
    unsigned char i;
    bit chip;
    
    if(ch < 32) return;
    ch -= 32;  // Font starts at space
    
    for(i = 0; i < 5; i++)
    {
        chip = (x >= 64);
        if(chip) x -= 64;
        
        glcd_cmd(GLCD_SET_PAGE | page, chip);
        glcd_cmd(GLCD_SET_Y | x, chip);
        glcd_data(font_5x7[ch][i], chip);
        
        x++;
        if(chip) x += 64;
    }
    
    // Space between characters
    chip = (x >= 64);
    if(chip) x -= 64;
    glcd_data(0x00, chip);
}

/*
 * DISPLAY STRING
 */
void glcd_puts(unsigned char x, unsigned char page, char *str)
{
    while(*str)
    {
        glcd_putchar(x, page, *str++);
        x += 6;
        
        if(x >= 128)
        {
            x = 0;
            page++;
            if(page >= 8) page = 0;
        }
    }
}

/*
 * DISPLAY NUMBER
 */
void glcd_putnumber(unsigned char x, unsigned char page, int num)
{
    char buf[8];
    sprintf(buf, "%d", num);
    glcd_puts(x, page, buf);
}
```

---

## 3. OLED Display (SSD1306)

### 3.1 SSD1306 I2C OLED

```
SSD1306 128×64 OLED DISPLAY
━━━━━━━━━━━━━━━━━━━━━━━━━━━

I2C CONNECTION:
─────────────

    8051 MCU              0.96" OLED Module
    ┌───────────┐         ┌─────────────────┐
    │           │         │                 │
    │   P2.0 ───┼── SDA ──┤ SDA             │
    │           │         │                 │
    │   P2.1 ───┼── SCL ──┤ SCL             │
    │           │         │                 │
    │   VCC ────┼─────────┤ VCC (3.3V/5V)   │
    │           │         │                 │
    │   GND ────┼─────────┤ GND             │
    │           │         │                 │
    └───────────┘         └─────────────────┘

I2C Address: 0x3C (or 0x3D if SA0=1)

MEMORY ORGANIZATION:
──────────────────

128×64 = 8192 pixels = 1024 bytes
Organized as 8 pages × 128 columns

Page addressing mode (vertical bytes):

    Col: 0   1   2  ...  127
    ┌────────────────────────┐
    │ █ █ █ █ █ █ ... █ █ █ │ Page 0 (Y=0-7)
    │ █ █ █ █ █ █ ... █ █ █ │ Page 1 (Y=8-15)
    │ ...                    │
    │ █ █ █ █ █ █ ... █ █ █ │ Page 7 (Y=56-63)
    └────────────────────────┘

Each column in a page is a vertical byte:
    Bit 0 = top pixel
    Bit 7 = bottom pixel
```

### 3.2 SSD1306 Driver Code

```c
/*
 * SSD1306 I2C OLED DRIVER
 * 128×64 display
 */

#include <reg51.h>

#define OLED_ADDR  0x3C

// Display buffer (1KB)
unsigned char xdata oled_buffer[1024];

// SSD1306 Commands
#define OLED_DISPLAYOFF         0xAE
#define OLED_DISPLAYON          0xAF
#define OLED_SETDISPLAYCLOCKDIV 0xD5
#define OLED_SETMULTIPLEX       0xA8
#define OLED_SETDISPLAYOFFSET   0xD3
#define OLED_SETSTARTLINE       0x40
#define OLED_CHARGEPUMP         0x8D
#define OLED_MEMORYMODE         0x20
#define OLED_SEGREMAP           0xA0
#define OLED_COMSCANDEC         0xC8
#define OLED_SETCOMPINS         0xDA
#define OLED_SETCONTRAST        0x81
#define OLED_SETPRECHARGE       0xD9
#define OLED_SETVCOMDETECT      0xDB
#define OLED_DISPLAYALLON_RESUME 0xA4
#define OLED_NORMALDISPLAY      0xA6
#define OLED_INVERTDISPLAY      0xA7

/*
 * SEND COMMAND TO OLED
 */
void oled_cmd(unsigned char cmd)
{
    i2c_start();
    i2c_write(OLED_ADDR << 1);  // Write mode
    i2c_write(0x00);             // Command mode
    i2c_write(cmd);
    i2c_stop();
}

/*
 * SEND DATA TO OLED
 */
void oled_data(unsigned char dat)
{
    i2c_start();
    i2c_write(OLED_ADDR << 1);
    i2c_write(0x40);             // Data mode
    i2c_write(dat);
    i2c_stop();
}

/*
 * SEND DATA BUFFER
 */
void oled_data_buffer(unsigned char *buf, unsigned int len)
{
    unsigned int i;
    
    i2c_start();
    i2c_write(OLED_ADDR << 1);
    i2c_write(0x40);
    
    for(i = 0; i < len; i++)
        i2c_write(buf[i]);
    
    i2c_stop();
}

/*
 * INITIALIZE OLED
 */
void oled_init(void)
{
    i2c_init();
    delay_ms(100);
    
    oled_cmd(OLED_DISPLAYOFF);
    
    oled_cmd(OLED_SETDISPLAYCLOCKDIV);
    oled_cmd(0x80);
    
    oled_cmd(OLED_SETMULTIPLEX);
    oled_cmd(63);  // 64 rows
    
    oled_cmd(OLED_SETDISPLAYOFFSET);
    oled_cmd(0x00);
    
    oled_cmd(OLED_SETSTARTLINE | 0x00);
    
    oled_cmd(OLED_CHARGEPUMP);
    oled_cmd(0x14);  // Enable charge pump
    
    oled_cmd(OLED_MEMORYMODE);
    oled_cmd(0x00);  // Horizontal addressing
    
    oled_cmd(OLED_SEGREMAP | 0x01);
    oled_cmd(OLED_COMSCANDEC);
    
    oled_cmd(OLED_SETCOMPINS);
    oled_cmd(0x12);
    
    oled_cmd(OLED_SETCONTRAST);
    oled_cmd(0xCF);
    
    oled_cmd(OLED_SETPRECHARGE);
    oled_cmd(0xF1);
    
    oled_cmd(OLED_SETVCOMDETECT);
    oled_cmd(0x40);
    
    oled_cmd(OLED_DISPLAYALLON_RESUME);
    oled_cmd(OLED_NORMALDISPLAY);
    
    oled_cmd(OLED_DISPLAYON);
}

/*
 * CLEAR DISPLAY BUFFER
 */
void oled_clear(void)
{
    unsigned int i;
    
    for(i = 0; i < 1024; i++)
        oled_buffer[i] = 0x00;
}

/*
 * UPDATE DISPLAY FROM BUFFER
 */
void oled_display(void)
{
    unsigned char page;
    
    for(page = 0; page < 8; page++)
    {
        oled_cmd(0xB0 + page);  // Set page
        oled_cmd(0x00);         // Lower column
        oled_cmd(0x10);         // Upper column
        
        oled_data_buffer(&oled_buffer[page * 128], 128);
    }
}

/*
 * SET PIXEL IN BUFFER
 */
void oled_set_pixel(unsigned char x, unsigned char y, bit color)
{
    unsigned int index;
    
    if(x >= 128 || y >= 64) return;
    
    index = x + (y / 8) * 128;
    
    if(color)
        oled_buffer[index] |= (1 << (y % 8));
    else
        oled_buffer[index] &= ~(1 << (y % 8));
}

/*
 * DRAW STRING ON OLED
 */
void oled_puts(unsigned char x, unsigned char y, char *str)
{
    unsigned char page = y / 8;
    unsigned int idx;
    unsigned char i, c;
    
    while(*str)
    {
        c = *str++ - 32;
        
        for(i = 0; i < 5; i++)
        {
            if(x >= 128) break;
            idx = x + page * 128;
            oled_buffer[idx] = font_5x7[c][i];
            x++;
        }
        
        // Space
        if(x < 128)
        {
            oled_buffer[x + page * 128] = 0x00;
            x++;
        }
    }
}

/*
 * DRAW PROGRESS BAR
 */
void oled_progress_bar(unsigned char x, unsigned char y,
                       unsigned char w, unsigned char h,
                       unsigned char percent)
{
    unsigned char filled_w;
    unsigned char i, j;
    
    if(percent > 100) percent = 100;
    filled_w = (unsigned int)w * percent / 100;
    
    // Border
    for(i = x; i < x + w; i++)
    {
        oled_set_pixel(i, y, 1);
        oled_set_pixel(i, y + h - 1, 1);
    }
    for(j = y; j < y + h; j++)
    {
        oled_set_pixel(x, j, 1);
        oled_set_pixel(x + w - 1, j, 1);
    }
    
    // Fill
    for(i = x + 2; i < x + filled_w - 2; i++)
        for(j = y + 2; j < y + h - 2; j++)
            oled_set_pixel(i, j, 1);
}
```

---

## 4. Menu System Design

### 4.1 Menu Structure

```c
/*
 * EMBEDDED MENU SYSTEM
 * For LCD/OLED displays
 */

#include <reg51.h>

// Menu item structure
typedef struct MenuItem {
    char *text;
    void (*action)(void);
    struct MenuItem *submenu;
    unsigned char submenu_count;
} MenuItem;

// Menu state
typedef struct {
    MenuItem *current_menu;
    unsigned char menu_count;
    unsigned char selected_index;
    unsigned char top_index;
    unsigned char max_visible;
} MenuState;

MenuState menu_state;

// Forward declarations for menu actions
void action_temp_display(void);
void action_led_control(void);
void action_motor_control(void);
void action_settings(void);
void action_about(void);

void action_set_time(void);
void action_set_brightness(void);
void action_back(void);

// Submenu: Settings
MenuItem settings_menu[] = {
    {"Set Time",       action_set_time,      NULL, 0},
    {"Brightness",     action_set_brightness, NULL, 0},
    {"Back",           action_back,          NULL, 0}
};

// Main menu
MenuItem main_menu[] = {
    {"Temperature",    action_temp_display,  NULL, 0},
    {"LED Control",    action_led_control,   NULL, 0},
    {"Motor Control",  action_motor_control, NULL, 0},
    {"Settings",       action_settings,      settings_menu, 3},
    {"About",          action_about,         NULL, 0}
};

/*
 * INITIALIZE MENU
 */
void menu_init(void)
{
    menu_state.current_menu = main_menu;
    menu_state.menu_count = 5;
    menu_state.selected_index = 0;
    menu_state.top_index = 0;
    menu_state.max_visible = 4;  // For 4-line display
}

/*
 * DISPLAY MENU
 */
void menu_display(void)
{
    unsigned char i;
    unsigned char display_idx;
    
    lcd_clear();
    
    for(i = 0; i < menu_state.max_visible; i++)
    {
        display_idx = menu_state.top_index + i;
        
        if(display_idx >= menu_state.menu_count)
            break;
        
        lcd_goto(i, 0);
        
        // Selection indicator
        if(display_idx == menu_state.selected_index)
            lcd_data('>');
        else
            lcd_data(' ');
        
        lcd_string(menu_state.current_menu[display_idx].text);
    }
}

/*
 * NAVIGATE UP
 */
void menu_up(void)
{
    if(menu_state.selected_index > 0)
    {
        menu_state.selected_index--;
        
        if(menu_state.selected_index < menu_state.top_index)
            menu_state.top_index--;
    }
    
    menu_display();
}

/*
 * NAVIGATE DOWN
 */
void menu_down(void)
{
    if(menu_state.selected_index < menu_state.menu_count - 1)
    {
        menu_state.selected_index++;
        
        if(menu_state.selected_index >= 
           menu_state.top_index + menu_state.max_visible)
            menu_state.top_index++;
    }
    
    menu_display();
}

/*
 * SELECT/ENTER
 */
void menu_select(void)
{
    MenuItem *item = &menu_state.current_menu[menu_state.selected_index];
    
    if(item->submenu != NULL)
    {
        // Enter submenu
        menu_state.current_menu = item->submenu;
        menu_state.menu_count = item->submenu_count;
        menu_state.selected_index = 0;
        menu_state.top_index = 0;
        menu_display();
    }
    else if(item->action != NULL)
    {
        // Execute action
        item->action();
    }
}

/*
 * MENU ACTIONS
 */
void action_temp_display(void)
{
    int temp;
    char buf[16];
    
    lcd_clear();
    lcd_string("Temperature:");
    
    while(!key_pressed())
    {
        temp = read_temperature();
        lcd_goto(1, 0);
        sprintf(buf, "%d.%d C   ", temp / 10, temp % 10);
        lcd_string(buf);
        delay_ms(500);
    }
    
    menu_display();
}

void action_back(void)
{
    menu_state.current_menu = main_menu;
    menu_state.menu_count = 5;
    menu_state.selected_index = 0;
    menu_state.top_index = 0;
    menu_display();
}

/*
 * MAIN WITH MENU SYSTEM
 */
void main(void)
{
    lcd_init();
    keypad_init();
    menu_init();
    menu_display();
    
    while(1)
    {
        unsigned char key = keypad_scan();
        
        switch(key)
        {
            case '2':  // Up
                menu_up();
                break;
            case '8':  // Down
                menu_down();
                break;
            case '5':  // Select
            case '#':
                menu_select();
                break;
        }
        
        delay_ms(50);
    }
}
```

---

## 5. Complete HMI Project

### 5.1 Multi-Page Dashboard

```c
/*
 * COMPLETE HMI DASHBOARD
 * With multiple pages and real-time updates
 */

#include <reg51.h>
#include <stdio.h>

// Page definitions
typedef enum {
    PAGE_HOME,
    PAGE_SENSORS,
    PAGE_CONTROLS,
    PAGE_SETTINGS,
    PAGE_COUNT
} Page;

Page current_page = PAGE_HOME;

// Sensor data
typedef struct {
    int temperature;
    int humidity;
    int light;
    int pressure;
} SensorData;

SensorData sensors;

// Control states
typedef struct {
    bit led1;
    bit led2;
    bit relay;
    unsigned char motor_speed;
} Controls;

Controls controls = {0, 0, 0, 0};

/*
 * DISPLAY HOME PAGE
 */
void display_home(void)
{
    char buf[17];
    
    oled_clear();
    
    // Title bar
    oled_puts(0, 0, "== Smart Home ==");
    
    // Time
    sprintf(buf, "%02d:%02d:%02d", 
            rtc_get_hours(), 
            rtc_get_minutes(), 
            rtc_get_seconds());
    oled_puts(40, 16, buf);
    
    // Date
    sprintf(buf, "%02d/%02d/20%02d",
            rtc_get_date(),
            rtc_get_month(),
            rtc_get_year());
    oled_puts(30, 28, buf);
    
    // Quick status
    sprintf(buf, "T:%dC H:%d%%", 
            sensors.temperature, 
            sensors.humidity);
    oled_puts(0, 48, buf);
    
    // Page indicator
    oled_puts(0, 56, "[1]2 3 4");
    
    oled_display();
}

/*
 * DISPLAY SENSORS PAGE
 */
void display_sensors(void)
{
    char buf[17];
    
    oled_clear();
    
    oled_puts(0, 0, "=== Sensors ===");
    
    // Temperature with bar
    sprintf(buf, "Temp: %d C", sensors.temperature);
    oled_puts(0, 16, buf);
    oled_progress_bar(70, 14, 50, 10, 
                      (sensors.temperature + 20) * 100 / 70);
    
    // Humidity with bar
    sprintf(buf, "Humid: %d%%", sensors.humidity);
    oled_puts(0, 28, buf);
    oled_progress_bar(70, 26, 50, 10, sensors.humidity);
    
    // Light level
    sprintf(buf, "Light: %d lux", sensors.light);
    oled_puts(0, 40, buf);
    
    // Page indicator
    oled_puts(0, 56, " 1[2]3 4");
    
    oled_display();
}

/*
 * DISPLAY CONTROLS PAGE
 */
void display_controls(void)
{
    char buf[17];
    
    oled_clear();
    
    oled_puts(0, 0, "=== Controls ===");
    
    // LED 1
    sprintf(buf, "LED1: %s", controls.led1 ? "ON " : "OFF");
    oled_puts(0, 16, buf);
    
    // LED 2
    sprintf(buf, "LED2: %s", controls.led2 ? "ON " : "OFF");
    oled_puts(0, 26, buf);
    
    // Relay
    sprintf(buf, "Relay: %s", controls.relay ? "ON " : "OFF");
    oled_puts(0, 36, buf);
    
    // Motor
    sprintf(buf, "Motor: %d%%", controls.motor_speed);
    oled_puts(0, 46, buf);
    oled_progress_bar(60, 44, 60, 10, controls.motor_speed);
    
    // Page indicator
    oled_puts(0, 56, " 1 2[3]4");
    
    oled_display();
}

/*
 * HANDLE KEY INPUT
 */
void handle_keys(void)
{
    unsigned char key = keypad_scan();
    
    switch(key)
    {
        case '1':
            current_page = PAGE_HOME;
            break;
            
        case '2':
            current_page = PAGE_SENSORS;
            break;
            
        case '3':
            current_page = PAGE_CONTROLS;
            break;
            
        case '4':
            current_page = PAGE_SETTINGS;
            break;
            
        // Control actions on controls page
        case 'A':
            if(current_page == PAGE_CONTROLS)
                controls.led1 = !controls.led1;
            break;
            
        case 'B':
            if(current_page == PAGE_CONTROLS)
                controls.led2 = !controls.led2;
            break;
            
        case 'C':
            if(current_page == PAGE_CONTROLS)
                controls.relay = !controls.relay;
            break;
            
        case '*':
            if(current_page == PAGE_CONTROLS)
            {
                controls.motor_speed -= 10;
                if(controls.motor_speed > 100)  // Wrapped
                    controls.motor_speed = 0;
            }
            break;
            
        case '#':
            if(current_page == PAGE_CONTROLS)
            {
                controls.motor_speed += 10;
                if(controls.motor_speed > 100)
                    controls.motor_speed = 100;
            }
            break;
    }
}

/*
 * APPLY CONTROL OUTPUTS
 */
void apply_controls(void)
{
    LED1 = controls.led1;
    LED2 = controls.led2;
    RELAY = controls.relay;
    set_motor_speed(controls.motor_speed);
}

/*
 * MAIN FUNCTION
 */
void main(void)
{
    // Initialize
    oled_init();
    keypad_init();
    adc_init();
    rtc_init();
    motor_init();
    
    while(1)
    {
        // Read sensors
        sensors.temperature = read_temperature();
        sensors.humidity = read_humidity();
        sensors.light = read_light();
        
        // Handle input
        handle_keys();
        
        // Update display
        switch(current_page)
        {
            case PAGE_HOME:
                display_home();
                break;
            case PAGE_SENSORS:
                display_sensors();
                break;
            case PAGE_CONTROLS:
                display_controls();
                break;
            case PAGE_SETTINGS:
                display_settings();
                break;
        }
        
        // Apply outputs
        apply_controls();
        
        delay_ms(100);
    }
}
```

---

## 📋 Display Comparison Table

| Feature | HD44780 16×2 | KS0108 128×64 | SSD1306 OLED |
|---------|--------------|---------------|--------------|
| Type | Character | Graphics | Graphics |
| Resolution | 16×2 chars | 128×64 pixels | 128×64 pixels |
| Interface | 4/8-bit parallel | 8-bit parallel | I2C/SPI |
| Controller | HD44780 | KS0108 | SSD1306 |
| Backlight | LED | LED | Self-emitting |
| Power | ~5mA | ~20mA | ~20mA |
| Contrast | Adjustable | Adjustable | Fixed |
| Viewing angle | Limited | Limited | Wide |
| Cost | Low | Medium | Medium |
| Ideal for | Simple text | Graphics, charts | Portable, graphics |

---

## ❓ Quick Revision Questions

1. **What is the purpose of the E (Enable) pin on HD44780 LCD?**
   <details>
   <summary>Show Answer</summary>
   Enable pin latches data on falling edge. When E goes from HIGH to LOW, the LCD reads the data/command from data pins. Must pulse E for each byte transferred.
   </details>

2. **How is a 128×64 GLCD organized into pages?**
   <details>
   <summary>Show Answer</summary>
   Display divided into 8 horizontal pages, each 8 pixels high. Each page contains 128 columns. Data written as vertical bytes - one byte controls 8 vertical pixels in a column.
   </details>

3. **What is CGRAM in HD44780 and how many custom characters can be stored?**
   <details>
   <summary>Show Answer</summary>
   CGRAM (Character Generator RAM) stores custom character patterns. Can store 8 custom 5×8 pixel characters at addresses 0-7. Each character uses 8 bytes (one per row).
   </details>

4. **Why does SSD1306 OLED use a display buffer in RAM?**
   <details>
   <summary>Show Answer</summary>
   Buffer enables read-modify-write operations for pixel manipulation. OLED cannot read back display memory efficiently. Buffer allows drawing shapes, fonts, and then updating display in one transfer.
   </details>

5. **What is the advantage of I2C over parallel interface for displays?**
   <details>
   <summary>Show Answer</summary>
   I2C uses only 2 wires (SDA, SCL) vs 10+ for parallel. Saves GPIO pins. Can connect multiple displays on same bus with different addresses. Simpler PCB routing.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [10.4 Communication Projects](04-communication-projects.md) | [Unit 10 Index](README.md) | [10.6 Complete System Projects](06-complete-system-projects.md) |

---

*[← Previous: Communication Projects](04-communication-projects.md) | [Next: Complete System Projects →](06-complete-system-projects.md)*
