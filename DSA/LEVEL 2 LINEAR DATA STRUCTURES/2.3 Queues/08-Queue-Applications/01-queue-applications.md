# Unit 8: Queue Applications

[← Previous: Monotonic Queue](../07-Monotonic-Queue/01-monotonic-queue.md) | [Back to README](../README.md)

---

## Chapter Overview

Queues power some of the most important algorithms and systems in computer science. This unit provides **previews** of BFS and level-order traversal (covered in depth in later modules), and explores real-world applications including process scheduling, print queues, message queues, and task scheduling.

---

## 1. BFS Traversal (Preview)

**Breadth-First Search (BFS)** explores a graph level by level — visiting all neighbors of a node before moving deeper. The queue is the engine behind BFS.

### Why Queue for BFS?

```
  Queue = FIFO = Process closest nodes first

  If you discover node B and C from A:
    Queue: [B, C]
    Process B first (same level as C), THEN go deeper.

  If you used a Stack (LIFO) instead:
    Stack: [C, B]  (B on top)
    Pop B → explore B's children → go DEEP → that's DFS!
```

### BFS Algorithm

```
BFS(graph, start):
    visited = set()
    queue = empty queue
    
    queue.enqueue(start)
    visited.add(start)
    
    while queue is not empty:
        node = queue.dequeue()
        PROCESS(node)
        
        for each neighbor of node:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.enqueue(neighbor)
```

### BFS Trace on a Graph

```
  Graph:
        A
       / \
      B   C
     / \   \
    D   E   F

  BFS from A:

  Step  │ Dequeue │ Enqueue     │ Queue State     │ Visited
  ──────┼─────────┼─────────────┼─────────────────┼──────────
  Init  │    -    │ A           │ [A]             │ {A}
    1   │    A    │ B, C        │ [B, C]          │ {A,B,C}
    2   │    B    │ D, E        │ [C, D, E]       │ {A,B,C,D,E}
    3   │    C    │ F           │ [D, E, F]       │ {A,B,C,D,E,F}
    4   │    D    │ (none)      │ [E, F]          │ same
    5   │    E    │ (none)      │ [F]             │ same
    6   │    F    │ (none)      │ []              │ same

  BFS Order: A → B → C → D → E → F
  (Level by level: Level 0: A, Level 1: B,C, Level 2: D,E,F)
```

### BFS vs DFS

```
  BFS (Queue):                    DFS (Stack/Recursion):
  
        A                               A
       / \                             / \
      B   C                           B   C
     / \   \                         / \   \
    D   E   F                       D   E   F

  Visit order:                    Visit order:
  A, B, C, D, E, F               A, B, D, E, C, F
  (level by level)                (branch by branch)
```

### BFS Applications

| Application | Why BFS? |
|-------------|----------|
| Shortest path (unweighted) | BFS finds shortest path by levels |
| Connected components | Explore all reachable nodes |
| Web crawling | Visit pages level by level |
| Social network (degrees) | Find people N connections away |
| Puzzle solving | Find minimum moves |

### Complexity

| Metric | Value |
|--------|:-----:|
| Time | O(V + E) — visit every vertex and edge once |
| Space | O(V) — queue + visited set |

---

## 2. Level Order Traversal (Preview)

**Level-order traversal** visits all nodes of a binary tree level by level, from left to right. It's BFS applied to trees.

### Algorithm

```
LEVEL_ORDER(root):
    if root == NULL: return
    queue = empty queue
    queue.enqueue(root)
    
    while queue is not empty:
        node = queue.dequeue()
        PROCESS(node)
        
        if node.left != NULL:
            queue.enqueue(node.left)
        if node.right != NULL:
            queue.enqueue(node.right)
```

### Level-by-Level Variant (Track Levels)

```
LEVEL_ORDER_BY_LEVEL(root):
    if root == NULL: return
    queue = empty queue
    queue.enqueue(root)
    level = 0
    
    while queue is not empty:
        levelSize = queue.size()      // # nodes at current level
        print "Level " + level + ":"
        
        for i = 0 to levelSize - 1:
            node = queue.dequeue()
            print node.value
            
            if node.left:  queue.enqueue(node.left)
            if node.right: queue.enqueue(node.right)
        
        level = level + 1
```

### Trace

```
  Tree:
           1
         /   \
        2     3
       / \     \
      4   5     6

  Queue trace:

  Level 0:
    Queue: [1]
    Process: 1 → enqueue 2, 3
    Queue becomes: [2, 3]

  Level 1: (levelSize = 2)
    Queue: [2, 3]
    Process: 2 → enqueue 4, 5
    Process: 3 → enqueue 6
    Queue becomes: [4, 5, 6]

  Level 2: (levelSize = 3)
    Queue: [4, 5, 6]
    Process: 4, 5, 6 → no children
    Queue becomes: []

  Output:
    Level 0: [1]
    Level 1: [2, 3]
    Level 2: [4, 5, 6]
```

### Level Order Variants

| Variant | Modification |
|---------|--------------|
| Zigzag traversal | Alternate left→right and right→left per level |
| Bottom-up level order | Reverse the result |
| Right side view | Last node of each level |
| Left side view | First node of each level |
| Average of levels | Compute average per level |
| Largest in each level | Max per level |

---

## 3. Process Scheduling

Operating systems use queues extensively for CPU scheduling.

### Round Robin Scheduling

The most common FIFO-based CPU scheduling algorithm.

```
  ┌─────────────────────────────────────────────────────┐
  │              ROUND ROBIN SCHEDULING                  │
  │              Time Quantum (TQ) = 4                   │
  │                                                      │
  │  Ready Queue:                                        │
  │  ┌────┬────┬────┐                                   │
  │  │ P1 │ P2 │ P3 │                                   │
  │  │ 10 │  4 │  6 │  (remaining burst times)          │
  │  └────┴────┴────┘                                   │
  │                                                      │
  │  CPU executes front process for TQ units,            │
  │  then re-enqueues if not finished.                   │
  └─────────────────────────────────────────────────────┘
```

### Round Robin Trace

```
Processes: P1(burst=10), P2(burst=4), P3(burst=6)
Time Quantum: TQ = 4

  Time │ Queue           │ Running │ Remaining    │ Event
  ─────┼─────────────────┼─────────┼──────────────┼───────────
    0  │ [P1,P2,P3]      │ P1      │ P1:10→6      │ P1 runs 4
    4  │ [P2,P3,P1]      │ P2      │ P2:4→0       │ P2 finishes!
    8  │ [P3,P1]         │ P3      │ P3:6→2       │ P3 runs 4
   12  │ [P1,P3]         │ P1      │ P1:6→2       │ P1 runs 4
   16  │ [P3,P1]         │ P3      │ P3:2→0       │ P3 finishes!
   18  │ [P1]            │ P1      │ P1:2→0       │ P1 finishes!
   20  │ []              │ -       │ All done     │

  Gantt Chart:
  ┌────────┬────────┬────────┬────────┬────┬────┐
  │   P1   │   P2   │   P3   │   P1   │ P3 │ P1 │
  ├────────┼────────┼────────┼────────┼────┼────┤
  0        4        8       12       16   18   20

  Turnaround Times:
    P1: 20 - 0 = 20
    P2:  8 - 0 =  8
    P3: 18 - 0 = 18
    Average = (20 + 8 + 18) / 3 = 15.33

  Waiting Times:
    P1: 20 - 10 = 10
    P2:  8 -  4 =  4
    P3: 18 -  6 = 12
    Average = (10 + 4 + 12) / 3 = 8.67
```

### Pseudocode

```
ROUND_ROBIN(processes[], TQ):
    queue = new Queue()
    for each p in processes:
        queue.enqueue(p)
    
    time = 0
    while queue is not empty:
        p = queue.dequeue()
        
        if p.remaining > TQ:
            // Run for time quantum, then re-enqueue
            time = time + TQ
            p.remaining = p.remaining - TQ
            queue.enqueue(p)
        else:
            // Process finishes
            time = time + p.remaining
            p.remaining = 0
            p.completionTime = time
```

### Multi-Level Queue Scheduling

```
  ┌────────────────────────────────┐
  │  HIGH PRIORITY QUEUE           │  ← Served first (system processes)
  │  [P1, P5, P8]                 │
  ├────────────────────────────────┤
  │  MEDIUM PRIORITY QUEUE         │  ← Served when high is empty
  │  [P2, P6]                     │
  ├────────────────────────────────┤
  │  LOW PRIORITY QUEUE            │  ← Served last (batch jobs)
  │  [P3, P4, P7]                 │
  └────────────────────────────────┘

  Each queue may use different scheduling:
    High: Round Robin (TQ=2)
    Medium: Round Robin (TQ=4)
    Low: FCFS (First Come First Served)
```

---

## 4. Print Queue

A print queue manages print jobs submitted to a printer, processing them in order.

### Print Queue Simulation

```
  ┌──────────────┐     ┌─────────────────────┐     ┌──────────┐
  │   Users      │     │    PRINT QUEUE       │     │ PRINTER  │
  │              │     │                      │     │          │
  │  User A ─────┼────→│ Job1 │ Job2 │ Job3  │────→│ Printing │
  │  User B ─────┤     │                      │     │ Job 1... │
  │  User C ─────┘     └─────────────────────┘     └──────────┘
  └──────────────┘
```

### Priority Print Queue Problem

**Problem:** Given print jobs with priorities, find how long until your job is printed (only the highest priority job prints next).

```
PRINT_QUEUE_WAIT(jobs[], myIndex):
    queue = new Queue()
    for i = 0 to len(jobs) - 1:
        queue.enqueue((jobs[i], i))    // (priority, original_index)
    
    time = 0
    while queue is not empty:
        front = queue.peek()
        
        // Check if any job in queue has higher priority
        hasHigher = false
        for each job in queue:
            if job.priority > front.priority:
                hasHigher = true
                break
        
        if hasHigher:
            // Move front to back (not highest priority)
            queue.enqueue(queue.dequeue())
        else:
            // Print this job (highest priority)
            printed = queue.dequeue()
            time = time + 1
            if printed.originalIndex == myIndex:
                return time
```

### Trace

```
  Jobs: [2, 1, 3, 2],  myIndex = 2  (Job with priority 3)

  Queue: [(2,0), (1,1), (3,2), (2,3)]

  Step 1: Front=(2,0). Higher exists? (3,2) YES → move to back
          [(1,1), (3,2), (2,3), (2,0)]

  Step 2: Front=(1,1). Higher exists? YES → move to back
          [(3,2), (2,3), (2,0), (1,1)]

  Step 3: Front=(3,2). Higher exists? NO → PRINT! time=1
          Is it myIndex=2? YES → Answer: 1
```

---

## 5. Message Queues

Message queues enable **asynchronous communication** between distributed system components.

### Concept

```
  ┌──────────┐     ┌───────────────────┐     ┌──────────┐
  │ PRODUCER │     │   MESSAGE QUEUE    │     │ CONSUMER │
  │ (Sender) │────→│                   │────→│(Receiver)│
  │          │     │ [M1][M2][M3][M4]  │     │          │
  └──────────┘     └───────────────────┘     └──────────┘
                   
  Producer and Consumer are DECOUPLED:
  - Producer adds messages without waiting for consumer
  - Consumer processes at its own pace
  - Queue buffers the difference in speed
```

### Message Queue Patterns

#### Point-to-Point (Queue)

```
  One producer, one consumer. Each message consumed once.

  Producer ──→ [M1][M2][M3] ──→ Consumer
```

#### Publish-Subscribe (Topic)

```
  One producer, multiple consumers. Each gets a copy.

  Producer ──→ [M1][M2][M3] ──→ Consumer A
                              ──→ Consumer B
                              ──→ Consumer C
```

#### Work Queue (Fan-Out)

```
  One producer, multiple workers. Load balanced.

  Producer ──→ [M1][M2][M3][M4][M5][M6]
                    │            │
                    ├──→ Worker 1 (gets M1, M4)
                    ├──→ Worker 2 (gets M2, M5)
                    └──→ Worker 3 (gets M3, M6)
```

### Real-World Message Queue Systems

| System | Type | Use Case |
|--------|------|----------|
| RabbitMQ | General purpose | Task queues, RPC |
| Apache Kafka | Log-based | Event streaming, analytics |
| Amazon SQS | Cloud managed | Decoupling microservices |
| Redis Queue | In-memory | Real-time processing |
| ZeroMQ | Library | High-performance messaging |

### Properties of Message Queues

| Property | Description |
|----------|-------------|
| **Durability** | Messages persist even if consumers are down |
| **Ordering** | FIFO within a single queue/partition |
| **At-least-once** | Messages delivered at least once |
| **Acknowledgment** | Consumer confirms processing |
| **Dead letter** | Failed messages moved to separate queue |
| **TTL** | Messages expire after a time limit |

### Simple Message Queue Simulation

```
class MessageQueue:
    queue = []
    
    PRODUCE(message):
        queue.enqueue(message)
        log("Produced: " + message.id)
    
    CONSUME():
        if queue.isEmpty():
            WAIT()           // Block or return null
        message = queue.dequeue()
        PROCESS(message)
        ACKNOWLEDGE(message)
        log("Consumed: " + message.id)
    
    // With acknowledgment:
    CONSUME_WITH_ACK():
        message = queue.dequeue()
        try:
            PROCESS(message)
            ACK(message)      // Remove permanently
        catch error:
            NACK(message)     // Re-enqueue for retry
            queue.enqueue(message)
```

---

## 6. Task Scheduling

Task scheduling uses queues to manage job execution order, dependencies, and resource allocation.

### Simple Task Scheduler

```
  ┌────────────────────────────────────────────────┐
  │              TASK SCHEDULER                     │
  │                                                 │
  │  Task Queue:                                    │
  │  ┌──────┬──────┬──────┬──────┐                 │
  │  │ T1   │ T2   │ T3   │ T4   │                 │
  │  │ 3ms  │ 5ms  │ 2ms  │ 4ms  │                 │
  │  └──────┴──────┴──────┴──────┘                 │
  │            │                                    │
  │            ▼                                    │
  │  ┌──────────────┐                              │
  │  │   EXECUTOR   │                              │
  │  │  Runs T1...  │                              │
  │  └──────────────┘                              │
  └────────────────────────────────────────────────┘
```

### Task Scheduler with Cooldown (LeetCode 621 Pattern)

**Problem:** Given tasks with cooldown period `n` — after executing a task, you must wait `n` intervals before executing the same task again. Find minimum intervals needed.

```
TASK_SCHEDULER(tasks[], n):
    freq = count frequency of each task
    maxFreq = maximum frequency
    maxCount = number of tasks with max frequency
    
    // Formula approach:
    // (maxFreq - 1) * (n + 1) + maxCount
    // But take max with total task count (no idle slots needed)
    
    result = max(len(tasks), (maxFreq - 1) * (n + 1) + maxCount)
    return result
```

### Trace

```
  Tasks: [A, A, A, B, B, B, C, C],  cooldown n = 2

  Frequencies: A:3, B:3, C:2
  maxFreq = 3, maxCount = 2 (A and B both have freq 3)

  Formula: (3-1) * (2+1) + 2 = 2 * 3 + 2 = 8

  Schedule:
  ┌───┬───┬───┬───┬───┬───┬───┬───┐
  │ A │ B │ C │ A │ B │ C │ A │ B │
  └───┴───┴───┴───┴───┴───┴───┴───┘
    1   2   3   4   5   6   7   8

  Visualization of formula:
  
  (maxFreq - 1) groups of (n + 1) slots + maxCount tail:
  
  ┌─────────────┬─────────────┬─────┐
  │  A  B  C    │  A  B  C    │ A B │
  │  group 1    │  group 2    │tail │
  └─────────────┴─────────────┴─────┘
    (n+1)=3        (n+1)=3     maxCount=2
  
  Total = 2 × 3 + 2 = 8 intervals  ✓
```

### Task DAG (Dependency Graph) Scheduling

Tasks with dependencies use **topological sort** powered by a queue (Kahn's algorithm).

```
  Task Dependencies:
    A → C
    B → C
    B → D
    C → E
    D → E

  DAG:
    A ──→ C ──┐
              ├──→ E
    B ──→ D ──┘
    │
    └──→ C

  Kahn's Algorithm (BFS Topological Sort):

  1. Compute in-degrees:
     A:0, B:0, C:2, D:1, E:2

  2. Enqueue nodes with in-degree 0:
     Queue: [A, B]

  3. Process:
     Dequeue A: reduce C's in-degree (C:2→1). Queue: [B]
     Dequeue B: reduce C's in-degree (C:1→0), D's (D:1→0).
                Enqueue C, D. Queue: [C, D]
     Dequeue C: reduce E's in-degree (E:2→1). Queue: [D]
     Dequeue D: reduce E's in-degree (E:1→0).
                Enqueue E. Queue: [E]
     Dequeue E: done. Queue: []

  Topological order: A, B, C, D, E
  (Valid execution order respecting dependencies)
```

### Application Map

```
  ┌──────────────────────────────────────────────────┐
  │               QUEUE IN THE REAL WORLD             │
  │                                                   │
  │  ┌─────────────┐    ┌───────────────┐            │
  │  │ Web Server  │    │ Load Balancer │            │
  │  │ Request Q   │    │ Work Queue    │            │
  │  └──────┬──────┘    └──────┬────────┘            │
  │         │                  │                      │
  │         ▼                  ▼                      │
  │  ┌─────────────┐    ┌───────────────┐            │
  │  │ Event Loop  │    │ Message Queue │            │
  │  │ (JS,Python) │    │ (RabbitMQ)    │            │
  │  └──────┬──────┘    └──────┬────────┘            │
  │         │                  │                      │
  │         ▼                  ▼                      │
  │  ┌─────────────┐    ┌───────────────┐            │
  │  │  OS Kernel  │    │  Database     │            │
  │  │ Ready Queue │    │ Query Queue   │            │
  │  └─────────────┘    └───────────────┘            │
  └──────────────────────────────────────────────────┘
```

---

## Summary Table

| Application | Queue Type | Key Idea |
|-------------|:----------:|----------|
| BFS | Standard queue | Explore graph level by level |
| Level-order traversal | Standard queue | Visit tree nodes per level |
| Round Robin scheduling | Circular queue | Fair CPU time slicing |
| Multi-level scheduling | Multiple queues | Priority-based process classes |
| Print queue | Priority queue | Highest priority prints first |
| Message queues | Persistent queue | Decouple producer and consumer |
| Pub-Sub | Topic queue | Broadcast to multiple consumers |
| Task scheduling | Standard/priority | Execute tasks in order |
| Topological sort | BFS queue | Process DAG in dependency order |
| Event loop | Event queue | Process events in arrival order |

---

## Quick Revision Questions

1. **Why does BFS use a queue instead of a stack? What would happen if you replaced the queue with a stack?**

2. **In level-order traversal, how do you determine when one level ends and the next begins?**

3. **In Round Robin scheduling, what happens when the time quantum is very large vs very small?**

4. **What is the main advantage of using message queues in distributed systems?**

5. **How does Kahn's algorithm for topological sort use a queue? What does in-degree represent?**

6. **In the task scheduler with cooldown problem, why does the formula `(maxFreq-1) * (n+1) + maxCount` work?**

---

## Answers (Hidden — Think First!)

<details>
<summary>Click to reveal answers</summary>

1. BFS explores **level by level** — all nodes at distance `d` before distance `d+1`. A queue's FIFO property ensures nodes discovered first (closer) are processed first. Replacing the queue with a **stack** would give **DFS (Depth-First Search)** — it would go deep into one branch before exploring siblings.

2. Record `levelSize = queue.size()` at the start of each level. Then dequeue exactly `levelSize` nodes — they all belong to the current level. Their children (enqueued during this process) form the next level.

3. **Very large TQ** → Each process finishes before preemption → degenerates to **FCFS** (First Come First Served), poor response time for short jobs. **Very small TQ** → Frequent context switches → high overhead. The ideal TQ balances responsiveness with low overhead (typically 10-100ms in real systems).

4. **Decoupling:** Producers and consumers don't need to run simultaneously or know about each other. Benefits: (1) handle traffic spikes (queue buffers), (2) independent scaling, (3) fault tolerance (messages persist if consumer crashes), (4) async processing (producer doesn't wait).

5. Start by computing **in-degree** (number of incoming edges) for each node. Enqueue all nodes with in-degree 0 (no dependencies). Dequeue a node, process it, and reduce in-degrees of its neighbors. When a neighbor's in-degree reaches 0, enqueue it. In-degree represents the number of **unresolved dependencies**.

6. The most frequent task(s) create the bottleneck. Between consecutive executions of the same task, we need `n+1` slots (the task itself + `n` cooldown slots). For `maxFreq` occurrences, we need `(maxFreq-1)` such groups. The final occurrence doesn't need cooldown, contributing `maxCount` (number of tasks at max frequency). We take `max(totalTasks, formula)` because when there are many distinct tasks, idle slots aren't needed.

</details>

---

## Course Wrap-Up

Congratulations on completing the Queues module! Here's what you've mastered:

```
  ┌────────────────────────────────────────────────────┐
  │  YOUR QUEUE KNOWLEDGE MAP                           │
  │                                                     │
  │  ✓ Unit 1: Fundamentals — FIFO, ADT, analogies     │
  │  ✓ Unit 2: Implementation — Array, LL, tradeoffs   │
  │  ✓ Unit 3: Circular Queue — Ring buffer, mod math  │
  │  ✓ Unit 4: Deque — Both-end access, sliding window │
  │  ✓ Unit 5: Priority Queue — Heaps, algorithms      │
  │  ✓ Unit 6: Queue Problems — Classic interview Qs    │
  │  ✓ Unit 7: Monotonic Queue — DP optimization        │
  │  ✓ Unit 8: Applications — BFS, scheduling, MQ      │
  └────────────────────────────────────────────────────┘
```

---

[← Previous: Monotonic Queue](../07-Monotonic-Queue/01-monotonic-queue.md) | [Back to README](../README.md)
