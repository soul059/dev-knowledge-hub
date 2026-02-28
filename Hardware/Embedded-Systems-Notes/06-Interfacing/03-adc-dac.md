# Chapter 6.3: ADC and DAC

## Introduction

Analog-to-Digital Converters (ADC) and Digital-to-Analog Converters (DAC) bridge the gap between the continuous analog world and digital processing. This chapter covers ADC/DAC architectures, configuration, and sensor interfacing.

---

## ADC Fundamentals

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ADC CONVERSION PROCESS                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Analog Signal → Sample & Hold → Quantization → Digital Code       │
│                                                                      │
│  Analog                                                              │
│  Voltage                                                             │
│  Vref ─┬─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─              │
│        │      ╱╲                                                    │
│        │    ╱    ╲        ╱╲                                        │
│        │  ╱        ╲    ╱    ╲                                      │
│        │╱            ╲╱        ╲                                    │
│      0 └───────────────────────────▶ Time                           │
│                                                                      │
│  Sampled                                                             │
│  Values   ●    ●     ●    ●    ●     ●    ●    ●                   │
│           │    │     │    │    │     │    │    │                   │
│           ▼    ▼     ▼    ▼    ▼     ▼    ▼    ▼                   │
│                                                                      │
│  Digital                                                             │
│  Codes   150  220   255  180   90   30    60   120                  │
│                                                                      │
│  KEY PARAMETERS:                                                     │
│                                                                      │
│  Resolution (n bits): Number of discrete levels = 2^n              │
│  • 8-bit ADC: 256 levels                                           │
│  • 10-bit ADC: 1024 levels                                         │
│  • 12-bit ADC: 4096 levels                                         │
│                                                                      │
│  LSB voltage = Vref / 2^n                                           │
│  • 12-bit, 3.3V ref: LSB = 3.3V / 4096 = 0.806 mV                  │
│                                                                      │
│  Conversion: Digital_Value = (Vin × 2^n) / Vref                     │
│  Voltage:    Vin = (Digital_Value × Vref) / 2^n                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ADC Architectures

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SUCCESSIVE APPROXIMATION (SAR) ADC                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Most common in MCUs - good balance of speed and accuracy          │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────┐       │
│  │                                                          │       │
│  │   Vin ──▶ S/H ──▶ ┌─────┐                               │       │
│  │                   │  -  │                               │       │
│  │                   │     ├──▶ Comparator ──▶ SAR Logic   │       │
│  │            ┌─────▶│  +  │         │                     │       │
│  │            │      └─────┘         │                     │       │
│  │            │                      ▼                     │       │
│  │      ┌─────┴─────┐         ┌───────────┐               │       │
│  │      │    DAC    │◀────────│ SAR       │──▶ Output     │       │
│  │      │ (internal)│         │ Register  │               │       │
│  │      └───────────┘         └───────────┘               │       │
│  │                                                          │       │
│  └──────────────────────────────────────────────────────────┘       │
│                                                                      │
│  CONVERSION PROCESS (12-bit example):                               │
│                                                                      │
│  Step │ Test Value │ Compare │ Result │ Output                      │
│  ─────┼────────────┼─────────┼────────┼────────                     │
│    1  │ 1000 0000  │ Vin >?  │  Yes   │ 1xxx xxxx xxxx              │
│    2  │ 1100 0000  │ Vin >?  │  No    │ 10xx xxxx xxxx              │
│    3  │ 1010 0000  │ Vin >?  │  Yes   │ 101x xxxx xxxx              │
│   ... │    ...     │   ...   │  ...   │    ...                      │
│   12  │ 1010 1001  │ Vin >?  │  Yes   │ 1010 1001 0101              │
│                                                                      │
│  N bits = N clock cycles                                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SIGMA-DELTA (ΔΣ) ADC                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  High resolution (16-24 bits), slower, great for sensors           │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────┐       │
│  │                                                          │       │
│  │  Vin ──▶(+)──▶ Integrator ──▶ Comparator ──▶ Digital    │       │
│  │          ↑                        │          Filter     │       │
│  │          │                        │             │       │       │
│  │          └────────────────────────┘             │       │       │
│  │               1-bit DAC                         ▼       │       │
│  │                                           High-res      │       │
│  │                                           Output        │       │
│  └──────────────────────────────────────────────────────────┘       │
│                                                                      │
│  • Oversampling: Sample much faster than Nyquist                   │
│  • Noise shaping: Push noise to high frequencies                   │
│  • Digital filter removes noise, increases resolution              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### ADC Comparison

| Type | Resolution | Speed | Use Case |
|------|------------|-------|----------|
| **Flash** | 4-8 bits | Very fast (GHz) | Video, RF |
| **SAR** | 8-16 bits | Medium (MSPS) | General purpose |
| **Sigma-Delta** | 16-24 bits | Slow (SPS) | Precision sensors |
| **Pipeline** | 10-16 bits | Fast (MSPS) | Data acquisition |

---

## ADC Configuration

```c
// ADC initialization (STM32)
void adc_init(void) {
    // Enable clocks
    RCC->APB2ENR |= RCC_APB2ENR_ADC1EN;
    RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;
    
    // Configure PA0 as analog input
    GPIOA->MODER |= (3 << (0 * 2));  // Analog mode
    
    // Configure ADC
    ADC1->CR1 = 0;                    // 12-bit resolution
    ADC1->CR2 = 0;
    ADC1->SQR3 = 0;                   // Channel 0 first in sequence
    ADC1->SMPR2 = (7 << 0);           // 480 cycles sample time
    
    // Enable ADC
    ADC1->CR2 |= ADC_CR2_ADON;
    
    // Wait for ADC to stabilize
    delay_us(10);
}

// Single conversion (polling)
uint16_t adc_read(uint8_t channel) {
    // Select channel
    ADC1->SQR3 = channel;
    
    // Start conversion
    ADC1->CR2 |= ADC_CR2_SWSTART;
    
    // Wait for conversion complete
    while (!(ADC1->SR & ADC_SR_EOC));
    
    // Read result
    return ADC1->DR;
}

// Convert to voltage
float adc_to_voltage(uint16_t adc_value) {
    const float VREF = 3.3f;
    const float ADC_MAX = 4095.0f;  // 12-bit
    return (adc_value * VREF) / ADC_MAX;
}

// Example usage
void read_sensor(void) {
    uint16_t raw = adc_read(0);       // Channel 0
    float voltage = adc_to_voltage(raw);
    printf("ADC: %d, Voltage: %.3f V\n", raw, voltage);
}
```

---

## Signal Conditioning

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SENSOR SIGNAL CONDITIONING                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Many sensors output signals that need conditioning:               │
│                                                                      │
│  VOLTAGE DIVIDER (resistive sensors):                              │
│                                                                      │
│  VCC ───┬                     VCC ───┬                              │
│         │                            │                              │
│        ─┴─ R1                       ─┴─ NTC                         │
│        ─┬─                          ─┬─ Thermistor                  │
│         │                            │                              │
│         ├───▶ ADC                    ├───▶ ADC                      │
│         │                            │                              │
│        ─┴─ R2                       ─┴─ R                           │
│        ─┬─                          ─┬─ (fixed)                     │
│         │                            │                              │
│        GND                          GND                             │
│                                                                      │
│  Vout = VCC × R2 / (R1 + R2)                                       │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  AMPLIFIER (weak signals):                                          │
│                                                                      │
│              ┌─────────────┐                                        │
│  Sensor ──┬──┤    Op-Amp   ├──▶ ADC                                │
│           │  │   (Gain)    │                                        │
│          ─┴─ └──────┬──────┘                                        │
│          GND        │                                               │
│                    Rf/Rin = Gain                                    │
│                                                                      │
│  ────────────────────────────────────────────────────────────────   │
│                                                                      │
│  LOW-PASS FILTER (noise reduction):                                 │
│                                                                      │
│  Signal ───[R]───┬──▶ ADC                                          │
│                  │                                                   │
│                 ─┴─ C                                                │
│                 ─┬─                                                  │
│                  │                                                   │
│                 GND                                                  │
│                                                                      │
│  fc = 1 / (2π × R × C)                                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Temperature Sensor Example

```c
// NTC Thermistor (10kΩ at 25°C)
// Using voltage divider with 10kΩ fixed resistor

#define R_FIXED       10000.0f    // 10kΩ
#define R_NOMINAL     10000.0f    // NTC at 25°C
#define T_NOMINAL     25.0f       // 25°C
#define B_COEFFICIENT 3950.0f     // B value from datasheet
#define VCC           3.3f
#define ADC_MAX       4095.0f

float read_temperature_ntc(uint8_t channel) {
    // Read ADC value
    uint16_t adc_value = adc_read(channel);
    
    // Calculate NTC resistance
    // Vout = VCC × R_fixed / (R_ntc + R_fixed)
    // R_ntc = R_fixed × (VCC/Vout - 1) = R_fixed × (ADC_MAX/adc - 1)
    float resistance = R_FIXED * (ADC_MAX / adc_value - 1.0f);
    
    // Steinhart-Hart simplified (B-parameter equation)
    // 1/T = 1/T0 + (1/B) × ln(R/R0)
    float steinhart;
    steinhart = resistance / R_NOMINAL;          // R/R0
    steinhart = logf(steinhart);                 // ln(R/R0)
    steinhart /= B_COEFFICIENT;                  // (1/B) × ln(R/R0)
    steinhart += 1.0f / (T_NOMINAL + 273.15f);   // + 1/T0
    steinhart = 1.0f / steinhart;                // Invert
    steinhart -= 273.15f;                        // Kelvin to Celsius
    
    return steinhart;
}

// Linear approximation for LM35 (10 mV/°C)
float read_temperature_lm35(uint8_t channel) {
    uint16_t adc_value = adc_read(channel);
    float voltage = (adc_value * VCC) / ADC_MAX;
    float temperature = voltage * 100.0f;  // 10 mV/°C
    return temperature;
}
```

---

## DAC Fundamentals

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DAC OPERATION                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Digital Code → DAC → Analog Voltage                               │
│                                                                      │
│  Vout = (Digital_Value × Vref) / 2^n                               │
│                                                                      │
│  12-bit DAC, Vref = 3.3V:                                          │
│  • Code 0    → 0.000 V                                             │
│  • Code 2048 → 1.650 V (mid-scale)                                 │
│  • Code 4095 → 3.299 V                                             │
│                                                                      │
│  DAC ARCHITECTURES:                                                  │
│                                                                      │
│  R-2R Ladder:                                                       │
│                                                                      │
│  MSB ───┬───[2R]───┬───[2R]───┬───[2R]───┬───[2R]──▶ LSB           │
│         │          │          │          │                          │
│        [R]        [R]        [R]        [R]                         │
│         │          │          │          │                          │
│         └──────────┴──────────┴──────────┴──────▶ Vout             │
│                                                                      │
│  APPLICATIONS:                                                       │
│  • Audio output                                                     │
│  • Waveform generation                                              │
│  • Programmable voltage reference                                   │
│  • Motor control (current reference)                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### DAC Implementation

```c
// DAC initialization (STM32 with built-in DAC)
void dac_init(void) {
    // Enable DAC clock
    RCC->APB1ENR |= RCC_APB1ENR_DACEN;
    RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;
    
    // Configure PA4 as analog (DAC1 output)
    GPIOA->MODER |= (3 << (4 * 2));
    
    // Enable DAC channel 1
    DAC->CR |= DAC_CR_EN1;
}

// Set DAC output value (12-bit)
void dac_write(uint16_t value) {
    if (value > 4095) value = 4095;
    DAC->DHR12R1 = value;
}

// Set DAC voltage
void dac_set_voltage(float voltage) {
    const float VREF = 3.3f;
    if (voltage < 0) voltage = 0;
    if (voltage > VREF) voltage = VREF;
    
    uint16_t value = (uint16_t)((voltage / VREF) * 4095.0f);
    dac_write(value);
}

// Generate sine wave using lookup table
const uint16_t sine_table[256] = {
    2048, 2098, 2148, 2198, 2248, 2298, 2347, 2396,
    // ... 256 values for one sine cycle
    2445, 2493, 2541, 2588, 2635, 2681, 2726, 2771,
    // (values range from 0 to 4095)
};

volatile uint8_t sine_index = 0;

void TIM6_IRQHandler(void) {
    if (TIM6->SR & TIM_SR_UIF) {
        TIM6->SR &= ~TIM_SR_UIF;
        
        dac_write(sine_table[sine_index++]);
        // sine_index auto-wraps at 256
    }
}

// Generate 1 kHz sine wave
// Timer rate = 256 samples × 1000 Hz = 256 kHz
void start_sine_wave(void) {
    // Configure TIM6 for 256 kHz interrupt
    TIM6->PSC = 0;
    TIM6->ARR = (72000000 / 256000) - 1;
    TIM6->DIER |= TIM_DIER_UIE;
    TIM6->CR1 |= TIM_CR1_CEN;
    
    NVIC_EnableIRQ(TIM6_DAC_IRQn);
}
```

---

## DMA with ADC

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ADC WITH DMA                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  DMA transfers ADC results to memory without CPU                   │
│                                                                      │
│  ┌─────────┐    ┌─────────┐    ┌─────────────────────────────┐     │
│  │   ADC   │───▶│   DMA   │───▶│        Memory Buffer        │     │
│  │         │    │Controller│   │  [0] [1] [2] [3] ... [N-1] │     │
│  └─────────┘    └─────────┘    └─────────────────────────────┘     │
│       ↑              │                       ↑                      │
│       │              │                       │                      │
│   Timer            Interrupt              CPU reads                 │
│   Trigger          when done              processed data            │
│                                                                      │
│  Benefits:                                                           │
│  • High sampling rates without CPU overhead                        │
│  • Continuous sampling with circular buffer                        │
│  • Multi-channel scanning                                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

```c
// ADC with DMA - Continuous multi-channel sampling
#define ADC_BUFFER_SIZE  100
#define NUM_CHANNELS     3

uint16_t adc_buffer[ADC_BUFFER_SIZE * NUM_CHANNELS];

void adc_dma_init(void) {
    // Configure DMA
    DMA2_Stream0->CR = 0;
    DMA2_Stream0->PAR = (uint32_t)&ADC1->DR;
    DMA2_Stream0->M0AR = (uint32_t)adc_buffer;
    DMA2_Stream0->NDTR = ADC_BUFFER_SIZE * NUM_CHANNELS;
    
    DMA2_Stream0->CR = DMA_SxCR_CHSEL_0 |     // Channel 0
                       DMA_SxCR_MSIZE_0 |      // 16-bit memory
                       DMA_SxCR_PSIZE_0 |      // 16-bit peripheral
                       DMA_SxCR_MINC |         // Memory increment
                       DMA_SxCR_CIRC |         // Circular mode
                       DMA_SxCR_TCIE;          // Transfer complete interrupt
    
    DMA2_Stream0->CR |= DMA_SxCR_EN;
    
    // Configure ADC for scan mode
    ADC1->CR1 |= ADC_CR1_SCAN;
    ADC1->CR2 |= ADC_CR2_DMA | ADC_CR2_DDS | ADC_CR2_CONT;
    
    // Configure channels 0, 1, 2
    ADC1->SQR1 = (NUM_CHANNELS - 1) << 20;  // 3 conversions
    ADC1->SQR3 = (0 << 0) | (1 << 5) | (2 << 10);
    
    // Start conversion
    ADC1->CR2 |= ADC_CR2_SWSTART;
}

void DMA2_Stream0_IRQHandler(void) {
    if (DMA2->LISR & DMA_LISR_TCIF0) {
        DMA2->LIFCR = DMA_LIFCR_CTCIF0;
        
        // Buffer full - process data
        process_adc_data(adc_buffer, ADC_BUFFER_SIZE * NUM_CHANNELS);
    }
}
```

---

## Summary Table

| Feature | ADC | DAC |
|---------|-----|-----|
| **Direction** | Analog → Digital | Digital → Analog |
| **Resolution** | 8-24 bits typical | 8-16 bits typical |
| **Speed** | SPS to MSPS | Fast (output limited) |
| **Inputs/Outputs** | Multiple channels | Usually 1-2 channels |
| **Key Parameter** | Sampling rate, SNR | Settling time, INL/DNL |

---

## Quick Revision Questions

1. **Calculate the LSB voltage for a 10-bit ADC with 5V reference. What voltage gives code 512?**

2. **Explain the difference between SAR and sigma-delta ADCs. Which is better for audio?**

3. **Why is sample rate important? What happens if you sample a 5 kHz signal at 8 kHz?**

4. **An NTC thermistor has 10kΩ at 25°C, 5kΩ at 40°C. Design a voltage divider for ADC input.**

5. **What is DMA and why is it useful for ADC applications?**

6. **A 12-bit DAC outputs 0-3.3V. What digital value produces 1.65V output?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Chapter 6.2: Timers and PWM](02-timers-pwm.md) | [Unit Index](README.md) | [Chapter 6.4: Display Interfacing](04-display-interfacing.md) |

---
