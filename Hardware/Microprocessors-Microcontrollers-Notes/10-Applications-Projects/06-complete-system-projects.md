# Chapter 10.6: Complete System Projects

## 📚 Chapter Overview

This chapter presents comprehensive capstone projects that integrate sensors, actuators, displays, and communication interfaces into complete embedded systems.

---

## 🎯 Learning Objectives

After completing this chapter, you will be able to:
- Design complete embedded system solutions
- Integrate multiple subsystems
- Implement real-world automation projects
- Apply proper software architecture
- Document and test embedded projects

---

## 1. Home Automation System

### 1.1 System Overview

```
HOME AUTOMATION CONTROLLER
━━━━━━━━━━━━━━━━━━━━━━━━━━

SYSTEM BLOCK DIAGRAM:
───────────────────

                    ┌─────────────────────────────────────────────────┐
                    │              HOME AUTOMATION SYSTEM              │
                    └─────────────────────────────────────────────────┘
                                          │
        ┌─────────────────────────────────┼─────────────────────────────────┐
        │                                 │                                 │
        ▼                                 ▼                                 ▼
┌───────────────┐               ┌───────────────┐               ┌───────────────┐
│   SENSORS     │               │  CONTROLLER   │               │   OUTPUTS     │
│               │               │    (8051)     │               │               │
│ • Temperature │──────────────►│               │──────────────►│ • Relays (4)  │
│ • Humidity    │               │ • ADC         │               │ • LEDs        │
│ • PIR Motion  │               │ • Timer/PWM   │               │ • Buzzer      │
│ • LDR Light   │               │ • I2C/SPI     │               │ • Fan (PWM)   │
│ • Gas Sensor  │               │ • UART        │               │ • Motor       │
│ • Door Switch │               │               │               │               │
└───────────────┘               └───────┬───────┘               └───────────────┘
                                        │
                    ┌───────────────────┼───────────────────┐
                    │                   │                   │
                    ▼                   ▼                   ▼
            ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
            │   DISPLAY     │   │    STORAGE    │   │ COMMUNICATION │
            │               │   │               │   │               │
            │ • LCD 20×4    │   │ • EEPROM      │   │ • Bluetooth   │
            │ • Status LEDs │   │ • RTC DS1307  │   │ • WiFi        │
            │ • Keypad      │   │ • SD Card     │   │ • IR Remote   │
            └───────────────┘   └───────────────┘   └───────────────┘

FEATURES:
────────
• 4-channel relay control (lights, fan, AC, appliance)
• Temperature-based fan speed control
• Motion-activated lighting
• Scheduled operations (RTC-based)
• Remote control via Bluetooth/WiFi
• Event logging to EEPROM/SD card
• Gas leak alarm
• Door open notification
```

### 1.2 Hardware Connections

```
CIRCUIT CONNECTIONS
━━━━━━━━━━━━━━━━━━━

              AT89S52/8051 MCU
         ┌───────────────────────┐
         │                       │
    P0.0 │◄── ADC0804 D0        │ P2.0 ──► Relay 1 (via ULN2003)
    P0.1 │◄── ADC0804 D1        │ P2.1 ──► Relay 2
    P0.2 │◄── ADC0804 D2        │ P2.2 ──► Relay 3
    P0.3 │◄── ADC0804 D3        │ P2.3 ──► Relay 4
    P0.4 │◄── ADC0804 D4        │ P2.4 ──► Buzzer
    P0.5 │◄── ADC0804 D5        │ P2.5 ──► Status LED
    P0.6 │◄── ADC0804 D6        │ P2.6 ──► I2C SDA (RTC/EEPROM)
    P0.7 │◄── ADC0804 D7        │ P2.7 ──► I2C SCL
         │                       │
    P1.0 │◄── PIR Sensor        │ P3.0 ──► UART TXD (Bluetooth)
    P1.1 │◄── Door Switch       │ P3.1 ◄── UART RXD
    P1.2 │◄── Gas Sensor (dig)  │ P3.2 ◄── INT0 (IR Receiver)
    P1.3 │──► ADC MUX A         │ P3.3 ◄── INT1 (PIR Interrupt)
    P1.4 │──► ADC MUX B         │ P3.4 ──► Fan PWM
    P1.5 │──► ADC RD            │ P3.5 ──► ADC WR
    P1.6 │──► ADC INTR          │ P3.6 ──► LCD RS
    P1.7 │◄── Keypad Row        │ P3.7 ──► LCD EN
         │                       │
         └───────────────────────┘

ANALOG MUX CHANNELS:
──────────────────
CH0: LM35 Temperature Sensor
CH1: Humidity Sensor
CH2: LDR Light Sensor
CH3: MQ-2 Gas Sensor (analog)
```

### 1.3 Complete System Code

```c
/*
 * HOME AUTOMATION SYSTEM
 * Complete implementation
 */

#include <reg51.h>
#include <stdio.h>
#include <string.h>

//=== Hardware Definitions ===
// Relays
sbit RELAY1 = P2^0;  // Living room light
sbit RELAY2 = P2^1;  // Bedroom light
sbit RELAY3 = P2^2;  // Kitchen light
sbit RELAY4 = P2^3;  // External appliance
sbit BUZZER = P2^4;
sbit STATUS_LED = P2^5;

// Sensors
sbit PIR_SENSOR = P1^0;
sbit DOOR_SWITCH = P1^1;
sbit GAS_DIGITAL = P1^2;

// ADC Control
sbit ADC_RD = P1^5;
sbit ADC_WR = P3^5;
sbit ADC_INTR = P1^6;
sbit MUX_A = P1^3;
sbit MUX_B = P1^4;

#define ADC_DATA P0

//=== System State ===
typedef struct {
    // Sensor readings
    int temperature;
    int humidity;
    int light_level;
    int gas_level;
    bit pir_detected;
    bit door_open;
    bit gas_alarm;
    
    // Relay states
    bit relay1_state;
    bit relay2_state;
    bit relay3_state;
    bit relay4_state;
    
    // Settings
    unsigned char temp_threshold;
    unsigned char light_threshold;
    unsigned char gas_threshold;
    bit auto_light_mode;
    bit auto_fan_mode;
    
    // Fan
    unsigned char fan_speed;
    
    // Time
    unsigned char hours;
    unsigned char minutes;
    unsigned char seconds;
} SystemState;

SystemState sys;

//=== ADC Functions ===
unsigned char read_adc(unsigned char channel)
{
    unsigned char value;
    
    // Set MUX channel
    MUX_A = channel & 0x01;
    MUX_B = (channel >> 1) & 0x01;
    
    // Start conversion
    ADC_WR = 0;
    ADC_WR = 1;
    
    // Wait for conversion
    while(ADC_INTR);
    
    // Read result
    ADC_RD = 0;
    value = ADC_DATA;
    ADC_RD = 1;
    
    return value;
}

//=== Sensor Reading Functions ===
void read_sensors(void)
{
    // Temperature (LM35: 10mV/°C)
    // ADC: 5V/256 = 19.5mV per step
    // Temp = ADC * 19.5 / 10 ≈ ADC * 2
    sys.temperature = read_adc(0) * 2;
    
    // Humidity (0-100%)
    sys.humidity = read_adc(1) * 100 / 255;
    
    // Light (0-100%, inverted for LDR)
    sys.light_level = 100 - (read_adc(2) * 100 / 255);
    
    // Gas level (0-100%)
    sys.gas_level = read_adc(3) * 100 / 255;
    
    // Digital sensors
    sys.pir_detected = PIR_SENSOR;
    sys.door_open = !DOOR_SWITCH;  // Active low
    sys.gas_alarm = (sys.gas_level > sys.gas_threshold) || !GAS_DIGITAL;
}

//=== Control Functions ===
void set_relay(unsigned char num, bit state)
{
    switch(num)
    {
        case 1: 
            RELAY1 = state; 
            sys.relay1_state = state;
            break;
        case 2: 
            RELAY2 = state; 
            sys.relay2_state = state;
            break;
        case 3: 
            RELAY3 = state; 
            sys.relay3_state = state;
            break;
        case 4: 
            RELAY4 = state; 
            sys.relay4_state = state;
            break;
    }
    
    log_event(num, state);  // Log to EEPROM
}

void toggle_relay(unsigned char num)
{
    switch(num)
    {
        case 1: set_relay(1, !sys.relay1_state); break;
        case 2: set_relay(2, !sys.relay2_state); break;
        case 3: set_relay(3, !sys.relay3_state); break;
        case 4: set_relay(4, !sys.relay4_state); break;
    }
}

void set_fan_speed(unsigned char speed)
{
    if(speed > 100) speed = 100;
    sys.fan_speed = speed;
    pwm_set_duty(speed);
}

//=== Automation Logic ===
void auto_control(void)
{
    // Auto light control based on motion and ambient light
    if(sys.auto_light_mode)
    {
        if(sys.pir_detected && sys.light_level < sys.light_threshold)
        {
            set_relay(1, 1);  // Turn on light
            // Auto-off timer could be added
        }
    }
    
    // Auto fan control based on temperature
    if(sys.auto_fan_mode)
    {
        if(sys.temperature > sys.temp_threshold + 5)
        {
            set_fan_speed(100);
        }
        else if(sys.temperature > sys.temp_threshold)
        {
            // Proportional control
            unsigned char speed = (sys.temperature - sys.temp_threshold) * 20;
            set_fan_speed(speed);
        }
        else
        {
            set_fan_speed(0);
        }
    }
    
    // Gas alarm
    if(sys.gas_alarm)
    {
        BUZZER = 1;
        STATUS_LED = 1;
        set_relay(4, 0);  // Turn off appliance
        send_alarm("GAS LEAK DETECTED!");
    }
    else
    {
        BUZZER = 0;
    }
    
    // Door notification
    if(sys.door_open)
    {
        send_notification("Door opened");
    }
}

//=== Display Functions ===
void update_display(void)
{
    char buf[21];
    
    lcd_goto(0, 0);
    sprintf(buf, "T:%02dC H:%02d%% L:%02d%%", 
            sys.temperature, sys.humidity, sys.light_level);
    lcd_string(buf);
    
    lcd_goto(1, 0);
    sprintf(buf, "R1:%d R2:%d R3:%d R4:%d", 
            sys.relay1_state, sys.relay2_state,
            sys.relay3_state, sys.relay4_state);
    lcd_string(buf);
    
    lcd_goto(2, 0);
    sprintf(buf, "Fan:%3d%% Gas:%02d%%  ", 
            sys.fan_speed, sys.gas_level);
    lcd_string(buf);
    
    lcd_goto(3, 0);
    sprintf(buf, "%02d:%02d:%02d  ", 
            sys.hours, sys.minutes, sys.seconds);
    lcd_string(buf);
    
    if(sys.pir_detected) lcd_string("PIR ");
    if(sys.door_open) lcd_string("DOOR");
}

//=== Bluetooth Command Handler ===
void handle_bluetooth_command(char *cmd)
{
    // Command format: "R1ON", "R1OFF", "FAN50", "AUTO1", etc.
    
    if(strncmp(cmd, "R1ON", 4) == 0)
        set_relay(1, 1);
    else if(strncmp(cmd, "R1OFF", 5) == 0)
        set_relay(1, 0);
    else if(strncmp(cmd, "R2ON", 4) == 0)
        set_relay(2, 1);
    else if(strncmp(cmd, "R2OFF", 5) == 0)
        set_relay(2, 0);
    else if(strncmp(cmd, "R3ON", 4) == 0)
        set_relay(3, 1);
    else if(strncmp(cmd, "R3OFF", 5) == 0)
        set_relay(3, 0);
    else if(strncmp(cmd, "R4ON", 4) == 0)
        set_relay(4, 1);
    else if(strncmp(cmd, "R4OFF", 5) == 0)
        set_relay(4, 0);
    else if(strncmp(cmd, "FAN", 3) == 0)
    {
        unsigned char speed = atoi(&cmd[3]);
        set_fan_speed(speed);
    }
    else if(strncmp(cmd, "STATUS", 6) == 0)
    {
        send_status();
    }
    else if(strncmp(cmd, "AUTO", 4) == 0)
    {
        sys.auto_light_mode = (cmd[4] == '1');
    }
}

void send_status(void)
{
    char buf[64];
    
    sprintf(buf, "T:%d,H:%d,L:%d,G:%d,R:%d%d%d%d,F:%d\r\n",
            sys.temperature, sys.humidity, sys.light_level,
            sys.gas_level, sys.relay1_state, sys.relay2_state,
            sys.relay3_state, sys.relay4_state, sys.fan_speed);
    
    uart_tx_string(buf);
}

//=== EEPROM Logging ===
unsigned int log_address = 0;

void log_event(unsigned char event_type, unsigned char value)
{
    unsigned char log_data[8];
    
    log_data[0] = sys.hours;
    log_data[1] = sys.minutes;
    log_data[2] = sys.seconds;
    log_data[3] = event_type;
    log_data[4] = value;
    log_data[5] = sys.temperature;
    log_data[6] = 0;  // Reserved
    log_data[7] = 0;
    
    eeprom_write_page(log_address, log_data, 8);
    log_address += 8;
    
    if(log_address >= 4000)  // Wrap around
        log_address = 0;
}

//=== Keypad Handler ===
void handle_keypad(void)
{
    unsigned char key = keypad_scan();
    
    switch(key)
    {
        case '1': toggle_relay(1); break;
        case '2': toggle_relay(2); break;
        case '3': toggle_relay(3); break;
        case '4': toggle_relay(4); break;
        case '5': set_fan_speed(0); break;
        case '6': set_fan_speed(50); break;
        case '7': set_fan_speed(100); break;
        case 'A': sys.auto_light_mode = !sys.auto_light_mode; break;
        case 'B': sys.auto_fan_mode = !sys.auto_fan_mode; break;
        case '*': enter_settings_menu(); break;
        case '#': show_log(); break;
    }
}

//=== Initialization ===
void system_init(void)
{
    // Initialize peripherals
    lcd_init();
    keypad_init();
    uart_init();
    i2c_init();
    pwm_init();
    
    // Load settings from EEPROM
    sys.temp_threshold = eeprom_read_byte(0x1000);
    sys.light_threshold = eeprom_read_byte(0x1001);
    sys.gas_threshold = eeprom_read_byte(0x1002);
    sys.auto_light_mode = eeprom_read_byte(0x1003);
    sys.auto_fan_mode = eeprom_read_byte(0x1004);
    
    // Default values if EEPROM empty
    if(sys.temp_threshold == 0xFF) sys.temp_threshold = 25;
    if(sys.light_threshold == 0xFF) sys.light_threshold = 30;
    if(sys.gas_threshold == 0xFF) sys.gas_threshold = 50;
    
    // Initialize outputs
    RELAY1 = 0; RELAY2 = 0; RELAY3 = 0; RELAY4 = 0;
    BUZZER = 0;
    set_fan_speed(0);
    
    // Read RTC
    rtc_read_time();
    
    lcd_string("Home Automation");
    lcd_goto(1, 0);
    lcd_string("System Starting...");
    delay_ms(2000);
    lcd_clear();
}

//=== Main Loop ===
void main(void)
{
    char bt_buffer[32];
    unsigned char bt_idx = 0;
    
    system_init();
    
    while(1)
    {
        // Read all sensors
        read_sensors();
        
        // Read RTC
        rtc_read_time();
        sys.hours = rtc_get_hours();
        sys.minutes = rtc_get_minutes();
        sys.seconds = rtc_get_seconds();
        
        // Execute automation logic
        auto_control();
        
        // Handle local input
        handle_keypad();
        
        // Handle Bluetooth commands
        if(uart_rx_available())
        {
            char ch = uart_rx_read();
            
            if(ch == '\r' || ch == '\n')
            {
                bt_buffer[bt_idx] = '\0';
                handle_bluetooth_command(bt_buffer);
                bt_idx = 0;
            }
            else if(bt_idx < 30)
            {
                bt_buffer[bt_idx++] = ch;
            }
        }
        
        // Update display
        update_display();
        
        // Status LED blink
        STATUS_LED = !STATUS_LED;
        
        delay_ms(200);
    }
}
```

---

## 2. Environmental Data Logger

### 2.1 System Design

```
ENVIRONMENTAL DATA LOGGER
━━━━━━━━━━━━━━━━━━━━━━━━━

FEATURES:
────────
• Multi-channel sensor logging
• SD card storage (FAT16)
• Real-time clock timestamping
• LCD display with graphs
• USB data transfer
• Alarm thresholds
• Battery backup

SYSTEM ARCHITECTURE:
──────────────────

    ┌─────────────────────────────────────────────────────────────┐
    │                    DATA LOGGER SYSTEM                        │
    └─────────────────────────────────────────────────────────────┘
                                │
    ┌───────────────────────────┼───────────────────────────────┐
    │                           │                               │
    ▼                           ▼                               ▼
┌──────────────┐        ┌──────────────┐              ┌──────────────┐
│  SENSORS     │        │  PROCESSOR   │              │   STORAGE    │
│              │        │              │              │              │
│ DHT22 T/H    │◄──────►│   8051       │◄────────────►│ SD Card      │
│ BMP180 Press │        │              │              │ 24C256 EEPROM│
│ TSL2561 Light│        │ • I2C Master │              │              │
│ MQ-135 Air   │        │ • SPI Master │              │              │
└──────────────┘        │ • ADC        │              └──────────────┘
                        │ • Timer      │
                        │ • UART       │
                        └──────┬───────┘
                               │
           ┌───────────────────┼───────────────────┐
           │                   │                   │
           ▼                   ▼                   ▼
    ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
    │   DISPLAY    │   │     RTC      │   │    OUTPUT    │
    │              │   │              │   │              │
    │ 128×64 OLED  │   │   DS3231     │   │ • UART/USB   │
    │ Status LEDs  │   │   Battery    │   │ • Buzzer     │
    │              │   │   Backup     │   │ • Alarm LED  │
    └──────────────┘   └──────────────┘   └──────────────┘

DATA FORMAT (CSV):
────────────────

Date,Time,Temp(C),Humidity(%),Pressure(hPa),Light(lux),AirQ
2024-01-15,10:30:00,23.5,45,1013.25,850,Good
2024-01-15,10:31:00,23.6,44,1013.20,855,Good
...
```

### 2.2 Data Logger Code

```c
/*
 * ENVIRONMENTAL DATA LOGGER
 * Complete implementation
 */

#include <reg51.h>
#include <stdio.h>

//=== Data Structure ===
typedef struct {
    unsigned char year;
    unsigned char month;
    unsigned char day;
    unsigned char hour;
    unsigned char minute;
    unsigned char second;
    int temperature;      // ×10 for 0.1°C resolution
    unsigned char humidity;
    unsigned int pressure; // hPa
    unsigned int light;    // lux
    unsigned char air_quality;
} LogRecord;

// Configuration
typedef struct {
    unsigned int log_interval;    // seconds
    int temp_min, temp_max;
    unsigned char humid_min, humid_max;
    bit enable_alarm;
    bit enable_display;
} LogConfig;

LogConfig config = {60, 150, 350, 30, 70, 1, 1};

// Statistics
typedef struct {
    int temp_min, temp_max;
    long temp_sum;
    unsigned int count;
} LogStats;

LogStats stats;

//=== SD Card File System ===
unsigned long file_position = 0;
unsigned char sector_buffer[512];

void create_log_file(void)
{
    char filename[13];
    LogRecord rec;
    
    // Get current date for filename
    rtc_read(&rec);
    sprintf(filename, "%02d%02d%02d.CSV", 
            rec.year, rec.month, rec.day);
    
    // Create file if not exists
    if(!sd_file_exists(filename))
    {
        sd_create_file(filename);
        
        // Write header
        sd_write_string(filename, 
            "Date,Time,Temp(C),Humidity(%),Pressure(hPa),Light(lux),AirQ\r\n");
    }
}

void log_to_sd(LogRecord *rec)
{
    char buf[80];
    char filename[13];
    
    sprintf(filename, "%02d%02d%02d.CSV",
            rec->year, rec->month, rec->day);
    
    sprintf(buf, "20%02d-%02d-%02d,%02d:%02d:%02d,%d.%d,%d,%d,%d,%s\r\n",
            rec->year, rec->month, rec->day,
            rec->hour, rec->minute, rec->second,
            rec->temperature / 10, abs(rec->temperature % 10),
            rec->humidity, rec->pressure, rec->light,
            get_air_quality_string(rec->air_quality));
    
    sd_append_string(filename, buf);
}

//=== Sensor Reading ===
void read_all_sensors(LogRecord *rec)
{
    // DHT22 Temperature and Humidity
    dht22_read(&rec->temperature, &rec->humidity);
    
    // BMP180 Pressure
    rec->pressure = bmp180_read_pressure();
    
    // TSL2561 Light (I2C)
    rec->light = tsl2561_read_lux();
    
    // MQ-135 Air Quality
    unsigned char adc_val = adc_read(0);
    rec->air_quality = classify_air_quality(adc_val);
    
    // Timestamp from RTC
    rtc_read(rec);
}

char* get_air_quality_string(unsigned char level)
{
    switch(level)
    {
        case 0: return "Good";
        case 1: return "Fair";
        case 2: return "Poor";
        case 3: return "Bad";
        default: return "Unknown";
    }
}

unsigned char classify_air_quality(unsigned char adc)
{
    if(adc < 50) return 0;      // Good
    else if(adc < 100) return 1; // Fair
    else if(adc < 150) return 2; // Poor
    else return 3;               // Bad
}

//=== Display Functions ===
void display_current(LogRecord *rec)
{
    char buf[17];
    
    oled_clear();
    
    // Line 1: Date and Time
    sprintf(buf, "%02d/%02d %02d:%02d:%02d",
            rec->day, rec->month, 
            rec->hour, rec->minute, rec->second);
    oled_puts(0, 0, buf);
    
    // Line 2: Temperature
    sprintf(buf, "Temp: %d.%dC", 
            rec->temperature / 10, 
            abs(rec->temperature % 10));
    oled_puts(0, 12, buf);
    
    // Line 3: Humidity
    sprintf(buf, "Humid: %d%%", rec->humidity);
    oled_puts(0, 24, buf);
    
    // Line 4: Pressure
    sprintf(buf, "Press: %dhPa", rec->pressure);
    oled_puts(0, 36, buf);
    
    // Line 5: Light and Air
    sprintf(buf, "L:%dlx A:%s", 
            rec->light, 
            get_air_quality_string(rec->air_quality));
    oled_puts(0, 48, buf);
    
    oled_display();
}

void display_graph(void)
{
    // Display last 24 hours temperature graph
    unsigned char i;
    LogRecord rec;
    unsigned char y;
    
    oled_clear();
    oled_puts(0, 0, "24h Temperature");
    
    // Draw axes
    oled_line(10, 15, 10, 55);   // Y axis
    oled_line(10, 55, 120, 55);  // X axis
    
    // Plot data points (simplified)
    for(i = 0; i < 24; i++)
    {
        // Read from EEPROM (hourly averages)
        int temp = eeprom_read_byte(0x100 + i * 2);
        temp |= eeprom_read_byte(0x101 + i * 2) << 8;
        
        // Scale to display
        // Assuming 0-50°C range mapped to 40 pixels
        y = 55 - (temp * 40 / 500);
        
        oled_set_pixel(15 + i * 4, y, 1);
        oled_set_pixel(16 + i * 4, y, 1);
    }
    
    oled_display();
}

//=== Alarm Functions ===
void check_alarms(LogRecord *rec)
{
    bit alarm = 0;
    
    if(rec->temperature < config.temp_min * 10 ||
       rec->temperature > config.temp_max * 10)
    {
        alarm = 1;
        uart_tx_string("ALARM: Temperature out of range!\r\n");
    }
    
    if(rec->humidity < config.humid_min ||
       rec->humidity > config.humid_max)
    {
        alarm = 1;
        uart_tx_string("ALARM: Humidity out of range!\r\n");
    }
    
    if(rec->air_quality >= 3)
    {
        alarm = 1;
        uart_tx_string("ALARM: Poor air quality!\r\n");
    }
    
    if(alarm && config.enable_alarm)
    {
        BUZZER = 1;
        ALARM_LED = 1;
        delay_ms(500);
        BUZZER = 0;
    }
    else
    {
        ALARM_LED = 0;
    }
}

//=== Statistics ===
void update_statistics(LogRecord *rec)
{
    if(rec->temperature < stats.temp_min)
        stats.temp_min = rec->temperature;
    if(rec->temperature > stats.temp_max)
        stats.temp_max = rec->temperature;
    
    stats.temp_sum += rec->temperature;
    stats.count++;
}

void display_statistics(void)
{
    char buf[32];
    int avg = stats.temp_sum / stats.count;
    
    oled_clear();
    oled_puts(0, 0, "Statistics");
    
    sprintf(buf, "Min: %d.%dC", 
            stats.temp_min / 10, abs(stats.temp_min % 10));
    oled_puts(0, 16, buf);
    
    sprintf(buf, "Max: %d.%dC", 
            stats.temp_max / 10, abs(stats.temp_max % 10));
    oled_puts(0, 28, buf);
    
    sprintf(buf, "Avg: %d.%dC", 
            avg / 10, abs(avg % 10));
    oled_puts(0, 40, buf);
    
    sprintf(buf, "Count: %d", stats.count);
    oled_puts(0, 52, buf);
    
    oled_display();
}

//=== Serial Commands ===
void handle_serial_command(char *cmd)
{
    if(strcmp(cmd, "STATUS") == 0)
    {
        LogRecord rec;
        read_all_sensors(&rec);
        
        char buf[80];
        sprintf(buf, "T:%d.%d,H:%d,P:%d,L:%d,A:%d\r\n",
                rec.temperature / 10, abs(rec.temperature % 10),
                rec.humidity, rec.pressure, rec.light,
                rec.air_quality);
        uart_tx_string(buf);
    }
    else if(strcmp(cmd, "DOWNLOAD") == 0)
    {
        // Send all logged data
        download_log();
    }
    else if(strncmp(cmd, "INTERVAL=", 9) == 0)
    {
        config.log_interval = atoi(&cmd[9]);
        uart_tx_string("OK\r\n");
    }
    else if(strcmp(cmd, "STATS") == 0)
    {
        display_statistics();
    }
}

//=== Main Function ===
void main(void)
{
    LogRecord current;
    unsigned int log_timer = 0;
    unsigned char display_mode = 0;
    
    // Initialize
    oled_init();
    i2c_init();
    spi_init();
    uart_init();
    rtc_init();
    sd_init();
    
    // Initialize statistics
    stats.temp_min = 9999;
    stats.temp_max = -9999;
    stats.temp_sum = 0;
    stats.count = 0;
    
    // Create log file
    create_log_file();
    
    oled_puts(0, 0, "Data Logger Ready");
    oled_display();
    delay_ms(2000);
    
    while(1)
    {
        // Read sensors
        read_all_sensors(&current);
        
        // Check alarms
        check_alarms(&current);
        
        // Update display
        if(config.enable_display)
        {
            switch(display_mode)
            {
                case 0: display_current(&current); break;
                case 1: display_graph(); break;
                case 2: display_statistics(); break;
            }
        }
        
        // Log at interval
        log_timer++;
        if(log_timer >= config.log_interval)
        {
            log_timer = 0;
            log_to_sd(&current);
            update_statistics(&current);
            STATUS_LED = !STATUS_LED;
        }
        
        // Handle keypad
        unsigned char key = keypad_scan();
        if(key == '*')
        {
            display_mode = (display_mode + 1) % 3;
        }
        
        // Handle serial commands
        if(uart_rx_available())
        {
            static char cmd_buf[32];
            static unsigned char cmd_idx = 0;
            
            char ch = uart_rx_read();
            
            if(ch == '\r' || ch == '\n')
            {
                cmd_buf[cmd_idx] = '\0';
                handle_serial_command(cmd_buf);
                cmd_idx = 0;
            }
            else if(cmd_idx < 30)
            {
                cmd_buf[cmd_idx++] = ch;
            }
        }
        
        delay_ms(1000);
    }
}
```

---

## 3. Line Following Robot

### 3.1 Robot Design

```
LINE FOLLOWING ROBOT
━━━━━━━━━━━━━━━━━━━━

ROBOT ARCHITECTURE:
─────────────────

             Front
        ┌─────────────┐
        │   IR   IR   │  IR Sensor Array
        │   S1   S2   │  (5 sensors)
        │ S0  S   S3  │
        │     2       │
        └──────┬──────┘
               │
        ┌──────┴──────┐
        │             │
        │    8051     │
        │  Controller │
        │             │
        └──────┬──────┘
               │
        ┌──────┴──────┐
        │   L293D     │
        │   Motor     │
        │   Driver    │
        └──────┬──────┘
               │
       ┌───────┴───────┐
       │               │
    ┌──┴──┐         ┌──┴──┐
    │Motor│         │Motor│
    │ Left│         │Right│
    └─────┘         └─────┘

SENSOR ARRAY CONFIGURATION:
─────────────────────────

    S0    S1    S2    S3    S4
    │     │     │     │     │
    ▼     ▼     ▼     ▼     ▼
  ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐
  │ IR│ │ IR│ │ IR│ │ IR│ │ IR│
  └───┘ └───┘ └───┘ └───┘ └───┘
    │     │     │     │     │
────┴─────┴─────┴─────┴─────┴────  Black Line
████████████████████████████████

Sensor Output: 1 = White (no line)
               0 = Black (on line)

DECISION LOGIC:
─────────────

Pattern      │ Action
─────────────┼─────────────────
0 0 0 0 0    │ All black - crossing/junction
0 0 1 0 0    │ Center - go straight
0 1 1 0 0    │ Slight left - soft right
0 0 1 1 0    │ Slight right - soft left
0 1 0 0 0    │ Left - turn right
0 0 0 1 0    │ Right - turn left
1 0 0 0 0    │ Far left - sharp right
0 0 0 0 1    │ Far right - sharp left
1 1 1 1 1    │ All white - lost line
```

### 3.2 Line Follower Code

```c
/*
 * PID LINE FOLLOWING ROBOT
 * 5-sensor array with PWM motor control
 */

#include <reg51.h>

//=== Hardware Definitions ===
// Sensor inputs (active low - 0 on black)
sbit SENSOR_0 = P1^0;  // Leftmost
sbit SENSOR_1 = P1^1;
sbit SENSOR_2 = P1^2;  // Center
sbit SENSOR_3 = P1^3;
sbit SENSOR_4 = P1^4;  // Rightmost

// Motor control
sbit MOTOR_L_EN = P2^0;
sbit MOTOR_L_IN1 = P2^1;
sbit MOTOR_L_IN2 = P2^2;
sbit MOTOR_R_EN = P2^3;
sbit MOTOR_R_IN1 = P2^4;
sbit MOTOR_R_IN2 = P2^5;

//=== PID Constants ===
float Kp = 20.0;
float Ki = 0.0;
float Kd = 15.0;

//=== State Variables ===
int last_error = 0;
float integral = 0;

// Base speed (0-100)
#define BASE_SPEED 60
#define MAX_SPEED  100
#define MIN_SPEED  0

// PWM variables
unsigned char pwm_left = 0;
unsigned char pwm_right = 0;
volatile unsigned char pwm_counter = 0;

//=== Timer ISR for PWM ===
void timer0_isr(void) interrupt 1
{
    TH0 = 0xFF;
    TL0 = 0xA4;
    
    pwm_counter++;
    if(pwm_counter >= 100)
        pwm_counter = 0;
    
    MOTOR_L_EN = (pwm_counter < pwm_left) ? 1 : 0;
    MOTOR_R_EN = (pwm_counter < pwm_right) ? 1 : 0;
}

//=== Read Sensor Array ===
unsigned char read_sensors(void)
{
    unsigned char sensors = 0;
    
    if(!SENSOR_0) sensors |= 0x10;  // Bit 4
    if(!SENSOR_1) sensors |= 0x08;  // Bit 3
    if(!SENSOR_2) sensors |= 0x04;  // Bit 2
    if(!SENSOR_3) sensors |= 0x02;  // Bit 1
    if(!SENSOR_4) sensors |= 0x01;  // Bit 0
    
    return sensors;
}

//=== Calculate Position Error ===
int calculate_error(unsigned char sensors)
{
    /*
     * Error calculation based on sensor weights:
     * S0 = -4, S1 = -2, S2 = 0, S3 = +2, S4 = +4
     * 
     * Error = weighted sum / number of sensors on line
     */
    
    int position = 0;
    int count = 0;
    
    if(sensors & 0x10) { position += -4; count++; }
    if(sensors & 0x08) { position += -2; count++; }
    if(sensors & 0x04) { position +=  0; count++; }
    if(sensors & 0x02) { position += +2; count++; }
    if(sensors & 0x01) { position += +4; count++; }
    
    if(count == 0)
    {
        // No sensors on line - use last known error
        return last_error > 0 ? 4 : -4;
    }
    
    return (position * 10) / count;
}

//=== PID Controller ===
int pid_compute(int error)
{
    float P, I, D;
    int output;
    
    // Proportional
    P = Kp * error;
    
    // Integral
    integral += error;
    if(integral > 100) integral = 100;
    if(integral < -100) integral = -100;
    I = Ki * integral;
    
    // Derivative
    D = Kd * (error - last_error);
    last_error = error;
    
    // Output
    output = (int)(P + I + D);
    
    return output;
}

//=== Motor Control ===
void set_motors(int left_speed, int right_speed)
{
    // Limit speeds
    if(left_speed > MAX_SPEED) left_speed = MAX_SPEED;
    if(left_speed < -MAX_SPEED) left_speed = -MAX_SPEED;
    if(right_speed > MAX_SPEED) right_speed = MAX_SPEED;
    if(right_speed < -MAX_SPEED) right_speed = -MAX_SPEED;
    
    // Left motor
    if(left_speed >= 0)
    {
        MOTOR_L_IN1 = 1;
        MOTOR_L_IN2 = 0;
        pwm_left = left_speed;
    }
    else
    {
        MOTOR_L_IN1 = 0;
        MOTOR_L_IN2 = 1;
        pwm_left = -left_speed;
    }
    
    // Right motor
    if(right_speed >= 0)
    {
        MOTOR_R_IN1 = 1;
        MOTOR_R_IN2 = 0;
        pwm_right = right_speed;
    }
    else
    {
        MOTOR_R_IN1 = 0;
        MOTOR_R_IN2 = 1;
        pwm_right = -right_speed;
    }
}

void motors_stop(void)
{
    MOTOR_L_IN1 = 0;
    MOTOR_L_IN2 = 0;
    MOTOR_R_IN1 = 0;
    MOTOR_R_IN2 = 0;
    pwm_left = 0;
    pwm_right = 0;
}

//=== Main Function ===
void main(void)
{
    unsigned char sensors;
    int error, correction;
    int left_speed, right_speed;
    
    // Initialize Timer 0 for PWM
    TMOD = 0x01;
    TH0 = 0xFF;
    TL0 = 0xA4;
    TR0 = 1;
    ET0 = 1;
    EA = 1;
    
    // Initialize motors forward
    MOTOR_L_IN1 = 1;
    MOTOR_L_IN2 = 0;
    MOTOR_R_IN1 = 1;
    MOTOR_R_IN2 = 0;
    
    // Wait for start button
    while(START_BUTTON);
    delay_ms(500);
    
    while(1)
    {
        // Read sensors
        sensors = read_sensors();
        
        // Check for special cases
        if(sensors == 0x1F)  // All on line (crossing)
        {
            // Handle intersection
            set_motors(BASE_SPEED, BASE_SPEED);
            continue;
        }
        
        if(sensors == 0x00)  // All off line (lost)
        {
            // Search mode - turn in last known direction
            if(last_error > 0)
                set_motors(BASE_SPEED, -BASE_SPEED/2);
            else
                set_motors(-BASE_SPEED/2, BASE_SPEED);
            continue;
        }
        
        // Calculate error
        error = calculate_error(sensors);
        
        // Compute PID correction
        correction = pid_compute(error);
        
        // Apply correction to motor speeds
        left_speed = BASE_SPEED + correction;
        right_speed = BASE_SPEED - correction;
        
        // Set motor speeds
        set_motors(left_speed, right_speed);
        
        // Small delay
        delay_ms(5);
    }
}
```

---

## 4. Digital Clock with Alarm

### 4.1 Complete Clock Implementation

```c
/*
 * DIGITAL CLOCK WITH ALARM
 * RTC-based with LCD display
 */

#include <reg51.h>
#include <stdio.h>

//=== Time Structure ===
typedef struct {
    unsigned char hour;
    unsigned char minute;
    unsigned char second;
    unsigned char day;
    unsigned char date;
    unsigned char month;
    unsigned char year;
} DateTime;

DateTime current_time;
DateTime alarm_time = {7, 0, 0, 0, 0, 0, 0};  // Default 7:00 AM

//=== State ===
bit alarm_enabled = 0;
bit alarm_triggered = 0;
bit setting_mode = 0;
unsigned char setting_pos = 0;
bit display_seconds = 1;

//=== Hardware ===
sbit BUZZER = P2^0;
sbit BUTTON_SET = P3^2;
sbit BUTTON_UP = P3^3;
sbit BUTTON_DOWN = P3^4;
sbit BUTTON_ALARM = P3^5;

//=== Display Functions ===
void display_time(void)
{
    char line1[17], line2[17];
    
    // Format time
    sprintf(line1, " %02d:%02d", 
            current_time.hour, current_time.minute);
    
    if(display_seconds)
    {
        sprintf(line1 + 6, ":%02d ", current_time.second);
    }
    else
    {
        sprintf(line1 + 6, "    ");
    }
    
    // Alarm indicator
    if(alarm_enabled)
        strcat(line1, "AL");
    else
        strcat(line1, "  ");
    
    // Format date
    sprintf(line2, "%s %02d/%02d/20%02d",
            get_day_name(current_time.day),
            current_time.date,
            current_time.month,
            current_time.year);
    
    lcd_goto(0, 0);
    lcd_string(line1);
    lcd_goto(1, 0);
    lcd_string(line2);
    
    // Highlight in setting mode
    if(setting_mode)
    {
        lcd_goto(0, setting_positions[setting_pos]);
        lcd_cmd(LCD_BLINK_ON);
    }
}

void display_alarm_setting(void)
{
    char buf[17];
    
    lcd_goto(0, 0);
    lcd_string("  Set Alarm:    ");
    
    lcd_goto(1, 0);
    sprintf(buf, "    %02d:%02d       ", 
            alarm_time.hour, alarm_time.minute);
    lcd_string(buf);
}

char* get_day_name(unsigned char day)
{
    code char *days[] = {"Sun", "Mon", "Tue", "Wed", 
                          "Thu", "Fri", "Sat"};
    return days[day % 7];
}

//=== Button Handling ===
void handle_buttons(void)
{
    static bit last_set = 1;
    static bit last_up = 1;
    static bit last_down = 1;
    static bit last_alarm = 1;
    
    // SET button
    if(!BUTTON_SET && last_set)
    {
        if(!setting_mode)
        {
            setting_mode = 1;
            setting_pos = 0;
        }
        else
        {
            setting_pos++;
            if(setting_pos >= 6)
            {
                setting_mode = 0;
                rtc_write(&current_time);
            }
        }
    }
    last_set = BUTTON_SET;
    
    // UP button
    if(!BUTTON_UP && last_up)
    {
        if(setting_mode)
        {
            increment_field(setting_pos);
        }
    }
    last_up = BUTTON_UP;
    
    // DOWN button  
    if(!BUTTON_DOWN && last_down)
    {
        if(setting_mode)
        {
            decrement_field(setting_pos);
        }
    }
    last_down = BUTTON_DOWN;
    
    // ALARM button
    if(!BUTTON_ALARM && last_alarm)
    {
        if(alarm_triggered)
        {
            alarm_triggered = 0;
            BUZZER = 0;
        }
        else
        {
            alarm_enabled = !alarm_enabled;
        }
    }
    last_alarm = BUTTON_ALARM;
}

void increment_field(unsigned char pos)
{
    switch(pos)
    {
        case 0:  // Hour
            current_time.hour = (current_time.hour + 1) % 24;
            break;
        case 1:  // Minute
            current_time.minute = (current_time.minute + 1) % 60;
            break;
        case 2:  // Day
            current_time.day = (current_time.day + 1) % 7;
            break;
        case 3:  // Date
            current_time.date = (current_time.date % 31) + 1;
            break;
        case 4:  // Month
            current_time.month = (current_time.month % 12) + 1;
            break;
        case 5:  // Year
            current_time.year = (current_time.year + 1) % 100;
            break;
    }
}

//=== Alarm Functions ===
void check_alarm(void)
{
    if(alarm_enabled && !alarm_triggered)
    {
        if(current_time.hour == alarm_time.hour &&
           current_time.minute == alarm_time.minute &&
           current_time.second == 0)
        {
            alarm_triggered = 1;
        }
    }
    
    if(alarm_triggered)
    {
        // Beep pattern
        BUZZER = (current_time.second % 2) ? 1 : 0;
    }
}

//=== Temperature Display ===
void display_temperature(void)
{
    int temp = rtc_read_temperature();  // DS3231 has built-in sensor
    char buf[17];
    
    sprintf(buf, "Temp: %d.%dC", temp / 4, (temp % 4) * 25);
    lcd_goto(1, 0);
    lcd_string(buf);
}

//=== Main Function ===
void main(void)
{
    unsigned char last_second = 0;
    unsigned char mode = 0;  // 0=time, 1=alarm set, 2=temp
    
    lcd_init();
    i2c_init();
    
    // Initialize RTC if needed
    if(rtc_is_stopped())
    {
        current_time.hour = 12;
        current_time.minute = 0;
        current_time.second = 0;
        current_time.day = 1;
        current_time.date = 1;
        current_time.month = 1;
        current_time.year = 24;
        rtc_write(&current_time);
    }
    
    lcd_string("Digital Clock");
    lcd_goto(1, 0);
    lcd_string("Starting...");
    delay_ms(1000);
    lcd_clear();
    
    while(1)
    {
        // Read time from RTC
        rtc_read(&current_time);
        
        // Check alarm
        check_alarm();
        
        // Handle buttons
        handle_buttons();
        
        // Update display
        if(setting_mode)
        {
            display_time();
        }
        else
        {
            switch(mode)
            {
                case 0:
                    display_time();
                    break;
                case 1:
                    display_alarm_setting();
                    break;
                case 2:
                    display_temperature();
                    break;
            }
        }
        
        // Blink colon
        if(current_time.second != last_second)
        {
            last_second = current_time.second;
            display_seconds = !display_seconds;
        }
        
        delay_ms(100);
    }
}
```

---

## 📋 Project Summary Table

| Project | Complexity | Sensors | Actuators | Display | Communication |
|---------|------------|---------|-----------|---------|---------------|
| Home Automation | High | 6 | 5 | LCD 20×4 | BT + WiFi |
| Data Logger | Medium | 4 | 1 | OLED | UART + SD |
| Line Follower | Medium | 5 | 2 motors | None | None |
| Digital Clock | Low | 0 | 1 | LCD 16×2 | I2C (RTC) |

---

## ❓ Quick Revision Questions

1. **What is the advantage of using a state machine for menu navigation?**
   <details>
   <summary>Show Answer</summary>
   State machines provide clear, organized control flow. Each state handles specific behavior. Transitions are well-defined. Easy to add new states/features. Simplifies debugging and maintenance.
   </details>

2. **How does PID control improve line following compared to simple ON/OFF control?**
   <details>
   <summary>Show Answer</summary>
   PID provides smooth, proportional response. P term gives quick response proportional to error. D term anticipates changes and reduces oscillation. I term eliminates steady-state error. Results in smoother, faster line tracking.
   </details>

3. **Why use RTC for timekeeping instead of timer interrupts?**
   <details>
   <summary>Show Answer</summary>
   RTC maintains time during power off with battery backup. More accurate (crystal oscillator). Independent of MCU operation. Handles calendar complexity (months, leap years). Frees MCU timers for other tasks.
   </details>

4. **What considerations are important for battery-powered data loggers?**
   <details>
   <summary>Show Answer</summary>
   Low power sleep modes between readings. Efficient sampling intervals. Low-power sensors. EEPROM/flash wear leveling. Battery monitoring. Data integrity on power loss. Minimal display updates.
   </details>

5. **How do you handle sensor calibration in embedded systems?**
   <details>
   <summary>Show Answer</summary>
   Store calibration coefficients in EEPROM. Implement calibration routine at startup or on command. Use known reference values. Apply offset and gain correction. Allow user to trigger recalibration. Log calibration date.
   </details>

---

## 🧭 Navigation

| Previous | Up | Course Complete |
|----------|-----|-----------------|
| [10.5 Display and HMI Systems](05-display-hmi-systems.md) | [Unit 10 Index](README.md) | 🎉 |

---

*[← Previous: Display and HMI Systems](05-display-hmi-systems.md) | [Course Complete - Return to Main Index →](../README.md)*

---

## 🎓 Course Completion

Congratulations on completing the **Microprocessors and Microcontrollers** course notes!

### What You've Learned:

1. **Microprocessor Fundamentals** - 8085/8086 architecture, instruction sets
2. **Assembly Programming** - Low-level programming techniques
3. **Peripheral Interfacing** - ADC, DAC, displays, keyboards
4. **Microcontroller Architecture** - 8051 features and programming
5. **Communication Protocols** - UART, I2C, SPI
6. **Memory Systems** - RAM, ROM, memory interfacing
7. **Advanced Processors** - x86 evolution, ARM architecture
8. **Complete Projects** - Home automation, data logging, robotics

### Next Steps:

- Practice with real hardware (development boards)
- Explore ARM Cortex-M microcontrollers
- Learn embedded Linux
- Study RTOS concepts
- Work on IoT projects

**Good luck with your studies and projects!** 🚀
