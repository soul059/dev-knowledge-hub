# Chapter 7.5: Power Management

## Introduction

Power management is critical for embedded systems, especially battery-powered and IoT devices. This chapter covers techniques for analyzing power consumption, implementing low-power modes, managing batteries, and designing efficient power supplies. Good power management extends battery life, reduces heat, and enables new applications.

---

## Power Budget Analysis

```
┌─────────────────────────────────────────────────────────────────────┐
│              POWER BUDGET METHODOLOGY                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  STEP 1: IDENTIFY OPERATING MODES                                   │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Mode          │ Duration    │ Frequency     │ Description    │   │
│  ├───────────────┼─────────────┼───────────────┼────────────────┤   │
│  │ Active        │ 10 ms       │ 1/min         │ Measure+TX     │   │
│  │ Light Sleep   │ 1 sec       │ Continuous    │ RTC active     │   │
│  │ Deep Sleep    │ 59 sec      │ 1/min         │ RAM retained   │   │
│  │ Standby       │ Hours       │ Rare          │ Waiting for    │   │
│  │               │             │               │ external event │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  STEP 2: MEASURE/ESTIMATE CURRENT FOR EACH MODE                     │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Mode          │ MCU       │ Radio    │ Sensor   │ TOTAL     │   │
│  ├───────────────┼───────────┼──────────┼──────────┼───────────┤   │
│  │ Active        │ 15 mA     │ 25 mA    │ 2 mA     │ 42 mA     │   │
│  │ Light Sleep   │ 1 mA      │ 0        │ 0        │ 1 mA      │   │
│  │ Deep Sleep    │ 5 µA      │ 0        │ 0        │ 5 µA      │   │
│  │ Standby       │ 0.5 µA    │ 0        │ 0        │ 0.5 µA    │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  STEP 3: CALCULATE AVERAGE CURRENT                                  │
│                                                                      │
│  I_avg = Σ(I_mode × t_mode) / T_total                               │
│                                                                      │
│  Example for 1-minute cycle:                                        │
│  • Active:     42 mA × 10 ms    = 0.42 mA·s                        │
│  • Light Sleep: 1 mA × 1 s      = 1.00 mA·s                        │
│  • Deep Sleep:  5 µA × 59 s     = 0.295 mA·s                       │
│                                                                      │
│  I_avg = (0.42 + 1.00 + 0.295) / 60 s = 28.6 µA                    │
│                                                                      │
│  STEP 4: CALCULATE BATTERY LIFE                                     │
│                                                                      │
│  Battery: 2× AA = 2× 2500 mAh = 5000 mAh                           │
│  (Note: Series connection, capacity doesn't double)                 │
│  Usable capacity (80%): 2000 mAh                                    │
│                                                                      │
│  Life = 2000 mAh / 0.0286 mA = 69,930 hours ≈ 8 years              │
│                                                                      │
│  Reality check: Include self-discharge, temperature effects        │
│  Practical estimate: ~3-5 years                                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Power Measurement

```
┌─────────────────────────────────────────────────────────────────────┐
│              POWER MEASUREMENT TECHNIQUES                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. SHUNT RESISTOR METHOD (Basic)                                   │
│  ┌────────────────────────────────────────────────────────────────┐│
│  │                                                                 ││
│  │   VCC ────┬───[R_shunt]───┬─── To Device                       ││
│  │           │               │                                     ││
│  │           │    V = IR     │                                     ││
│  │           └───────┬───────┘                                     ││
│  │                   │                                              ││
│  │              Oscilloscope                                       ││
│  │                                                                 ││
│  │   R_shunt = 1Ω typical                                         ││
│  │   1mA → 1mV across resistor                                    ││
│  │                                                                 ││
│  │   Issues: Voltage drop affects device, µA hard to measure      ││
│  └────────────────────────────────────────────────────────────────┘│
│                                                                      │
│  2. CURRENT SENSE AMPLIFIER                                         │
│  ┌────────────────────────────────────────────────────────────────┐│
│  │                                                                 ││
│  │   VCC ──┬──[R=0.1Ω]──┬── To Device                             ││
│  │         │            │                                          ││
│  │         └──[INA219]──┘                                         ││
│  │              │                                                  ││
│  │         I2C to MCU                                              ││
│  │                                                                 ││
│  │   INA219: ±3.2A range, 0.8mA resolution                        ││
│  │   INA226: ±36V, better for low current                         ││
│  │                                                                 ││
│  └────────────────────────────────────────────────────────────────┘│
│                                                                      │
│  3. PROFESSIONAL TOOLS                                              │
│  ┌────────────────────────────────────────────────────────────────┐│
│  │ Tool               │ Range          │ Features                 ││
│  ├────────────────────┼────────────────┼──────────────────────────┤│
│  │ Nordic PPK2        │ 200nA - 1A     │ nRF dev, USB power       ││
│  │ Joulescope         │ 2nA - 3A       │ PC software, capture     ││
│  │ Keysight N6705C    │ nA - 50A       │ Lab-grade, $$$$          ││
│  │ Otii Arc           │ 100nA - 3A     │ Timeline, triggers       ││
│  │ µCurrent           │ 1nA - 1A       │ Low-cost, manual range   ││
│  └────────────────────────────────────────────────────────────────┘│
│                                                                      │
│  MEASUREMENT BEST PRACTICES:                                        │
│  • Remove debug probe (consumes power)                              │
│  • Measure at expected operating temperature                        │
│  • Capture transient peaks (RF transmission)                        │
│  • Account for all current paths (LEDs, pull-ups)                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Low-Power Design Techniques

### MCU Power Modes

```
┌─────────────────────────────────────────────────────────────────────┐
│              ARM CORTEX-M LOW POWER MODES                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │              POWER MODE HIERARCHY                             │  │
│  │                                                                │  │
│  │  RUN MODE (Full speed)                                        │  │
│  │  ├── CPU running                                              │  │
│  │  ├── All peripherals active                                   │  │
│  │  └── Current: 10-100 mA                                       │  │
│  │      │                                                         │  │
│  │      ▼ WFI instruction                                        │  │
│  │  SLEEP MODE                                                   │  │
│  │  ├── CPU stopped, peripherals running                         │  │
│  │  ├── Wake: any interrupt                                      │  │
│  │  └── Current: 1-10 mA                                         │  │
│  │      │                                                         │  │
│  │      ▼ SLEEPDEEP + WFI                                        │  │
│  │  STOP MODE (STM32) / Low-Power Sleep                          │  │
│  │  ├── Most clocks stopped                                      │  │
│  │  ├── RAM retained                                             │  │
│  │  ├── Wake: RTC, GPIO, limited peripherals                     │  │
│  │  └── Current: 1-100 µA                                        │  │
│  │      │                                                         │  │
│  │      ▼                                                         │  │
│  │  STANDBY MODE                                                 │  │
│  │  ├── Main regulator off                                       │  │
│  │  ├── RAM lost (some backup RAM)                               │  │
│  │  ├── Wake: RTC, WKUP pins                                     │  │
│  │  └── Current: 0.3-3 µA                                        │  │
│  │      │                                                         │  │
│  │      ▼                                                         │  │
│  │  SHUTDOWN/OFF                                                 │  │
│  │  ├── All power off                                            │  │
│  │  ├── Wake: WKUP pins only                                     │  │
│  │  └── Current: 10-300 nA                                       │  │
│  │                                                                │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  WAKE-UP SOURCES BY MODE:                                           │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Mode      │ Interrupt │ RTC  │ GPIO │ UART │ ADC │ Timer   │  │
│  ├───────────┼───────────┼──────┼──────┼──────┼─────┼─────────┤  │
│  │ Sleep     │    ✓      │  ✓   │  ✓   │  ✓   │  ✓  │   ✓     │  │
│  │ Stop      │    ─      │  ✓   │  ✓   │  ─   │  ─  │   ─     │  │
│  │ Standby   │    ─      │  ✓   │  ✓*  │  ─   │  ─  │   ─     │  │
│  └──────────────────────────────────────────────────────────────┘  │
│  * Only WKUP pins, not all GPIO                                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Low-Power Code Implementation

```c
/* ================================================================
 * Low-Power Mode Implementation (STM32)
 * ================================================================ */

#include "stm32l4xx.h"

/* ================================================================
 * Enter Sleep Mode (WFI - Wait For Interrupt)
 * Wakes on any interrupt
 * ================================================================ */
void enter_sleep_mode(void) {
    /* Clear SLEEPDEEP bit to ensure regular sleep */
    SCB->SCR &= ~SCB_SCR_SLEEPDEEP_Msk;
    
    /* Wait for interrupt */
    __WFI();
    
    /* Execution continues here after wake-up */
}

/* ================================================================
 * Enter Stop Mode
 * Very low power, RAM retained, limited wake sources
 * ================================================================ */
void enter_stop_mode(void) {
    /* Disable SysTick to prevent immediate wake */
    SysTick->CTRL &= ~SysTick_CTRL_ENABLE_Msk;
    
    /* Configure low-power regulator */
    PWR->CR1 |= PWR_CR1_LPMS_STOP1;  /* Stop 1 mode */
    
    /* Set SLEEPDEEP bit */
    SCB->SCR |= SCB_SCR_SLEEPDEEP_Msk;
    
    /* Clear wake-up flag */
    PWR->SCR |= PWR_SCR_CWUF;
    
    /* Wait for interrupt (enters Stop mode) */
    __WFI();
    
    /* === WAKE-UP OCCURS HERE === */
    
    /* Reconfigure clocks (stopped in Stop mode) */
    SystemClock_Config();
    
    /* Re-enable SysTick */
    SysTick->CTRL |= SysTick_CTRL_ENABLE_Msk;
}

/* ================================================================
 * Enter Standby Mode
 * Lowest power, RAM lost, wake causes reset
 * ================================================================ */
void enter_standby_mode(void) {
    /* Save critical data to backup registers */
    RTC->BKP0R = critical_data;
    
    /* Configure WKUP pin */
    PWR->CR3 |= PWR_CR3_EWUP1;  /* Enable WKUP pin 1 */
    
    /* Configure Standby mode */
    PWR->CR1 |= PWR_CR1_LPMS_STANDBY;
    
    /* Set SLEEPDEEP bit */
    SCB->SCR |= SCB_SCR_SLEEPDEEP_Msk;
    
    /* Clear wake-up flag */
    PWR->SCR |= PWR_SCR_CWUF;
    
    /* Enter Standby mode */
    __WFI();
    
    /* This line never reached - wake causes reset */
}

/* ================================================================
 * Configure RTC Wake-up Timer
 * ================================================================ */
void configure_rtc_wakeup(uint32_t seconds) {
    /* Enable RTC clock */
    RCC->APB1ENR1 |= RCC_APB1ENR1_RTCAPBEN;
    RCC->BDCR |= RCC_BDCR_RTCEN;
    
    /* Disable write protection */
    RTC->WPR = 0xCA;
    RTC->WPR = 0x53;
    
    /* Disable wakeup timer to configure */
    RTC->CR &= ~RTC_CR_WUTE;
    while (!(RTC->ISR & RTC_ISR_WUTWF));
    
    /* Configure 1Hz clock source (32768Hz / 16 / 2048 ≈ 1Hz) */
    RTC->CR = (RTC->CR & ~RTC_CR_WUCKSEL) | 0x04;
    
    /* Set wakeup counter */
    RTC->WUTR = seconds - 1;
    
    /* Enable wakeup timer and interrupt */
    RTC->CR |= RTC_CR_WUTE | RTC_CR_WUTIE;
    
    /* Re-enable write protection */
    RTC->WPR = 0xFF;
    
    /* Enable RTC interrupt in NVIC */
    NVIC_EnableIRQ(RTC_WKUP_IRQn);
}

/* ================================================================
 * Typical Low-Power Application Loop
 * ================================================================ */
void low_power_main_loop(void) {
    while (1) {
        /* Do periodic work */
        read_sensors();
        process_data();
        transmit_data();
        
        /* Configure wake-up after 60 seconds */
        configure_rtc_wakeup(60);
        
        /* Enter lowest appropriate power mode */
        enter_stop_mode();
        
        /* Woke up - check wake source */
        if (rtc_wakeup_occurred()) {
            /* Normal timed wake-up */
        } else if (gpio_wakeup_occurred()) {
            /* External event */
            handle_external_event();
        }
    }
}
```

---

## Peripheral Power Optimization

```
┌─────────────────────────────────────────────────────────────────────┐
│              PERIPHERAL POWER OPTIMIZATION                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. CLOCK GATING                                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ /* Disable unused peripheral clocks */                       │  │
│  │ RCC->APB1ENR &= ~RCC_APB1ENR_TIM2EN;   /* Timer 2 off */    │  │
│  │ RCC->APB2ENR &= ~RCC_APB2ENR_SPI1EN;   /* SPI1 off */       │  │
│  │ RCC->AHB1ENR &= ~RCC_AHB1ENR_DMA2EN;   /* DMA2 off */       │  │
│  │                                                               │  │
│  │ /* Only enable when needed */                                │  │
│  │ RCC->APB1ENR |= RCC_APB1ENR_I2C1EN;    /* I2C1 on */        │  │
│  │ read_sensor();                                                │  │
│  │ RCC->APB1ENR &= ~RCC_APB1ENR_I2C1EN;   /* I2C1 off */       │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  2. GPIO CONFIGURATION                                              │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ POWER-CONSCIOUS GPIO SETUP:                                  │  │
│  │                                                               │  │
│  │ ┌─────────────┬────────────────────────────────────────┐    │  │
│  │ │ State       │ Current                                │    │  │
│  │ ├─────────────┼────────────────────────────────────────┤    │  │
│  │ │ Floating    │ WORST! Leakage from mid-rail voltage  │    │  │
│  │ │ Input+PU/PD │ Pull current: VCC/R_pull              │    │  │
│  │ │ Output Low  │ OK if driving nothing                  │    │  │
│  │ │ Output High │ OK if driving nothing                  │    │  │
│  │ │ Analog      │ BEST - digital input buffer disabled  │    │  │
│  │ └─────────────┴────────────────────────────────────────┘    │  │
│  │                                                               │  │
│  │ /* Configure unused pins as analog */                        │  │
│  │ GPIOA->MODER = 0xFFFFFFFF;  /* All analog */                │  │
│  │ GPIOB->MODER = 0xFFFFFFFF;                                   │  │
│  │ /* Then configure only used pins */                          │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  3. EXTERNAL PERIPHERAL CONTROL                                     │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                                                               │  │
│  │  MCU ────[GPIO]───┬───[PMOS]───┬─── VCC                     │  │
│  │           │       │            │                              │  │
│  │          Low=ON   └────────────┼─── Sensor VDD               │  │
│  │                                │                              │  │
│  │                          ┌─────┴─────┐                       │  │
│  │                          │  SENSOR   │                       │  │
│  │                          │  Module   │                       │  │
│  │                          └───────────┘                       │  │
│  │                                                               │  │
│  │  /* Turn sensor on only when needed */                       │  │
│  │  void sensor_power_on(void) {                                │  │
│  │      GPIO_SetLow(SENSOR_POWER_PIN);  /* PMOS on */          │  │
│  │      delay_ms(10);  /* Sensor startup time */                │  │
│  │  }                                                            │  │
│  │                                                               │  │
│  │  void sensor_power_off(void) {                               │  │
│  │      GPIO_SetHigh(SENSOR_POWER_PIN);  /* PMOS off */        │  │
│  │  }                                                            │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  4. VOLTAGE SCALING                                                 │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Lower voltage = Lower power (P = CV²f)                       │  │
│  │                                                               │  │
│  │ ┌────────────┬────────────┬─────────────┐                   │  │
│  │ │ Range      │ VDD        │ Max Freq    │                   │  │
│  │ ├────────────┼────────────┼─────────────┤                   │  │
│  │ │ Range 1    │ 1.2V       │ 80 MHz      │                   │  │
│  │ │ Range 2    │ 1.0V       │ 26 MHz      │                   │  │
│  │ │ Low Power  │ 0.9V       │ 2 MHz       │                   │  │
│  │ └────────────┴────────────┴─────────────┘                   │  │
│  │                                                               │  │
│  │ Trade-off: Lower frequency = slower execution               │  │
│  │ But total energy may be similar or better                   │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Battery Management

### Battery Characteristics

```
┌─────────────────────────────────────────────────────────────────────┐
│              BATTERY TECHNOLOGY COMPARISON                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐│
│  │ Type          │ Voltage │ Wh/kg │ Cycles │ Self-Disch │ Use   ││
│  ├───────────────┼─────────┼───────┼────────┼────────────┼───────┤│
│  │ Alkaline      │ 1.5V    │ 100   │ 1      │ 2%/year    │ Remote││
│  │ Lithium (prim)│ 3.0V    │ 250   │ 1      │ 1%/year    │ Meters││
│  │ Li-Ion        │ 3.7V    │ 150   │ 500+   │ 3%/month   │ Phone ││
│  │ LiPo          │ 3.7V    │ 180   │ 300+   │ 5%/month   │ Drone ││
│  │ LiFePO4       │ 3.2V    │ 90    │ 2000+  │ 3%/month   │ Solar ││
│  │ NiMH          │ 1.2V    │ 80    │ 500    │ 20%/month  │ Toys  ││
│  │ Coin Cell     │ 3.0V    │ 30    │ 1      │ 1%/year    │ RTC   ││
│  └────────────────────────────────────────────────────────────────┘│
│                                                                      │
│  DISCHARGE CURVES:                                                  │
│                                                                      │
│  Voltage                                                             │
│     │      Alkaline (1.5V)                                          │
│  1.5├───────╲                                                       │
│     │        ╲__________                                            │
│  1.2├                   ╲____                                       │
│     │                        ╲___                                   │
│  0.9├──────────────────────────────                                │
│     │                                                               │
│     └────────────────────────────────▶ Capacity %                   │
│     0%           50%           100%                                 │
│                                                                      │
│  Voltage                                                             │
│     │      Li-Ion (3.7V nominal)                                    │
│  4.2├──╲                                                            │
│     │   ╲_____                                                      │
│  3.7├         ╲___________                                          │
│     │                     ╲_____                                    │
│  3.0├──────────────────────────────                                │
│     │                                                               │
│     └────────────────────────────────▶ Capacity %                   │
│     0%           50%           100%                                 │
│                                                                      │
│  Li-Ion: Much flatter discharge curve = stable voltage             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Battery Monitoring

```c
/* ================================================================
 * Battery Voltage and State-of-Charge Monitoring
 * ================================================================ */

#include "adc.h"
#include "battery.h"

/* Voltage divider: VBAT --[R1]--+--[R2]-- GND
 *                               |
 *                             ADC_IN
 * 
 * For 3.7V LiPo, max 4.2V:
 * R1 = 100K, R2 = 100K -> divide by 2 -> max 2.1V to ADC
 */
#define VBAT_DIVIDER_RATIO  2.0f
#define ADC_VREF           3.3f
#define ADC_MAX            4095

/* Li-Ion voltage to SoC lookup table (rough approximation) */
static const struct {
    float voltage;
    uint8_t soc_percent;
} soc_table[] = {
    {4.20f, 100},
    {4.10f, 90},
    {4.00f, 80},
    {3.90f, 70},
    {3.80f, 60},
    {3.70f, 50},
    {3.60f, 40},
    {3.50f, 30},
    {3.40f, 20},
    {3.30f, 10},
    {3.00f, 0}
};

float battery_get_voltage(void) {
    uint16_t adc_value = adc_read(ADC_CHANNEL_VBAT);
    
    /* Convert to voltage at divider midpoint */
    float v_adc = (adc_value * ADC_VREF) / ADC_MAX;
    
    /* Scale up by divider ratio */
    float v_bat = v_adc * VBAT_DIVIDER_RATIO;
    
    return v_bat;
}

uint8_t battery_get_soc(void) {
    float voltage = battery_get_voltage();
    
    /* Find SoC from lookup table */
    for (int i = 0; i < sizeof(soc_table)/sizeof(soc_table[0]) - 1; i++) {
        if (voltage >= soc_table[i+1].voltage) {
            /* Linear interpolation between points */
            float v_high = soc_table[i].voltage;
            float v_low = soc_table[i+1].voltage;
            float soc_high = soc_table[i].soc_percent;
            float soc_low = soc_table[i+1].soc_percent;
            
            float ratio = (voltage - v_low) / (v_high - v_low);
            return (uint8_t)(soc_low + ratio * (soc_high - soc_low));
        }
    }
    
    return 0;  /* Below minimum */
}

battery_status_t battery_get_status(void) {
    float voltage = battery_get_voltage();
    
    if (voltage > 4.15f) {
        return BATTERY_FULL;
    } else if (voltage > 3.5f) {
        return BATTERY_OK;
    } else if (voltage > 3.2f) {
        return BATTERY_LOW;
    } else {
        return BATTERY_CRITICAL;
    }
}

/* Low battery protection */
void battery_check_and_protect(void) {
    if (battery_get_status() == BATTERY_CRITICAL) {
        /* Save state to flash */
        save_critical_state();
        
        /* Enter lowest power mode */
        enter_shutdown_mode();
    }
}
```

---

## Power Supply Design

```
┌─────────────────────────────────────────────────────────────────────┐
│              POWER SUPPLY TOPOLOGIES                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. LINEAR REGULATOR (LDO)                                          │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                                                               │  │
│  │   VIN ──────┬────[LDO]────┬────── VOUT                       │  │
│  │             │             │                                   │  │
│  │            ═╪═           ═╪═                                  │  │
│  │           CIN           COUT                                  │  │
│  │                                                               │  │
│  │   Efficiency = VOUT / VIN                                    │  │
│  │                                                               │  │
│  │   Example: 5V → 3.3V = 66% efficiency                       │  │
│  │            3.7V → 3.3V = 89% efficiency                     │  │
│  │                                                               │  │
│  │   ✓ Simple, low noise    ✗ Heat when VIN >> VOUT            │  │
│  │   ✓ Low cost, small      ✗ VIN must be > VOUT + dropout     │  │
│  │                                                               │  │
│  │   Good LDOs for low power:                                   │  │
│  │   • MCP1700 (Iq = 1.6µA)                                    │  │
│  │   • TPS7A02 (Iq = 25nA!)                                    │  │
│  │   • AP2112 (Iq = 55µA, higher current)                      │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  2. BUCK CONVERTER (Step-Down)                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                                                               │  │
│  │   VIN ────┬────[SW]────┬────[L]────┬────── VOUT             │  │
│  │           │            │           │                          │  │
│  │          ═╪═         [D]         ═╪═                         │  │
│  │         CIN           │          COUT                        │  │
│  │                      ═╧═                                      │  │
│  │                                                               │  │
│  │   Efficiency: 85-95% typical                                 │  │
│  │                                                               │  │
│  │   ✓ High efficiency     ✗ More components                    │  │
│  │   ✓ Handles VIN >> VOUT ✗ Switching noise                   │  │
│  │                                                               │  │
│  │   Popular ICs: TPS62840 (60nA Iq), MP2155, LTC3440          │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  3. BOOST CONVERTER (Step-Up)                                       │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                                                               │  │
│  │   VIN ────[L]────┬────[D]────┬────── VOUT                   │  │
│  │                  │           │                                │  │
│  │                [SW]        ═╪═                               │  │
│  │                  │         COUT                              │  │
│  │                 ═╧═                                           │  │
│  │                                                               │  │
│  │   Use case: 1.5V battery → 3.3V MCU                         │  │
│  │                                                               │  │
│  │   Popular ICs: TPS61021A, MCP1640, MAX1722                  │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  CHOOSING A REGULATOR:                                              │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Condition                    │ Recommendation                │  │
│  ├──────────────────────────────┼───────────────────────────────┤  │
│  │ VIN close to VOUT (< 1V)     │ LDO (low dropout)            │  │
│  │ VIN >> VOUT, high current    │ Buck converter               │  │
│  │ VIN < VOUT                   │ Boost converter              │  │
│  │ Ultra-low power, simple      │ LDO with low Iq              │  │
│  │ Noise-sensitive (analog)     │ LDO for clean power          │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Energy Harvesting

```
┌─────────────────────────────────────────────────────────────────────┐
│              ENERGY HARVESTING OPTIONS                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐│
│  │ Source          │ Power Available    │ Use Case               ││
│  ├─────────────────┼────────────────────┼────────────────────────┤│
│  │ Indoor solar    │ 10-100 µW/cm²      │ Sensors, displays      ││
│  │ Outdoor solar   │ 10-100 mW/cm²      │ Weather stations       ││
│  │ Thermoelectric  │ 10-100 µW/cm² (ΔT) │ Body heat, machinery   ││
│  │ Vibration       │ 1-100 µW           │ Industrial, vehicles   ││
│  │ RF harvesting   │ 0.1-10 µW          │ RFID, near transmitter ││
│  └────────────────────────────────────────────────────────────────┘│
│                                                                      │
│  SOLAR HARVESTING CIRCUIT:                                          │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                                                               │  │
│  │  ┌─────────┐    ┌─────────────┐    ┌─────────┐             │  │
│  │  │  SOLAR  │───▶│   PMIC      │───▶│ Storage │───▶ Load    │  │
│  │  │  PANEL  │    │ (BQ25570)   │    │ (Super- │              │  │
│  │  │         │    │             │    │   cap)  │              │  │
│  │  └─────────┘    │ • MPPT      │    └─────────┘              │  │
│  │                 │ • Buck/Boost │                              │  │
│  │                 │ • Charging   │                              │  │
│  │                 └─────────────┘                               │  │
│  │                                                               │  │
│  │  Popular PMICs:                                              │  │
│  │  • BQ25570: Solar/thermal, 100nA Iq, coldstart from 330mV   │  │
│  │  • AEM10941: Solar, indoor optimized                        │  │
│  │  • MAX20361: Multi-source harvesting                        │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ENERGY BUDGET EXAMPLE (Solar-powered sensor):                      │
│                                                                      │
│  HARVESTING:                                                        │
│  • 4 cm² solar cell indoor: 4 × 10 µW = 40 µW average             │
│  • Daily harvest: 40 µW × 8 hours light = 1.15 mWh                 │
│                                                                      │
│  CONSUMPTION:                                                        │
│  • Sleep 99.9%: 2 µA × 3.3V = 6.6 µW                               │
│  • Active 0.1%: 10 mA × 3.3V × 0.001 = 33 µW average              │
│  • Total: ~40 µW                                                    │
│                                                                      │
│  BALANCE: 40 µW harvest ≈ 40 µW consume → SUSTAINABLE!            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Power Design Checklist

```
┌─────────────────────────────────────────────────────────────────────┐
│              LOW-POWER DESIGN CHECKLIST                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  HARDWARE DESIGN:                                                    │
│  □ Choose MCU with good low-power modes                             │
│  □ Select LDO with low quiescent current (< 1µA for battery)       │
│  □ Use power switches for external peripherals                      │
│  □ Avoid pull-up/pull-down on unused pins                          │
│  □ Consider leakage through ESD protection                          │
│  □ Place decoupling caps close to ICs                               │
│                                                                      │
│  GPIO CONFIGURATION:                                                 │
│  □ Configure unused pins as analog (no floating inputs!)           │
│  □ Disable internal pull-ups when external present                 │
│  □ Drive outputs to rails (avoid mid-level)                        │
│  □ Turn off LEDs when not needed                                    │
│                                                                      │
│  CLOCK MANAGEMENT:                                                   │
│  □ Use lowest frequency that meets timing                          │
│  □ Switch to internal RC when precision not needed                 │
│  □ Disable peripheral clocks when idle                              │
│  □ Stop main oscillator in deep sleep                               │
│                                                                      │
│  SOFTWARE DESIGN:                                                    │
│  □ Implement sleep modes between activities                         │
│  □ Use interrupts instead of polling                                │
│  □ Batch operations to maximize sleep time                          │
│  □ Optimize wake-up time (fast clock restore)                      │
│  □ Monitor battery voltage and protect from over-discharge         │
│                                                                      │
│  MEASUREMENT:                                                        │
│  □ Measure current in each operating mode                           │
│  □ Profile current during transitions                               │
│  □ Verify sleep current meets target                                │
│  □ Test across temperature range                                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **Power Budget** | Calculate average current from mode durations and currents |
| **MCU Power Modes** | Sleep < Stop < Standby < Shutdown (decreasing power) |
| **Clock Gating** | Disable peripheral clocks when not in use |
| **GPIO Config** | Unused pins should be analog or driven (not floating) |
| **LDO vs Buck** | LDO simple but wastes power when VIN >> VOUT |
| **Battery SoC** | Estimate remaining capacity from voltage curve |
| **Energy Harvesting** | Solar, thermal, vibration can power low-duty sensors |
| **Quiescent Current** | Regulator Iq critical for always-on systems |

---

## Quick Revision Questions

1. **Calculate the average current and battery life for a device that is active (10mA) for 100ms and sleeps (5µA) for 10s in each cycle, powered by a 1000mAh battery.**

2. **Explain the difference between Sleep, Stop, and Standby modes in ARM Cortex-M. What are the wake-up sources for each?**

3. **Why should unused GPIO pins be configured as analog mode in low-power applications?**

4. **Compare LDO and buck converter regulators. When would you choose each for a battery-powered device?**

5. **What is quiescent current (Iq) and why is it important for always-on embedded systems?**

6. **Describe a solar energy harvesting system for an IoT sensor. What components are needed and how do you ensure energy balance?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Chapter 7.4: Testing and Validation](04-testing-validation.md) | [Unit 7: System Design](README.md) | [Unit 8: Case Studies](../08-Case-Studies/README.md) |

---
