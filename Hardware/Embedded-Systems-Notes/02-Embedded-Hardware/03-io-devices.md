# Chapter 2.3: I/O Devices

## Chapter Overview

Input/Output (I/O) devices enable embedded systems to interact with the external world. This chapter covers the fundamental I/O mechanisms, including GPIO, timers, communication interfaces, and analog interfaces that form the bridge between the digital processor and real-world signals.

---

## 2.3.1 General Purpose I/O (GPIO)

### GPIO Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    GPIO PIN ARCHITECTURE                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                              VDD                                     │
│                               │                                      │
│                          ┌────┴────┐                                │
│                          │ Pull-up │ (Optional)                     │
│                          │ Resistor│                                │
│                          └────┬────┘                                │
│                               │                                      │
│  ┌─────────────────┐         │         ┌─────────────────┐         │
│  │   Data Out      │    ┌────┴────┐    │   External      │         │
│  │   Register      │───►│ Output  │────┤   Pin           │         │
│  └─────────────────┘    │ Driver  │    │                 │         │
│                         │ (P-MOS) │    │    ┌──────┐     │         │
│                         └────┬────┘    │────┤ Load │     │         │
│                              │         │    └──────┘     │         │
│                         ┌────┴────┐    │                 │         │
│                         │ Output  │    │                 │         │
│                         │ Driver  │◄───┤                 │         │
│                         │ (N-MOS) │    │                 │         │
│                         └────┬────┘    └─────────────────┘         │
│                              │                  │                   │
│                          ┌───┴───┐              │                   │
│                          │Pull-dn│              │                   │
│                          └───┬───┘              │                   │
│                              │                  │                   │
│                             GND                 │                   │
│                                                 │                   │
│  ┌─────────────────┐    ┌──────────┐           │                   │
│  │   Data In       │◄───│ Schmitt  │◄──────────┘                   │
│  │   Register      │    │ Trigger  │                               │
│  └─────────────────┘    └──────────┘                               │
│                                                                      │
│  GPIO MODES:                                                         │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Mode           │ Output    │ Input     │ Description       │     │
│  │────────────────────────────────────────────────────────────│     │
│  │ Input Float    │ Disabled  │ Enabled   │ High impedance    │     │
│  │ Input Pull-up  │ Disabled  │ Enabled   │ Internal pull-up  │     │
│  │ Input Pull-dn  │ Disabled  │ Enabled   │ Internal pull-down│     │
│  │ Output Push-Pull│ Enabled  │ Optional  │ Drive high/low    │     │
│  │ Output Open-Drain│ N-only  │ Optional  │ Needs external PU │     │
│  │ Analog         │ Disabled  │ Disabled  │ For ADC/DAC       │     │
│  │ Alternate Func │ Peripheral│ Peripheral│ UART, SPI, etc.   │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### GPIO Electrical Characteristics

```
┌─────────────────────────────────────────────────────────────────────┐
│                    GPIO ELECTRICAL PARAMETERS                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  INPUT THRESHOLDS (3.3V logic):                                     │
│                                                                      │
│  VDD (3.3V) ─────────────────────────────────────────               │
│                                                                      │
│  VIH (2.0V) ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─   Logic HIGH threshold           │
│                                                                      │
│  VHYST      ─────────────────────   Hysteresis band (Schmitt)       │
│                                                                      │
│  VIL (0.8V) ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─   Logic LOW threshold            │
│                                                                      │
│  GND (0V)   ─────────────────────────────────────────               │
│                                                                      │
│  OUTPUT DRIVE CAPABILITY:                                            │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Parameter          │ Typical Value    │ Notes              │     │
│  │────────────────────────────────────────────────────────────│     │
│  │ VOH (High output)  │ VDD - 0.4V       │ At rated current   │     │
│  │ VOL (Low output)   │ 0.4V             │ At rated current   │     │
│  │ IOH (Source curr)  │ 4-25 mA          │ Depends on pin     │     │
│  │ IOL (Sink curr)    │ 4-25 mA          │ Depends on pin     │     │
│  │ Total package      │ 100-200 mA       │ All pins combined  │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  LED DRIVING EXAMPLE:                                                │
│                                                                      │
│    MCU Pin                                                           │
│       │                                                              │
│       ├──────┬─────────────────────┐                                │
│       │      │                     │                                │
│       │   ┌──┴──┐               ┌──┴──┐                            │
│       │   │ R   │               │ LED │                            │
│       │   │330Ω │               │     │                            │
│       │   └──┬──┘               └──┬──┘                            │
│       │      │                     │                                │
│       │      └──────┬──────────────┘                                │
│       │             │                                               │
│      GND           GND                                              │
│   (Sinking)     (Sourcing)                                          │
│                                                                      │
│  I_LED = (VDD - V_LED - VOL) / R                                    │
│  I_LED = (3.3 - 2.0 - 0.4) / 330 = 2.7 mA                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2.3.2 Timer/Counter Peripherals

### Timer Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TIMER/COUNTER ARCHITECTURE                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │                      TIMER BLOCK DIAGRAM                       │ │
│  │                                                                │ │
│  │  Clock Source     Prescaler       Counter        Compare/     │ │
│  │  ┌─────────┐     ┌─────────┐     ┌─────────┐    Capture       │ │
│  │  │ System  │────►│  /1     │────►│         │                  │ │
│  │  │ Clock   │     │  /2     │     │  16/32  │   ┌─────────┐   │ │
│  │  └─────────┘     │  /4     │     │   bit   │──►│Compare 1│   │ │
│  │                  │  ...    │     │ Counter │   └────┬────┘   │ │
│  │  ┌─────────┐     │  /256   │     │         │        │        │ │
│  │  │External │────►│         │     │  CNT    │   ┌────▼────┐   │ │
│  │  │ Pin     │     └────┬────┘     └────┬────┘   │  Output │   │ │
│  │  └─────────┘          │               │        │  Control│   │ │
│  │                       │               │        └────┬────┘   │ │
│  │              Prescaler│        Counter│             │        │ │
│  │              Register │        Register             │        │ │
│  │                       │               │        PWM Output    │ │
│  │                       │               │                      │ │
│  │               ┌───────▼───────────────▼────────┐             │ │
│  │               │     INTERRUPT GENERATION       │             │ │
│  │               │  • Overflow                    │             │ │
│  │               │  • Compare match               │             │ │
│  │               │  • Capture event               │             │ │
│  │               └────────────────────────────────┘             │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  TIMER PERIOD CALCULATION:                                           │
│                                                                      │
│  Timer Period = (Prescaler × (Counter_Max + 1)) / f_clock           │
│                                                                      │
│  Example: 16-bit timer, 72 MHz clock, Prescaler = 72                │
│  Period = (72 × 65536) / 72,000,000 = 65.536 ms                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### PWM Generation

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PWM SIGNAL GENERATION                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PWM TIMING DIAGRAM:                                                 │
│                                                                      │
│  Counter   ARR ────────────────────────────────────────────         │
│  Value          \              /\              /\                    │
│             CCR  \────────────/──\────────────/──\──────            │
│                   \          /    \          /    \                  │
│                    \        /      \        /      \                 │
│                     \      /        \      /        \                │
│                      \    /          \    /          \               │
│              0 ───────\──/────────────\──/────────────\──           │
│                                                                      │
│  PWM        ──┐     ┌────┐     ┌────┐     ┌────┐                    │
│  Output       │     │    │     │    │     │    │                    │
│               └─────┘    └─────┘    └─────┘    └─────                │
│                                                                      │
│               │◄───────►│                                            │
│               Duty Cycle = CCR/ARR                                   │
│                                                                      │
│               │◄────────────────►│                                   │
│                   Period = ARR                                       │
│                                                                      │
│  PWM PARAMETERS:                                                     │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ ARR (Auto-Reload Register): Sets PWM period               │     │
│  │ CCR (Capture/Compare Register): Sets duty cycle           │     │
│  │ Duty Cycle = CCR / ARR × 100%                             │     │
│  │ Frequency = f_timer / ARR                                  │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  EXAMPLE: Generate 1 kHz PWM with 75% duty cycle                    │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ f_clock = 72 MHz, Prescaler = 72 → f_timer = 1 MHz         │     │
│  │ ARR = f_timer / f_pwm = 1,000,000 / 1,000 = 1000           │     │
│  │ CCR = ARR × 0.75 = 750                                      │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2.3.3 Communication Interfaces

### Serial Communication Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                SERIAL COMMUNICATION INTERFACES                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  COMPARISON TABLE:                                                   │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Protocol │ Wires │ Speed      │ Topology    │ Use Case    │     │
│  │──────────────────────────────────────────────────────────── │    │
│  │ UART     │ 2     │ 115.2 kbps │ Point-Point │ Debug, GPS  │     │
│  │ SPI      │ 4+    │ 50+ Mbps   │ Master-Slave│ Flash, ADC  │     │
│  │ I2C      │ 2     │ 3.4 Mbps   │ Multi-Master│ Sensors     │     │
│  │ USB      │ 4     │ 480 Mbps   │ Host-Device │ Peripherals │     │
│  │ CAN      │ 2     │ 1 Mbps     │ Multi-Master│ Automotive  │     │
│  │ Ethernet │ 4/8   │ 1+ Gbps    │ Network     │ Networking  │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  UART (Universal Asynchronous Receiver-Transmitter):                │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │                                                            │     │
│  │  Device A              Device B                            │     │
│  │  ┌──────┐              ┌──────┐                           │     │
│  │  │      │ TX ────────► │ RX   │                           │     │
│  │  │ UART │              │ UART │                           │     │
│  │  │      │ RX ◄──────── │ TX   │                           │     │
│  │  │      │ GND ──────── │ GND  │                           │     │
│  │  └──────┘              └──────┘                           │     │
│  │                                                            │     │
│  │  Frame Format:                                             │     │
│  │  ┌─────┬───┬───┬───┬───┬───┬───┬───┬───┬─────┬──────┐    │     │
│  │  │Start│ D0│ D1│ D2│ D3│ D4│ D5│ D6│ D7│Parity│Stop  │    │     │
│  │  │ bit │   │   │   │   │   │   │   │   │(opt) │bit(s)│    │     │
│  │  └─────┴───┴───┴───┴───┴───┴───┴───┴───┴─────┴──────┘    │     │
│  │                                                            │     │
│  │  Baud Rate = f_clock / (16 × USARTDIV)                    │     │
│  │                                                            │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### SPI Interface

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SPI (SERIAL PERIPHERAL INTERFACE)                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  CONNECTION DIAGRAM:                                                 │
│                                                                      │
│       MASTER                           SLAVE                         │
│    ┌──────────┐                    ┌──────────┐                     │
│    │          │ SCLK ─────────────►│          │                     │
│    │          │ MOSI ─────────────►│          │                     │
│    │   MCU    │ MISO ◄───────────── │ Device  │                     │
│    │          │ SS   ─────────────►│          │                     │
│    │          │                    │          │                     │
│    └──────────┘                    └──────────┘                     │
│                                                                      │
│  MULTIPLE SLAVES:                                                    │
│                                                                      │
│       MASTER                SLAVE 1      SLAVE 2      SLAVE 3       │
│    ┌──────────┐          ┌────────┐   ┌────────┐   ┌────────┐      │
│    │          │ SCLK ───►│        │──►│        │──►│        │      │
│    │          │ MOSI ───►│        │──►│        │──►│        │      │
│    │          │ MISO ◄───│        │◄──│        │◄──│        │      │
│    │          │ SS1  ───►│ CS     │   │        │   │        │      │
│    │          │ SS2  ────│────────│──►│ CS     │   │        │      │
│    │          │ SS3  ────│────────│───│────────│──►│ CS     │      │
│    └──────────┘          └────────┘   └────────┘   └────────┘      │
│                                                                      │
│  SPI TIMING (Mode 0: CPOL=0, CPHA=0):                               │
│                                                                      │
│  SS    ─────┐                                       ┌───────        │
│             └───────────────────────────────────────┘               │
│                                                                      │
│  SCLK  ─────────┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐ ─────        │
│                 └──┘  └──┘  └──┘  └──┘  └──┘  └──┘  └──┘            │
│                                                                      │
│  MOSI  ─────────┬─────┬─────┬─────┬─────┬─────┬─────┬─────          │
│                 │ D7  │ D6  │ D5  │ D4  │ D3  │ D2  │ D1  │          │
│                 └─────┴─────┴─────┴─────┴─────┴─────┴─────          │
│                 ↑     ↑     ↑     ↑     ↑     ↑     ↑               │
│              Sample on rising edge                                   │
│                                                                      │
│  SPI MODES:                                                          │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Mode │ CPOL │ CPHA │ Clock Idle │ Data Sample              │   │
│  │──────────────────────────────────────────────────────────── │   │
│  │  0   │  0   │  0   │   Low      │ Rising edge              │   │
│  │  1   │  0   │  1   │   Low      │ Falling edge             │   │
│  │  2   │  1   │  0   │   High     │ Falling edge             │   │
│  │  3   │  1   │  1   │   High     │ Rising edge              │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### I2C Interface

```
┌─────────────────────────────────────────────────────────────────────┐
│                    I2C (INTER-INTEGRATED CIRCUIT)                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  BUS TOPOLOGY:                                                       │
│                                                                      │
│           VDD                                                        │
│            │                                                         │
│       ┌────┴────┐     ┌────┴────┐                                   │
│       │  R_pu   │     │  R_pu   │  (Pull-up resistors)             │
│       └────┬────┘     └────┬────┘                                   │
│            │               │                                         │
│    SDA ────┼───────────────┼────────────────────────────────        │
│            │               │          │          │                   │
│    SCL ────│───────────────│──────────│──────────│──────────        │
│            │               │          │          │                   │
│       ┌────┴────┐     ┌────┴────┐┌────┴────┐┌────┴────┐            │
│       │ MASTER  │     │ SLAVE 1 ││ SLAVE 2 ││ SLAVE 3 │            │
│       │  (MCU)  │     │  0x48   ││  0x50   ││  0x68   │            │
│       └─────────┘     └─────────┘└─────────┘└─────────┘            │
│                                                                      │
│  I2C FRAME FORMAT:                                                   │
│                                                                      │
│  ┌─────┬──────────────┬───┬──────────────┬───┬──────────────┬─────┐ │
│  │START│ 7-bit Address│R/W│     ACK      │   8-bit Data   │ ACK  │ │
│  └─────┴──────────────┴───┴──────────────┴───┴──────────────┴─────┘ │
│                                                                      │
│  I2C TIMING DIAGRAM:                                                 │
│                                                                      │
│  SDA  ──┐   ┌───────┐ ┌───────┐ ┌───────┐         ┌───             │
│         └───┘ A6    └─┘ A5    └─┘ A4..A0└─────────┘ ACK            │
│                                                                      │
│  SCL  ────────┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───           │
│               └─┘   └─┘   └─┘   └─┘   └─┘   └─┘   └─┘               │
│                                                                      │
│       START    Data bits transmitted               ACK              │
│    (SDA falls  on SCL high                     (Slave pulls         │
│    while SCL                                    SDA low)            │
│     is high)                                                        │
│                                                                      │
│  I2C SPEEDS:                                                         │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Mode              │ Speed        │ Pull-up Resistance      │   │
│  │──────────────────────────────────────────────────────────── │   │
│  │ Standard Mode     │ 100 kHz      │ 4.7 kΩ - 10 kΩ          │   │
│  │ Fast Mode         │ 400 kHz      │ 1 kΩ - 4.7 kΩ           │   │
│  │ Fast Mode Plus    │ 1 MHz        │ 500 Ω - 1 kΩ            │   │
│  │ High Speed Mode   │ 3.4 MHz      │ Special drivers         │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2.3.4 Analog Interfaces (ADC/DAC)

### ADC Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ADC (ANALOG TO DIGITAL CONVERTER)                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SAR ADC BLOCK DIAGRAM:                                              │
│                                                                      │
│  Analog    ┌─────────┐   ┌──────────┐   ┌─────────┐   Digital      │
│  Input ───►│ Sample  │──►│    SAR   │──►│ Digital │──► Output      │
│            │ & Hold  │   │   Logic  │   │ Output  │   (n-bits)     │
│            └─────────┘   └────┬─────┘   └─────────┘                │
│                               │                                      │
│                          ┌────┴────┐                                │
│                          │   DAC   │ (Internal)                     │
│                          └────┬────┘                                │
│                               │                                      │
│                          ┌────┴────┐                                │
│                          │Comparator│                               │
│                          └─────────┘                                │
│                                                                      │
│  ADC RESOLUTION AND STEP SIZE:                                       │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │                                                            │     │
│  │  Resolution (bits) │ Levels │ Step Size (3.3V ref)        │     │
│  │────────────────────────────────────────────────────────────│     │
│  │        8           │  256   │  12.89 mV                   │     │
│  │       10           │ 1024   │   3.22 mV                   │     │
│  │       12           │ 4096   │   0.81 mV                   │     │
│  │       16           │ 65536  │   0.05 mV                   │     │
│  │                                                            │     │
│  │  Step Size = V_ref / 2^n                                   │     │
│  │  Digital Value = (V_in / V_ref) × (2^n - 1)               │     │
│  │                                                            │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  ADC CONVERSION TIMING:                                              │
│                                                                      │
│  ──┬─────────────┬───────────────────┬──────────────┬──────────     │
│    │  Sampling   │    Conversion     │   Result     │               │
│    │   Time      │      Time         │   Ready      │               │
│    │◄───────────►│◄─────────────────►│◄────────────►│               │
│                                                                      │
│  Total Time = Sampling Time + Conversion Time                       │
│  Max Sample Rate = 1 / Total Time                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### DAC Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DAC (DIGITAL TO ANALOG CONVERTER)                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  R-2R LADDER DAC:                                                    │
│                                                                      │
│                  V_ref                                               │
│                    │                                                 │
│            ┌───────┴───────┐                                        │
│            │       R       │                                        │
│            └───────┬───────┘                                        │
│                    │                                                 │
│            ┌───────┼───────┐                                        │
│            │       │       │                                        │
│           2R       R      2R                                        │
│            │       │       │                                        │
│            ▼       │       ▼                                        │
│         ┌──┴──┐    │    ┌──┴──┐                                    │
│     D3 ─┤     │    │    │     ├─ D2                                │
│         │ SW  │    │    │ SW  │                                    │
│         └──┬──┘    │    └──┬──┘                                    │
│            │       │       │                                        │
│            ▼       ▼       ▼                                        │
│           GND     ...     GND          ───►  V_out                  │
│                                                                      │
│  V_out = V_ref × (D3×2³ + D2×2² + D1×2¹ + D0×2⁰) / 2^n             │
│                                                                      │
│  DAC OUTPUT WAVEFORM GENERATION:                                     │
│                                                                      │
│  Output                                                              │
│  Voltage    ┌──┐        ┌──┐        ┌──┐                            │
│        3.3V │  │   ┌────┤  │   ┌────┤  │                            │
│             │  │   │    │  │   │    │  │                            │
│        1.65V│  ├───┘    │  ├───┘    │  ├────                        │
│             │  │        │  │        │  │                            │
│          0V ┘  └────────┘  └────────┘  └────                        │
│                                                                      │
│             Digital values: 0, 128, 255, 128, 0, 128, 255...        │
│                            (8-bit example)                          │
│                                                                      │
│  APPLICATIONS:                                                       │
│  • Audio output                                                      │
│  • Waveform generation (sine, triangle, sawtooth)                   │
│  • Motor speed control (with PWM filter)                            │
│  • Programmable voltage references                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| I/O Type | Signals | Speed | Typical Use |
|----------|---------|-------|-------------|
| **GPIO** | Digital I/O | DC-MHz | LEDs, buttons, control |
| **Timer/PWM** | Timing signals | Variable | Motor control, timing |
| **UART** | Serial async | 115 kbps | Debug, GPS, modems |
| **SPI** | 4-wire sync | 50+ Mbps | Flash, displays, ADCs |
| **I2C** | 2-wire sync | 3.4 Mbps | Sensors, EEPROMs |
| **ADC** | Analog input | 1-5 MSPS | Sensors, audio |
| **DAC** | Analog output | 1 MSPS | Audio, waveforms |

---

## Quick Revision Questions

1. **What are the different GPIO modes and when would you use each?**

2. **Calculate the PWM frequency and duty cycle given ARR=1000 and CCR=250 with a 1 MHz timer clock.**

3. **Compare SPI and I2C interfaces in terms of wires, speed, and topology.**

4. **What is the step size of a 12-bit ADC with a 3.3V reference voltage?**

5. **Explain the difference between push-pull and open-drain output modes.**

6. **Draw the I2C start and stop conditions showing SDA and SCL signals.**

---

## Navigation

| Previous | Up | Next |
|----------|------|------|
| [Memory Types](02-memory-types.md) | [Unit 2 Index](README.md) | [Sensors and Actuators](04-sensors-and-actuators.md) |

---
