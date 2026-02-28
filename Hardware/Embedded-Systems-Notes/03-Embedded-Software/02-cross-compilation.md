# Chapter 3.2: Cross-Compilation and Toolchains

## Chapter Overview

Cross-compilation is the process of compiling code on a host machine to run on a different target architecture. This chapter covers toolchain components, linker scripts, build systems, and debugging techniques essential for embedded development.

---

## 3.2.1 Cross-Compilation Concepts

### Host vs Target Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CROSS-COMPILATION OVERVIEW                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  NATIVE vs CROSS COMPILATION:                                        │
│                                                                      │
│  Native Compilation:                                                 │
│  ┌──────────────────┐          ┌──────────────────┐                │
│  │   x86 PC         │          │   x86 PC         │                │
│  │                  │  build   │                  │                │
│  │  Source Code ───►│─────────►│  Executable      │                │
│  │                  │          │                  │                │
│  │  (gcc)           │          │  (runs here)     │                │
│  └──────────────────┘          └──────────────────┘                │
│                                                                      │
│  Cross Compilation:                                                  │
│  ┌──────────────────┐          ┌──────────────────┐                │
│  │   x86 PC (Host)  │          │   ARM MCU (Tgt)  │                │
│  │                  │  build   │                  │                │
│  │  Source Code ───►│─────────►│  Firmware        │                │
│  │                  │          │                  │                │
│  │  (arm-none-eabi- │  flash   │  (runs here)     │                │
│  │   gcc)           │─────────►│                  │                │
│  └──────────────────┘          └──────────────────┘                │
│                                                                      │
│  WHY CROSS-COMPILE:                                                  │
│  • Target has no OS to run compiler                                 │
│  • Target has limited resources                                     │
│  • Development is faster on powerful host                           │
│  • Easier debugging and testing                                     │
│                                                                      │
│  COMMON CROSS-COMPILER PREFIXES:                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Prefix              │ Target Architecture                   │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ arm-none-eabi-      │ ARM Cortex-M (bare-metal)            │    │
│  │ arm-linux-gnueabi-  │ ARM Linux                            │    │
│  │ avr-                │ AVR (Arduino)                         │    │
│  │ xtensa-esp32-elf-   │ ESP32                                 │    │
│  │ riscv32-unknown-elf-│ RISC-V                               │    │
│  │ mips-linux-gnu-     │ MIPS Linux                           │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.2.2 Toolchain Components

### Build Toolchain Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TOOLCHAIN COMPONENTS                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  COMPILATION STAGES:                                                 │
│                                                                      │
│  main.c ──► Preprocessor ──► Compiler ──► Assembler ──► Linker     │
│              (cpp)          (cc1)        (as)          (ld)         │
│                │              │            │             │          │
│                ▼              ▼            ▼             ▼          │
│            main.i         main.s       main.o       firmware.elf    │
│          (expanded)     (assembly)    (object)      (executable)    │
│                                                                      │
│  TOOLCHAIN PROGRAMS:                                                 │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Tool          │ Command              │ Function             │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Compiler      │ arm-none-eabi-gcc    │ C to object code     │    │
│  │ Assembler     │ arm-none-eabi-as     │ Assembly to object   │    │
│  │ Linker        │ arm-none-eabi-ld     │ Link objects → ELF   │    │
│  │ Archiver      │ arm-none-eabi-ar     │ Create libraries     │    │
│  │ Object copy   │ arm-none-eabi-objcopy│ Convert formats      │    │
│  │ Object dump   │ arm-none-eabi-objdump│ Disassemble/analyze  │    │
│  │ Size          │ arm-none-eabi-size   │ Show section sizes   │    │
│  │ nm            │ arm-none-eabi-nm     │ List symbols         │    │
│  │ Debugger      │ arm-none-eabi-gdb    │ Debug executable     │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  DETAILED BUILD FLOW:                                                │
│                                                                      │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐                           │
│  │ main.c  │   │ uart.c  │   │ gpio.c  │  Source files             │
│  └────┬────┘   └────┬────┘   └────┬────┘                           │
│       │             │             │                                 │
│       ▼             ▼             ▼                                 │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐                           │
│  │ main.o  │   │ uart.o  │   │ gpio.o  │  Object files             │
│  └────┬────┘   └────┬────┘   └────┬────┘                           │
│       │             │             │                                 │
│       └─────────────┼─────────────┘                                 │
│                     ▼                                               │
│              ┌────────────┐                                         │
│              │   Linker   │◄─── linker.ld (linker script)          │
│              │    (ld)    │◄─── startup.o (startup code)           │
│              └─────┬──────┘◄─── libc.a (C library)                 │
│                    │                                                │
│                    ▼                                                │
│              ┌────────────┐                                         │
│              │firmware.elf│  ELF executable                         │
│              └─────┬──────┘                                         │
│                    │                                                │
│        ┌───────────┼───────────┐                                   │
│        ▼           ▼           ▼                                   │
│   firmware.bin  firmware.hex  firmware.map                         │
│   (for flash)   (for flash)   (symbol map)                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.2.3 Linker Scripts

### Memory Layout Definition

```
┌─────────────────────────────────────────────────────────────────────┐
│                    LINKER SCRIPTS                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PURPOSE OF LINKER SCRIPT:                                           │
│  • Define memory regions (Flash, RAM)                               │
│  • Place code and data sections                                     │
│  • Set stack and heap locations                                     │
│  • Define entry point                                               │
│                                                                      │
│  BASIC LINKER SCRIPT STRUCTURE:                                      │
│                                                                      │
│  /* STM32F4 linker script example */                                │
│                                                                      │
│  /* Memory regions */                                                │
│  MEMORY                                                              │
│  {                                                                   │
│      FLASH (rx)  : ORIGIN = 0x08000000, LENGTH = 512K               │
│      SRAM (rwx)  : ORIGIN = 0x20000000, LENGTH = 128K               │
│  }                                                                   │
│                                                                      │
│  /* Entry point */                                                   │
│  ENTRY(Reset_Handler)                                                │
│                                                                      │
│  /* Section placement */                                             │
│  SECTIONS                                                            │
│  {                                                                   │
│      /* Vector table at start of Flash */                           │
│      .isr_vector :                                                   │
│      {                                                               │
│          . = ALIGN(4);                                              │
│          KEEP(*(.isr_vector))                                       │
│          . = ALIGN(4);                                              │
│      } > FLASH                                                       │
│                                                                      │
│      /* Program code in Flash */                                     │
│      .text :                                                         │
│      {                                                               │
│          . = ALIGN(4);                                              │
│          *(.text)           /* Code */                              │
│          *(.text*)          /* Code sections */                     │
│          *(.rodata)         /* Read-only data */                    │
│          *(.rodata*)                                                │
│          . = ALIGN(4);                                              │
│          _etext = .;        /* End of text symbol */                │
│      } > FLASH                                                       │
│                                                                      │
│      /* Initialized data (copied to RAM) */                         │
│      .data :                                                         │
│      {                                                               │
│          . = ALIGN(4);                                              │
│          _sdata = .;        /* Start of data */                     │
│          *(.data)                                                   │
│          *(.data*)                                                  │
│          . = ALIGN(4);                                              │
│          _edata = .;        /* End of data */                       │
│      } > SRAM AT> FLASH     /* Load from Flash, run in RAM */       │
│                                                                      │
│      /* Uninitialized data (zeroed in RAM) */                       │
│      .bss :                                                          │
│      {                                                               │
│          . = ALIGN(4);                                              │
│          _sbss = .;         /* Start of bss */                      │
│          *(.bss)                                                    │
│          *(.bss*)                                                   │
│          *(COMMON)                                                  │
│          . = ALIGN(4);                                              │
│          _ebss = .;         /* End of bss */                        │
│      } > SRAM                                                        │
│                                                                      │
│      /* Stack at end of RAM */                                       │
│      ._stack :                                                       │
│      {                                                               │
│          . = ALIGN(8);                                              │
│          . = . + 0x1000;    /* 4KB stack */                         │
│          _estack = .;       /* Top of stack */                      │
│      } > SRAM                                                        │
│  }                                                                   │
│                                                                      │
│  MEMORY LAYOUT VISUALIZATION:                                        │
│                                                                      │
│  FLASH (0x08000000)           SRAM (0x20000000)                     │
│  ┌─────────────────┐          ┌─────────────────┐                   │
│  │   ISR Vector    │          │      .data      │ ◄── Initialized   │
│  │   Table         │          │   (from Flash)  │     globals       │
│  ├─────────────────┤          ├─────────────────┤                   │
│  │                 │          │      .bss       │ ◄── Zero-init     │
│  │    .text        │          │   (zeroed)      │     globals       │
│  │   (Code)        │          ├─────────────────┤                   │
│  │                 │          │                 │                   │
│  ├─────────────────┤          │      Heap       │ ◄── Dynamic       │
│  │   .rodata       │          │       ↓         │     allocation    │
│  │ (Constants)     │          │                 │                   │
│  ├─────────────────┤          │       ↑         │                   │
│  │ .data (LMA)     │────────► │      Stack      │ ◄── Local vars    │
│  │ (Load address)  │  copy    │                 │                   │
│  └─────────────────┘          └─────────────────┘                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.2.4 Startup Code

### System Initialization

```
┌─────────────────────────────────────────────────────────────────────┐
│                    STARTUP CODE                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  STARTUP SEQUENCE:                                                   │
│                                                                      │
│  Power On / Reset                                                    │
│        │                                                             │
│        ▼                                                             │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │ 1. Load Stack Pointer from vector table (0x00000000)         │  │
│  │    SP = *((uint32_t *)0x00000000)                            │  │
│  └───────────────────────────────────────────────────────────────┘  │
│        │                                                             │
│        ▼                                                             │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │ 2. Jump to Reset_Handler (vector table offset 0x04)          │  │
│  │    PC = *((uint32_t *)0x00000004)                            │  │
│  └───────────────────────────────────────────────────────────────┘  │
│        │                                                             │
│        ▼                                                             │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │ 3. Reset_Handler:                                             │  │
│  │    a. Copy .data from Flash to RAM                           │  │
│  │    b. Zero-fill .bss section                                 │  │
│  │    c. Initialize system (clocks, peripherals)                │  │
│  │    d. Call __libc_init_array() (C++ constructors)            │  │
│  │    e. Call main()                                            │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  STARTUP CODE EXAMPLE (startup.c):                                   │
│                                                                      │
│  extern uint32_t _etext, _sdata, _edata, _sbss, _ebss, _estack;     │
│                                                                      │
│  void Reset_Handler(void) {                                         │
│      uint32_t *src, *dst;                                           │
│                                                                      │
│      /* Copy .data section from Flash to RAM */                     │
│      src = &_etext;                                                 │
│      dst = &_sdata;                                                 │
│      while (dst < &_edata) {                                        │
│          *dst++ = *src++;                                           │
│      }                                                               │
│                                                                      │
│      /* Zero-fill .bss section */                                   │
│      dst = &_sbss;                                                  │
│      while (dst < &_ebss) {                                         │
│          *dst++ = 0;                                                │
│      }                                                               │
│                                                                      │
│      /* Initialize system */                                        │
│      SystemInit();                                                   │
│                                                                      │
│      /* Call main() */                                               │
│      main();                                                         │
│                                                                      │
│      /* Never return, infinite loop */                              │
│      while(1);                                                       │
│  }                                                                   │
│                                                                      │
│  VECTOR TABLE STRUCTURE (ARM Cortex-M):                             │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Offset │ Vector           │ Description                    │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ 0x0000 │ Initial SP       │ Stack pointer initial value    │    │
│  │ 0x0004 │ Reset_Handler    │ Reset vector (entry point)     │    │
│  │ 0x0008 │ NMI_Handler      │ Non-maskable interrupt         │    │
│  │ 0x000C │ HardFault_Handler│ Hard fault                     │    │
│  │ 0x0010 │ MemManage_Handler│ Memory management fault        │    │
│  │ 0x0014 │ BusFault_Handler │ Bus fault                      │    │
│  │ 0x0018 │ UsageFault_Handler│ Usage fault                   │    │
│  │  ...   │ ...              │ ...                            │    │
│  │ 0x003C │ SysTick_Handler  │ System tick timer              │    │
│  │ 0x0040+│ IRQn_Handler     │ External interrupts            │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.2.5 Build Systems

### Makefile and CMake

```
┌─────────────────────────────────────────────────────────────────────┐
│                    BUILD SYSTEMS                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  MAKEFILE EXAMPLE:                                                   │
│                                                                      │
│  # Toolchain                                                         │
│  PREFIX = arm-none-eabi-                                            │
│  CC     = $(PREFIX)gcc                                              │
│  AS     = $(PREFIX)as                                               │
│  LD     = $(PREFIX)ld                                               │
│  OBJCPY = $(PREFIX)objcopy                                          │
│  SIZE   = $(PREFIX)size                                             │
│                                                                      │
│  # Compiler flags                                                    │
│  CFLAGS  = -mcpu=cortex-m4 -mthumb -mfloat-abi=hard                │
│  CFLAGS += -O2 -g -Wall -ffunction-sections -fdata-sections        │
│  CFLAGS += -I./inc                                                  │
│                                                                      │
│  # Linker flags                                                      │
│  LDFLAGS  = -T linker.ld                                            │
│  LDFLAGS += -Wl,--gc-sections -Wl,-Map=output.map                   │
│                                                                      │
│  # Source files                                                      │
│  SRCS = src/main.c src/uart.c src/gpio.c                           │
│  OBJS = $(SRCS:.c=.o)                                               │
│                                                                      │
│  # Target                                                            │
│  TARGET = firmware                                                   │
│                                                                      │
│  all: $(TARGET).bin                                                 │
│                                                                      │
│  $(TARGET).elf: $(OBJS) startup.o                                   │
│      $(CC) $(CFLAGS) $(LDFLAGS) -o $@ $^                            │
│      $(SIZE) $@                                                      │
│                                                                      │
│  $(TARGET).bin: $(TARGET).elf                                       │
│      $(OBJCPY) -O binary $< $@                                      │
│                                                                      │
│  %.o: %.c                                                            │
│      $(CC) $(CFLAGS) -c -o $@ $<                                    │
│                                                                      │
│  clean:                                                              │
│      rm -f $(OBJS) $(TARGET).elf $(TARGET).bin                      │
│                                                                      │
│  flash: $(TARGET).bin                                               │
│      openocd -f board/stm32f4discovery.cfg \                        │
│               -c "program $< reset exit"                            │
│                                                                      │
│  BUILD PROCESS:                                                      │
│                                                                      │
│  $ make clean                                                        │
│  $ make all                                                          │
│  $ make flash                                                        │
│                                                                      │
│  OUTPUT ANALYSIS:                                                    │
│                                                                      │
│  $ arm-none-eabi-size firmware.elf                                  │
│     text    data     bss     dec     hex filename                   │
│    12340     128    2048   14516    38b4 firmware.elf               │
│                                                                      │
│  text  = code + rodata (goes to Flash)                              │
│  data  = initialized variables (Flash → RAM)                        │
│  bss   = uninitialized variables (RAM, zeroed)                      │
│  dec   = text + data + bss (total)                                  │
│                                                                      │
│  Flash usage: text + data = 12468 bytes                             │
│  RAM usage: data + bss = 2176 bytes                                 │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.2.6 Debugging Techniques

### Debug Tools and Methods

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DEBUGGING TECHNIQUES                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  DEBUG INTERFACES:                                                   │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Interface │ Pins │ Speed  │ Features                       │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ JTAG      │ 5    │ Fast   │ Full debug, boundary scan      │    │
│  │ SWD       │ 2    │ Fast   │ ARM-specific, fewer pins       │    │
│  │ UART      │ 2    │ Slow   │ printf debugging               │    │
│  │ SWO       │ 1    │ Medium │ Trace output (ITM)             │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  GDB DEBUGGING SESSION:                                              │
│                                                                      │
│  # Terminal 1: Start OpenOCD                                        │
│  $ openocd -f interface/stlink.cfg -f target/stm32f4x.cfg           │
│                                                                      │
│  # Terminal 2: Start GDB                                            │
│  $ arm-none-eabi-gdb firmware.elf                                   │
│  (gdb) target remote :3333                                          │
│  (gdb) monitor reset halt                                           │
│  (gdb) load                                                          │
│  (gdb) break main                                                    │
│  (gdb) continue                                                      │
│                                                                      │
│  COMMON GDB COMMANDS:                                                │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Command           │ Description                            │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ break <loc>       │ Set breakpoint                         │    │
│  │ continue (c)      │ Continue execution                     │    │
│  │ step (s)          │ Step into function                     │    │
│  │ next (n)          │ Step over function                     │    │
│  │ print <var>       │ Print variable value                   │    │
│  │ x/10x 0x20000000  │ Examine memory (10 words, hex)         │    │
│  │ info registers    │ Show CPU registers                     │    │
│  │ backtrace (bt)    │ Show call stack                        │    │
│  │ watch <var>       │ Break on variable change               │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  PRINTF DEBUGGING (Semi-hosting/UART):                              │
│                                                                      │
│  // Retarget printf to UART                                         │
│  int _write(int fd, char *ptr, int len) {                           │
│      for (int i = 0; i < len; i++) {                                │
│          while(!(UART1->SR & UART_SR_TXE)); // Wait TX empty        │
│          UART1->DR = ptr[i];                                        │
│      }                                                               │
│      return len;                                                     │
│  }                                                                   │
│                                                                      │
│  // Usage                                                            │
│  printf("Value: %d\n", sensor_value);                               │
│                                                                      │
│  LED DEBUGGING (Blink Patterns):                                     │
│                                                                      │
│  Pattern         │ Meaning                                          │
│  ─────────────────────────────────────                              │
│  Slow blink      │ Normal operation                                 │
│  Fast blink      │ Error state                                      │
│  SOS pattern     │ Critical error                                   │
│  Solid ON        │ Stuck in loop                                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Description | Key Command/File |
|---------|-------------|------------------|
| **Cross-compiler** | Compiles for different architecture | `arm-none-eabi-gcc` |
| **Linker script** | Defines memory layout | `.ld` file |
| **Startup code** | Initializes system before main() | `startup.c/.s` |
| **Object file** | Compiled module, not yet linked | `.o` files |
| **ELF file** | Executable with debug info | `firmware.elf` |
| **Binary file** | Raw firmware for flashing | `firmware.bin` |
| **Map file** | Symbol addresses and sizes | `firmware.map` |
| **GDB** | GNU debugger for embedded | `arm-none-eabi-gdb` |

---

## Quick Revision Questions

1. **What is the difference between native compilation and cross-compilation?**

2. **List the main components of an embedded toolchain and their functions.**

3. **Explain the purpose of the MEMORY and SECTIONS blocks in a linker script.**

4. **What are the main tasks performed by startup code before main() is called?**

5. **How do you calculate Flash and RAM usage from the `arm-none-eabi-size` output?**

6. **What is the difference between JTAG and SWD debug interfaces?**

---

## Navigation

| Previous | Up | Next |
|----------|------|------|
| [Embedded C Programming](01-embedded-c-programming.md) | [Unit 3 Index](README.md) | [Firmware Architecture](03-firmware-architecture.md) |

---
