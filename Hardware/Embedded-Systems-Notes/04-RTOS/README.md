# Unit 4: Real-Time Operating Systems (RTOS)

## Unit Overview

Real-Time Operating Systems (RTOS) provide deterministic task scheduling, resource management, and inter-task communication for embedded applications. This unit covers RTOS concepts, task management, synchronization primitives, and practical implementation using popular RTOS platforms.

---

## Learning Objectives

After completing this unit, you will be able to:

- Understand real-time system requirements and constraints
- Design and implement multi-tasking applications using RTOS
- Use synchronization mechanisms (semaphores, mutexes, queues)
- Analyze and prevent priority inversion problems
- Choose appropriate RTOS for your application

---

## Unit Contents

| Chapter | Topic | Key Concepts |
|---------|-------|--------------|
| 4.1 | [Real-Time Concepts](01-real-time-concepts.md) | Hard/soft real-time, deadlines, latency |
| 4.2 | [RTOS Architecture](02-rtos-architecture.md) | Kernel, scheduler, context switching |
| 4.3 | [Task Management](03-task-management.md) | Task states, creation, priorities |
| 4.4 | [Scheduling Algorithms](04-scheduling-algorithms.md) | Preemptive, round-robin, rate monotonic |
| 4.5 | [Synchronization](05-synchronization.md) | Semaphores, mutexes, critical sections |
| 4.6 | [Inter-Task Communication](06-inter-task-communication.md) | Queues, mailboxes, event flags |

---

## RTOS Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    RTOS ARCHITECTURE                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    APPLICATION TASKS                         │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │    │
│  │  │ Task 1  │  │ Task 2  │  │ Task 3  │  │ Task n  │        │    │
│  │  │ (High)  │  │ (Med)   │  │ (Low)   │  │  ...    │        │    │
│  │  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘        │    │
│  │       │            │            │            │              │    │
│  └───────┼────────────┼────────────┼────────────┼──────────────┘    │
│          │            │            │            │                   │
│          └────────────┴─────┬──────┴────────────┘                   │
│                             │                                        │
│  ┌──────────────────────────▼──────────────────────────────────┐    │
│  │                    RTOS KERNEL                               │    │
│  │  ┌──────────────────────────────────────────────────────┐   │    │
│  │  │  SCHEDULER                                           │   │    │
│  │  │  • Priority-based preemptive scheduling              │   │    │
│  │  │  • Context switching                                 │   │    │
│  │  │  • Ready queue management                            │   │    │
│  │  └──────────────────────────────────────────────────────┘   │    │
│  │                                                              │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │    │
│  │  │   Timer     │  │  Memory     │  │  Synchronization    │ │    │
│  │  │  Services   │  │  Management │  │  Primitives         │ │    │
│  │  │  • Delays   │  │  • Stacks   │  │  • Semaphores       │ │    │
│  │  │  • Timeouts │  │  • Pools    │  │  • Mutexes          │ │    │
│  │  │  • Periodic │  │  • Heap     │  │  • Queues           │ │    │
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘ │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    HARDWARE                                  │    │
│  │  • SysTick Timer (scheduler tick)                           │    │
│  │  • NVIC (interrupt controller)                              │    │
│  │  • CPU (context switching support)                          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Popular RTOS Platforms

```
┌─────────────────────────────────────────────────────────────────────┐
│                    RTOS COMPARISON                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ RTOS      │ License    │ RAM   │ Features     │ Use Case   │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ FreeRTOS  │ MIT        │ <1KB  │ Basic, widely│ General    │    │
│  │           │            │       │ supported    │ purpose    │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ Zephyr    │ Apache 2.0 │ 2KB+  │ Rich stack,  │ IoT, BLE   │    │
│  │           │            │       │ networking   │            │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ ThreadX   │ MIT (Azure)│ <2KB  │ Certified,   │ Safety-    │    │
│  │           │            │       │ deterministic│ critical   │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ RTEMS     │ BSD        │ 10KB+ │ POSIX-like,  │ Aerospace  │    │
│  │           │            │       │ SMP support  │            │    │
│  │─────────────────────────────────────────────────────────────│    │
│  │ μC/OS-III │ Commercial │ 2KB+  │ Certified,   │ Industrial │    │
│  │           │            │       │ full-featured│            │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Key Concepts Preview

| Concept | Description |
|---------|-------------|
| **Task/Thread** | Independent unit of execution with own stack |
| **Scheduler** | Kernel component that selects which task to run |
| **Context Switch** | Saving/restoring task state when switching |
| **Priority** | Numeric value determining task importance |
| **Semaphore** | Counter for resource synchronization |
| **Mutex** | Binary lock with ownership for mutual exclusion |
| **Queue** | FIFO buffer for passing data between tasks |
| **Deadline** | Time by which task must complete |

---

## Navigation

| Previous Unit | Course Index | Next Unit |
|---------------|--------------|-----------|
| [Unit 3: Embedded Software](../03-Embedded-Software/README.md) | [Main Index](../README.md) | [Unit 5: Communication Protocols](../05-Communication-Protocols/README.md) |

---
