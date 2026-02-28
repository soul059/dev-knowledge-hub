# Chapter 7.3: Prototyping and Development

## Introduction

Prototyping bridges the gap between concept and production. This chapter covers the tools, techniques, and best practices for developing embedded systems from initial proof-of-concept through to production-ready firmware. Effective prototyping reduces risk, validates designs early, and accelerates time-to-market.

---

## Prototyping Stages

```
┌─────────────────────────────────────────────────────────────────────┐
│              PROTOTYPE EVOLUTION                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  STAGE 1: PROOF OF CONCEPT                                          │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  • Development board (Arduino, Nucleo, etc.)                  │  │
│  │  • Breadboard connections                                     │  │
│  │  • Off-the-shelf modules                                      │  │
│  │  • Basic functionality demonstration                          │  │
│  │                                                                │  │
│  │  ┌─────────┐      ┌─────────────┐      ┌──────────┐         │  │
│  │  │ Arduino │─────▶│ Breadboard  │─────▶│ Modules  │         │  │
│  │  │  Uno    │      │ with wires  │      │ sensors  │         │  │
│  │  └─────────┘      └─────────────┘      └──────────┘         │  │
│  │                                                                │  │
│  │  Goal: "Can this work?"                                       │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                          │                                          │
│                          ▼                                          │
│  STAGE 2: ENGINEERING PROTOTYPE                                     │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  • Target MCU evaluation board                                │  │
│  │  • Custom daughter boards or shields                          │  │
│  │  • Closer to final component selection                        │  │
│  │  • Performance and timing validation                          │  │
│  │                                                                │  │
│  │  ┌─────────┐      ┌─────────────┐      ┌──────────┐         │  │
│  │  │ Nucleo  │─────▶│ Custom PCB  │─────▶│ Selected │         │  │
│  │  │ STM32   │      │ Prototype   │      │components│         │  │
│  │  └─────────┘      └─────────────┘      └──────────┘         │  │
│  │                                                                │  │
│  │  Goal: "Does it meet requirements?"                           │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                          │                                          │
│                          ▼                                          │
│  STAGE 3: PRE-PRODUCTION PROTOTYPE                                  │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  • Full custom PCB design                                     │  │
│  │  • Production components                                      │  │
│  │  • Form factor close to final                                 │  │
│  │  • EMC, environmental testing                                 │  │
│  │                                                                │  │
│  │  ┌────────────────────────────────────────────────┐          │  │
│  │  │            Production-like PCB                  │          │  │
│  │  │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐          │          │  │
│  │  │  │ MCU  │ │Power │ │Sensor│ │ Conn │          │          │  │
│  │  │  └──────┘ └──────┘ └──────┘ └──────┘          │          │  │
│  │  └────────────────────────────────────────────────┘          │  │
│  │                                                                │  │
│  │  Goal: "Ready for production?"                                │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                          │                                          │
│                          ▼                                          │
│  STAGE 4: PRODUCTION                                                │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │  • Final manufacturing files                                  │  │
│  │  • Production testing fixtures                                │  │
│  │  • Assembly documentation                                     │  │
│  │  • Mass production                                            │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Development Boards

### Popular Development Platforms

```
┌─────────────────────────────────────────────────────────────────────┐
│              DEVELOPMENT BOARD COMPARISON                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  BEGINNER-FRIENDLY:                                                  │
│  ┌──────────────┬────────────┬──────────────┬───────────────────┐  │
│  │ Board        │ MCU        │ Clock       │ Best For          │  │
│  ├──────────────┼────────────┼──────────────┼───────────────────┤  │
│  │ Arduino Uno  │ ATmega328P │ 16 MHz      │ Learning, hobby   │  │
│  │ Arduino Mega │ ATmega2560 │ 16 MHz      │ More I/O needed   │  │
│  │ Arduino Nano │ ATmega328P │ 16 MHz      │ Small projects    │  │
│  └──────────────┴────────────┴──────────────┴───────────────────┘  │
│                                                                      │
│  ARM CORTEX-M (Professional):                                        │
│  ┌──────────────┬────────────┬──────────────┬───────────────────┐  │
│  │ Board        │ MCU        │ Clock       │ Best For          │  │
│  ├──────────────┼────────────┼──────────────┼───────────────────┤  │
│  │ Nucleo-F401  │ STM32F401  │ 84 MHz      │ General purpose   │  │
│  │ Nucleo-F446  │ STM32F446  │ 180 MHz     │ High performance  │  │
│  │ Discovery    │ Various    │ Various     │ Specific features │  │
│  │ LPC1768      │ LPC1768    │ 96 MHz      │ Rapid prototyping │  │
│  │ FRDM-K64F    │ MK64F      │ 120 MHz     │ NXP development   │  │
│  └──────────────┴────────────┴──────────────┴───────────────────┘  │
│                                                                      │
│  WiFi/BLUETOOTH:                                                     │
│  ┌──────────────┬────────────┬──────────────┬───────────────────┐  │
│  │ Board        │ MCU        │ Connectivity │ Best For          │  │
│  ├──────────────┼────────────┼──────────────┼───────────────────┤  │
│  │ ESP32 DevKit │ ESP32      │ WiFi + BLE  │ IoT projects      │  │
│  │ ESP8266      │ ESP8266    │ WiFi        │ Simple IoT        │  │
│  │ nRF52 DK     │ nRF52832   │ BLE         │ Low-power BLE     │  │
│  │ Raspberry Pi │ BCM2711    │ WiFi + BLE  │ Linux embedded    │  │
│  │ Pico W       │ RP2040     │ WiFi        │ MicroPython       │  │
│  └──────────────┴────────────┴──────────────┴───────────────────┘  │
│                                                                      │
│  SELECTION CRITERIA:                                                 │
│  ├─ Does it have the peripherals I need?                           │
│  ├─ Is the MCU available for production?                           │
│  ├─ What's the toolchain/IDE support?                              │
│  ├─ Is there good documentation and community?                     │
│  └─ Does it fit my power/performance budget?                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Development Board Features

```
┌─────────────────────────────────────────────────────────────────────┐
│              NUCLEO-64 BOARD ARCHITECTURE                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                        ST-LINK/V2-1                         │   │
│  │              (Integrated Debugger/Programmer)                │   │
│  │  ┌─────────┐   ┌─────────┐   ┌─────────────────────┐       │   │
│  │  │ SWD     │   │ VCP     │   │    USB to PC        │       │   │
│  │  │ Debug   │   │ UART    │   │                     │       │   │
│  │  └────┬────┘   └────┬────┘   └──────────┬──────────┘       │   │
│  └───────┼─────────────┼───────────────────┼───────────────────┘   │
│          │             │                   │                        │
│  ════════╪═════════════╪═══════════════════╪════════════════════   │
│          │             │                   │                        │
│  ┌───────▼─────────────▼───────────────────▼───────────────────┐   │
│  │                    TARGET MCU                                │   │
│  │                   (STM32F4xx)                                │   │
│  │                                                              │   │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────────────┐    │   │
│  │  │ Arduino    │  │ Morpho     │  │ On-board:          │    │   │
│  │  │ Headers    │  │ Headers    │  │ • User LED (PA5)   │    │   │
│  │  │            │  │ (all pins) │  │ • User Button (PC13)│    │   │
│  │  │ D0-D15     │  │            │  │ • 32.768kHz Crystal │    │   │
│  │  │ A0-A5      │  │            │  │ • Reset Button     │    │   │
│  │  └────────────┘  └────────────┘  └────────────────────┘    │   │
│  │                                                              │   │
│  │  POWER OPTIONS:                                              │   │
│  │  • USB powered (5V from ST-LINK)                            │   │
│  │  • External power (VIN: 7-12V)                              │   │
│  │  • 3.3V or 5V logic level                                   │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  KEY FEATURES FOR PROTOTYPING:                                      │
│  ✓ No external programmer needed                                   │
│  ✓ Virtual COM port for debug output                               │
│  ✓ Compatible with Arduino shields                                 │
│  ✓ Full pin access via Morpho headers                              │
│  ✓ Online mbed and STM32CubeIDE support                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Development Tools

### Integrated Development Environments

```
┌─────────────────────────────────────────────────────────────────────┐
│              IDE OPTIONS FOR EMBEDDED DEVELOPMENT                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  VENDOR-SPECIFIC (Free):                                             │
│  ┌────────────────┬────────────────────────────────────────────┐   │
│  │ IDE            │ Description                                │   │
│  ├────────────────┼────────────────────────────────────────────┤   │
│  │ STM32CubeIDE   │ Eclipse-based, STM32 full support          │   │
│  │                │ CubeMX integration for pin/peripheral      │   │
│  │                │ config, free unlimited                      │   │
│  ├────────────────┼────────────────────────────────────────────┤   │
│  │ MCUXpresso     │ NXP LPC, Kinetis support                   │   │
│  │                │ Eclipse-based, config tools                 │   │
│  ├────────────────┼────────────────────────────────────────────┤   │
│  │ MPLAB X        │ Microchip PIC, dsPIC, SAM                  │   │
│  │                │ NetBeans-based                              │   │
│  ├────────────────┼────────────────────────────────────────────┤   │
│  │ Code Composer  │ Texas Instruments MSP430, C2000            │   │
│  │ Studio         │ Eclipse-based                               │   │
│  └────────────────┴────────────────────────────────────────────┘   │
│                                                                      │
│  COMMERCIAL (Paid):                                                  │
│  ┌────────────────┬────────────────────────────────────────────┐   │
│  │ IDE            │ Description                                │   │
│  ├────────────────┼────────────────────────────────────────────┤   │
│  │ Keil MDK       │ ARM Cortex-M, professional debugger        │   │
│  │                │ RTOS-aware debugging, code analysis        │   │
│  ├────────────────┼────────────────────────────────────────────┤   │
│  │ IAR Embedded   │ Multiple architectures, best optimization  │   │
│  │ Workbench      │ Safety certification support               │   │
│  ├────────────────┼────────────────────────────────────────────┤   │
│  │ Segger         │ ARM Cortex, excellent debugger             │   │
│  │ Embedded Studio│ Free for non-commercial ARM                │   │
│  └────────────────┴────────────────────────────────────────────┘   │
│                                                                      │
│  OPEN SOURCE:                                                        │
│  ┌────────────────┬────────────────────────────────────────────┐   │
│  │ Tool           │ Description                                │   │
│  ├────────────────┼────────────────────────────────────────────┤   │
│  │ VS Code +      │ Flexible, many extensions available        │   │
│  │ PlatformIO     │ Multi-platform, library management         │   │
│  ├────────────────┼────────────────────────────────────────────┤   │
│  │ Eclipse + GCC  │ Standard open-source toolchain             │   │
│  │ ARM            │ Requires manual setup                       │   │
│  └────────────────┴────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Debug and Programming Tools

```
┌─────────────────────────────────────────────────────────────────────┐
│              DEBUGGING INTERFACES                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  JTAG (Joint Test Action Group):                                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                                                              │   │
│  │   HOST PC                  DEBUG PROBE           TARGET      │   │
│  │  ┌────────┐              ┌──────────┐         ┌──────────┐  │   │
│  │  │  IDE   │◀───USB───────│  J-Link  │───JTAG──│   MCU    │  │   │
│  │  │Debugger│              │  ST-Link │         │          │  │   │
│  │  └────────┘              └──────────┘         └──────────┘  │   │
│  │                                                              │   │
│  │  JTAG Signals (20-pin):                                     │   │
│  │  • TCK - Test Clock                                         │   │
│  │  • TMS - Test Mode Select                                   │   │
│  │  • TDI - Test Data In                                       │   │
│  │  • TDO - Test Data Out                                      │   │
│  │  • TRST - Test Reset (optional)                             │   │
│  │                                                              │   │
│  │  Features: Boundary scan, daisy-chain, full debug           │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  SWD (Serial Wire Debug) - ARM Cortex:                              │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                                                              │   │
│  │   Only 2 signals (+ ground):                                │   │
│  │   • SWDIO - Data (bidirectional)                            │   │
│  │   • SWCLK - Clock                                           │   │
│  │                                                              │   │
│  │   ┌────────────────────────────────────────────────────┐   │   │
│  │   │  PIN    │ JTAG      │ SWD                          │   │   │
│  │   ├─────────┼───────────┼──────────────────────────────┤   │   │
│  │   │ Pins    │ 4-5 data  │ 2 data                       │   │   │
│  │   │ Speed   │ Full      │ Same                         │   │   │
│  │   │ Daisy   │ Yes       │ No (single target)          │   │   │
│  │   │ Boundary│ Yes       │ No                           │   │   │
│  │   └─────────┴───────────┴──────────────────────────────┘   │   │
│  │                                                              │   │
│  │   Preferred for ARM due to fewer pins                       │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  POPULAR DEBUG PROBES:                                              │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Probe          │ Price    │ Features                       │   │
│  ├────────────────┼──────────┼────────────────────────────────┤   │
│  │ ST-Link V2     │ $3-20    │ STM32, basic debugging         │   │
│  │ ST-Link V3     │ $35      │ Faster, SWO trace              │   │
│  │ J-Link EDU     │ $60      │ Multi-vendor, fast, RTT        │   │
│  │ J-Link Pro     │ $400+    │ Professional, full features    │   │
│  │ CMSIS-DAP      │ $15-30   │ Open source, many clones       │   │
│  │ Black Magic    │ $60      │ Built-in GDB server            │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Version Control for Embedded

### Git Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│              VERSION CONTROL STRUCTURE                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  REPOSITORY STRUCTURE:                                               │
│                                                                      │
│  project-name/                                                       │
│  ├── .git/                     # Git repository                     │
│  ├── .gitignore                # Ignore build artifacts             │
│  ├── README.md                 # Project documentation              │
│  ├── LICENSE                   # License file                       │
│  │                                                                   │
│  ├── docs/                     # Documentation                      │
│  │   ├── requirements/                                              │
│  │   ├── design/                                                    │
│  │   └── user-manual/                                               │
│  │                                                                   │
│  ├── hardware/                 # Hardware design files              │
│  │   ├── schematic/                                                 │
│  │   ├── pcb/                                                       │
│  │   └── bom/                                                       │
│  │                                                                   │
│  ├── firmware/                 # Embedded software                  │
│  │   ├── src/                  # Source files                       │
│  │   ├── include/              # Header files                       │
│  │   ├── drivers/              # HAL and drivers                    │
│  │   ├── lib/                  # Third-party libraries             │
│  │   ├── tests/                # Unit tests                        │
│  │   ├── CMakeLists.txt        # Build configuration               │
│  │   └── Makefile                                                   │
│  │                                                                   │
│  ├── tools/                    # Scripts, utilities                 │
│  │   ├── flash.sh                                                   │
│  │   └── test.sh                                                    │
│  │                                                                   │
│  └── build/                    # Build output (IGNORED)             │
│                                                                      │
│  .GITIGNORE EXAMPLE:                                                 │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ # Build outputs                                              │   │
│  │ build/                                                       │   │
│  │ *.o                                                          │   │
│  │ *.elf                                                        │   │
│  │ *.bin                                                        │   │
│  │ *.hex                                                        │   │
│  │ *.map                                                        │   │
│  │                                                              │   │
│  │ # IDE settings (may want to track some)                     │   │
│  │ .vscode/                                                     │   │
│  │ *.ioc                       # CubeMX (or track it)          │   │
│  │                                                              │   │
│  │ # OS files                                                   │   │
│  │ .DS_Store                                                    │   │
│  │ Thumbs.db                                                    │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Branching Strategy

```
┌─────────────────────────────────────────────────────────────────────┐
│              GIT BRANCHING FOR EMBEDDED                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  GITFLOW FOR FIRMWARE:                                              │
│                                                                      │
│  main (production releases)                                         │
│  ────●─────────────────────●─────────────────────●────▶            │
│      │                     │                     │                   │
│      │   v1.0.0            │   v1.1.0            │   v2.0.0         │
│      │                     │                     │                   │
│  develop                   │                     │                   │
│  ────●────●────●────●─────●────●────●────●─────●────●────▶        │
│           │    │    │           │    │    │                         │
│           │    │    │           │    │    └── bugfix/issue-45       │
│           │    │    │           │    └────── feature/ble-support    │
│           │    │    │           └─────────── hotfix/critical-bug    │
│           │    │    └────── feature/new-sensor                     │
│           │    └─────────── feature/power-save                     │
│           └──────────────── bugfix/timer-fix                       │
│                                                                      │
│  BRANCH NAMING:                                                      │
│  ├── feature/<name>     New functionality                          │
│  ├── bugfix/<issue>     Bug fixes                                  │
│  ├── hotfix/<name>      Urgent production fixes                    │
│  ├── release/<version>  Release preparation                        │
│  └── experiment/<name>  Experimental (may be discarded)            │
│                                                                      │
│  COMMIT MESSAGE FORMAT:                                              │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ type(scope): subject                                         │   │
│  │                                                              │   │
│  │ Body with details                                            │   │
│  │                                                              │   │
│  │ Fixes #123                                                   │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  EXAMPLES:                                                           │
│  • feat(sensor): add temperature compensation algorithm             │
│  • fix(uart): correct baud rate calculation for 115200              │
│  • refactor(hal): simplify GPIO initialization                      │
│  • docs(readme): update build instructions                          │
│  • test(adc): add unit tests for oversampling                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Build Systems

### Makefile Example

```makefile
# ================================================================
# Embedded Project Makefile
# ================================================================

# Project name
PROJECT = firmware

# Toolchain
CC = arm-none-eabi-gcc
OBJCOPY = arm-none-eabi-objcopy
SIZE = arm-none-eabi-size

# MCU settings
MCU = cortex-m4
FPU = fpv4-sp-d16
FLOAT-ABI = hard

# Directories
SRC_DIR = src
INC_DIR = include
DRV_DIR = drivers
BUILD_DIR = build

# Source files
SOURCES = $(wildcard $(SRC_DIR)/*.c)
SOURCES += $(wildcard $(DRV_DIR)/*.c)

# Object files
OBJECTS = $(patsubst %.c,$(BUILD_DIR)/%.o,$(notdir $(SOURCES)))

# Include paths
INCLUDES = -I$(INC_DIR) -I$(DRV_DIR)

# Compiler flags
CFLAGS = -mcpu=$(MCU) -mthumb -mfpu=$(FPU) -mfloat-abi=$(FLOAT-ABI)
CFLAGS += -Wall -Wextra -Werror
CFLAGS += -Os -g
CFLAGS += -ffunction-sections -fdata-sections
CFLAGS += $(INCLUDES)

# Linker flags
LDFLAGS = -mcpu=$(MCU) -mthumb
LDFLAGS += -T linker_script.ld
LDFLAGS += -Wl,--gc-sections
LDFLAGS += -Wl,-Map=$(BUILD_DIR)/$(PROJECT).map

# Targets
.PHONY: all clean flash

all: $(BUILD_DIR)/$(PROJECT).elf $(BUILD_DIR)/$(PROJECT).bin size

$(BUILD_DIR)/$(PROJECT).elf: $(OBJECTS)
	@echo "Linking..."
	$(CC) $(LDFLAGS) -o $@ $^

$(BUILD_DIR)/$(PROJECT).bin: $(BUILD_DIR)/$(PROJECT).elf
	$(OBJCOPY) -O binary $< $@

$(BUILD_DIR)/%.o: $(SRC_DIR)/%.c | $(BUILD_DIR)
	@echo "Compiling $<..."
	$(CC) $(CFLAGS) -c -o $@ $<

$(BUILD_DIR)/%.o: $(DRV_DIR)/%.c | $(BUILD_DIR)
	@echo "Compiling $<..."
	$(CC) $(CFLAGS) -c -o $@ $<

$(BUILD_DIR):
	mkdir -p $(BUILD_DIR)

size: $(BUILD_DIR)/$(PROJECT).elf
	$(SIZE) $<

clean:
	rm -rf $(BUILD_DIR)

flash: $(BUILD_DIR)/$(PROJECT).bin
	st-flash write $< 0x8000000
```

### CMake Example

```cmake
# ================================================================
# CMakeLists.txt for Embedded ARM Project
# ================================================================

cmake_minimum_required(VERSION 3.16)

# Toolchain file (set before project)
set(CMAKE_TOOLCHAIN_FILE ${CMAKE_SOURCE_DIR}/arm-none-eabi.cmake)

project(firmware C ASM)

# MCU-specific settings
set(MCU_FAMILY STM32F4xx)
set(MCU_MODEL STM32F401xE)
set(CPU_PARAMETERS
    -mcpu=cortex-m4
    -mthumb
    -mfpu=fpv4-sp-d16
    -mfloat-abi=hard
)

# Executable
add_executable(${PROJECT_NAME}.elf)

# Source files
target_sources(${PROJECT_NAME}.elf PRIVATE
    src/main.c
    src/system.c
    drivers/gpio.c
    drivers/uart.c
    startup/startup_stm32f401xe.s
)

# Include directories
target_include_directories(${PROJECT_NAME}.elf PRIVATE
    include
    drivers
    ${MCU_FAMILY}/Include
    CMSIS/Include
)

# Compiler definitions
target_compile_definitions(${PROJECT_NAME}.elf PRIVATE
    ${MCU_MODEL}
    USE_HAL_DRIVER
)

# Compiler flags
target_compile_options(${PROJECT_NAME}.elf PRIVATE
    ${CPU_PARAMETERS}
    -Wall -Wextra -Werror
    -Os -g
    -ffunction-sections
    -fdata-sections
)

# Linker flags
target_link_options(${PROJECT_NAME}.elf PRIVATE
    ${CPU_PARAMETERS}
    -T${CMAKE_SOURCE_DIR}/linker_script.ld
    -Wl,--gc-sections
    -Wl,-Map=${PROJECT_NAME}.map
    --specs=nano.specs
)

# Generate binary and hex files
add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -O binary 
            ${PROJECT_NAME}.elf ${PROJECT_NAME}.bin
    COMMAND ${CMAKE_OBJCOPY} -O ihex 
            ${PROJECT_NAME}.elf ${PROJECT_NAME}.hex
    COMMAND ${CMAKE_SIZE} ${PROJECT_NAME}.elf
)
```

---

## Rapid Prototyping Techniques

### Hardware Prototyping

```
┌─────────────────────────────────────────────────────────────────────┐
│              RAPID HARDWARE PROTOTYPING                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  BREADBOARD (Solderless):                                           │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  + ─────────────────────────────────────────────────── +    │   │
│  │  - ─────────────────────────────────────────────────── -    │   │
│  │                                                              │   │
│  │  ● ● ● ● ● │ │ ● ● ● ● ●    (rows connected)              │   │
│  │  ● ● ● ● ● │ │ ● ● ● ● ●                                   │   │
│  │  ● ● ● ● ● │ │ ● ● ● ● ●    Each row of 5 connected      │   │
│  │  ● ● ● ● ● │ │ ● ● ● ● ●    Center gap for ICs           │   │
│  │  ● ● ● ● ● │ │ ● ● ● ● ●                                   │   │
│  │                                                              │   │
│  │  + ─────────────────────────────────────────────────── +    │   │
│  │  - ─────────────────────────────────────────────────── -    │   │
│  └─────────────────────────────────────────────────────────────┘   │
│  ✓ Fast iteration     ✗ Poor for high frequency                   │
│  ✓ No soldering       ✗ Unreliable connections                    │
│                                                                      │
│  PERFBOARD (Prototype board):                                       │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○                               │   │
│  │  ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○    Individual holes           │   │
│  │  ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○    Wire or solder connections │   │
│  │  ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○                               │   │
│  └─────────────────────────────────────────────────────────────┘   │
│  ✓ More permanent     ✗ Time consuming                             │
│  ✓ Better for testing ✗ Hard to modify                             │
│                                                                      │
│  MODULAR APPROACH (Grove, Qwiic, Click):                            │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                                                              │   │
│  │  ┌───────┐      ┌───────┐      ┌───────┐      ┌───────┐   │   │
│  │  │ Base  │══════│Sensor │══════│Display│══════│Comms  │   │   │
│  │  │ Board │ I2C  │Module │      │Module │      │Module │   │   │
│  │  └───────┘      └───────┘      └───────┘      └───────┘   │   │
│  │                                                              │   │
│  │  Standardized connectors, plug-and-play                     │   │
│  └─────────────────────────────────────────────────────────────┘   │
│  ✓ Very fast          ✗ Higher cost                                │
│  ✓ No wiring errors   ✗ Limited customization                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Software Prototyping

```
┌─────────────────────────────────────────────────────────────────────┐
│              RAPID SOFTWARE PROTOTYPING                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  LEVELS OF ABSTRACTION:                                              │
│                                                                      │
│  HIGH LEVEL (Fastest iteration):                                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  MicroPython / CircuitPython                                │   │
│  │  ┌─────────────────────────────────────────────────────┐   │   │
│  │  │ from machine import Pin, I2C                        │   │   │
│  │  │ import bme280                                        │   │   │
│  │  │                                                      │   │   │
│  │  │ i2c = I2C(0, sda=Pin(0), scl=Pin(1))               │   │   │
│  │  │ sensor = bme280.BME280(i2c=i2c)                     │   │   │
│  │  │ print(sensor.temperature)                           │   │   │
│  │  └─────────────────────────────────────────────────────┘   │   │
│  │  ✓ Interactive REPL    ✓ Easy to experiment                │   │
│  │  ✗ Slower execution    ✗ Limited MCU support               │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  MEDIUM LEVEL (Arduino ecosystem):                                  │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  Arduino Framework (C++)                                    │   │
│  │  ┌─────────────────────────────────────────────────────┐   │   │
│  │  │ #include <Wire.h>                                    │   │   │
│  │  │ #include <BME280.h>                                  │   │   │
│  │  │                                                      │   │   │
│  │  │ BME280 sensor;                                       │   │   │
│  │  │ void setup() { sensor.begin(); }                     │   │   │
│  │  │ void loop() { Serial.println(sensor.temp()); }       │   │   │
│  │  └─────────────────────────────────────────────────────┘   │   │
│  │  ✓ Huge library collection    ✓ Cross-platform            │   │
│  │  ✗ Abstraction overhead       ✗ Less control              │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  LOW LEVEL (Production code):                                       │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  Bare-metal C / HAL                                         │   │
│  │  ┌─────────────────────────────────────────────────────┐   │   │
│  │  │ #include "stm32f4xx.h"                               │   │   │
│  │  │ #include "i2c_driver.h"                              │   │   │
│  │  │                                                      │   │   │
│  │  │ uint8_t data[8];                                     │   │   │
│  │  │ i2c_read(I2C1, BME280_ADDR, data, 8);               │   │   │
│  │  │ temperature = bme280_calc_temp(data);                │   │   │
│  │  └─────────────────────────────────────────────────────┘   │   │
│  │  ✓ Full control          ✓ Optimized performance           │   │
│  │  ✗ Longer development    ✗ More expertise needed           │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  STRATEGY: Start high, move lower as design stabilizes             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Documentation

### Essential Documentation

```
┌─────────────────────────────────────────────────────────────────────┐
│              PROJECT DOCUMENTATION                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. README.md (Project overview)                                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ # Project Name                                              │   │
│  │                                                              │   │
│  │ ## Description                                               │   │
│  │ Brief description of what this project does.                │   │
│  │                                                              │   │
│  │ ## Hardware                                                  │   │
│  │ - MCU: STM32F401RE                                          │   │
│  │ - Sensors: BME280, MPU6050                                  │   │
│  │                                                              │   │
│  │ ## Building                                                  │   │
│  │ ```bash                                                      │   │
│  │ mkdir build && cd build                                     │   │
│  │ cmake .. && make                                            │   │
│  │ ```                                                          │   │
│  │                                                              │   │
│  │ ## Flashing                                                  │   │
│  │ ```bash                                                      │   │
│  │ make flash                                                   │   │
│  │ ```                                                          │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  2. HARDWARE.md (Hardware documentation)                            │
│  • Schematic overview                                               │
│  • Pin assignments table                                            │
│  • Power requirements                                               │
│  • BOM (Bill of Materials)                                          │
│                                                                      │
│  3. API Documentation (Doxygen comments)                            │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ /**                                                          │   │
│  │  * @brief Initialize the temperature sensor                 │   │
│  │  * @param config Pointer to configuration structure         │   │
│  │  * @return 0 on success, negative error code on failure    │   │
│  │  *                                                          │   │
│  │  * @note Must be called before reading temperature          │   │
│  │  * @see temp_sensor_read()                                  │   │
│  │  */                                                          │   │
│  │ int temp_sensor_init(const temp_config_t *config);          │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  4. CHANGELOG.md (Version history)                                  │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ # Changelog                                                  │   │
│  │                                                              │   │
│  │ ## [1.2.0] - 2024-01-15                                     │   │
│  │ ### Added                                                    │   │
│  │ - BLE connectivity support                                   │   │
│  │ - Low power sleep mode                                       │   │
│  │                                                              │   │
│  │ ### Fixed                                                    │   │
│  │ - UART buffer overflow issue (#45)                          │   │
│  │                                                              │   │
│  │ ## [1.1.0] - 2024-01-01                                     │   │
│  │ ...                                                          │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **Prototype Stages** | PoC → Engineering → Pre-production → Production |
| **Development Boards** | Arduino (learning), Nucleo/Discovery (professional), ESP32 (IoT) |
| **IDEs** | Vendor-specific (free), Commercial (Keil, IAR), Open source (VS Code) |
| **Debug Interfaces** | JTAG (full featured), SWD (2-wire, ARM preferred) |
| **Version Control** | Git with structured branching (main/develop/feature) |
| **Build Systems** | Makefile (simple), CMake (cross-platform) |
| **Rapid Prototyping** | Breadboard, modular systems, high-level languages |
| **Documentation** | README, API docs, changelog, hardware specs |

---

## Quick Revision Questions

1. **What are the four stages of prototype evolution? What is the goal of each stage?**

2. **Compare Arduino-based and ARM Cortex-M development boards. When would you choose each?**

3. **Explain the difference between JTAG and SWD debugging interfaces. Which would you use for an ARM Cortex-M4 project?**

4. **Describe a good Git branching strategy for embedded firmware development. What naming conventions would you use?**

5. **Compare Makefile and CMake for embedded builds. What are the advantages of each?**

6. **When prototyping software, why might you start with MicroPython and later move to C? What trade-offs are involved?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Chapter 7.2: Hardware-Software Co-Design](02-hardware-software-codesign.md) | [Unit 7: System Design](README.md) | [Chapter 7.4: Testing and Validation](04-testing-validation.md) |

---
