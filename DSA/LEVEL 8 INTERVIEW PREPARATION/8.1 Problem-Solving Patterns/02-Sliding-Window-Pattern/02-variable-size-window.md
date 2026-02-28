# Chapter 2: Variable Size Window

## 📋 Chapter Overview
The **Variable Size Sliding Window** dynamically grows and shrinks based on a condition. This is more powerful and flexible than fixed-size windows, handling problems like "longest/shortest substring with condition X."

---

## 🎯 Core Concept

```
  VARIABLE SIZE SLIDING WINDOW
  
  The window EXPANDS to explore and SHRINKS to satisfy a condition.
  
  Array:  [ 2  3  1  2  4  3 ]     Target sum ≥ 7
  
  Step 1:  [2] 3  1  2  4  3       sum=2   < 7  → EXPAND
  Step 2:  [2  3] 1  2  4  3       sum=5   < 7  → EXPAND
  Step 3:  [2  3  1] 2  4  3       sum=6   < 7  → EXPAND
  Step 4:  [2  3  1  2] 4  3       sum=8   ≥ 7  → RECORD len=4, SHRINK
  Step 5:   2 [3  1  2] 4  3       sum=6   < 7  → EXPAND
  Step 6:   2 [3  1  2  4] 3       sum=10  ≥ 7  → RECORD len=4, SHRINK
  Step 7:   2  3 [1  2  4] 3       sum=7   ≥ 7  → RECORD len=3, SHRINK
  Step 8:   2  3  1 [2  4] 3       sum=6   < 7  → EXPAND
  Step 9:   2  3  1 [2  4  3]      sum=9   ≥ 7  → RECORD len=3, SHRINK
  Step10:   2  3  1  2 [4  3]      sum=7   ≥ 7  → RECORD len=2 ← MIN!
  Step11:   2  3  1  2  4 [3]      sum=3   < 7  → EXPAND (done)
  
  Answer: minimum length = 2 (subarray [4, 3])
```

### Key Difference from Fixed Window:

```
  ┌──────────────────────────────────────────────────────┐
  │                                                      │
  │  FIXED WINDOW:     left moves exactly with right     │
  │                    Window size = constant = k         │
  │  ┌──────────┐                                        │
  │  │  k = 3   │  Always 3 elements                     │
  │  └──────────┘                                        │
  │                                                      │
  │  VARIABLE WINDOW:  left moves based on condition     │
  │                    Window size = dynamic              │
  │  ┌──┐ to ┌────────────────┐  Size changes!           │
  │  └──┘    └────────────────┘                          │
  │                                                      │
  └──────────────────────────────────────────────────────┘
```

---

## 📐 The Template

```
PSEUDOCODE — Variable Size Sliding Window:

function variableSlidingWindow(arr, condition):
    left = 0
    result = initial_value    // 0 for max, ∞ for min
    state = {}                // window state tracker
    
    for right = 0 to n - 1:
        // STEP 1: EXPAND — add arr[right] to state
        updateState(state, arr[right])
        
        // STEP 2: SHRINK — while window is invalid
        while not isValid(state, condition):
            removeFromState(state, arr[left])
            left += 1
        
        // STEP 3: UPDATE — record best result
        result = best(result, right - left + 1)
    
    return result
```

### Two Variants:

```
  ┌───────────────────────────────────────────────────────┐
  │                                                       │
  │  VARIANT 1: LONGEST / MAXIMUM                         │
  │  ─────────────────────────────                        │
  │  Expand → if invalid, SHRINK until valid → record     │
  │  Update AFTER shrinking (window is valid)              │
  │                                                       │
  │  for right = 0 to n-1:                                │
  │      add arr[right]                                   │
  │      while INVALID:                                   │
  │          remove arr[left], left++                      │
  │      result = max(result, right - left + 1)           │
  │                                                       │
  │  VARIANT 2: SHORTEST / MINIMUM                         │
  │  ─────────────────────────────                        │
  │  Expand → if valid, RECORD & SHRINK to find shorter   │
  │  Update BEFORE shrinking (window is valid)             │
  │                                                       │
  │  for right = 0 to n-1:                                │
  │      add arr[right]                                   │
  │      while VALID:                                     │
  │          result = min(result, right - left + 1)       │
  │          remove arr[left], left++                      │
  │                                                       │
  └───────────────────────────────────────────────────────┘
```

---

## 🧪 Trace 1: Longest Substring Without Repeating Characters

**Problem:** Find the length of the longest substring without repeating characters.

**Input:** `s = "abcabcbb"`

```
  State: HashSet to track characters in current window
  
  ┌──────┬───────┬──────────┬───────────┬─────────────┬────────┐
  │ right│ char  │ Action   │  Window   │    Set      │  max   │
  ├──────┼───────┼──────────┼───────────┼─────────────┼────────┤
  │  0   │  'a'  │ add      │ "a"       │ {a}         │   1    │
  │  1   │  'b'  │ add      │ "ab"      │ {a,b}       │   2    │
  │  2   │  'c'  │ add      │ "abc"     │ {a,b,c}     │   3    │
  │  3   │  'a'  │ DUP! shrk│ "bca"     │ {b,c,a}     │   3    │
  │  4   │  'b'  │ DUP! shrk│ "cab"     │ {c,a,b}     │   3    │
  │  5   │  'c'  │ DUP! shrk│ "abc"     │ {a,b,c}     │   3    │
  │  6   │  'b'  │ DUP! shrk│ "cb"      │ {c,b}       │   3    │
  │  7   │  'b'  │ DUP! shrk│ "b"       │ {b}         │   3    │
  └──────┴───────┴──────────┴───────────┴─────────────┴────────┘
  
  Answer: 3 ("abc")
```

### Pseudocode:

```
function longestSubstringNoRepeat(s):
    left = 0
    maxLen = 0
    charSet = empty set
    
    for right = 0 to len(s) - 1:
        // Shrink while duplicate exists
        while s[right] in charSet:
            charSet.remove(s[left])
            left += 1
        
        // Add current character
        charSet.add(s[right])
        
        // Update result
        maxLen = max(maxLen, right - left + 1)
    
    return maxLen
```

**Time:** O(n) — each character is added and removed at most once  
**Space:** O(min(n, 26)) — bounded by alphabet size

---

## 🧪 Trace 2: Minimum Size Subarray Sum

**Problem:** Find the minimum length subarray with sum ≥ target.

**Input:** `arr = [2, 3, 1, 2, 4, 3]`, `target = 7`

```
  ┌──────┬──────┬────────┬──────────┬──────────────────┬────────┐
  │ right│  add │ sum    │ ≥ 7?     │  Window          │ minLen │
  ├──────┼──────┼────────┼──────────┼──────────────────┼────────┤
  │  0   │  +2  │   2    │ No       │ [2]              │   ∞    │
  │  1   │  +3  │   5    │ No       │ [2,3]            │   ∞    │
  │  2   │  +1  │   6    │ No       │ [2,3,1]          │   ∞    │
  │  3   │  +2  │   8    │ YES→shrk │ [2,3,1,2] len=4  │   4    │
  │      │  -2  │   6    │ No       │ [3,1,2]          │   4    │
  │  4   │  +4  │  10    │ YES→shrk │ [3,1,2,4] len=4  │   4    │
  │      │  -3  │   7    │ YES→shrk │ [1,2,4]   len=3  │   3    │
  │      │  -1  │   6    │ No       │ [2,4]            │   3    │
  │  5   │  +3  │   9    │ YES→shrk │ [2,4,3]   len=3  │   3    │
  │      │  -2  │   7    │ YES→shrk │ [4,3]     len=2  │   2    │
  │      │  -4  │   3    │ No       │ [3]              │   2    │
  └──────┴──────┴────────┴──────────┴──────────────────┴────────┘
  
  Answer: minLen = 2 (subarray [4, 3])
```

### Pseudocode:

```
function minSubarrayLen(target, arr):
    left = 0
    currentSum = 0
    minLen = INFINITY
    
    for right = 0 to len(arr) - 1:
        currentSum += arr[right]
        
        // Shrink while condition is satisfied (find shorter)
        while currentSum >= target:
            minLen = min(minLen, right - left + 1)
            currentSum -= arr[left]
            left += 1
    
    return minLen if minLen != INFINITY else 0
```

---

## 🧪 Trace 3: Longest Substring with at Most K Distinct Characters

**Problem:** Find the longest substring with at most K distinct characters.

**Input:** `s = "eceba"`, `k = 2`

```
  State: HashMap {char → count}
  
  ┌──────┬──────┬──────────────┬───────────┬──────────┬────────┐
  │ right│ char │ map          │ distinct  │ window   │  max   │
  ├──────┼──────┼──────────────┼───────────┼──────────┼────────┤
  │  0   │ 'e'  │ {e:1}        │    1      │ "e"      │   1    │
  │  1   │ 'c'  │ {e:1,c:1}    │    2      │ "ec"     │   2    │
  │  2   │ 'e'  │ {e:2,c:1}    │    2      │ "ece"    │   3    │
  │  3   │ 'b'  │ {e:2,c:1,b:1}│    3>2!   │ SHRINK   │        │
  │      │      │ remove 'e':  │           │          │        │
  │      │      │ {e:1,c:1,b:1}│    3>2!   │ SHRINK   │        │
  │      │      │ remove 'c':  │           │          │        │
  │      │      │ {e:1,b:1}    │    2      │ "eb"     │   3    │
  │  4   │ 'a'  │ {e:1,b:1,a:1}│    3>2!   │ SHRINK   │        │
  │      │      │ remove 'e':  │           │          │        │
  │      │      │ {b:1,a:1}    │    2      │ "ba"     │   3    │
  └──────┴──────┴──────────────┴───────────┴──────────┴────────┘
  
  Answer: 3 ("ece")
```

---

## 🔍 How the Window Logic Works

```
  ┌────────────────────────────────────────────────────┐
  │          VARIABLE WINDOW STATE MACHINE             │
  │                                                    │
  │            ┌──────────────┐                        │
  │     ┌─────►│   EXPAND     │                        │
  │     │      │  right++     │                        │
  │     │      │  add element │                        │
  │     │      └──────┬───────┘                        │
  │     │             │                                │
  │     │             ▼                                │
  │     │      ┌──────────────┐                        │
  │     │      │  CHECK       │──── Valid? ──► UPDATE  │
  │     │      │  Condition   │     result             │
  │     │      └──────┬───────┘                        │
  │     │             │                                │
  │     │         Invalid?                             │
  │     │             │                                │
  │     │             ▼                                │
  │     │      ┌──────────────┐                        │
  │     └──────│   SHRINK     │                        │
  │            │  left++      │                        │
  │            │  rmv element │                        │
  │            └──────────────┘                        │
  │                                                    │
  └────────────────────────────────────────────────────┘
```

---

## 📊 Complexity Analysis

| Aspect | Analysis |
|--------|----------|
| **Outer loop** | `right` moves from 0 to n-1: O(n) iterations |
| **Inner loop** | `left` moves from 0 to n-1 *total*: O(n) total |
| **Combined** | Each element is added once and removed once: O(n) |
| **Space** | Depends on state: O(1) for sum, O(k) for hash map |

### Why Both Pointers Together Give O(n):

```
  Total work = (right pointer moves) + (left pointer moves)
             = n + n
             = O(2n)
             = O(n)
  
  The LEFT pointer NEVER moves backward, so its total
  movement across ALL iterations is at most n.
```

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Variable Window | Window size changes dynamically based on condition |
| Expand | Move right pointer to grow window |
| Shrink | Move left pointer to reduce window until valid |
| Longest variant | Shrink when invalid, update when valid |
| Shortest variant | Update when valid, then shrink to find shorter |
| Time Complexity | O(n) — each element added/removed at most once |
| Key Signal | "longest/shortest substring/subarray with condition" |

---

## ❓ Quick Revision Questions

1. **How does variable window differ from fixed window?**
   > In variable window, the size changes dynamically — the left pointer moves based on a condition, not just following the right pointer at a fixed distance.

2. **For "longest" problems, when do you update the result?**
   > After shrinking — update when the window is valid (just became valid or stayed valid).

3. **For "shortest" problems, when do you update the result?**
   > Before shrinking — update when the window first becomes valid, then shrink to try to find shorter.

4. **Why is the time complexity O(n) even though there are nested loops?**
   > The left pointer moves at most n times total across all iterations, so total work is O(n + n) = O(n).

5. **What data structures typically track window state?**
   > Hash maps (frequency counting), hash sets (distinctness), or simple variables (sum, count).

6. **How do you handle "at most K distinct characters"?**
   > Use a hash map for character frequencies. When distinct count exceeds K, shrink from the left until count ≤ K.

---

[← Previous: Fixed Size Window](01-fixed-size-window.md) | [Next: Template and Variations →](03-template-and-variations.md)

[← Back to README](../README.md)
