# Chapter 3.4: Classical Synchronization Problems

## Overview

Several classical problems are used to test and demonstrate synchronization mechanisms. These problems represent real-world scenarios and serve as benchmarks for evaluating synchronization solutions.

---

## 1. Producer-Consumer Problem

### Problem Description

```
┌──────────────────────────────────────────────────────────────────┐
│                   PRODUCER-CONSUMER PROBLEM                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Scenario:                                                       │
│  • Producers create data items and place in a buffer            │
│  • Consumers remove data items from the buffer                  │
│  • Buffer has finite capacity (bounded buffer)                  │
│                                                                  │
│  ┌──────────┐      ┌────────────────────┐      ┌──────────┐     │
│  │ PRODUCER │ ───→ │      BUFFER        │ ───→ │ CONSUMER │     │
│  └──────────┘      │ [■][■][□][□][□]    │      └──────────┘     │
│                    │  ↑           ↑     │                        │
│                    │  in         out    │                        │
│                    └────────────────────┘                        │
│                                                                  │
│  Constraints:                                                    │
│  1. Producer must not add when buffer is FULL                   │
│  2. Consumer must not remove when buffer is EMPTY               │
│  3. Only one process accesses buffer at a time                  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Real-World Examples

| Scenario | Producer | Consumer | Buffer |
|----------|----------|----------|--------|
| Print spooler | Applications | Printer driver | Print queue |
| Web server | Network handler | Worker threads | Request queue |
| Keyboard | Keyboard driver | Application | Input buffer |
| Pipe | Writer process | Reader process | Pipe buffer |

### Solution Using Semaphores

```c
#define BUFFER_SIZE 5

// Shared data
int buffer[BUFFER_SIZE];
int in = 0;     // Next position to produce
int out = 0;    // Next position to consume

// Semaphores
semaphore mutex = 1;            // Mutual exclusion for buffer access
semaphore empty = BUFFER_SIZE;  // Count of empty slots
semaphore full = 0;             // Count of full slots

// PRODUCER
void producer() {
    int item;
    while (true) {
        item = produce_item();  // Create new item
        
        wait(empty);            // Wait for empty slot
        wait(mutex);            // Enter critical section
        
        buffer[in] = item;      // Add item to buffer
        in = (in + 1) % BUFFER_SIZE;
        
        signal(mutex);          // Exit critical section
        signal(full);           // Increment full count
    }
}

// CONSUMER
void consumer() {
    int item;
    while (true) {
        wait(full);             // Wait for item
        wait(mutex);            // Enter critical section
        
        item = buffer[out];     // Remove item from buffer
        out = (out + 1) % BUFFER_SIZE;
        
        signal(mutex);          // Exit critical section
        signal(empty);          // Increment empty count
        
        consume_item(item);     // Use the item
    }
}
```

### Visualization of Semaphore Values

```
Buffer Size = 5, Initially empty

State                    empty   full   mutex
─────                    ─────   ────   ─────
Initial                    5       0      1

Producer produces:
  wait(empty)              4       0      1
  wait(mutex)              4       0      0
  [add item]               4       0      0
  signal(mutex)            4       0      1
  signal(full)             4       1      1

Consumer consumes:
  wait(full)               4       0      1
  wait(mutex)              4       0      0
  [remove item]            4       0      0
  signal(mutex)            4       0      1
  signal(empty)            5       0      1
```

### Why Order Matters

```
WRONG ORDER (can cause deadlock):

Producer:                   Consumer:
wait(mutex);  ← Lock first  wait(mutex);  ← Lock first
wait(empty);  ← May block   wait(full);   ← May block

If buffer full: Producer has mutex, waits for empty
If buffer empty: Consumer has mutex, waits for full
Neither can proceed → DEADLOCK!

CORRECT ORDER:
wait(empty/full);  ← Semaphore first
wait(mutex);       ← Then lock
```

---

## 2. Readers-Writers Problem

### Problem Description

```
┌──────────────────────────────────────────────────────────────────┐
│                   READERS-WRITERS PROBLEM                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Scenario:                                                       │
│  • A shared database/resource                                   │
│  • Multiple readers can read simultaneously                     │
│  • Writers need exclusive access                                │
│                                                                  │
│          ┌─────────────────────────────┐                        │
│          │       SHARED DATA           │                        │
│          │  ┌─────────────────────┐    │                        │
│          │  │  DATABASE/FILE     │    │                        │
│          │  └─────────────────────┘    │                        │
│          └─────────────────────────────┘                        │
│                ↑       ↑       ↑                                │
│            ┌───┴───┐   │   ┌───┴───┐                           │
│            │Reader1│   │   │Reader2│  ← Can read together      │
│            └───────┘   │   └───────┘                           │
│                    ┌───┴───┐                                    │
│                    │Writer │  ← Needs exclusive access          │
│                    └───────┘                                    │
│                                                                  │
│  Rules:                                                          │
│  1. Multiple readers can read at the same time                  │
│  2. Only one writer can write at a time                         │
│  3. No reader can read while writer is writing                  │
│  4. No writer can write while reader is reading                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Variations

| Variation | Priority | Problem |
|-----------|----------|---------|
| **First Readers-Writers** | Readers | Writers may starve |
| **Second Readers-Writers** | Writers | Readers may starve |
| **Third (Fair)** | FIFO | More complex |

### First Readers-Writers Solution (Reader Priority)

```c
// Shared variables
int read_count = 0;         // Number of readers currently reading

// Semaphores
semaphore mutex = 1;        // Protects read_count
semaphore rw_mutex = 1;     // Exclusive access for writing

// WRITER
void writer() {
    while (true) {
        wait(rw_mutex);     // Get exclusive access
        
        // WRITING
        // ... write to shared data ...
        
        signal(rw_mutex);   // Release access
    }
}

// READER
void reader() {
    while (true) {
        wait(mutex);                // Protect read_count
        read_count++;
        if (read_count == 1)        // First reader
            wait(rw_mutex);         // Block writers
        signal(mutex);
        
        // READING
        // ... read shared data ...
        
        wait(mutex);                // Protect read_count
        read_count--;
        if (read_count == 0)        // Last reader
            signal(rw_mutex);       // Allow writers
        signal(mutex);
    }
}
```

### How It Works

```
┌──────────────────────────────────────────────────────────────────┐
│              FIRST READERS-WRITERS EXECUTION                      │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Scenario: R1 and R2 are readers, W1 is a writer                │
│                                                                  │
│  1. R1 arrives:                                                  │
│     read_count = 1 (first reader)                               │
│     wait(rw_mutex) - acquires lock                              │
│     R1 is reading                                               │
│                                                                  │
│  2. W1 arrives:                                                  │
│     wait(rw_mutex) - BLOCKS (held by readers)                   │
│                                                                  │
│  3. R2 arrives:                                                  │
│     read_count = 2                                              │
│     Not first, so skip wait(rw_mutex)                           │
│     R2 starts reading (with R1)                                 │
│                                                                  │
│  4. R1 finishes:                                                 │
│     read_count = 1                                              │
│     Not last, so skip signal(rw_mutex)                          │
│                                                                  │
│  5. R2 finishes:                                                 │
│     read_count = 0 (last reader)                                │
│     signal(rw_mutex) - W1 can proceed                           │
│                                                                  │
│  6. W1 starts writing (exclusive access)                        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Writer Priority Solution (Second Readers-Writers)

```c
// Additional variables
int write_count = 0;        // Number of waiting/active writers
semaphore read_try = 1;     // Writers can block new readers

// WRITER (with priority)
void writer() {
    while (true) {
        wait(mutex2);               // Protect write_count
        write_count++;
        if (write_count == 1)       // First writer
            wait(read_try);         // Block new readers
        signal(mutex2);
        
        wait(rw_mutex);             // Get exclusive access
        
        // WRITING
        
        signal(rw_mutex);
        
        wait(mutex2);
        write_count--;
        if (write_count == 0)       // Last writer
            signal(read_try);       // Allow readers
        signal(mutex2);
    }
}

// READER (modified)
void reader() {
    while (true) {
        wait(read_try);             // Check if writers waiting
        wait(mutex);
        read_count++;
        if (read_count == 1)
            wait(rw_mutex);
        signal(mutex);
        signal(read_try);           // Release quickly
        
        // READING
        
        wait(mutex);
        read_count--;
        if (read_count == 0)
            signal(rw_mutex);
        signal(mutex);
    }
}
```

---

## 3. Dining Philosophers Problem

### Problem Description

```
┌──────────────────────────────────────────────────────────────────┐
│                DINING PHILOSOPHERS PROBLEM                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Five philosophers sit around a circular table                   │
│  Each philosopher alternates between THINKING and EATING        │
│  Between each pair of philosophers is ONE chopstick              │
│  To eat, a philosopher needs BOTH chopsticks (left and right)   │
│                                                                  │
│                        [P0]                                      │
│                     c4/    \c0                                   │
│                      /      \                                    │
│                  [P4]        [P1]                                │
│                    |          |                                  │
│                  c3|          |c1                                │
│                    |          |                                  │
│                  [P3]────────[P2]                                │
│                        c2                                        │
│                                                                  │
│  Challenge: Avoid deadlock and starvation!                      │
│                                                                  │
│  If all philosophers pick up left chopstick simultaneously:     │
│  → Everyone waits for right chopstick                          │
│  → DEADLOCK!                                                    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Naive Solution (DEADLOCK PRONE!)

```c
semaphore chopstick[5] = {1, 1, 1, 1, 1};

// Philosopher i
void philosopher(int i) {
    while (true) {
        think();
        
        wait(chopstick[i]);           // Pick up left
        wait(chopstick[(i+1) % 5]);   // Pick up right
        
        eat();
        
        signal(chopstick[(i+1) % 5]); // Put down right
        signal(chopstick[i]);          // Put down left
    }
}
```

**Why This Deadlocks:**

```
All philosophers pick up left chopstick simultaneously:
P0: holds c0, wants c1
P1: holds c1, wants c2
P2: holds c2, wants c3
P3: holds c3, wants c4
P4: holds c4, wants c0

Circular wait → DEADLOCK!
```

### Solution 1: Asymmetric Ordering

```c
// Odd philosophers pick left first, even pick right first
void philosopher(int i) {
    while (true) {
        think();
        
        if (i % 2 == 0) {
            wait(chopstick[i]);           // Left first
            wait(chopstick[(i+1) % 5]);   // Then right
        } else {
            wait(chopstick[(i+1) % 5]);   // Right first
            wait(chopstick[i]);           // Then left
        }
        
        eat();
        
        signal(chopstick[(i+1) % 5]);
        signal(chopstick[i]);
    }
}
```

### Solution 2: Limit Concurrent Diners

```c
semaphore chopstick[5] = {1, 1, 1, 1, 1};
semaphore room = 4;  // Only 4 philosophers can try at once

void philosopher(int i) {
    while (true) {
        think();
        
        wait(room);                       // Enter dining room
        wait(chopstick[i]);
        wait(chopstick[(i+1) % 5]);
        
        eat();
        
        signal(chopstick[(i+1) % 5]);
        signal(chopstick[i]);
        signal(room);                     // Leave dining room
    }
}
```

**Why This Works:**

```
With only 4 philosophers allowed to try:
• At least one philosopher can get both chopsticks
• No circular wait possible with 4 philosophers and 5 chopsticks
```

### Solution 3: Pick Up Both or None (Atomic)

```c
semaphore mutex = 1;
int state[5] = {THINKING, THINKING, THINKING, THINKING, THINKING};

#define THINKING 0
#define HUNGRY   1
#define EATING   2
#define LEFT     (i + 4) % 5
#define RIGHT    (i + 1) % 5

semaphore self[5] = {0, 0, 0, 0, 0};

void test(int i) {
    if (state[i] == HUNGRY && 
        state[LEFT] != EATING && 
        state[RIGHT] != EATING) {
        state[i] = EATING;
        signal(self[i]);
    }
}

void take_forks(int i) {
    wait(mutex);
    state[i] = HUNGRY;
    test(i);                  // Try to get both forks
    signal(mutex);
    wait(self[i]);            // Block if couldn't get both
}

void put_forks(int i) {
    wait(mutex);
    state[i] = THINKING;
    test(LEFT);               // Check if left neighbor can eat
    test(RIGHT);              // Check if right neighbor can eat
    signal(mutex);
}

void philosopher(int i) {
    while (true) {
        think();
        take_forks(i);
        eat();
        put_forks(i);
    }
}
```

---

## 4. Sleeping Barber Problem

### Problem Description

```
┌──────────────────────────────────────────────────────────────────┐
│                  SLEEPING BARBER PROBLEM                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────────────────────────┐                   │
│  │            BARBER SHOP                   │                   │
│  │                                          │                   │
│  │   Waiting Room        Barber Chair       │                   │
│  │   ┌──┐ ┌──┐ ┌──┐     ┌─────────┐        │                   │
│  │   │C1│ │C2│ │C3│     │ BARBER  │        │                   │
│  │   └──┘ └──┘ └──┘     │  💈    │        │                   │
│  │   (n chairs)          └─────────┘        │                   │
│  │                                          │                   │
│  └──────────────────────────────────────────┘                   │
│                                                                  │
│  Rules:                                                          │
│  • If no customers, barber sleeps in barber chair               │
│  • If barber is sleeping, customer wakes him up                 │
│  • If barber is busy, customer waits (if chair available)       │
│  • If no waiting chairs, customer leaves                        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Solution

```c
#define CHAIRS 5        // Waiting chairs

int waiting = 0;        // Customers waiting

semaphore customers = 0;    // Waiting customers (signals barber)
semaphore barber = 0;       // Barber ready (signals customer)
semaphore mutex = 1;        // Protects 'waiting'

// BARBER
void barber_process() {
    while (true) {
        wait(customers);        // Sleep if no customers
        wait(mutex);
        waiting--;              // Take one customer
        signal(mutex);
        signal(barber);         // Barber is ready
        cut_hair();             // Cutting hair (outside CS)
    }
}

// CUSTOMER
void customer_process() {
    wait(mutex);
    if (waiting < CHAIRS) {     // If room available
        waiting++;              // Take a seat
        signal(mutex);
        signal(customers);      // Wake barber if sleeping
        wait(barber);           // Wait for barber to be ready
        get_haircut();          // Getting haircut
    } else {
        signal(mutex);          // No room, leave
        leave_shop();
    }
}
```

---

## Summary Comparison

| Problem | Key Challenge | Main Resources | Solution Technique |
|---------|---------------|----------------|-------------------|
| **Producer-Consumer** | Buffer full/empty | Buffer, slots | Counting semaphores |
| **Readers-Writers** | Concurrent reads, exclusive writes | Shared data | Read count + mutex |
| **Dining Philosophers** | Circular deadlock | Chopsticks | Order/limit/atomic |
| **Sleeping Barber** | Sleep/wake coordination | Chairs, barber | Signaling pattern |

---

## Quick Revision Questions

1. In producer-consumer, why must we wait on empty/full before mutex?
2. How does the first readers-writers solution cause writer starvation?
3. Why does the naive dining philosophers solution deadlock?
4. How does limiting to 4 philosophers prevent deadlock?
5. In sleeping barber, how does the barber know when to sleep?

---

[← Previous: Semaphores](./03-semaphores.md) | [Next: Monitors →](./05-monitors.md)
