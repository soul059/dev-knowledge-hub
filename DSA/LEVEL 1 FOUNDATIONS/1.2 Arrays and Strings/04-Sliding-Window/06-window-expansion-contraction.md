# Chapter 6: Window Expansion and Contraction Patterns

[← Previous: Minimum Window Substring](05-minimum-window-substring.md) | [Back to README](../README.md) | [Next Unit: Prefix Sum →](../05-Prefix-Sum-and-Difference-Array/01-prefix-sum-concept.md)

---

## Overview

This chapter synthesizes everything about sliding windows by analyzing the **expansion/contraction mechanics** in depth. We study when to expand, when to contract, and how to choose the right pattern for different problem types.

---

## The Two Core Patterns

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                                                                  │
  │  PATTERN A: FIND LONGEST                                         │
  │  ─────────────────────                                           │
  │  Expand right → if INVALID, shrink left until VALID → update     │
  │                                                                  │
  │  PATTERN B: FIND SHORTEST                                        │
  │  ──────────────────────                                          │
  │  Expand right → while VALID, update answer + shrink left         │
  │                                                                  │
  └──────────────────────────────────────────────────────────────────┘
```

---

## Pattern A: Find Longest (Detailed)

```
FUNCTION findLongest(data, n):
    left = 0
    state = initial
    best = 0

    FOR right = 0 TO n-1:
        // 1. EXPAND: include data[right]
        ADD data[right] TO state

        // 2. CONTRACT: shrink while invalid
        WHILE state is INVALID:
            REMOVE data[left] FROM state
            left = left + 1

        // 3. ANSWER: window [left..right] is valid
        best = MAX(best, right - left + 1)

    RETURN best
```

### Lifecycle Diagram

```
  For each right position:

  ┌──────────────┐     ┌───────────────┐     ┌──────────────┐
  │   EXPAND     │────▶│   CHECK       │────▶│   ANSWER     │
  │   right++    │     │   valid?      │     │   update best│
  │   add elem   │     │     │         │     └──────────────┘
  └──────────────┘     │     │ NO      │            ▲
                       │     ▼         │            │
                       │  CONTRACT     │            │
                       │  left++       │────────────┘
                       │  remove elem  │    (after becoming valid)
                       └───────────────┘
```

### Example Problems for Pattern A

| Problem | State | Invalid When |
|---------|-------|-------------|
| Longest substring, no repeats | char set | duplicate char found |
| Longest with at most k distinct | char freq map | distinct count > k |
| Longest with at most k zeros | zero count | zeros > k |
| Max consecutive ones (flip k) | zero count | zeros > k |

---

## Pattern B: Find Shortest (Detailed)

```
FUNCTION findShortest(data, n):
    left = 0
    state = initial
    best = INFINITY

    FOR right = 0 TO n-1:
        // 1. EXPAND: include data[right]
        ADD data[right] TO state

        // 2. CONTRACT: shrink while valid
        WHILE state is VALID:
            best = MIN(best, right - left + 1)
            REMOVE data[left] FROM state
            left = left + 1

    RETURN best (or -1/0 if INFINITY)
```

### Lifecycle Diagram

```
  For each right position:

  ┌──────────────┐     ┌───────────────┐
  │   EXPAND     │────▶│   CHECK       │
  │   right++    │     │   valid?      │
  │   add elem   │     │     │         │
  └──────────────┘     │     │ YES     │
                       │     ▼         │
                       │  ANSWER       │
                       │  update best  │
                       │     │         │
                       │     ▼         │
                       │  CONTRACT     │──┐
                       │  left++       │  │ loop back
                       │  remove elem  │──┘ if still valid
                       └───────────────┘
```

### Example Problems for Pattern B

| Problem | State | Valid When |
|---------|-------|-----------|
| Min window substring | freq map + have count | all chars of t present |
| Smallest subarray sum ≥ S | running sum | sum ≥ S |
| Shortest covering substring | char count | all required chars met |

---

## Side-by-Side Comparison

```
  LONGEST: "Find the LARGEST window that satisfies X"
  ──────────────────────────────────────────────────
  
  FOR right = 0 TO n-1:
      expand(right)
      WHILE INVALID:        ← shrink until VALID
          contract(left++)
      answer = MAX(answer, window size)   ← answer AFTER loop
  
  
  SHORTEST: "Find the SMALLEST window that satisfies X"
  ────────────────────────────────────────────────────
  
  FOR right = 0 TO n-1:
      expand(right)
      WHILE VALID:          ← shrink while VALID
          answer = MIN(answer, window size)  ← answer INSIDE loop
          contract(left++)
```

```
  ┌────────────────────────────────────────────────────┐
  │  KEY DIFFERENCE:                                    │
  │                                                    │
  │  LONGEST:  Contract while INVALID, answer AFTER    │
  │  SHORTEST: Contract while VALID, answer INSIDE     │
  │                                                    │
  │  Think of it this way:                             │
  │  LONGEST:  "Keep the window as big as possible"    │
  │  SHORTEST: "Shrink the window as small as possible"│
  └────────────────────────────────────────────────────┘
```

---

## Pattern C: Count Valid Subarrays

```
FUNCTION countValid(data, n):
    left = 0
    state = initial
    count = 0

    FOR right = 0 TO n-1:
        ADD data[right] TO state

        WHILE state is INVALID:
            REMOVE data[left] FROM state
            left = left + 1

        // All subarrays ending at right, starting from left to right
        count = count + (right - left + 1)

    RETURN count
```

### Why `right - left + 1`?

```
  Window: [left, left+1, ..., right]
  
  Valid subarrays ending at 'right':
    [right]                   ← length 1
    [right-1, right]          ← length 2
    ...
    [left, left+1, ..., right] ← length right-left+1
    
  Total: right - left + 1 new subarrays

  Example:  left=2, right=5 → subarrays [5], [4,5], [3,5], [2,5] = 4
```

---

## Window State: What to Track?

```
  ┌──────────────────┬───────────────────────────────────┐
  │ State Type       │ Use When                          │
  ├──────────────────┼───────────────────────────────────┤
  │ Single variable  │ Sum, product, count of zeros      │
  │ (int/float)      │ Simple aggregate operations       │
  ├──────────────────┼───────────────────────────────────┤
  │ HashMap/FreqMap  │ Character frequencies, distinct   │
  │                  │ element counts, anagram matching   │
  ├──────────────────┼───────────────────────────────────┤
  │ Counter variable │ "have" (matched chars),           │
  │                  │ distinct count, zero count        │
  ├──────────────────┼───────────────────────────────────┤
  │ Deque/Monotonic  │ Max/min in sliding window         │
  │ Queue            │ (advanced — O(1) per operation)   │
  ├──────────────────┼───────────────────────────────────┤
  │ Set              │ All unique elements in window     │
  └──────────────────┴───────────────────────────────────┘
```

---

## Expansion Mechanics

When element enters the window (right moves right):

```
  ┌────────────────────────────────────────────────┐
  │  Sum:            windowSum += arr[right]       │
  │  Frequency:      freq[arr[right]]++            │
  │  Distinct count: if freq becomes 1, count++    │
  │  Zero count:     if arr[right]==0, zeros++     │
  │  Set:            set.add(arr[right])           │
  │  Product:        windowProd *= arr[right]      │
  │  Max in window:  (need monotonic deque)        │
  └────────────────────────────────────────────────┘
```

---

## Contraction Mechanics

When element leaves the window (left moves right):

```
  ┌────────────────────────────────────────────────┐
  │  Sum:            windowSum -= arr[left]        │
  │  Frequency:      freq[arr[left]]--             │
  │  Distinct count: if freq becomes 0, count--    │
  │  Zero count:     if arr[left]==0, zeros--      │
  │  Set:            if freq==0, set.remove(...)   │
  │  Product:        windowProd /= arr[left]       │
  │  Max in window:  if deque.front()==left, pop   │
  └────────────────────────────────────────────────┘
```

---

## Complete Worked Example: Longest with At Most K Distinct

```
  s = "araaci",  k = 2

  State: freq map + distinct counter
  Pattern: LONGEST → contract while INVALID (distinct > k)

  r=0, 'a': freq={a:1}, distinct=1, valid → best=1
  r=1, 'r': freq={a:1,r:1}, distinct=2, valid → best=2
  r=2, 'a': freq={a:2,r:1}, distinct=2, valid → best=3
  r=3, 'a': freq={a:3,r:1}, distinct=2, valid → best=4
  r=4, 'c': freq={a:3,r:1,c:1}, distinct=3, INVALID!
     contract: remove 'a', freq={a:2,r:1,c:1}, distinct=3, INVALID
     contract: remove 'r', freq={a:2,c:1}, distinct=2, VALID → best=4
     left=2
  r=5, 'i': freq={a:2,c:1,i:1}, distinct=3, INVALID!
     contract: remove 'a', freq={a:1,c:1,i:1}, distinct=3, INVALID
     contract: remove 'a', freq={c:1,i:1}, distinct=2, VALID → best=4
     left=4

  Answer: 4  ("araa")  ✓
```

---

## Monotonicity Requirement

For sliding window to work, the condition must be **monotonic**:

```
  For LONGEST problems:
  • If window [left..right] is INVALID,
    window [left..right+1] might become MORE invalid (never helps)
    window [left+1..right] might become VALID (shrinking helps)

  For SHORTEST problems:
  • If window [left..right] is VALID,
    window [left+1..right] might still be VALID (keep shrinking)
    window [left..right+1] stays VALID (growing doesn't help)

  ┌──────────────────────────────────────────────────┐
  │  Non-monotonic condition → sliding window FAILS  │
  │                                                  │
  │  Example: "subarray sum equals exactly k"         │
  │  Adding elements can increase or decrease sum     │
  │  (if negatives exist) → not monotonic            │
  │                                                  │
  │  Fix: Use prefix sums + hashmap instead          │
  └──────────────────────────────────────────────────┘
```

---

## Decision Tree: Which Pattern to Use?

```
  Given a subarray/substring problem:
  
  Is window size fixed?
    │
    ├── YES → FIXED WINDOW (Unit 4, Ch 1)
    │         slide by adding new, removing old
    │
    └── NO → VARIABLE WINDOW
              │
              What are you finding?
              │
              ├── LONGEST → Pattern A
              │   Contract while INVALID
              │   Answer AFTER contraction
              │
              ├── SHORTEST → Pattern B
              │   Contract while VALID
              │   Answer INSIDE contraction
              │
              ├── COUNT → Pattern C
              │   Like Pattern A but count += window size
              │
              └── EXACT (e.g., exactly k)
                  Use "at most k" - "at most k-1"
                  or prefix sum approach
```

---

## Practice Problem Matrix

```
  ┌────┬──────────────────────────────────┬─────────┬──────────┐
  │ #  │ Problem                          │ Pattern │ State    │
  ├────┼──────────────────────────────────┼─────────┼──────────┤
  │ 1  │ Max sum subarray size k          │ Fixed   │ sum      │
  │ 2  │ Longest no repeating chars       │ A       │ set/map  │
  │ 3  │ Min window substring             │ B       │ freq+cnt │
  │ 4  │ Longest ≤ k distinct             │ A       │ freq+cnt │
  │ 5  │ Smallest subarray sum ≥ S        │ B       │ sum      │
  │ 6  │ Max consecutive ones (flip k)    │ A       │ zero cnt │
  │ 7  │ Count subarrays product < k      │ C       │ product  │
  │ 8  │ Longest repeating replacement    │ A       │ freq+max │
  │ 9  │ Find all anagrams                │ Fixed   │ freq     │
  │ 10 │ Subarrays with k distinct        │ C+trick │ freq+cnt │
  └────┴──────────────────────────────────┴─────────┴──────────┘
```

---

## Common Pitfalls

```
  ❌ Using LONGEST pattern for SHORTEST problem (or vice versa)
     Result: Wrong answers — contraction logic is reversed

  ❌ Not updating state correctly on contraction
     E.g., forgetting to decrement distinct count when freq drops to 0

  ❌ Forgetting edge cases
     • Empty input
     • Window larger than array
     • No valid window exists

  ❌ Non-monotonic conditions
     Sliding window CANNOT handle "subarray sum == k" with negatives
     Use prefix sum + hashmap instead

  ❌ Moving left past right
     Always ensure left ≤ right + 1
     Window size 0 is valid (empty window)

  ❌ Using window for non-contiguous problems
     Sliding window = CONTIGUOUS subarrays only
     For subsequences, use DP or other techniques
```

---

## Template Summary Card

```
  ╔══════════════════════════════════════════════════╗
  ║           SLIDING WINDOW CHEAT SHEET             ║
  ╠══════════════════════════════════════════════════╣
  ║                                                  ║
  ║  FIXED (size k):                                 ║
  ║    Build first window, then slide:               ║
  ║    sum = sum + arr[i] - arr[i-k]                 ║
  ║                                                  ║
  ║  VARIABLE — LONGEST:                             ║
  ║    expand → while INVALID: contract → answer     ║
  ║                                                  ║
  ║  VARIABLE — SHORTEST:                            ║
  ║    expand → while VALID: answer + contract       ║
  ║                                                  ║
  ║  VARIABLE — COUNT:                               ║
  ║    expand → while INVALID: contract              ║
  ║    count += right - left + 1                     ║
  ║                                                  ║
  ║  EXACTLY K:                                      ║
  ║    atMost(k) - atMost(k-1)                       ║
  ║                                                  ║
  ║  TIME: O(n)   SPACE: O(1) to O(n)               ║
  ╚══════════════════════════════════════════════════╝
```

---

## Summary Table

| Aspect | Pattern A (Longest) | Pattern B (Shortest) | Pattern C (Count) |
|--------|-------------------|--------------------|--------------------|
| Goal | Maximize window | Minimize window | Count valid windows |
| Contract when | INVALID | VALID | INVALID |
| Answer location | After while loop | Inside while loop | After while loop |
| Answer update | MAX | MIN | += (right-left+1) |
| Example | No repeats | Min window sub | Subarrays with sum < k |

---

## Quick Revision Questions

1. **What is the key difference** between the "find longest" and "find shortest" sliding window patterns?

2. **In the "count valid subarrays" pattern**, why do we add `right - left + 1` instead of just 1?

3. **What property must the condition have** for sliding window to be applicable?

4. **Given a problem: "find the longest subarray with sum ≤ k"** — which pattern would you use? What would be the window state?

5. **Why doesn't sliding window work** for "subarray with exact sum k" when the array contains negative numbers?

6. **Name three different types of window state** and give a problem example for each.

---

[← Previous: Minimum Window Substring](05-minimum-window-substring.md) | [Back to README](../README.md) | [Next Unit: Prefix Sum →](../05-Prefix-Sum-and-Difference-Array/01-prefix-sum-concept.md)
