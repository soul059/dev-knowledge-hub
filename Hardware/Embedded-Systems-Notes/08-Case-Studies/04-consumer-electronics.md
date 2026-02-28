# Chapter 8.4: Consumer Electronics Embedded Systems

## Introduction

Consumer electronics represent the highest-volume segment of embedded systems, with billions of devices manufactured annually. From fitness trackers and smart speakers to wireless earbuds and gaming controllers, these products demand aggressive cost optimization, excellent user experience, rapid development cycles, and long battery life. This chapter explores the design considerations, architecture patterns, and optimization techniques unique to consumer electronics.

---

## Consumer Electronics Landscape

```
┌─────────────────────────────────────────────────────────────────────┐
│              CONSUMER ELECTRONICS CATEGORIES                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    WEARABLES                                   │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      │ │
│  │  │ Fitness  │  │ Smart    │  │ Wireless │  │ Smart    │      │ │
│  │  │ Tracker  │  │ Watch    │  │ Earbuds  │  │ Glasses  │      │ │
│  │  │          │  │          │  │          │  │          │      │ │
│  │  │ • Steps  │  │ • Notif. │  │ • Audio  │  │ • AR     │      │ │
│  │  │ • HR     │  │ • Apps   │  │ • ANC    │  │ • Display│      │ │
│  │  │ • Sleep  │  │ • Health │  │ • Voice  │  │ • Camera │      │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘      │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    SMART HOME                                  │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      │ │
│  │  │ Smart    │  │ Video    │  │ Smart    │  │ Robot    │      │ │
│  │  │ Speaker  │  │ Doorbell │  │ Thermo.  │  │ Vacuum   │      │ │
│  │  │          │  │          │  │          │  │          │      │ │
│  │  │ • Voice  │  │ • Camera │  │ • Temp   │  │ • Nav    │      │ │
│  │  │ • Audio  │  │ • Motion │  │ • WiFi   │  │ • Mapping│      │ │
│  │  │ • AI     │  │ • 2-way  │  │ • Learn  │  │ • Sensors│      │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘      │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    GAMING & ENTERTAINMENT                      │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      │ │
│  │  │ Game     │  │ VR       │  │ Streaming│  │ E-Reader │      │ │
│  │  │ Controller│ │ Headset  │  │ Device   │  │          │      │ │
│  │  │          │  │          │  │          │  │          │      │ │
│  │  │ • Haptic │  │ • IMU    │  │ • Video  │  │ • E-Ink  │      │ │
│  │  │ • Low lat│  │ • Display│  │ • WiFi   │  │ • Touch  │      │ │
│  │  │ • BT/USB │  │ • Audio  │  │ • HDMI   │  │ • Light  │      │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘      │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  KEY DESIGN DRIVERS:                                                 │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │ Factor        │ Consumer Expectation                          │ │
│  ├───────────────┼───────────────────────────────────────────────┤ │
│  │ Cost          │ Aggressive BOM targets (<$10-50)             │ │
│  │ Size          │ Compact, pocket-sized, lightweight           │ │
│  │ Battery       │ Days to weeks of use                         │ │
│  │ UX            │ Seamless, instant-on, intuitive              │ │
│  │ Connectivity  │ Bluetooth, WiFi, app integration             │ │
│  │ Time-to-market│ 6-12 months development cycle               │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Fitness Tracker Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│              FITNESS TRACKER SYSTEM ARCHITECTURE                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                   HARDWARE BLOCK DIAGRAM                       │ │
│  │                                                                │ │
│  │                    ┌─────────────────┐                        │ │
│  │                    │   DISPLAY       │                        │ │
│  │                    │   AMOLED/LCD    │                        │ │
│  │                    │   128x64-240x240│                        │ │
│  │                    └────────┬────────┘                        │ │
│  │                             │ SPI                             │ │
│  │  ┌────────────────────────────────────────────────────────┐  │ │
│  │  │                                                         │  │ │
│  │  │                   MAIN MCU                              │  │ │
│  │  │              (Nordic nRF52840)                          │  │ │
│  │  │                                                         │  │ │
│  │  │  ┌─────────────────────────────────────────────────┐   │  │ │
│  │  │  │  ARM Cortex-M4F @ 64MHz                         │   │  │ │
│  │  │  │  • 256KB RAM / 1MB Flash                        │   │  │ │
│  │  │  │  • BLE 5.0 integrated                           │   │  │ │
│  │  │  │  • Active: 3mA, Sleep: 1µA                      │   │  │ │
│  │  │  │  • Hardware crypto (AES, SHA)                   │   │  │ │
│  │  │  └─────────────────────────────────────────────────┘   │  │ │
│  │  │                                                         │  │ │
│  │  └───────────┬─────────────┬─────────────┬────────────────┘  │ │
│  │              │             │             │                    │ │
│  │  ┌───────────▼──┐  ┌───────▼───┐  ┌──────▼─────┐             │ │
│  │  │ Accelerometer│  │  PPG      │  │   GPS      │             │ │
│  │  │ + Gyroscope  │  │ Heart Rate│  │ (Optional) │             │ │
│  │  │ (BMI270)     │  │ (MAX30102)│  │            │             │ │
│  │  │              │  │           │  │            │             │ │
│  │  │ I2C/SPI      │  │ I2C       │  │ UART       │             │ │
│  │  └──────────────┘  └───────────┘  └────────────┘             │ │
│  │                                                                │ │
│  │  ┌──────────────────────────────────────────────────────────┐ │ │
│  │  │                    POWER SYSTEM                          │ │ │
│  │  │                                                          │ │ │
│  │  │  ┌─────────┐    ┌─────────┐    ┌─────────┐              │ │ │
│  │  │  │ LiPo    │───▶│ PMIC    │───▶│ 1.8V/3V │              │ │ │
│  │  │  │ 100mAh  │    │ BQ25120A│    │ Rails   │              │ │ │
│  │  │  └─────────┘    └────┬────┘    └─────────┘              │ │ │
│  │  │                      │                                   │ │ │
│  │  │                 Fuel Gauge                               │ │ │
│  │  │                 MAX17048                                 │ │ │
│  │  └──────────────────────────────────────────────────────────┘ │ │
│  │                                                                │ │
│  │  ┌──────────────────────────────────────────────────────────┐ │ │
│  │  │  ADDITIONAL SENSORS                                      │ │ │
│  │  │                                                          │ │ │
│  │  │  ┌──────────┐  ┌──────────┐  ┌──────────┐              │ │ │
│  │  │  │  Touch   │  │ Ambient  │  │Barometer │              │ │ │
│  │  │  │ Buttons  │  │  Light   │  │(Altitude)│              │ │ │
│  │  │  └──────────┘  └──────────┘  └──────────┘              │ │ │
│  │  └──────────────────────────────────────────────────────────┘ │ │
│  │                                                                │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Step Counter Implementation

```c
/* ================================================================
 * Step Counter Algorithm (Pedometer)
 * Low-power fitness tracker implementation
 * ================================================================ */

#include <stdint.h>
#include <stdbool.h>
#include <math.h>

/* Configuration */
#define SAMPLE_RATE_HZ          50      /* Accelerometer sample rate */
#define WINDOW_SIZE             20      /* Moving average window */
#define STEP_THRESHOLD          1.2f    /* G-force threshold */
#define MIN_STEP_INTERVAL_MS    250     /* Minimum time between steps */
#define MAX_STEP_INTERVAL_MS    2000    /* Maximum time between steps */

/* Step counter state */
typedef struct {
    /* Raw data buffer */
    float accel_magnitude[WINDOW_SIZE];
    uint8_t buffer_index;
    
    /* Filtering */
    float filtered_value;
    float baseline;
    
    /* Peak detection */
    float peak_value;
    bool in_step;
    uint32_t last_step_time;
    
    /* Outputs */
    uint32_t step_count;
    uint32_t daily_steps;
    float cadence;  /* Steps per minute */
    
    /* Cadence calculation */
    uint32_t step_times[8];
    uint8_t step_time_index;
} pedometer_t;

/* ================================================================
 * Initialize Pedometer
 * ================================================================ */
void pedometer_init(pedometer_t *ped) {
    memset(ped, 0, sizeof(pedometer_t));
    ped->baseline = 1.0f;  /* 1G at rest */
}

/* ================================================================
 * Process Accelerometer Sample
 * Called at 50Hz with X, Y, Z accelerometer readings
 * ================================================================ */
void pedometer_process(pedometer_t *ped, float x, float y, float z,
                       uint32_t timestamp_ms) {
    /* Calculate magnitude of acceleration vector */
    float magnitude = sqrtf(x*x + y*y + z*z);
    
    /* Store in circular buffer */
    ped->accel_magnitude[ped->buffer_index] = magnitude;
    ped->buffer_index = (ped->buffer_index + 1) % WINDOW_SIZE;
    
    /* Low-pass filter (moving average) */
    float sum = 0;
    for (int i = 0; i < WINDOW_SIZE; i++) {
        sum += ped->accel_magnitude[i];
    }
    ped->filtered_value = sum / WINDOW_SIZE;
    
    /* Adaptive baseline (slow update) */
    ped->baseline = 0.999f * ped->baseline + 0.001f * ped->filtered_value;
    
    /* Deviation from baseline */
    float deviation = ped->filtered_value - ped->baseline;
    
    /* Step detection state machine */
    uint32_t time_since_last = timestamp_ms - ped->last_step_time;
    
    if (!ped->in_step) {
        /* Looking for step start (peak) */
        if (deviation > (STEP_THRESHOLD - 1.0f) &&
            time_since_last > MIN_STEP_INTERVAL_MS) {
            ped->in_step = true;
            ped->peak_value = deviation;
        }
    } else {
        /* Track peak */
        if (deviation > ped->peak_value) {
            ped->peak_value = deviation;
        }
        
        /* Detect step end (zero crossing) */
        if (deviation < 0) {
            /* Valid step if peak was high enough */
            if (ped->peak_value > (STEP_THRESHOLD - 1.0f) &&
                time_since_last < MAX_STEP_INTERVAL_MS) {
                
                /* Count the step */
                ped->step_count++;
                ped->daily_steps++;
                
                /* Store time for cadence calculation */
                ped->step_times[ped->step_time_index] = timestamp_ms;
                ped->step_time_index = (ped->step_time_index + 1) % 8;
                
                /* Calculate cadence (steps per minute) */
                uint32_t oldest = ped->step_times[(ped->step_time_index) % 8];
                if (timestamp_ms > oldest && oldest > 0) {
                    float interval_sec = (timestamp_ms - oldest) / 1000.0f;
                    ped->cadence = (8.0f / interval_sec) * 60.0f;
                }
                
                ped->last_step_time = timestamp_ms;
            }
            
            ped->in_step = false;
            ped->peak_value = 0;
        }
    }
}

/* ================================================================
 * Low-Power Accelerometer Configuration
 * ================================================================ */
void accel_configure_low_power(void) {
    /* BMI270 configuration for step counting */
    
    /* Set to low-power mode with FIFO */
    bmi270_write_reg(BMI2_PWR_CONF_ADDR, 
        BMI2_ADV_POWER_SAVE_EN);
    
    /* Configure ODR to 50Hz */
    bmi270_write_reg(BMI2_ACC_CONF_ADDR,
        (BMI2_ACC_ODR_50HZ) |
        (BMI2_ACC_NORMAL_AVG4 << 4));
    
    /* Set range to ±4G */
    bmi270_write_reg(BMI2_ACC_RANGE_ADDR, BMI2_ACC_RANGE_4G);
    
    /* Configure FIFO watermark for batch processing */
    uint16_t watermark = 50;  /* 1 second of data */
    bmi270_write_reg(BMI2_FIFO_WTM_0_ADDR, watermark & 0xFF);
    bmi270_write_reg(BMI2_FIFO_WTM_1_ADDR, watermark >> 8);
    
    /* Enable FIFO watermark interrupt */
    bmi270_write_reg(BMI2_INT1_MAP_ADDR, BMI2_FWM_INT);
}
```

---

## Wireless Earbuds System

```
┌─────────────────────────────────────────────────────────────────────┐
│              TRUE WIRELESS STEREO (TWS) EARBUDS                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                   LEFT EARBUD                                  │ │
│  │                                                                │ │
│  │  ┌──────────────────────────────────────────────────────────┐ │ │
│  │  │                BT AUDIO SoC                               │ │ │
│  │  │             (Qualcomm QCC3040)                            │ │ │
│  │  │                                                           │ │ │
│  │  │  • Dual-core Kalimba DSP + ARM Cortex-M0                 │ │ │
│  │  │  • Bluetooth 5.2 with aptX Adaptive                      │ │ │
│  │  │  • Active Noise Cancellation (ANC)                       │ │ │
│  │  │  • Active: 7mA, Standby: 50µA                            │ │ │
│  │  └──────────────────────────────────────────────────────────┘ │ │
│  │       │           │           │           │                   │ │
│  │       │ I2S       │ I2C       │ GPIO      │ Analog           │ │
│  │       ▼           ▼           ▼           ▼                   │ │
│  │  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐              │ │
│  │  │ CODEC  │  │  ANC   │  │ Touch  │  │  MIC   │              │ │
│  │  │ + AMP  │  │Feedback│  │ Sensor │  │ x2     │              │ │
│  │  │ (DAC)  │  │  MIC   │  │        │  │(FF+FB) │              │ │
│  │  └───┬────┘  └────────┘  └────────┘  └────────┘              │ │
│  │      │                                                        │ │
│  │      ▼                                                        │ │
│  │  ┌────────┐  ┌────────────────────────┐                      │ │
│  │  │Speaker │  │      Li-Ion 55mAh      │                      │ │
│  │  │ Driver │  │    (4-6 hours play)    │                      │ │
│  │  └────────┘  └────────────────────────┘                      │ │
│  │                                                                │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                   CHARGING CASE                                │ │
│  │                                                                │ │
│  │  ┌──────────────────────────────────────────────────────────┐ │ │
│  │  │                                                           │ │ │
│  │  │   ┌─────────────┐              ┌─────────────┐           │ │ │
│  │  │   │  Left Bay   │              │  Right Bay  │           │ │ │
│  │  │   │   ○ ○ ○     │              │    ○ ○ ○    │           │ │ │
│  │  │   │(Pogo pins)  │              │ (Pogo pins) │           │ │ │
│  │  │   └──────┬──────┘              └──────┬──────┘           │ │ │
│  │  │          │                            │                   │ │ │
│  │  │          └────────────┬───────────────┘                   │ │ │
│  │  │                       │                                   │ │ │
│  │  │               ┌───────▼───────┐                          │ │ │
│  │  │               │  Case MCU     │                          │ │ │
│  │  │               │ (STM32G0)     │                          │ │ │
│  │  │               │               │                          │ │ │
│  │  │               │ • Charge mgmt │                          │ │ │
│  │  │               │ • Fuel gauge  │                          │ │ │
│  │  │               │ • LED control │                          │ │ │
│  │  │               │ • Lid detect  │                          │ │ │
│  │  │               └───────┬───────┘                          │ │ │
│  │  │                       │                                   │ │ │
│  │  │  ┌────────────────────┴────────────────────────────────┐ │ │ │
│  │  │  │               Li-Ion 500mAh                          │ │ │ │
│  │  │  │            (5x full earbud charges)                  │ │ │ │
│  │  │  └─────────────────────────────────────────────────────┘ │ │ │
│  │  │                                                           │ │ │
│  │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐                  │ │ │
│  │  │  │ USB-C   │  │ Qi Coil │  │  LEDs   │                  │ │ │
│  │  │  │ Charge  │  │(Wireless│  │ Status  │                  │ │ │
│  │  │  └─────────┘  └─────────┘  └─────────┘                  │ │ │
│  │  │                                                           │ │ │
│  │  └──────────────────────────────────────────────────────────┘ │ │
│  │                                                                │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  AUDIO PROCESSING PIPELINE:                                          │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                                                                │ │
│  │  BT Audio ──▶ Decoder ──▶ EQ ──▶ ANC Mix ──▶ DAC ──▶ Speaker │ │
│  │  (aptX)      (SBC/AAC)  (5-band) (Hybrid)   (24-bit)         │ │
│  │                           │                                   │ │
│  │                    Feedforward ◀── FF Mic                     │ │
│  │                    Feedback ◀──── FB Mic                      │ │
│  │                                                                │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Smart Speaker Voice Processing

```
┌─────────────────────────────────────────────────────────────────────┐
│              SMART SPEAKER VOICE PIPELINE                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  HARDWARE ARCHITECTURE:                                              │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                                                                │ │
│  │   ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐       │ │
│  │   │MIC 1│  │MIC 2│  │MIC 3│  │MIC 4│  │MIC 5│  │MIC 6│       │ │
│  │   └──┬──┘  └──┬──┘  └──┬──┘  └──┬──┘  └──┬──┘  └──┬──┘       │ │
│  │      └────────┴────────┴───┬────┴────────┴────────┘          │ │
│  │                            │                                  │ │
│  │                     ┌──────▼──────┐                          │ │
│  │                     │ Audio Codec │                          │ │
│  │                     │  (8-ch PDM) │                          │ │
│  │                     └──────┬──────┘                          │ │
│  │                            │ I2S                              │ │
│  │   ┌────────────────────────▼─────────────────────────────┐   │ │
│  │   │                    MAIN SoC                           │   │ │
│  │   │               (MediaTek MT8167S)                      │   │ │
│  │   │                                                       │   │ │
│  │   │  ┌───────────────────────────────────────────────┐   │   │ │
│  │   │  │  Quad-core ARM Cortex-A35 @ 1.3GHz            │   │   │ │
│  │   │  │  + DSP for audio processing                   │   │   │ │
│  │   │  │  + 1GB RAM / 8GB eMMC                         │   │   │ │
│  │   │  │  + WiFi 802.11ac + BT 5.0                     │   │   │ │
│  │   │  └───────────────────────────────────────────────┘   │   │ │
│  │   │                                                       │   │ │
│  │   └───────────┬────────────────────────────┬─────────────┘   │ │
│  │               │                            │                  │ │
│  │        ┌──────▼──────┐              ┌──────▼──────┐          │ │
│  │        │  Class-D    │              │   LED Ring  │          │ │
│  │        │ Amplifier   │              │ (12 RGB)    │          │ │
│  │        │ (TAS5805M)  │              │             │          │ │
│  │        └──────┬──────┘              └─────────────┘          │ │
│  │               │                                               │ │
│  │        ┌──────▼──────┐                                       │ │
│  │        │   Speaker   │                                       │ │
│  │        │   Driver    │                                       │ │
│  │        └─────────────┘                                       │ │
│  │                                                                │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  VOICE PROCESSING PIPELINE:                                          │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                                                                │ │
│  │  ┌────────────┐   ┌────────────┐   ┌────────────┐            │ │
│  │  │ 6-ch Mic   │──▶│ Beamforming│──▶│   Echo     │            │ │
│  │  │   Array    │   │ (DSP)      │   │Cancellation│            │ │
│  │  └────────────┘   └────────────┘   └─────┬──────┘            │ │
│  │                                          │                    │ │
│  │                   ┌──────────────────────▼──────────────────┐│ │
│  │                   │                                          ││ │
│  │  ┌────────────┐   │   ┌────────────┐   ┌────────────┐       ││ │
│  │  │  Keyword   │◀──┤   │   Noise    │──▶│  Voice     │       ││ │
│  │  │  Spotting  │   │   │ Suppression│   │  Activity  │       ││ │
│  │  │ "Hey Google"   │   └────────────┘   │ Detection  │       ││ │
│  │  └─────┬──────┘   │                    └────────────┘       ││ │
│  │        │          └──────────────────────────────────────────┘│ │
│  │        │ Wake Event                                           │ │
│  │        ▼                                                      │ │
│  │  ┌────────────┐   ┌────────────┐   ┌────────────┐            │ │
│  │  │  Record    │──▶│  Stream to │──▶│   Cloud    │            │ │
│  │  │  Audio     │   │   Cloud    │   │    ASR     │            │ │
│  │  └────────────┘   └────────────┘   └─────┬──────┘            │ │
│  │                                          │                    │ │
│  │                                          ▼                    │ │
│  │  ┌────────────┐   ┌────────────┐   ┌────────────┐            │ │
│  │  │   Audio    │◀──│    TTS     │◀──│    NLU     │            │ │
│  │  │  Playback  │   │ (Text-to-  │   │ (Intent)   │            │ │
│  │  └────────────┘   │   Speech)  │   └────────────┘            │ │
│  │                   └────────────┘                              │ │
│  │                                                                │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Touch Interface Design

```c
/* ================================================================
 * Capacitive Touch Button Driver
 * Consumer electronics UI implementation
 * ================================================================ */

#include <stdint.h>
#include <stdbool.h>

/* Touch configuration */
#define NUM_TOUCH_BUTTONS       4
#define TOUCH_THRESHOLD         100     /* Counts above baseline */
#define LONG_PRESS_MS           800
#define DOUBLE_TAP_WINDOW_MS    300
#define DEBOUNCE_MS             50

/* Touch button state */
typedef enum {
    TOUCH_IDLE,
    TOUCH_PRESSED,
    TOUCH_HELD,
    TOUCH_RELEASED
} touch_state_t;

/* Touch event types */
typedef enum {
    EVENT_NONE,
    EVENT_TAP,
    EVENT_DOUBLE_TAP,
    EVENT_LONG_PRESS,
    EVENT_SWIPE_LEFT,
    EVENT_SWIPE_RIGHT
} touch_event_t;

/* Touch button data */
typedef struct {
    uint16_t raw_value;
    uint16_t baseline;
    uint16_t delta;
    touch_state_t state;
    uint32_t press_start_time;
    uint32_t last_tap_time;
    uint8_t tap_count;
} touch_button_t;

/* Global touch state */
static touch_button_t buttons[NUM_TOUCH_BUTTONS];

/* ================================================================
 * Initialize Touch Sensing
 * ================================================================ */
void touch_init(void) {
    /* Configure touch sensing hardware */
    /* (Using STM32 TSC or dedicated touch controller) */
    
    for (int i = 0; i < NUM_TOUCH_BUTTONS; i++) {
        buttons[i].state = TOUCH_IDLE;
        buttons[i].baseline = 0;
        buttons[i].tap_count = 0;
    }
    
    /* Calibrate baseline */
    touch_calibrate();
}

/* ================================================================
 * Calibrate Touch Baseline
 * ================================================================ */
void touch_calibrate(void) {
    /* Average multiple readings for stable baseline */
    for (int i = 0; i < NUM_TOUCH_BUTTONS; i++) {
        uint32_t sum = 0;
        for (int j = 0; j < 16; j++) {
            sum += touch_read_raw(i);
            delay_ms(5);
        }
        buttons[i].baseline = sum / 16;
    }
}

/* ================================================================
 * Process Touch Input (call at 100Hz)
 * ================================================================ */
touch_event_t touch_process(uint8_t button_id, uint32_t timestamp_ms) {
    touch_button_t *btn = &buttons[button_id];
    touch_event_t event = EVENT_NONE;
    
    /* Read current value */
    btn->raw_value = touch_read_raw(button_id);
    
    /* Calculate delta from baseline */
    if (btn->raw_value > btn->baseline) {
        btn->delta = btn->raw_value - btn->baseline;
    } else {
        btn->delta = 0;
        /* Slowly adapt baseline */
        btn->baseline = (btn->baseline * 255 + btn->raw_value) / 256;
    }
    
    /* State machine */
    bool touched = (btn->delta > TOUCH_THRESHOLD);
    
    switch (btn->state) {
        case TOUCH_IDLE:
            if (touched) {
                btn->state = TOUCH_PRESSED;
                btn->press_start_time = timestamp_ms;
            }
            /* Check for double-tap timeout */
            if (btn->tap_count > 0 && 
                (timestamp_ms - btn->last_tap_time) > DOUBLE_TAP_WINDOW_MS) {
                if (btn->tap_count == 1) {
                    event = EVENT_TAP;
                }
                btn->tap_count = 0;
            }
            break;
            
        case TOUCH_PRESSED:
            if (!touched) {
                /* Released - check for tap */
                uint32_t press_duration = timestamp_ms - btn->press_start_time;
                if (press_duration < LONG_PRESS_MS) {
                    btn->tap_count++;
                    btn->last_tap_time = timestamp_ms;
                    
                    if (btn->tap_count >= 2) {
                        event = EVENT_DOUBLE_TAP;
                        btn->tap_count = 0;
                    }
                }
                btn->state = TOUCH_IDLE;
            } else {
                /* Still pressed - check for long press */
                uint32_t press_duration = timestamp_ms - btn->press_start_time;
                if (press_duration >= LONG_PRESS_MS) {
                    btn->state = TOUCH_HELD;
                    btn->tap_count = 0;
                    event = EVENT_LONG_PRESS;
                }
            }
            break;
            
        case TOUCH_HELD:
            if (!touched) {
                btn->state = TOUCH_IDLE;
            }
            break;
            
        default:
            btn->state = TOUCH_IDLE;
            break;
    }
    
    return event;
}

/* ================================================================
 * Swipe Detection (for slider interfaces)
 * ================================================================ */
typedef struct {
    int16_t start_x;
    int16_t current_x;
    uint32_t start_time;
    bool active;
} swipe_detector_t;

touch_event_t detect_swipe(swipe_detector_t *swipe, 
                           int16_t touch_x, 
                           bool touching,
                           uint32_t timestamp_ms) {
    touch_event_t event = EVENT_NONE;
    
    if (touching && !swipe->active) {
        /* Start new swipe */
        swipe->start_x = touch_x;
        swipe->start_time = timestamp_ms;
        swipe->active = true;
    } else if (touching && swipe->active) {
        /* Update current position */
        swipe->current_x = touch_x;
    } else if (!touching && swipe->active) {
        /* End swipe - check direction */
        int16_t delta_x = swipe->current_x - swipe->start_x;
        uint32_t duration = timestamp_ms - swipe->start_time;
        
        /* Minimum swipe distance and maximum time */
        if (duration < 500 && abs(delta_x) > 50) {
            if (delta_x > 0) {
                event = EVENT_SWIPE_RIGHT;
            } else {
                event = EVENT_SWIPE_LEFT;
            }
        }
        
        swipe->active = false;
    }
    
    return event;
}
```

---

## Cost Optimization Strategies

```
┌─────────────────────────────────────────────────────────────────────┐
│              BOM COST OPTIMIZATION                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  COMPONENT COST BREAKDOWN (Typical Fitness Tracker):                │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │ Component           │ Low-End  │ Mid-Range│ Premium │         │ │
│  ├─────────────────────┼──────────┼──────────┼─────────┼─────────┤ │
│  │ MCU + BLE           │   $1.50  │   $3.00  │  $5.00  │         │ │
│  │ Display (OLED/LCD)  │   $1.00  │   $4.00  │ $12.00  │         │ │
│  │ Accelerometer       │   $0.30  │   $0.80  │  $2.00  │         │ │
│  │ Heart Rate (PPG)    │   $0.50  │   $1.50  │  $3.00  │         │ │
│  │ Battery (LiPo)      │   $0.80  │   $1.20  │  $2.00  │         │ │
│  │ PMIC + Charger      │   $0.40  │   $0.80  │  $1.50  │         │ │
│  │ PCB                 │   $0.50  │   $0.80  │  $1.20  │         │ │
│  │ Passives/Connectors │   $0.30  │   $0.50  │  $0.80  │         │ │
│  │ Enclosure + Strap   │   $1.50  │   $3.00  │  $6.00  │         │ │
│  │ Assembly            │   $1.00  │   $1.50  │  $2.50  │         │ │
│  ├─────────────────────┼──────────┼──────────┼─────────┼─────────┤ │
│  │ TOTAL BOM           │   $7.80  │  $17.10  │ $36.00  │         │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  COST REDUCTION TECHNIQUES:                                          │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                                                                │ │
│  │  1. INTEGRATION                                                │ │
│  │     • Use SoC with integrated BLE (vs discrete radio)         │ │
│  │     • Integrated PMIC with fuel gauge                         │ │
│  │     • Combo sensors (accel + gyro + mag)                      │ │
│  │                                                                │ │
│  │  2. COMPONENT SELECTION                                       │ │
│  │     • Use mature, high-volume parts                           │ │
│  │     • Consider Chinese alternatives (careful with quality)    │ │
│  │     • Design for common passives (0402, 5% tolerance)         │ │
│  │                                                                │ │
│  │  3. PCB OPTIMIZATION                                          │ │
│  │     • Minimize layers (2-layer if possible)                   │ │
│  │     • Standard PCB materials (FR4)                            │ │
│  │     • Panelize for production                                 │ │
│  │                                                                │ │
│  │  4. MANUFACTURING                                             │ │
│  │     • Design for automated assembly                           │ │
│  │     • Minimize unique components                              │ │
│  │     • Use standard packages (avoid BGA for simple designs)    │ │
│  │                                                                │ │
│  │  5. TESTING                                                   │ │
│  │     • Design for test (test points, JTAG)                    │ │
│  │     • In-circuit test capability                              │ │
│  │     • Automated functional test                               │ │
│  │                                                                │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  VOLUME PRICING EFFECT:                                              │
│                                                                      │
│  Price/Unit                                                          │
│      │                                                               │
│   $15├──●                                                            │
│      │    ╲                                                          │
│   $12├     ●                                                         │
│      │       ╲                                                       │
│    $9├         ●                                                     │
│      │           ╲                                                   │
│    $6├              ●──────●──────●                                  │
│      │                                                               │
│      └────┬─────┬─────┬─────┬─────┬─────▶                           │
│         1K    10K   100K    1M    10M   Volume                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## User Experience (UX) Design

```
┌─────────────────────────────────────────────────────────────────────┐
│              EMBEDDED UX DESIGN PRINCIPLES                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  RESPONSE TIME REQUIREMENTS:                                         │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │ Interaction          │ Target    │ Maximum  │ User Perception │ │
│  ├──────────────────────┼───────────┼──────────┼─────────────────┤ │
│  │ Button press         │ <10ms     │ 100ms    │ Instantaneous   │ │
│  │ Screen update        │ <16ms     │ 50ms     │ Smooth          │ │
│  │ Voice response       │ <200ms    │ 500ms    │ Conversational  │ │
│  │ Network operation    │ <1s       │ 3s       │ Brief wait      │ │
│  │ Startup time         │ <1s       │ 3s       │ Acceptable      │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  FEEDBACK MECHANISMS:                                                │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                                                                │ │
│  │   VISUAL             AUDITORY           HAPTIC                │ │
│  │   ┌──────────┐      ┌──────────┐      ┌──────────┐           │ │
│  │   │  LED     │      │  Beep/   │      │  Vibrate │           │ │
│  │   │  Display │      │  Tone    │      │  Motor   │           │ │
│  │   │  Icons   │      │  Voice   │      │  Pattern │           │ │
│  │   └──────────┘      └──────────┘      └──────────┘           │ │
│  │                                                                │ │
│  │   Use case:          Use case:         Use case:              │ │
│  │   • Status info      • Alerts          • Confirmation         │ │
│  │   • Progress         • Confirmations   • Notifications        │ │
│  │   • Navigation       • Errors          • Wearable alerts      │ │
│  │                                                                │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  BATTERY INDICATOR PATTERNS:                                         │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                                                                │ │
│  │   100%  ████████████  Full (Green)                            │ │
│  │    75%  █████████░░░  Good (Green)                            │ │
│  │    50%  ██████░░░░░░  Medium (Yellow)                         │ │
│  │    25%  ███░░░░░░░░░  Low (Orange)                            │ │
│  │    10%  █░░░░░░░░░░░  Critical (Red, blinking)                │ │
│  │     5%  ░░░░░░░░░░░░  Shutdown warning                        │ │
│  │                                                                │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  SLEEP/WAKE TRANSITIONS:                                             │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                                                                │ │
│  │   ACTIVE ───────────────▶ SCREEN OFF ───────────────▶ SLEEP  │ │
│  │            Timeout (30s)              Timeout (5min)          │ │
│  │                                                                │ │
│  │   SLEEP ────────────────────────────────────────────▶ ACTIVE  │ │
│  │          Button press / Raise-to-wake / Touch                 │ │
│  │                                                                │ │
│  │   Wake animations:                                             │ │
│  │   • Fade in display (100-200ms)                               │ │
│  │   • Show clock first, then other widgets                      │ │
│  │   • Delay network sync until user interaction                 │ │
│  │                                                                │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **Wearables** | Battery-powered, compact devices (trackers, watches, earbuds) |
| **Step Counter** | Accelerometer-based with peak detection algorithm |
| **TWS Earbuds** | True Wireless Stereo with BT SoC, ANC, codec |
| **Smart Speaker** | Mic array, beamforming, wake word, cloud ASR |
| **Touch Interface** | Capacitive sensing with gesture detection |
| **BOM Optimization** | Integration, volume pricing, component selection |
| **UX Latency** | <100ms for button, <16ms for display refresh |
| **Haptic Feedback** | Vibration motor patterns for user confirmation |
| **Power States** | Active → Screen Off → Deep Sleep transitions |

---

## Quick Revision Questions

1. **Design a fitness tracker power budget targeting 7-day battery life with 100mAh battery. What average current can you consume?**

2. **Explain the step counting algorithm. What parameters need tuning for accuracy?**

3. **What are the key components in a TWS earbud system? How does ANC (Active Noise Cancellation) work?**

4. **Describe the voice processing pipeline in a smart speaker from microphone to cloud response.**

5. **What strategies can reduce BOM cost by 30%? What are the tradeoffs?**

6. **Define response time requirements for different user interactions. Why is <100ms important for button presses?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Chapter 8.3: IoT Applications](03-iot-applications.md) | [Unit 8: Case Studies](README.md) | [Main Index](../README.md) |

---
