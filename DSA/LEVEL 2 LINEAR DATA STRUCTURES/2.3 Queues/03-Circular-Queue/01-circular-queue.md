# Unit 3: Circular Queue

[← Previous: Queue Implementation](../02-Queue-Implementation/01-queue-implementation.md) | [Back to README](../README.md) | [Next: Deque →](../04-Deque/01-deque.md)

---

## Chapter Overview

The **circular queue** (also called a **ring buffer**) is the most practical array-based queue implementation. It solves the wasted-space problem of linear array queues by treating the array as if it wraps around in a circle. This unit covers the motivation, implementation, edge cases, and real-world applications.

---

## 1. Why Circular Queue?

### The Problem with Linear Array Queue

In a linear array queue with front pointer movement:

```
After several Enqueue + Dequeue cycles:

  Index:    0     1     2     3     4
          ┌─────┬─────┬─────┬─────┬─────┐
          │ xx  │ xx  │ xx  │ 40  │ 50  │
          └─────┴─────┴─────┴─────┴─────┘
            ↑                 ↑     ↑
          WASTED            front  rear
          SPACE!

  Indices 0, 1, 2 are UNUSED but UNAVAILABLE.
  Can't enqueue even though 3 slots are free!
  → FALSE OVERFLOW
```

### The Circular Queue Solution

Treat the array as a **circle** — when `rear` reaches the end, it wraps back to index 0.

```
  LINEAR VIEW:                    CIRCULAR VIEW:

  ┌───┬───┬───┬───┬───┐              ┌───┐
  │ 0 │ 1 │ 2 │ 3 │ 4 │           ┌──│ 0 │──┐
  └───┴───┴───┴───┴───┘           │  └───┘  │
     ↑               ↑          ┌───┐     ┌───┐
   wraps ←──────── end         │ 4 │     │ 1 │
                               └───┘     └───┘
                                  │  ┌───┐  │
                                  └──│ 3 │──┘
                                     └───┘
                                      │
                                    ┌───┐
                                    │ 2 │
                                    └───┘

  After index 4 comes index 0 again!
```

### The Magic Formula

```
next_index = (current_index + 1) % capacity
```

Examples with capacity = 5:
| Current | (current + 1) % 5 | Next |
|:-------:|:------------------:|:----:|
| 0 | 1 % 5 = 1 | 1 |
| 1 | 2 % 5 = 2 | 2 |
| 2 | 3 % 5 = 3 | 3 |
| 3 | 4 % 5 = 4 | 4 |
| **4** | **5 % 5 = 0** | **0** ← Wraps! |

---

## 2. Implementation Details

### Data Structure

```
class CircularQueue:
    array[]          // Fixed-size array
    capacity         // Maximum number of elements
    front            // Index of front element
    rear             // Index of rear element
    size             // Current number of elements
```

### Initialization

```
INIT(cap):
    capacity = cap
    array = new Array[capacity]
    front = 0
    rear = -1
    size = 0
```

### Complete Pseudocode

```
ENQUEUE(item):
    if size == capacity:
        ERROR "Queue Overflow"
    
    rear = (rear + 1) % capacity      // Advance rear (wraps around)
    array[rear] = item
    size = size + 1

DEQUEUE():
    if size == 0:
        ERROR "Queue Underflow"
    
    item = array[front]
    front = (front + 1) % capacity    // Advance front (wraps around)
    size = size - 1
    return item

PEEK():
    if size == 0:
        ERROR "Queue Underflow"
    return array[front]

IS_EMPTY():
    return size == 0

IS_FULL():
    return size == capacity

GET_REAR():
    if size == 0:
        ERROR "Queue Underflow"
    return array[rear]
```

### Step-by-Step Trace

```
Capacity = 5, front = 0, rear = -1, size = 0

═══ Enqueue(10): rear = (−1+1)%5 = 0 ═══

  ┌──────┬──────┬──────┬──────┬──────┐
  │  10  │      │      │      │      │
  └──────┴──────┴──────┴──────┴──────┘
     ↑
   f=0, r=0     size=1

═══ Enqueue(20): rear = (0+1)%5 = 1 ═══

  ┌──────┬──────┬──────┬──────┬──────┐
  │  10  │  20  │      │      │      │
  └──────┴──────┴──────┴──────┴──────┘
     ↑      ↑
    f=0    r=1   size=2

═══ Enqueue(30): rear = (1+1)%5 = 2 ═══

  ┌──────┬──────┬──────┬──────┬──────┐
  │  10  │  20  │  30  │      │      │
  └──────┴──────┴──────┴──────┴──────┘
     ↑             ↑
    f=0           r=2   size=3

═══ Dequeue() → 10: front = (0+1)%5 = 1 ═══

  ┌──────┬──────┬──────┬──────┬──────┐
  │  --  │  20  │  30  │      │      │
  └──────┴──────┴──────┴──────┴──────┘
            ↑      ↑
           f=1    r=2   size=2

═══ Enqueue(40): rear = (2+1)%5 = 3 ═══

  ┌──────┬──────┬──────┬──────┬──────┐
  │  --  │  20  │  30  │  40  │      │
  └──────┴──────┴──────┴──────┴──────┘
            ↑                   ↑
           f=1                r=3   size=3

═══ Enqueue(50): rear = (3+1)%5 = 4 ═══

  ┌──────┬──────┬──────┬──────┬──────┐
  │  --  │  20  │  30  │  40  │  50  │
  └──────┴──────┴──────┴──────┴──────┘
            ↑                    ↑
           f=1                  r=4   size=4

═══ Enqueue(60): rear = (4+1)%5 = 0  ← WRAPS AROUND! ═══

  ┌──────┬──────┬──────┬──────┬──────┐
  │  60  │  20  │  30  │  40  │  50  │
  └──────┴──────┴──────┴──────┴──────┘
     ↑      ↑
    r=0    f=1   size=5  → FULL!

═══ Enqueue(70): size == capacity → OVERFLOW ═══

═══ Dequeue() → 20: front = (1+1)%5 = 2 ═══

  ┌──────┬──────┬──────┬──────┬──────┐
  │  60  │  --  │  30  │  40  │  50  │
  └──────┴──────┴──────┴──────┴──────┘
     ↑             ↑
    r=0           f=2   size=4

  Logical order: 30, 40, 50, 60
  (wraps: index 2→3→4→0)
```

### Circular View of Final State

```
              f=2
               ↓
            ┌──────┐
         ┌──│  30  │──┐
         │  └──────┘  │
      ┌──────┐     ┌──────┐
      │  --  │     │  40  │
      │  i=1 │     │  i=3 │
      └──────┘     └──────┘
         │  ┌──────┐  │
         └──│  50  │──┘
            └──────┘
             i=4  │
            ┌──────┐
            │  60  │
            └──────┘
              i=0
               ↑
              r=0

  Logical order (front→rear): 30→40→50→60
```

---

## 3. Full vs Empty Detection

This is the **classic challenge** of circular queues. When `front == rear`, is the queue full or empty?

### The Ambiguity Problem

```
EMPTY Queue:                    FULL Queue:
front == rear?                  front == rear?

  ┌───┬───┬───┬───┬───┐         ┌───┬───┬───┬───┬───┐
  │   │   │   │   │   │         │ A │ B │ C │ D │ E │
  └───┴───┴───┴───┴───┘         └───┴───┴───┴───┴───┘
    ↑                              ↑
  f=r=0                          f=r=0

  Both states look the same if we only track front and rear!
```

### Solution 1: Use a Size Counter (Recommended)

```
IS_EMPTY():  return size == 0
IS_FULL():   return size == capacity
```

- **Pros:** Unambiguous, simple, provides O(1) `size()`
- **Cons:** Extra variable to maintain

### Solution 2: Waste One Slot

Reserve one slot — queue is full when `(rear + 1) % capacity == front`.

```
Maximum elements = capacity - 1

FULL condition:  (rear + 1) % capacity == front
EMPTY condition: rear == front  (or use a flag)

  FULL (capacity=5, usable=4):
  ┌───┬───┬───┬───┬───┐
  │ D │   │ A │ B │ C │
  └───┴───┴───┴───┴───┘
    ↑   ↑   ↑
   r=0 GAP f=2

  (rear+1) % 5 = 1... wait, but front=2.
  Actually: we check (rear+1)%cap == front
  Here next of rear(0) = 1, front = 2    → NOT full yet

  ┌───┬───┬───┬───┬───┐
  │ D │ E │ A │ B │ C │     Wait — now only 1 gap expected
  └───┴───┴───┴───┴───┘
    ↑   ↑   ↑
        r=1 f=2

  (rear+1)%5 = (1+1)%5 = 2 == front(2) → FULL!
  Only 4 elements stored in capacity-5 array.
```

- **Pros:** No extra variable
- **Cons:** Wastes one slot, can confuse capacity vs usable size

### Solution 3: Boolean Flag

```
Use a boolean `lastWasEnqueue`:
  - After enqueue: lastWasEnqueue = true
  - After dequeue: lastWasEnqueue = false

When front == rear:
  - If lastWasEnqueue → FULL
  - If !lastWasEnqueue → EMPTY
```

- **Pros:** Uses full capacity
- **Cons:** Extra boolean, slightly more complex logic

### Comparison

| Method | Space Overhead | Wasted Slots | Complexity |
|--------|:--------------:|:------------:|:----------:|
| Size counter | 1 integer | 0 | Simple ✓ |
| Waste one slot | 0 | 1 slot | Moderate |
| Boolean flag | 1 boolean | 0 | Moderate |

> **Recommendation:** Use the **size counter** approach. It's the clearest and most maintainable.

---

## 4. Circular Array Technique

The circular array technique is a general pattern used beyond just queues.

### Modular Arithmetic Essentials

```
The MOD operator (%) is the key to circular behavior.

  a % n  always gives a result in the range [0, n-1]

  Examples (n=5):
    0 % 5 = 0        5 % 5 = 0       10 % 5 = 0
    1 % 5 = 1        6 % 5 = 1       11 % 5 = 1
    2 % 5 = 2        7 % 5 = 2       12 % 5 = 2
    3 % 5 = 3        8 % 5 = 3       13 % 5 = 3
    4 % 5 = 4        9 % 5 = 4       14 % 5 = 4
```

### Common Formulas

| Operation | Formula |
|-----------|---------|
| Next index | `(i + 1) % capacity` |
| Previous index | `(i - 1 + capacity) % capacity` |
| Advance by k | `(i + k) % capacity` |
| Distance front→rear | `(rear - front + capacity) % capacity` |
| Number of elements | `size` variable *or* `(rear - front + capacity) % capacity + 1` |

### Why `(i - 1 + capacity) % capacity` for Previous?

```
If i = 0 and capacity = 5:
  (0 - 1) % 5 = -1 % 5   → Implementation-defined in some languages!
  
  Safe version:
  (0 - 1 + 5) % 5 = 4 % 5 = 4  ✓  (wraps to last index)
```

> **Important:** In languages like C/C++, the result of `%` with negative numbers is implementation-defined. Always add `capacity` before taking mod to ensure positive results.

### Traversal Pattern

To traverse all elements from front to rear:

```
TRAVERSE():
    if size == 0:
        return
    
    index = front
    for count = 0 to size - 1:
        PROCESS(array[index])
        index = (index + 1) % capacity
```

### Trace of Traversal

```
  ┌──────┬──────┬──────┬──────┬──────┐
  │  60  │  --  │  30  │  40  │  50  │
  └──────┴──────┴──────┴──────┴──────┘
     ↑             ↑
    r=0           f=2     size=4

  Traversal:
    count=0: index=2 → 30,  next index=(2+1)%5=3
    count=1: index=3 → 40,  next index=(3+1)%5=4
    count=2: index=4 → 50,  next index=(4+1)%5=0  ← wraps!
    count=3: index=0 → 60,  done

  Output: 30, 40, 50, 60 ✓
```

---

## 5. Applications

### Application 1: CPU Scheduling (Round Robin)

```
  Round Robin with Time Quantum = 2:

  ┌──────────────────────────────────────────────┐
  │              READY QUEUE                      │
  │   ┌────┬────┬────┐                           │
  │   │ P3 │ P2 │ P1 │ ← front                  │
  │   └────┴────┴────┘                           │
  │          ↓                                    │
  │     Dequeue P1 → Run for 2 units             │
  │          ↓                                    │
  │     P1 not done? Enqueue P1 back             │
  │   ┌────┬────┬────┐                           │
  │   │ P1 │ P3 │ P2 │ ← front                  │
  │   └────┴────┴────┘                           │
  └──────────────────────────────────────────────┘
```

### Application 2: Traffic Signal System

```
  Circular queue of signal phases:

        ┌─────────┐
     ┌──│  GREEN  │──┐
     │  └─────────┘  │
  ┌──────┐        ┌────────┐
  │ RED  │        │ YELLOW │
  └──────┘        └────────┘
     │  ┌─────────┐  │
     └──│  RED+   │──┘
        │ YELLOW  │
        └─────────┘

  Repeats: GREEN → YELLOW → RED → RED+YELLOW → GREEN → ...
```

### Application 3: Streaming Data Buffer

```
  Producer writes data → Buffer (circular queue) → Consumer reads

  ┌──────────┐     ┌───┬───┬───┬───┬───┐     ┌──────────┐
  │ PRODUCER │────→│ D │ E │ A │ B │ C │────→│ CONSUMER │
  │ (writes) │     └───┴───┴───┴───┴───┘     │ (reads)  │
  └──────────┘       ↑           ↑            └──────────┘
                   rear        front
  
  - Producer enqueues at rear
  - Consumer dequeues from front
  - Circular: never needs reallocation
```

### Application 4: Keyboard Buffer

```
  Key presses stored in circular buffer:

  User types: H, E, L, L, O
  
  ┌───┬───┬───┬───┬───┬───┬───┬───┐
  │ H │ E │ L │ L │ O │   │   │   │
  └───┴───┴───┴───┴───┴───┴───┴───┘
    ↑                   ↑
  front               rear

  OS reads from front at its own pace.
  If buffer fills → oldest keys overwritten (ring buffer behavior).
```

---

## 6. Ring Buffer Concept

A **ring buffer** is a circular queue used in systems programming where the **oldest data is overwritten** when the buffer is full (no overflow error).

### Difference from Regular Circular Queue

| Feature | Circular Queue | Ring Buffer |
|---------|:--------------:|:-----------:|
| Full behavior | Error / block | **Overwrite oldest** |
| Data loss | Never | Possible (by design) |
| Use case | Task queues | Streaming / logging |

### Ring Buffer Pseudocode

```
WRITE(item):    // Always succeeds — overwrites if full
    rear = (rear + 1) % capacity
    
    if size == capacity:
        // Overwriting oldest — advance front
        front = (front + 1) % capacity
    else:
        size = size + 1
    
    buffer[rear] = item

READ():
    if size == 0:
        ERROR "Buffer Empty"
    
    item = buffer[front]
    front = (front + 1) % capacity
    size = size - 1
    return item
```

### Ring Buffer Overwrite Trace

```
Capacity = 3

─── Write(A): rear=0 ───
  ┌───┬───┬───┐
  │ A │   │   │   size=1, f=0, r=0
  └───┴───┴───┘

─── Write(B): rear=1 ───
  ┌───┬───┬───┐
  │ A │ B │   │   size=2, f=0, r=1
  └───┴───┴───┘

─── Write(C): rear=2 ───
  ┌───┬───┬───┐
  │ A │ B │ C │   size=3(FULL), f=0, r=2
  └───┴───┴───┘

─── Write(D): rear=(2+1)%3=0, OVERWRITE! front advances to 1 ───
  ┌───┬───┬───┐
  │ D │ B │ C │   size=3(FULL), f=1, r=0
  └───┴───┴───┘
       ↑
     front
  
  'A' was overwritten! Oldest data lost (by design).
  Logical order: B, C, D

─── Write(E): rear=(0+1)%3=1, OVERWRITE! front advances to 2 ───
  ┌───┬───┬───┐
  │ D │ E │ C │   size=3(FULL), f=2, r=1
  └───┴───┴───┘
            ↑
          front

  'B' was overwritten!
  Logical order: C, D, E
```

### Real-World Ring Buffer Uses

| Application | What's Buffered | Why Overwrite? |
|-------------|-----------------|----------------|
| **Audio playback** | Sound samples | Only recent samples matter |
| **Network logging** | Packet history | Old logs less relevant |
| **Game replay** | Last N frames | Memory bounded |
| **Kernel logging** | `dmesg` ring buffer | Fixed memory for logs |
| **Sensor data** | Temperature readings | Only recent data needed |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Linear queue problem | Wasted space at front after dequeues |
| Circular solution | Wrap around using `(index + 1) % capacity` |
| Full detection | Best: use `size == capacity` |
| Empty detection | Best: use `size == 0` |
| Waste-one-slot | Full when `(rear+1)%cap == front`, wastes 1 slot |
| Traversal | Loop `size` times, advancing index with `% capacity` |
| Previous index | `(i - 1 + capacity) % capacity` (avoids negative mod) |
| Ring buffer | Circular queue that overwrites oldest on overflow |
| Ring buffer use | Streaming, logging, audio — where old data can be lost |
| All operations | O(1) time — enqueue, dequeue, peek, isEmpty, isFull |

---

## Quick Revision Questions

1. **What is the "false overflow" problem in linear array queues, and how does a circular queue fix it?**

2. **Given a circular queue with capacity=6, front=4, and rear=2 with size=5, trace a dequeue followed by an enqueue(99).**

3. **Why is `(i - 1 + capacity) % capacity` used instead of `(i - 1) % capacity` for finding the previous index?**

4. **Explain three different strategies for distinguishing full vs empty in a circular queue. Which do you prefer and why?**

5. **How does a ring buffer differ from a standard circular queue when the buffer is full? Give a use case.**

6. **A circular queue has capacity=4 and these operations: Enqueue(A), Enqueue(B), Enqueue(C), Dequeue(), Dequeue(), Enqueue(D), Enqueue(E), Enqueue(F). Draw the final state.**

---

## Answers (Hidden — Think First!)

<details>
<summary>Click to reveal answers</summary>

1. **False overflow** occurs when `rear` reaches the last index but slots at the beginning (before `front`) are unused. The queue reports "full" even though space exists. A circular queue wraps `rear` back to index 0 using `(rear + 1) % capacity`, reusing those freed slots.

2. State: capacity=6, front=4, rear=2, size=5
   ```
   Indices:  0   1   2   3   4   5
           [ x ] [x] [x] [ ] [x] [x]    f=4, r=2
   
   Dequeue(): item=array[4], front=(4+1)%6=5, size=4
           [ x ] [x] [x] [ ] [--] [x]   f=5, r=2
   
   Enqueue(99): rear=(2+1)%6=3, array[3]=99, size=5
           [ x ] [x] [x] [99] [--] [x]  f=5, r=3
   ```

3. In some languages (C, C++, Java), `-1 % 5` can return `-1` instead of `4`. Adding `capacity` first ensures the number is always non-negative before taking mod: `(-1 + 5) % 5 = 4 % 5 = 4`.

4. **(a) Size counter:** Track `size` variable — full when `size == capacity`, empty when `size == 0`. **(b) Waste one slot:** Full when `(rear+1)%cap == front`. **(c) Boolean flag:** Track `lastWasEnqueue` — when `front == rear`, check the flag. **Preferred: size counter** — simplest, unambiguous, provides O(1) `size()` with no wasted space.

5. A standard circular queue rejects new elements (overflow error) when full. A ring buffer **overwrites the oldest element** — it advances `front` to discard old data and writes new data at `rear`. **Use case:** Audio playback buffer — if the consumer is slow, old audio samples are irrelevant, so overwriting them keeps the stream current.

6. Trace with capacity=4:
   ```
   Enqueue(A): r=0      [A][ ][ ][ ]  f=0, r=0, s=1
   Enqueue(B): r=1      [A][B][ ][ ]  f=0, r=1, s=2
   Enqueue(C): r=2      [A][B][C][ ]  f=0, r=2, s=3
   Dequeue()→A: f=1     [-][B][C][ ]  f=1, r=2, s=2
   Dequeue()→B: f=2     [-][-][C][ ]  f=2, r=2, s=1
   Enqueue(D): r=3      [-][-][C][D]  f=2, r=3, s=2
   Enqueue(E): r=0 WRAP [E][-][C][D]  f=2, r=0, s=3
   Enqueue(F): r=1 WRAP [E][F][C][D]  f=2, r=1, s=4 FULL
   ```
   Final: `[E][F][C][D]`, front=2, rear=1, logical order: C, D, E, F

</details>

---

[← Previous: Queue Implementation](../02-Queue-Implementation/01-queue-implementation.md) | [Back to README](../README.md) | [Next: Deque →](../04-Deque/01-deque.md)
