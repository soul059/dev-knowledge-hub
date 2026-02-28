# Chapter 4: Template and Variations

## 📋 Chapter Overview
Reusable greedy templates and common variations: sorting strategies, two-pass techniques, and heap-assisted greedy.

---

## 📐 Template 1: Sort-and-Scan

```
function greedySortScan(items):
    sort items by some criterion
    result = initial value
    
    for each item in sorted order:
        if item satisfies condition:
            include item in solution
            update state
    
    return result
```

### When to Use

```
  • Activity selection → sort by finish
  • Fractional knapsack → sort by value/weight
  • Assign cookies → sort both arrays
  • Minimum platforms → sort events
```

---

## 📐 Template 2: Two-Pass Greedy

```
function twoPassGreedy(arr):
    result = array of defaults
    
    // Pass 1: left to right
    for i = 1 to n-1:
        if condition(arr[i], arr[i-1]):
            result[i] = f(result[i-1])
    
    // Pass 2: right to left
    for i = n-2 down to 0:
        if condition(arr[i], arr[i+1]):
            result[i] = max(result[i], f(result[i+1]))
    
    return result
```

### When to Use

```
  • Candy distribution
  • Trapping rain water (can also use two pointers)
  • Product of array except self (prefix/suffix)
```

---

## 📐 Template 3: Greedy with Heap

```
function heapGreedy(items):
    heap = min-heap or max-heap
    
    sort or preprocess items
    
    for each item:
        add to heap
        if heap violates constraint:
            remove worst element
    
    return heap contents or accumulated value
```

### Example: Meeting Rooms II (Heap Variant)

```
function minMeetingRooms(intervals):
    sort intervals by start time
    minHeap = []  // stores end times of active meetings
    
    for each [start, end]:
        if minHeap not empty and minHeap.peek() <= start:
            minHeap.poll()  // reuse room
        minHeap.add(end)
    
    return minHeap.size()
```

### Trace

```
  Intervals: [0,30] [5,10] [15,20]
  Sorted by start: [0,30] [5,10] [15,20]
  
  [0,30]:  heap = [30].               rooms = 1
  [5,10]:  peek=30 > 5 → no reuse.    
           heap = [10,30].             rooms = 2
  [15,20]: peek=10 ≤ 15 → poll 10.    
           heap = [20,30].             rooms = 2
  
  Answer: 2
```

---

## 📐 Template 4: Greedy + Stack (Monotonic)

```
function greedyStack(arr, k):
    stack = []
    remaining = k
    
    for each element:
        while stack not empty AND remaining > 0 AND
              stack.top is worse than current:
            stack.pop()
            remaining--
        stack.push(element)
    
    // Remove remaining from end if needed
    while remaining > 0:
        stack.pop()
        remaining--
    
    return stack
```

### Example: Remove K Digits (LeetCode 402)

```
  num = "1432219", k = 3
  
  Goal: remove 3 digits to make smallest number
  
  stack = []
  '1': push → [1]
  '4': 4 > 1 → push → [1,4]
  '3': 3 < 4 → pop 4, k=2 → push → [1,3]
  '2': 2 < 3 → pop 3, k=1 → push → [1,2]
  '2': 2 = 2 → push → [1,2,2]
  '1': 1 < 2 → pop 2, k=0 → push → [1,2,1]
  '9': push → [1,2,1,9]
  
  Result: "1219"
```

---

## 🔄 Common Greedy Sorting Strategies

| Sort By | Used For | Why |
|---------|----------|-----|
| End time | Activity selection, arrows | Leaves maximum future room |
| Start time | Merge intervals, meeting rooms | Process in chronological order |
| Value/weight ratio | Fractional knapsack | Maximize value per unit |
| Deadline | Job scheduling | Prioritize urgent tasks |
| Difficulty / size | Assign cookies, boats | Match easy tasks first |
| Frequency | Huffman, task scheduler | Handle bottleneck first |

---

## 📝 Variation: Assign Cookies (LeetCode 455)

```
  Children have greed factors, cookies have sizes.
  Each child gets at most one cookie (size ≥ greed).
  Maximize satisfied children.
  
function assignCookies(children, cookies):
    sort children ascending
    sort cookies ascending
    child = 0, cookie = 0
    
    while child < len(children) and cookie < len(cookies):
        if cookies[cookie] >= children[child]:
            child++     // satisfied
        cookie++        // try next cookie either way
    
    return child
```

---

## 📝 Variation: Boats to Save People (LeetCode 881)

```
  People have weights, boat capacity = limit, max 2 people per boat.
  Minimize boats.
  
function numRescueBoats(people, limit):
    sort people ascending
    left = 0, right = n-1
    boats = 0
    
    while left <= right:
        if people[left] + people[right] <= limit:
            left++      // pair lightest with heaviest
        right--         // heaviest always goes
        boats++
    
    return boats
```

---

## 📊 Template Decision Guide

| Situation | Template | Examples |
|-----------|----------|----------|
| Select/schedule items | Sort-and-scan | Activity, cookies, boats |
| Left/right neighbor constraints | Two-pass | Candy, trapping rain |
| Dynamic constraint management | Heap-greedy | Meeting rooms, top-k |
| Build optimal sequence | Stack-greedy | Remove K digits, next greater |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Sort-and-scan | Most common greedy template |
| Two-pass | Handle bidirectional constraints |
| Heap-assisted | Dynamically manage best candidates |
| Stack-based | Build optimal sequence greedily |
| Sorting criterion | Critical — wrong sort = wrong answer |

---

## ❓ Revision Questions

1. **Most common greedy template?** → Sort-and-scan: sort by appropriate criterion, greedily process in order.
2. **Two-pass greedy: when needed?** → When constraints come from both directions (left and right neighbors).
3. **Heap in greedy: purpose?** → Efficiently track/update the best (or worst) candidate as items are processed.
4. **Remove K digits: why stack?** → Stack maintains smallest-so-far sequence; pop larger digits when a smaller one arrives.
5. **Why is sort criterion critical?** → Different criteria lead to different greedy orderings; wrong one can produce suboptimal results.

---

[← Previous: Array Greedy](03-array-greedy.md) | [Next: Classic Problems →](05-classic-problems.md)
