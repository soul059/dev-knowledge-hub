# Chapter 3: Template and Variations

## 📋 Chapter Overview
Master the **universal sliding window template** and its variations. Learn to adapt the template for different problem types with minimal changes.

---

## 🏗️ The Universal Template

```
PSEUDOCODE — Universal Sliding Window:

function slidingWindow(input):
    left = 0
    result = initial_value
    state = initialize()    // HashMap, counter, sum, set, etc.
    
    for right = 0 to len(input) - 1:
        // ──── STEP 1: EXPAND ────
        // Add input[right] into window state
        state.add(input[right])
        
        // ──── STEP 2: CONTRACT ────
        // While window violates the constraint
        while windowIsInvalid(state):
            state.remove(input[left])
            left += 1
        
        // ──── STEP 3: UPDATE ────
        // Record the best valid window
        result = updateResult(result, left, right)
    
    return result
```

### Architecture:

```
  ┌──────────────────────────────────────────────────────┐
  │              SLIDING WINDOW MACHINE                  │
  │                                                      │
  │   INPUT:  [ a  b  c  d  e  f  g  h  i  j  k ]      │
  │             ↑                       ↑                │
  │           left                    right              │
  │                                                      │
  │   ┌────────────────────────────────────┐              │
  │   │         WINDOW STATE              │              │
  │   │  ┌────────────┬─────────────┐     │              │
  │   │  │  Expand    │  Contract   │     │              │
  │   │  │  (right++) │  (left++)   │     │              │
  │   │  └──────┬─────┴──────┬──────┘     │              │
  │   │         │            │            │              │
  │   │         ▼            ▼            │              │
  │   │  ┌─────────────────────────┐      │              │
  │   │  │   CHECK CONDITION       │      │              │
  │   │  │   valid? → update result│      │              │
  │   │  │   invalid? → contract   │      │              │
  │   │  └─────────────────────────┘      │              │
  │   └────────────────────────────────────┘              │
  │                                                      │
  │   OUTPUT: result (max length, min length, count, etc)│
  └──────────────────────────────────────────────────────┘
```

---

## 🔄 Variation 1: Longest Window with Condition

**Use when:** "Find the LONGEST substring/subarray satisfying condition X"

```
PSEUDOCODE — Longest Valid Window:

function longestWindow(arr, condition):
    left = 0
    maxLen = 0
    state = initialize()
    
    for right = 0 to n - 1:
        state.add(arr[right])           // expand
        
        while NOT condition(state):     // window invalid?
            state.remove(arr[left])     // shrink
            left += 1
        
        // Window is now VALID — record its size
        maxLen = max(maxLen, right - left + 1)
    
    return maxLen
```

**Examples:**
- Longest substring without repeating characters
- Longest substring with at most K distinct characters
- Longest ones after flipping at most K zeros

---

## 🔄 Variation 2: Shortest Window with Condition

**Use when:** "Find the SHORTEST substring/subarray satisfying condition X"

```
PSEUDOCODE — Shortest Valid Window:

function shortestWindow(arr, condition):
    left = 0
    minLen = INFINITY
    state = initialize()
    
    for right = 0 to n - 1:
        state.add(arr[right])           // expand
        
        while condition(state):         // window VALID?
            minLen = min(minLen, right - left + 1)  // record!
            state.remove(arr[left])     // try shorter
            left += 1
    
    return minLen if minLen != INFINITY else 0
```

**Examples:**
- Minimum size subarray with sum ≥ S
- Minimum window substring

---

## 🔄 Variation 3: Count Windows

**Use when:** "Count number of subarrays/substrings satisfying condition X"

```
PSEUDOCODE — Count Valid Windows:

function countWindows(arr, condition):
    left = 0
    count = 0
    state = initialize()
    
    for right = 0 to n - 1:
        state.add(arr[right])
        
        while NOT condition(state):
            state.remove(arr[left])
            left += 1
        
        // All windows ending at 'right' with left in [left..right] are valid
        count += (right - left + 1)
    
    return count
```

**Why `right - left + 1`?**

```
  Valid window: arr[left..right]
  
  All subarrays ending at 'right' that are valid:
  [left..right], [left+1..right], ..., [right..right]
  
  Count = right - left + 1
  
  Example: arr = [1, 2, 3], all valid
  right=0: [1]             → 1 subarray
  right=1: [1,2], [2]      → 2 subarrays
  right=2: [1,2,3],[2,3],[3] → 3 subarrays
  Total = 1 + 2 + 3 = 6
```

---

## 🔄 Variation 4: Exact Condition (using "at most" trick)

**Use when:** "Count subarrays with EXACTLY K distinct" (or similar exact constraint)

```
  KEY INSIGHT:
  
  exactly(K) = atMost(K) - atMost(K - 1)
  
  ┌────────────────────────────────────────┐
  │  atMost(3):   subarrays with ≤3 distinct │
  │  atMost(2):   subarrays with ≤2 distinct │
  │                                        │
  │  exactly(3) = atMost(3) - atMost(2)    │
  │                                        │
  │  This works because:                   │
  │  atMost(3) includes subarrays with     │
  │  0, 1, 2, or 3 distinct chars          │
  │  atMost(2) includes subarrays with     │
  │  0, 1, or 2 distinct chars             │
  │  Subtracting removes 0, 1, 2 → leaves 3│
  └────────────────────────────────────────┘
```

```
PSEUDOCODE — Exactly K using At-Most-K:

function exactlyK(arr, k):
    return atMostK(arr, k) - atMostK(arr, k - 1)

function atMostK(arr, k):
    left = 0
    count = 0
    freq = {}
    
    for right = 0 to n - 1:
        freq[arr[right]] += 1
        
        while len(freq) > k:
            freq[arr[left]] -= 1
            if freq[arr[left]] == 0:
                delete freq[arr[left]]
            left += 1
        
        count += (right - left + 1)
    
    return count
```

---

## 🔄 Variation 5: Sliding Window with Deque (Max/Min tracking)

**Use when:** You need the max or min of each window efficiently.

```
  MONOTONIC DEQUE — Track max of each window of size k
  
  Array: [1, 3, -1, -3, 5, 3, 6, 7]    k = 3
  
  Deque stores INDICES, maintaining decreasing order of values.
  
  ┌──────┬──────────────┬─────────┬────────────┐
  │  i   │  Deque (vals) │ Window  │ Max        │
  ├──────┼──────────────┼─────────┼────────────┤
  │  0   │  [1]          │ [1]     │  -         │
  │  1   │  [3]          │ [1,3]   │  -         │
  │  2   │  [3,-1]       │ [1,3,-1]│  3         │
  │  3   │  [3,-1,-3]    │ [3,-1,-3]│ 3         │
  │  4   │  [5]          │ [-1,-3,5]│ 5         │
  │  5   │  [5,3]        │ [-3,5,3]│  5         │
  │  6   │  [6]          │ [5,3,6] │  6         │
  │  7   │  [7]          │ [3,6,7] │  7         │
  └──────┴──────────────┴─────────┴────────────┘
  
  Output: [3, 3, 5, 5, 6, 7]
```

```
PSEUDOCODE — Max of All Windows of Size K:

function maxSlidingWindow(arr, k):
    deque = []      // stores indices  
    result = []
    
    for i = 0 to n - 1:
        // Remove elements outside window
        while deque not empty AND deque.front() <= i - k:
            deque.popFront()
        
        // Remove smaller elements (maintain decreasing)
        while deque not empty AND arr[deque.back()] <= arr[i]:
            deque.popBack()
        
        deque.pushBack(i)
        
        // Window is full (i >= k-1)
        if i >= k - 1:
            result.append(arr[deque.front()])
    
    return result
```

---

## 🗺️ Variation Decision Map

```
  ┌─────────────────────────────┐
  │  What does the problem ask? │
  └──────────┬──────────────────┘
             │
    ┌────────┼────────┬──────────────┐
    ▼        ▼        ▼              ▼
  LONGEST  SHORTEST  COUNT         MAX/MIN
  subarray subarray  subarrays     of window
    │        │        │              │
    ▼        ▼        ▼              ▼
  Var 1    Var 2    Var 3           Var 5
  shrink   shrink   add            Monotonic
  when     when     right-left+1   Deque
  invalid  valid    each step
                      │
                      ▼
                   EXACTLY K?
                   Use Var 4:
                   atMost(k) - atMost(k-1)
```

---

## 📊 Template Comparison

| Variant | Shrink Condition | Update Timing | Result Type |
|---------|-----------------|---------------|-------------|
| Longest | While invalid | After shrinking | max(len) |
| Shortest | While valid | Before shrinking | min(len) |
| Count | While invalid | After shrinking | sum(right-left+1) |
| Exactly K | atMost(K) - atMost(K-1) | After shrinking | difference |
| Max/Min Window | Window too large | After full window | deque front |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Universal Template | Expand → Check → Shrink → Update |
| Longest Variant | Shrink when invalid, update when valid |
| Shortest Variant | Shrink when valid (try shorter), update before shrink |
| Count Variant | Every valid window adds (right - left + 1) |
| Exact K Trick | exactly(K) = atMost(K) - atMost(K-1) |
| Deque Variant | Monotonic deque for max/min within window |
| Adaptation | Only the condition check and state update change |

---

## ❓ Quick Revision Questions

1. **What are the three steps in every sliding window iteration?**
   > Expand (add right element), Contract (shrink if needed), Update (record best result).

2. **How do you count subarrays with exactly K distinct elements?**
   > Use the trick: `exactly(K) = atMost(K) - atMost(K-1)`, implementing `atMost` with sliding window.

3. **What is the key difference between "longest" and "shortest" variants?**
   > Longest shrinks when invalid and updates when valid. Shortest shrinks when valid and updates before shrinking.

4. **Why does `count += (right - left + 1)` work for counting valid subarrays?**
   > Because if `[left..right]` is valid, then all subarrays `[left..right], [left+1..right], ..., [right..right]` are also valid, giving `right - left + 1` new valid subarrays.

5. **When should you use a monotonic deque?**
   > When you need to track the maximum or minimum value within each sliding window efficiently (O(1) per query).

6. **What stays the same across all window variations?**
   > The core structure: two pointers (left, right), a state variable, and the expand-check-shrink loop.

---

[← Previous: Variable Size Window](02-variable-size-window.md) | [Next: Classic Problems →](04-classic-problems.md)

[← Back to README](../README.md)
