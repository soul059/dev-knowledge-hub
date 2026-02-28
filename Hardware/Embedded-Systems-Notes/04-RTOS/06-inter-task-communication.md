# Chapter 4.6: Inter-Task Communication

## Introduction

Inter-task communication (ITC) allows tasks to exchange data and coordinate activities. Proper ITC mechanisms ensure thread-safe data transfer without race conditions.

---

## Communication Mechanisms Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ITC MECHANISMS                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Mechanism      │ Data Size  │ Blocking │ Use Case           │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │ Queue          │ Variable   │ Yes      │ Producer-consumer  │    │
│  │ Mailbox        │ Fixed msg  │ Yes      │ Message passing    │    │
│  │ Event Flags    │ Bits       │ Yes      │ State signaling    │    │
│  │ Stream Buffer  │ Bytes      │ Yes      │ Continuous data    │    │
│  │ Message Buffer │ Messages   │ Yes      │ Variable-len msgs  │    │
│  │ Task Notify    │ 32 bits    │ Yes      │ Light signaling    │    │
│  │ Shared Memory  │ Any        │ No       │ Direct access      │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  SELECTION GUIDE:                                                    │
│                                                                      │
│  Need to pass structured data? ──────▶ Queue                        │
│  Need to signal event occurred? ─────▶ Binary Semaphore / Notify    │
│  Need to track multiple conditions? ─▶ Event Flags                  │
│  Need streaming byte data? ──────────▶ Stream Buffer                │
│  Need lightweight notification? ─────▶ Task Notification            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Queues

Queues are the primary mechanism for passing data between tasks.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    QUEUE STRUCTURE                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  FIFO Queue (First In, First Out):                                  │
│                                                                      │
│              Front                              Rear                 │
│              (Read)                             (Write)              │
│                │                                  │                  │
│                ▼                                  ▼                  │
│  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐                  │
│  │ D1  │ D2  │ D3  │ D4  │     │     │     │     │                  │
│  │ ◀───│─────│─────│─────│─────│─────│─────│───▶ │                  │
│  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘                  │
│    Out                                        In                     │
│                                                                      │
│  Queue Properties:                                                   │
│  • Fixed item size (set at creation)                                │
│  • Fixed length (number of items)                                   │
│  • Items copied into/out of queue (by value)                        │
│  • Thread-safe: multiple readers/writers OK                         │
│  • Blocking: tasks can wait for space/data                          │
│                                                                      │
│  OPERATIONS:                                                         │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ xQueueSend()      │ Add item to back (FIFO)                 │    │
│  │ xQueueSendToFront │ Add item to front (LIFO behavior)       │    │
│  │ xQueueReceive()   │ Remove item from front                  │    │
│  │ xQueuePeek()      │ Read front item without removing        │    │
│  │ uxQueueMessagesWaiting() │ Number of items in queue         │    │
│  │ uxQueueSpacesAvailable() │ Free slots in queue              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Queue Example: Sensor to Display

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PRODUCER-CONSUMER PATTERN                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────┐     ┌──────────────┐     ┌────────────┐             │
│  │  SENSOR    │     │              │     │  DISPLAY   │             │
│  │   TASK     │────▶│    QUEUE     │────▶│   TASK     │             │
│  │ (Producer) │     │              │     │ (Consumer) │             │
│  └────────────┘     └──────────────┘     └────────────┘             │
│                                                                      │
│  Multiple producers, single consumer pattern:                       │
│                                                                      │
│  ┌──────────┐                                                        │
│  │ Sensor 1 │────┐                                                   │
│  └──────────┘    │     ┌──────────────┐     ┌────────────┐          │
│  ┌──────────┐    │     │              │     │  DISPLAY   │          │
│  │ Sensor 2 │────┼────▶│    QUEUE     │────▶│   TASK     │          │
│  └──────────┘    │     │              │     │            │          │
│  ┌──────────┐    │     └──────────────┘     └────────────┘          │
│  │ Sensor 3 │────┘                                                   │
│  └──────────┘                                                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

```c
// Data structure passed through queue
typedef struct {
    uint8_t  sensor_id;
    int16_t  temperature;
    uint32_t timestamp;
} SensorData_t;

// Queue handle
QueueHandle_t sensor_queue;

void setup(void) {
    // Create queue: 10 items of SensorData_t
    sensor_queue = xQueueCreate(10, sizeof(SensorData_t));
    
    if (sensor_queue == NULL) {
        // Handle error - out of memory
    }
}

// Producer task (reads sensors)
void sensor_task(void *pv) {
    SensorData_t data;
    TickType_t last_wake = xTaskGetTickCount();
    
    while (1) {
        vTaskDelayUntil(&last_wake, pdMS_TO_TICKS(100));
        
        // Read sensor
        data.sensor_id = TEMP_SENSOR;
        data.temperature = read_temperature();
        data.timestamp = xTaskGetTickCount();
        
        // Send to queue (wait up to 10ms if full)
        if (xQueueSend(sensor_queue, &data, pdMS_TO_TICKS(10)) != pdTRUE) {
            // Queue full - data lost or handle overflow
            error_count++;
        }
    }
}

// Consumer task (updates display)
void display_task(void *pv) {
    SensorData_t data;
    
    while (1) {
        // Wait indefinitely for data
        if (xQueueReceive(sensor_queue, &data, portMAX_DELAY) == pdTRUE) {
            // Format and display
            lcd_printf("Sensor %d: %d.%d C", 
                      data.sensor_id,
                      data.temperature / 10,
                      data.temperature % 10);
        }
    }
}
```

### Queue from ISR

```c
void ADC_IRQHandler(void) {
    SensorData_t data;
    BaseType_t higher_priority_woken = pdFALSE;
    
    // Read ADC
    data.sensor_id = ADC_CHANNEL;
    data.temperature = ADC->DR;
    data.timestamp = get_timestamp();
    
    // Use FromISR version!
    xQueueSendFromISR(sensor_queue, &data, &higher_priority_woken);
    
    // Yield if a higher priority task was woken
    portYIELD_FROM_ISR(higher_priority_woken);
}
```

---

## Event Groups (Event Flags)

Event groups allow tasks to wait for combinations of events.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    EVENT GROUP STRUCTURE                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Event Group: 24 bits (configurable)                                │
│                                                                      │
│  Bit: 23 22 21 20 19 18 17 16 15 14 13 12 11 10  9  8  7  6  5  4  3  2  1  0 │
│       ┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐ │
│       │ 0│ 0│ 0│ 0│ 0│ 0│ 0│ 0│ 0│ 0│ 0│ 0│ 0│ 0│ 0│ 0│ 0│ 0│ 0│ 0│ 0│ 1│ 1│ 0│ │
│       └──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘ │
│                                                                      │
│  Define event bits:                                                  │
│  #define EVENT_BUTTON     (1 << 0)  // Bit 0                        │
│  #define EVENT_SENSOR     (1 << 1)  // Bit 1                        │
│  #define EVENT_TIMER      (1 << 2)  // Bit 2                        │
│  #define EVENT_NETWORK    (1 << 3)  // Bit 3                        │
│                                                                      │
│  Current state: EVENT_SENSOR | EVENT_TIMER set (0x06)               │
│                                                                      │
│  WAIT OPTIONS:                                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Wait for ANY: Wake when any specified bit is set            │    │
│  │ Wait for ALL: Wake only when all specified bits are set     │    │
│  │ Clear on exit: Automatically clear bits after waking        │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Event Group Example

```c
#define EVENT_BUTTON_PRESSED    (1 << 0)
#define EVENT_DATA_READY        (1 << 1)
#define EVENT_NETWORK_CONNECTED (1 << 2)
#define EVENT_ALL_INIT_DONE     (EVENT_BUTTON_PRESSED | EVENT_DATA_READY | EVENT_NETWORK_CONNECTED)

EventGroupHandle_t system_events;

void setup(void) {
    system_events = xEventGroupCreate();
}

// Task 1: Button handler
void button_task(void *pv) {
    while (1) {
        if (button_pressed()) {
            xEventGroupSetBits(system_events, EVENT_BUTTON_PRESSED);
        }
        vTaskDelay(pdMS_TO_TICKS(50));
    }
}

// Task 2: Sensor task
void sensor_task(void *pv) {
    while (1) {
        read_sensors();
        xEventGroupSetBits(system_events, EVENT_DATA_READY);
        vTaskDelay(pdMS_TO_TICKS(100));
    }
}

// Task 3: Wait for multiple events
void coordinator_task(void *pv) {
    EventBits_t bits;
    
    while (1) {
        // Wait for ALL events, clear on exit, no timeout
        bits = xEventGroupWaitBits(
            system_events,
            EVENT_ALL_INIT_DONE,    // Bits to wait for
            pdTRUE,                  // Clear bits on exit
            pdTRUE,                  // Wait for ALL bits
            portMAX_DELAY            // Wait forever
        );
        
        if ((bits & EVENT_ALL_INIT_DONE) == EVENT_ALL_INIT_DONE) {
            // All systems ready!
            process_all_data();
        }
    }
}
```

---

## Task Notifications

Lightweight alternative to semaphores/event groups (more efficient).

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TASK NOTIFICATIONS                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Each task has a built-in 32-bit notification value:                │
│                                                                      │
│  TCB                                                                 │
│  ┌────────────────────────────────────────┐                         │
│  │ ...                                    │                         │
│  │ notification_value: 0x00000000         │  ← 32-bit value         │
│  │ notification_state: NOT_WAITING        │  ← Pending flag         │
│  │ ...                                    │                         │
│  └────────────────────────────────────────┘                         │
│                                                                      │
│  ADVANTAGES:                                                         │
│  • 45% faster than binary semaphore                                 │
│  • Uses less RAM (no separate object)                               │
│  • Direct task-to-task (no queue handle needed)                     │
│                                                                      │
│  LIMITATIONS:                                                        │
│  • Can only notify one specific task                                │
│  • ISR cannot receive notifications (only send)                     │
│  • Not suitable for multiple producers to same consumer             │
│                                                                      │
│  NOTIFICATION ACTIONS:                                               │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ eNoAction:        Just wake the task                        │    │
│  │ eSetBits:         OR bits with current value                │    │
│  │ eIncrement:       Add 1 to current value                    │    │
│  │ eSetValueWithOverwrite: Set to specific value               │    │
│  │ eSetValueWithoutOverwrite: Set only if previous read        │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Task Notification Example

```c
TaskHandle_t worker_task_handle;

// ISR notifies task
void UART_IRQHandler(void) {
    BaseType_t higher_woken = pdFALSE;
    
    // Notify task (like giving a semaphore)
    vTaskNotifyGiveFromISR(worker_task_handle, &higher_woken);
    
    portYIELD_FROM_ISR(higher_woken);
}

// Task waits for notification
void worker_task(void *pv) {
    while (1) {
        // Wait for notification (like taking a semaphore)
        ulTaskNotifyTake(pdTRUE,  // Clear on exit
                        portMAX_DELAY);
        
        // Process UART data
        process_uart_data();
    }
}

// Using notification value as event flags
void event_task(void *pv) {
    uint32_t notification_value;
    
    while (1) {
        xTaskNotifyWait(
            0x00,               // Don't clear on entry
            UINT32_MAX,         // Clear all on exit
            &notification_value,
            portMAX_DELAY
        );
        
        if (notification_value & EVENT_BUTTON) {
            handle_button();
        }
        if (notification_value & EVENT_SENSOR) {
            handle_sensor();
        }
    }
}
```

---

## Stream Buffers

For continuous byte streams (like UART data).

```
┌─────────────────────────────────────────────────────────────────────┐
│                    STREAM BUFFER                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Byte-oriented FIFO with trigger level:                             │
│                                                                      │
│  ┌─────────────────────────────────────────────────────┐            │
│  │ H │ e │ l │ l │ o │   │ W │ o │ r │ l │ d │   │   │ │            │
│  └─────────────────────────────────────────────────────┘            │
│    ▲                                   ▲                 ▲          │
│  Read                                Write          Trigger=5       │
│                                                                      │
│  Trigger Level:                                                      │
│  • Reader only wakes when at least N bytes available               │
│  • Reduces context switches for small data packets                  │
│                                                                      │
│  Use Cases:                                                          │
│  • UART receive buffers                                              │
│  • DMA completion handling                                           │
│  • Network packet assembly                                           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

```c
StreamBufferHandle_t uart_stream;

void setup(void) {
    // 128 byte buffer, trigger at 1 byte
    uart_stream = xStreamBufferCreate(128, 1);
}

void UART_IRQHandler(void) {
    uint8_t byte = UART->DR;
    BaseType_t woken = pdFALSE;
    
    xStreamBufferSendFromISR(uart_stream, &byte, 1, &woken);
    portYIELD_FROM_ISR(woken);
}

void uart_task(void *pv) {
    uint8_t buffer[64];
    size_t received;
    
    while (1) {
        // Block until data available
        received = xStreamBufferReceive(uart_stream, 
                                        buffer, 
                                        sizeof(buffer),
                                        portMAX_DELAY);
        
        // Process received bytes
        process_data(buffer, received);
    }
}
```

---

## Queue Sets

Wait on multiple queues/semaphores simultaneously.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    QUEUE SET                                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Wait on multiple sources:                                          │
│                                                                      │
│  ┌──────────────┐                                                    │
│  │   Queue 1    │───┐                                                │
│  │  (Sensors)   │   │                                                │
│  └──────────────┘   │     ┌──────────────┐     ┌────────────┐       │
│  ┌──────────────┐   │     │              │     │   TASK     │       │
│  │   Queue 2    │───┼────▶│  QUEUE SET   │────▶│  (waits    │       │
│  │  (Network)   │   │     │              │     │   on set)  │       │
│  └──────────────┘   │     └──────────────┘     └────────────┘       │
│  ┌──────────────┐   │                                                │
│  │  Semaphore   │───┘                                                │
│  │   (Timer)    │                                                    │
│  └──────────────┘                                                    │
│                                                                      │
│  Task blocks on queue set, wakes when ANY member has data          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

```c
QueueSetHandle_t queue_set;
QueueHandle_t sensor_queue, network_queue;
SemaphoreHandle_t timer_sem;

void setup(void) {
    // Create queues and semaphore
    sensor_queue = xQueueCreate(10, sizeof(SensorData_t));
    network_queue = xQueueCreate(5, sizeof(NetworkPacket_t));
    timer_sem = xSemaphoreCreateBinary();
    
    // Create queue set (size = sum of all member lengths)
    queue_set = xQueueCreateSet(10 + 5 + 1);
    
    // Add members to set
    xQueueAddToSet(sensor_queue, queue_set);
    xQueueAddToSet(network_queue, queue_set);
    xQueueAddToSet(timer_sem, queue_set);
}

void handler_task(void *pv) {
    QueueSetMemberHandle_t activated;
    
    while (1) {
        // Wait for any member to have data
        activated = xQueueSelectFromSet(queue_set, portMAX_DELAY);
        
        if (activated == sensor_queue) {
            SensorData_t data;
            xQueueReceive(sensor_queue, &data, 0);
            handle_sensor(data);
        }
        else if (activated == network_queue) {
            NetworkPacket_t packet;
            xQueueReceive(network_queue, &packet, 0);
            handle_network(packet);
        }
        else if (activated == timer_sem) {
            xSemaphoreTake(timer_sem, 0);
            handle_timer();
        }
    }
}
```

---

## Comparison of ITC Mechanisms

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ITC SELECTION GUIDE                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────┬───────────────────────────────────────────────┐ │
│  │ Need           │ Best Choice                                   │ │
│  ├────────────────┼───────────────────────────────────────────────┤ │
│  │ Simple signal  │ Binary Semaphore or Task Notification         │ │
│  │ (ISR to task)  │ (Notification is faster)                      │ │
│  ├────────────────┼───────────────────────────────────────────────┤ │
│  │ Pass data      │ Queue (copies data, thread-safe)              │ │
│  │ between tasks  │                                               │ │
│  ├────────────────┼───────────────────────────────────────────────┤ │
│  │ Multiple event │ Event Group (up to 24 events)                 │ │
│  │ conditions     │                                               │ │
│  ├────────────────┼───────────────────────────────────────────────┤ │
│  │ Byte stream    │ Stream Buffer (UART, DMA)                     │ │
│  │                │                                               │ │
│  ├────────────────┼───────────────────────────────────────────────┤ │
│  │ Variable-size  │ Message Buffer                                │ │
│  │ messages       │                                               │ │
│  ├────────────────┼───────────────────────────────────────────────┤ │
│  │ Wait on        │ Queue Set                                     │ │
│  │ multiple src   │                                               │ │
│  ├────────────────┼───────────────────────────────────────────────┤ │
│  │ Large data     │ Queue of pointers (+ memory pool)             │ │
│  │ structures     │                                               │ │
│  └────────────────┴───────────────────────────────────────────────┘ │
│                                                                      │
│  RAM USAGE COMPARISON:                                               │
│  ┌──────────────────────┬─────────────────────────────────────────┐ │
│  │ Mechanism            │ Typical RAM (FreeRTOS)                  │ │
│  ├──────────────────────┼─────────────────────────────────────────┤ │
│  │ Task Notification    │ 0 (built into TCB)                      │ │
│  │ Binary Semaphore     │ ~80 bytes                               │ │
│  │ Queue (10 × 4 bytes) │ ~120 bytes                              │ │
│  │ Event Group          │ ~24 bytes                               │ │
│  │ Stream Buffer (128B) │ ~180 bytes                              │ │
│  └──────────────────────┴─────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Mechanism | Data Type | Multi-Reader | From ISR | RAM |
|-----------|-----------|--------------|----------|-----|
| **Queue** | Struct/value | Yes | Yes (FromISR) | Medium |
| **Event Group** | Bits (24) | Yes | Yes | Low |
| **Task Notification** | 32-bit | No (1 task) | Yes | None |
| **Stream Buffer** | Bytes | No (1:1) | Yes | Medium |
| **Message Buffer** | Variable msg | No (1:1) | Yes | Medium |
| **Queue Set** | Multiple sources | N/A | No | Low |

---

## Quick Revision Questions

1. **What is the difference between a queue and a stream buffer? When would you use each?**

2. **A sensor task needs to send temperature readings to a display task. Design the communication using a queue. Include the data structure and both task implementations.**

3. **What are the advantages of task notifications over binary semaphores? What are the limitations?**

4. **An embedded system needs to wait for three events: button press, sensor ready, and network connected. Which ITC mechanism is best? Show how to wait for all three.**

5. **Why must you use "FromISR" versions of queue/semaphore functions in an interrupt handler? What happens if you use the regular version?**

6. **You need to handle data from both UART and SPI, each with its own queue. How can a single task efficiently wait for data from either source?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Chapter 4.5: Synchronization](05-synchronization.md) | [Unit Index](README.md) | [Unit 5: Communication Protocols](../05-Communication-Protocols/README.md) |

---
