# Unit 7: Monotonic Queue

[← Previous: Queue Problems](../06-Queue-Problems/01-queue-problems.md) | [Back to README](../README.md) | [Next: Queue Applications →](../08-Queue-Applications/01-queue-applications.md)

---

## Chapter Overview

A **monotonic queue** (monotonic deque) is a powerful technique that maintains elements in a strictly increasing or decreasing order. It transforms O(n*k) sliding window problems into O(n) solutions. This unit covers the concept, classic applications, DP optimizations, and pattern recognition.

---

## 1. What is a Monotonic Queue?

A **monotonic queue** is a deque where elements are maintained in a **monotonically increasing** or **monotonically decreasing** order. When a new element violates the ordering, elements are removed from the back until the invariant is restored.

### Monotonic Decreasing Queue (for Maximum problems)

```
  All elements from front to rear are in DECREASING order.
  Front = maximum element.

  Deque:  [ 8,  5,  3,  1 ]
            ↑               ↑
          front            rear
         (max)

  New element 6 arrives:
  - Remove 1 (1 < 6)
  - Remove 3 (3 < 6)
  - Remove 5 (5 < 6)
  - Insert 6

  Deque:  [ 8,  6 ]          ← Still decreasing!
```

### Monotonic Increasing Queue (for Minimum problems)

```
  All elements from front to rear are in INCREASING order.
  Front = minimum element.

  Deque:  [ 1,  3,  5,  8 ]
            ↑               ↑
          front            rear
         (min)

  New element 4 arrives:
  - Remove 8 (8 > 4)
  - Remove 5 (5 > 4)
  - Insert 4

  Deque:  [ 1,  3,  4 ]      ← Still increasing!
```

### Core Template

```
MONOTONIC_QUEUE_TEMPLATE(arr, k):
    deque = empty deque (stores indices)
    result = []

    for i = 0 to n - 1:
        // Step 1: Remove expired elements (outside window)
        while deque not empty AND deque.front() < i - k + 1:
            deque.deleteFront()

        // Step 2: Remove elements violating monotonic property
        //   For MAX: remove while arr[deque.rear()] <= arr[i]
        //   For MIN: remove while arr[deque.rear()] >= arr[i]
        while deque not empty AND CONDITION(deque.rear(), i):
            deque.deleteRear()

        // Step 3: Insert current element
        deque.insertRear(i)

        // Step 4: Record answer (once window is formed)
        if i >= k - 1:
            result.append(arr[deque.front()])

    return result
```

### Why It Works

```
  Key Insight: We store INDICES, not values.
  
  ┌──────────────────────────────────────────────────────┐
  │  INVARIANT (for max problems):                        │
  │                                                       │
  │  1. Deque contains indices in INCREASING order         │
  │  2. Values at those indices are in DECREASING order    │
  │  3. All indices are within the current window          │
  │  4. Front index → current window maximum              │
  │                                                       │
  │  WHY remove smaller elements?                         │
  │  If arr[j] ≤ arr[i] and j < i, then:                  │
  │  - j will leave the window BEFORE i                   │
  │  - arr[j] can NEVER be the maximum while arr[i] is    │
  │    in the window                                      │
  │  → arr[j] is useless — discard it!                    │
  └──────────────────────────────────────────────────────┘
```

---

## 2. Sliding Window Maximum

### Problem

Given array `arr` and window size `k`, find the maximum in every contiguous window of size `k`.

### Full Solution with Monotonic Decreasing Deque

```
SLIDING_WINDOW_MAX(arr, k):
    deque = []        // Stores indices, values are decreasing
    result = []
    n = len(arr)

    for i = 0 to n - 1:
        // Remove indices that are out of the window
        while deque not empty AND deque[0] < i - k + 1:
            deque.popFront()

        // Remove indices whose values are ≤ arr[i]
        // (they can never be maximum while arr[i] exists)
        while deque not empty AND arr[deque[-1]] <= arr[i]:
            deque.popBack()

        deque.pushBack(i)

        // Window is fully formed when i >= k - 1
        if i >= k - 1:
            result.append(arr[deque[0]])

    return result
```

### Detailed Trace

```
arr = [1, 3, -1, -3, 5, 3, 6, 7],  k = 3

┌─────┬───────┬────────────────────────────┬─────────────────┬────────┐
│  i  │ arr[i]│  Deque (indices)           │ Deque (values)  │ Result │
├─────┼───────┼────────────────────────────┼─────────────────┼────────┤
│  0  │   1   │  [0]                       │  [1]            │  -     │
│  1  │   3   │  Remove 0 (1≤3); [1]      │  [3]            │  -     │
│  2  │  -1   │  [1, 2]                    │  [3, -1]        │  3     │
│  3  │  -3   │  [1, 2, 3]                 │  [3, -1, -3]    │  3     │
│  4  │   5   │  Expire 1; Rm 3,2; [4]    │  [5]            │  5     │
│  5  │   3   │  [4, 5]                    │  [5, 3]         │  5     │
│  6  │   6   │  Rm 5,4; [6]              │  [6]            │  6     │
│  7  │   7   │  Rm 6; [7]                │  [7]            │  7     │
└─────┴───────┴────────────────────────────┴─────────────────┴────────┘

Result: [3, 3, 5, 5, 6, 7]

Detailed breakdown for i=4 (arr[4]=5):
  Before: deque = [1, 2, 3], values = [3, -1, -3]
  
  1. Expire check: deque[0]=1, window starts at 4-3+1=2
     1 < 2 → remove 1 from front → deque = [2, 3]
  
  2. Monotonic check:
     arr[deque[-1]] = arr[3] = -3 ≤ 5 → remove 3 → deque = [2]
     arr[deque[-1]] = arr[2] = -1 ≤ 5 → remove 2 → deque = []
  
  3. Insert 4: deque = [4]
  
  4. i=4 ≥ k-1=2 → result: arr[deque[0]] = arr[4] = 5 ✓
```

### Visual: Deque Maintains "Potential Maximums"

```
  Window [2..4]: elements = [-1, -3, 5]

  Deque at i=4: [4]  →  Only 5 is a "potential maximum"
  
  Why not keep -1 or -3?
  Because 5 is larger AND newer (later index).
  5 will be in the window at least as long as -1 and -3.
  So -1 and -3 can NEVER be the answer while 5 is around.

      ┌──────────── 5 dominates ────────────┐
      │                                      │
  ..., -1, -3, [5], ..., ..., ...
                ↑
          5 entered the window.
          -1 and -3 will leave the window
          BEFORE 5 does. They are useless.
```

### Complexity

| Metric | Value | Reason |
|--------|:-----:|--------|
| Time | **O(n)** | Each index enters and leaves deque at most once |
| Space | **O(k)** | Deque holds at most k indices |

---

## 3. Sliding Window Minimum

### Problem

Same as sliding window maximum, but find the **minimum** in each window.

### Solution: Monotonic Increasing Deque

The only change: reverse the comparison in step 2.

```
SLIDING_WINDOW_MIN(arr, k):
    deque = []
    result = []

    for i = 0 to n - 1:
        // Remove expired
        while deque not empty AND deque[0] < i - k + 1:
            deque.popFront()

        // Remove elements >= arr[i] (they can never be minimum)
        while deque not empty AND arr[deque[-1]] >= arr[i]:  // ← Changed!
            deque.popBack()

        deque.pushBack(i)

        if i >= k - 1:
            result.append(arr[deque[0]])

    return result
```

### Trace

```
arr = [4, 2, 5, 1, 3, 6],  k = 3

┌─────┬───────┬────────────────────┬─────────────────┬────────┐
│  i  │ arr[i]│  Deque (indices)   │ Deque (values)  │ Result │
├─────┼───────┼────────────────────┼─────────────────┼────────┤
│  0  │   4   │  [0]               │  [4]            │  -     │
│  1  │   2   │  Rm 0(4≥2); [1]  │  [2]            │  -     │
│  2  │   5   │  [1, 2]            │  [2, 5]         │  2     │
│  3  │   1   │  Exp 1; Rm 2; [3] │  [1]            │  1     │
│  4  │   3   │  [3, 4]            │  [1, 3]         │  1     │
│  5  │   6   │  Exp 3; [4, 5]    │  [3, 6]         │  3     │
└─────┴───────┴────────────────────┴─────────────────┴────────┘

Result: [2, 1, 1, 3]

INCREASING deque: front always holds the minimum!
```

### Max vs Min Comparison

| Problem | Deque Order | Remove When | Front Holds |
|---------|:-----------:|:-----------:|:-----------:|
| Window Maximum | Decreasing | `arr[rear] ≤ arr[i]` | Maximum |
| Window Minimum | Increasing | `arr[rear] ≥ arr[i]` | Minimum |

---

## 4. Jump Game Optimization

### Problem

Given an array `arr` of length `n`, starting at index 0, at each index `i` you can jump to any index `j` where `i+1 ≤ j ≤ i+k`. Each index has a cost `arr[j]`. Find the minimum cost to reach index `n-1`.

```
Equivalently: dp[i] = arr[i] + min(dp[j]) for j in [i-k, i-1]
```

### Brute Force DP — O(n*k)

```
dp[0] = arr[0]
for i = 1 to n-1:
    dp[i] = arr[i] + min(dp[i-k], dp[i-k+1], ..., dp[i-1])
```

### Optimized with Monotonic Queue — O(n)

```
JUMP_GAME_MIN_COST(arr, k):
    n = len(arr)
    dp = new array[n]
    dp[0] = arr[0]
    deque = [0]    // Monotonic increasing deque of indices

    for i = 1 to n - 1:
        // Remove expired indices (more than k steps back)
        while deque not empty AND deque[0] < i - k:
            deque.popFront()

        // dp[i] = arr[i] + minimum dp in window = dp[deque.front()]
        dp[i] = arr[i] + dp[deque[0]]

        // Maintain increasing order of dp values
        while deque not empty AND dp[deque[-1]] >= dp[i]:
            deque.popBack()

        deque.pushBack(i)

    return dp[n - 1]
```

### Trace

```
arr = [1, 3, 2, 4, 3],  k = 2

dp[0] = 1
Deque: [0]

i=1: Expire: 0 ≥ 1-2=-1 → keep
     dp[1] = 3 + dp[0] = 4
     Deque: dp[0]=1, dp[1]=4. 4≥1? No remove. Deque: [0, 1]

i=2: Expire: 0 ≥ 2-2=0 → keep
     dp[2] = 2 + dp[0] = 3
     Deque: dp[1]=4 ≥ 3? → remove 1. dp[0]=1 ≥ 3? No.
     Deque: [0, 2]

i=3: Expire: 0 < 3-2=1 → remove 0
     Deque: [2]
     dp[3] = 4 + dp[2] = 7
     dp[2]=3, dp[3]=7. 7≥3? No. Deque: [2, 3]

i=4: Expire: 2 < 4-2=2 → No (2 ≥ 2)
     dp[4] = 3 + dp[2] = 6
     dp[3]=7 ≥ 6? → remove 3. dp[2]=3 ≥ 6? No.
     Deque: [2, 4]

Answer: dp[4] = 6

Verification:
  Path: 0 → 2 → 4 = arr[0] + arr[2] + arr[4] = 1 + 2 + 3 = 6 ✓
```

### The Key DP Optimization Pattern

```
┌────────────────────────────────────────────────────────┐
│  PATTERN: DP with Sliding Window Min/Max               │
│                                                        │
│  Standard DP:                                           │
│    dp[i] = f(arr[i], min/max of dp[i-k ... i-1])       │
│    → O(n*k) because finding min/max in window is O(k)  │
│                                                        │
│  Optimized DP:                                          │
│    Use monotonic deque to maintain window min/max        │
│    → O(n) because deque provides O(1) min/max           │
│                                                        │
│  Speedup: O(n*k) → O(n)                                │
└────────────────────────────────────────────────────────┘
```

---

## 5. Constrained Subsequence Sum

### Problem

Given an integer array `nums` and an integer `k`, find the maximum sum of a non-empty subsequence such that for every two consecutive elements `nums[i]` and `nums[j]` in the subsequence, `j - i ≤ k`.

### Formulation

```
dp[i] = nums[i] + max(0, max(dp[j]) for j in [i-k, i-1])

The max(0, ...) part means we can "start fresh" — don't take
any previous element if all dp values in the window are negative.

Answer = max(dp[0], dp[1], ..., dp[n-1])
```

### Optimized Solution

```
CONSTRAINED_SUBSEQ_SUM(nums, k):
    n = len(nums)
    dp = copy of nums     // dp[i] starts with nums[i]
    deque = []            // Monotonic DECREASING deque
    result = -∞

    for i = 0 to n - 1:
        // Remove expired
        while deque not empty AND deque[0] < i - k:
            deque.popFront()

        // Best previous value (or 0 if all negative)
        if deque not empty AND dp[deque[0]] > 0:
            dp[i] = nums[i] + dp[deque[0]]

        // Track overall maximum
        result = max(result, dp[i])

        // Maintain decreasing order (we want max at front)
        while deque not empty AND dp[deque[-1]] <= dp[i]:
            deque.popBack()

        deque.pushBack(i)

    return result
```

### Trace

```
nums = [10, 2, -10, 5, 20],  k = 2

i=0: dp[0]=10, deque=[] → no prev → dp[0]=10
     Deque: [0]
     result = 10

i=1: deque front=0, dp[0]=10 > 0 → dp[1] = 2+10 = 12
     dp[1]=12 > dp[0]=10 → remove 0
     Deque: [1]
     result = 12

i=2: deque front=1, dp[1]=12 > 0 → dp[2] = -10+12 = 2
     dp[2]=2 < dp[1]=12 → keep
     Deque: [1, 2]
     result = 12

i=3: Expire: 1 < 3-2=1 → No (1 ≥ 1)
     deque front=1, dp[1]=12 > 0 → dp[3] = 5+12 = 17
     dp[3]=17 > dp[2]=2 → remove 2, > dp[1]=12 → remove 1
     Deque: [3]
     result = 17

i=4: deque front=3, dp[3]=17 > 0 → dp[4] = 20+17 = 37
     dp[4]=37 > dp[3]=17 → remove 3
     Deque: [4]
     result = 37

Answer: 37  (subsequence: 10, 2, 5, 20 via indices 0→1→3→4)
```

---

## 6. Pattern Recognition

### When to Use a Monotonic Queue

```
  ┌──────────────────────────────────────────────────────────┐
  │  RED FLAGS that suggest monotonic queue:                   │
  │                                                           │
  │  1. "Sliding window" + "maximum/minimum"                  │
  │  2. DP recurrence: dp[i] depends on min/max of dp[i-k..i-1]│
  │  3. "Subarray of size k" + "optimize from O(nk) to O(n)" │
  │  4. "Next greater/smaller element" (monotonic stack cousin)│
  │  5. "Constrained" + "subsequence" + "maximum/minimum"    │
  │  6. Any problem combining ORDER + WINDOW + EXTREMES      │
  └──────────────────────────────────────────────────────────┘
```

### Monotonic Queue vs Monotonic Stack

| Feature | Monotonic Queue | Monotonic Stack |
|---------|:---------------:|:---------------:|
| Structure | Deque (both ends) | Stack (one end) |
| Remove from | Front AND back | Top only |
| Expiration | Elements expire (window) | No expiration |
| Use case | Sliding window min/max | Next greater/smaller element |
| Window bound | Yes (size k) | No |
| Time | O(n) | O(n) |

### Problem Classification

| Problem | Type | Deque Order | What's at Front |
|---------|:----:|:-----------:|:---------------:|
| Sliding window max | Decreasing | Large→Small | Maximum |
| Sliding window min | Increasing | Small→Large | Minimum |
| Jump game (min cost) | Increasing | Small→Large | Min dp value |
| Constrained subseq sum | Decreasing | Large→Small | Max dp value |
| Shortest subarray sum ≥ K | Increasing | Small→Large | Best start |

### Common Mistakes to Avoid

```
┌────────────────────────────────────────────────────────────┐
│  MISTAKE 1: Storing values instead of indices               │
│  Fix: Store INDICES — you need them for expiration checks   │
│                                                             │
│  MISTAKE 2: Forgetting to remove expired front elements     │
│  Fix: Always check deque[0] < i - k + 1 FIRST              │
│                                                             │
│  MISTAKE 3: Wrong comparison direction (≤ vs ≥)             │
│  Fix: For MAX: remove rear when arr[rear] ≤ arr[i]          │
│       For MIN: remove rear when arr[rear] ≥ arr[i]          │
│                                                             │
│  MISTAKE 4: Off-by-one in window bounds                     │
│  Fix: Window [i-k+1, i] has size k. First result at i=k-1  │
│                                                             │
│  MISTAKE 5: Thinking it's O(n*k) due to nested while loops  │
│  Fix: Each element enters and exits deque ONCE → O(n) total │
└────────────────────────────────────────────────────────────┘
```

### Decision Tree

```
  Given a problem:

  Is there a sliding window / bounded range?
    │
    ├── YES → Need max in window? → Monotonic DECREASING deque
    │       → Need min in window? → Monotonic INCREASING deque
    │
    └── NO → Is it about next greater/smaller?
              │
              ├── YES → Use monotonic STACK
              │
              └── NO  → Probably not a monotonic DS problem
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Monotonic queue | Deque maintaining increasing or decreasing order |
| Decreasing deque | Front = maximum; for sliding window max problems |
| Increasing deque | Front = minimum; for sliding window min problems |
| Store indices | Essential for checking window expiration |
| Expiration check | Remove front elements outside window bounds |
| Rear removal | Remove elements that can never be the answer |
| Time complexity | O(n) — each element enters/exits deque once |
| DP optimization | Convert O(n*k) DP to O(n) using monotonic deque |
| vs Monotonic stack | Stack has no expiration, used for next greater/smaller |
| Key problems | Sliding max/min, jump game, constrained subsequence |

---

## Quick Revision Questions

1. **What is the invariant maintained by a monotonic decreasing queue? What information does the front element provide?**

2. **In the sliding window maximum, why do we remove elements from the rear when `arr[deque.rear()] ≤ arr[i]`? What does this accomplish?**

3. **Why do we store indices in the deque rather than values? What would break if we stored values?**

4. **How does a monotonic queue optimize a DP recurrence from O(n*k) to O(n)? Explain the key insight.**

5. **What is the difference between a monotonic queue and a monotonic stack? When would you use each?**

6. **Given a sliding window minimum problem, what order should the deque maintain, and what comparison do you use when inserting?**

---

## Answers (Hidden — Think First!)

<details>
<summary>Click to reveal answers</summary>

1. **Invariant:** Values at indices stored in the deque are in strictly decreasing order from front to rear. All indices are within the current window. **Front element:** The index of the current window's maximum value.

2. If `arr[deque.rear()] ≤ arr[i]`, the rear element is smaller (or equal) AND older (earlier index). It will leave the window before `arr[i]`, so it can **never** be the maximum in any future window containing `arr[i]`. Removing it keeps only "useful potential maximums" in the deque.

3. We need indices to check **expiration** — whether an element has left the window (`deque[0] < i - k + 1`). If we stored only values, we couldn't determine which window an element belongs to, couldn't expire old elements, and might report maximums from outside the window.

4. The DP recurrence `dp[i] = f(min/max of dp[i-k..i-1])` normally requires scanning k elements for the min/max → O(k) per step → O(n*k) total. The monotonic deque maintains the window's min/max at its front in O(1). Since each element enters and exits the deque once, the total maintenance cost is O(n), making the overall DP O(n).

5. **Monotonic queue (deque):** Removes from both front (expiration) and rear (ordering). Used for **sliding window** problems with bounded windows. **Monotonic stack:** Removes from top only (no expiration). Used for **next greater/smaller element** problems without a window constraint. Queue has a window bound; stack doesn't.

6. The deque should maintain **increasing order** (smallest at front). When inserting index `i`, remove from the rear while `arr[deque.rear()] ≥ arr[i]` — elements larger than `arr[i]` can never be the minimum while `arr[i]` is in the window.

</details>

---

[← Previous: Queue Problems](../06-Queue-Problems/01-queue-problems.md) | [Back to README](../README.md) | [Next: Queue Applications →](../08-Queue-Applications/01-queue-applications.md)
