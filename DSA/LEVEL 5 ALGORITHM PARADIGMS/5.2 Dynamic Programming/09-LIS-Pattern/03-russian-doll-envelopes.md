# Chapter 3: Russian Doll Envelopes

## 📋 Overview
Given envelopes with (width, height), find the **maximum number of envelopes you can nest** (Russian doll style). An envelope fits inside another only if **both** width and height are strictly smaller. This reduces to **LIS in 2D**.

---

## 🧠 Core Concept

```
envelopes = [[5,4],[6,4],[6,7],[2,3]]

Nesting chain: [2,3] → [5,4] → [6,7]  (length 3)
  2<5 & 3<4 ✓,  5<6 & 4<7 ✓

Cannot: [5,4] → [6,4]  (4 is NOT < 4, need strictly smaller)
Cannot: [6,4] → [6,7]  (6 is NOT < 6)

Answer: 3
```

---

## 🔨 Key Insight: Sort + LIS

```
Strategy:
  1. Sort by width ASCENDING
  2. For same width, sort by height DESCENDING
  3. Find LIS on heights only

Why descending height for same width?
  ┌─────────────────────────────────────────────┐
  │ envelopes: [6,4] and [6,7]                 │
  │ Same width 6 — can't nest either inside    │
  │                                             │
  │ If heights sorted ascending: [6,4],[6,7]   │
  │   LIS on heights: [4,7] — WRONG! picks both│
  │   But width 6=6, can't nest!               │
  │                                             │
  │ If heights sorted DESCENDING: [6,7],[6,4]  │
  │   LIS on heights: [7] or [4] — length 1   │
  │   Correct! Can only pick one with width 6  │
  └─────────────────────────────────────────────┘

Descending height ensures we NEVER pick two envelopes 
with the same width in our LIS.
```

---

## 💻 Pseudocode

```
function maxEnvelopes(envelopes):
    // Sort by width ascending, then height descending
    sort envelopes by (width ASC, height DESC)
    
    // Extract heights
    heights = [env[1] for env in envelopes]
    
    // Find LIS on heights using O(n log n) approach
    return LIS(heights)

function LIS(nums):
    tails = []
    for num in nums:
        lo = 0, hi = len(tails)
        while lo < hi:
            mid = (lo + hi) / 2
            if tails[mid] < num:
                lo = mid + 1
            else:
                hi = mid
        if lo == len(tails):
            tails.append(num)
        else:
            tails[lo] = num
    return len(tails)
```

---

## 🔬 Trace

```
envelopes = [[5,4],[6,4],[6,7],[2,3]]

Step 1: Sort (width ASC, height DESC):
  [[2,3],[5,4],[6,7],[6,4]]
         ↑          ↑
  Width 6 appears twice → height sorted DESC: 7 before 4

Step 2: Extract heights = [3, 4, 7, 4]

Step 3: LIS on [3, 4, 7, 4]:
  tails = []
  
  num=3: append → tails=[3]
  num=4: 4>3, append → tails=[3,4]
  num=7: 7>4, append → tails=[3,4,7]
  num=4: 4<7, binary search → pos=1 (tails[1]=4≥4)
         tails[1]=4 → tails=[3,4,7] (no change since 4=4)
  
  LIS length = 3 ✓
  
  Corresponds to envelopes: [2,3]→[5,4]→[6,7]
```

---

## 🌐 3D Extension: Box Stacking

```
Given boxes with (length, width, height).
Stack boxes such that each lower box is strictly larger 
in ALL dimensions.

Approach:
  1. Sort by one dimension (e.g., length)
  2. Use O(n²) DP (can't easily reduce to 1D LIS)

function maxStack(boxes):
    sort boxes by length ASC
    dp = array of size n, all 1
    
    for i = 1 to n-1:
        for j = 0 to i-1:
            if boxes[j].l < boxes[i].l AND
               boxes[j].w < boxes[i].w AND
               boxes[j].h < boxes[i].h:
                dp[i] = max(dp[i], dp[j] + 1)
    
    return max(dp)

Note: For 3+ dimensions, the sort trick doesn't extend
to O(n log n). Must use O(n²) DP or advanced structures.
```

---

## ⚠️ Edge Cases

```
1. All same width: Only one envelope can be selected
   [[3,5],[3,4],[3,3]] → After sort: [[3,5],[3,4],[3,3]]
   Heights: [5,4,3] → LIS = 1

2. All same height: Same as LIS on widths
   [[1,3],[2,3],[3,3]] → After sort: [[1,3],[2,3],[3,3]]
   Heights: [3,3,3] → LIS = 1
   (Can't nest since heights are equal, not strictly less)

3. Single envelope: Answer is 1

4. All nestable:
   [[1,1],[2,2],[3,3]] → Heights: [1,2,3] → LIS = 3
```

---

## 📊 Complexity

| Approach | Sort | LIS | Total Time | Space |
|----------|------|-----|-----------|-------|
| Sort + O(n²) LIS | O(n log n) | O(n²) | O(n²) | O(n) |
| Sort + O(n log n) LIS | O(n log n) | O(n log n) | O(n log n) | O(n) |

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Sorting** | Width ASC, Height DESC |
| **Why DESC height** | Prevents picking two envelopes with same width |
| **After sorting** | LIS on heights only |
| **Complexity** | O(n log n) with binary search LIS |
| **3D extension** | O(n²) DP, sort trick doesn't generalize |
| **Strict inequality** | Both dimensions must be strictly less |

---

## ❓ Quick Revision Questions

1. **Why do we sort height in descending order for same width?**
2. **After sorting, why does LIS on heights give the correct answer?**
3. **What happens if we sort both width and height ascending?**
4. **Can this approach extend to 3D (box stacking)?**
5. **What is the optimal time complexity and which LIS algorithm achieves it?**
6. **What if the problem uses ≤ instead of < for nesting?**

---

[← Previous: Number of LIS](02-number-of-lis.md) | [Next: Maximum Sum Increasing Subsequence →](04-max-sum-increasing-subsequence.md)

[← Back to README](../README.md)
