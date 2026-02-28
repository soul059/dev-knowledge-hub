# Unit 4: Deque (Double-Ended Queue)

[← Previous: Circular Queue](../03-Circular-Queue/01-circular-queue.md) | [Back to README](../README.md) | [Next: Priority Queue →](../05-Priority-Queue/01-priority-queue.md)

---

## Chapter Overview

A **deque** (pronounced "deck") is a generalized queue that allows insertion and deletion at **both ends** — front and rear. This unit covers the deque ADT, implementation strategies, restricted variants, and the classic sliding window maximum problem.

---

## 1. What is a Deque?

A **Double-Ended Queue (Deque)** supports four operations: insert/delete at front AND insert/delete at rear.

```
       insertFront                               insertRear
       ──────────→                               ──────────→
                  ┌──────┬──────┬──────┬──────┐
                  │  A   │  B   │  C   │  D   │
                  └──────┴──────┴──────┴──────┘
       ←──────────                               ←──────────
       deleteFront                               deleteRear

  FRONT ─────────────────────────────────────── REAR
```

### Deque vs Queue vs Stack

```
  QUEUE:    Insert at REAR only,  Delete from FRONT only
            ──→  [A][B][C][D]  ──→

  STACK:    Insert at TOP only,   Delete from TOP only
            ↕    [D]
                 [C]
                 [B]
                 [A]

  DEQUE:    Insert at BOTH ends,  Delete from BOTH ends
         ←──→  [A][B][C][D]  ←──→  (superset of queue AND stack!)
```

> **Key Insight:** A deque can simulate both a queue (use one end for insert, other for delete) and a stack (use same end for both insert and delete).

### Deque ADT

| Operation | Description | Time |
|-----------|-------------|:----:|
| `insertFront(item)` | Add item at the front | O(1) |
| `insertRear(item)` | Add item at the rear | O(1) |
| `deleteFront()` | Remove and return front item | O(1) |
| `deleteRear()` | Remove and return rear item | O(1) |
| `peekFront()` | View front without removing | O(1) |
| `peekRear()` | View rear without removing | O(1) |
| `isEmpty()` | Check if deque is empty | O(1) |
| `size()` | Return number of elements | O(1) |

---

## 2. Operations (Front, Rear)

### Detailed Operation Walkthrough

```
═══ Start with empty deque ═══
  [ ]    front=0, rear=-1, size=0

═══ insertRear(10) ═══
  [ 10 ]    Elements: 10
   ↑  ↑
   f  r

═══ insertRear(20) ═══
  [ 10 | 20 ]    Elements: 10, 20
    ↑    ↑
    f    r

═══ insertFront(5) ═══
  [ 5 | 10 | 20 ]    Elements: 5, 10, 20
    ↑         ↑
    f         r

═══ insertFront(1) ═══
  [ 1 | 5 | 10 | 20 ]    Elements: 1, 5, 10, 20
    ↑              ↑
    f              r

═══ deleteRear() → 20 ═══
  [ 1 | 5 | 10 ]    Elements: 1, 5, 10
    ↑         ↑
    f         r

═══ deleteFront() → 1 ═══
  [ 5 | 10 ]    Elements: 5, 10
    ↑    ↑
    f    r

═══ insertRear(30) ═══
  [ 5 | 10 | 30 ]    Elements: 5, 10, 30
    ↑          ↑
    f          r

═══ deleteFront() → 5 ═══
  [ 10 | 30 ]    Elements: 10, 30
    ↑     ↑
    f     r
```

### Simulating Queue and Stack with Deque

```
QUEUE using Deque:
  enqueue(x) → insertRear(x)
  dequeue()  → deleteFront()

STACK using Deque:
  push(x) → insertRear(x)      (or insertFront)
  pop()   → deleteRear()       (or deleteFront, same end)
```

---

## 3. Implementation Approaches

### Approach 1: Circular Array-Based Deque

```
class CircularDeque:
    array[]
    capacity
    front        // Index of front element
    rear         // Index of rear element
    size

    INIT(cap):
        capacity = cap
        array = new Array[capacity]
        front = 0
        rear = -1
        size = 0
```

### Pseudocode

```
INSERT_FRONT(item):
    if size == capacity:
        ERROR "Deque Overflow"
    front = (front - 1 + capacity) % capacity   // Move front backward
    array[front] = item
    if size == 0:
        rear = front                             // First element
    size = size + 1

INSERT_REAR(item):
    if size == capacity:
        ERROR "Deque Overflow"
    rear = (rear + 1) % capacity                 // Move rear forward
    array[rear] = item
    if size == 0:
        front = rear                             // First element
    size = size + 1

DELETE_FRONT():
    if size == 0:
        ERROR "Deque Underflow"
    item = array[front]
    front = (front + 1) % capacity               // Move front forward
    size = size - 1
    return item

DELETE_REAR():
    if size == 0:
        ERROR "Deque Underflow"
    item = array[rear]
    rear = (rear - 1 + capacity) % capacity      // Move rear backward
    size = size - 1
    return item

PEEK_FRONT():
    if size == 0: ERROR "Underflow"
    return array[front]

PEEK_REAR():
    if size == 0: ERROR "Underflow"
    return array[rear]
```

### Circular Array Trace

```
Capacity = 5

═══ insertRear(A):  rear=(−1+1)%5=0 ═══
  Index:   0    1    2    3    4
         [ A ] [  ] [  ] [  ] [  ]
         f=0, r=0, size=1

═══ insertRear(B):  rear=(0+1)%5=1 ═══
         [ A ] [ B ] [  ] [  ] [  ]
         f=0, r=1, size=2

═══ insertFront(Z):  front=(0−1+5)%5=4 ═══
         [ A ] [ B ] [  ] [  ] [ Z ]
                                 ↑
         f=4, r=1, size=3

  Logical order: Z, A, B   (index 4→0→1)

═══ insertFront(Y):  front=(4−1+5)%5=3 ═══
         [ A ] [ B ] [  ] [ Y ] [ Z ]
                            ↑
         f=3, r=1, size=4

  Logical order: Y, Z, A, B   (index 3→4→0→1)

═══ deleteRear() → B:  rear=(1−1+5)%5=0 ═══
         [ A ] [  ] [  ] [ Y ] [ Z ]
           ↑                ↑
         f=3, r=0, size=3

  Logical order: Y, Z, A   (index 3→4→0)

═══ deleteFront() → Y:  front=(3+1)%5=4 ═══
         [ A ] [  ] [  ] [  ] [ Z ]
           ↑                    ↑
         f=4, r=0, size=2

  Logical order: Z, A   (index 4→0)
```

### Approach 2: Doubly Linked List-Based Deque

```
  front                                            rear
    │                                                │
    ▼                                                ▼
  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
  │ NULL ← prev  │    │  prev        │    │  prev        │
  │     10       │←──→│     20       │←──→│     30       │
  │  next →      │    │  next →      │    │  next → NULL │
  └──────────────┘    └──────────────┘    └──────────────┘
```

### Doubly Linked List Pseudocode

```
class DLLNode:
    data
    prev = NULL
    next = NULL

class DLLDeque:
    front = NULL
    rear  = NULL
    size  = 0

    INSERT_FRONT(item):
        newNode = new DLLNode(item)
        if front == NULL:          // Empty
            front = rear = newNode
        else:
            newNode.next = front
            front.prev = newNode
            front = newNode
        size = size + 1

    INSERT_REAR(item):
        newNode = new DLLNode(item)
        if rear == NULL:           // Empty
            front = rear = newNode
        else:
            newNode.prev = rear
            rear.next = newNode
            rear = newNode
        size = size + 1

    DELETE_FRONT():
        if front == NULL: ERROR "Underflow"
        item = front.data
        front = front.next
        if front == NULL:
            rear = NULL
        else:
            front.prev = NULL
        size = size - 1
        return item

    DELETE_REAR():
        if rear == NULL: ERROR "Underflow"
        item = rear.data
        rear = rear.prev
        if rear == NULL:
            front = NULL
        else:
            rear.next = NULL
        size = size - 1
        return item
```

### Comparison of Approaches

| Criteria | Circular Array | Doubly Linked List |
|----------|:--------------:|:------------------:|
| All operations | O(1) | O(1) |
| Memory | Fixed, contiguous | Dynamic, scattered |
| Cache performance | Excellent | Poor |
| Resize | Needs copying | N/A (grows freely) |
| Extra memory/element | 0 | 2 pointers (8-16 bytes) |
| Implementation | Moderate (mod math) | Moderate (pointer mgmt) |

---

## 4. Input-Restricted Deque

An **input-restricted deque** allows insertion at only **one end** but deletion from **both ends**.

```
  INSERT allowed ONLY at rear:

       ✖ insertFront                           insertRear ✔
                      ┌──────┬──────┬──────┐
                      │  A   │  B   │  C   │
                      └──────┴──────┴──────┘
       deleteFront ✔                           deleteRear ✔
```

### Allowed Operations

| Operation | Allowed? |
|-----------|:--------:|
| insertFront | ✖ |
| insertRear | ✔ |
| deleteFront | ✔ |
| deleteRear | ✔ |

### Use Case

- Process elements in FIFO order normally, but occasionally need to **undo the last insertion** (deleteRear).
- Example: Text editor undo — insert characters at end, delete from either end.

### What can it generate?

An input-restricted deque with input `1, 2, 3` can generate these output permutations:

```
Input: 1, 2, 3 (always inserted at rear)

Possible outputs (by choosing deleteFront or deleteRear):
  1, 2, 3    (all deleteFront — normal queue order)
  1, 3, 2    (deleteFront 1, deleteRear 3, deleteFront 2)
  2, 1, 3    (...)
  3, 1, 2    
  3, 2, 1    (all deleteRear — stack order)

Cannot generate: 2, 3, 1
```

---

## 5. Output-Restricted Deque

An **output-restricted deque** allows deletion from only **one end** but insertion at **both ends**.

```
       insertFront ✔                           insertRear ✔
                      ┌──────┬──────┬──────┐
                      │  A   │  B   │  C   │
                      └──────┴──────┴──────┘
       deleteFront ✔                           ✖ deleteRear
```

### Allowed Operations

| Operation | Allowed? |
|-----------|:--------:|
| insertFront | ✔ |
| insertRear | ✔ |
| deleteFront | ✔ |
| deleteRear | ✖ |

### Use Case

- Priority insertion at front (urgent tasks) with normal FIFO processing.
- Example: Emergency room — critical patients inserted at front, all patients exit from front when served.

### Restricted Deque Comparison

```
┌─────────────────────────────────┬──────────────┬───────────────┐
│          Operation              │ Input-       │ Output-       │
│                                 │ Restricted   │ Restricted    │
├─────────────────────────────────┼──────────────┼───────────────┤
│ insertFront                     │      ✖       │      ✔        │
│ insertRear                      │      ✔       │      ✔        │
│ deleteFront                     │      ✔       │      ✔        │
│ deleteRear                      │      ✔       │      ✖        │
├─────────────────────────────────┼──────────────┼───────────────┤
│ Can simulate queue?             │      ✔       │      ✔        │
│ Can simulate stack?             │      ✔       │      ✔        │
│ Can simulate full deque?        │      ✖       │      ✖        │
└─────────────────────────────────┴──────────────┴───────────────┘
```

---

## 6. Sliding Window Maximum

This is the **most important deque application** for interviews and competitive programming.

### Problem Statement

Given an array of integers and a window size `k`, find the maximum element in every contiguous window of size `k`.

```
Input:  arr = [1, 3, -1, -3, 5, 3, 6, 7],  k = 3

Windows:
  [1, 3, -1] -3, 5, 3, 6, 7    → max = 3
  1, [3, -1, -3] 5, 3, 6, 7    → max = 3
  1, 3, [-1, -3, 5] 3, 6, 7    → max = 5
  1, 3, -1, [-3, 5, 3] 6, 7    → max = 5
  1, 3, -1, -3, [5, 3, 6] 7    → max = 6
  1, 3, -1, -3, 5, [3, 6, 7]   → max = 7

Output: [3, 3, 5, 5, 6, 7]
```

### Brute Force Approach

```
For each window position i:
    max = -∞
    for j = i to i + k - 1:
        max = MAX(max, arr[j])
    output max

Time: O(n * k)
```

### Optimal Approach: Monotonic Deque — O(n)

**Key Idea:** Maintain a deque of **indices** such that the values at those indices are in **decreasing order**. The front of the deque always holds the index of the current window's maximum.

```
SLIDING_WINDOW_MAX(arr, k):
    deque = empty deque (stores indices)
    result = []

    for i = 0 to n - 1:
        // 1. Remove indices outside the current window
        while deque is not empty AND deque.front() < i - k + 1:
            deque.deleteFront()

        // 2. Remove smaller elements from rear (they'll never be max)
        while deque is not empty AND arr[deque.rear()] <= arr[i]:
            deque.deleteRear()

        // 3. Add current index
        deque.insertRear(i)

        // 4. Record result once we have a full window
        if i >= k - 1:
            result.append(arr[deque.front()])

    return result
```

### Step-by-Step Trace

```
arr = [1, 3, -1, -3, 5, 3, 6, 7],  k = 3

i=0: arr[0]=1
  Deque (indices): [0]          Values: [1]
  Window not full yet.

i=1: arr[1]=3
  Remove from rear: arr[0]=1 ≤ 3 → remove 0
  Deque: [1]                    Values: [3]
  Window not full yet.

i=2: arr[2]=-1
  -1 < arr[1]=3, don't remove
  Deque: [1, 2]                 Values: [3, -1]
  Window [0..2] full → max = arr[1] = 3  ✓

i=3: arr[3]=-3
  Remove expired: 1 ≥ 3-3+1=1 → keep 1
  -3 < arr[2]=-1, don't remove
  Deque: [1, 2, 3]             Values: [3, -1, -3]
  Window [1..3] → max = arr[1] = 3  ✓

i=4: arr[4]=5
  Remove expired: 1 < 4-3+1=2 → remove 1 ✓
  Remove from rear: arr[3]=-3 ≤ 5 → remove 3
  Remove from rear: arr[2]=-1 ≤ 5 → remove 2
  Deque: [4]                    Values: [5]
  Window [2..4] → max = arr[4] = 5  ✓

i=5: arr[5]=3
  3 < arr[4]=5, don't remove
  Deque: [4, 5]                 Values: [5, 3]
  Window [3..5] → max = arr[4] = 5  ✓

i=6: arr[6]=6
  Remove from rear: arr[5]=3 ≤ 6 → remove 5
  Remove from rear: arr[4]=5 ≤ 6 → remove 4
  Deque: [6]                    Values: [6]
  Window [4..6] → max = arr[6] = 6  ✓

i=7: arr[7]=7
  Remove from rear: arr[6]=6 ≤ 7 → remove 6
  Deque: [7]                    Values: [7]
  Window [5..7] → max = arr[7] = 7  ✓

Result: [3, 3, 5, 5, 6, 7]  ✓
```

### Visual: Deque State at Each Step

```
i=0:  Deque: [ 0 ]              → 1
i=1:  Deque: [ 1 ]              → 3
i=2:  Deque: [ 1, 2 ]           → 3, -1         max=3
i=3:  Deque: [ 1, 2, 3 ]        → 3, -1, -3     max=3
i=4:  Deque: [ 4 ]              → 5              max=5
i=5:  Deque: [ 4, 5 ]           → 5, 3           max=5
i=6:  Deque: [ 6 ]              → 6              max=6
i=7:  Deque: [ 7 ]              → 7              max=7

Note: Values in deque are always DECREASING (monotonic property)
      Front always holds the maximum!
```

### Why This Works

```
  Intuition: If arr[j] ≥ arr[i] and j > i,
             then arr[i] can NEVER be the window maximum
             while arr[j] is in the window.

  So we remove arr[i] from consideration (delete from rear).

  The deque maintains indices of "potential maximums" in
  decreasing order of their values.

  ┌─────────────────────────────────────────────┐
  │ DEQUE INVARIANT:                             │
  │   arr[deque[0]] > arr[deque[1]] > ... >      │
  │   arr[deque[last]]                           │
  │                                              │
  │   All indices are within the current window   │
  └─────────────────────────────────────────────┘
```

### Complexity Analysis

| Metric | Value | Reason |
|--------|:-----:|--------|
| Time | **O(n)** | Each element enters and leaves deque at most once |
| Space | **O(k)** | Deque holds at most `k` indices |

> Despite nested while loops, the total work across all iterations is O(n) because each index is added and removed at most once (**amortized analysis**).

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Deque | Insert and delete at both front and rear |
| Simulates | Both queue (FIFO) and stack (LIFO) |
| Circular array deque | Uses mod arithmetic for both ends, O(1) all ops |
| Doubly linked list deque | Dynamic size, 2 pointers per node overhead |
| Input-restricted | Insert at one end, delete from both |
| Output-restricted | Insert at both ends, delete from one |
| Sliding window max | Monotonic deque — O(n) optimal solution |
| Monotonic property | Deque values always decreasing (for max problems) |
| Language support | Java `ArrayDeque`, Python `collections.deque`, C++ `std::deque` |

---

## Quick Revision Questions

1. **How does a deque differ from a standard queue? List all four primary operations of a deque.**

2. **Show how a deque can simulate both a queue and a stack. Which operations map to each?**

3. **In a circular array-based deque, what formula computes the new front index when inserting at front? Why is `+ capacity` needed?**

4. **What is the difference between input-restricted and output-restricted deques? Give a use case for each.**

5. **In the sliding window maximum problem, why do we remove elements from the rear of the deque when a larger element arrives?**

6. **What is the time complexity of the sliding window maximum using a deque? Why isn't it O(n*k) despite the nested while loops?**

---

## Answers (Hidden — Think First!)

<details>
<summary>Click to reveal answers</summary>

1. A standard queue allows insert at rear and delete from front only. A deque allows **insertFront, insertRear, deleteFront, deleteRear** — four operations instead of two.

2. **Queue:** `enqueue(x) → insertRear(x)`, `dequeue() → deleteFront()`. **Stack:** `push(x) → insertRear(x)`, `pop() → deleteRear()` (both operations on the same end).

3. `front = (front - 1 + capacity) % capacity`. Adding `capacity` prevents negative results from the modulo operation. Without it, `(0 - 1) % 5` might return `-1` in some languages instead of `4`.

4. **Input-restricted:** Insert at rear only, delete from both ends. Use case: undo last insertion (deleteRear) while processing in order (deleteFront). **Output-restricted:** Insert at both ends, delete from front only. Use case: emergency room — urgent patients inserted at front, normal at rear, all served from front.

5. If a new element `arr[i]` is larger than `arr[deque.rear()]`, the rear element can **never** be the maximum for any future window that includes `arr[i]`. It's permanently overshadowed, so we discard it to keep only potential maximums.

6. **O(n)**, not O(n*k). Each element enters the deque exactly once and leaves exactly once across all iterations. The total number of insertions + deletions is at most `2n`, giving O(n) amortized. The while loops don't re-process elements — they just clean up previously added elements.

</details>

---

[← Previous: Circular Queue](../03-Circular-Queue/01-circular-queue.md) | [Back to README](../README.md) | [Next: Priority Queue →](../05-Priority-Queue/01-priority-queue.md)
