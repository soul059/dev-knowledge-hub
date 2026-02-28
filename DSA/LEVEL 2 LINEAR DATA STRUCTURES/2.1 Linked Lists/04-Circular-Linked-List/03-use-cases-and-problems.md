# Chapter 3: Use Cases and Classic Problems

[← Previous: Circular List Operations](02-circular-list-operations.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Unit 5 — Concept and Cycle Detection →](../05-Fast-And-Slow-Pointers/01-concept-and-cycle-detection.md)

---

## Chapter Overview

This chapter covers **real-world applications** of circular linked lists and the classic **Josephus Problem** — a famous algorithmic puzzle naturally solved with circular lists.

---

## 3.1 Round Robin Scheduling

The most common application of circular linked lists.

### Concept

In **Round Robin (RR) CPU scheduling**, each process gets a fixed time slice (quantum). After the time expires, the process moves to the back and the next process runs.

```
  ┌──► [P1] ──► [P2] ──► [P3] ──► [P4] ──┐
  │                                         │
  └─────────────────────────────────────────┘
                    ↑
                 current

  Time quantum = 4ms
  
  Step 1: Run P1 for 4ms → move to P2
  Step 2: Run P2 for 4ms → move to P3
  Step 3: Run P3 for 4ms → move to P4
  Step 4: Run P4 for 4ms → move to P1 (circular!)
  Step 5: Run P1 for 4ms → ...
  
  If P2 finishes: remove from circle, continue with P3
  If new process P5 arrives: insert into circle
```

### Why Circular List?

```
  With regular list:
  ┌─ After reaching last process, must reset to first  → extra logic
  └─ "Is current the last?" check needed every time

  With circular list:
  ┌─ Just do: current = current.next
  └─ Naturally loops — no boundary checking needed!
```

### Pseudocode: Round Robin Simulator

```
function roundRobin(TAIL, quantum):
    if TAIL == NULL: return
    
    current = TAIL.next              ← Start at first process
    
    while TAIL.next ≠ TAIL or current.remainingTime > 0:
        if current.remainingTime ≤ quantum:
            print("Process", current.id, "completed")
            next = current.next
            TAIL = deleteNode(TAIL, current)
            current = next
        else:
            current.remainingTime -= quantum
            print("Process", current.id, "ran for", quantum, "ms")
            current = current.next
    
    print("All processes completed")
```

---

## 3.2 The Josephus Problem

### Problem Statement

`n` people stand in a circle. Starting from person 1, every `k`-th person is eliminated. Find the survivor.

### Example: n=7, k=3

```
  Initial circle (7 people):

     1
   7   2
  6     3
   5   4

  Start at 1, count k=3:
  
  Round 1: Count 1→2→3 → Eliminate 3
     1
   7   2
  6     
   5   4

  Round 2: Count 4→5→6 → Eliminate 6
     1
   7   2
  
   5   4

  Round 3: Count 7→1→2 → Eliminate 2
     1
   7   
  
   5   4

  Round 4: Count 4→5→7 → Eliminate 7
     1
   
  
   5   4

  Round 5: Count 1→4→5 → Eliminate 5
     1
  
  
      4

  Round 6: Count 1→4→1 → Eliminate 1
  
  
  
      4

  Survivor: Person 4! ✓
```

### Solution Using Circular Linked List

```
function josephus(n, k):
    // Build circular list: 1 → 2 → 3 → ... → n → 1
    HEAD = createNode(1)
    current = HEAD
    for i = 2 to n:
        current.next = createNode(i)
        current = current.next
    current.next = HEAD              ← Make it circular
    
    // TAIL starts at the last node
    prev = current                   ← prev is the node before current
    current = HEAD
    
    while current.next ≠ current:    ← Until one person remains
        // Count k-1 steps (current is person 1)
        for i = 1 to k - 1:
            prev = current
            current = current.next
        
        // Eliminate current person
        print("Eliminated:", current.data)
        prev.next = current.next     ← Bypass current
        temp = current
        current = current.next       ← Move to next person
        free(temp)
    
    print("Survivor:", current.data)
    return current.data
```

### Step-by-Step Trace: n=5, k=2

```
  Circle: 1 → 2 → 3 → 4 → 5 → (back to 1)

  Round 1: Start at 1, count 2: 1→2 → Eliminate 2
  Circle: 1 → 3 → 4 → 5 → (back to 1)

  Round 2: Start at 3, count 2: 3→4 → Eliminate 4
  Circle: 1 → 3 → 5 → (back to 1)

  Round 3: Start at 5, count 2: 5→1 → Eliminate 1
  Circle: 3 → 5 → (back to 3)

  Round 4: Start at 3, count 2: 3→5 → Eliminate 5
  Circle: 3 → (back to 3)

  Survivor: 3
```

### Mathematical Solution (O(n) without linked list)

```
function josephusMath(n, k):
    survivor = 0                     ← Base case: 1 person → index 0
    for i = 2 to n:
        survivor = (survivor + k) % i
    return survivor + 1              ← Convert 0-indexed to 1-indexed
```

| Approach | Time | Space |
|----------|------|-------|
| Circular Linked List | O(nk) | O(n) |
| Mathematical formula | O(n) | O(1) |
| Recursive formula | O(n) | O(n) stack |

---

## 3.3 Circular Buffer / Ring Buffer

A **ring buffer** uses a fixed-size circular array, but the concept maps to circular linked lists:

```
  Write pointer
       │
       ▼
  ┌──► [A] ──► [B] ──► [C] ──► [_] ──► [_] ──┐
  │                               ↑              │
  └───────────────────────────────┘──────────────┘
                              Read pointer

  Producer writes at write pointer, advances
  Consumer reads at read pointer, advances
  
  When write catches up to read → buffer full
  When read catches up to write → buffer empty
```

### Applications

- Audio/video streaming buffers
- Network packet buffers
- Keyboard input buffers
- Producer-consumer queues

---

## 3.4 Multiplayer Game Turn Management

```
  ┌──► [Player1] ──► [Player2] ──► [Player3] ──► [Player4] ──┐
  │                                                             │
  └─────────────────────────────────────────────────────────────┘
                           ↑
                        current turn

  After Player2's turn:
    current = current.next → Player3's turn
  
  Player3 is eliminated:
    Delete Player3 from circle
    current = current.next → Player4's turn
  
  Natural cycling: after Player4 → back to Player1
```

---

## 3.5 Music Playlist (Repeat All Mode)

```
  ┌──► [Song1] ──► [Song2] ──► [Song3] ──► [Song4] ──┐
  │                                                      │
  └──────────────────────────────────────────────────────┘
                      ↑
                 now playing

  Next button: current = current.next
  Previous:    Need DCLL for O(1) backward
  
  Shuffle: Randomly pick a node
  Remove song: Delete node from circle
  Add song: Insert node into circle
```

---

## 3.6 Token Ring Network Protocol

```
  ┌──► [PC1] ──► [PC2] ──► [PC3] ──► [PC4] ──┐
  │                                              │
  └──────────────────────────────────────────────┘

  A "token" circulates around the ring.
  Only the PC holding the token can transmit data.
  After transmitting, pass the token to the next PC.
  
  Implemented using circular linked list:
  - token pointer moves: token = token.next
  - PC joins: insert into circle
  - PC leaves: delete from circle
```

---

## 3.7 Split and Merge Circular Lists

### Split a Circular List into Two Halves

Use the **slow-fast pointer** technique:

```
function splitCircular(HEAD):
    if HEAD == NULL: return NULL, NULL
    
    slow = HEAD
    fast = HEAD
    
    while fast.next ≠ HEAD and fast.next.next ≠ HEAD:
        slow = slow.next
        fast = fast.next.next
    
    // slow is at the midpoint
    HEAD2 = slow.next
    
    // Find the last node of the second half
    if fast.next.next == HEAD:
        fast = fast.next
    // fast.next should be HEAD
    
    fast.next = HEAD2             ← Second half becomes circular
    slow.next = HEAD              ← First half becomes circular
    
    return HEAD, HEAD2
```

### Merge Two Circular Lists

```
function mergeCircular(TAIL1, TAIL2):
    if TAIL1 == NULL: return TAIL2
    if TAIL2 == NULL: return TAIL1
    
    // Save HEAD pointers
    HEAD1 = TAIL1.next
    HEAD2 = TAIL2.next
    
    // Connect: TAIL1 → HEAD2, TAIL2 → HEAD1
    TAIL1.next = HEAD2
    TAIL2.next = HEAD1
    
    return TAIL2                  ← New TAIL is TAIL2
```

### Diagram: Merging

```
  Before:
  Circle 1: ┌──► [1] ──► [2] ──► [3] ──┐
             └──────────────────────────┘
             
  Circle 2: ┌──► [4] ──► [5] ──┐
             └─────────────────┘
  
  After (connect TAIL1→HEAD2 and TAIL2→HEAD1):
  
  ┌──► [1] ──► [2] ──► [3] ──► [4] ──► [5] ──┐
  │                                             │
  └─────────────────────────────────────────────┘
```

---

## 3.8 Problem-Solving Patterns for Circular Lists

| Pattern | When to Use |
|---------|-------------|
| **TAIL pointer management** | Always — prefer TAIL over HEAD |
| **do-while traversal** | Any traversal on circular list |
| **Single-node check** | `node.next == node` before deletion |
| **Count steps** | Josephus-type problems |
| **Slow-fast pointers** | Split circular list |
| **Save HEAD before modifying** | Before any structural change |

---

## Summary Table

| Application | Why Circular? |
|-------------|--------------|
| Round Robin scheduling | Natural cycling, no boundary check |
| Josephus problem | Continuous elimination in a circle |
| Ring buffer | Fixed-size cyclic data stream |
| Multiplayer games | Turn cycling |
| Music playlist (repeat) | Continuous loop playback |
| Token ring network | Token passes around in circle |

| Problem Type | Complexity |
|-------------|-----------|
| Josephus (linked list) | O(nk) time, O(n) space |
| Josephus (math formula) | O(n) time, O(1) space |
| Split circular list | O(n) time, O(1) space |
| Merge circular lists | O(1) time, O(1) space |

---

## Quick Revision Questions

1. **Why is a circular linked list ideal for round-robin scheduling?**
   <details><summary>Answer</summary>After the last process, `current.next` automatically points to the first process. No boundary checking or resetting is needed — the circular structure provides natural cycling.</details>

2. **In the Josephus problem with n=6 and k=2, what order are people eliminated?**
   <details><summary>Answer</summary>Elimination order: 2, 4, 6, 3, 1. Survivor: 5. (Count 2 from each current starting position.)</details>

3. **What is the time complexity of the Josephus problem using a circular linked list?**
   <details><summary>Answer</summary>O(nk) — for each of n-1 eliminations, we count k steps. The mathematical formula solves it in O(n).</details>

4. **How do you merge two circular linked lists in O(1)?**
   <details><summary>Answer</summary>Save HEAD1 = TAIL1.next and HEAD2 = TAIL2.next. Then set TAIL1.next = HEAD2 and TAIL2.next = HEAD1. Only 2 pointer changes!</details>

5. **What is a ring buffer and why use circular structure?**
   <details><summary>Answer</summary>A ring buffer is a fixed-size buffer where the write position wraps around to the beginning when it reaches the end. Circular structure avoids copying or shifting data — the producer and consumer just advance their pointers.</details>

6. **How do you split a circular linked list into two equal halves?**
   <details><summary>Answer</summary>Use slow-fast pointers: slow moves 1 step, fast moves 2 steps. When fast returns to HEAD (or its next is HEAD), slow is at the midpoint. Break the circular links at the midpoint to create two circular lists.</details>

---

[← Previous: Circular List Operations](02-circular-list-operations.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Unit 5 — Concept and Cycle Detection →](../05-Fast-And-Slow-Pointers/01-concept-and-cycle-detection.md)
