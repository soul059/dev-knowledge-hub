# Chapter 3: Two-Heap Pattern

## 📋 Chapter Overview
Using two heaps (max-heap + min-heap) to maintain a dynamic median and solve related streaming problems.

---

## 🧠 Core Concept

```
  Split elements into two halves:
  
  MAX HEAP (left half)      MIN HEAP (right half)
  ┌──────────────────┐      ┌──────────────────┐
  │ stores smaller    │      │ stores larger     │
  │ half of numbers   │      │ half of numbers   │
  │                   │      │                   │
  │ top = max of left │      │ top = min of right│
  └──────────────────┘      └──────────────────┘
  
  Invariant: maxHeap.top ≤ minHeap.top
  Balance:   |maxHeap.size - minHeap.size| ≤ 1
  
  Median:
  • Both same size → average of both tops
  • maxHeap larger → maxHeap.top
```

---

## 📝 Problem 1: Find Median from Data Stream (LeetCode 295)

```
class MedianFinder:
    maxHeap = []   // left half (negate values for max heap)
    minHeap = []   // right half
    
    addNum(num):
        // Always add to maxHeap first, then balance
        maxHeap.push(num)
        
        // Ensure maxHeap.top ≤ minHeap.top
        minHeap.push(maxHeap.poll())
        
        // Balance sizes: maxHeap should be ≥ minHeap size
        if minHeap.size > maxHeap.size:
            maxHeap.push(minHeap.poll())
    
    findMedian():
        if maxHeap.size > minHeap.size:
            return maxHeap.top
        return (maxHeap.top + minHeap.top) / 2.0
```

### Trace

```
  addNum(1):
    maxH push 1 → [1]. Poll 1 → minH push 1 → [1]
    minH(1) > maxH(0) → maxH push(minH.poll=1) → maxH=[1], minH=[]
    
  addNum(2):
    maxH push 2 → [2,1]. Poll 2 → minH push 2 → [2]
    sizes equal → done. maxH=[1], minH=[2]
    Median: (1+2)/2 = 1.5
    
  addNum(3):
    maxH push 3 → [3,1]. Poll 3 → minH push 3 → [2,3]
    minH(2) > maxH(1) → maxH push(minH.poll=2) → maxH=[2,1], minH=[3]
    Median: maxH.top = 2
    
  State: maxH=[2,1], minH=[3]
  Left half: 1, 2     Right half: 3
  Sorted: 1, [2], 3   Median = 2 ✓
```

**Complexity:** O(log n) per add, O(1) per findMedian

---

## 📝 Problem 2: Sliding Window Median (LeetCode 480)

```
function medianSlidingWindow(nums, k):
    maxHeap = []   // left half
    minHeap = []   // right half
    result = []
    
    for i = 0 to n-1:
        addNum(nums[i])   // add to appropriate heap
        
        if i >= k:
            removeNum(nums[i - k])  // lazy removal
        
        if i >= k - 1:
            result.add(findMedian())
    
    return result
```

### Lazy Deletion Approach

```
  Instead of physically removing from heap (expensive),
  mark element as "to be deleted" in a hash map.
  
  When element appears at top of a heap during
  poll or peek, check if it's marked. If yes, discard and skip.
  
  This keeps operations amortized O(log n).
```

**Complexity:** O(n log k) time, O(k) space

---

## 📝 Problem 3: IPO — Maximize Capital (LeetCode 502)

**Problem:** You have initial capital `w` and can do at most `k` projects. Each project has a profit and minimum capital requirement. Maximize total capital.

```
function findMaximizedCapital(k, w, profits, capital):
    projects = zip and sort by capital (ascending)
    maxHeap = []  // profits of available projects
    i = 0
    
    for _ = 0 to k-1:
        // Add all projects affordable with current capital
        while i < n and projects[i].capital <= w:
            maxHeap.push(projects[i].profit)
            i++
        
        if maxHeap is empty: break
        w += maxHeap.poll()   // pick most profitable
    
    return w
```

### Trace

```
  k=2, w=0
  profits=[1,2,3], capital=[0,1,1]
  Sorted by capital: (0,1), (1,2), (1,3)
  
  Round 1: w=0. Add (0,1). maxHeap=[1]. Pick 1. w=0+1=1
  Round 2: w=1. Add (1,2),(1,3). maxHeap=[3,2]. Pick 3. w=1+3=4
  
  Answer: 4
```

**Key insight:** Two different heaps — min heap (by capital) to find affordable projects, max heap (by profit) to pick the best.

---

## 📐 Two-Heap Pattern Template

```
  1. MAX HEAP: stores left/smaller half
  2. MIN HEAP: stores right/larger half
  
  Add element:
    → push to maxHeap
    → move maxHeap.top to minHeap (ensures ordering)
    → rebalance sizes
  
  Query median:
    → if sizes equal: average of both tops
    → if maxHeap larger: maxHeap.top
```

---

## 📊 Two-Heap Applications

| Problem | MaxHeap Stores | MinHeap Stores | Purpose |
|---------|---------------|----------------|---------|
| Stream median | Left half | Right half | O(1) median |
| Sliding window median | Left of window | Right of window | Rolling median |
| IPO | Available profits | Projects by cost | Max profit selection |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Two-heap median | Max heap (left) + min heap (right), balanced sizes |
| Invariant | maxHeap.top ≤ minHeap.top, sizes differ by ≤ 1 |
| Lazy deletion | Mark elements; discard at top during operations |
| IPO pattern | Sort by one dimension, heap by another |

---

## ❓ Revision Questions

1. **Why two heaps for median?** → Max heap gives largest of left half, min heap gives smallest of right half — median is at the boundary of both.
2. **Balance invariant?** → Sizes differ by at most 1; maxHeap.top ≤ minHeap.top. This ensures the median is always accessible at the tops.
3. **Sliding window: why lazy deletion?** → Directly removing from a heap's interior is O(n). Lazy deletion defers removal until the element surfaces at the top — amortized O(log n).
4. **IPO: why two different heaps?** → Min heap sorts projects by capital (find affordable ones). Max heap sorts by profit (pick the most profitable among affordable).
5. **Median of odd count?** → MaxHeap has one extra element; its top is the median.

---

[← Previous: Top-K Problems](02-top-k-problems.md) | [Next: Classic Heap Problems →](04-classic-problems.md)
