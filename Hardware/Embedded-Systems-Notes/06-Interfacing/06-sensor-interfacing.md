# Chapter 6.6: Sensor Interfacing

## Introduction

Sensors convert physical quantities into electrical signals that microcontrollers can process. This chapter covers common sensor types, interfacing techniques, and signal processing fundamentals.

---

## Sensor Categories

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SENSOR CLASSIFICATION                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  BY OUTPUT TYPE:                                                     │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Type       │ Output           │ Interface    │ Examples   │     │
│  ├────────────────────────────────────────────────────────────┤     │
│  │ Analog     │ Voltage/Current  │ ADC          │ LDR, NTC   │     │
│  │ Digital    │ Logic levels     │ GPIO         │ PIR, reed  │     │
│  │ PWM/Freq   │ Pulse train      │ Timer IC     │ HC-SR04    │     │
│  │ Serial     │ Data packets     │ UART/I2C/SPI │ BMP280     │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  BY MEASURED QUANTITY:                                               │
│                                                                      │
│  Temperature:  NTC thermistor, LM35, DS18B20, BME280               │
│  Light:        LDR, photodiode, phototransistor, TSL2561           │
│  Motion:       PIR, accelerometer, gyroscope, IMU                  │
│  Distance:     Ultrasonic (HC-SR04), IR (GP2Y0A21), LiDAR          │
│  Pressure:     BMP280, MPX5010                                     │
│  Humidity:     DHT11, DHT22, BME280                                │
│  Gas:          MQ-2, MQ-7, CCS811                                  │
│  Sound:        Microphone, MAX4466                                 │
│  Touch:        Capacitive, resistive                               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Temperature Sensors

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TEMPERATURE SENSOR TYPES                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  NTC THERMISTOR (Analog):                                           │
│                                                                      │
│  VCC ───┬                    R = R25 × exp(B × (1/T - 1/T25))      │
│         │                                                            │
│        ─┴─ R (fixed)         Where:                                 │
│        ─┬─ 10kΩ              R25 = resistance at 25°C              │
│         │                    B = B-coefficient (~3950)              │
│         ├───▶ ADC            T = temperature in Kelvin             │
│         │                                                            │
│        ─┴─ NTC                                                       │
│        ─┬─ (10kΩ@25°C)       Pros: Cheap, fast                      │
│         │                    Cons: Non-linear, limited range        │
│        GND                                                           │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  LM35 (Analog):              DS18B20 (Digital 1-Wire):             │
│                                                                      │
│  VCC─┬─LM35─┬─GND           VCC ───┬                                │
│      │      │                      │                                │
│     ─┴─     └──▶ ADC              ─┴─ 4.7kΩ pull-up                │
│                                   ─┬─                                │
│  Output: 10 mV/°C                  │                                │
│  0°C = 0V, 100°C = 1V              ├───▶ DQ (data)                  │
│                                    │                                │
│  Pros: Linear output               │                                │
│  Cons: Limited range              DS18B20                           │
│                                    │                                │
│                                   GND                                │
│                                                                      │
│                                   12-bit resolution                 │
│                                   -55°C to +125°C                   │
│                                   Each has unique ID                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### DS18B20 Driver

```c
// 1-Wire protocol for DS18B20
#define DQ_PIN       0
#define DQ_PORT      GPIOA

#define DQ_HIGH()    gpio_mode_input_pullup(DQ_PORT, DQ_PIN)  // Release
#define DQ_LOW()     do { gpio_mode_output(DQ_PORT, DQ_PIN); \
                         GPIO_CLEAR(DQ_PORT, DQ_PIN); } while(0)
#define DQ_READ()    GPIO_READ(DQ_PORT, DQ_PIN)

// 1-Wire reset and presence detect
bool onewire_reset(void) {
    DQ_LOW();
    delay_us(480);
    DQ_HIGH();
    delay_us(70);
    bool presence = !DQ_READ();  // Low = device present
    delay_us(410);
    return presence;
}

// Write a bit
void onewire_write_bit(bool bit) {
    DQ_LOW();
    if (bit) {
        delay_us(6);
        DQ_HIGH();
        delay_us(64);
    } else {
        delay_us(60);
        DQ_HIGH();
        delay_us(10);
    }
}

// Read a bit
bool onewire_read_bit(void) {
    DQ_LOW();
    delay_us(6);
    DQ_HIGH();
    delay_us(9);
    bool bit = DQ_READ();
    delay_us(55);
    return bit;
}

// Write byte
void onewire_write_byte(uint8_t data) {
    for (int i = 0; i < 8; i++) {
        onewire_write_bit(data & 0x01);
        data >>= 1;
    }
}

// Read byte
uint8_t onewire_read_byte(void) {
    uint8_t data = 0;
    for (int i = 0; i < 8; i++) {
        if (onewire_read_bit()) {
            data |= (1 << i);
        }
    }
    return data;
}

// DS18B20 commands
#define CMD_SKIP_ROM        0xCC
#define CMD_CONVERT_T       0x44
#define CMD_READ_SCRATCHPAD 0xBE

float ds18b20_read_temperature(void) {
    // Start conversion
    onewire_reset();
    onewire_write_byte(CMD_SKIP_ROM);
    onewire_write_byte(CMD_CONVERT_T);
    
    // Wait for conversion (750ms for 12-bit)
    delay_ms(750);
    
    // Read result
    onewire_reset();
    onewire_write_byte(CMD_SKIP_ROM);
    onewire_write_byte(CMD_READ_SCRATCHPAD);
    
    uint8_t lsb = onewire_read_byte();
    uint8_t msb = onewire_read_byte();
    
    int16_t raw = (msb << 8) | lsb;
    return raw / 16.0f;  // 12-bit resolution: 0.0625°C per bit
}
```

---

## Light Sensors

```
┌─────────────────────────────────────────────────────────────────────┐
│                    LIGHT SENSOR TYPES                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  LDR (Light Dependent Resistor):                                    │
│                                                                      │
│  VCC ───┬                    Resistance varies with light:         │
│         │                    Dark: 1MΩ+                             │
│        ─┴─ R (10kΩ)          Bright: 1kΩ                           │
│        ─┬─                                                          │
│         │                    Simple voltage divider readout        │
│         ├───▶ ADC                                                   │
│         │                                                            │
│        ─┴─ LDR                                                       │
│        ─┬─                                                          │
│         │                                                            │
│        GND                                                           │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  PHOTODIODE / PHOTOTRANSISTOR:                                      │
│                                                                      │
│  Photodiode (current mode):   Phototransistor:                     │
│                                                                      │
│  VCC ─────┬                   VCC ─────┬                            │
│           │                           ─┴─ R                         │
│       ┌───┘                           ─┬─                           │
│       │ ▲ Light                        │                            │
│       │─┼─                             ├───▶ ADC                    │
│       │ │                              │                            │
│       ▼ │    Trans-                   ─┼─ Light ↓                   │
│       ──┘    impedance                ─┴─                           │
│       │      amplifier                 │                            │
│      GND                              GND                            │
│                                                                      │
│  Faster response than LDR                                           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

```c
// LDR reading with calibration
uint16_t read_light_level(void) {
    return adc_read(ADC_CHANNEL_LDR);  // 0-4095
}

// Convert to approximate lux (needs calibration)
float ldr_to_lux(uint16_t adc_value) {
    // Approximate conversion (varies by LDR type)
    float voltage = (adc_value * 3.3f) / 4095.0f;
    float resistance = 10000.0f * (3.3f - voltage) / voltage;
    
    // Empirical formula (adjust constants for your LDR)
    float lux = 500000.0f / resistance;
    return lux;
}

// Threshold-based detection
bool is_dark(void) {
    return read_light_level() < 1000;  // Threshold ~25% of range
}
```

---

## Distance Sensors

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ULTRASONIC SENSOR (HC-SR04)                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  OPERATION:                                                          │
│                                                                      │
│  1. Send 10µs trigger pulse                                         │
│  2. Sensor emits 8× 40kHz ultrasonic bursts                        │
│  3. Echo pin goes HIGH when burst sent                              │
│  4. Echo pin goes LOW when echo received                            │
│  5. Pulse duration = time for sound round-trip                      │
│                                                                      │
│  TIMING:                                                             │
│                                                                      │
│  Trigger: ──────┐    ┌────────────────────────────────────          │
│                 └────┘                                               │
│                 10µs                                                 │
│                                                                      │
│  Echo:    ─────────────┐                              ┌─────        │
│                        └──────────────────────────────┘             │
│                        ◄──────── t (µs) ─────────────▶              │
│                                                                      │
│  DISTANCE CALCULATION:                                               │
│                                                                      │
│  Distance = (t × 343 m/s) / 2  [round trip]                        │
│  Distance_cm = t / 58                                               │
│                                                                      │
│  Range: 2cm - 400cm                                                 │
│  Accuracy: ±3mm                                                     │
│                                                                      │
│  WIRING:                                                             │
│  ┌─────────────────────────────────────────────────────────┐        │
│  │ HC-SR04 │ MCU                                           │        │
│  ├─────────────────────────────────────────────────────────┤        │
│  │ VCC     │ 5V                                            │        │
│  │ GND     │ GND                                           │        │
│  │ Trig    │ GPIO (output)                                 │        │
│  │ Echo    │ GPIO (input) - use voltage divider for 3.3V  │        │
│  └─────────────────────────────────────────────────────────┘        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

```c
// HC-SR04 ultrasonic sensor
#define TRIG_PIN  0  // PA0
#define ECHO_PIN  1  // PA1

void hcsr04_init(void) {
    gpio_mode_output(GPIOA, TRIG_PIN);
    gpio_mode_input(GPIOA, ECHO_PIN);
    GPIO_CLEAR(GPIOA, TRIG_PIN);
}

uint32_t hcsr04_measure_cm(void) {
    // Send trigger pulse
    GPIO_SET(GPIOA, TRIG_PIN);
    delay_us(10);
    GPIO_CLEAR(GPIOA, TRIG_PIN);
    
    // Wait for echo to go HIGH
    uint32_t timeout = 30000;  // 30ms timeout
    while (!GPIO_READ(GPIOA, ECHO_PIN) && timeout--) {
        delay_us(1);
    }
    if (timeout == 0) return 0;  // Timeout
    
    // Measure echo pulse duration
    uint32_t start = micros();
    while (GPIO_READ(GPIOA, ECHO_PIN) && timeout--) {
        delay_us(1);
    }
    uint32_t pulse_us = micros() - start;
    
    // Convert to cm
    return pulse_us / 58;
}

// Average multiple readings for stability
uint32_t hcsr04_measure_avg(uint8_t samples) {
    uint32_t sum = 0;
    for (int i = 0; i < samples; i++) {
        sum += hcsr04_measure_cm();
        delay_ms(10);
    }
    return sum / samples;
}
```

---

## Inertial Measurement Unit (IMU)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MPU6050 6-AXIS IMU                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Contains: 3-axis accelerometer + 3-axis gyroscope                 │
│  Interface: I2C (address 0x68 or 0x69)                             │
│                                                                      │
│  ACCELEROMETER:                  GYROSCOPE:                         │
│                                                                      │
│       Y                                Y                            │
│       │                                │                            │
│       │    Z                           │    Z                       │
│       │   ╱                            │   ╱                        │
│       │  ╱                             │  ╱                         │
│       │ ╱                              │ ╱                          │
│       ├──────── X                      ├──────── X                  │
│      ╱                                ╱                             │
│     ╱                                ╱                              │
│                                                                      │
│  Measures:                         Measures:                        │
│  Linear acceleration (g)           Angular velocity (°/s)           │
│  Tilt angle (when stationary)      Rotation rate                    │
│                                                                      │
│  REGISTERS:                                                          │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Address │ Data                                             │     │
│  ├────────────────────────────────────────────────────────────┤     │
│  │  0x3B   │ ACCEL_XOUT_H (accelerometer data)                │     │
│  │  0x43   │ GYRO_XOUT_H (gyroscope data)                     │     │
│  │  0x6B   │ PWR_MGMT_1 (power management)                    │     │
│  │  0x75   │ WHO_AM_I (device ID = 0x68)                      │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

```c
#define MPU6050_ADDR      0x68
#define REG_PWR_MGMT_1    0x6B
#define REG_ACCEL_XOUT_H  0x3B
#define REG_WHO_AM_I      0x75

typedef struct {
    int16_t accel_x, accel_y, accel_z;
    int16_t gyro_x, gyro_y, gyro_z;
    int16_t temp;
} MPU6050_Data_t;

bool mpu6050_init(void) {
    // Check device ID
    uint8_t who_am_i;
    i2c_read_reg(MPU6050_ADDR, REG_WHO_AM_I, &who_am_i, 1);
    if (who_am_i != 0x68) return false;
    
    // Wake up device (clear sleep bit)
    uint8_t data = 0x00;
    i2c_write_reg(MPU6050_ADDR, REG_PWR_MGMT_1, &data, 1);
    
    delay_ms(100);
    return true;
}

void mpu6050_read(MPU6050_Data_t *data) {
    uint8_t buffer[14];
    
    // Read 14 bytes starting from ACCEL_XOUT_H
    i2c_read_reg(MPU6050_ADDR, REG_ACCEL_XOUT_H, buffer, 14);
    
    // Parse data (big-endian)
    data->accel_x = (buffer[0] << 8) | buffer[1];
    data->accel_y = (buffer[2] << 8) | buffer[3];
    data->accel_z = (buffer[4] << 8) | buffer[5];
    data->temp    = (buffer[6] << 8) | buffer[7];
    data->gyro_x  = (buffer[8] << 8) | buffer[9];
    data->gyro_y  = (buffer[10] << 8) | buffer[11];
    data->gyro_z  = (buffer[12] << 8) | buffer[13];
}

// Convert to physical units
float accel_to_g(int16_t raw) {
    // ±2g range: 16384 LSB/g
    return raw / 16384.0f;
}

float gyro_to_dps(int16_t raw) {
    // ±250°/s range: 131 LSB/(°/s)
    return raw / 131.0f;
}

// Calculate tilt angle from accelerometer
float calculate_pitch(MPU6050_Data_t *data) {
    float ax = accel_to_g(data->accel_x);
    float ay = accel_to_g(data->accel_y);
    float az = accel_to_g(data->accel_z);
    
    return atan2f(ax, sqrtf(ay*ay + az*az)) * 180.0f / M_PI;
}

float calculate_roll(MPU6050_Data_t *data) {
    float ay = accel_to_g(data->accel_y);
    float az = accel_to_g(data->accel_z);
    
    return atan2f(ay, az) * 180.0f / M_PI;
}
```

---

## Environmental Sensors

```
┌─────────────────────────────────────────────────────────────────────┐
│                    BME280 (Temp + Humidity + Pressure)               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Interface: I2C (0x76 or 0x77) or SPI                              │
│                                                                      │
│  MEASUREMENTS:                                                       │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Parameter   │ Range              │ Resolution             │     │
│  ├────────────────────────────────────────────────────────────┤     │
│  │ Temperature │ -40 to +85°C       │ 0.01°C                 │     │
│  │ Humidity    │ 0 to 100% RH       │ 0.008% RH              │     │
│  │ Pressure    │ 300 to 1100 hPa    │ 0.18 Pa                │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  ALTITUDE CALCULATION:                                               │
│                                                                      │
│  Altitude = 44330 × (1 - (P/P0)^0.1903)                            │
│                                                                      │
│  Where: P = measured pressure                                       │
│         P0 = sea-level pressure (1013.25 hPa)                      │
│                                                                      │
│  APPLICATION: Weather stations, indoor air quality, altitude        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Sensor Comparison

| Sensor | Type | Interface | Range | Accuracy |
|--------|------|-----------|-------|----------|
| **NTC 10k** | Temperature | ADC | -40 to 125°C | ±1°C |
| **DS18B20** | Temperature | 1-Wire | -55 to 125°C | ±0.5°C |
| **BME280** | Temp/Humid/Press | I2C/SPI | Multi | High |
| **LDR** | Light | ADC | ~1-1M lux | Low |
| **HC-SR04** | Distance | GPIO | 2-400cm | ±3mm |
| **MPU6050** | Motion | I2C | ±2-16g, ±250-2000°/s | Varies |
| **DHT22** | Temp/Humidity | 1-Wire | 0-100% RH | ±2% RH |

---

## Summary Table

| Sensor Type | Output | Processing | Considerations |
|-------------|--------|------------|----------------|
| **Resistive** | Voltage divider | ADC + math | Linearization |
| **Voltage** | Direct voltage | ADC + scaling | Reference accuracy |
| **Digital** | Protocol data | Serial comm | Timing critical |
| **Pulse** | Time measurement | Timer capture | Precision timing |
| **Current** | 4-20mA | Current to voltage | Industrial standard |

---

## Quick Revision Questions

1. **What is the difference between NTC and PTC thermistors? How is NTC read?**

2. **Explain the 1-Wire protocol basics. Why is a pull-up resistor required?**

3. **Calculate the distance if HC-SR04 echo pulse is 1160 µs.**

4. **What data does an IMU provide? How do you calculate tilt angle?**

5. **Why might you choose a digital sensor (like DS18B20) over analog (like LM35)?**

6. **Design a simple weather station: what sensors and interfaces would you use?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Chapter 6.5: Motor Control](05-motor-control.md) | [Unit Index](README.md) | [Unit 7: System Design](../07-System-Design/README.md) |

---
