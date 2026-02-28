# Chapter 3.5: Boot Sequence

## Chapter Overview

The boot sequence is the process that occurs from power-on until the application code begins executing. This chapter covers the boot process for embedded systems, including reset handling, startup code, bootloaders, and firmware update mechanisms.

---

## 3.5.1 Boot Process Overview

### Power-On Reset Sequence

```
┌─────────────────────────────────────────────────────────────────────┐
│                    BOOT PROCESS OVERVIEW                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  COMPLETE BOOT SEQUENCE:                                             │
│                                                                      │
│  Power On                                                            │
│     │                                                                │
│     ▼                                                                │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ 1. HARDWARE RESET                                            │   │
│  │    • Power supply stabilization                              │   │
│  │    • Reset signal deasserted                                 │   │
│  │    • CPU starts from reset vector                            │   │
│  └──────────────────────────┬───────────────────────────────────┘   │
│                              │                                       │
│                              ▼                                       │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ 2. BOOT MODE SELECTION (MCU-specific)                        │   │
│  │    • Check BOOT pins or option bytes                         │   │
│  │    • Boot from: Flash, System Memory, or SRAM                │   │
│  └──────────────────────────┬───────────────────────────────────┘   │
│                              │                                       │
│                              ▼                                       │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ 3. VECTOR TABLE FETCH                                        │   │
│  │    • Load initial SP from address 0x00000000                 │   │
│  │    • Load Reset_Handler from address 0x00000004              │   │
│  └──────────────────────────┬───────────────────────────────────┘   │
│                              │                                       │
│                              ▼                                       │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ 4. STARTUP CODE (Reset_Handler)                              │   │
│  │    • Copy .data section from Flash to RAM                    │   │
│  │    • Zero-fill .bss section                                  │   │
│  │    • Initialize C runtime (optional)                         │   │
│  └──────────────────────────┬───────────────────────────────────┘   │
│                              │                                       │
│                              ▼                                       │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ 5. SYSTEM INITIALIZATION                                     │   │
│  │    • Configure clock system (PLL, prescalers)                │   │
│  │    • Initialize critical peripherals                         │   │
│  │    • Configure NVIC                                          │   │
│  └──────────────────────────┬───────────────────────────────────┘   │
│                              │                                       │
│                              ▼                                       │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ 6. MAIN APPLICATION                                          │   │
│  │    • Application-specific initialization                     │   │
│  │    • Enter main loop or RTOS scheduler                       │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  TIME BREAKDOWN (typical ARM Cortex-M):                             │
│                                                                      │
│  │ Reset │ Vector │ Startup │ SysInit │ App Init │ Main Loop │     │
│  │ ~1ms  │ ~10µs  │ ~100µs  │ ~1ms    │ varies   │           │     │
│  └───────┴────────┴─────────┴─────────┴──────────┴───────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.5.2 Vector Table

### ARM Cortex-M Vector Table Structure

```
┌─────────────────────────────────────────────────────────────────────┐
│                    VECTOR TABLE                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  VECTOR TABLE LAYOUT (ARM Cortex-M):                                │
│                                                                      │
│  Address        │ Vector              │ Description                 │
│  ───────────────────────────────────────────────────────────────    │
│  0x0000_0000    │ Initial SP          │ Stack pointer initial value │
│  0x0000_0004    │ Reset_Handler       │ Reset vector (entry point)  │
│  0x0000_0008    │ NMI_Handler         │ Non-Maskable Interrupt      │
│  0x0000_000C    │ HardFault_Handler   │ Hard Fault                  │
│  0x0000_0010    │ MemManage_Handler   │ MPU Fault                   │
│  0x0000_0014    │ BusFault_Handler    │ Bus Fault                   │
│  0x0000_0018    │ UsageFault_Handler  │ Usage Fault                 │
│  0x0000_001C    │ Reserved            │ -                           │
│  0x0000_0020    │ Reserved            │ -                           │
│  0x0000_0024    │ Reserved            │ -                           │
│  0x0000_0028    │ Reserved            │ -                           │
│  0x0000_002C    │ SVC_Handler         │ SuperVisor Call             │
│  0x0000_0030    │ DebugMon_Handler    │ Debug Monitor               │
│  0x0000_0034    │ Reserved            │ -                           │
│  0x0000_0038    │ PendSV_Handler      │ Pendable SV                 │
│  0x0000_003C    │ SysTick_Handler     │ System Tick Timer           │
│  0x0000_0040    │ IRQ0_Handler        │ External Interrupt #0       │
│  0x0000_0044    │ IRQ1_Handler        │ External Interrupt #1       │
│  ...            │ ...                 │ ...                         │
│                                                                      │
│  VECTOR TABLE IN CODE:                                               │
│                                                                      │
│  extern uint32_t _estack;  // From linker script                    │
│                                                                      │
│  __attribute__((section(".isr_vector")))                            │
│  const uint32_t vector_table[] = {                                  │
│      (uint32_t)&_estack,           // Initial Stack Pointer         │
│      (uint32_t)Reset_Handler,      // Reset                         │
│      (uint32_t)NMI_Handler,        // NMI                           │
│      (uint32_t)HardFault_Handler,  // Hard Fault                    │
│      (uint32_t)MemManage_Handler,  // MPU Fault                     │
│      (uint32_t)BusFault_Handler,   // Bus Fault                     │
│      (uint32_t)UsageFault_Handler, // Usage Fault                   │
│      0, 0, 0, 0,                   // Reserved                      │
│      (uint32_t)SVC_Handler,        // SVCall                        │
│      (uint32_t)DebugMon_Handler,   // Debug Monitor                 │
│      0,                            // Reserved                      │
│      (uint32_t)PendSV_Handler,     // PendSV                        │
│      (uint32_t)SysTick_Handler,    // SysTick                       │
│      // External interrupts                                          │
│      (uint32_t)WWDG_IRQHandler,    // IRQ 0                         │
│      (uint32_t)PVD_IRQHandler,     // IRQ 1                         │
│      // ... more IRQ handlers                                        │
│  };                                                                  │
│                                                                      │
│  DEFAULT HANDLER PATTERN:                                            │
│                                                                      │
│  // Weak definitions for unused handlers                            │
│  __attribute__((weak))                                               │
│  void NMI_Handler(void) {                                           │
│      while(1);  // Infinite loop on unhandled interrupt             │
│  }                                                                   │
│                                                                      │
│  // Can be overridden in application code                           │
│  void NMI_Handler(void) {                                           │
│      // Custom NMI handling                                         │
│  }                                                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.5.3 Startup Code Details

### Detailed Startup Implementation

```
┌─────────────────────────────────────────────────────────────────────┐
│                    STARTUP CODE                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  RESET HANDLER IMPLEMENTATION:                                       │
│                                                                      │
│  /* Symbols defined in linker script */                             │
│  extern uint32_t _sidata;  // .data load address (in Flash)         │
│  extern uint32_t _sdata;   // .data start (in RAM)                  │
│  extern uint32_t _edata;   // .data end (in RAM)                    │
│  extern uint32_t _sbss;    // .bss start                            │
│  extern uint32_t _ebss;    // .bss end                              │
│  extern uint32_t _estack;  // Stack top                             │
│                                                                      │
│  extern void main(void);                                            │
│  extern void SystemInit(void);                                      │
│  extern void __libc_init_array(void);                               │
│                                                                      │
│  void Reset_Handler(void) {                                         │
│      uint32_t *src, *dst;                                           │
│                                                                      │
│      /* 1. Copy .data from Flash to RAM */                          │
│      src = &_sidata;                                                │
│      dst = &_sdata;                                                 │
│      while (dst < &_edata) {                                        │
│          *dst++ = *src++;                                           │
│      }                                                               │
│                                                                      │
│      /* 2. Zero-fill .bss section */                                │
│      dst = &_sbss;                                                  │
│      while (dst < &_ebss) {                                         │
│          *dst++ = 0;                                                │
│      }                                                               │
│                                                                      │
│      /* 3. Initialize system (clocks, etc.) */                      │
│      SystemInit();                                                   │
│                                                                      │
│      /* 4. Call static constructors (C++) */                        │
│      __libc_init_array();                                           │
│                                                                      │
│      /* 5. Call main() */                                           │
│      main();                                                         │
│                                                                      │
│      /* 6. Should never reach here */                               │
│      while(1);                                                       │
│  }                                                                   │
│                                                                      │
│  MEMORY LAYOUT DURING STARTUP:                                       │
│                                                                      │
│  FLASH                           RAM                                │
│  ┌─────────────────┐            ┌─────────────────┐                │
│  │  Vector Table   │            │                 │                │
│  ├─────────────────┤            │  ↓ Stack grows  │                │
│  │                 │            │    downward     │                │
│  │   .text (Code)  │            │                 │                │
│  │                 │            ├─────────────────┤ _estack        │
│  ├─────────────────┤            │                 │                │
│  │   .rodata       │            │    (unused)     │                │
│  │  (Constants)    │            │                 │                │
│  ├─────────────────┤ _sidata    ├─────────────────┤ _ebss          │
│  │   .data init    │───────────►│    .bss         │ (zeroed)       │
│  │   values        │  copy      │                 │                │
│  └─────────────────┘            ├─────────────────┤ _sbss/_edata   │
│                                 │    .data        │ (copied)       │
│                                 │                 │                │
│                                 └─────────────────┘ _sdata         │
│                                                                      │
│  SYSTEM INITIALIZATION:                                              │
│                                                                      │
│  void SystemInit(void) {                                            │
│      // 1. Configure Flash latency (wait states)                    │
│      FLASH->ACR = FLASH_ACR_LATENCY_5WS;                            │
│                                                                      │
│      // 2. Enable HSE (external crystal)                            │
│      RCC->CR |= RCC_CR_HSEON;                                       │
│      while(!(RCC->CR & RCC_CR_HSERDY));                             │
│                                                                      │
│      // 3. Configure PLL                                            │
│      RCC->PLLCFGR = PLL_M | (PLL_N << 6) | PLL_P | PLL_Q;          │
│      RCC->CR |= RCC_CR_PLLON;                                       │
│      while(!(RCC->CR & RCC_CR_PLLRDY));                             │
│                                                                      │
│      // 4. Configure bus prescalers                                 │
│      RCC->CFGR = RCC_CFGR_HPRE_DIV1 |   // AHB = 168 MHz           │
│                  RCC_CFGR_PPRE1_DIV4 |  // APB1 = 42 MHz           │
│                  RCC_CFGR_PPRE2_DIV2;   // APB2 = 84 MHz           │
│                                                                      │
│      // 5. Switch to PLL as system clock                            │
│      RCC->CFGR |= RCC_CFGR_SW_PLL;                                  │
│      while((RCC->CFGR & RCC_CFGR_SWS) != RCC_CFGR_SWS_PLL);        │
│  }                                                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.5.4 Bootloader Design

### Bootloader Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    BOOTLOADER DESIGN                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  MEMORY LAYOUT WITH BOOTLOADER:                                      │
│                                                                      │
│  Flash Memory Map:                                                   │
│  ┌─────────────────────────────────┐  0x0800_0000                   │
│  │       BOOTLOADER                │                                │
│  │    (Vector Table @ 0x0000)      │                                │
│  │    • UART/USB interface         │                                │
│  │    • Flash programming          │  16-64 KB                      │
│  │    • CRC verification           │                                │
│  ├─────────────────────────────────┤  0x0800_8000                   │
│  │       APPLICATION               │                                │
│  │    (Vector Table @ 0x8000)      │                                │
│  │    • Main application code      │                                │
│  │    • Application data           │  Remaining Flash               │
│  │                                 │                                │
│  ├─────────────────────────────────┤  0x080F_0000                   │
│  │     CONFIGURATION DATA          │                                │
│  │    • App version                │                                │
│  │    • CRC checksum               │  Last sector                   │
│  │    • Boot flags                 │                                │
│  └─────────────────────────────────┘  0x0810_0000                   │
│                                                                      │
│  BOOTLOADER FLOW:                                                    │
│                                                                      │
│       Reset                                                          │
│         │                                                            │
│         ▼                                                            │
│  ┌─────────────────┐                                                │
│  │ Bootloader Init │                                                │
│  └────────┬────────┘                                                │
│           │                                                          │
│           ▼                                                          │
│  ┌─────────────────┐     Yes    ┌─────────────────┐                 │
│  │ Update Request? ├───────────►│ Receive & Flash │                 │
│  │ (Button/Flag)   │            │ New Firmware    │                 │
│  └────────┬────────┘            └────────┬────────┘                 │
│           │ No                           │                           │
│           ▼                              │                           │
│  ┌─────────────────┐                     │                           │
│  │ Valid App?      │◄────────────────────┘                          │
│  │ (Check CRC)     │                                                │
│  └────────┬────────┘                                                │
│           │ Yes                                                      │
│           ▼                                                          │
│  ┌─────────────────┐                                                │
│  │ Jump to App     │                                                │
│  │ @ 0x0800_8000   │                                                │
│  └─────────────────┘                                                │
│                                                                      │
│  BOOTLOADER CODE:                                                    │
│                                                                      │
│  #define APP_ADDRESS  0x08008000                                    │
│                                                                      │
│  typedef void (*pFunction)(void);                                   │
│                                                                      │
│  void jump_to_application(void) {                                   │
│      uint32_t app_stack = *((uint32_t *)APP_ADDRESS);               │
│      uint32_t app_reset = *((uint32_t *)(APP_ADDRESS + 4));         │
│                                                                      │
│      // Check if valid stack pointer                                │
│      if ((app_stack & 0x2FFE0000) != 0x20000000) {                  │
│          return;  // Invalid application                            │
│      }                                                               │
│                                                                      │
│      // Disable interrupts                                          │
│      __disable_irq();                                               │
│                                                                      │
│      // Set vector table offset                                     │
│      SCB->VTOR = APP_ADDRESS;                                       │
│                                                                      │
│      // Set stack pointer                                           │
│      __set_MSP(app_stack);                                          │
│                                                                      │
│      // Jump to application reset handler                           │
│      pFunction app_entry = (pFunction)app_reset;                    │
│      app_entry();                                                    │
│  }                                                                   │
│                                                                      │
│  APPLICATION LINKER SCRIPT MODIFICATION:                             │
│                                                                      │
│  MEMORY                                                              │
│  {                                                                   │
│      FLASH (rx)  : ORIGIN = 0x08008000, LENGTH = 480K               │
│      SRAM (rwx)  : ORIGIN = 0x20000000, LENGTH = 128K               │
│  }                                                                   │
│                                                                      │
│  /* Vector table must be at app start */                            │
│  /* Application must set VTOR = 0x08008000 in SystemInit */         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.5.5 Firmware Update Mechanisms

### OTA and In-System Programming

```
┌─────────────────────────────────────────────────────────────────────┐
│                    FIRMWARE UPDATE MECHANISMS                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  UPDATE METHODS:                                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Method     │ Interface  │ Advantages        │ Use Case      │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ JTAG/SWD   │ Debug port │ Full access       │ Development   │    │
│  │ UART       │ Serial     │ Simple, low cost  │ Production    │    │
│  │ USB DFU    │ USB        │ Fast, standard    │ Consumer      │    │
│  │ SD Card    │ SPI/SDIO   │ No PC needed      │ Field update  │    │
│  │ OTA        │ WiFi/BLE   │ Remote update     │ IoT devices   │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  DUAL-BANK (A/B) UPDATE:                                            │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │                    FLASH MEMORY                            │     │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │     │
│  │  │  Bootloader  │  │   Bank A     │  │   Bank B     │    │     │
│  │  │              │  │  (Active)    │  │  (Inactive)  │    │     │
│  │  │   16 KB      │  │   240 KB     │  │   240 KB     │    │     │
│  │  └──────────────┘  └──────────────┘  └──────────────┘    │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  Update Process:                                                     │
│  1. Download new firmware to inactive bank (B)                      │
│  2. Verify CRC/signature                                            │
│  3. Set flag to boot from Bank B                                    │
│  4. Reboot                                                           │
│  5. Bootloader reads flag, jumps to Bank B                          │
│                                                                      │
│  Rollback: If new firmware fails, boot old from Bank A              │
│                                                                      │
│  FIRMWARE IMAGE FORMAT:                                              │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  Header (64 bytes)                                         │     │
│  │  ┌─────────────────────────────────────────────────────┐  │     │
│  │  │ Magic Number    │ 0x464D5257 ("FMRW")               │  │     │
│  │  │ Version         │ 1.2.3                              │  │     │
│  │  │ Image Size      │ 65536 bytes                        │  │     │
│  │  │ CRC32           │ 0xABCD1234                         │  │     │
│  │  │ Entry Point     │ 0x08008000                         │  │     │
│  │  │ Build Date      │ 2024-01-15                         │  │     │
│  │  │ Signature       │ (256 bytes for secure boot)        │  │     │
│  │  └─────────────────────────────────────────────────────┘  │     │
│  ├────────────────────────────────────────────────────────────┤     │
│  │  Firmware Binary Data                                      │     │
│  │  (Application code)                                        │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  CRC VERIFICATION:                                                   │
│                                                                      │
│  uint32_t crc32_calculate(uint8_t *data, uint32_t len) {            │
│      uint32_t crc = 0xFFFFFFFF;                                     │
│      for (uint32_t i = 0; i < len; i++) {                           │
│          crc ^= data[i];                                            │
│          for (int j = 0; j < 8; j++) {                              │
│              crc = (crc >> 1) ^ (0xEDB88320 & -(crc & 1));          │
│          }                                                           │
│      }                                                               │
│      return ~crc;                                                    │
│  }                                                                   │
│                                                                      │
│  int verify_firmware(uint32_t addr, uint32_t size, uint32_t expected_crc) {│
│      uint32_t calc_crc = crc32_calculate((uint8_t *)addr, size);   │
│      return (calc_crc == expected_crc) ? 0 : -1;                    │
│  }                                                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Stage | Description | Key Operations |
|-------|-------------|----------------|
| **Hardware Reset** | Power-on initialization | Voltage stabilization, reset release |
| **Vector Fetch** | Load SP and PC | Read from 0x00000000 and 0x00000004 |
| **Startup Code** | Prepare C environment | Copy .data, zero .bss, call SystemInit |
| **System Init** | Configure hardware | Clock setup, peripheral enable |
| **Bootloader** | Firmware management | Update, verify, jump to app |
| **Application** | Main program | Initialize, run main loop |

---

## Quick Revision Questions

1. **What are the first two values read from the vector table on reset?**

2. **Explain why .data must be copied from Flash to RAM during startup.**

3. **What is the purpose of zeroing the .bss section?**

4. **How does a bootloader safely jump to the application code?**

5. **What is the advantage of a dual-bank (A/B) firmware update scheme?**

6. **Why is CRC verification important before executing new firmware?**

---

## Navigation

| Previous | Up | Next |
|----------|------|------|
| [Device Drivers](04-device-drivers.md) | [Unit 3 Index](README.md) | [Unit 4: RTOS](../04-RTOS/README.md) |

---
