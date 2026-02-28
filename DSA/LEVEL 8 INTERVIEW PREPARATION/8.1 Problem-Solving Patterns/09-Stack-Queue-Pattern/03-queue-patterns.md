# Chapter 3: Queue Patterns

## 📋 Chapter Overview
Queue fundamentals, deque techniques, sliding window maximum, and design problems using stacks and queues.

---

## 🧠 Queue — FIFO Principle

```
  ENQUEUE 1   ENQUEUE 2   ENQUEUE 3   DEQUEUE → 1
  
  Front→ ┌─┬─┬─┐ ←Back
         │1│ │ │            │1│2│ │          │1│2│3│         │2│3│ │
         └─┴─┴─┘            └─┴─┴─┘          └─┴─┴─┘         └─┴─┴─┘
  
  First In, First Out
  Enqueue/Dequeue: O(1)
```

---

## 📝 Problem 1: Sliding Window Maximum (LeetCode 239)

**Problem:** Given array and window size k, find max in each window position.

```
  nums = [1, 3, -1, -3, 5, 3, 6, 7], k = 3
  
  Window position          Max
  [1  3  -1] -3  5  3  6  7    → 3
   1 [3  -1  -3] 5  3  6  7    → 3
   1  3 [-1  -3  5] 3  6  7    → 5
   1  3  -1 [-3  5  3] 6  7    → 5
   1  3  -1  -3 [5  3  6] 7    → 6
   1  3  -1  -3  5 [3  6  7]   → 7
```

### Pseudocode (Monotonic Deque)

```
function maxSlidingWindow(nums, k):
    deque = []   // stores indices, decreasing by value
    result = []
    
    for i = 0 to n-1:
        // Remove elements outside window
        while deque not empty and deque.front() < i - k + 1:
            deque.pollFirst()
        
        // Remove smaller elements (they can't be max)
        while deque not empty and nums[deque.back()] < nums[i]:
            deque.pollLast()
        
        deque.addLast(i)
        
        // Window is full
        if i >= k - 1:
            result.add(nums[deque.front()])
    
    return result
```

### Trace

```
  nums = [1, 3, -1, -3, 5, 3, 6, 7], k = 3
  
  i=0 (1): deque=[0]
  i=1 (3): 3>1 → remove 0. deque=[1]
  i=2 (-1): deque=[1,2]. Window full. max=nums[1]=3. result=[3]
  i=3 (-3): deque=[1,2,3]. max=nums[1]=3. result=[3,3]
  i=4 (5): 5>-3→rem 3. 5>-1→rem 2. 5>3→rem 1. deque=[4]. max=5. result=[3,3,5]
  i=5 (3): deque=[4,5]. max=nums[4]=5. result=[3,3,5,5]
  i=6 (6): 6>3→rem 5. 6>5→rem 4. deque=[6]. max=6. result=[3,3,5,5,6]
  i=7 (7): 7>6→rem 6. deque=[7]. max=7. result=[3,3,5,5,6,7]
```

**Complexity:** O(n) time (each index added/removed at most once), O(k) space

---

## 📝 Problem 2: Implement Queue using Stacks (LeetCode 232)

```
class MyQueue:
    pushStack = []
    popStack = []
    
    push(x):
        pushStack.push(x)
    
    pop():
        if popStack is empty:
            while pushStack not empty:
                popStack.push(pushStack.pop())
        return popStack.pop()
    
    peek():
        if popStack is empty:
            while pushStack not empty:
                popStack.push(pushStack.pop())
        return popStack.top()
    
    empty():
        return pushStack.empty() and popStack.empty()
```

### Trace

```
  push(1): pushStack=[1]     popStack=[]
  push(2): pushStack=[1,2]   popStack=[]
  peek():  transfer → push=[] pop=[2,1]. return 1
  pop():   pop from popStack → return 1. pop=[2]
  push(3): pushStack=[3]     popStack=[2]
  pop():   popStack not empty → return 2. pop=[]
  pop():   transfer → push=[] pop=[3]. return 3
```

**Amortized O(1)** per operation — each element transferred at most once.

---

## 📝 Problem 3: Implement Stack using Queues (LeetCode 225)

```
class MyStack:
    queue = []
    
    push(x):
        queue.enqueue(x)
        // Rotate: bring x to front
        for i = 0 to queue.size() - 2:
            queue.enqueue(queue.dequeue())
    
    pop():
        return queue.dequeue()
    
    top():
        return queue.front()
    
    empty():
        return queue.empty()
```

### Trace

```
  push(1): queue=[1]
  push(2): enqueue 2 → [1,2]. rotate 1 → [2,1]
  push(3): enqueue 3 → [2,1,3]. rotate 2 → [1,3,2]. rotate 1 → [3,2,1]
  top():   front = 3 ✓
  pop():   dequeue → 3 ✓. queue=[2,1]
```

---

## 📝 Problem 4: Design Circular Queue (LeetCode 622)

```
class CircularQueue:
    arr = array of size k
    front = 0, rear = -1, count = 0, capacity = k
    
    enQueue(val):
        if isFull(): return false
        rear = (rear + 1) % capacity
        arr[rear] = val
        count++
        return true
    
    deQueue():
        if isEmpty(): return false
        front = (front + 1) % capacity
        count--
        return true
    
    Front(): return isEmpty() ? -1 : arr[front]
    Rear():  return isEmpty() ? -1 : arr[rear]
    isEmpty(): return count == 0
    isFull():  return count == capacity
```

```
  Circular array (capacity 4):
  
  After enqueue 1,2,3:        After dequeue:
  ┌───┬───┬───┬───┐           ┌───┬───┬───┬───┐
  │ 1 │ 2 │ 3 │   │           │   │ 2 │ 3 │   │
  └───┴───┴───┴───┘           └───┴───┴───┴───┘
    ↑           ↑                    ↑       ↑
   front      rear                front    rear
  
  After enqueue 4, 5:
  ┌───┬───┬───┬───┐
  │ 5 │ 2 │ 3 │ 4 │     ← wraps around!
  └───┴───┴───┴───┘
    ↑   ↑
   rear front
```

---

## 📊 Stack vs Queue Comparison

| Feature | Stack | Queue |
|---------|-------|-------|
| Order | LIFO | FIFO |
| Insert | push (top) | enqueue (rear) |
| Remove | pop (top) | dequeue (front) |
| Use cases | DFS, undo, matching | BFS, scheduling, buffering |
| Monotonic variant | Next greater/smaller | Sliding window max/min |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Monotonic deque | Sliding window max/min in O(n) |
| Queue from stacks | Two stacks, lazy transfer — amortized O(1) |
| Stack from queue | Rotate after each push — O(n) push, O(1) pop |
| Circular queue | Modulo arithmetic for wrap-around |
| Deque | Double-ended queue — operations at both ends O(1) |

---

## ❓ Revision Questions

1. **Sliding window max: why deque not heap?** → Deque gives O(n) total; heap gives O(n log k). Deque also removes out-of-window elements efficiently.
2. **Queue from two stacks: why amortized O(1)?** → Each element is transferred from pushStack to popStack at most once during its lifetime.
3. **Monotonic deque: what's the invariant?** → Elements (by value) are in decreasing order from front to back; front is always the current window maximum.
4. **Circular queue: why modulo?** → `(index + 1) % capacity` wraps the pointer back to 0 when it reaches the end of the array.
5. **When to use deque vs stack?** → Deque when you need to access/remove from both ends (sliding window); stack when only one end matters (matching, DFS).

---

[← Previous: Monotonic Stack](02-monotonic-stack.md) | [Next: Classic Problems →](04-classic-problems.md)
