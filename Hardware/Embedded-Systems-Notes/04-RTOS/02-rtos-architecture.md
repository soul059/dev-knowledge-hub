# Chapter 4.2: RTOS Architecture

## Introduction

An RTOS kernel provides the core services required for multi-tasking: task management, scheduling, synchronization, and resource management. Understanding the internal architecture of an RTOS is essential for designing efficient and reliable real-time systems.

---

## RTOS Kernel Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    RTOS KERNEL STRUCTURE                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    APPLICATION LAYER                         │    │
│  │   Task 1    Task 2    Task 3    ...    Task N               │    │
│  └─────────────────────────┬───────────────────────────────────┘    │
│                            │ RTOS API Calls                          │
│                            ▼                                         │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    KERNEL SERVICES                           │    │
│  │  ┌───────────────────────────────────────────────────────┐  │    │
│  │  │  Task Management                                      │  │    │
│  │  │  • Create/Delete tasks                                │  │    │
│  │  │  • Suspend/Resume tasks                               │  │    │
│  │  │  • Change priority                                    │  │    │
│  │  └───────────────────────────────────────────────────────┘  │    │
│  │  ┌───────────────────────────────────────────────────────┐  │    │
│  │  │  Scheduler                                            │  │    │
│  │  │  • Priority-based preemption                          │  │    │
│  │  │  • Ready queue management                             │  │    │
│  │  │  • Context switching                                  │  │    │
│  │  └───────────────────────────────────────────────────────┘  │    │
│  │  ┌───────────────────────────────────────────────────────┐  │    │
│  │  │  Synchronization                                      │  │    │
│  │  │  • Semaphores, Mutexes                                │  │    │
│  │  │  • Event flags, Queues                                │  │    │
│  │  │  • Critical sections                                  │  │    │
│  │  └───────────────────────────────────────────────────────┘  │    │
│  │  ┌───────────────────────────────────────────────────────┐  │    │
│  │  │  Timer Services                                       │  │    │
│  │  │  • System tick                                        │  │    │
│  │  │  • Delays, Timeouts                                   │  │    │
│  │  │  • Software timers                                    │  │    │
│  │  └───────────────────────────────────────────────────────┘  │    │
│  │  ┌───────────────────────────────────────────────────────┐  │    │
│  │  │  Memory Management                                    │  │    │
│  │  │  • Task stacks                                        │  │    │
│  │  │  • Memory pools                                       │  │    │
│  │  │  • Dynamic allocation (optional)                      │  │    │
│  │  └───────────────────────────────────────────────────────┘  │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                            │                                         │
│                            ▼                                         │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    HARDWARE ABSTRACTION                      │    │
│  │   Interrupts     Timer      Memory      Peripherals         │    │
│  │   (NVIC)        (SysTick)   (MPU)                           │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Task Control Block (TCB)

The TCB is the kernel's data structure for managing each task.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TASK CONTROL BLOCK (TCB)                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  TCB for Task "SensorRead"                                  │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │  Task ID           │  0x0003                                │    │
│  │  Task Name         │  "SensorRead"                          │    │
│  │  State             │  READY                                 │    │
│  │  Priority          │  5 (0=highest)                         │    │
│  │  Stack Pointer     │  0x20001000                            │    │
│  │  Stack Base        │  0x20000C00                            │    │
│  │  Stack Size        │  1024 bytes                            │    │
│  │  Entry Point       │  sensor_task()                         │    │
│  │  Wait Object       │  NULL (not waiting)                    │    │
│  │  Timeout           │  0                                     │    │
│  │  Time Slice        │  10 ticks                              │    │
│  │  Next TCB (list)   │  0x20002000                            │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  TCB LINKED LIST:                                                    │
│                                                                      │
│  Ready Queue (Priority 5):                                          │
│                                                                      │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐                         │
│  │ TCB_A   │───▶│ TCB_B   │───▶│ TCB_C   │───▶ NULL                │
│  │ Task A  │    │ Task B  │    │ Task C  │                          │
│  │ Pri: 5  │    │ Pri: 5  │    │ Pri: 5  │                          │
│  └─────────┘    └─────────┘    └─────────┘                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### TCB Structure in C

```c
typedef struct TCB {
    // Context (saved on stack, pointer here)
    uint32_t *stack_ptr;        // Current stack pointer
    
    // Task identification
    uint32_t task_id;           // Unique task ID
    char     name[16];          // Task name for debugging
    
    // State management
    TaskState_t state;          // READY, RUNNING, BLOCKED, SUSPENDED
    uint8_t  priority;          // 0 = highest priority
    uint8_t  base_priority;     // Original priority (for inheritance)
    
    // Stack info
    uint32_t *stack_base;       // Bottom of stack
    uint32_t stack_size;        // Stack size in bytes
    
    // Timing
    uint32_t delay_ticks;       // Ticks remaining for delay
    uint32_t time_slice;        // Ticks for round-robin
    
    // Synchronization
    void     *wait_object;      // Semaphore/queue being waited on
    
    // Linked list pointers
    struct TCB *next;           // Next TCB in queue
    struct TCB *prev;           // Previous TCB in queue
} TCB_t;
```

---

## Task States

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TASK STATE DIAGRAM                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                         ┌─────────────┐                             │
│                         │   DORMANT   │                             │
│                         │  (Created)  │                             │
│                         └──────┬──────┘                             │
│                                │ vTaskCreate()                       │
│                                ▼                                     │
│                         ┌─────────────┐                             │
│            ┌───────────▶│    READY    │◀────────────┐               │
│            │            │  (Runnable) │             │               │
│            │            └──────┬──────┘             │               │
│            │                   │                    │               │
│            │                   │ Scheduler          │               │
│            │                   │ selects            │               │
│            │ Preempted         ▼                    │ Event/         │
│            │ (higher pri)┌─────────────┐            │ Timeout        │
│            │             │   RUNNING   │            │               │
│            └─────────────│  (Active)   │            │               │
│                          └──────┬──────┘            │               │
│                                 │                   │               │
│            ┌────────────────────┼───────────────────┤               │
│            │                    │                   │               │
│            ▼                    ▼                   │               │
│     ┌─────────────┐      ┌─────────────┐           │               │
│     │  SUSPENDED  │      │   BLOCKED   │───────────┘               │
│     │ (Paused by  │      │ (Waiting)   │                           │
│     │  request)   │      └─────────────┘                           │
│     └─────────────┘                                                 │
│            │                                                        │
│            │ vTaskResume()                                          │
│            └──────────────────────▶ READY                           │
│                                                                      │
├─────────────────────────────────────────────────────────────────────┤
│  STATE TRANSITIONS:                                                  │
│                                                                      │
│  ┌────────────────────┬──────────────────────────────────────────┐  │
│  │ From → To          │ Trigger                                  │  │
│  ├────────────────────┼──────────────────────────────────────────┤  │
│  │ Dormant → Ready    │ Task creation (xTaskCreate)              │  │
│  │ Ready → Running    │ Scheduler selects (highest priority)     │  │
│  │ Running → Ready    │ Preempted by higher priority task        │  │
│  │ Running → Blocked  │ Wait for semaphore/queue/delay           │  │
│  │ Blocked → Ready    │ Event occurs or timeout expires          │  │
│  │ Running → Suspended│ vTaskSuspend() called                    │  │
│  │ Suspended → Ready  │ vTaskResume() called                     │  │
│  │ Any → Dormant      │ vTaskDelete() called                     │  │
│  └────────────────────┴──────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Context Switching

Context switching saves the current task's state and restores another task's state.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CONTEXT SWITCH MECHANISM                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  STEP 1: Save Current Task Context                                  │
│                                                                      │
│  Task A Running         PUSH to Stack A                              │
│  ┌─────────────────┐    ┌─────────────────┐                         │
│  │ R0-R12          │───▶│ R0              │ ◀── SP (Stack A)        │
│  │ LR (R14)        │    │ R1              │                         │
│  │ PC (R15)        │    │ R2              │                         │
│  │ PSR             │    │ ...             │                         │
│  │ SP (R13)        │    │ R12             │                         │
│  └─────────────────┘    │ LR              │                         │
│                         │ PC              │                         │
│                         │ xPSR            │                         │
│                         └─────────────────┘                         │
│                                                                      │
│  STEP 2: Update TCB                                                  │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ TCB_A.stack_ptr = current SP                                │    │
│  │ TCB_A.state = READY                                         │    │
│  │ current_task = TCB_B                                        │    │
│  │ TCB_B.state = RUNNING                                       │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  STEP 3: Restore New Task Context                                   │
│                                                                      │
│  ┌─────────────────┐    POP from Stack B                            │
│  │ R0              │ ◀── SP (Stack B)                               │
│  │ R1              │    ┌─────────────────┐                         │
│  │ R2              │───▶│ R0-R12          │  Task B Runs            │
│  │ ...             │    │ LR              │                         │
│  │ R12             │    │ PC ─────────────┼─▶ Continues execution   │
│  │ LR              │    │ PSR             │                         │
│  │ PC              │    └─────────────────┘                         │
│  │ xPSR            │                                                │
│  └─────────────────┘                                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Context Switch Implementation (ARM Cortex-M)

```c
// Triggered by PendSV exception (lowest priority)
__attribute__((naked)) void PendSV_Handler(void) {
    __asm volatile(
        // === Save current task context ===
        "MRS     R0, PSP              \n"  // Get Process Stack Pointer
        "STMDB   R0!, {R4-R11}        \n"  // Save R4-R11 (R0-R3 auto-saved)
        
        // Save SP to current TCB
        "LDR     R1, =current_tcb     \n"
        "LDR     R1, [R1]             \n"
        "STR     R0, [R1]             \n"  // TCB->stack_ptr = SP
        
        // === Call scheduler to pick next task ===
        "PUSH    {LR}                 \n"
        "BL      scheduler_next_task  \n"  // Returns new TCB in current_tcb
        "POP     {LR}                 \n"
        
        // === Restore new task context ===
        "LDR     R1, =current_tcb     \n"
        "LDR     R1, [R1]             \n"
        "LDR     R0, [R1]             \n"  // SP = new_TCB->stack_ptr
        
        "LDMIA   R0!, {R4-R11}        \n"  // Restore R4-R11
        "MSR     PSP, R0              \n"  // Set new PSP
        
        "BX      LR                   \n"  // Return (restores R0-R3, PC, PSR)
    );
}
```

---

## System Tick and Timing

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SYSTEM TICK (SysTick)                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SysTick Timer generates periodic interrupts for:                   │
│    • Time-slicing (round-robin among equal priorities)              │
│    • Delay countdown (vTaskDelay)                                   │
│    • Timeout management                                              │
│    • Software timer callbacks                                        │
│                                                                      │
│                                                                      │
│  TIMING DIAGRAM:                                                     │
│                                                                      │
│  SysTick │    │    │    │    │    │    │    │    │                  │
│  IRQ     ▼    ▼    ▼    ▼    ▼    ▼    ▼    ▼    ▼                  │
│       ───┬────┬────┬────┬────┬────┬────┬────┬────┬───▶ Time         │
│          │    │    │    │    │    │    │    │    │                  │
│          │◀──►│                                                      │
│          1 tick = 1ms (typical)                                      │
│                                                                      │
│                                                                      │
│  TICK INTERRUPT HANDLER:                                             │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  void SysTick_Handler(void) {                               │    │
│  │      // 1. Increment system tick counter                    │    │
│  │      tick_count++;                                          │    │
│  │                                                             │    │
│  │      // 2. Decrement delay counters for blocked tasks       │    │
│  │      for each task in delay_list:                           │    │
│  │          if (--task->delay_ticks == 0)                      │    │
│  │              move task to ready_queue;                      │    │
│  │                                                             │    │
│  │      // 3. Check software timers                            │    │
│  │      process_expired_timers();                              │    │
│  │                                                             │    │
│  │      // 4. Trigger scheduler (if preemption enabled)        │    │
│  │      if (preemption_needed)                                 │    │
│  │          trigger_PendSV();                                  │    │
│  │  }                                                          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│                                                                      │
│  TICK RATE SELECTION:                                                │
│                                                                      │
│  ┌──────────────────┬────────────────────────────────────────────┐  │
│  │ Tick Rate        │ Trade-offs                                 │  │
│  ├──────────────────┼────────────────────────────────────────────┤  │
│  │ 1 kHz (1ms)      │ Good resolution, ~1% CPU overhead          │  │
│  │ 100 Hz (10ms)    │ Low overhead, coarse timing                │  │
│  │ 10 kHz (100μs)   │ Fine resolution, higher overhead           │  │
│  └──────────────────┴────────────────────────────────────────────┘  │
│                                                                      │
│  CPU Overhead = (Tick_ISR_Time / Tick_Period) × 100%                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Ready Queue Management

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PRIORITY-BASED READY QUEUES                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Priority 0 (Highest):                                              │
│  ┌─────────┐                                                        │
│  │ TaskA   │───▶ NULL                                               │
│  └─────────┘                                                        │
│                                                                      │
│  Priority 1:                                                         │
│  ┌─────────┐    ┌─────────┐                                         │
│  │ TaskB   │───▶│ TaskC   │───▶ NULL                                │
│  └─────────┘    └─────────┘                                         │
│                                                                      │
│  Priority 2:                                                         │
│  NULL (empty)                                                        │
│                                                                      │
│  Priority 3:                                                         │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐                          │
│  │ TaskD   │───▶│ TaskE   │───▶│ TaskF   │───▶ NULL                 │
│  └─────────┘    └─────────┘    └─────────┘                          │
│                                                                      │
│       ...                                                            │
│                                                                      │
│  Priority N (Lowest):                                               │
│  ┌─────────┐                                                        │
│  │ Idle    │───▶ NULL                                               │
│  └─────────┘                                                        │
│                                                                      │
│                                                                      │
│  SCHEDULER ALGORITHM:                                                │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  TCB* get_highest_priority_task(void) {                     │    │
│  │      for (pri = 0; pri < MAX_PRIORITY; pri++) {             │    │
│  │          if (ready_queue[pri] != NULL) {                    │    │
│  │              return ready_queue[pri];                       │    │
│  │          }                                                  │    │
│  │      }                                                      │    │
│  │      return idle_task;  // Always returns something         │    │
│  │  }                                                          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  OPTIMIZED: Bitmap for O(1) lookup                                  │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  ready_bitmap: [1][1][0][1][0][0]...[1]                     │    │
│  │                 ↑                                            │    │
│  │                 Bit set = tasks ready at this priority      │    │
│  │                                                             │    │
│  │  highest_pri = CLZ(ready_bitmap);  // Count Leading Zeros   │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Memory Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    RTOS MEMORY LAYOUT                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  HIGH ADDRESS                                                        │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    MSP (Main Stack)                          │    │
│  │                    Used by: ISRs, Kernel                     │    │
│  │                    Size: ~1KB typical                        │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │                    Task N Stack                              │    │
│  │                    (grows down ↓)                            │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │                    Task 2 Stack                              │    │
│  │                    (grows down ↓)                            │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │                    Task 1 Stack                              │    │
│  │                    (grows down ↓)                            │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │                    Heap (Dynamic Memory)                     │    │
│  │                    (grows up ↑)                              │    │
│  │                    Used by: malloc, pvPortMalloc            │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │                    BSS (Zero-initialized data)               │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │                    Data (Initialized variables)              │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │                    Kernel Data (TCBs, Queues)               │    │
│  │                    Statically allocated                      │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │                    Code (Flash)                              │    │
│  │                    .text section                             │    │
│  └─────────────────────────────────────────────────────────────┘    │
│  LOW ADDRESS                                                         │
│                                                                      │
│                                                                      │
│  STACK SIZING:                                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Stack usage = Local variables + Function call depth +       │    │
│  │               Interrupt context + Safety margin             │    │
│  │                                                             │    │
│  │ Typical sizes:                                               │    │
│  │   Simple task:    256-512 bytes                             │    │
│  │   Complex task:   1024-2048 bytes                           │    │
│  │   With printf:    2048+ bytes (printf uses lots of stack!)  │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Stack Overflow Detection

```
┌─────────────────────────────────────────────────────────────────────┐
│                    STACK OVERFLOW PROTECTION                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  METHOD 1: Canary Value                                             │
│                                                                      │
│  Stack Top ──▶ ┌──────────────────┐                                 │
│                │  Local vars      │                                 │
│                │  ...             │                                 │
│                │  ...             │                                 │
│                │                  │  ← SP grows down                │
│                ├──────────────────┤                                 │
│                │  CANARY = 0xDEAD │  ← Check periodically           │
│  Stack Base ──▶ └──────────────────┘                                 │
│                                                                      │
│  If CANARY != 0xDEAD → Stack overflow detected!                     │
│                                                                      │
│                                                                      │
│  METHOD 2: MPU (Memory Protection Unit)                             │
│                                                                      │
│  Stack Top ──▶ ┌──────────────────┐                                 │
│                │  Task Stack      │  ← Read/Write allowed           │
│                │  ...             │                                 │
│                ├──────────────────┤                                 │
│                │  Guard Region    │  ← No access (triggers fault)   │
│  Stack Base ──▶ └──────────────────┘                                 │
│                                                                      │
│  MPU triggers HardFault on guard access → Immediate detection       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Interrupt Handling in RTOS

```
┌─────────────────────────────────────────────────────────────────────┐
│                    INTERRUPT HANDLING                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  PRIORITY LEVELS (ARM Cortex-M):                                    │
│                                                                      │
│  Higher Priority                                                     │
│       ▲                                                              │
│       │    ┌────────────────────────────────────────┐               │
│       │    │  HardFault, NMI        (non-maskable)  │               │
│       │    ├────────────────────────────────────────┤               │
│       │    │  SysTick               (kernel tick)   │               │
│       │    ├────────────────────────────────────────┤               │
│       │    │  Peripheral IRQs       (hardware)      │               │
│       │    ├────────────────────────────────────────┤               │
│       │    │  PendSV                (context switch)│               │
│       │    ├────────────────────────────────────────┤               │
│       │    │  TASKS                 (user code)     │               │
│       ▼    └────────────────────────────────────────┘               │
│  Lower Priority                                                      │
│                                                                      │
│                                                                      │
│  ISR BEST PRACTICES:                                                 │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                                                             │    │
│  │  void UART_IRQHandler(void) {                               │    │
│  │      // 1. Read data from peripheral                        │    │
│  │      uint8_t data = UART->DR;                               │    │
│  │                                                             │    │
│  │      // 2. Post to queue (deferred processing)              │    │
│  │      BaseType_t xHigherPriorityWoken = pdFALSE;             │    │
│  │      xQueueSendFromISR(rx_queue, &data,                     │    │
│  │                        &xHigherPriorityWoken);              │    │
│  │                                                             │    │
│  │      // 3. Request context switch if needed                 │    │
│  │      portYIELD_FROM_ISR(xHigherPriorityWoken);              │    │
│  │  }                                                          │    │
│  │                                                             │    │
│  │  // Separate task handles data processing                   │    │
│  │  void uart_task(void *pvParameters) {                       │    │
│  │      uint8_t data;                                          │    │
│  │      while (1) {                                            │    │
│  │          xQueueReceive(rx_queue, &data, portMAX_DELAY);     │    │
│  │          process_data(data);  // Complex processing here    │    │
│  │      }                                                      │    │
│  │  }                                                          │    │
│  │                                                             │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  KEY RULES:                                                          │
│  • Keep ISR short (defer work to tasks)                             │
│  • Use "FromISR" versions of RTOS APIs                              │
│  • Don't block in ISR (no delays, no waiting)                       │
│  • Signal context switch request at end                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Component | Purpose | Key Data |
|-----------|---------|----------|
| **TCB** | Task state storage | Stack pointer, priority, state, wait object |
| **Ready Queue** | List of runnable tasks | Linked list per priority level |
| **Scheduler** | Selects next task | Highest priority ready task wins |
| **Context Switch** | Save/restore CPU state | R0-R15, PSR via PendSV |
| **SysTick** | Periodic timing | 1ms typical, drives delays/timeslicing |
| **MSP** | Kernel/ISR stack | Separate from task stacks |
| **PSP** | Task stack | Each task has dedicated stack |

---

## Quick Revision Questions

1. **What information is stored in a TCB? Why is each field necessary?**

2. **Draw the task state diagram and explain the transitions from RUNNING to BLOCKED and back to READY.**

3. **Why does ARM Cortex-M use PendSV for context switching instead of a regular interrupt? What priority should PendSV have?**

4. **A task has a stack of 512 bytes but crashes with stack overflow. What could cause this, and how would you detect it?**

5. **Explain why ISRs must use special "FromISR" versions of RTOS API functions. What happens if you call the regular version from an ISR?**

6. **Calculate the CPU overhead if the SysTick ISR takes 2μs to execute and the tick rate is 1kHz. What if the rate increases to 10kHz?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Chapter 4.1: Real-Time Concepts](01-real-time-concepts.md) | [Unit Index](README.md) | [Chapter 4.3: Task Management](03-task-management.md) |

---
