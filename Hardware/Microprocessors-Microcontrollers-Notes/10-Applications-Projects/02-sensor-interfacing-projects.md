# Chapter 10.2: Sensor Interfacing Projects

## 📚 Chapter Overview

This chapter covers practical projects for interfacing various sensors with microcontrollers, including analog sensors, digital sensors, and sensor calibration techniques.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Interface analog sensors using ADC
- Communicate with digital sensors via I2C and 1-Wire
- Implement sensor calibration routines
- Process sensor data for accuracy
- Build complete sensor-based projects

---

## 1. Temperature Sensor Projects

### 1.1 LM35 Analog Temperature Sensor

```
LM35 TEMPERATURE SENSOR PROJECT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SPECIFICATIONS:
─────────────
• Output: 10 mV/°C
• Range: -55°C to +150°C (LM35)
• Accuracy: ±0.5°C at 25°C
• Supply: 4V to 30V

CIRCUIT DIAGRAM:
──────────────

                    +5V
                     │
                     │
               ┌─────┴─────┐
               │   LM35    │
               │           │
               │  ┌─┬─┬─┐  │
               └──┤V│O│G├──┘
                  │ │ │
                  │ │ └────────► GND
                  │ │
                  │ └───────┬──► To ADC (P1.0)
                  │         │
           +5V ◄──┘     ┌───┴───┐
                        │ 0.1µF │
                        │       │
                        └───┬───┘
                            │
                           GND

ADC0804 CONNECTION:
─────────────────

                         +5V    +5V
                          │      │
                          │    ┌─┴─┐
                   ┌──────┴────┤   │
                   │           └───┘
             ┌─────┴────────────────────────────────┐
             │                ADC0804               │
             │                                      │
    LM35 ───►│ VIN+                          DB0-7 │──► P0 (8051)
             │                                      │
        GND─►│ VIN-                             RD │◄── P3.2
             │                                      │
        GND─►│ AGND                             WR │◄── P3.3
             │                                      │
             │ CLKIN                           INTR │──► P3.5
             │   │                                  │
             │   ├───[10K]───┤├── 150pF ──►GND     │
             │   │                                  │
             └───┴──────────────────────────────────┘
```

### 1.2 LM35 Code Implementation

```c
/*
 * LM35 TEMPERATURE SENSOR WITH ADC0804
 * Platform: 8051
 */

#include <reg51.h>
#include <stdio.h>

// ADC0804 Pins
sbit ADC_RD   = P3^2;
sbit ADC_WR   = P3^3;
sbit ADC_INTR = P3^5;

// LCD Pins
#define LCD_DATA P2
sbit LCD_RS = P1^0;
sbit LCD_RW = P1^1;
sbit LCD_EN = P1^2;

// Function prototypes
void delay_ms(unsigned int ms);
void lcd_init(void);
void lcd_cmd(unsigned char cmd);
void lcd_data(unsigned char dat);
void lcd_string(char *str);
void lcd_goto(unsigned char row, unsigned char col);
unsigned char adc_read(void);
float convert_to_celsius(unsigned char adc_value);

/*
 * MAIN FUNCTION
 */
void main(void)
{
    unsigned char adc_value;
    float temperature;
    char display[16];
    
    // Initialize
    lcd_init();
    ADC_RD = 1;
    ADC_WR = 1;
    
    lcd_string("Temp Monitor");
    delay_ms(1000);
    lcd_cmd(0x01);  // Clear
    
    while(1)
    {
        // Read ADC
        adc_value = adc_read();
        
        // Convert to Celsius
        temperature = convert_to_celsius(adc_value);
        
        // Display on LCD
        lcd_goto(0, 0);
        sprintf(display, "Temp: %.1f C  ", temperature);
        lcd_string(display);
        
        // Display ADC value on line 2
        lcd_goto(1, 0);
        sprintf(display, "ADC: %03d     ", adc_value);
        lcd_string(display);
        
        delay_ms(500);
    }
}

/*
 * ADC READ FUNCTION
 * Returns 8-bit ADC value
 */
unsigned char adc_read(void)
{
    unsigned char value;
    
    // Start conversion (WR pulse)
    ADC_WR = 0;
    delay_ms(1);
    ADC_WR = 1;
    
    // Wait for conversion (INTR goes low)
    while(ADC_INTR == 1);
    
    // Read data (RD pulse)
    ADC_RD = 0;
    delay_ms(1);
    value = P0;
    ADC_RD = 1;
    
    return value;
}

/*
 * CONVERT ADC VALUE TO CELSIUS
 * 
 * LM35: 10mV/°C
 * ADC: 5V / 256 steps = 19.53 mV/step
 * 
 * Temperature = ADC_value × (5000 mV / 256) / 10 mV/°C
 *             = ADC_value × 1.953 °C
 * 
 * For better accuracy with 2.56V reference:
 * Temperature = ADC_value × (2560 mV / 256) / 10 = ADC_value
 */
float convert_to_celsius(unsigned char adc_value)
{
    // Using 5V reference
    return (float)adc_value * 500.0 / 256.0;
}
```

### 1.3 DHT11 Digital Temperature/Humidity Sensor

```
DHT11 SENSOR PROJECT
━━━━━━━━━━━━━━━━━━━━

SPECIFICATIONS:
─────────────
• Temperature: 0-50°C, ±2°C accuracy
• Humidity: 20-90% RH, ±5% accuracy
• Interface: Single wire digital
• Sampling rate: 1 Hz max

PROTOCOL TIMING:
──────────────

MCU START SIGNAL:
                 ┌────────────────────────────────┐
                 │                                │
    ─────────────┘   18ms LOW    │    20-40µs    └─── Float high
                                 │
                                 ▼
DHT11 RESPONSE:
    ┌──────┐     ┌────┐
    │ 80µs │     │80µs│     Then data...
────┘      └─────┘    └────

DATA BIT ENCODING:
    ┌──┐          50µs low + 26-28µs high = '0'
────┘  └──────────
    
    ┌──┐          50µs low + 70µs high = '1'  
────┘  └──────────────

DATA FORMAT (40 bits):
┌──────────────┬──────────────┬──────────────┬──────────────┬──────────────┐
│ Humidity Int │ Humidity Dec │   Temp Int   │  Temp Dec    │   Checksum   │
│   (8 bits)   │   (8 bits)   │   (8 bits)   │   (8 bits)   │   (8 bits)   │
└──────────────┴──────────────┴──────────────┴──────────────┴──────────────┘
```

```c
/*
 * DHT11 SENSOR INTERFACE
 * Single-wire protocol implementation
 */

#include <reg51.h>

sbit DHT11_PIN = P2^0;

// Data storage
unsigned char humidity_int, humidity_dec;
unsigned char temperature_int, temperature_dec;
unsigned char checksum;

/*
 * READ DHT11 SENSOR
 * Returns 1 on success, 0 on failure
 */
unsigned char dht11_read(void)
{
    unsigned char i, j;
    unsigned char data[5] = {0};
    
    // STEP 1: Send start signal
    DHT11_PIN = 0;      // Pull low
    delay_ms(20);       // Hold for 18-20ms
    DHT11_PIN = 1;      // Release
    delay_us(30);       // Wait 20-40µs
    
    // STEP 2: Check DHT11 response
    if(DHT11_PIN == 1)  // Should be low
        return 0;       // No response
    
    delay_us(80);       // Wait through 80µs low
    
    if(DHT11_PIN == 0)  // Should be high now
        return 0;
    
    delay_us(80);       // Wait through 80µs high
    
    // STEP 3: Read 40 bits of data
    for(i = 0; i < 5; i++)
    {
        for(j = 0; j < 8; j++)
        {
            // Wait for pin to go high (start of bit)
            while(DHT11_PIN == 0);
            
            // Measure high time
            delay_us(35);  // Wait past '0' threshold
            
            if(DHT11_PIN == 1)
            {
                // It's a '1' (high for ~70µs)
                data[i] |= (1 << (7 - j));
                while(DHT11_PIN == 1);  // Wait for end
            }
            // else it's a '0' (already went low)
        }
    }
    
    // STEP 4: Verify checksum
    if(data[4] != (data[0] + data[1] + data[2] + data[3]))
        return 0;  // Checksum error
    
    // STEP 5: Store data
    humidity_int = data[0];
    humidity_dec = data[1];
    temperature_int = data[2];
    temperature_dec = data[3];
    checksum = data[4];
    
    return 1;  // Success
}

void main(void)
{
    char buf[16];
    
    lcd_init();
    
    while(1)
    {
        if(dht11_read())
        {
            lcd_goto(0, 0);
            sprintf(buf, "Temp: %d.%d C", temperature_int, temperature_dec);
            lcd_string(buf);
            
            lcd_goto(1, 0);
            sprintf(buf, "Hum:  %d.%d %%", humidity_int, humidity_dec);
            lcd_string(buf);
        }
        else
        {
            lcd_goto(0, 0);
            lcd_string("Sensor Error!   ");
        }
        
        delay_ms(2000);  // DHT11 needs 1 second between reads
    }
}
```

---

## 2. Distance Measurement Projects

### 2.1 HC-SR04 Ultrasonic Sensor

```
HC-SR04 ULTRASONIC DISTANCE SENSOR
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SPECIFICATIONS:
─────────────
• Range: 2 cm to 400 cm
• Resolution: 0.3 cm
• Beam angle: 15 degrees
• Trigger: 10µs pulse

OPERATION:
─────────

┌───────────────────────────────────────────────────────────────────┐
│                                                                   │
│   1. MCU sends 10µs trigger pulse                                 │
│                                                                   │
│      TRIG ─────┐     ┌─────────────────────────────────           │
│                └─────┘  10µs                                      │
│                                                                   │
│   2. Sensor sends 8 ultrasonic pulses at 40 kHz                   │
│                                                                   │
│   3. Sensor sets ECHO high while waiting for return               │
│                                                                   │
│      ECHO ─────────────┐                     ┌────────────        │
│                        └─────────────────────┘                    │
│                        │◄──── Echo pulse ────►│                   │
│                                 width                             │
│                                                                   │
│   4. Distance = (Echo pulse width × Speed of sound) / 2           │
│                                                                   │
│      Distance (cm) = Echo pulse (µs) / 58                         │
│      Distance (inch) = Echo pulse (µs) / 148                      │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘

CIRCUIT:
───────

    ┌───────────────────────────────────────────────────┐
    │                     8051                          │
    │                                                   │
    │    P3.2 ─────────────────────► TRIG               │
    │                                                   │
    │    P3.3 ◄───────────────────── ECHO               │
    │                                                   │
    │    Timer 0: Measure echo pulse width              │
    │                                                   │
    └───────────────────────────────────────────────────┘

            HC-SR04
    ┌─────────────────────┐
    │  ┌───┐     ┌───┐   │
    │  │ TX│     │ RX│   │
    │  └───┘     └───┘   │
    │                    │
    │ VCC GND TRIG ECHO  │
    └──┬───┬────┬────┬───┘
       │   │    │    │
      +5V GND  MCU  MCU
```

```c
/*
 * HC-SR04 ULTRASONIC DISTANCE SENSOR
 * Using Timer for pulse measurement
 */

#include <reg51.h>
#include <stdio.h>

sbit TRIG = P3^2;
sbit ECHO = P3^3;

unsigned int distance_cm;

/*
 * MEASURE DISTANCE
 * Returns distance in centimeters
 */
unsigned int measure_distance(void)
{
    unsigned int time_us;
    unsigned int distance;
    
    // Configure Timer 0 in Mode 1 (16-bit)
    TMOD = (TMOD & 0xF0) | 0x01;
    TH0 = 0;
    TL0 = 0;
    TF0 = 0;
    
    // Send 10µs trigger pulse
    TRIG = 1;
    delay_us(10);
    TRIG = 0;
    
    // Wait for echo to go high
    while(ECHO == 0);
    
    // Start timer
    TR0 = 1;
    
    // Wait for echo to go low (or timeout)
    while(ECHO == 1 && TF0 == 0);
    
    // Stop timer
    TR0 = 0;
    
    // Check for timeout
    if(TF0 == 1)
    {
        TF0 = 0;
        return 0;  // Out of range
    }
    
    // Calculate time in microseconds
    // At 11.0592 MHz, 1 timer tick = 1.085 µs
    time_us = ((unsigned int)TH0 << 8) | TL0;
    time_us = time_us * 1085 / 1000;  // Convert to µs
    
    // Calculate distance
    // Speed of sound = 343 m/s = 34.3 cm/ms = 0.0343 cm/µs
    // Distance = time × 0.0343 / 2 = time / 58.3
    distance = time_us / 58;
    
    return distance;
}

void main(void)
{
    char buf[16];
    
    lcd_init();
    TRIG = 0;
    
    lcd_string("Distance Meter");
    delay_ms(1000);
    lcd_cmd(0x01);
    
    while(1)
    {
        distance_cm = measure_distance();
        
        lcd_goto(0, 0);
        if(distance_cm == 0)
        {
            lcd_string("Out of Range!   ");
        }
        else
        {
            sprintf(buf, "Dist: %d cm     ", distance_cm);
            lcd_string(buf);
        }
        
        delay_ms(200);  // Min measurement interval
    }
}
```

---

## 3. Light and Proximity Sensors

### 3.1 LDR Light Sensor

```
LDR (LIGHT DEPENDENT RESISTOR) PROJECT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CIRCUIT (VOLTAGE DIVIDER):
────────────────────────

      +5V
       │
       │
    ┌──┴──┐
    │ LDR │   Resistance varies with light:
    │     │   Dark: ~1 MΩ
    └──┬──┘   Bright: ~1 kΩ
       │
       ├──────────► To ADC
       │
    ┌──┴──┐
    │ 10K │   Fixed resistor
    │     │
    └──┬──┘
       │
      GND

OUTPUT VOLTAGE:
─────────────
V_out = 5V × R_fixed / (R_fixed + R_LDR)

• In dark: V_out ≈ 0.05V (LDR = 1MΩ)
• In bright: V_out ≈ 4.5V (LDR = 1kΩ)
```

```c
/*
 * LDR LIGHT SENSOR WITH AUTO LAMP CONTROL
 */

#include <reg51.h>

sbit LAMP = P1^0;    // Relay control
sbit LED_STATUS = P1^1;

// Threshold values (0-255)
#define DARK_THRESHOLD   50   // Below this = dark
#define LIGHT_THRESHOLD  150  // Above this = bright

void main(void)
{
    unsigned char light_level;
    unsigned char lamp_state = 0;
    
    lcd_init();
    adc_init();
    
    LAMP = 0;  // Lamp off initially
    
    while(1)
    {
        // Read light level
        light_level = adc_read(0);  // Channel 0
        
        // Display light level
        lcd_goto(0, 0);
        lcd_string("Light: ");
        lcd_number(light_level);
        lcd_string("   ");
        
        // Hysteresis-based lamp control
        if(light_level < DARK_THRESHOLD && lamp_state == 0)
        {
            // It's dark, turn on lamp
            LAMP = 1;
            lamp_state = 1;
            lcd_goto(1, 0);
            lcd_string("Lamp: ON       ");
        }
        else if(light_level > LIGHT_THRESHOLD && lamp_state == 1)
        {
            // It's bright, turn off lamp
            LAMP = 0;
            lamp_state = 0;
            lcd_goto(1, 0);
            lcd_string("Lamp: OFF      ");
        }
        
        // Status LED follows lamp
        LED_STATUS = lamp_state;
        
        delay_ms(100);
    }
}
```

### 3.2 PIR Motion Sensor

```
PIR MOTION SENSOR PROJECT
━━━━━━━━━━━━━━━━━━━━━━━━━

HC-SR501 PIR SENSOR:
──────────────────

┌─────────────────────────────────────────┐
│              HC-SR501                   │
│                                         │
│   ┌───────────────────────────────┐    │
│   │                               │    │
│   │   Fresnel Lens (Dome cover)   │    │
│   │                               │    │
│   └───────────────────────────────┘    │
│                                         │
│   [Sensitivity]  [Time Delay]          │
│      Pot 1          Pot 2              │
│                                         │
│   Jumper: H = Repeatable trigger       │
│           L = Single trigger           │
│                                         │
│   VCC  OUT  GND                        │
│    │    │    │                         │
└────┼────┼────┼─────────────────────────┘
     │    │    │
    +5V  MCU  GND

SPECIFICATIONS:
• Detection range: 3-7 meters
• Detection angle: 120°
• Output: 3.3V high when motion detected
• Time delay: 3-300 seconds (adjustable)
```

```c
/*
 * PIR MOTION DETECTOR WITH ALARM
 */

#include <reg51.h>

sbit PIR_OUT = P3^2;    // External interrupt 0
sbit BUZZER = P1^0;
sbit LED_ALARM = P1^1;
sbit RELAY = P1^2;      // For light/siren

volatile bit motion_detected = 0;
volatile unsigned int alarm_timer = 0;

#define ALARM_DURATION 5000  // 5 seconds

/*
 * EXTERNAL INTERRUPT 0 ISR
 * Triggered on PIR motion detection
 */
void ext0_isr(void) interrupt 0
{
    motion_detected = 1;
    alarm_timer = ALARM_DURATION;
}

/*
 * TIMER 0 ISR - 1ms tick
 */
void timer0_isr(void) interrupt 1
{
    TH0 = 0xFC;  // Reload for 1ms at 11.0592 MHz
    TL0 = 0x66;
    
    if(alarm_timer > 0)
    {
        alarm_timer--;
    }
}

void main(void)
{
    // Initialize timer 0 for 1ms interrupts
    TMOD = 0x01;  // Mode 1
    TH0 = 0xFC;
    TL0 = 0x66;
    TR0 = 1;
    ET0 = 1;
    
    // Initialize external interrupt 0
    IT0 = 1;  // Edge triggered
    EX0 = 1;  // Enable
    
    // Enable global interrupts
    EA = 1;
    
    // Initialize outputs
    BUZZER = 0;
    LED_ALARM = 0;
    RELAY = 0;
    
    lcd_init();
    lcd_string("Motion Detector");
    lcd_goto(1, 0);
    lcd_string("Status: Ready");
    
    while(1)
    {
        if(motion_detected)
        {
            // Show alert
            lcd_goto(1, 0);
            lcd_string("MOTION DETECTED!");
            
            // Activate alarm
            BUZZER = 1;
            LED_ALARM = 1;
            RELAY = 1;
            
            motion_detected = 0;  // Clear flag
        }
        
        // Deactivate alarm after timeout
        if(alarm_timer == 0 && BUZZER == 1)
        {
            BUZZER = 0;
            LED_ALARM = 0;
            RELAY = 0;
            
            lcd_goto(1, 0);
            lcd_string("Status: Ready   ");
        }
        
        delay_ms(100);
    }
}
```

---

## 4. I2C Sensor Interface

### 4.1 BMP180 Pressure/Temperature Sensor

```
BMP180 BAROMETRIC PRESSURE SENSOR
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SPECIFICATIONS:
─────────────
• Pressure range: 300-1100 hPa
• Temperature range: -40°C to +85°C
• Interface: I2C (address 0x77)
• Resolution: Up to 0.01 hPa

I2C CONNECTION:
─────────────

    ┌─────────────────────────────────────────────────────┐
    │                    8051                             │
    │                                                     │
    │   P1.0 ──────┬──────────────────────► SCL          │
    │              │                                      │
    │           ┌──┴──┐                                  │
    │           │ 4.7K│ Pull-up                          │
    │           └──┬──┘                                  │
    │              │                                      │
    │             +3.3V                                   │
    │                                                     │
    │   P1.1 ──────┬──────────────────────► SDA          │
    │              │                                      │
    │           ┌──┴──┐                                  │
    │           │ 4.7K│ Pull-up                          │
    │           └──┬──┘                                  │
    │              │                                      │
    │             +3.3V                                   │
    │                                                     │
    └─────────────────────────────────────────────────────┘

BMP180 REGISTERS:
───────────────
0xAA-0xBE: Calibration data (read on startup)
0xF4:      Control register
0xF6-0xF8: Data registers

MEASUREMENT SEQUENCE:
───────────────────
1. Read calibration data (once at startup)
2. Start temperature measurement
3. Wait 4.5ms
4. Read temperature
5. Start pressure measurement
6. Wait 4.5-25.5ms (based on oversampling)
7. Read pressure
8. Calculate compensated values
```

```c
/*
 * SOFTWARE I2C FOR 8051
 */

sbit SCL = P1^0;
sbit SDA = P1^1;

#define BMP180_ADDR  0x77

void i2c_delay(void)
{
    unsigned char i;
    for(i = 0; i < 10; i++);
}

void i2c_start(void)
{
    SDA = 1;
    SCL = 1;
    i2c_delay();
    SDA = 0;
    i2c_delay();
    SCL = 0;
}

void i2c_stop(void)
{
    SDA = 0;
    SCL = 1;
    i2c_delay();
    SDA = 1;
    i2c_delay();
}

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
    
    // Get ACK
    SDA = 1;
    SCL = 1;
    i2c_delay();
    ack = SDA;
    SCL = 0;
    
    return ack;
}

unsigned char i2c_read(bit ack)
{
    unsigned char i, dat = 0;
    
    SDA = 1;  // Release SDA
    
    for(i = 0; i < 8; i++)
    {
        dat <<= 1;
        SCL = 1;
        i2c_delay();
        if(SDA) dat |= 1;
        SCL = 0;
        i2c_delay();
    }
    
    // Send ACK or NACK
    SDA = ack ? 0 : 1;
    SCL = 1;
    i2c_delay();
    SCL = 0;
    SDA = 1;
    
    return dat;
}

/*
 * BMP180 FUNCTIONS
 */

unsigned char bmp180_read_byte(unsigned char reg)
{
    unsigned char value;
    
    i2c_start();
    i2c_write(BMP180_ADDR << 1);      // Write mode
    i2c_write(reg);
    i2c_start();                       // Repeated start
    i2c_write((BMP180_ADDR << 1) | 1); // Read mode
    value = i2c_read(0);               // NACK after last byte
    i2c_stop();
    
    return value;
}

void bmp180_read_calibration(void)
{
    // Read calibration coefficients from 0xAA-0xBE
    // AC1, AC2, AC3, AC4, AC5, AC6, B1, B2, MB, MC, MD
    // (Implementation depends on specific needs)
}

long bmp180_read_temperature(void)
{
    long UT;
    
    // Start temperature measurement
    i2c_start();
    i2c_write(BMP180_ADDR << 1);
    i2c_write(0xF4);           // Control register
    i2c_write(0x2E);           // Temperature command
    i2c_stop();
    
    delay_ms(5);               // Wait for conversion
    
    // Read result
    UT = bmp180_read_byte(0xF6) << 8;
    UT |= bmp180_read_byte(0xF7);
    
    // Apply compensation (using calibration data)
    // Return temperature in 0.1°C units
    
    return UT;  // Simplified - add compensation in real code
}
```

---

## 5. Sensor Calibration

### 5.1 Calibration Techniques

```
SENSOR CALIBRATION METHODS
━━━━━━━━━━━━━━━━━━━━━━━━━━

1. SINGLE-POINT CALIBRATION (Offset):
───────────────────────────────────

    Actual
      ▲
      │      ┌─────── Calibrated (after offset)
      │     /
      │    /
      │   /
      │  /  ── Raw sensor
      │ /
      ├──────────────────► Measured
      
    calibrated = measured + offset

2. TWO-POINT CALIBRATION (Offset + Gain):
───────────────────────────────────────

    Actual
      ▲       ┌─────── Calibrated
      │      /
      │     /
      │    /
      │   /    Raw sensor (wrong slope)
      │  /   /
      │ /   /
      ├────────────────────► Measured
      
    calibrated = (measured - low_measured) × 
                 (high_actual - low_actual) /
                 (high_measured - low_measured) +
                 low_actual

3. MULTI-POINT CALIBRATION (Lookup Table):
────────────────────────────────────────

    For non-linear sensors, use lookup table with interpolation.
    
    Measured: |  0 |  50 | 100 | 150 | 200 | 250 |
    Actual:   | 10 |  55 | 105 | 160 | 210 | 255 |
```

```c
/*
 * TEMPERATURE SENSOR CALIBRATION
 */

// Calibration coefficients (determined during calibration)
#define OFFSET    -2.5    // Offset in degrees
#define GAIN      1.02    // Gain factor

// Lookup table for non-linear correction
code int cal_table[11] = {
    // Index = raw/25, Value = correction in 0.1°C
    0, 2, 5, 8, 10, 12, 15, 18, 22, 25, 28
};

/*
 * APPLY TWO-POINT CALIBRATION
 */
float calibrate_temperature(float raw_temp)
{
    float calibrated;
    
    // Apply gain and offset
    calibrated = (raw_temp * GAIN) + OFFSET;
    
    return calibrated;
}

/*
 * APPLY LOOKUP TABLE CALIBRATION
 */
int calibrate_with_table(int raw_value)
{
    int index;
    int correction;
    int fraction;
    
    // Find table index
    index = raw_value / 25;
    
    if(index >= 10)
        return raw_value + cal_table[10];
    
    // Linear interpolation between points
    fraction = raw_value % 25;
    correction = cal_table[index] + 
                 (cal_table[index + 1] - cal_table[index]) * 
                 fraction / 25;
    
    return raw_value + correction;
}

/*
 * CALIBRATION PROCEDURE
 * Called during manufacturing/setup
 */
void perform_calibration(void)
{
    int raw_low, raw_high;
    float actual_low = 0.0;    // Known reference: 0°C (ice water)
    float actual_high = 100.0; // Known reference: 100°C (boiling)
    
    lcd_string("Place in 0C ref");
    lcd_goto(1, 0);
    lcd_string("Press key...");
    wait_for_key();
    raw_low = read_sensor_average(10);
    
    lcd_cmd(0x01);
    lcd_string("Place in 100C ref");
    lcd_goto(1, 0);
    lcd_string("Press key...");
    wait_for_key();
    raw_high = read_sensor_average(10);
    
    // Calculate calibration coefficients
    float gain = (actual_high - actual_low) / (raw_high - raw_low);
    float offset = actual_low - (raw_low * gain);
    
    // Store in EEPROM
    eeprom_write_float(EEPROM_GAIN_ADDR, gain);
    eeprom_write_float(EEPROM_OFFSET_ADDR, offset);
    
    lcd_cmd(0x01);
    lcd_string("Calibration Done");
}
```

---

## 📋 Sensor Summary Table

| Sensor | Type | Interface | Range | Accuracy | Typical Use |
|--------|------|-----------|-------|----------|-------------|
| LM35 | Temperature | Analog | -55 to 150°C | ±0.5°C | General temp |
| DHT11 | Temp+Humidity | 1-Wire | 0-50°C, 20-90%RH | ±2°C, ±5% | Weather station |
| DS18B20 | Temperature | 1-Wire | -55 to 125°C | ±0.5°C | Precision temp |
| HC-SR04 | Distance | Pulse | 2-400 cm | ±3mm | Proximity |
| LDR | Light | Analog | - | - | Light detection |
| PIR | Motion | Digital | 3-7m | - | Security |
| BMP180 | Pressure | I2C | 300-1100 hPa | ±0.12 hPa | Altitude |
| MPU6050 | IMU | I2C | ±16g, ±2000°/s | - | Motion tracking |

---

## ❓ Quick Revision Questions

1. **How does the DHT11 encode data bits?**
   <details>
   <summary>Show Answer</summary>
   DHT11 uses pulse width encoding: 50µs low followed by 26-28µs high = '0', 50µs low followed by ~70µs high = '1'. Total 40 bits: 8 humidity int, 8 humidity dec, 8 temp int, 8 temp dec, 8 checksum.
   </details>

2. **How do you calculate distance from HC-SR04?**
   <details>
   <summary>Show Answer</summary>
   Distance = (Echo pulse width in µs) / 58 for cm, or / 148 for inches. This accounts for sound traveling to object and back (÷2) at 343 m/s speed.
   </details>

3. **Why use hysteresis in threshold-based sensing?**
   <details>
   <summary>Show Answer</summary>
   Hysteresis prevents rapid on/off oscillation when sensor value hovers near threshold. Use two thresholds: turn ON below lower threshold, turn OFF above higher threshold. Provides stable switching.
   </details>

4. **What is the purpose of pull-up resistors in I2C?**
   <details>
   <summary>Show Answer</summary>
   I2C uses open-drain outputs that can only pull lines LOW. Pull-ups (typically 4.7K) provide the HIGH state. Both SDA and SCL need pull-ups. Value affects maximum bus speed.
   </details>

5. **Why is calibration important for sensors?**
   <details>
   <summary>Show Answer</summary>
   Manufacturing tolerances cause offset and gain errors. Calibration corrects these using known references. Two-point calibration fixes both offset and gain. Improves accuracy to match specifications.
   </details>

6. **How does a PIR sensor detect motion?**
   <details>
   <summary>Show Answer</summary>
   PIR detects changes in infrared radiation from warm objects (humans, animals). Uses two sensor elements behind Fresnel lens. Motion causes different IR levels on elements, generating output pulse.
   </details>

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [10.1 Embedded System Design](01-embedded-system-design.md) | [Unit 10 Index](README.md) | [10.3 Motor Control Systems](03-motor-control-systems.md) |

---

*[← Previous: Embedded System Design](01-embedded-system-design.md) | [Next: Motor Control Systems →](03-motor-control-systems.md)*
