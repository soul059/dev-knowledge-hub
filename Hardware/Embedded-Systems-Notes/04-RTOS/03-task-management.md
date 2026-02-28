# Chapter 4.3: Task Management

## Introduction

Task management encompasses creating, deleting, suspending, resuming, and prioritizing tasks. Proper task design is crucial for building reliable real-time systems with predictable behavior.

---

## Task Creation

### FreeRTOS Task Creation API

```c
BaseType_t xTaskCreate(
    TaskFunction_t pvTaskCode,      // Pointer to task function
    const char * const pcName,      // Task name (for debugging)
    configSTACK_DEPTH_TYPE usStackDepth,  // Stack size (words)
    void * const pvParameters,      // Parameters passed to task
    UBaseType_t uxPriority,         // Task priority
    TaskHandle_t * const pxCreatedTask    // Handle (output)
);
```

### Task Creation Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TASK CREATION PROCESS                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  xTaskCreate() called                                               │
│         │                                                           │
│         ▼                                                           │
│  ┌─────────────────────────────────────────┐                        │
│  │  1. ALLOCATE TCB                        │                        │
│  │     - From heap or static pool          │                        │
│  │     - Initialize all fields             │                        │
│  └──────────────────┬──────────────────────┘                        │
│                     ▼                                                │
│  ┌─────────────────────────────────────────┐                        │
│  │  2. ALLOCATE STACK                      │                        │
│  │     - Size = usStackDepth × 4 bytes     │                        │
│  │     - Fill with pattern (0xA5A5A5A5)    │                        │
│  └──────────────────┬──────────────────────┘                        │
│                     ▼                                                │
│  ┌─────────────────────────────────────────┐                        │
│  │  3. INITIALIZE STACK FRAME              │                        │
│  │     - PC = pvTaskCode (entry point)     │                        │
│  │     - R0 = pvParameters                 │                        │
│  │     - LR = task exit handler            │                        │
│  │     - xPSR = Thumb bit set              │                        │
│  └──────────────────┬──────────────────────┘                        │
│                     ▼                                                │
│  ┌─────────────────────────────────────────┐                        │
│  │  4. SET TCB FIELDS                      │                        │
│  │     - priority = uxPriority             │                        │
│  │     - state = READY                     │                        │
│  │     - stack_ptr = initialized SP        │                        │
│  └──────────────────┬──────────────────────┘                        │
│                     ▼                                                │
│  ┌─────────────────────────────────────────┐                        │
│  │  5. ADD TO READY QUEUE                  │                        │
│  │     - Insert at tail of priority list   │                        │
│  │     - Update ready bitmap               │                        │
│  └──────────────────┬──────────────────────┘                        │
│                     ▼                                                │
│  ┌─────────────────────────────────────────┐                        │
│  │  6. TRIGGER SCHEDULER (if running)      │                        │
│  │     - May preempt current task          │                        │
│  └─────────────────────────────────────────┘                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Initial Stack Frame

```
┌─────────────────────────────────────────────────────────────────────┐
│                    INITIAL STACK FRAME (ARM Cortex-M)                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Stack Top (High Address)                                           │
│        │                                                             │
│        ▼                                                             │
│  ┌─────────────────────────────────────────┐                        │
│  │  xPSR      = 0x01000000                 │  ← Thumb bit set       │
│  │  PC        = task_function              │  ← Entry point         │
│  │  LR        = task_exit_error            │  ← If task returns     │
│  │  R12       = 0                          │                        │
│  │  R3        = 0                          │  (Auto-saved by HW)    │
│  │  R2        = 0                          │                        │
│  │  R1        = 0                          │                        │
│  │  R0        = pvParameters               │  ← Task argument       │
│  ├─────────────────────────────────────────┤                        │
│  │  R11       = 0                          │                        │
│  │  R10       = 0                          │                        │
│  │  R9        = 0                          │  (Saved by software)   │
│  │  R8        = 0                          │                        │
│  │  R7        = 0                          │                        │
│  │  R6        = 0                          │                        │
│  │  R5        = 0                          │                        │
│  │  R4        = 0                          │                        │
│  └─────────────────────────────────────────┘                        │
│        ▲                                                             │
│        │                                                             │
│  tcb->stack_ptr (Initial SP)                                        │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Task Function Structure

```c
/* Task function prototype */
void task_function(void *pvParameters) {
    // 1. One-time initialization
    init_resources();
    
    // 2. Infinite loop (tasks should never return)
    while (1) {
        // 3. Wait for event/data/time
        wait_for_trigger();
        
        // 4. Do work
        process_data();
        
        // 5. Optional: yield or delay
        vTaskDelay(pdMS_TO_TICKS(100));
    }
    
    // 6. Should never reach here
    // If task must end, call vTaskDelete(NULL);
}
```

### Complete Example

```c
#include "FreeRTOS.h"
#include "task.h"

// Task handles
TaskHandle_t sensor_task_handle;
TaskHandle_t display_task_handle;

// Task configuration
#define SENSOR_STACK_SIZE   256   // Words (1024 bytes)
#define SENSOR_PRIORITY     3
#define DISPLAY_STACK_SIZE  512   // Words (2048 bytes)
#define DISPLAY_PRIORITY    2

// Sensor task: reads sensor every 100ms
void sensor_task(void *pvParameters) {
    uint32_t sensor_id = (uint32_t)pvParameters;
    TickType_t last_wake = xTaskGetTickCount();
    
    while (1) {
        // Periodic execution (precise timing)
        vTaskDelayUntil(&last_wake, pdMS_TO_TICKS(100));
        
        // Read sensor
        int value = read_sensor(sensor_id);
        
        // Send to queue for display task
        xQueueSend(display_queue, &value, 0);
    }
}

// Display task: updates LCD when data available
void display_task(void *pvParameters) {
    int value;
    
    while (1) {
        // Block until data available
        if (xQueueReceive(display_queue, &value, portMAX_DELAY)) {
            update_lcd(value);
        }
    }
}

int main(void) {
    // Initialize hardware
    hardware_init();
    
    // Create tasks
    xTaskCreate(sensor_task, "Sensor", SENSOR_STACK_SIZE,
                (void*)1, SENSOR_PRIORITY, &sensor_task_handle);
    
    xTaskCreate(display_task, "Display", DISPLAY_STACK_SIZE,
                NULL, DISPLAY_PRIORITY, &display_task_handle);
    
    // Start scheduler (never returns)
    vTaskStartScheduler();
    
    // Should never reach here
    while (1);
}
```

---

## Task Priorities

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PRIORITY ASSIGNMENT                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Priority                     Task Type                              │
│     │                                                                │
│  Highest ┌────────────────────────────────────────────────────┐     │
│     0    │  Critical real-time (motor control, safety)       │     │
│          └────────────────────────────────────────────────────┘     │
│     1    ┌────────────────────────────────────────────────────┐     │
│          │  High-priority I/O (sensor reading)               │     │
│          └────────────────────────────────────────────────────┘     │
│     2    ┌────────────────────────────────────────────────────┐     │
│          │  Communication (UART, network)                    │     │
│          └────────────────────────────────────────────────────┘     │
│     3    ┌────────────────────────────────────────────────────┐     │
│          │  Data processing                                  │     │
│          └────────────────────────────────────────────────────┘     │
│     4    ┌────────────────────────────────────────────────────┐     │
│          │  User interface, display                          │     │
│          └────────────────────────────────────────────────────┘     │
│  Lowest  ┌────────────────────────────────────────────────────┐     │
│  (idle)  │  Background tasks, logging                        │     │
│          └────────────────────────────────────────────────────┘     │
│                                                                      │
│                                                                      │
│  PRIORITY GUIDELINES:                                                │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ • Shorter period → Higher priority (Rate Monotonic)        │    │
│  │ • Safety-critical → Highest priority                       │    │
│  │ • Avoid priority 0 for application tasks (reserve for OS) │    │
│  │ • Use few distinct priority levels (4-8 typically)         │    │
│  │ • Tasks with same priority: round-robin within level       │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Dynamic Priority Change

```c
// Get current priority
UBaseType_t current = uxTaskPriorityGet(task_handle);

// Change priority
vTaskPrioritySet(task_handle, new_priority);

// Change own priority
vTaskPrioritySet(NULL, new_priority);
```

---

## Task Control Operations

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TASK CONTROL API                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────────┬────────────────────────────────────────────┐  │
│  │ Function         │ Effect                                     │  │
│  ├──────────────────┼────────────────────────────────────────────┤  │
│  │ vTaskSuspend()   │ RUNNING/READY → SUSPENDED                  │  │
│  │                  │ Task won't run until resumed               │  │
│  ├──────────────────┼────────────────────────────────────────────┤  │
│  │ vTaskResume()    │ SUSPENDED → READY                          │  │
│  │                  │ Task can run again                         │  │
│  ├──────────────────┼────────────────────────────────────────────┤  │
│  │ xTaskResumeFromISR() │ Resume from ISR (thread-safe)        │  │
│  ├──────────────────┼────────────────────────────────────────────┤  │
│  │ vTaskDelete()    │ Any → DELETED                              │  │
│  │                  │ Frees TCB and stack                        │  │
│  ├──────────────────┼────────────────────────────────────────────┤  │
│  │ vTaskDelay()     │ RUNNING → BLOCKED for N ticks              │  │
│  │                  │ Relative delay from now                    │  │
│  ├──────────────────┼────────────────────────────────────────────┤  │
│  │ vTaskDelayUntil()│ RUNNING → BLOCKED until time               │  │
│  │                  │ Absolute delay (periodic tasks)            │  │
│  ├──────────────────┼────────────────────────────────────────────┤  │
│  │ taskYIELD()      │ Yield to same-priority tasks               │  │
│  │                  │ RUNNING → READY (voluntarily)              │  │
│  └──────────────────┴────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Suspend and Resume Example

```c
TaskHandle_t motor_task;

void motor_control_task(void *pv) {
    while (1) {
        // Motor control logic
        control_motor();
        vTaskDelay(pdMS_TO_TICKS(10));
    }
}

void emergency_stop_isr(void) {
    // Immediately stop motor task
    vTaskSuspend(motor_task);
    stop_motor_hardware();
}

void resume_operation(void) {
    // Resume after safety check
    if (safety_check_passed()) {
        vTaskResume(motor_task);
    }
}
```

---

## Delay Functions

### vTaskDelay vs vTaskDelayUntil

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DELAY COMPARISON                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  vTaskDelay(100):  RELATIVE delay from when function called        │
│                                                                      │
│   Task executes              Task executes                          │
│      │                          │                                    │
│  ────┴───────────────────────────┴─────────────────────────────▶    │
│      │        100ms              │        100ms                      │
│      │◄──────────────────────────│◄────────────────────             │
│      │     ┌──────────────┐      │     ┌──────────────┐             │
│      │     │ 10ms work    │      │     │ 15ms work    │             │
│      │                    │      │                    │              │
│            │◄── 100ms ───►│           │◄── 100ms ───►│              │
│                                                                      │
│  Total period = work_time + 100ms (VARIABLE!)                       │
│                                                                      │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                      │
│  vTaskDelayUntil(&last, 100):  ABSOLUTE delay (periodic)            │
│                                                                      │
│   Task executes              Task executes                          │
│      │                          │                                    │
│  ────┴──────────────────────────┴─────────────────────────────▶     │
│      │◄─────── 100ms ──────────►│◄────── 100ms ─────────►│          │
│      │     ┌─────┐              │    ┌───────┐           │          │
│      │     │10ms │              │    │ 15ms  │           │          │
│                                                                      │
│  Total period = exactly 100ms (FIXED!)                              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Periodic Task with vTaskDelayUntil

```c
void periodic_task(void *pv) {
    TickType_t last_wake_time = xTaskGetTickCount();
    const TickType_t period = pdMS_TO_TICKS(100);  // 100ms
    
    while (1) {
        // This ensures exactly 100ms between starts
        vTaskDelayUntil(&last_wake_time, period);
        
        // Do work (variable execution time is OK)
        read_sensors();
        process_data();
        update_outputs();
    }
}
```

---

## Task Information and Debug

### Task Status Query

```c
// Get task state
eTaskState state = eTaskGetState(task_handle);

// States: eRunning, eReady, eBlocked, eSuspended, eDeleted

// Get task name
char* name = pcTaskGetName(task_handle);

// Get high water mark (minimum free stack ever)
UBaseType_t stack_remaining = uxTaskGetStackHighWaterMark(task_handle);
```

### Runtime Statistics

```c
// Enable in FreeRTOSConfig.h:
// #define configUSE_TRACE_FACILITY 1
// #define configGENERATE_RUN_TIME_STATS 1

void print_task_stats(void) {
    char buffer[512];
    
    // Get formatted task list
    vTaskList(buffer);
    printf("Task\t\tState\tPri\tStack\tNum\n");
    printf("%s\n", buffer);
    
    // Get CPU usage
    vTaskGetRunTimeStats(buffer);
    printf("Task\t\tAbs Time\t%% Time\n");
    printf("%s\n", buffer);
}

/* Example output:
Task            State   Pri     Stack   Num
Sensor          R       3       180     1
Display         B       2       350     2
Idle            R       0       120     3

Task            Abs Time        % Time
Sensor          5432            12%
Display         2156            5%
Idle            35000           83%
*/
```

---

## Static vs Dynamic Task Creation

```
┌─────────────────────────────────────────────────────────────────────┐
│                    STATIC vs DYNAMIC ALLOCATION                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  DYNAMIC (xTaskCreate):                                              │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ TaskHandle_t handle;                                        │    │
│  │ xTaskCreate(task_func, "name", 256, NULL, 2, &handle);      │    │
│  │                                                             │    │
│  │ ✓ Simple API                                                │    │
│  │ ✓ Flexible (create/delete at runtime)                      │    │
│  │ ✗ Requires heap                                             │    │
│  │ ✗ Allocation may fail                                       │    │
│  │ ✗ Fragmentation risk                                        │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  STATIC (xTaskCreateStatic):                                         │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ // Statically allocate TCB and stack                        │    │
│  │ static StaticTask_t task_tcb;                               │    │
│  │ static StackType_t task_stack[256];                         │    │
│  │                                                             │    │
│  │ TaskHandle_t handle = xTaskCreateStatic(                    │    │
│  │     task_func, "name",                                      │    │
│  │     256,              // Stack size                         │    │
│  │     NULL,             // Parameters                         │    │
│  │     2,                // Priority                           │    │
│  │     task_stack,       // Stack buffer                       │    │
│  │     &task_tcb         // TCB buffer                         │    │
│  │ );                                                          │    │
│  │                                                             │    │
│  │ ✓ No heap required                                          │    │
│  │ ✓ Guaranteed allocation (no runtime failure)               │    │
│  │ ✓ Deterministic memory usage                                │    │
│  │ ✓ Required for safety-critical systems                      │    │
│  │ ✗ Cannot delete and reclaim memory                          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  RECOMMENDATION:                                                     │
│  • Use STATIC for fixed set of tasks (most embedded systems)        │
│  • Use DYNAMIC only if tasks are truly temporary                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## The Idle Task

```
┌─────────────────────────────────────────────────────────────────────┐
│                    IDLE TASK                                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  The Idle task is created automatically by the kernel:              │
│  • Runs at lowest priority (0 in FreeRTOS convention)               │
│  • Ensures there's always a task to run                             │
│  • Handles cleanup of deleted tasks                                 │
│  • Can trigger low-power modes                                      │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  // Idle task (simplified)                                  │    │
│  │  void idle_task(void) {                                     │    │
│  │      while (1) {                                            │    │
│  │          // Clean up deleted task memory                    │    │
│  │          prvCheckTasksWaitingTermination();                 │    │
│  │                                                             │    │
│  │          // Call user hook (if configured)                  │    │
│  │          #if configUSE_IDLE_HOOK                            │    │
│  │          vApplicationIdleHook();                            │    │
│  │          #endif                                             │    │
│  │      }                                                      │    │
│  │  }                                                          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  IDLE HOOK (user code in idle time):                                │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  void vApplicationIdleHook(void) {                          │    │
│  │      // Enter low-power mode                                │    │
│  │      __WFI();  // Wait For Interrupt (sleep)                │    │
│  │                                                             │    │
│  │      // Or do background work                               │    │
│  │      // (must not block!)                                   │    │
│  │  }                                                          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  TICKLESS IDLE (for extreme power saving):                          │
│  • Suppress tick interrupts during long idle periods               │
│  • Wake on next task timeout or external interrupt                 │
│  • Requires configUSE_TICKLESS_IDLE = 1                            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Task Design Guidelines

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TASK DESIGN BEST PRACTICES                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. TASK DECOMPOSITION                                              │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ • One task per logical function (sensor, actuator, comm)   │    │
│  │ • Avoid monolithic tasks that do everything               │    │
│  │ • Keep tasks focused and cohesive                          │    │
│  │ • Typical system: 5-15 tasks                               │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  2. STACK SIZING                                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Rule of thumb:                                              │    │
│  │   Stack = max_call_depth × avg_frame_size + interrupt_ctx  │    │
│  │         + safety_margin(25%)                                │    │
│  │                                                             │    │
│  │ Typical sizes:                                              │    │
│  │   Simple task (no nested calls):    256-512 bytes          │    │
│  │   Moderate task:                    512-1024 bytes         │    │
│  │   Complex task (with printf):       2048+ bytes            │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  3. PRIORITY ASSIGNMENT                                              │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ • Rate Monotonic: shorter period = higher priority         │    │
│  │ • Urgency: time-critical tasks higher                      │    │
│  │ • Use few priority levels (reduces analysis complexity)    │    │
│  │ • Avoid priority 0 (reserve for kernel/safety)             │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  4. AVOID COMMON MISTAKES                                            │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ ✗ Don't busy-wait (burns CPU)                              │    │
│  │ ✗ Don't share global data without synchronization          │    │
│  │ ✗ Don't call blocking functions from ISR                   │    │
│  │ ✗ Don't return from task function (infinite loop!)         │    │
│  │ ✗ Don't create too many tasks (overhead)                   │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| API Function | Purpose | Notes |
|--------------|---------|-------|
| `xTaskCreate()` | Create task dynamically | Uses heap |
| `xTaskCreateStatic()` | Create task statically | No heap needed |
| `vTaskDelete()` | Delete task | Pass NULL for self |
| `vTaskSuspend()` | Pause task | Stops execution |
| `vTaskResume()` | Resume suspended task | Moves to READY |
| `vTaskDelay()` | Relative delay | From call time |
| `vTaskDelayUntil()` | Absolute delay | For periodic tasks |
| `vTaskPrioritySet()` | Change priority | Runtime adjustment |
| `uxTaskGetStackHighWaterMark()` | Check stack usage | Debug/sizing |
| `taskYIELD()` | Voluntary yield | Same-priority tasks |

---

## Quick Revision Questions

1. **What are the minimum required parameters for xTaskCreate()? What happens if stack size is too small?**

2. **A task reads a sensor every 50ms. Should it use vTaskDelay() or vTaskDelayUntil()? Explain the timing difference.**

3. **What is the purpose of the idle task? What should (and shouldn't) you do in the idle hook?**

4. **Explain the difference between static and dynamic task creation. When would you prefer each?**

5. **A system has tasks with periods: 10ms, 25ms, 50ms, 100ms. Using Rate Monotonic scheduling, assign priorities (0=highest).**

6. **Why must a task function contain an infinite loop? What happens if a task returns?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Chapter 4.2: RTOS Architecture](02-rtos-architecture.md) | [Unit Index](README.md) | [Chapter 4.4: Scheduling Algorithms](04-scheduling-algorithms.md) |

---
