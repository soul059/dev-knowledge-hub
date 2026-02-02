# Chapter 3.5: Monitors

## Overview

Monitors are a high-level synchronization construct that simplifies the complexity of using semaphores. They encapsulate shared data, operations, and synchronization into a single construct, making concurrent programming safer and easier.

---

## Problems with Semaphores

```
┌──────────────────────────────────────────────────────────────────┐
│              WHY WE NEED MONITORS                                 │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Semaphore Issues:                                               │
│                                                                  │
│  1. Easy to make mistakes:                                       │
│     • Forget to call wait() → No mutual exclusion               │
│     • Forget to call signal() → Deadlock                        │
│     • Wrong order of wait/signal → Various bugs                 │
│                                                                  │
│  2. Scattered code:                                              │
│     • wait() and signal() spread throughout program             │
│     • Hard to verify correctness                                │
│                                                                  │
│  3. Low-level:                                                   │
│     • Programmer manages all details                            │
│     • Error-prone                                               │
│                                                                  │
│  Common Mistakes:                                                │
│  ┌──────────────────────────────────────────┐                   │
│  │ signal(mutex);  // Wrong!                │                   │
│  │ // Critical Section                      │                   │
│  │ wait(mutex);    // Wrong order!          │                   │
│  └──────────────────────────────────────────┘                   │
│                                                                  │
│  ┌──────────────────────────────────────────┐                   │
│  │ wait(mutex);                             │                   │
│  │ // Critical Section                      │                   │
│  │ // Forgot signal(mutex)! → Deadlock      │                   │
│  └──────────────────────────────────────────┘                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## What is a Monitor?

### Definition

A **Monitor** is an abstract data type that encapsulates:
- Private shared variables
- Procedures to operate on them
- Automatic mutual exclusion

```
┌──────────────────────────────────────────────────────────────────┐
│                      MONITOR STRUCTURE                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐     │
│  │                    MONITOR                              │     │
│  │  ┌──────────────────────────────────────────────────┐  │     │
│  │  │           Shared Variables                        │  │     │
│  │  │   int count;                                     │  │     │
│  │  │   queue buffer;                                  │  │     │
│  │  └──────────────────────────────────────────────────┘  │     │
│  │                                                        │     │
│  │  ┌──────────────────────────────────────────────────┐  │     │
│  │  │           Condition Variables                     │  │     │
│  │  │   condition notFull, notEmpty;                   │  │     │
│  │  └──────────────────────────────────────────────────┘  │     │
│  │                                                        │     │
│  │  ┌──────────────────────────────────────────────────┐  │     │
│  │  │           Procedures                              │  │     │
│  │  │   procedure insert(item) { ... }                 │  │     │
│  │  │   procedure remove() { ... }                     │  │     │
│  │  └──────────────────────────────────────────────────┘  │     │
│  │                                                        │     │
│  │  ┌──────────────────────────────────────────────────┐  │     │
│  │  │           Initialization Code                     │  │     │
│  │  │   count = 0;                                     │  │     │
│  │  └──────────────────────────────────────────────────┘  │     │
│  └────────────────────────────────────────────────────────┘     │
│                                                                  │
│  Key Property: Only ONE process can be active inside            │
│                the monitor at any time!                         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Key Properties

| Property | Description |
|----------|-------------|
| **Encapsulation** | Shared data hidden inside monitor |
| **Automatic Mutual Exclusion** | Only one process executes monitor code at a time |
| **Entry Queue** | Processes wait here to enter monitor |
| **Condition Variables** | For waiting inside monitor |

---

## Monitor Syntax (Pseudocode)

```
monitor monitor_name {
    // Shared variable declarations
    int shared_data;
    
    // Condition variable declarations
    condition x, y;
    
    // Procedures
    procedure P1(...) {
        ...
    }
    
    procedure P2(...) {
        ...
    }
    
    // Initialization code
    initialization_code {
        shared_data = initial_value;
    }
}
```

---

## Condition Variables

### Purpose

While mutual exclusion is automatic, processes may need to wait for specific conditions inside the monitor. Condition variables provide this capability.

### Operations

| Operation | Syntax | Effect |
|-----------|--------|--------|
| **wait** | `x.wait()` | Suspend the process on condition x |
| **signal** | `x.signal()` | Wake up one process waiting on x |

```
┌──────────────────────────────────────────────────────────────────┐
│                 CONDITION VARIABLE OPERATIONS                     │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  x.wait():                                                       │
│  1. Process releases the monitor (allows others in)             │
│  2. Process is suspended on condition x's queue                 │
│  3. When awakened, process re-acquires monitor                  │
│                                                                  │
│  x.signal():                                                     │
│  1. If no process waiting on x → No effect                     │
│  2. If processes waiting on x → Wake up exactly ONE            │
│                                                                  │
│  Important: signal() does NOT block the signaler                │
│  (unlike semaphore signal which is always non-blocking)         │
│                                                                  │
│           ┌─────────────────────────────┐                       │
│  Entering │        MONITOR              │ Entry Queue           │
│   ───────→│                             │←── [P3][P4][P5]       │
│           │  Active: P1                 │                       │
│           │                             │                       │
│           │  ┌─────────────────────┐    │                       │
│           │  │ Condition x queue   │    │                       │
│           │  │ [P2]                │    │                       │
│           │  └─────────────────────┘    │                       │
│           │                             │                       │
│           └─────────────────────────────┘                       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Signal Semantics: Hoare vs Mesa

When a process signals a condition, who runs next?

### Hoare Semantics (Signal-and-Wait)

```
┌──────────────────────────────────────────────────────────────────┐
│                    HOARE SEMANTICS                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  When P1 signals condition x:                                    │
│  1. P1 immediately suspends                                      │
│  2. Waiting process P2 runs immediately                         │
│  3. When P2 exits/waits, P1 resumes                             │
│                                                                  │
│  Timeline:                                                       │
│  P1: [running] ─── signal(x) ─── [suspended] ─── [resumes]      │
│                        │              ↑              ↑          │
│                        ↓              │              │          │
│  P2: [waiting] ─────────────── [runs] ── [exits/waits]          │
│                                                                  │
│  Advantage: Condition guaranteed true when P2 runs              │
│  Disadvantage: Extra context switch                             │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Mesa Semantics (Signal-and-Continue)

```
┌──────────────────────────────────────────────────────────────────┐
│                    MESA SEMANTICS                                 │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  When P1 signals condition x:                                    │
│  1. P1 continues running                                         │
│  2. P2 is made ready but doesn't run immediately                │
│  3. P2 competes to enter when P1 exits                          │
│                                                                  │
│  Timeline:                                                       │
│  P1: [running] ── signal(x) ── [continues] ── [exits]           │
│                       │                          │              │
│                       ↓                          ↓              │
│  P2: [waiting] ───────────── [ready] ────── [runs]              │
│                                                                  │
│  Consequence: Condition may NOT be true when P2 runs!           │
│               (another process may have changed it)              │
│                                                                  │
│  Solution: Use while loop instead of if                         │
│                                                                  │
│  Hoare:                     Mesa:                                │
│  if (!condition)            while (!condition)                   │
│      x.wait();                  x.wait();                        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Comparison

| Aspect | Hoare | Mesa |
|--------|-------|------|
| **After signal** | Signaler waits | Signaler continues |
| **Condition check** | `if` sufficient | Must use `while` |
| **Context switches** | More | Fewer |
| **Implementation** | More complex | Simpler |
| **Used in** | Theory, some languages | Java, Pthreads, most systems |

---

## Producer-Consumer with Monitor

### Monitor Implementation

```
monitor ProducerConsumer {
    int buffer[N];
    int count = 0;
    int in = 0, out = 0;
    
    condition notFull, notEmpty;
    
    procedure insert(int item) {
        while (count == N)          // Buffer full
            notFull.wait();         // Wait for space
        
        buffer[in] = item;
        in = (in + 1) % N;
        count++;
        
        notEmpty.signal();          // Signal consumer
    }
    
    procedure remove() : int {
        while (count == 0)          // Buffer empty
            notEmpty.wait();        // Wait for item
        
        int item = buffer[out];
        out = (out + 1) % N;
        count--;
        
        notFull.signal();           // Signal producer
        return item;
    }
}

// PRODUCER
procedure producer() {
    while (true) {
        int item = produce();
        ProducerConsumer.insert(item);
    }
}

// CONSUMER
procedure consumer() {
    while (true) {
        int item = ProducerConsumer.remove();
        consume(item);
    }
}
```

### Comparison: Monitor vs Semaphore

```
SEMAPHORE SOLUTION:                    MONITOR SOLUTION:
─────────────────                      ────────────────

// Producer                            monitor {
wait(empty);                             procedure insert(item) {
wait(mutex);                               while (count == N)
buffer[in] = item;                           notFull.wait();
in = (in+1) % N;                           buffer[in] = item;
signal(mutex);                             in = (in+1) % N;
signal(full);                              count++;
                                           notEmpty.signal();
                                         }
                                       }

Notice: No explicit mutex needed!
        Mutual exclusion automatic!
```

---

## Readers-Writers with Monitor

```
monitor ReadersWriters {
    int readers = 0;
    bool writing = false;
    
    condition canRead, canWrite;
    
    procedure startRead() {
        while (writing)
            canRead.wait();
        readers++;
    }
    
    procedure endRead() {
        readers--;
        if (readers == 0)
            canWrite.signal();
    }
    
    procedure startWrite() {
        while (writing || readers > 0)
            canWrite.wait();
        writing = true;
    }
    
    procedure endWrite() {
        writing = false;
        canWrite.signal();      // Prefer writers
        canRead.broadcast();    // Wake all waiting readers
    }
}

// READER
procedure reader() {
    ReadersWriters.startRead();
    // Read data
    ReadersWriters.endRead();
}

// WRITER
procedure writer() {
    ReadersWriters.startWrite();
    // Write data
    ReadersWriters.endWrite();
}
```

---

## Dining Philosophers with Monitor

```
monitor DiningPhilosophers {
    enum State { THINKING, HUNGRY, EATING };
    State state[5] = { THINKING, THINKING, THINKING, THINKING, THINKING };
    condition self[5];
    
    procedure test(int i) {
        if (state[(i+4) % 5] != EATING &&
            state[i] == HUNGRY &&
            state[(i+1) % 5] != EATING) {
            state[i] = EATING;
            self[i].signal();
        }
    }
    
    procedure pickup(int i) {
        state[i] = HUNGRY;
        test(i);
        if (state[i] != EATING)
            self[i].wait();
    }
    
    procedure putdown(int i) {
        state[i] = THINKING;
        test((i + 4) % 5);    // Check left neighbor
        test((i + 1) % 5);    // Check right neighbor
    }
}

// PHILOSOPHER i
procedure philosopher(int i) {
    while (true) {
        think();
        DiningPhilosophers.pickup(i);
        eat();
        DiningPhilosophers.putdown(i);
    }
}
```

---

## Monitor Implementation in Languages

### Java synchronized

```java
class BoundedBuffer {
    private int[] buffer;
    private int count = 0, in = 0, out = 0;
    
    public synchronized void insert(int item) throws InterruptedException {
        while (count == buffer.length)
            wait();                     // Condition wait
        
        buffer[in] = item;
        in = (in + 1) % buffer.length;
        count++;
        
        notifyAll();                    // Signal all
    }
    
    public synchronized int remove() throws InterruptedException {
        while (count == 0)
            wait();
        
        int item = buffer[out];
        out = (out + 1) % buffer.length;
        count--;
        
        notifyAll();
        return item;
    }
}
```

### Pthreads (C)

```c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t notFull = PTHREAD_COND_INITIALIZER;
pthread_cond_t notEmpty = PTHREAD_COND_INITIALIZER;

void insert(int item) {
    pthread_mutex_lock(&mutex);
    
    while (count == N)
        pthread_cond_wait(&notFull, &mutex);
    
    buffer[in] = item;
    in = (in + 1) % N;
    count++;
    
    pthread_cond_signal(&notEmpty);
    pthread_mutex_unlock(&mutex);
}
```

---

## Summary

| Concept | Description |
|---------|-------------|
| **Monitor** | High-level synchronization with automatic mutual exclusion |
| **Condition Variable** | Wait/signal mechanism inside monitor |
| **wait()** | Suspend and release monitor |
| **signal()** | Wake one waiting process |
| **Hoare Semantics** | Signaler waits, signaled runs immediately |
| **Mesa Semantics** | Signaler continues, use `while` for conditions |

### Advantages of Monitors

| Advantage | Description |
|-----------|-------------|
| **Automatic mutual exclusion** | No explicit lock/unlock needed |
| **Encapsulation** | Data and operations together |
| **Less error-prone** | Compiler enforces correct usage |
| **Cleaner code** | Easier to understand and maintain |

---

## Quick Revision Questions

1. What problems with semaphores do monitors solve?
2. What is the difference between monitor entry queue and condition queue?
3. Explain Hoare vs Mesa semantics.
4. Why must we use `while` instead of `if` with Mesa monitors?
5. How does Java implement monitors?

---

[← Previous: Classical Problems](./04-classical-problems.md) | [Next: Deadlock Introduction →](../04-Deadlocks/01-deadlock-introduction.md)
