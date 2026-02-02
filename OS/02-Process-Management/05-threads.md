# Chapter 2.5: Threads

## Overview

Threads are the fundamental unit of CPU utilization within a process. Understanding threads is essential for modern programming and operating system concepts, as most applications today are multithreaded.

---

## What is a Thread?

### Definition
A **Thread** (also called a Lightweight Process) is the smallest sequence of programmed instructions that can be managed independently by a scheduler. It is a basic unit of CPU utilization.

### Thread vs Process

```
┌─────────────────────────────────────────────────────────────────────┐
│                        PROCESS                                       │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                    SHARED RESOURCES                            │  │
│  │  • Code Section (Text)                                        │  │
│  │  • Data Section (Global Variables)                            │  │
│  │  • Open Files                                                 │  │
│  │  • Signals                                                    │  │
│  │  • Heap Memory                                                │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐            │
│  │  Thread 1   │    │  Thread 2   │    │  Thread 3   │            │
│  ├─────────────┤    ├─────────────┤    ├─────────────┤            │
│  │ Thread ID   │    │ Thread ID   │    │ Thread ID   │            │
│  │ Registers   │    │ Registers   │    │ Registers   │            │
│  │ Stack       │    │ Stack       │    │ Stack       │            │
│  │ PC          │    │ PC          │    │ PC          │            │
│  │ State       │    │ State       │    │ State       │            │
│  └─────────────┘    └─────────────┘    └─────────────┘            │
│       ↑                   ↑                   ↑                    │
│       └───────────────────┴───────────────────┘                    │
│                 Each thread has its own                             │
│                 execution context (private)                         │
└─────────────────────────────────────────────────────────────────────┘
```

### Analogy

Think of a **restaurant**:
- **Process** = The entire restaurant (kitchen, dining area, resources)
- **Threads** = Individual cooks working in the same kitchen

All cooks (threads) share the same kitchen equipment (memory, files) but each cook has their own:
- Current task (program counter)
- Cooking notes (registers)
- Personal workspace (stack)

---

## Single-Threaded vs Multi-Threaded

### Single-Threaded Process

```
┌─────────────────────────────────┐
│         SINGLE-THREADED         │
│           PROCESS               │
├─────────────────────────────────┤
│ ┌─────────────────────────────┐ │
│ │         Code                │ │
│ ├─────────────────────────────┤ │
│ │         Data                │ │
│ ├─────────────────────────────┤ │
│ │         Files               │ │
│ ├─────────────────────────────┤ │
│ │       Registers             │ │
│ ├─────────────────────────────┤ │
│ │        Stack                │ │
│ └─────────────────────────────┘ │
│                                 │
│      One thread of control     │
│             │                  │
│             ↓                  │
└─────────────────────────────────┘
```

### Multi-Threaded Process

```
┌──────────────────────────────────────────────────────────────┐
│                    MULTI-THREADED PROCESS                     │
├──────────────────────────────────────────────────────────────┤
│ ┌────────────────────────────────────────────────────────┐   │
│ │              Shared: Code, Data, Files                 │   │
│ └────────────────────────────────────────────────────────┘   │
│                                                              │
│ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐          │
│ │   Thread 1   │ │   Thread 2   │ │   Thread 3   │          │
│ │ ┌──────────┐ │ │ ┌──────────┐ │ │ ┌──────────┐ │          │
│ │ │Registers │ │ │ │Registers │ │ │ │Registers │ │          │
│ │ ├──────────┤ │ │ ├──────────┤ │ │ ├──────────┤ │          │
│ │ │  Stack   │ │ │ │  Stack   │ │ │ │  Stack   │ │          │
│ │ └──────────┘ │ │ └──────────┘ │ │ └──────────┘ │          │
│ └──────────────┘ └──────────────┘ └──────────────┘          │
│        │                │                │                   │
│        ↓                ↓                ↓                   │
│   Concurrent Execution Paths                                 │
└──────────────────────────────────────────────────────────────┘
```

---

## Thread Components

### What Threads Share (Within a Process)

| Shared Resource | Description |
|-----------------|-------------|
| **Address Space** | Same virtual memory |
| **Code Section** | Same program instructions |
| **Data Section** | Global variables |
| **Open Files** | File descriptors |
| **Signals** | Signal handlers |
| **Heap** | Dynamic memory |

### What Each Thread Has (Private)

| Private Resource | Description |
|------------------|-------------|
| **Thread ID** | Unique identifier |
| **Program Counter** | Current instruction |
| **Register Set** | CPU register values |
| **Stack** | Local variables, function calls |
| **State** | Running, ready, blocked |

---

## Why Use Threads?

### 1. Responsiveness

```
Example: Web Browser

Without Threads:              With Threads:
┌─────────────────────┐      ┌─────────────────────┐
│ Loading image...    │      │ Thread 1: Display UI│
│ Page frozen until   │      │ Thread 2: Load image│
│ image loads         │      │ Thread 3: Run script│
│                     │      │                     │
│ User frustrated!    │      │ UI always responsive│
└─────────────────────┘      └─────────────────────┘
```

### 2. Resource Sharing

Threads share memory and resources by default, making communication efficient.

```
Process Communication:              Thread Communication:
┌─────────┐    ┌─────────┐         ┌─────────────────────────┐
│Process A│    │Process B│         │        Process          │
│  Data   │    │  Data   │         │ ┌───────┐  ┌───────┐   │
└────┬────┘    └────┬────┘         │ │Thread1│  │Thread2│   │
     │              │              │ └───┬───┘  └───┬───┘   │
     └──────┬───────┘              │     └─Shared──┘        │
            │                      │       Data              │
     IPC (pipes, shared            │                        │
     memory, messages)             └────────────────────────┘
     
     Complex, overhead              Simple, direct access
```

### 3. Economy

Creating threads is much faster and cheaper than creating processes.

```
Resource Comparison:
┌───────────────────────────────────────────────────────────┐
│                                                           │
│  Process Creation:                                        │
│  • Allocate new address space          ~30× slower       │
│  • Copy page tables                                       │
│  • Create PCB                                             │
│  • Allocate resources                                     │
│                                                           │
│  Thread Creation:                                         │
│  • Allocate stack                      Much faster        │
│  • Create thread control block                            │
│  • Share parent's resources                               │
│                                                           │
└───────────────────────────────────────────────────────────┘

Typical ratios:
• Thread creation: 10-100× faster than process creation
• Context switch: 5-10× faster between threads
```

### 4. Scalability (Multiprocessor Utilization)

```
Single-threaded on 4-core CPU:      Multi-threaded on 4-core CPU:
┌───────┬───────┬───────┬───────┐   ┌───────┬───────┬───────┬───────┐
│ Core 0│ Core 1│ Core 2│ Core 3│   │ Core 0│ Core 1│ Core 2│ Core 3│
│ ████  │ Idle  │ Idle  │ Idle  │   │Thread1│Thread2│Thread3│Thread4│
└───────┴───────┴───────┴───────┘   └───────┴───────┴───────┴───────┘
    25% utilization                     100% utilization (ideal)
```

---

## Types of Threads

### 1. User-Level Threads (ULT)

Thread management done by user-level thread library. Kernel is unaware of threads.

```
┌────────────────────────────────────────────────────────────────┐
│                         USER SPACE                              │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    Thread Library                         │  │
│  │  ┌─────────┐    ┌─────────┐    ┌─────────┐              │  │
│  │  │ Thread 1│    │ Thread 2│    │ Thread 3│              │  │
│  │  └────┬────┘    └────┬────┘    └────┬────┘              │  │
│  │       └──────────────┼──────────────┘                    │  │
│  │                      ↓                                    │  │
│  │              Thread Scheduler                             │  │
│  │              (User Level)                                 │  │
│  └──────────────────────┼───────────────────────────────────┘  │
├─────────────────────────┼──────────────────────────────────────┤
│                         ↓           KERNEL SPACE               │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                     Kernel                                │  │
│  │                                                          │  │
│  │    Sees only ONE process (not individual threads)        │  │
│  │                                                          │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────┘
```

**Advantages:**
| Advantage | Explanation |
|-----------|-------------|
| Fast switching | No kernel mode switch needed |
| Portable | Can run on any OS |
| Flexible scheduling | Application can choose policy |

**Disadvantages:**
| Disadvantage | Explanation |
|--------------|-------------|
| Blocking problem | One thread blocks → entire process blocks |
| No true parallelism | Kernel schedules process, not threads |
| Can't use multiple CPUs | Kernel sees one entity |

### 2. Kernel-Level Threads (KLT)

Thread management done by the kernel. OS is aware of all threads.

```
┌────────────────────────────────────────────────────────────────┐
│                         USER SPACE                              │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    Application                            │  │
│  │  ┌─────────┐    ┌─────────┐    ┌─────────┐              │  │
│  │  │ Thread 1│    │ Thread 2│    │ Thread 3│              │  │
│  │  └────┬────┘    └────┬────┘    └────┬────┘              │  │
│  └───────┼──────────────┼──────────────┼────────────────────┘  │
├──────────┼──────────────┼──────────────┼───────────────────────┤
│          ↓              ↓              ↓     KERNEL SPACE      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                     Kernel                                │  │
│  │  ┌─────────┐    ┌─────────┐    ┌─────────┐              │  │
│  │  │ KThread1│    │ KThread2│    │ KThread3│              │  │
│  │  └────┬────┘    └────┬────┘    └────┬────┘              │  │
│  │       └──────────────┼──────────────┘                    │  │
│  │                      ↓                                    │  │
│  │              Kernel Scheduler                             │  │
│  │     (Schedules individual threads to CPUs)                │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────┘
```

**Advantages:**
| Advantage | Explanation |
|-----------|-------------|
| True parallelism | Can run on multiple CPUs |
| No blocking issue | One thread blocks, others continue |
| Better for multicore | Full CPU utilization |

**Disadvantages:**
| Disadvantage | Explanation |
|--------------|-------------|
| Slower switching | Requires kernel mode switch |
| More overhead | Kernel manages thread structures |
| Less portable | Depends on OS support |

### Comparison

| Aspect | User-Level Threads | Kernel-Level Threads |
|--------|-------------------|---------------------|
| **Management** | User library | Kernel |
| **Kernel Aware** | No | Yes |
| **Creation Speed** | Fast | Slower |
| **Context Switch** | Fast (no mode switch) | Slower (mode switch) |
| **Blocking** | Blocks entire process | Blocks only that thread |
| **Multiprocessor** | No true parallelism | True parallelism |
| **Example** | POSIX Pthreads (some) | Windows, Linux threads |

---

## Multithreading Models

How user-level threads are mapped to kernel-level threads.

### 1. Many-to-One Model

Many user threads mapped to one kernel thread.

```
┌─────────────────────────────────────────┐
│              USER SPACE                  │
│  ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐        │
│  │ U │ │ U │ │ U │ │ U │ │ U │        │
│  │ T │ │ T │ │ T │ │ T │ │ T │        │
│  └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘        │
│    └─────┴─────┼─────┴─────┘           │
│                ↓                        │
├────────────────┼────────────────────────┤
│                ↓      KERNEL SPACE      │
│            ┌───────┐                    │
│            │  KT   │ (One kernel thread)│
│            └───────┘                    │
└─────────────────────────────────────────┘

Problem: One thread blocks → all block
         No parallelism on multicore
```

**Examples**: Solaris Green Threads, GNU Portable Threads

### 2. One-to-One Model

Each user thread maps to one kernel thread.

```
┌─────────────────────────────────────────┐
│              USER SPACE                  │
│  ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐        │
│  │ U │ │ U │ │ U │ │ U │ │ U │        │
│  │ T │ │ T │ │ T │ │ T │ │ T │        │
│  └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘        │
│    │     │     │     │     │           │
│    ↓     ↓     ↓     ↓     ↓           │
├────┼─────┼─────┼─────┼─────┼───────────┤
│    ↓     ↓     ↓     ↓     ↓  KERNEL   │
│  ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐        │
│  │KT │ │KT │ │KT │ │KT │ │KT │        │
│  └───┘ └───┘ └───┘ └───┘ └───┘        │
└─────────────────────────────────────────┘

Advantage: True concurrency
Disadvantage: Creating many threads is expensive
```

**Examples**: Windows, Linux, macOS

### 3. Many-to-Many Model

Many user threads mapped to many kernel threads (M:N).

```
┌─────────────────────────────────────────┐
│              USER SPACE                  │
│  ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐  │
│  │ U │ │ U │ │ U │ │ U │ │ U │ │ U │  │
│  │ T │ │ T │ │ T │ │ T │ │ T │ │ T │  │
│  └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘  │
│    └──┬──┴──┬──┴──┬──┴──┬──┴──┬──┘     │
│       │     │     │     │     │        │
│       ↓     ↓     ↓     ↓     ↓        │
├───────┼─────┼─────┼─────┼─────┼────────┤
│       ↓     ↓     ↓     ↓     ↓ KERNEL │
│     ┌───┐ ┌───┐ ┌───┐ ┌───┐           │
│     │KT │ │KT │ │KT │ │KT │           │
│     └───┘ └───┘ └───┘ └───┘           │
└─────────────────────────────────────────┘

6 user threads mapped to 4 kernel threads
Best of both worlds
```

**Examples**: Solaris prior to version 9, Windows with ThreadFiber

### Model Comparison

| Model | Concurrency | Blocking | Complexity |
|-------|-------------|----------|------------|
| Many-to-One | Limited | One blocks all | Low |
| One-to-One | Full | Independent | Medium |
| Many-to-Many | Flexible | Optimized | High |

---

## Thread Libraries

### 1. POSIX Pthreads

Standard API for thread creation and synchronization.

```c
#include <pthread.h>
#include <stdio.h>

void* thread_function(void* arg) {
    int* num = (int*)arg;
    printf("Thread: Received %d\n", *num);
    return NULL;
}

int main() {
    pthread_t thread;
    int value = 42;
    
    // Create thread
    pthread_create(&thread, NULL, thread_function, &value);
    
    // Wait for thread to complete
    pthread_join(thread, NULL);
    
    printf("Main: Thread finished\n");
    return 0;
}
```

### 2. Windows Threads

```c
#include <windows.h>
#include <stdio.h>

DWORD WINAPI ThreadFunc(LPVOID lpParam) {
    int* num = (int*)lpParam;
    printf("Thread: Received %d\n", *num);
    return 0;
}

int main() {
    HANDLE thread;
    int value = 42;
    
    // Create thread
    thread = CreateThread(NULL, 0, ThreadFunc, &value, 0, NULL);
    
    // Wait for thread
    WaitForSingleObject(thread, INFINITE);
    
    CloseHandle(thread);
    return 0;
}
```

### 3. Java Threads

```java
// Method 1: Extending Thread class
class MyThread extends Thread {
    public void run() {
        System.out.println("Thread running");
    }
}

// Method 2: Implementing Runnable interface
class MyRunnable implements Runnable {
    public void run() {
        System.out.println("Runnable running");
    }
}

public class Main {
    public static void main(String[] args) {
        // Using Thread
        MyThread t1 = new MyThread();
        t1.start();
        
        // Using Runnable
        Thread t2 = new Thread(new MyRunnable());
        t2.start();
        
        // Using Lambda (Java 8+)
        Thread t3 = new Thread(() -> {
            System.out.println("Lambda thread running");
        });
        t3.start();
    }
}
```

---

## Threading Issues

### 1. Thread Safety

When multiple threads access shared data, issues can arise:

```
Problem: Race Condition

Thread 1: Read counter (value = 5)
Thread 1: Increment (value = 6)
                                    Thread 2: Read counter (value = 5)
Thread 1: Write counter (counter = 6)
                                    Thread 2: Increment (value = 6)
                                    Thread 2: Write counter (counter = 6)

Expected: counter = 7
Actual: counter = 6 (one increment lost!)
```

### 2. fork() and exec() in Multithreaded Programs

```
Question: When a thread calls fork(), should child 
          have all threads or just the calling thread?

Option 1: Duplicate all threads
          - Complex, may cause issues
          - What if other threads are in critical sections?

Option 2: Duplicate only calling thread
          - Simpler, more common
          - What if forked thread needs other threads' resources?

Most UNIX: Only calling thread is duplicated
If exec() follows fork(): Only calling thread needed anyway
```

### 3. Signal Handling

Where should signals be delivered in multithreaded program?

```
┌──────────────────────────────────────────────────┐
│  Signal Delivery Options:                        │
│                                                  │
│  1. Deliver to thread that caused signal         │
│     (e.g., divide by zero)                       │
│                                                  │
│  2. Deliver to every thread in process           │
│     (e.g., Ctrl+C termination)                   │
│                                                  │
│  3. Deliver to certain threads only              │
│     (based on signal mask)                       │
│                                                  │
│  4. Assign specific thread for all signals       │
│     (signal handling thread)                     │
└──────────────────────────────────────────────────┘
```

### 4. Thread Cancellation

Terminating a thread before it completes.

| Type | Description | Cleanup |
|------|-------------|---------|
| **Asynchronous** | Immediate termination | May leave data inconsistent |
| **Deferred** | Thread checks cancellation points | Allows cleanup |

```c
// Pthread cancellation example
pthread_t thread;
pthread_create(&thread, NULL, worker_function, NULL);

// Later, request cancellation
pthread_cancel(thread);

// In worker_function, check for cancellation
void* worker_function(void* arg) {
    while(1) {
        // Do work
        pthread_testcancel();  // Cancellation point
    }
}
```

### 5. Thread-Local Storage (TLS)

Each thread needs its own copy of certain data.

```
┌────────────────────────────────────────────────────────┐
│                    PROCESS                              │
│  ┌────────────────────────────────────────────────┐    │
│  │           Shared Data (Global)                  │    │
│  │              int shared_var;                    │    │
│  └────────────────────────────────────────────────┘    │
│                                                        │
│  ┌───────────────┐      ┌───────────────┐            │
│  │   Thread 1    │      │   Thread 2    │            │
│  │ ┌───────────┐ │      │ ┌───────────┐ │            │
│  │ │  TLS      │ │      │ │  TLS      │ │            │
│  │ │ errno = 5 │ │      │ │ errno = 9 │ │            │
│  │ └───────────┘ │      │ └───────────┘ │            │
│  └───────────────┘      └───────────────┘            │
│                                                        │
│   Each thread has its own 'errno' value               │
└────────────────────────────────────────────────────────┘
```

---

## Multithreading Examples

### Web Server

```
┌──────────────────────────────────────────────────────────────┐
│                    MULTI-THREADED WEB SERVER                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────┐                                        │
│  │  Main Thread    │                                        │
│  │  (Dispatcher)   │                                        │
│  └────────┬────────┘                                        │
│           │                                                  │
│           │ Accept connections                               │
│           │ Create worker thread for each request           │
│           ↓                                                  │
│  ┌────────────────────────────────────────────────────────┐ │
│  │                  Thread Pool                            │ │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐          │ │
│  │  │Worker 1│ │Worker 2│ │Worker 3│ │Worker 4│          │ │
│  │  │ (busy) │ │ (busy) │ │ (idle) │ │ (idle) │          │ │
│  │  └────────┘ └────────┘ └────────┘ └────────┘          │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  Benefits:                                                   │
│  • Concurrent request handling                               │
│  • Responsive even under load                                │
│  • Efficient resource sharing                                │
└──────────────────────────────────────────────────────────────┘
```

### Word Processor

```
┌──────────────────────────────────────────────────────────────┐
│                    WORD PROCESSOR THREADS                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Thread 1: User Interface                                    │
│  └─→ Handles keyboard, mouse, display                       │
│                                                              │
│  Thread 2: Spell Checker                                     │
│  └─→ Checks spelling in background                          │
│                                                              │
│  Thread 3: Auto-Save                                         │
│  └─→ Periodically saves document                            │
│                                                              │
│  Thread 4: Print Formatter                                   │
│  └─→ Prepares document for printing                         │
│                                                              │
│  All threads share the document data                         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **Thread** | Lightweight unit of execution within a process |
| **Shared** | Code, data, files (within process) |
| **Private** | Stack, registers, PC (per thread) |
| **ULT** | User-level threads (managed by library) |
| **KLT** | Kernel-level threads (managed by OS) |
| **Benefits** | Responsiveness, resource sharing, economy, scalability |

### Thread Models Summary

| Model | User : Kernel | Description |
|-------|---------------|-------------|
| Many-to-One | N : 1 | Many user threads, one kernel thread |
| One-to-One | 1 : 1 | Each user thread has kernel thread |
| Many-to-Many | M : N | M user threads to N kernel threads |

---

## Quick Revision Questions

1. What is the difference between a process and a thread?
2. What resources do threads share within a process?
3. Compare user-level and kernel-level threads.
4. Explain the three multithreading models.
5. What is thread-local storage and why is it needed?

---

[← Previous: CPU Scheduling Algorithms](./04-cpu-scheduling-algorithms.md) | [Next: Inter-Process Communication →](./06-inter-process-communication.md)
