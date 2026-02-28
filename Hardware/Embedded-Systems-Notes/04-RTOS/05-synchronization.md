# Chapter 4.5: Synchronization

## Introduction

When multiple tasks share resources or need to coordinate their execution, synchronization primitives are essential. Without proper synchronization, race conditions, data corruption, and deadlocks can occur.

---

## The Critical Section Problem

```
┌─────────────────────────────────────────────────────────────────────┐
│                    RACE CONDITION EXAMPLE                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Shared variable: count = 5                                          │
│                                                                      │
│  Task A: count++         Task B: count++                            │
│                                                                      │
│  EXPECTED: count = 7                                                 │
│                                                                      │
│  RACE CONDITION (what can happen):                                  │
│                                                                      │
│  Time   Task A              Task B              count                │
│  ────   ──────              ──────              ─────                │
│   t0    Read count (5)                          5                    │
│   t1                        Read count (5)      5                    │
│   t2    Compute 5+1=6                           5                    │
│   t3                        Compute 5+1=6       5                    │
│   t4    Write count=6                           6                    │
│   t5                        Write count=6       6  ← WRONG!          │
│                                                                      │
│  RESULT: count = 6 (should be 7)                                    │
│                                                                      │
│  This happens because count++ is NOT atomic:                        │
│    1. LOAD R0, [count]    // Read                                   │
│    2. ADD R0, R0, #1      // Increment                              │
│    3. STR R0, [count]     // Write                                  │
│                                                                      │
│  Preemption between these steps causes the race condition!          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Critical Sections

A critical section is code that accesses shared resources and must be protected.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CRITICAL SECTION PROTECTION                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  REQUIREMENTS FOR CORRECT CRITICAL SECTIONS:                        │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ 1. MUTUAL EXCLUSION                                         │    │
│  │    Only one task can be in the critical section at a time   │    │
│  │                                                             │    │
│  │ 2. PROGRESS                                                 │    │
│  │    If no task is in the critical section and tasks want    │    │
│  │    to enter, one must be allowed to enter                  │    │
│  │                                                             │    │
│  │ 3. BOUNDED WAITING                                          │    │
│  │    A task waiting to enter will eventually be allowed       │    │
│  │    (no starvation)                                          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  STRUCTURE:                                                          │
│                                                                      │
│  ┌───────────────────────────────────────────────────────┐          │
│  │  // Non-critical section (can be preempted)           │          │
│  │                                                       │          │
│  │  ENTER_CRITICAL();  ◀── Acquire lock                 │          │
│  │  ┌─────────────────────────────────────────────────┐ │          │
│  │  │  // Critical section                            │ │          │
│  │  │  shared_data++;  // Safe now                    │ │          │
│  │  │  update_shared_buffer();                        │ │          │
│  │  └─────────────────────────────────────────────────┘ │          │
│  │  EXIT_CRITICAL();   ◀── Release lock                 │          │
│  │                                                       │          │
│  │  // Non-critical section                              │          │
│  └───────────────────────────────────────────────────────┘          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Critical Section Methods

```c
// Method 1: Disable interrupts (simple but blocks everything)
void critical_with_interrupt_disable(void) {
    uint32_t primask;
    
    primask = __get_PRIMASK();  // Save interrupt state
    __disable_irq();            // Disable all interrupts
    
    // Critical section - nothing can preempt
    shared_count++;
    
    __set_PRIMASK(primask);     // Restore interrupt state
}

// Method 2: RTOS critical section (suspends scheduler)
void critical_with_scheduler(void) {
    taskENTER_CRITICAL();
    
    // Critical section - tasks can't preempt
    // (ISRs above configMAX_SYSCALL_INTERRUPT_PRIORITY can still run)
    shared_count++;
    
    taskEXIT_CRITICAL();
}

// Method 3: Suspend scheduler (tasks blocked, ISRs run normally)
void critical_with_suspend(void) {
    vTaskSuspendAll();
    
    // No task switches, but ISRs run
    shared_count++;
    
    xTaskResumeAll();
}
```

---

## Semaphores

Semaphores are counting synchronization primitives.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SEMAPHORE CONCEPT                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  A semaphore is a non-negative integer with two atomic operations:  │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  WAIT (P, pend, take):                                      │    │
│  │    if (count > 0) {                                         │    │
│  │        count--;           // Acquire                        │    │
│  │        // Continue        // Got the semaphore              │    │
│  │    } else {                                                 │    │
│  │        // Block           // Wait for someone to signal     │    │
│  │    }                                                        │    │
│  ├─────────────────────────────────────────────────────────────┤    │
│  │  SIGNAL (V, post, give):                                    │    │
│  │    count++;               // Release                        │    │
│  │    // Wake waiting task (if any)                           │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  TYPES:                                                              │
│                                                                      │
│  ┌─────────────────┬───────────────────────────────────────────┐    │
│  │ Type            │ Description                               │    │
│  ├─────────────────┼───────────────────────────────────────────┤    │
│  │ Binary (0/1)    │ Mutual exclusion, signaling               │    │
│  │ Counting (0-N)  │ Resource pool, producer-consumer          │    │
│  └─────────────────┴───────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Binary Semaphore for Signaling

```
┌─────────────────────────────────────────────────────────────────────┐
│                    BINARY SEMAPHORE (SIGNALING)                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Use case: ISR signals task that data is ready                      │
│                                                                      │
│  ISR                          Task                                   │
│   │                            │                                     │
│   │                            │ xSemaphoreTake(sem)                │
│   │                            │ (blocks, count=0)                  │
│   │                            ▼                                     │
│   │  Data arrives         ┌─────────┐                               │
│   │                       │ BLOCKED │                               │
│   ▼                       │ waiting │                               │
│  xSemaphoreGiveFromISR()  └────┬────┘                               │
│  (count=0→1)                   │                                     │
│   │                            │ Wakes up                           │
│   │                            ▼                                     │
│   │                       ┌─────────┐                               │
│   │                       │ RUNNING │                               │
│   │                       │ process │                               │
│   │                       │  data   │                               │
│   │                       └─────────┘                               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

```c
SemaphoreHandle_t data_ready_sem;

void setup(void) {
    // Create binary semaphore (initially empty)
    data_ready_sem = xSemaphoreCreateBinary();
}

void UART_IRQHandler(void) {
    uint8_t data = UART->DR;
    buffer_write(data);
    
    BaseType_t woken = pdFALSE;
    xSemaphoreGiveFromISR(data_ready_sem, &woken);
    portYIELD_FROM_ISR(woken);
}

void data_processing_task(void *pv) {
    while (1) {
        // Wait for ISR to signal data ready
        xSemaphoreTake(data_ready_sem, portMAX_DELAY);
        
        // Process all available data
        while (buffer_has_data()) {
            process(buffer_read());
        }
    }
}
```

### Counting Semaphore for Resource Pool

```
┌─────────────────────────────────────────────────────────────────────┐
│                    COUNTING SEMAPHORE (RESOURCE POOL)                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Use case: Manage pool of 3 identical resources (e.g., DMA channels)│
│                                                                      │
│  Semaphore count = number of available resources                    │
│                                                                      │
│  Initial state: count = 3 (all resources free)                      │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │ Resource Pool                                                 │  │
│  │  ┌─────┐  ┌─────┐  ┌─────┐                                   │  │
│  │  │ R0  │  │ R1  │  │ R2  │   count = 3                       │  │
│  │  │FREE │  │FREE │  │FREE │                                   │  │
│  │  └─────┘  └─────┘  └─────┘                                   │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  Task A takes: count = 2                                            │
│  Task B takes: count = 1                                            │
│  Task C takes: count = 0                                            │
│  Task D tries to take: BLOCKS (no resources)                        │
│  Task A gives back: count = 1, Task D unblocks                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

```c
#define NUM_DMA_CHANNELS 3
SemaphoreHandle_t dma_sem;

void setup(void) {
    // Create counting semaphore: max=3, initial=3
    dma_sem = xSemaphoreCreateCounting(NUM_DMA_CHANNELS, NUM_DMA_CHANNELS);
}

int acquire_dma_channel(void) {
    // Wait for available channel (blocks if all in use)
    xSemaphoreTake(dma_sem, portMAX_DELAY);
    return allocate_channel();  // Returns channel 0, 1, or 2
}

void release_dma_channel(int channel) {
    free_channel(channel);
    xSemaphoreGive(dma_sem);  // Increment count, wake waiters
}
```

---

## Mutexes

A mutex (mutual exclusion) is a binary semaphore with ownership.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MUTEX vs BINARY SEMAPHORE                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────────┬─────────────────────────────────────────┐   │
│  │ Feature            │ Binary Semaphore │ Mutex               │   │
│  ├────────────────────┼──────────────────┼─────────────────────┤   │
│  │ Ownership          │ No               │ Yes (task-owned)    │   │
│  │ Who can release?   │ Any task         │ Only owner          │   │
│  │ Priority inherit.  │ No               │ Yes (optional)      │   │
│  │ Recursive locking  │ No               │ Yes (optional)      │   │
│  │ Use case           │ Signaling        │ Mutual exclusion    │   │
│  └────────────────────┴──────────────────┴─────────────────────┘   │
│                                                                      │
│  MUTEX OWNERSHIP:                                                    │
│                                                                      │
│  Task A                        Task B                                │
│    │                             │                                   │
│    │ xSemaphoreTake(mutex)       │                                   │
│    │ (A owns mutex)              │                                   │
│    ▼                             │                                   │
│  ┌────────────┐                  │                                   │
│  │ In critical│                  │ xSemaphoreTake(mutex)            │
│  │  section   │                  │ (blocks - A owns it)             │
│  └────────────┘                  ▼                                   │
│    │                        ┌─────────┐                             │
│    │                        │ BLOCKED │                             │
│    │ xSemaphoreGive(mutex)  │ waiting │                             │
│    │ (A releases)           └────┬────┘                             │
│    │                             │ B wakes, becomes owner            │
│    │                             ▼                                   │
│    │                        ┌────────────┐                          │
│    │                        │ In critical│                          │
│    │                        │  section   │                          │
│    │                        └────────────┘                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Mutex Example

```c
SemaphoreHandle_t uart_mutex;

void setup(void) {
    uart_mutex = xSemaphoreCreateMutex();
}

void safe_uart_print(const char *msg) {
    // Only one task can print at a time
    xSemaphoreTake(uart_mutex, portMAX_DELAY);
    
    // Critical section: exclusive UART access
    uart_send_string(msg);
    
    xSemaphoreGive(uart_mutex);
}

// Task A
void task_a(void *pv) {
    while (1) {
        safe_uart_print("Task A: Hello\n");
        vTaskDelay(100);
    }
}

// Task B
void task_b(void *pv) {
    while (1) {
        safe_uart_print("Task B: World\n");
        vTaskDelay(150);
    }
}

// Output is never interleaved thanks to mutex
```

---

## Priority Inversion and Inheritance

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PRIORITY INVERSION PROBLEM                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Tasks: High (H), Medium (M), Low (L)                               │
│  L holds a mutex that H needs                                       │
│                                                                      │
│  WITHOUT Priority Inheritance:                                      │
│                                                                      │
│  H: ──────────────BLOCKED──────────────────▓▓▓▓▓                    │
│  M: ─────────▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓──────────────                    │
│  L: ▓▓▓──────────────────────────────▓▓────────                     │
│          │                           │                               │
│      ─────────────────────────────────▶ Time                        │
│      t0  t1                         t2  t3                          │
│                                                                      │
│  t0: L takes mutex, starts critical section                        │
│  t1: H becomes ready, preempts L                                    │
│      H tries mutex, BLOCKED (L owns it)                             │
│      M becomes ready, runs (higher than L!)                         │
│  t1-t2: M runs for a LONG time                                      │
│         H is blocked waiting for L                                  │
│         L can't run because M is running!                           │
│  t2: M finishes, L runs, releases mutex                             │
│  t3: H finally runs                                                 │
│                                                                      │
│  PROBLEM: High-priority H was blocked by medium-priority M!        │
│           This is "unbounded" priority inversion.                   │
│                                                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  WITH Priority Inheritance:                                         │
│                                                                      │
│  H: ──────BLOCKED───▓▓▓▓▓                                           │
│  M: ─────────────────────▓▓▓▓▓▓▓▓                                   │
│  L: ▓▓▓───▓▓▓▓▓▓▓▓▓▓─────────────────                               │
│          │   │                                                       │
│      ─────────────────────────────────▶ Time                        │
│      t0  t1  t2     t3                                              │
│                                                                      │
│  t0: L takes mutex                                                  │
│  t1: H becomes ready, tries mutex, BLOCKED                          │
│      L INHERITS H's priority temporarily!                           │
│  t1-t2: L runs at high priority (M can't preempt)                   │
│  t2: L releases mutex, priority restored to low                     │
│      H runs immediately                                              │
│  t3: H finishes, M runs                                             │
│                                                                      │
│  SOLUTION: L is "boosted" so it finishes quickly                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Priority Inheritance in FreeRTOS

```c
// Mutexes automatically support priority inheritance
SemaphoreHandle_t mutex = xSemaphoreCreateMutex();

// FreeRTOS handles the priority boost automatically:
// - When high-priority task blocks on mutex
// - Owner's priority is raised to match
// - When mutex released, priority restored
```

---

## Deadlock

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DEADLOCK CONDITION                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Deadlock occurs when tasks are waiting for each other forever:     │
│                                                                      │
│  Task A                         Task B                               │
│    │                              │                                  │
│    │ Take(mutex1) ✓               │ Take(mutex2) ✓                  │
│    │                              │                                  │
│    ▼                              ▼                                  │
│  ┌──────────┐                 ┌──────────┐                          │
│  │ Holds    │                 │ Holds    │                          │
│  │ mutex1   │                 │ mutex2   │                          │
│  └──────────┘                 └──────────┘                          │
│    │                              │                                  │
│    │ Take(mutex2)                 │ Take(mutex1)                    │
│    │ BLOCKED!                     │ BLOCKED!                        │
│    ▼                              ▼                                  │
│    ┌────────────────────────────────┐                               │
│    │         DEADLOCK!              │                               │
│    │  A waits for B to release 2   │                               │
│    │  B waits for A to release 1   │                               │
│    │  Neither can proceed          │                               │
│    └────────────────────────────────┘                               │
│                                                                      │
│  FOUR CONDITIONS FOR DEADLOCK (all must be true):                   │
│  1. Mutual exclusion: Resources are non-sharable                   │
│  2. Hold and wait: Task holds resource while waiting for another   │
│  3. No preemption: Resources can't be forcibly taken               │
│  4. Circular wait: A→B→C→...→A dependency cycle                    │
│                                                                      │
│  PREVENTION STRATEGIES:                                              │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ • Always acquire locks in same order (break circular wait) │    │
│  │ • Use timeout on lock acquisition                           │    │
│  │ • Acquire all needed locks atomically                       │    │
│  │ • Use lock hierarchies                                      │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Deadlock Prevention

```c
// WRONG: Different order leads to deadlock
void task_a(void) {
    xSemaphoreTake(mutex1, portMAX_DELAY);
    xSemaphoreTake(mutex2, portMAX_DELAY);  // May deadlock!
    // ...
    xSemaphoreGive(mutex2);
    xSemaphoreGive(mutex1);
}

void task_b(void) {
    xSemaphoreTake(mutex2, portMAX_DELAY);
    xSemaphoreTake(mutex1, portMAX_DELAY);  // Deadlock risk!
    // ...
    xSemaphoreGive(mutex1);
    xSemaphoreGive(mutex2);
}

// CORRECT: Same order prevents deadlock
void task_a(void) {
    xSemaphoreTake(mutex1, portMAX_DELAY);  // Always 1 first
    xSemaphoreTake(mutex2, portMAX_DELAY);
    // ...
    xSemaphoreGive(mutex2);
    xSemaphoreGive(mutex1);
}

void task_b(void) {
    xSemaphoreTake(mutex1, portMAX_DELAY);  // Always 1 first
    xSemaphoreTake(mutex2, portMAX_DELAY);
    // ...
    xSemaphoreGive(mutex2);
    xSemaphoreGive(mutex1);
}

// ALTERNATIVE: Use timeout to detect and recover
void task_with_timeout(void) {
    if (xSemaphoreTake(mutex1, pdMS_TO_TICKS(100)) == pdTRUE) {
        if (xSemaphoreTake(mutex2, pdMS_TO_TICKS(100)) == pdTRUE) {
            // Got both, proceed
            xSemaphoreGive(mutex2);
        }
        xSemaphoreGive(mutex1);
    } else {
        // Timeout - handle gracefully
    }
}
```

---

## Recursive Mutex

```
┌─────────────────────────────────────────────────────────────────────┐
│                    RECURSIVE MUTEX                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  A recursive mutex allows the same task to lock it multiple times:  │
│                                                                      │
│  void function_a(void) {                                             │
│      xSemaphoreTakeRecursive(mutex, portMAX_DELAY);  // count = 1   │
│      // ...                                                          │
│      function_b();  // Calls another function that also locks       │
│      // ...                                                          │
│      xSemaphoreGiveRecursive(mutex);                 // count = 0   │
│  }                                                                   │
│                                                                      │
│  void function_b(void) {                                             │
│      xSemaphoreTakeRecursive(mutex, portMAX_DELAY);  // count = 2   │
│      // ...                                                          │
│      xSemaphoreGiveRecursive(mutex);                 // count = 1   │
│  }                                                                   │
│                                                                      │
│  LOCK COUNT TRACKING:                                                │
│                                                                      │
│  function_a() called                                                 │
│       │ Take → count = 1                                            │
│       ▼                                                              │
│  function_b() called                                                 │
│       │ Take → count = 2                                            │
│       │ Give → count = 1                                            │
│       ▼                                                              │
│  Back in function_a()                                                │
│       │ Give → count = 0 (mutex now free)                           │
│       ▼                                                              │
│  Done                                                                │
│                                                                      │
│  NOTE: Give must be called same number of times as Take!            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Primitive | Purpose | Key Property |
|-----------|---------|--------------|
| **Critical Section** | Short exclusive access | Disables preemption/interrupts |
| **Binary Semaphore** | Signaling between tasks/ISR | No ownership, give from anywhere |
| **Counting Semaphore** | Resource pool management | Counts available resources |
| **Mutex** | Mutual exclusion | Ownership, priority inheritance |
| **Recursive Mutex** | Nested locking | Same owner can lock multiple times |

---

## Quick Revision Questions

1. **What is a race condition? Give an example with shared variable increment and explain why it fails.**

2. **Explain the difference between a binary semaphore and a mutex. When would you use each?**

3. **What is priority inversion? How does priority inheritance solve it?**

4. **Two tasks need mutexes A and B. Task 1 acquires A then B, Task 2 acquires B then A. What problem can occur? How to prevent it?**

5. **An ISR needs to signal a task that data is ready. Should it use a mutex or a binary semaphore? Why?**

6. **What is a recursive mutex? When is it necessary?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [Chapter 4.4: Scheduling Algorithms](04-scheduling-algorithms.md) | [Unit Index](README.md) | [Chapter 4.6: Inter-Task Communication](06-inter-task-communication.md) |

---
