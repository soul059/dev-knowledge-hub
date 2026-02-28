# Chapter 7.4: Testing and Validation

## Introduction

Testing is critical for embedded systems because failures can be costly, dangerous, or impossible to fix after deployment. This chapter covers testing strategies from unit testing individual functions to system-level validation, including specialized techniques for hardware-software integration.

---

## Testing Pyramid

```
┌─────────────────────────────────────────────────────────────────────┐
│              EMBEDDED TESTING PYRAMID                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                          /\                                          │
│                         /  \                                         │
│                        /    \       SYSTEM/ACCEPTANCE TESTS         │
│                       / E2E  \      • Complete system                │
│                      /________\     • Real hardware                  │
│                     /          \    • Field conditions              │
│                    /            \                                    │
│                   /  INTEGRATION \   INTEGRATION TESTS              │
│                  /     TESTS      \  • Module interactions          │
│                 /__________________\ • HW-SW interface              │
│                /                    \                                │
│               /                      \   UNIT TESTS                 │
│              /      UNIT TESTS        \  • Individual functions     │
│             /__________________________\ • Fast, isolated           │
│                                          • Mock dependencies         │
│                                                                      │
│  QUANTITY:    Few ◀─────────────────────────────────▶ Many         │
│  SPEED:       Slow ◀────────────────────────────────▶ Fast         │
│  ISOLATION:   Low ◀─────────────────────────────────▶ High         │
│  COST:        High ◀────────────────────────────────▶ Low          │
│                                                                      │
│  EMBEDDED-SPECIFIC CONSIDERATIONS:                                   │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Level        │ Can run on    │ Hardware needed              │  │
│  ├──────────────┼───────────────┼──────────────────────────────┤  │
│  │ Unit         │ Host PC       │ None (mocked)                │  │
│  │ Integration  │ Simulator/HW  │ Development board            │  │
│  │ System       │ Target only   │ Full prototype               │  │
│  │ Acceptance   │ Target only   │ Production hardware          │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Unit Testing

### Testing Framework Example

```c
/* ================================================================
 * Unity Test Framework Example
 * Unity is a lightweight C test framework for embedded
 * ================================================================ */

/* test_temperature.c */
#include "unity.h"
#include "temperature.h"
#include "mock_adc.h"  /* Generated mock for ADC driver */

void setUp(void) {
    /* Called before each test */
    temperature_init();
}

void tearDown(void) {
    /* Called after each test */
}

/* ================================================================
 * Test: Temperature calculation from ADC value
 * ================================================================ */
void test_temperature_from_adc_at_25C(void) {
    /* Given: ADC reading corresponds to 25°C */
    uint16_t adc_value = 2048;  /* Mid-scale for 25°C */
    
    /* When: Convert ADC to temperature */
    float temp = temperature_from_adc(adc_value);
    
    /* Then: Temperature should be 25°C ±0.5 */
    TEST_ASSERT_FLOAT_WITHIN(0.5f, 25.0f, temp);
}

void test_temperature_from_adc_at_0C(void) {
    uint16_t adc_value = 1024;  /* Corresponds to 0°C */
    float temp = temperature_from_adc(adc_value);
    TEST_ASSERT_FLOAT_WITHIN(0.5f, 0.0f, temp);
}

void test_temperature_from_adc_at_100C(void) {
    uint16_t adc_value = 3072;  /* Corresponds to 100°C */
    float temp = temperature_from_adc(adc_value);
    TEST_ASSERT_FLOAT_WITHIN(0.5f, 100.0f, temp);
}

/* ================================================================
 * Test: Reading temperature with mocked ADC
 * ================================================================ */
void test_read_temperature_success(void) {
    /* Given: ADC will return a specific value */
    adc_read_ExpectAndReturn(ADC_CHANNEL_TEMP, 2048);
    
    /* When: Read temperature */
    float temp;
    int result = temperature_read(&temp);
    
    /* Then: Should succeed with correct value */
    TEST_ASSERT_EQUAL(0, result);
    TEST_ASSERT_FLOAT_WITHIN(0.5f, 25.0f, temp);
}

void test_read_temperature_adc_error(void) {
    /* Given: ADC returns error */
    adc_read_ExpectAndReturn(ADC_CHANNEL_TEMP, -1);
    
    /* When: Read temperature */
    float temp;
    int result = temperature_read(&temp);
    
    /* Then: Should return error code */
    TEST_ASSERT_EQUAL(-1, result);
}

/* ================================================================
 * Test: Boundary conditions
 * ================================================================ */
void test_temperature_min_valid_adc(void) {
    float temp = temperature_from_adc(0);
    TEST_ASSERT_TRUE(temp >= TEMP_MIN);
}

void test_temperature_max_valid_adc(void) {
    float temp = temperature_from_adc(4095);
    TEST_ASSERT_TRUE(temp <= TEMP_MAX);
}

/* ================================================================
 * Main test runner
 * ================================================================ */
int main(void) {
    UNITY_BEGIN();
    
    RUN_TEST(test_temperature_from_adc_at_25C);
    RUN_TEST(test_temperature_from_adc_at_0C);
    RUN_TEST(test_temperature_from_adc_at_100C);
    RUN_TEST(test_read_temperature_success);
    RUN_TEST(test_read_temperature_adc_error);
    RUN_TEST(test_temperature_min_valid_adc);
    RUN_TEST(test_temperature_max_valid_adc);
    
    return UNITY_END();
}
```

### Mocking Hardware Dependencies

```
┌─────────────────────────────────────────────────────────────────────┐
│              MOCKING FOR EMBEDDED                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PRODUCTION CODE:                      TEST CODE:                    │
│                                                                      │
│  ┌──────────────┐                     ┌──────────────┐             │
│  │ Application  │                     │    Test      │             │
│  │   Logic      │                     │   Runner     │             │
│  └──────┬───────┘                     └──────┬───────┘             │
│         │                                    │                      │
│         ▼                                    ▼                      │
│  ┌──────────────┐                     ┌──────────────┐             │
│  │     HAL      │                     │     HAL      │             │
│  │  Interface   │ ◀────SAME API────▶  │  Interface   │             │
│  └──────┬───────┘                     └──────┬───────┘             │
│         │                                    │                      │
│         ▼                                    ▼                      │
│  ┌──────────────┐                     ┌──────────────┐             │
│  │   Actual     │                     │    Mock      │             │
│  │  Hardware    │                     │ Implementation│             │
│  │  Driver      │                     │ (returns fake │             │
│  └──────────────┘                     │   values)    │             │
│                                        └──────────────┘             │
│                                                                      │
│  CMock EXAMPLE (generated from hal_adc.h):                          │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ /* Original function */                                      │  │
│  │ int adc_read(uint8_t channel);                               │  │
│  │                                                               │  │
│  │ /* Mock functions (auto-generated) */                        │  │
│  │ void adc_read_ExpectAndReturn(uint8_t channel, int retval);  │  │
│  │ void adc_read_ExpectWithArrayAndReturn(uint8_t *channel,     │  │
│  │                                         int retval);          │  │
│  │ void adc_read_StubWithCallback(int (*callback)(uint8_t));    │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  BENEFITS:                                                           │
│  • Test logic without hardware                                      │
│  • Faster test execution (run on PC)                                │
│  • Reproducible edge cases                                          │
│  • CI/CD integration                                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Integration Testing

### On-Target Testing

```
┌─────────────────────────────────────────────────────────────────────┐
│              INTEGRATION TEST SETUP                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐│
│  │                         HOST PC                                 ││
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐        ││
│  │  │ Test Script │───▶│  Serial     │───▶│  Test       │        ││
│  │  │ (Python)    │    │  Interface  │    │  Log/Report │        ││
│  │  └─────────────┘    └──────┬──────┘    └─────────────┘        ││
│  └────────────────────────────┼───────────────────────────────────┘│
│                               │ USB/UART                            │
│  ┌────────────────────────────▼───────────────────────────────────┐│
│  │                     TARGET BOARD                                ││
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐        ││
│  │  │ Application │───▶│ Test        │───▶│ UART        │        ││
│  │  │   Code      │    │ Harness     │    │ Output      │        ││
│  │  └─────────────┘    └─────────────┘    └─────────────┘        ││
│  │                                                                 ││
│  │  ┌─────────────────────────────────────────────────────────┐   ││
│  │  │                  REAL HARDWARE                          │   ││
│  │  │   ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐           │   ││
│  │  │   │Sensor│   │ ADC  │   │ GPIO │   │ PWM  │           │   ││
│  │  │   └──────┘   └──────┘   └──────┘   └──────┘           │   ││
│  │  └─────────────────────────────────────────────────────────┘   ││
│  └────────────────────────────────────────────────────────────────┘│
│                                                                      │
│  ON-TARGET TEST EXAMPLE:                                            │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ void test_uart_loopback(void) {                              │  │
│  │     uint8_t tx_data[] = {0xAA, 0x55, 0x12, 0x34};           │  │
│  │     uint8_t rx_data[4];                                      │  │
│  │                                                               │  │
│  │     /* Connect TX to RX externally for loopback */          │  │
│  │     uart_send(UART1, tx_data, 4);                           │  │
│  │     uart_receive(UART1, rx_data, 4, 100); /* 100ms timeout */│  │
│  │                                                               │  │
│  │     TEST_ASSERT_EQUAL_UINT8_ARRAY(tx_data, rx_data, 4);     │  │
│  │ }                                                             │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Module Interaction Testing

```c
/* ================================================================
 * Integration Test: Sensor + Processing + Output
 * Tests the complete data flow through multiple modules
 * ================================================================ */

#include "unity.h"
#include "sensor_manager.h"
#include "data_processor.h"
#include "display_driver.h"

/* Integration test using real hardware */
void test_sensor_to_display_pipeline(void) {
    /* Initialize all modules */
    sensor_manager_init();
    data_processor_init();
    display_init();
    
    /* Trigger sensor reading */
    sensor_data_t raw_data;
    int result = sensor_read(&raw_data);
    TEST_ASSERT_EQUAL(0, result);
    
    /* Process the data */
    processed_data_t processed;
    result = data_process(&raw_data, &processed);
    TEST_ASSERT_EQUAL(0, result);
    
    /* Verify processed data is in valid range */
    TEST_ASSERT_TRUE(processed.temperature >= -40.0f);
    TEST_ASSERT_TRUE(processed.temperature <= 125.0f);
    
    /* Update display */
    result = display_show_temperature(processed.temperature);
    TEST_ASSERT_EQUAL(0, result);
    
    /* Verify display state */
    TEST_ASSERT_TRUE(display_is_updated());
}

/* Test error propagation through the pipeline */
void test_sensor_error_handling(void) {
    /* Simulate sensor disconnect (implementation-specific) */
    sensor_simulate_disconnect();
    
    sensor_data_t raw_data;
    int result = sensor_read(&raw_data);
    
    /* Should report error */
    TEST_ASSERT_EQUAL(SENSOR_ERROR_DISCONNECTED, result);
    
    /* Display should show error message */
    TEST_ASSERT_TRUE(display_shows_error());
    
    /* Restore sensor */
    sensor_simulate_reconnect();
}

/* Test timing requirements */
void test_control_loop_timing(void) {
    uint32_t start_time = timer_get_ms();
    
    /* Run one control loop iteration */
    control_loop_iterate();
    
    uint32_t elapsed = timer_get_ms() - start_time;
    
    /* Must complete within 10ms */
    TEST_ASSERT_TRUE(elapsed < 10);
}
```

---

## Hardware-in-the-Loop (HIL) Testing

```
┌─────────────────────────────────────────────────────────────────────┐
│              HARDWARE-IN-THE-LOOP TEST SYSTEM                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  HIL simulates the real-world environment for the embedded system   │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐│
│  │                      HOST PC                                    ││
│  │  ┌─────────────────────────────────────────────────────────┐   ││
│  │  │             SIMULATION SOFTWARE                          │   ││
│  │  │                                                          │   ││
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │   ││
│  │  │  │ Plant Model │  │ Test Script │  │   Logger    │     │   ││
│  │  │  │ (Simulink/  │  │  (Python)   │  │             │     │   ││
│  │  │  │  MATLAB)    │  │             │  │             │     │   ││
│  │  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘     │   ││
│  │  └─────────┼────────────────┼────────────────┼─────────────┘   ││
│  └────────────┼────────────────┼────────────────┼─────────────────┘│
│               │                │                │                   │
│  ┌────────────▼────────────────▼────────────────▼─────────────────┐│
│  │              DAQ / IO INTERFACE HARDWARE                        ││
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐                ││
│  │  │  Analog    │  │  Digital   │  │   PWM      │                ││
│  │  │  Out (DAC) │  │    I/O     │  │  Capture   │                ││
│  │  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘                ││
│  └────────┼───────────────┼───────────────┼───────────────────────┘│
│           │               │               │                         │
│  ┌────────▼───────────────▼───────────────▼───────────────────────┐│
│  │              DEVICE UNDER TEST (DUT)                            ││
│  │  ┌────────────────────────────────────────────────────────┐    ││
│  │  │              EMBEDDED CONTROLLER                        │    ││
│  │  │                                                         │    ││
│  │  │  Sensor    ──▶  Control    ──▶  Actuator               │    ││
│  │  │  Inputs         Algorithm       Outputs                │    ││
│  │  │                                                         │    ││
│  │  └────────────────────────────────────────────────────────┘    ││
│  └────────────────────────────────────────────────────────────────┘│
│                                                                      │
│  EXAMPLE: Motor Controller HIL                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                                                               │  │
│  │   Simulated          DUT                    Simulated        │  │
│  │   Environment        Controller             Actuator         │  │
│  │                                                               │  │
│  │   ┌─────────┐       ┌─────────┐            ┌─────────┐      │  │
│  │   │ Motor   │─ADC──▶│ MCU     │───PWM────▶ │ DAQ     │      │  │
│  │   │ Model   │ Speed │ Control │   Command  │ Capture │      │  │
│  │   │         │◀─DAC──│ Loop    │            │         │      │  │
│  │   │ V=IR+Ldi│       │         │            │         │      │  │
│  │   │ J*dw=T  │       │         │            │         │      │  │
│  │   └─────────┘       └─────────┘            └─────────┘      │  │
│  │                                                               │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  HIL TEST SCENARIOS:                                                │
│  • Normal operation at various setpoints                            │
│  • Step response and transient behavior                             │
│  • Fault injection (sensor failure, actuator stuck)                 │
│  • Environmental extremes (temperature, load)                       │
│  • Edge cases impossible to create with real hardware               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Debugging Techniques

### Common Debugging Methods

```
┌─────────────────────────────────────────────────────────────────────┐
│              DEBUGGING TECHNIQUES                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. PRINTF DEBUGGING (Simple but effective)                        │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ /* Redirect printf to UART for debug output */               │  │
│  │ int _write(int fd, char *buf, int len) {                     │  │
│  │     HAL_UART_Transmit(&huart2, (uint8_t*)buf, len, 1000);   │  │
│  │     return len;                                               │  │
│  │ }                                                             │  │
│  │                                                               │  │
│  │ /* Usage */                                                   │  │
│  │ printf("[%lu] Temp: %.2f, State: %d\n",                      │  │
│  │        HAL_GetTick(), temperature, state);                   │  │
│  │                                                               │  │
│  │ /* Conditional debug */                                       │  │
│  │ #ifdef DEBUG                                                  │  │
│  │     #define DBG_PRINT(...) printf(__VA_ARGS__)               │  │
│  │ #else                                                         │  │
│  │     #define DBG_PRINT(...)                                    │  │
│  │ #endif                                                        │  │
│  └──────────────────────────────────────────────────────────────┘  │
│  ✓ Simple           ✗ Affects timing    ✗ Uses resources           │
│                                                                      │
│  2. LED/GPIO DEBUGGING (Minimal overhead)                           │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ /* Toggle GPIO to mark code execution */                     │  │
│  │ #define DEBUG_PIN_HIGH()  GPIOB->BSRR = (1 << 5)            │  │
│  │ #define DEBUG_PIN_LOW()   GPIOB->BSRR = (1 << 21)           │  │
│  │ #define DEBUG_PIN_TOGGLE() GPIOB->ODR ^= (1 << 5)           │  │
│  │                                                               │  │
│  │ void ISR_Handler(void) {                                     │  │
│  │     DEBUG_PIN_HIGH();   /* Mark ISR entry */                 │  │
│  │     /* ISR code */                                            │  │
│  │     DEBUG_PIN_LOW();    /* Mark ISR exit */                  │  │
│  │ }                                                             │  │
│  │                                                               │  │
│  │ /* View with oscilloscope or logic analyzer */               │  │
│  └──────────────────────────────────────────────────────────────┘  │
│  ✓ Fast              ✓ Timing analysis    ✗ Limited info           │
│                                                                      │
│  3. SEGGER RTT (Real-Time Transfer)                                 │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ /* Uses debug probe memory interface - no UART needed */     │  │
│  │ #include "SEGGER_RTT.h"                                      │  │
│  │                                                               │  │
│  │ SEGGER_RTT_printf(0, "Value: %d\n", value);                 │  │
│  │                                                               │  │
│  │ /* Very fast, minimal CPU overhead */                        │  │
│  └──────────────────────────────────────────────────────────────┘  │
│  ✓ Very fast         ✓ Non-intrusive      ✗ Needs J-Link           │
│                                                                      │
│  4. HARDWARE DEBUGGER (Full featured)                               │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Features:                                                     │  │
│  │ • Breakpoints (hardware and software)                        │  │
│  │ • Single-step execution                                      │  │
│  │ • Variable inspection                                        │  │
│  │ • Memory view                                                 │  │
│  │ • Call stack                                                  │  │
│  │ • Peripheral register view                                   │  │
│  │ • Trace (instruction/data)                                   │  │
│  │                                                               │  │
│  │ WATCHPOINT EXAMPLE:                                          │  │
│  │ Break when variable 'buffer[0]' changes value                │  │
│  │ → Catches buffer corruption                                  │  │
│  └──────────────────────────────────────────────────────────────┘  │
│  ✓ Full visibility   ✗ Stops execution    ✗ May mask timing bugs   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Fault Analysis

```
┌─────────────────────────────────────────────────────────────────────┐
│              HARD FAULT DEBUGGING                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  When ARM Cortex-M crashes, it saves context to stack:              │
│                                                                      │
│  Stack Pointer (SP)                                                  │
│       │                                                              │
│       ▼                                                              │
│  ┌────────────┐                                                     │
│  │    R0      │  SP + 0x00                                          │
│  ├────────────┤                                                     │
│  │    R1      │  SP + 0x04                                          │
│  ├────────────┤                                                     │
│  │    R2      │  SP + 0x08                                          │
│  ├────────────┤                                                     │
│  │    R3      │  SP + 0x0C                                          │
│  ├────────────┤                                                     │
│  │    R12     │  SP + 0x10                                          │
│  ├────────────┤                                                     │
│  │    LR      │  SP + 0x14  (Return address)                        │
│  ├────────────┤                                                     │
│  │    PC      │  SP + 0x18  (Fault address!)                        │
│  ├────────────┤                                                     │
│  │    xPSR    │  SP + 0x1C                                          │
│  └────────────┘                                                     │
│                                                                      │
│  HARD FAULT HANDLER:                                                 │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ void HardFault_Handler(void) {                               │  │
│  │     __asm volatile (                                          │  │
│  │         "tst lr, #4            \n"  /* Check which SP */     │  │
│  │         "ite eq                 \n"                           │  │
│  │         "mrseq r0, msp          \n"  /* Main SP */           │  │
│  │         "mrsne r0, psp          \n"  /* Process SP */        │  │
│  │         "b hard_fault_handler_c \n"                           │  │
│  │     );                                                        │  │
│  │ }                                                             │  │
│  │                                                               │  │
│  │ void hard_fault_handler_c(uint32_t *sp) {                    │  │
│  │     uint32_t r0  = sp[0];                                    │  │
│  │     uint32_t r1  = sp[1];                                    │  │
│  │     uint32_t r2  = sp[2];                                    │  │
│  │     uint32_t r3  = sp[3];                                    │  │
│  │     uint32_t r12 = sp[4];                                    │  │
│  │     uint32_t lr  = sp[5];                                    │  │
│  │     uint32_t pc  = sp[6];  /* << FAULT LOCATION! */         │  │
│  │     uint32_t psr = sp[7];                                    │  │
│  │                                                               │  │
│  │     /* Log or display fault info */                          │  │
│  │     printf("HARD FAULT at PC=0x%08X, LR=0x%08X\n", pc, lr); │  │
│  │                                                               │  │
│  │     /* Check fault status registers */                       │  │
│  │     uint32_t cfsr = SCB->CFSR;                               │  │
│  │     uint32_t hfsr = SCB->HFSR;                               │  │
│  │     uint32_t bfar = SCB->BFAR;                               │  │
│  │                                                               │  │
│  │     while(1);  /* Halt for debugger */                       │  │
│  │ }                                                             │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  COMMON CAUSES:                                                      │
│  • NULL pointer dereference (PC near 0x00000000)                    │
│  • Stack overflow (SP outside RAM)                                  │
│  • Invalid instruction (corrupted code)                             │
│  • Unaligned access (for word operations)                           │
│  • Memory protection violation                                      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Test Automation

### Continuous Integration

```
┌─────────────────────────────────────────────────────────────────────┐
│              CI/CD FOR EMBEDDED                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  GITHUB ACTIONS EXAMPLE:                                            │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ # .github/workflows/ci.yml                                   │  │
│  │                                                               │  │
│  │ name: Embedded CI                                             │  │
│  │                                                               │  │
│  │ on: [push, pull_request]                                     │  │
│  │                                                               │  │
│  │ jobs:                                                         │  │
│  │   build:                                                      │  │
│  │     runs-on: ubuntu-latest                                   │  │
│  │     steps:                                                    │  │
│  │       - uses: actions/checkout@v3                            │  │
│  │                                                               │  │
│  │       - name: Install ARM toolchain                          │  │
│  │         run: |                                                │  │
│  │           sudo apt-get update                                 │  │
│  │           sudo apt-get install gcc-arm-none-eabi             │  │
│  │                                                               │  │
│  │       - name: Build firmware                                  │  │
│  │         run: |                                                │  │
│  │           mkdir build && cd build                            │  │
│  │           cmake .. && make                                   │  │
│  │                                                               │  │
│  │       - name: Run unit tests                                 │  │
│  │         run: |                                                │  │
│  │           cd tests && make test                              │  │
│  │                                                               │  │
│  │       - name: Static analysis                                │  │
│  │         run: |                                                │  │
│  │           cppcheck --error-exitcode=1 src/                   │  │
│  │                                                               │  │
│  │       - name: Upload artifacts                               │  │
│  │         uses: actions/upload-artifact@v3                     │  │
│  │         with:                                                 │  │
│  │           name: firmware                                      │  │
│  │           path: build/firmware.bin                           │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  CI PIPELINE STAGES:                                                 │
│                                                                      │
│  ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐  │
│  │ Build  │──▶│ Unit   │──▶│ Static │──▶│ Size   │──▶│ Deploy │  │
│  │        │   │ Tests  │   │Analysis│   │ Check  │   │(staging)│  │
│  └────────┘   └────────┘   └────────┘   └────────┘   └────────┘  │
│                                                                      │
│  STATIC ANALYSIS TOOLS:                                              │
│  • cppcheck - General C/C++ issues                                  │
│  • PC-lint - Commercial, comprehensive                               │
│  • Coverity - Advanced, cloud-based                                 │
│  • MISRA checkers - Safety-critical standards                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Validation and Certification

```
┌─────────────────────────────────────────────────────────────────────┐
│              VALIDATION PROCESS                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  VERIFICATION vs VALIDATION:                                        │
│                                                                      │
│  VERIFICATION: "Are we building it right?"                          │
│  ├─ Code reviews                                                    │
│  ├─ Unit testing                                                    │
│  ├─ Static analysis                                                 │
│  └─ Design reviews                                                  │
│                                                                      │
│  VALIDATION: "Are we building the right thing?"                     │
│  ├─ System testing                                                  │
│  ├─ User acceptance testing                                         │
│  ├─ Field trials                                                    │
│  └─ Requirements traceability                                       │
│                                                                      │
│  ════════════════════════════════════════════════════════════════   │
│                                                                      │
│  ENVIRONMENTAL TESTING:                                              │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Test              │ Standard         │ Purpose              │  │
│  ├───────────────────┼──────────────────┼──────────────────────┤  │
│  │ Temperature cycle │ IEC 60068-2-14   │ Thermal stress       │  │
│  │ Humidity          │ IEC 60068-2-78   │ Moisture resistance  │  │
│  │ Vibration         │ IEC 60068-2-6    │ Mechanical stress    │  │
│  │ Shock             │ IEC 60068-2-27   │ Drop/impact          │  │
│  │ ESD               │ IEC 61000-4-2    │ Electrostatic        │  │
│  │ EMC Immunity      │ IEC 61000-4-3    │ RF interference      │  │
│  │ EMC Emissions     │ CISPR 32         │ Radiated/conducted   │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  SAFETY STANDARDS BY DOMAIN:                                        │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Domain            │ Standard         │ Levels              │  │
│  ├───────────────────┼──────────────────┼──────────────────────┤  │
│  │ Automotive        │ ISO 26262        │ ASIL A-D            │  │
│  │ Medical           │ IEC 62304        │ Class A-C            │  │
│  │ Industrial        │ IEC 61508        │ SIL 1-4              │  │
│  │ Aviation          │ DO-178C          │ DAL A-E              │  │
│  │ Railway           │ EN 50128         │ SIL 1-4              │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  DOCUMENTATION REQUIREMENTS:                                         │
│  • Test plans and procedures                                        │
│  • Test reports with evidence                                       │
│  • Requirements traceability matrix                                 │
│  • Design documentation                                             │
│  • Risk analysis (FMEA)                                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **Testing Pyramid** | Unit (many) → Integration → System (few) |
| **Unit Testing** | Test individual functions with mocked dependencies |
| **Mocking** | Fake hardware interfaces for host-based testing |
| **Integration Testing** | Test module interactions on target hardware |
| **HIL Testing** | Simulate real-world environment with hardware |
| **Debugging** | Printf, GPIO, RTT, hardware debugger methods |
| **Hard Fault Analysis** | Examine stack and fault registers on crash |
| **CI/CD** | Automated build, test, and analysis pipeline |
| **Validation** | Environmental, EMC, and safety certification |

---

## Quick Revision Questions

1. **Explain the embedded testing pyramid. Why are unit tests at the base?**

2. **What is mocking and why is it important for unit testing embedded software? Give an example of mocking an ADC driver.**

3. **What is Hardware-in-the-Loop (HIL) testing? When would you use it instead of testing with real hardware?**

4. **Compare printf debugging, GPIO debugging, and hardware debugger breakpoints. What are the trade-offs of each?**

5. **How would you diagnose a Hard Fault on ARM Cortex-M? What information can you extract from the fault handler?**

6. **What is the difference between verification and validation? Give examples of each for an embedded medical device.**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Chapter 7.3: Prototyping and Development](03-prototyping-development.md) | [Unit 7: System Design](README.md) | [Chapter 7.5: Power Management](05-power-management.md) |

---
