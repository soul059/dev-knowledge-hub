# Chapter 6: When to Apply Sliding Window

## 📋 Chapter Overview
A definitive guide to recognizing when sliding window is the right approach — and when it's NOT.

---

## ✅ When TO Use Sliding Window

```
  ┌─────────────────────────────────────────────────────────┐
  │          SLIDING WINDOW SIGNAL CHECKLIST                │
  │                                                         │
  │  ✓ Problem involves CONTIGUOUS subarray or substring    │
  │  ✓ Data is LINEAR (array or string)                     │
  │  ✓ Looking for longest/shortest/count/max/min           │
  │  ✓ Window property can be maintained INCREMENTALLY      │
  │  ✓ Adding/removing one element has O(1) state update    │
  │                                                         │
  │  If ALL of these are true → SLIDING WINDOW              │
  └─────────────────────────────────────────────────────────┘
```

### Keyword Triggers:

| Keyword/Phrase | Variant |
|----------------|---------|
| "subarray of size k" | Fixed Window |
| "longest substring" | Variable Window (longest) |
| "shortest subarray" | Variable Window (shortest) |
| "count subarrays with..." | Count Window |
| "contiguous sequence" | Window |
| "at most K distinct" | Variable + HashMap |
| "maximum sum of size K" | Fixed + Sum |
| "anagram/permutation of pattern" | Fixed + Frequency |
| "without repeating" | Variable + Set |

---

## ❌ When NOT to Use Sliding Window

```
  ┌─────────────────────────────────────────────────────────┐
  │         SLIDING WINDOW DOES NOT WORK WHEN:              │
  │                                                         │
  │  ✗ Subsequence (not contiguous) → Use DP               │
  │  ✗ Best pair (not window) → Use Two Pointers / HashMap │
  │  ✗ All subsets → Use Backtracking                       │
  │  ✗ Tree/Graph → Use BFS/DFS                            │
  │  ✗ Negative numbers + exact sum → Use Prefix Sum + Map │
  │  ✗ Window state can't be updated in O(1)               │
  │  ✗ Non-contiguous elements                              │
  └─────────────────────────────────────────────────────────┘
```

### Common Confusion:

| Problem | Looks Like Window? | Actually |
|---------|-------------------|----------|
| "Longest increasing subsequence" | Yes | DP (non-contiguous!) |
| "Subarray sum = K (with negatives)" | Yes | Prefix Sum + HashMap |
| "Two sum" | No | HashMap / Two Pointers |
| "Maximum product subarray" | Maybe | DP (negative flips sign) |
| "Number of subarrays with odd sum" | Maybe | Prefix sum parity |

---

## 🗺️ Decision Flowchart

```
  ┌─────────────────────────────────────┐
  │ Is it about CONTIGUOUS subarray     │
  │ or substring?                       │
  └────────────┬──────────┬─────────────┘
               │          │
              YES         NO
               │          │
               ▼          ▼
  ┌────────────────┐   Use DP, Two Ptrs,
  │Can window state│   Backtrack, etc.
  │be updated O(1)?│
  └───┬────────┬───┘
     YES      NO
      │        │
      ▼        ▼
  ┌──────┐  Consider Prefix
  │Is the│  Sum, Segment Tree,
  │window│  or other approach
  │size  │
  │known?│
  └─┬──┬─┘
   YES  NO
    │    │
    ▼    ▼
  Fixed  Variable
  Window Window
    │    │
    ▼    ▼
  ┌──────────────────────────┐
  │ Looking for max/longest? │──► Variant: Longest
  │ Looking for min/shortest?│──► Variant: Shortest
  │ Looking for count?       │──► Variant: Count
  │ Need exact K?            │──► atMost(K)-atMost(K-1)
  └──────────────────────────┘
```

---

## 📊 Pattern Comparison: Sliding Window vs Similar Patterns

| Feature | Sliding Window | Two Pointers | Prefix Sum |
|---------|---------------|-------------|------------|
| Data | Contiguous | Sorted/pairs | Running total |
| Pointers | left/right (same dir) | Various directions | Single scan |
| State | Window state | Comparison | Cumulative |
| Negative nums | Tricky | Works | Works |
| Time | O(n) | O(n) | O(n) |
| Best for | Subarray/substring | Pairs, sorted | Exact sums, range queries |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Use When | Contiguous, linear, incremental state update |
| Don't Use | Non-contiguous, graphs/trees, negative numbers + exact sum |
| Fixed vs Variable | Known window size → fixed; condition-based → variable |
| Confusion Points | Subsequence ≠ subarray; exact sum with negatives needs prefix sum |
| Keywords | "subarray", "substring", "contiguous", "window", "at most K" |

---

## ❓ Quick Revision Questions

1. **What is the single most important property for sliding window to work?**
   > The problem must involve **contiguous** elements (subarray or substring), and the window state must be updatable in O(1).

2. **Why can't you use sliding window for "longest increasing subsequence"?**
   > Because a subsequence is non-contiguous — elements don't need to be adjacent.

3. **When negative numbers appear in a subarray sum problem, what should you use instead?**
   > Prefix Sum + HashMap approach.

4. **What determines fixed vs variable window?**
   > If the window size is given or computable upfront → fixed. If it depends on a condition → variable.

5. **Name three keywords that signal sliding window.**
   > "contiguous subarray", "longest substring", "at most K distinct".

6. **Can sliding window work on linked lists?**
   > Technically yes for singly-linked lists (same direction traversal), but it's uncommon. It's primarily for arrays and strings.

---

[← Previous: Optimization Tricks](05-optimization-tricks.md)

[← Back to README](../README.md) | [Next Unit: Two Pointers Pattern →](../03-Two-Pointers-Pattern/01-same-direction.md)
