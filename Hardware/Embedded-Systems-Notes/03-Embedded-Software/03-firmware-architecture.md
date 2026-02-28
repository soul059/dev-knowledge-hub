# Chapter 3.3: Firmware Architecture

## Chapter Overview

Firmware architecture defines how embedded software is structured and organized. This chapter covers common architectural patterns including super-loop, event-driven, layered architecture, and state machines that are fundamental to embedded system design.

---

## 3.3.1 Super-Loop Architecture

### Basic Polling-Based Design

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SUPER-LOOP ARCHITECTURE                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  BASIC STRUCTURE:                                                    │
│                                                                      │
│  int main(void) {                                                    │
│      // Initialization (runs once)                                   │
│      system_init();                                                  │
│      uart_init();                                                    │
│      gpio_init();                                                    │
│      timer_init();                                                   │
│                                                                      │
│      // Super-loop (runs forever)                                    │
│      while(1) {                                                      │
│          task1_handler();   // Poll sensor                          │
│          task2_handler();   // Process data                         │
│          task3_handler();   // Update display                       │
│          task4_handler();   // Check buttons                        │
│      }                                                               │
│  }                                                                   │
│                                                                      │
│  EXECUTION TIMELINE:                                                 │
│                                                                      │
│  Time ──────────────────────────────────────────────────────────►   │
│                                                                      │
│  │Init│ T1 │ T2 │ T3 │ T4 │ T1 │ T2 │ T3 │ T4 │ T1 │ ...         │
│  │    │    │    │    │    │    │    │    │    │    │             │
│  └────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴─────────────   │
│                                                                      │
│       │◄── One loop iteration ──►│                                  │
│              (cycle time)                                            │
│                                                                      │
│  ADVANTAGES:                     DISADVANTAGES:                      │
│  ┌────────────────────────┐     ┌────────────────────────┐          │
│  │ ✓ Simple to understand │     │ ✗ Poor responsiveness  │          │
│  │ ✓ No context switching │     │ ✗ Timing unpredictable │          │
│  │ ✓ Low memory overhead  │     │ ✗ CPU always running   │          │
│  │ ✓ Deterministic order  │     │ ✗ Hard to add features │          │
│  │ ✓ Easy to debug        │     │ ✗ No priority handling │          │
│  └────────────────────────┘     └────────────────────────┘          │
│                                                                      │
│  TIMING PROBLEM ILLUSTRATION:                                        │
│                                                                      │
│  Slow task blocks others:                                            │
│                                                                      │
│  │ T1 (1ms) │ T2 (100ms) │ T3 (1ms) │ T4 (1ms) │                   │
│  │          │            │          │          │                    │
│  └──────────┴────────────┴──────────┴──────────┘                    │
│                                                                      │
│  T3 and T4 delayed by 100ms due to slow T2!                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.3.2 Time-Triggered Architecture

### Periodic Task Scheduling

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TIME-TRIGGERED ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  COOPERATIVE SCHEDULER CONCEPT:                                      │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    Timer Interrupt (1ms tick)               │    │
│  │                            │                                 │    │
│  │                            ▼                                 │    │
│  │            ┌───────────────────────────────┐                │    │
│  │            │    Scheduler (sets flags)     │                │    │
│  │            └───────────────────────────────┘                │    │
│  │                            │                                 │    │
│  │            ┌───────────────┼───────────────┐                │    │
│  │            ▼               ▼               ▼                │    │
│  │      ┌─────────┐     ┌─────────┐     ┌─────────┐           │    │
│  │      │Task 1   │     │Task 2   │     │Task 3   │           │    │
│  │      │10ms flag│     │50ms flag│     │100ms flg│           │    │
│  │      └─────────┘     └─────────┘     └─────────┘           │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  IMPLEMENTATION:                                                     │
│                                                                      │
│  // Task configuration                                               │
│  typedef struct {                                                    │
│      void (*task)(void);     // Task function pointer               │
│      uint32_t period;        // Period in ticks                     │
│      uint32_t delay;         // Initial delay                       │
│      uint8_t  run;           // Run flag                            │
│  } Task_t;                                                           │
│                                                                      │
│  Task_t tasks[] = {                                                  │
│      {task_sensor,   10,  0, 0},  // Every 10ms                     │
│      {task_display,  100, 5, 0},  // Every 100ms, offset 5ms        │
│      {task_comms,    50,  0, 0},  // Every 50ms                     │
│  };                                                                  │
│                                                                      │
│  // Timer ISR (1ms tick)                                            │
│  void SysTick_Handler(void) {                                       │
│      for (int i = 0; i < NUM_TASKS; i++) {                          │
│          if (tasks[i].delay > 0) {                                  │
│              tasks[i].delay--;                                      │
│          } else {                                                    │
│              tasks[i].delay = tasks[i].period - 1;                  │
│              tasks[i].run = 1;  // Set run flag                     │
│          }                                                           │
│      }                                                               │
│  }                                                                   │
│                                                                      │
│  // Main loop (dispatcher)                                           │
│  while(1) {                                                          │
│      for (int i = 0; i < NUM_TASKS; i++) {                          │
│          if (tasks[i].run) {                                        │
│              tasks[i].run = 0;                                      │
│              tasks[i].task();  // Execute task                      │
│          }                                                           │
│      }                                                               │
│      __WFI();  // Sleep until next interrupt                        │
│  }                                                                   │
│                                                                      │
│  TIMING DIAGRAM:                                                     │
│                                                                      │
│  Tick  │ 0 │ 10 │ 20 │ 30 │ 40 │ 50 │ 60 │ 70 │ 80 │ 90 │100│     │
│  ──────┼───┼────┼────┼────┼────┼────┼────┼────┼────┼────┼───│     │
│  T1 10ms│ X │  X │  X │  X │  X │  X │  X │  X │  X │  X │ X │     │
│  T2 50ms│   │    │    │    │    │  X │    │    │    │    │ X │     │
│  T3 100ms│  │    │    │    │    │    │    │    │    │    │ X │     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.3.3 Event-Driven Architecture

### Interrupt and Flag-Based Design

```
┌─────────────────────────────────────────────────────────────────────┐
│                    EVENT-DRIVEN ARCHITECTURE                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  EVENT FLOW:                                                         │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                                                              │   │
│  │   External        ISR            Event         Main Loop     │   │
│  │   Events          Handler        Queue         Dispatcher    │   │
│  │                                                              │   │
│  │  ┌────────┐     ┌────────┐     ┌────────┐     ┌────────┐   │   │
│  │  │ Button │────►│ GPIO   │────►│        │     │        │   │   │
│  │  │ Press  │     │ ISR    │     │        │     │        │   │   │
│  │  └────────┘     └────────┘     │        │────►│ Event  │   │   │
│  │                                │ Event  │     │ Process│   │   │
│  │  ┌────────┐     ┌────────┐     │ Queue  │     │        │   │   │
│  │  │ Timer  │────►│ Timer  │────►│        │     │        │   │   │
│  │  │ Expire │     │ ISR    │     │        │     │        │   │   │
│  │  └────────┘     └────────┘     │        │     │        │   │   │
│  │                                │        │     │        │   │   │
│  │  ┌────────┐     ┌────────┐     │        │     │        │   │   │
│  │  │ UART   │────►│ UART   │────►│        │     │        │   │   │
│  │  │ RX     │     │ ISR    │     │        │     │        │   │   │
│  │  └────────┘     └────────┘     └────────┘     └────────┘   │   │
│  │                                                              │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  EVENT QUEUE IMPLEMENTATION:                                         │
│                                                                      │
│  typedef enum {                                                      │
│      EVT_NONE = 0,                                                   │
│      EVT_BUTTON_PRESSED,                                             │
│      EVT_BUTTON_RELEASED,                                            │
│      EVT_TIMER_EXPIRED,                                              │
│      EVT_UART_RX_COMPLETE,                                           │
│      EVT_ADC_COMPLETE,                                               │
│  } EventType_t;                                                      │
│                                                                      │
│  typedef struct {                                                    │
│      EventType_t type;                                               │
│      uint32_t    data;                                               │
│  } Event_t;                                                          │
│                                                                      │
│  // Circular event queue                                             │
│  #define QUEUE_SIZE 16                                               │
│  volatile Event_t event_queue[QUEUE_SIZE];                           │
│  volatile uint8_t queue_head = 0;                                    │
│  volatile uint8_t queue_tail = 0;                                    │
│                                                                      │
│  void event_post(EventType_t type, uint32_t data) {                 │
│      uint8_t next = (queue_head + 1) % QUEUE_SIZE;                  │
│      if (next != queue_tail) {  // Not full                         │
│          event_queue[queue_head].type = type;                       │
│          event_queue[queue_head].data = data;                       │
│          queue_head = next;                                          │
│      }                                                               │
│  }                                                                   │
│                                                                      │
│  // Main event loop                                                  │
│  while(1) {                                                          │
│      if (queue_head != queue_tail) {                                │
│          Event_t evt = event_queue[queue_tail];                     │
│          queue_tail = (queue_tail + 1) % QUEUE_SIZE;                │
│                                                                      │
│          switch(evt.type) {                                          │
│              case EVT_BUTTON_PRESSED:                                │
│                  handle_button(evt.data);                            │
│                  break;                                              │
│              case EVT_TIMER_EXPIRED:                                 │
│                  handle_timer();                                     │
│                  break;                                              │
│              // ... other events                                     │
│          }                                                           │
│      } else {                                                        │
│          __WFI();  // Sleep until event                             │
│      }                                                               │
│  }                                                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.3.4 State Machine Architecture

### Finite State Machine (FSM) Design

```
┌─────────────────────────────────────────────────────────────────────┐
│                    STATE MACHINE ARCHITECTURE                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  TRAFFIC LIGHT STATE DIAGRAM:                                        │
│                                                                      │
│             ┌─────────────────────────────────────────┐              │
│             │                                         │              │
│             ▼                                         │              │
│        ┌─────────┐   timer_30s   ┌─────────┐         │              │
│        │  GREEN  │──────────────►│ YELLOW  │         │              │
│        │         │               │         │         │              │
│        └─────────┘               └────┬────┘         │              │
│             ▲                         │              │              │
│             │                    timer_5s            │              │
│             │                         │              │              │
│             │                         ▼              │              │
│             │                    ┌─────────┐         │              │
│             │       timer_30s    │   RED   │         │              │
│             └────────────────────│         │─────────┘              │
│                                  └─────────┘   emergency_btn        │
│                                       │                              │
│                                       │ pedestrian_btn              │
│                                       ▼                              │
│                                  ┌─────────┐                        │
│                                  │ PED_WALK│                        │
│                                  └─────────┘                        │
│                                                                      │
│  TABLE-DRIVEN STATE MACHINE:                                         │
│                                                                      │
│  typedef enum {                                                      │
│      STATE_GREEN, STATE_YELLOW, STATE_RED, STATE_PED_WALK           │
│  } State_t;                                                          │
│                                                                      │
│  typedef enum {                                                      │
│      EVT_TIMER, EVT_PED_BUTTON, EVT_EMERGENCY                       │
│  } Event_t;                                                          │
│                                                                      │
│  typedef struct {                                                    │
│      State_t next_state;                                             │
│      void (*action)(void);                                           │
│  } Transition_t;                                                     │
│                                                                      │
│  // State transition table                                           │
│  const Transition_t fsm_table[4][3] = {                             │
│      /*           EVT_TIMER        EVT_PED_BTN      EVT_EMERGENCY */│
│      /* GREEN  */ {{YELLOW,green_to_yellow}, {GREEN,NULL}, ...},   │
│      /* YELLOW */ {{RED, yellow_to_red},     {YELLOW,NULL}, ...},  │
│      /* RED    */ {{GREEN, red_to_green},    {PED_WALK,ped}, ...}, │
│      /* PED    */ {{RED, ped_to_red},        {PED_WALK,NULL},...}, │
│  };                                                                  │
│                                                                      │
│  // State machine execution                                          │
│  void fsm_process(Event_t event) {                                  │
│      Transition_t trans = fsm_table[current_state][event];          │
│      if (trans.action != NULL) {                                    │
│          trans.action();  // Execute transition action              │
│      }                                                               │
│      current_state = trans.next_state;                              │
│  }                                                                   │
│                                                                      │
│  SWITCH-CASE STATE MACHINE:                                          │
│                                                                      │
│  void state_machine_run(void) {                                     │
│      switch(current_state) {                                         │
│          case STATE_GREEN:                                           │
│              green_led_on();                                         │
│              if (timer_expired(30000)) {                            │
│                  current_state = STATE_YELLOW;                      │
│              }                                                       │
│              break;                                                  │
│                                                                      │
│          case STATE_YELLOW:                                          │
│              yellow_led_on();                                        │
│              if (timer_expired(5000)) {                             │
│                  current_state = STATE_RED;                         │
│              }                                                       │
│              break;                                                  │
│                                                                      │
│          case STATE_RED:                                             │
│              red_led_on();                                           │
│              if (pedestrian_button()) {                             │
│                  current_state = STATE_PED_WALK;                    │
│              } else if (timer_expired(30000)) {                     │
│                  current_state = STATE_GREEN;                       │
│              }                                                       │
│              break;                                                  │
│          // ... other states                                         │
│      }                                                               │
│  }                                                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3.3.5 Layered Architecture

### Hardware Abstraction Layer (HAL)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    LAYERED ARCHITECTURE                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SOFTWARE LAYERS:                                                    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    APPLICATION LAYER                        │    │
│  │  • Business logic, algorithms, user interface              │    │
│  │  • Example: temperature_controller.c                       │    │
│  │                                                              │    │
│  │  void control_temperature(void) {                           │    │
│  │      int temp = sensor_read();  // Uses service layer       │    │
│  │      if (temp > setpoint) {                                 │    │
│  │          actuator_set(COOLING_ON);                          │    │
│  │      }                                                       │    │
│  │  }                                                           │    │
│  └──────────────────────────┬──────────────────────────────────┘    │
│                             │ API calls                             │
│                             ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    SERVICE/DRIVER LAYER                     │    │
│  │  • Device-independent abstractions                         │    │
│  │  • Example: sensor.c, actuator.c                           │    │
│  │                                                              │    │
│  │  int sensor_read(void) {                                    │    │
│  │      return hal_adc_read(TEMP_CHANNEL);  // Uses HAL        │    │
│  │  }                                                           │    │
│  └──────────────────────────┬──────────────────────────────────┘    │
│                             │ HAL calls                             │
│                             ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │               HARDWARE ABSTRACTION LAYER (HAL)              │    │
│  │  • MCU-specific, but portable API                          │    │
│  │  • Example: hal_adc.c, hal_gpio.c, hal_uart.c              │    │
│  │                                                              │    │
│  │  int hal_adc_read(uint8_t channel) {                        │    │
│  │      ADC1->SQR3 = channel;                                  │    │
│  │      ADC1->CR2 |= ADC_CR2_SWSTART;                          │    │
│  │      while(!(ADC1->SR & ADC_SR_EOC));                       │    │
│  │      return ADC1->DR;                                        │    │
│  │  }                                                           │    │
│  └──────────────────────────┬──────────────────────────────────┘    │
│                             │ Register access                       │
│                             ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    HARDWARE LAYER                           │    │
│  │  • Physical registers, peripherals                         │    │
│  │  • MCU header file: stm32f4xx.h                            │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  BENEFITS OF LAYERING:                                               │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │ Benefit           │ Description                            │     │
│  │────────────────────────────────────────────────────────────│     │
│  │ Portability       │ Change HAL only when switching MCU    │     │
│  │ Testability       │ Mock lower layers for unit testing    │     │
│  │ Maintainability   │ Changes isolated to specific layer    │     │
│  │ Reusability       │ Layers can be used in other projects  │     │
│  │ Team Development  │ Different teams work on different     │     │
│  │                   │ layers                                 │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  FILE STRUCTURE:                                                     │
│                                                                      │
│  project/                                                            │
│  ├── app/                   # Application layer                     │
│  │   ├── main.c                                                     │
│  │   └── temperature_ctrl.c                                         │
│  ├── drivers/               # Service/Driver layer                  │
│  │   ├── sensor.c                                                   │
│  │   └── actuator.c                                                 │
│  ├── hal/                   # Hardware Abstraction                  │
│  │   ├── hal_adc.c                                                  │
│  │   ├── hal_gpio.c                                                 │
│  │   └── hal_uart.c                                                 │
│  ├── cmsis/                 # MCU support                           │
│  │   ├── stm32f4xx.h                                                │
│  │   └── system_stm32f4xx.c                                         │
│  └── inc/                   # Header files                          │
│      └── *.h                                                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Architecture | Description | Best For |
|--------------|-------------|----------|
| **Super-loop** | Simple polling loop | Small, simple systems |
| **Time-triggered** | Periodic task execution | Regular sampling, control |
| **Event-driven** | Respond to events/interrupts | Reactive systems, low power |
| **State machine** | Discrete states and transitions | Protocol handlers, UI |
| **Layered/HAL** | Abstraction layers | Portable, maintainable code |

---

## Quick Revision Questions

1. **What are the main advantages and disadvantages of super-loop architecture?**

2. **How does a time-triggered scheduler differ from a super-loop?**

3. **Design a simple event queue with push and pop operations for an embedded system.**

4. **Draw a state diagram for a simple vending machine with states: IDLE, COIN_INSERTED, DISPENSING.**

5. **What are the benefits of using a Hardware Abstraction Layer (HAL)?**

6. **Explain why `__WFI()` (Wait For Interrupt) is used in event-driven systems.**

---

## Navigation

| Previous | Up | Next |
|----------|------|------|
| [Cross-Compilation](02-cross-compilation.md) | [Unit 3 Index](README.md) | [Device Drivers](04-device-drivers.md) |

---
