# Chapter 2.5: Power Supply Design

## Chapter Overview

Power supply is the lifeblood of any embedded system. This chapter covers power supply architectures, voltage regulation techniques, battery management, and power optimization strategies essential for embedded system design.

---

## 2.5.1 Power Supply Fundamentals

### Embedded System Power Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    POWER DISTRIBUTION ARCHITECTURE                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │                     POWER FLOW DIAGRAM                         │ │
│  │                                                                │ │
│  │  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐│ │
│  │  │  Power   │    │  Power   │    │  Voltage │    │   Load   ││ │
│  │  │  Source  │───►│Protection│───►│Regulation│───►│  (MCU,   ││ │
│  │  │          │    │          │    │          │    │ Sensors) ││ │
│  │  └──────────┘    └──────────┘    └──────────┘    └──────────┘│ │
│  │                                                                │ │
│  │  Sources:        Protection:      Regulation:     Loads:      │ │
│  │  • AC Mains      • Fuses          • Linear (LDO)  • MCU       │ │
│  │  • Battery       • TVS Diodes     • Switching     • Memory    │ │
│  │  • USB           • Polarity Prot  • Charge Pump   • Sensors   │ │
│  │  • Solar         • Overcurrent    • Multiple      • Actuators │ │
│  │  • PoE           • ESD Protection   Rails         • RF Modules│ │
│  │                                                                │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  VOLTAGE RAILS IN TYPICAL EMBEDDED SYSTEM:                          │
│                                                                      │
│              12V Input                                               │
│                 │                                                    │
│                 ▼                                                    │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                    Buck Converter                            │   │
│  └──────────────────────────┬───────────────────────────────────┘   │
│                              │                                       │
│              ┌───────────────┼───────────────┐                      │
│              ▼               ▼               ▼                      │
│         ┌────────┐      ┌────────┐      ┌────────┐                 │
│         │  5V    │      │  3.3V  │      │  1.8V  │                 │
│         │  Rail  │      │  Rail  │      │  Rail  │                 │
│         └───┬────┘      └───┬────┘      └───┬────┘                 │
│             │               │               │                       │
│         ┌───┴───┐       ┌───┴───┐       ┌───┴───┐                  │
│         │ Motor │       │  MCU  │       │ Core  │                  │
│         │ Driver│       │ I/O   │       │ Logic │                  │
│         │ USB   │       │Sensors│       │Memory │                  │
│         └───────┘       └───────┘       └───────┘                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Power Budget Calculation

```
┌─────────────────────────────────────────────────────────────────────┐
│                    POWER BUDGET ANALYSIS                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  POWER CALCULATION FORMULAS:                                         │
│                                                                      │
│  P = V × I                    (Power = Voltage × Current)           │
│  P = I² × R                   (Power in resistive load)             │
│  P = V² / R                   (Power from voltage)                  │
│                                                                      │
│  EXAMPLE POWER BUDGET TABLE:                                         │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Component      │ Voltage │ Current  │ Power   │ Duty Cycle │    │
│  │────────────────────────────────────────────────────────────── │   │
│  │ STM32F4 MCU    │  3.3V   │  50 mA   │ 165 mW  │   100%     │    │
│  │ WiFi Module    │  3.3V   │ 250 mA   │ 825 mW  │    10%     │    │
│  │ LCD Display    │  3.3V   │  30 mA   │  99 mW  │    50%     │    │
│  │ Sensors (×4)   │  3.3V   │  20 mA   │  66 mW  │   100%     │    │
│  │ LED Indicators │  3.3V   │  10 mA   │  33 mW  │    20%     │    │
│  │ Motor Driver   │  5.0V   │ 500 mA   │2500 mW  │    25%     │    │
│  │────────────────────────────────────────────────────────────── │   │
│  │ TOTAL (Peak)   │    -    │ 860 mA   │3688 mW  │     -      │    │
│  │ TOTAL (Average)│    -    │ 285 mA   │ 850 mW  │     -      │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  AVERAGE CURRENT CALCULATION:                                        │
│                                                                      │
│  I_avg = Σ (I_component × Duty_Cycle)                               │
│                                                                      │
│  WiFi: 250 mA × 10% = 25 mA average                                 │
│  Motor: 500 mA × 25% = 125 mA average                               │
│                                                                      │
│  BATTERY LIFE ESTIMATION:                                            │
│                                                                      │
│  Battery Life (hours) = Battery Capacity (mAh) / I_avg (mA)         │
│                                                                      │
│  Example: 2000 mAh / 285 mA = 7 hours                               │
│                                                                      │
│  With 80% efficiency factor:                                         │
│  Actual Life = 7 hours × 0.8 = 5.6 hours                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2.5.2 Voltage Regulators

### Linear Regulators (LDO)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    LINEAR (LDO) REGULATORS                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  LDO INTERNAL BLOCK DIAGRAM:                                         │
│                                                                      │
│           V_in                              V_out                    │
│             │                                 │                      │
│             ▼                                 ▼                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │          │                                 │                 │   │
│  │          │    ┌─────────────────┐         │                 │   │
│  │          └───►│   Pass Element  │─────────┴──────►          │   │
│  │               │   (MOSFET)      │                            │   │
│  │               └────────┬────────┘                            │   │
│  │                        │                                     │   │
│  │                   Gate │                                     │   │
│  │                   Drive│                                     │   │
│  │                        │                                     │   │
│  │               ┌────────┴────────┐                            │   │
│  │  V_ref ──────►│   Error Amp     │                            │   │
│  │               │                 │                            │   │
│  │               └────────┬────────┘                            │   │
│  │                        │                                     │   │
│  │                   ┌────┴────┐                                │   │
│  │                   │ Voltage │◄──────────────────────┐       │   │
│  │                   │ Divider │                       │       │   │
│  │                   └─────────┘                       │       │   │
│  │                                                     │       │   │
│  │                                              Feedback│       │   │
│  │                                                     │       │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  KEY PARAMETERS:                                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Parameter           │ Definition                           │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Dropout Voltage     │ Min (V_in - V_out) for regulation    │    │
│  │                     │ LDO: 0.1-0.5V, Standard: 2-3V        │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Quiescent Current   │ Current consumed by regulator itself │    │
│  │ (Iq)                │ Range: 1µA - 10mA                    │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Line Regulation     │ ΔV_out / ΔV_in (typically 0.01-0.1%) │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Load Regulation     │ ΔV_out / ΔI_load (typically 0.1-1%)  │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ PSRR                │ Power Supply Rejection Ratio (dB)    │    │
│  │                     │ Higher = better noise rejection      │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  EFFICIENCY CALCULATION:                                             │
│                                                                      │
│  η = (V_out / V_in) × 100%                                          │
│                                                                      │
│  Example: 5V to 3.3V LDO                                            │
│  η = (3.3V / 5V) × 100% = 66%                                       │
│                                                                      │
│  Power Dissipation: P_loss = (V_in - V_out) × I_out                 │
│  P_loss = (5V - 3.3V) × 500mA = 0.85W (Heatsink may be needed!)    │
│                                                                      │
│  COMMON LDO ICs:                                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Part Number │ V_out  │ I_max │ V_dropout │ Iq      │ Notes │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ LM7805      │ 5V     │ 1A    │ 2V        │ 5mA     │Classic│    │
│  │ AMS1117    │ 3.3V   │ 1A    │ 1.3V      │ 5mA     │Common │    │
│  │ MCP1700    │ 3.3V   │ 250mA │ 0.18V     │ 1.6µA   │Low Iq │    │
│  │ TPS7A33    │ Adj    │ 1A    │ 0.2V      │ 1mA     │Low Noise│   │
│  │ LP5907     │ 3.3V   │ 250mA │ 0.12V     │ 0.43µA  │Ultra-LP│    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Switching Regulators

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SWITCHING REGULATORS                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  BUCK CONVERTER (Step-Down):                                         │
│                                                                      │
│       V_in          L                                                │
│         │     ┌────────────┐                                        │
│         │     │            │                                        │
│    ┌────┴────┐│  ┌────┐   │    ┌─────┐                            │
│    │ Switch  │├──┤    ├───┴────┤ C_out├────► V_out                 │
│    │  (Q)    ││  │ L  │        │      │                            │
│    └────┬────┘│  └────┘        └──┬───┘                            │
│         │     │                    │                                │
│    ┌────┴────┐│                   │                                │
│    │ Diode   ││                   │                                │
│    │  (D)    ││                   │                                │
│    └────┬────┘│                   │                                │
│         │     │                   │                                │
│        GND   GND                 GND                                │
│                                                                      │
│  BUCK OPERATION WAVEFORMS:                                           │
│                                                                      │
│  Switch ────┐     ┌─────┐     ┌─────┐     ┌─────                   │
│  Control    └─────┘     └─────┘     └─────┘                        │
│              │◄──►│                                                 │
│               D×T   (1-D)×T                                         │
│                                                                      │
│  V_L     ────┐     ┌─────────┐     ┌─────────                      │
│  (Inductor)  │     │         │     │                               │
│              └─────┘         └─────┘                                │
│            Vin-Vout        -Vout                                    │
│                                                                      │
│  I_L        ╱╲      ╱╲      ╱╲      ╱╲                             │
│  (Inductor)╱  ╲    ╱  ╲    ╱  ╲    ╱  ╲                            │
│           ╱    ╲  ╱    ╲  ╱    ╲  ╱    ╲                           │
│                                                                      │
│  V_out = D × V_in                                                   │
│  Where D = Duty Cycle (0 to 1)                                      │
│                                                                      │
│  BOOST CONVERTER (Step-Up):                                          │
│                                                                      │
│                 L                                                    │
│   V_in    ┌───────┐          ┌─────┐                               │
│     │     │       │     D    │     │                               │
│     └─────┤   L   ├───────┬──┤ C   ├────► V_out                   │
│           │       │  │    │  │     │                               │
│           └───────┘  │    │  └──┬──┘                               │
│                   ┌──┴──┐ │     │                                  │
│                   │  Q  │ │     │                                  │
│                   └──┬──┘ │     │                                  │
│                      │    │     │                                  │
│                     GND  GND   GND                                  │
│                                                                      │
│  V_out = V_in / (1 - D)                                             │
│                                                                      │
│  BUCK-BOOST CONVERTER:                                               │
│  V_out = V_in × D / (1 - D)                                         │
│  Can step up or down depending on D                                 │
│                                                                      │
│  SWITCHING REGULATOR COMPARISON:                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Topology   │ V_out vs V_in │ Efficiency │ Complexity        │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Buck       │ V_out < V_in  │ 85-95%     │ Low               │    │
│  │ Boost      │ V_out > V_in  │ 80-95%     │ Medium            │    │
│  │ Buck-Boost │ Either        │ 80-90%     │ Medium            │    │
│  │ SEPIC      │ Either        │ 80-90%     │ High              │    │
│  │ Flyback    │ Isolated      │ 75-85%     │ High              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### LDO vs Switching Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│                    LDO vs SWITCHING REGULATORS                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  COMPARISON TABLE:                                                   │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Parameter        │ LDO              │ Switching           │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Efficiency       │ V_out/V_in       │ 85-95%              │    │
│  │                  │ (can be <50%)    │                     │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Output Noise     │ Very Low         │ Higher (switching)  │    │
│  │                  │ (excellent PSRR) │ (needs filtering)   │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Cost             │ Low              │ Higher              │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Size             │ Small            │ Larger (inductor)   │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Heat Dissipation │ High             │ Low                 │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ EMI              │ None             │ Significant         │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Step-up possible │ No               │ Yes                 │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  WHEN TO USE:                                                        │
│                                                                      │
│  USE LDO WHEN:                    USE SWITCHING WHEN:               │
│  • Low voltage drop (< 1V)        • Large voltage drop               │
│  • Low current (< 500mA)          • High current requirements        │
│  • Noise-sensitive analog         • Battery-powered devices          │
│  • Simple design needed           • Efficiency is critical           │
│  • Cost is primary concern        • Step-up/step-down needed         │
│                                                                      │
│  HYBRID APPROACH:                                                    │
│                                                                      │
│  V_in ──► [Switching] ──► [LDO] ──► V_out                           │
│  (12V)      (85% eff)     (clean)   (3.3V)                          │
│                                                                      │
│  High efficiency + Low noise output                                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2.5.3 Battery Technology

### Battery Types for Embedded Systems

```
┌─────────────────────────────────────────────────────────────────────┐
│                    BATTERY TECHNOLOGIES                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  BATTERY TYPE COMPARISON:                                            │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Type       │Voltage│Energy  │Cycles│Self    │Application   │    │
│  │            │(Cell) │Density │      │Dischg  │              │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Alkaline   │ 1.5V  │ 150    │ 1    │ 2%/yr  │Remote, toys  │    │
│  │ (Primary)  │       │ Wh/kg  │      │        │              │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ NiMH       │ 1.2V  │ 100    │ 500  │ 20%/mo │Tools, toys   │    │
│  │ (Recharge) │       │ Wh/kg  │      │        │              │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Li-Ion     │ 3.7V  │ 250    │ 500- │ 5%/mo  │Mobile, laptop│    │
│  │            │       │ Wh/kg  │ 1000 │        │              │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ LiPo       │ 3.7V  │ 200    │ 300- │ 5%/mo  │Drones, RC    │    │
│  │            │       │ Wh/kg  │ 500  │        │              │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ LiFePO4   │ 3.2V  │ 120    │ 2000+│ 3%/mo  │EV, Solar     │    │
│  │            │       │ Wh/kg  │      │        │              │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Coin Cell  │ 3.0V  │ 250    │ 1    │ 1%/yr  │RTC, backup   │    │
│  │ (CR2032)   │       │ Wh/kg  │      │        │              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  LITHIUM-ION DISCHARGE CURVE:                                        │
│                                                                      │
│  Voltage                                                             │
│  (V)                                                                 │
│  4.2V ─────────┐                                                    │
│                 ╲                                                    │
│  3.7V ──────────╲────────────────────────                           │
│                  ╲                      ╲                           │
│  3.0V ───────────────────────────────────╲──── Cutoff               │
│                                           ╲                         │
│  2.5V ─────────────────────────────────────╲─── Damage              │
│                                                                      │
│        0%    25%    50%    75%    100%                              │
│                  State of Charge (%)                                │
│                                                                      │
│  CHARGING PROFILE (Li-Ion):                                          │
│                                                                      │
│         │ Pre-  │ Constant │ Constant │ Top-up  │                  │
│         │charge │ Current  │ Voltage  │ (Float) │                  │
│         │       │          │          │         │                  │
│  Voltage│       │    ╱─────┼──────────┼─────────                   │
│         │      ╱│   ╱      │          │                            │
│         │    ╱  │  ╱       │          │                            │
│         │  ╱    │ ╱        │          │                            │
│         │╱      │╱         │          │                            │
│  Current│───────┼──────────╲          │                            │
│         │       │           ╲         │                            │
│         │       │            ╲        │                            │
│         │       │             ╲_______│                            │
│         └───────┴──────────────────────                            │
│            Time ──────────────────────►                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Battery Management System (BMS)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    BATTERY MANAGEMENT SYSTEM                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  BMS FUNCTIONS:                                                      │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │                     BMS BLOCK DIAGRAM                          │ │
│  │                                                                │ │
│  │  Battery    ┌────────────────────────────────────┐    Load    │ │
│  │  Pack   ───►│         BMS Controller             │───► /      │ │
│  │             │                                    │   Charger  │ │
│  │             │  ┌──────────┐   ┌──────────┐      │            │ │
│  │   Cell 1 ──►│  │ Voltage  │   │Protection│      │            │ │
│  │   Cell 2 ──►│  │Monitoring│   │ Logic    │      │            │ │
│  │   Cell n ──►│  └────┬─────┘   └────┬─────┘      │            │ │
│  │             │       │              │             │            │ │
│  │   Temp   ──►│  ┌────┴─────────────┴────┐        │            │ │
│  │  Sensor     │  │     MCU / ASIC        │        │            │ │
│  │             │  └────┬─────────────┬────┘        │            │ │
│  │   Current──►│       │             │             │            │ │
│  │   Sense     │  ┌────┴────┐   ┌────┴────┐       │            │ │
│  │             │  │ Cell    │   │ Fuel    │       │            │ │
│  │             │  │Balancing│   │ Gauge   │       │            │ │
│  │             │  └─────────┘   └─────────┘       │            │ │
│  │             └────────────────────────────────────┘            │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  PROTECTION FEATURES:                                                │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Protection       │ Threshold          │ Action             │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Over-voltage     │ > 4.25V per cell   │ Stop charging      │    │
│  │ Under-voltage    │ < 2.7V per cell    │ Disconnect load    │    │
│  │ Over-current     │ > rated current    │ Disconnect         │    │
│  │ Short-circuit    │ Immediate detect   │ Disconnect         │    │
│  │ Over-temperature │ > 60°C             │ Reduce/stop        │    │
│  │ Under-temperature│ < 0°C (charging)   │ Stop charging      │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  CELL BALANCING:                                                     │
│                                                                      │
│  Before Balancing:        After Balancing:                          │
│  ┌────┬────┬────┬────┐    ┌────┬────┬────┬────┐                    │
│  │4.2V│4.1V│3.9V│4.0V│    │4.0V│4.0V│4.0V│4.0V│                    │
│  └────┴────┴────┴────┘    └────┴────┴────┴────┘                    │
│   Cell1 2   3   4          Cell1 2   3   4                          │
│                                                                      │
│  Passive Balancing: Discharge higher cells through resistors       │
│  Active Balancing: Transfer energy from high to low cells          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2.5.4 Power Management ICs (PMIC)

### PMIC Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    POWER MANAGEMENT IC (PMIC)                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  INTEGRATED PMIC BLOCK DIAGRAM:                                      │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │                         PMIC                                   │ │
│  │                                                                │ │
│  │  V_BAT ──►┌──────────┐                                        │ │
│  │           │ Battery  │──► Battery Status                      │ │
│  │  V_USB ──►│ Charger  │                                        │ │
│  │           └────┬─────┘                                        │ │
│  │                │                                               │ │
│  │           ┌────┴─────┐                                        │ │
│  │           │  Power   │                                        │ │
│  │           │  Path    │                                        │ │
│  │           │ Control  │                                        │ │
│  │           └────┬─────┘                                        │ │
│  │                │                                               │ │
│  │     ┌──────────┼──────────┬──────────┬──────────┐            │ │
│  │     │          │          │          │          │            │ │
│  │  ┌──┴──┐   ┌──┴──┐   ┌──┴──┐   ┌──┴──┐   ┌──┴──┐          │ │
│  │  │Buck1│   │Buck2│   │LDO1 │   │LDO2 │   │LDO3 │          │ │
│  │  │3.3V │   │1.8V │   │2.8V │   │1.2V │   │3.0V │          │ │
│  │  └──┬──┘   └──┬──┘   └──┬──┘   └──┬──┘   └──┬──┘          │ │
│  │     │         │         │         │         │              │ │
│  │     ▼         ▼         ▼         ▼         ▼              │ │
│  │   Main      Memory    Camera    Core     Sensors           │ │
│  │   Logic      I/O      Module    Logic                      │ │
│  │                                                                │ │
│  │  ┌──────────────────────────────────────────────────────────┐ │ │
│  │  │              Control Interface (I2C)                     │ │ │
│  │  │  • Enable/Disable rails  • Set voltages                 │ │ │
│  │  │  • Read status           • Configure sequencing          │ │ │
│  │  └──────────────────────────────────────────────────────────┘ │ │
│  │                                                                │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  POWER-UP SEQUENCE:                                                  │
│                                                                      │
│  Time ───────────────────────────────────────────►                  │
│                                                                      │
│  Core    ─────────┐                                                 │
│  (1.2V)           └─────────────────────────────                    │
│                   │                                                  │
│  Memory  ─────────│───────┐                                         │
│  (1.8V)           │       └─────────────────────                    │
│                   │       │                                          │
│  I/O     ─────────│───────│───────┐                                 │
│  (3.3V)           │       │       └─────────────                    │
│                   │       │       │                                  │
│         POR    Rail 1  Rail 2  Rail 3                               │
│         Done   Stable  Stable  Stable                               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2.5.5 Protection Circuits

### Power Protection Design

```
┌─────────────────────────────────────────────────────────────────────┐
│                    POWER PROTECTION CIRCUITS                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  COMPLETE PROTECTION CIRCUIT:                                        │
│                                                                      │
│                 F1                TVS                                │
│  V_in ──┬──────┤├──────┬──────┬──────┬────────► To Regulator       │
│         │      Fuse    │      │      │                              │
│         │              │      │      │                              │
│         │         ┌────┴────┐ │  ┌───┴───┐                         │
│         │         │  Reverse│ │  │ Input │                         │
│         │         │  Polarity│ │  │ Filter│                         │
│         │         │  (Diode)│ │  │ (LC)  │                         │
│         │         └────┬────┘ │  └───┬───┘                         │
│         │              │      │      │                              │
│        GND            GND    GND    GND                             │
│                                                                      │
│  REVERSE POLARITY PROTECTION:                                        │
│                                                                      │
│  Method 1: Series Diode        Method 2: P-FET                      │
│  ┌────────────────────┐        ┌────────────────────┐              │
│  │                    │        │                    │              │
│  │  V_in ──►│├──► V_out        │  V_in ──┤►│──► V_out             │
│  │     (Schottky)      │        │    (P-MOSFET)      │              │
│  │                    │        │                    │              │
│  │  Loss: 0.3-0.5V    │        │  Loss: I²×Rds_on  │              │
│  │  (Low efficiency)  │        │  (~10mV)           │              │
│  └────────────────────┘        └────────────────────┘              │
│                                                                      │
│  ESD PROTECTION:                                                     │
│                                                                      │
│  Signal ──┬──────────┬──────────► To IC                            │
│           │          │                                              │
│        ┌──┴──┐   ┌──┴──┐                                           │
│        │ TVS │   │ R   │  (Series resistor limits current)        │
│        │     │   │100Ω │                                           │
│        └──┬──┘   └─────┘                                           │
│           │                                                         │
│          GND                                                        │
│                                                                      │
│  TVS DIODE CHARACTERISTICS:                                          │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Parameter          │ Symbol │ Typical Value               │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Standoff Voltage   │ V_RWM  │ Max working voltage         │    │
│  │ Breakdown Voltage  │ V_BR   │ Voltage where conduction    │    │
│  │ Clamping Voltage   │ V_C    │ Voltage during ESD event    │    │
│  │ Peak Pulse Current │ I_PP   │ Max surge current           │    │
│  │ Capacitance        │ C_J    │ For high-speed lines        │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Topic | Key Points | Formulas/Values |
|-------|-----------|-----------------|
| **Power Budget** | Calculate peak and average | P = V × I, Life = Capacity/I_avg |
| **LDO** | Simple, low noise, low efficiency | η = V_out/V_in |
| **Switching** | High efficiency, complex, noisy | Buck: V_out = D × V_in |
| **Li-Ion Battery** | 3.7V nominal, 4.2V max, 3.0V min | 500-1000 cycles |
| **BMS** | Protection, balancing, fuel gauge | OV: 4.25V, UV: 2.7V |
| **PMIC** | Integrated solution, I2C control | Multiple rails + charger |
| **Protection** | Fuse, TVS, reverse polarity | ESD: TVS diodes |

---

## Quick Revision Questions

1. **Calculate the efficiency of an LDO converting 12V to 5V at 500mA. What is the power dissipation?**

2. **What is the output voltage of a buck converter with V_in = 12V and duty cycle D = 0.25?**

3. **Compare the advantages and disadvantages of linear vs switching regulators.**

4. **What are the main protection features implemented in a Battery Management System?**

5. **Why is power sequencing important in multi-rail systems, and how is it typically implemented?**

6. **Design a reverse polarity protection circuit using a P-channel MOSFET and explain its operation.**

---

## Navigation

| Previous | Up | Next |
|----------|------|------|
| [Sensors and Actuators](04-sensors-and-actuators.md) | [Unit 2 Index](README.md) | [Unit 3: Embedded Software](../03-Embedded-Software/README.md) |

---
