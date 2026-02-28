# Chapter 8.2: Medical Device Embedded Systems

## Introduction

Medical embedded systems directly impact patient health and safety, requiring exceptional reliability, accuracy, and regulatory compliance. From bedside monitors to implantable pacemakers, these devices operate under stringent quality standards (IEC 62304, FDA 21 CFR Part 820) and often must function continuously for years. This chapter explores the unique challenges and design patterns for medical device development.

---

## Medical Device Classifications

```
┌─────────────────────────────────────────────────────────────────────┐
│              MEDICAL DEVICE RISK CLASSIFICATION                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  FDA CLASSIFICATION:                                                 │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │ Class │ Risk    │ Regulatory Path │ Examples                  │ │
│  ├───────┼─────────┼─────────────────┼───────────────────────────┤ │
│  │   I   │ Low     │ General Controls│ Bandages, tongue          │ │
│  │       │         │                 │ depressors, stethoscope   │ │
│  ├───────┼─────────┼─────────────────┼───────────────────────────┤ │
│  │   II  │ Medium  │ 510(k) Clearance│ Blood pressure monitor,   │ │
│  │       │         │                 │ infusion pump, CT scanner │ │
│  ├───────┼─────────┼─────────────────┼───────────────────────────┤ │
│  │  III  │ High    │ PMA Approval    │ Pacemaker, implantable    │ │
│  │       │         │                 │ defibrillator, stent      │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  IEC 62304 SOFTWARE SAFETY CLASSIFICATION:                          │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │ Class │ Definition                      │ Requirements         │ │
│  ├───────┼─────────────────────────────────┼──────────────────────┤ │
│  │   A   │ No injury possible              │ Basic processes      │ │
│  ├───────┼─────────────────────────────────┼──────────────────────┤ │
│  │   B   │ Non-serious injury possible     │ Enhanced processes,  │ │
│  │       │                                 │ verification         │ │
│  ├───────┼─────────────────────────────────┼──────────────────────┤ │
│  │   C   │ Death or serious injury possible│ Full lifecycle,      │ │
│  │       │                                 │ complete traceability│ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  DEVICE CATEGORIES:                                                  │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                    DIAGNOSTIC                                 │  │
│  │   • Patient monitors (ECG, SpO2, BP)                         │  │
│  │   • Imaging systems (X-ray, MRI, ultrasound)                 │  │
│  │   • Laboratory analyzers (blood chemistry)                   │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                    THERAPEUTIC                                │  │
│  │   • Infusion pumps                                           │  │
│  │   • Ventilators                                              │  │
│  │   • Dialysis machines                                        │  │
│  │   • Surgical robots                                          │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                   IMPLANTABLE                                 │  │
│  │   • Pacemakers                                               │  │
│  │   • Implantable cardioverter-defibrillators (ICD)            │  │
│  │   • Cochlear implants                                        │  │
│  │   • Drug delivery systems                                    │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Patient Monitoring System

```
┌─────────────────────────────────────────────────────────────────────┐
│              MULTI-PARAMETER PATIENT MONITOR                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    SENSOR MODULES                              │ │
│  │                                                                │ │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐          │ │
│  │  │   ECG   │  │  SpO2   │  │   BP    │  │  Temp   │          │ │
│  │  │ 3/5/12  │  │ Pulse   │  │ NIBP/   │  │ Core/   │          │ │
│  │  │  Lead   │  │ Oximeter│  │   IBP   │  │ Surface │          │ │
│  │  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘          │ │
│  │       │            │            │            │                │ │
│  │       │     ┌─────────────────────────────┐ │                │ │
│  │       │     │    Analog Front End (AFE)   │ │                │ │
│  │       └────▶│  • High-impedance inputs    │◀┘                │ │
│  │             │  • Defibrillator protection │                  │ │
│  │             │  • Isolation (medical-grade)│                  │ │
│  │             │  • Low-noise amplifiers     │                  │ │
│  │             └─────────────┬───────────────┘                  │ │
│  └───────────────────────────┼───────────────────────────────────┘ │
│                              │                                      │
│  ┌───────────────────────────▼───────────────────────────────────┐ │
│  │                 SIGNAL PROCESSING UNIT                         │ │
│  │                                                                │ │
│  │  ┌──────────────────────────────────────────────────────────┐ │ │
│  │  │ ADC (24-bit, low-noise)                                  │ │ │
│  │  │   │                                                      │ │ │
│  │  │   ▼                                                      │ │ │
│  │  │ Digital Filtering                                        │ │ │
│  │  │   • 50/60Hz notch (power line interference)             │ │ │
│  │  │   • Baseline wander removal                              │ │ │
│  │  │   • High-frequency noise filtering                       │ │ │
│  │  │   │                                                      │ │ │
│  │  │   ▼                                                      │ │ │
│  │  │ Feature Extraction                                       │ │ │
│  │  │   • R-peak detection (ECG)                              │ │ │
│  │  │   • Heart rate calculation                               │ │ │
│  │  │   • SpO2 ratio calculation                              │ │ │
│  │  │   │                                                      │ │ │
│  │  │   ▼                                                      │ │ │
│  │  │ Arrhythmia Detection                                     │ │ │
│  │  │   • Pattern matching                                     │ │ │
│  │  │   • Alarm generation                                     │ │ │
│  │  └──────────────────────────────────────────────────────────┘ │ │
│  └────────────────────────────┬──────────────────────────────────┘ │
│                               │                                     │
│  ┌────────────────────────────▼──────────────────────────────────┐ │
│  │                     DISPLAY & INTERFACE                        │ │
│  │  ┌─────────────────────────────────────────────────────────┐  │ │
│  │  │ ♥ HR: 72 bpm        SpO2: 98%        BP: 120/80 mmHg    │  │ │
│  │  │ ───────────────────────────────────────────────────────  │  │ │
│  │  │ ECG Lead II:                                            │  │ │
│  │  │    ╱╲      ╱╲      ╱╲      ╱╲      ╱╲      ╱╲          │  │ │
│  │  │ ──╱  ╲────╱  ╲────╱  ╲────╱  ╲────╱  ╲────╱  ╲──       │  │ │
│  │  │                                                          │  │ │
│  │  │ SpO2 Pleth:                                             │  │ │
│  │  │  ╱╲    ╱╲    ╱╲    ╱╲    ╱╲    ╱╲    ╱╲    ╱╲         │  │ │
│  │  │ ╱  ╲  ╱  ╲  ╱  ╲  ╱  ╲  ╱  ╲  ╱  ╲  ╱  ╲  ╱  ╲        │  │ │
│  │  │                                                          │  │ │
│  │  │ ALARMS: [SILENCE] [LIMITS] [TRENDS] [RECORD]            │  │ │
│  │  └─────────────────────────────────────────────────────────┘  │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### ECG Signal Processing

```c
/* ================================================================
 * ECG Signal Processing - Medical Grade
 * IEC 62304 Class C Software
 * ================================================================ */

#include <stdint.h>
#include <stdbool.h>

/* ECG signal constants */
#define ECG_SAMPLE_RATE     500    /* 500 Hz sampling */
#define ECG_BUFFER_SIZE     2048   /* 4 second buffer */
#define QRS_DETECTION_THRESHOLD  0.6

/* ================================================================
 * Digital Filters
 * ================================================================ */

/* Second-order IIR filter structure */
typedef struct {
    float b0, b1, b2;  /* Numerator coefficients */
    float a1, a2;      /* Denominator coefficients */
    float x1, x2;      /* Input delay line */
    float y1, y2;      /* Output delay line */
} biquad_filter_t;

/* 50Hz notch filter coefficients (Q=30, fs=500Hz) */
static const biquad_filter_t notch_50hz_init = {
    .b0 = 0.9691f,
    .b1 = -1.6856f,
    .b2 = 0.9691f,
    .a1 = -1.6856f,
    .a2 = 0.9382f,
    .x1 = 0, .x2 = 0,
    .y1 = 0, .y2 = 0
};

/* Bandpass filter (0.5-40Hz for ECG) */
static const biquad_filter_t highpass_05hz_init = {
    .b0 = 0.9961f,
    .b1 = -1.9922f,
    .b2 = 0.9961f,
    .a1 = -1.9921f,
    .a2 = 0.9922f,
    .x1 = 0, .x2 = 0,
    .y1 = 0, .y2 = 0
};

/* Apply biquad filter to sample */
float biquad_process(biquad_filter_t *f, float input) {
    float output = f->b0 * input + f->b1 * f->x1 + f->b2 * f->x2
                   - f->a1 * f->y1 - f->a2 * f->y2;
    
    /* Update delay lines */
    f->x2 = f->x1;
    f->x1 = input;
    f->y2 = f->y1;
    f->y1 = output;
    
    return output;
}

/* ================================================================
 * R-Peak Detection (Pan-Tompkins Algorithm)
 * ================================================================ */

typedef struct {
    /* Filters */
    biquad_filter_t notch_filter;
    biquad_filter_t highpass_filter;
    
    /* Derivative buffer */
    float derivative_buffer[5];
    uint8_t derivative_index;
    
    /* Moving average */
    float integration_buffer[30];  /* 60ms window at 500Hz */
    uint8_t integration_index;
    
    /* Thresholds (adaptive) */
    float threshold_i1;
    float threshold_i2;
    float spk_i;
    float npk_i;
    
    /* Timing */
    uint32_t rr_intervals[8];
    uint8_t rr_index;
    uint32_t last_r_time;
    uint32_t sample_count;
    
    /* Outputs */
    uint16_t heart_rate;
    bool r_peak_detected;
} ecg_processor_t;

/* Initialize ECG processor */
void ecg_processor_init(ecg_processor_t *proc) {
    proc->notch_filter = notch_50hz_init;
    proc->highpass_filter = highpass_05hz_init;
    proc->derivative_index = 0;
    proc->integration_index = 0;
    proc->threshold_i1 = 0.25f;
    proc->threshold_i2 = 0.125f;
    proc->spk_i = 0;
    proc->npk_i = 0;
    proc->rr_index = 0;
    proc->last_r_time = 0;
    proc->sample_count = 0;
    proc->heart_rate = 0;
    proc->r_peak_detected = false;
}

/* Process single ECG sample */
void ecg_process_sample(ecg_processor_t *proc, float raw_sample) {
    float sample;
    
    /* Step 1: Apply notch filter (50/60Hz removal) */
    sample = biquad_process(&proc->notch_filter, raw_sample);
    
    /* Step 2: Apply highpass filter (baseline wander removal) */
    sample = biquad_process(&proc->highpass_filter, sample);
    
    /* Step 3: Derivative (5-point) */
    proc->derivative_buffer[proc->derivative_index] = sample;
    proc->derivative_index = (proc->derivative_index + 1) % 5;
    
    float derivative = 0;
    for (int i = 0; i < 5; i++) {
        int weight = (i < 2) ? (i - 2) : (i - 2);
        derivative += weight * proc->derivative_buffer[i];
    }
    derivative = derivative / 8.0f;
    
    /* Step 4: Squaring */
    float squared = derivative * derivative;
    
    /* Step 5: Moving window integration */
    proc->integration_buffer[proc->integration_index] = squared;
    proc->integration_index = (proc->integration_index + 1) % 30;
    
    float integrated = 0;
    for (int i = 0; i < 30; i++) {
        integrated += proc->integration_buffer[i];
    }
    integrated = integrated / 30.0f;
    
    /* Step 6: Adaptive thresholding */
    proc->sample_count++;
    proc->r_peak_detected = false;
    
    /* 200ms refractory period (100 samples at 500Hz) */
    if (proc->sample_count - proc->last_r_time > 100) {
        if (integrated > proc->threshold_i1) {
            /* R-peak detected */
            proc->r_peak_detected = true;
            
            /* Update signal peak estimate */
            proc->spk_i = 0.125f * integrated + 0.875f * proc->spk_i;
            
            /* Calculate RR interval and heart rate */
            uint32_t rr_interval = proc->sample_count - proc->last_r_time;
            proc->rr_intervals[proc->rr_index] = rr_interval;
            proc->rr_index = (proc->rr_index + 1) % 8;
            
            /* Average RR interval */
            uint32_t avg_rr = 0;
            for (int i = 0; i < 8; i++) {
                avg_rr += proc->rr_intervals[i];
            }
            avg_rr /= 8;
            
            /* Heart rate = 60 * sample_rate / RR_interval */
            if (avg_rr > 0) {
                proc->heart_rate = (60 * ECG_SAMPLE_RATE) / avg_rr;
            }
            
            proc->last_r_time = proc->sample_count;
        } else {
            /* Update noise peak estimate */
            proc->npk_i = 0.125f * integrated + 0.875f * proc->npk_i;
        }
        
        /* Update thresholds */
        proc->threshold_i1 = proc->npk_i + 
            0.25f * (proc->spk_i - proc->npk_i);
        proc->threshold_i2 = 0.5f * proc->threshold_i1;
    }
}
```

---

## Implantable Pacemaker Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│              IMPLANTABLE PACEMAKER ARCHITECTURE                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    HERMETICALLY SEALED CAN                     │ │
│  │                    (Titanium, 20-30cc volume)                  │ │
│  │                                                                │ │
│  │  ┌──────────────────────────────────────────────────────────┐ │ │
│  │  │               BATTERY (Lithium-Iodine)                   │ │ │
│  │  │                 • 1.5-2.8V, 1-2 Ah                       │ │ │
│  │  │                 • 8-15 year lifetime                     │ │ │
│  │  │                 • Predictable depletion                  │ │ │
│  │  └──────────────────────────────────────────────────────────┘ │ │
│  │                              │                                 │ │
│  │  ┌──────────────────────────▼───────────────────────────────┐ │ │
│  │  │                   POWER MANAGEMENT                        │ │ │
│  │  │  • DC-DC converter for multiple voltage rails            │ │ │
│  │  │  • Ultra-low quiescent current (<1µA)                    │ │ │
│  │  │  • Battery voltage monitoring (EOL detection)            │ │ │
│  │  └──────────────────────────┬───────────────────────────────┘ │ │
│  │                              │                                 │ │
│  │  ┌──────────────────────────▼───────────────────────────────┐ │ │
│  │  │              ASIC / MICROCONTROLLER                       │ │ │
│  │  │                                                           │ │ │
│  │  │  ┌─────────────────────────────────────────────────────┐ │ │ │
│  │  │  │  • Ultra-low power MCU (ARM Cortex-M0+)            │ │ │ │
│  │  │  │  • Operating current: 10-50 µA active              │ │ │ │
│  │  │  │  • Sleep current: 100 nA                           │ │ │ │
│  │  │  │  • Custom ASIC for sensing/pacing                  │ │ │ │
│  │  │  └─────────────────────────────────────────────────────┘ │ │ │
│  │  │                                                           │ │ │
│  │  │  ┌─────────────────────────────────────────────────────┐ │ │ │
│  │  │  │  SENSE AMPLIFIER                                    │ │ │ │
│  │  │  │  • Input: 1-10 mV cardiac signals                  │ │ │ │
│  │  │  │  • Programmable sensitivity                         │ │ │ │
│  │  │  │  • Noise rejection filters                          │ │ │ │
│  │  │  └─────────────────────────────────────────────────────┘ │ │ │
│  │  │                                                           │ │ │
│  │  │  ┌─────────────────────────────────────────────────────┐ │ │ │
│  │  │  │  OUTPUT STAGE                                       │ │ │ │
│  │  │  │  • Pacing pulse: 2.5-5V, 0.1-1.5ms                │ │ │ │
│  │  │  │  • Charge-balanced (no DC component)               │ │ │ │
│  │  │  │  • Programmable amplitude/duration                  │ │ │ │
│  │  │  └─────────────────────────────────────────────────────┘ │ │ │
│  │  │                                                           │ │ │
│  │  │  ┌─────────────────────────────────────────────────────┐ │ │ │
│  │  │  │  TELEMETRY (RF)                                     │ │ │ │
│  │  │  │  • MICS band: 402-405 MHz                          │ │ │ │
│  │  │  │  • Inductive coupling for programming              │ │ │ │
│  │  │  │  • Encrypted communication                          │ │ │ │
│  │  │  └─────────────────────────────────────────────────────┘ │ │ │
│  │  └──────────────────────────────────────────────────────────┘ │ │
│  │                                                                │ │
│  └────────────────────────────┬───────────────────────────────────┘ │
│                               │ Feedthrough connector               │
│  ┌────────────────────────────▼───────────────────────────────────┐ │
│  │                         LEADS                                   │ │
│  │   ┌────────────────────────────────────────────────────────┐   │ │
│  │   │  Atrial Lead ─────────────────────────▶ Right Atrium   │   │ │
│  │   │  (Bipolar, silicone insulated)                         │   │ │
│  │   └────────────────────────────────────────────────────────┘   │ │
│  │   ┌────────────────────────────────────────────────────────┐   │ │
│  │   │  Ventricular Lead ────────────────────▶ Right Ventricle│   │ │
│  │   │  (Active fixation, steroid-eluting)                    │   │ │
│  │   └────────────────────────────────────────────────────────┘   │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Pacing Algorithm

```
┌─────────────────────────────────────────────────────────────────────┐
│              DUAL-CHAMBER PACING (DDD MODE)                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  TIMING DIAGRAM:                                                     │
│                                                                      │
│       AV Interval           VA Interval                              │
│      ├──────────┤    ├─────────────────────────┤                    │
│                                                                      │
│  A:  │   As     │              │   Ap      │                        │
│      └───┬──────┴──────────────┴───┬───────┘                        │
│          │                         │                                 │
│  V:      │         │   Vs          │         │  Vp                  │
│      ────┴─────────┴───┬───────────┴─────────┴──┬────               │
│                        │                        │                    │
│                                                                      │
│  As = Atrial sense     Ap = Atrial pace                             │
│  Vs = Ventricular sense Vp = Ventricular pace                       │
│                                                                      │
│  STATE MACHINE:                                                      │
│                                                                      │
│         ┌─────────────────────────────────────────────┐             │
│         │                                             │             │
│         ▼                                             │             │
│  ┌────────────┐    A-sense    ┌────────────┐         │             │
│  │   WAIT     │──────────────▶│    AV      │         │             │
│  │  A-EVENT   │               │   DELAY    │         │             │
│  └────────────┘               └─────┬──────┘         │             │
│         │                           │                │             │
│         │ Timeout                   │ Timeout or     │             │
│         │ (VA interval)             │ V-sense        │             │
│         ▼                           ▼                │             │
│  ┌────────────┐               ┌────────────┐         │             │
│  │   PACE     │               │   WAIT     │         │             │
│  │  ATRIUM    │               │  V-EVENT   │─────────┘             │
│  └─────┬──────┘               └─────┬──────┘                        │
│        │                            │                               │
│        │                            │ V-sense or                    │
│        │                            │ Timeout→Pace V                │
│        │                            ▼                               │
│        │                      ┌────────────┐                        │
│        └─────────────────────▶│   START    │                        │
│                               │ VA INTERVAL│                        │
│                               └────────────┘                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Infusion Pump Safety Design

```
┌─────────────────────────────────────────────────────────────────────┐
│              INFUSION PUMP SAFETY ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  HARDWARE ARCHITECTURE:                                              │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                                                                │ │
│  │  ┌─────────────┐                      ┌─────────────┐         │ │
│  │  │   MAIN      │    Heartbeat         │  SAFETY     │         │ │
│  │  │ CONTROLLER  │◀════════════════════▶│ CONTROLLER  │         │ │
│  │  │             │    (Watchdog)        │             │         │ │
│  │  │ • Dose      │                      │ • Flow rate │         │ │
│  │  │   calculation│                     │   monitoring│         │ │
│  │  │ • User      │                      │ • Dose limit│         │ │
│  │  │   interface │                      │   checking  │         │ │
│  │  │ • Logging   │                      │ • Occlusion │         │ │
│  │  │             │                      │   detection │         │ │
│  │  └──────┬──────┘                      └──────┬──────┘         │ │
│  │         │                                    │                 │ │
│  │         │        ┌────────────────┐          │                 │ │
│  │         └───────▶│  MOTOR DRIVER  │◀─────────┘                 │ │
│  │                  │                │                            │ │
│  │                  │ Both must agree│                            │ │
│  │                  │ to enable motor│                            │ │
│  │                  └───────┬────────┘                            │ │
│  │                          │                                     │ │
│  │                          ▼                                     │ │
│  │  ┌────────────────────────────────────────────────────────┐   │ │
│  │  │                    MOTOR                                │   │ │
│  │  │                 (Stepper/DC)                            │   │ │
│  │  │                      │                                  │   │ │
│  │  │                      ▼                                  │   │ │
│  │  │               ┌──────────┐                              │   │ │
│  │  │               │  Pump    │                              │   │ │
│  │  │               │Mechanism │                              │   │ │
│  │  │               │(Syringe/ │                              │   │ │
│  │  │               │Peristaltic)                             │   │ │
│  │  │               └────┬─────┘                              │   │ │
│  │  │                    │                                    │   │ │
│  │  │             ┌──────▼──────┐                             │   │ │
│  │  │             │Flow Sensor  │                             │   │ │
│  │  │             │(Ultrasonic/ │                             │   │ │
│  │  │             │ Drop count) │                             │   │ │
│  │  │             └─────────────┘                             │   │ │
│  │  └────────────────────────────────────────────────────────┘   │ │
│  │                                                                │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  SAFETY CHECKS:                                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │ Check           │ Method                    │ Action          │ │
│  ├──────────────────┼───────────────────────────┼─────────────────┤ │
│  │ Free-flow        │ Clamp sensor, gravity     │ Stop pump       │ │
│  │ Occlusion        │ Pressure sensor           │ Alarm, stop     │ │
│  │ Air-in-line      │ Ultrasonic detector       │ Stop, alarm     │ │
│  │ Over-dose        │ Dose limit table          │ Block infusion  │ │
│  │ Under-dose       │ Flow rate feedback        │ Alarm           │ │
│  │ Empty reservoir  │ Weight/level sensor       │ Alarm           │ │
│  │ Door open        │ Interlock switch          │ Pause, alarm    │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Safety-Critical Code Pattern

```c
/* ================================================================
 * Infusion Pump - Dose Validation
 * IEC 62304 Class C - Safety Critical
 * ================================================================ */

#include <stdint.h>
#include <stdbool.h>
#include <string.h>

/* Drug library entry */
typedef struct {
    uint32_t drug_id;
    char     drug_name[32];
    float    max_bolus_ml;
    float    max_rate_ml_hr;
    float    min_rate_ml_hr;
    float    max_dose_mg_kg_hr;     /* Weight-based limit */
    float    concentration_mg_ml;
    uint32_t crc32;                  /* Integrity check */
} drug_entry_t;

/* Patient data */
typedef struct {
    float weight_kg;
    uint8_t age_years;
} patient_data_t;

/* Dose calculation result */
typedef enum {
    DOSE_OK = 0,
    DOSE_EXCEEDS_RATE_LIMIT,
    DOSE_EXCEEDS_BOLUS_LIMIT,
    DOSE_BELOW_MINIMUM,
    DOSE_EXCEEDS_WEIGHT_LIMIT,
    DOSE_DRUG_NOT_FOUND,
    DOSE_CRC_ERROR,
    DOSE_VALIDATION_FAILED
} dose_result_t;

/* CRC32 verification */
static bool verify_drug_entry(const drug_entry_t *drug) {
    uint32_t calculated_crc = calculate_crc32(
        (uint8_t*)drug, 
        sizeof(drug_entry_t) - sizeof(uint32_t)
    );
    return (calculated_crc == drug->crc32);
}

/* ================================================================
 * Dose Validation - Triple Redundancy
 * ================================================================ */
dose_result_t validate_dose(
    const drug_entry_t *drug,
    const patient_data_t *patient,
    float requested_rate_ml_hr,
    float bolus_ml
) {
    dose_result_t result1, result2, result3;
    
    /* VALIDATION PASS 1 - Main calculation */
    result1 = validate_dose_impl(drug, patient, 
                                  requested_rate_ml_hr, bolus_ml);
    
    /* VALIDATION PASS 2 - Redundant calculation with inverted logic */
    result2 = validate_dose_impl_alt(drug, patient,
                                      requested_rate_ml_hr, bolus_ml);
    
    /* VALIDATION PASS 3 - Independent implementation */
    result3 = validate_dose_impl_v2(drug, patient,
                                     requested_rate_ml_hr, bolus_ml);
    
    /* Voting - all three must agree */
    if (result1 == result2 && result2 == result3) {
        return result1;
    }
    
    /* Disagreement - fail safe */
    log_safety_event("Dose validation mismatch: %d, %d, %d",
                     result1, result2, result3);
    return DOSE_VALIDATION_FAILED;
}

static dose_result_t validate_dose_impl(
    const drug_entry_t *drug,
    const patient_data_t *patient,
    float requested_rate_ml_hr,
    float bolus_ml
) {
    /* Verify drug library integrity */
    if (!verify_drug_entry(drug)) {
        return DOSE_CRC_ERROR;
    }
    
    /* Check rate limits */
    if (requested_rate_ml_hr > drug->max_rate_ml_hr) {
        return DOSE_EXCEEDS_RATE_LIMIT;
    }
    
    if (requested_rate_ml_hr < drug->min_rate_ml_hr &&
        requested_rate_ml_hr != 0) {
        return DOSE_BELOW_MINIMUM;
    }
    
    /* Check bolus limit */
    if (bolus_ml > drug->max_bolus_ml) {
        return DOSE_EXCEEDS_BOLUS_LIMIT;
    }
    
    /* Weight-based dose limit */
    float dose_mg_hr = requested_rate_ml_hr * drug->concentration_mg_ml;
    float dose_mg_kg_hr = dose_mg_hr / patient->weight_kg;
    
    if (dose_mg_kg_hr > drug->max_dose_mg_kg_hr) {
        return DOSE_EXCEEDS_WEIGHT_LIMIT;
    }
    
    return DOSE_OK;
}

/* ================================================================
 * Safe State Handler
 * ================================================================ */
void enter_safe_state(const char *reason) {
    /* 1. Stop motor immediately */
    motor_emergency_stop();
    
    /* 2. Close all valves */
    valve_close_all();
    
    /* 3. Sound alarm */
    alarm_activate(ALARM_PRIORITY_HIGH);
    
    /* 4. Log event with timestamp */
    log_safety_event("SAFE STATE: %s", reason);
    
    /* 5. Display error */
    display_error_screen(reason);
    
    /* 6. Notify safety controller */
    safety_controller_notify(SAFE_STATE_ENTERED);
    
    /* 7. Remain in safe state until reset */
    while (1) {
        watchdog_feed();  /* Keep alive for logging */
        if (user_acknowledged()) {
            break;
        }
    }
}
```

---

## IEC 62304 Development Lifecycle

```
┌─────────────────────────────────────────────────────────────────────┐
│              IEC 62304 SOFTWARE LIFECYCLE                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │                    DEVELOPMENT PROCESS                          │ │
│  │                                                                 │ │
│  │   ┌─────────────────┐                                          │ │
│  │   │ 5.1 Software    │ ─────────────────────────────────────┐   │ │
│  │   │ Development     │                                      │   │ │
│  │   │ Planning        │                                      │   │ │
│  │   └────────┬────────┘                                      │   │ │
│  │            │                                               │   │ │
│  │            ▼                                               │   │ │
│  │   ┌─────────────────┐                                      │   │ │
│  │   │ 5.2 Software    │ ◀──────── System Requirements       │   │ │
│  │   │ Requirements    │          (Input from ISO 14971)     │   │ │
│  │   │ Analysis        │                                      │   │ │
│  │   └────────┬────────┘                                      │   │ │
│  │            │                                               │   │ │
│  │            ▼                                               │   │ │
│  │   ┌─────────────────┐    ┌─────────────────┐              │   │ │
│  │   │ 5.3 Software    │───▶│ 5.4 Software    │              │   │ │
│  │   │ Architectural   │    │ Detailed Design │              │   │ │
│  │   │ Design          │    │                 │              │   │ │
│  │   └─────────────────┘    └────────┬────────┘              │   │ │
│  │                                   │                        │   │ │
│  │                                   ▼                        │   │ │
│  │                          ┌─────────────────┐              │   │ │
│  │                          │ 5.5 Software    │              │   │ │
│  │                          │ Unit           │              │   │ │
│  │                          │ Implementation  │              │   │ │
│  │                          └────────┬────────┘              │   │ │
│  │                                   │                        │   │ │
│  │            ┌──────────────────────┼───────────────────┐   │   │ │
│  │            ▼                      ▼                   ▼   │   │ │
│  │   ┌─────────────────┐    ┌─────────────────┐   ┌─────────┴─┐│   │
│  │   │ 5.6 Software    │    │ 5.7 Software    │   │    5.8    ││   │
│  │   │ Unit            │───▶│ Integration     │──▶│ Software  ││   │
│  │   │ Verification    │    │ & Testing       │   │ System    ││   │
│  │   └─────────────────┘    └─────────────────┘   │ Testing   ││   │
│  │                                                 └─────┬─────┘│   │
│  │                                                       │      │   │
│  │   ┌───────────────────────────────────────────────────┘      │   │
│  │   │                                                          │   │
│  │   ▼                                                          │   │
│  │   ┌─────────────────┐                                        │   │
│  │   │ 5.9 Software    │ ────────────────────────────────────┘  │ │
│  │   │ Release         │                                        │ │
│  │   │                 │                                        │ │
│  │   └─────────────────┘                                        │ │
│  │                                                               │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  DOCUMENTATION REQUIREMENTS BY CLASS:                               │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │ Activity                    │  A  │  B  │  C  │               │ │
│  ├─────────────────────────────┼─────┼─────┼─────┤               │ │
│  │ Development planning        │  ●  │  ●  │  ●  │               │ │
│  │ Requirements analysis       │  ●  │  ●  │  ●  │               │ │
│  │ Architectural design        │  ○  │  ●  │  ●  │               │ │
│  │ Detailed design             │  ○  │  ○  │  ●  │               │ │
│  │ Unit implementation         │  ●  │  ●  │  ●  │               │ │
│  │ Unit verification           │  ○  │  ●  │  ●  │               │ │
│  │ Integration testing         │  ○  │  ●  │  ●  │               │ │
│  │ System testing              │  ●  │  ●  │  ●  │               │ │
│  │ Risk management             │  ●  │  ●  │  ●  │               │ │
│  │ Traceability                │  ○  │  ●  │  ●  │               │ │
│  ├─────────────────────────────┴─────┴─────┴─────┤               │ │
│  │ ● = Required    ○ = Recommended/Optional      │               │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **Device Class** | FDA Class I/II/III based on risk; IEC 62304 Class A/B/C for software |
| **Patient Monitor** | Multi-parameter diagnostic with ECG, SpO2, BP sensing |
| **Signal Processing** | Digital filtering, R-peak detection, arrhythmia analysis |
| **Pacemaker** | Implantable pulse generator with sensing and pacing leads |
| **DDD Mode** | Dual-chamber pacing with sensing and pacing in both chambers |
| **Infusion Pump** | Drug delivery with multiple safety interlocks |
| **Defense in Depth** | Multiple independent safety barriers |
| **IEC 62304** | Medical device software lifecycle standard |
| **Risk Management** | ISO 14971 hazard analysis integrated into development |

---

## Quick Revision Questions

1. **Explain the FDA device classification system. What regulatory path is required for a Class III device?**

2. **Describe the signal processing pipeline for ECG R-peak detection. Why are notch and highpass filters necessary?**

3. **What are the key components of an implantable pacemaker? How does it achieve 10+ year battery life?**

4. **Explain DDD pacing mode. What are the AV interval and VA interval?**

5. **List five safety mechanisms used in infusion pumps. Why is dual-controller architecture used?**

6. **What are the documentation requirements for IEC 62304 Class C software? How does this differ from Class A?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Chapter 8.1: Automotive Systems](01-automotive-systems.md) | [Unit 8: Case Studies](README.md) | [Chapter 8.3: IoT Applications](03-iot-applications.md) |

---
