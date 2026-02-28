# Unit 11: Sorting Algorithm Comparison — Stability Analysis

[← Previous: Comparison Table](01-comparison-table.md) | [Next: Time Complexity Deep Dive →](03-time-complexity-deep-dive.md)

---

## Overview

Stability is one of the most important — and most frequently misunderstood — properties of a sorting algorithm. This chapter explains what stability means, why it matters, and which algorithms guarantee it.

---

## What Is Stability?

```
  A sorting algorithm is STABLE if elements with equal keys
  maintain their original relative order after sorting.
  
  Original:     (Alice, 85)  (Bob, 90)  (Carol, 85)  (Dave, 90)
                     ①           ②          ③            ④
  
  Sort by score:
  
  STABLE result:
    (Alice, 85)  (Carol, 85)  (Bob, 90)  (Dave, 90)
        ①            ③           ②           ④
    ↑ Alice before Carol (same score, original order kept) ✓
    ↑ Bob before Dave    (same score, original order kept) ✓
  
  UNSTABLE result (also valid, but loses relative order):
    (Carol, 85)  (Alice, 85)  (Dave, 90)  (Bob, 90)
        ③            ①           ④           ②
    ↑ Carol before Alice?  Original had Alice first! ✗
```

---

## Stability Classification

```
  ┌──────────────────────────────────────────────────────────┐
  │              STABLE ALGORITHMS                          │
  ├──────────────────────────────────────────────────────────┤
  │  Bubble Sort     — adjacent swaps only                  │
  │  Insertion Sort  — shifts right, never jumps past equal │
  │  Merge Sort      — left element wins on tie            │
  │  Counting Sort   — right-to-left placement preserves   │
  │  Radix Sort      — uses stable sub-sort (counting)     │
  │  Timsort         — merge + insertion (both stable)     │
  ├──────────────────────────────────────────────────────────┤
  │              UNSTABLE ALGORITHMS                        │
  ├──────────────────────────────────────────────────────────┤
  │  Selection Sort  — swaps distant elements              │
  │  Quick Sort      — partition swaps across equal keys   │
  │  Heap Sort       — heap operations destroy order       │
  ├──────────────────────────────────────────────────────────┤
  │              DEPENDS ON IMPLEMENTATION                  │
  ├──────────────────────────────────────────────────────────┤
  │  Bucket Sort     — stable iff inner sort is stable     │
  └──────────────────────────────────────────────────────────┘
```

---

## Why Each Algorithm Is or Isn't Stable

### Bubble Sort — STABLE

```
  Only swaps ADJACENT elements, and only when left > right.
  Equal elements are NEVER swapped.
  
  [3a, 3b, 2, 1]
  Pass 1: [3a, 2, 1, 3b]  → 3a and 3b never swapped
  Pass 2: [2, 1, 3a, 3b]  → 3a still before 3b ✓
  Pass 3: [1, 2, 3a, 3b]  → stable ✓
```

### Insertion Sort — STABLE

```
  Shifts elements right only when element > key (NOT ≥).
  Equal elements stay in front of the key.
  
  [3a, 2, 3b, 1]
  Insert 2:  [2, 3a, 3b, 1]  — 3a stays before 3b
  Insert 3b: [2, 3a, 3b, 1]  — 3b stays after 3a (3a = 3b, no shift)
  Insert 1:  [1, 2, 3a, 3b]  — stable ✓
  
  KEY: use ">" not ">=" in comparison
  while (j >= 0 && arr[j] > key)   ← STABLE (equal stays)
  while (j >= 0 && arr[j] >= key)  ← UNSTABLE!
```

### Merge Sort — STABLE

```
  During the MERGE step, when left[i] == right[j],
  we take from the LEFT array first.
  
  Left:  [3a, 5a]    Right: [3b, 5b]
  
  Merge: 3a vs 3b → take 3a (left wins tie)
         5a vs 3b → take 3b
         5a vs 5b → take 5a (left wins tie)
         take 5b
  
  Result: [3a, 3b, 5a, 5b] ← original order preserved ✓
  
  KEY: use "<=" for left element:
  if (left[i] <= right[j])   ← STABLE
  if (left[i] < right[j])    ← UNSTABLE (right wins tie)
```

### Selection Sort — UNSTABLE

```
  Swaps the minimum element with the current position.
  This JUMPS over equal elements.
  
  [3a, 3b, 1]
  
  Find min (index 2) → swap with position 0:
  [1, 3b, 3a]     ← 3a moved AFTER 3b! Unstable! ✗
  
  The long-distance swap destroys relative order.
```

### Quick Sort — UNSTABLE

```
  Partition swaps elements across the pivot.
  
  [4a, 3, 4b, 2]   pivot = 4b
  
  Lomuto partition:
  i = -1
  j=0: 4a > 4b? No swap... wait, 4a = 4b → no swap
  j=1: 3 < 4b → swap arr[0] and arr[1] → [3, 4a, 4b, 2]
  j=2: (pivot)
  j=3: 2 < 4b → swap arr[1] and arr[3] → [3, 2, 4b, 4a]
  Swap pivot: [3, 2, 4b, 4a]
  
  4a now AFTER 4b → unstable ✗
```

### Heap Sort — UNSTABLE

```
  Building a heap rearranges elements based on the heap property,
  destroying the original order of equal elements.
  
  [3a, 3b, 1]
  
  Build max-heap: [3a, 3b, 1]  or  [3b, 3a, 1]
  (heap property doesn't specify order of equal children)
  
  Extract max (3a or 3b): whichever is at root
  Result may have 3b before 3a → unstable ✗
```

### Counting Sort — STABLE

```
  Right-to-left placement in the output array preserves order.
  
  Input:  [A=3, B=1, C=3, D=2]   (A,C have same key 3)
  
  Count:  [0, 1, 1, 2]    (0 zeros, 1 one, 1 two, 2 threes)
  Prefix: [0, 1, 2, 4]    (cumulative)
  
  Right-to-left placement:
  D=2: output[prefix[2]-1] = output[1] → prefix[2]=1
  C=3: output[prefix[3]-1] = output[3] → prefix[3]=3
  A=3: output[prefix[3]-1] = output[2] → prefix[3]=2  ← ERROR if left-to-right
  
  Wait—process RIGHT-TO-LEFT:
  D=2: output[1]
  C=3: output[3]
  B=1: output[0]
  A=3: output[2]  ← A (earlier) gets lower index → A before C ✓
  
  Result: [B=1, D=2, A=3, C=3]  → A before C, stable ✓
```

---

## Why Stability Matters

### Use Case 1: Multi-Key Sorting

```
  Sort by LAST NAME, then by FIRST NAME:
  
  Original:
  (Bob, Smith)  (Alice, Smith)  (Charlie, Jones)  (Alice, Jones)
  
  Step 1: Sort by FIRST NAME (stable sort):
  (Alice, Smith)  (Alice, Jones)  (Bob, Smith)  (Charlie, Jones)
  
  Step 2: Sort by LAST NAME (stable sort):
  (Alice, Jones)  (Charlie, Jones)  (Alice, Smith)  (Bob, Smith)
     ↑ Alices maintain first-name order within same last name ✓
  
  With UNSTABLE sort in step 2:
  (Charlie, Jones)  (Alice, Jones)  (Bob, Smith)  (Alice, Smith)
     ↑ Jones group order changed! Step 1's work was lost! ✗
```

### Use Case 2: Radix Sort Depends on Stability

```
  Radix sort processes digits from least significant to most.
  
  If the sub-sort (counting sort) were unstable:
  
  Sort [170, 90, 802, 2, 24, 45, 75, 66]
  
  By 1s digit: [170, 90, 802, 2, 24, 45, 75, 66]
  By 10s digit: If not stable, 802 and 2 (same 10s digit = 0)
                could swap, destroying the 1s digit ordering.
  
  Stability ensures each digit pass PRESERVES all previous passes.
```

### Use Case 3: Database Operations

```
  "Sort customers by region, then by revenue within each region"
  
  Stable sort: sort by revenue first, then by region.
  The region sort preserves the revenue ordering within each group.
  
  Equivalent to SQL:  ORDER BY region, revenue
```

---

## Making Unstable Sorts Stable

### Technique: Augment with Original Index

```java
// Wrap each element with its original index
int[][] augmented = new int[n][2];
for (int i = 0; i < n; i++) {
    augmented[i][0] = arr[i];   // value
    augmented[i][1] = i;         // original index
}

// Sort with tie-breaking by original index
Arrays.sort(augmented, (a, b) -> {
    if (a[0] != b[0]) return Integer.compare(a[0], b[0]);
    return Integer.compare(a[1], b[1]);  // break ties
});

// Extract sorted values
for (int i = 0; i < n; i++) {
    arr[i] = augmented[i][0];
}
```

**Cost**: O(n) extra space, O(n log n) time unchanged.

---

## Summary Table

| Algorithm | Stable? | Why / How |
|-----------|---------|-----------|
| Bubble Sort | Yes | Adjacent swaps, `>` not `>=` |
| Insertion Sort | Yes | Shifts right on `>`, not `>=` |
| Merge Sort | Yes | Left element wins on equal |
| Counting Sort | Yes | Right-to-left output placement |
| Radix Sort | Yes | Uses stable counting sort |
| Selection Sort | No | Long-distance swap |
| Quick Sort | No | Partition crosses equal elements |
| Heap Sort | No | Heap structure ignores original order |
| Bucket Sort | Depends | Stable if inner sort is stable |

---

## Quick Revision Questions

1. **Define stability and give a concrete example showing why it matters.**
2. **Why does changing `>` to `>=` in insertion sort break stability?**
3. **How does merge sort maintain stability during the merge step?**
4. **Show with an example why selection sort is unstable.**
5. **How can you make any unstable sort stable? What's the cost?**
6. **Why is stability essential for radix sort to work correctly?**

---

[← Previous: Comparison Table](01-comparison-table.md) | [Next: Time Complexity Deep Dive →](03-time-complexity-deep-dive.md)
