# Chapter 2: Uniform Distribution & When Interpolation Shines

[← Previous: Concept](01-concept.md) | [Next: Comparison & Practice →](03-comparison-and-practice.md)

---

## Why Distribution Matters

```
   Interpolation search's performance depends ENTIRELY
   on how the data is distributed:
   
   ┌──────────────────────────────────────────┐
   │  Uniform distribution → O(log log n) 🎯  │
   │  Skewed distribution  → O(n) 💥          │
   └──────────────────────────────────────────┘
   
   The formula ASSUMES values are roughly evenly spaced.
   When they are, the probe position is extremely accurate.
   When they aren't, the probe may be way off.
```

---

## Uniform vs Skewed Distribution

```
   UNIFORM: Values are evenly spaced
   ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬────┐
   │10 │20 │30 │40 │50 │60 │70 │80 │90 │100 │
   └───┴───┴───┴───┴───┴───┴───┴───┴───┴────┘
   
   Interpolation is incredibly accurate:
   target=70 → pos = (70-10)/(100-10) × 9 = 6.0 → EXACT! 🎯
   
   
   SKEWED: Values are clustered
   ┌───┬───┬───┬───┬───┬───┬───┬───┬────┬─────┐
   │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │ 8 │500│1000 │
   └───┴───┴───┴───┴───┴───┴───┴───┴────┴─────┘
   
   Interpolation is misleading:
   target=8 → pos = (8-1)/(1000-1) × 9 = 0.063 → pos=0 ❌
   Only eliminates 1 element per step → becomes O(n)
```

---

## The O(log log n) Analysis — Intuition

```
   WHY is interpolation O(log log n) for uniform data?
   
   With uniform distribution, the probe hits approximately
   the correct position. The error is proportional to √(range).
   
   After each probe:
   - Binary search: range → range/2 (halves)
   - Interpolation: range → √(range) (square roots!)
   
   How many times can you take √ of n before reaching 1?
   
   n → √n → √(√n) → √(√(√n)) → ... → 1
   
   This is log₂(log₂(n)) steps!
   
   Example: n = 2^32 (≈ 4 billion)
   log₂(n) = 32
   log₂(32) = 5
   
   Binary search:        32 comparisons
   Interpolation search:  5 comparisons! 🚀
```

---

## Formal Expected Complexity

```
   Theorem (Perl, Itai, Avni, 1978):
   
   For n elements drawn uniformly at random:
   
   E[comparisons] = log₂(log₂(n)) + O(1)
   
   Proof sketch:
   After one interpolation step on uniform data in [lo, hi]:
   - Expected new range size ≈ √(hi - lo)
   - Recurrence: T(n) = T(√n) + O(1)
   - Let n = 2^k: T(2^k) = T(2^(k/2)) + O(1)
   - Let k = 2^j: T(2^(2^j)) = T(2^(2^(j-1))) + O(1) = O(j)
   - Since k = log n, j = log k = log log n
   
   Therefore T(n) = O(log log n) ✓
```

---

## Numbers That Show the Power

| n | log₂ n | log₂ log₂ n | Speedup |
|---|--------|-------------|---------|
| 256 | 8 | 3 | 2.7× |
| 65,536 | 16 | 4 | 4× |
| 10⁶ | 20 | 4.3 | 4.6× |
| 10⁹ | 30 | 4.9 | 6.1× |
| 2³² | 32 | 5 | 6.4× |
| 10¹⁸ | 60 | 5.9 | 10× |

```
   As n grows, the advantage becomes more dramatic!
   
   But remember: this only holds for UNIFORM data.
```

---

## When Does Interpolation Search Degrade?

### Worst Case: O(n)

```
   Exponentially spaced data:
   arr = [1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024]
   
   target = 3
   pos = (3-1)/(1024-1) × 10 = 0.019 → pos = 0
   arr[0] = 1 < 3 → lo = 1
   
   pos = (3-2)/(1024-2) × 9 = 0.0088 → pos = 1
   arr[1] = 2 < 3 → lo = 2
   
   pos = (3-4)/(1024-4) × 8 = negative → STOPS
   
   For carefully crafted inputs, each step only
   eliminates one element → O(n) total.
```

### The Adversarial Construction

```
   To force O(n) worst case:
   
   [1, 2, 3, 4, 5, ..., k, 10^9]
   
   target = k
   
   The huge last element makes the formula always guess
   too low, scanning linearly from the left. Each step
   eliminates only 1 element.
```

---

## Improving Worst Case: Interpolation-Binary Hybrid

```
   Idea: Alternate between interpolation and binary search.
   
   - On even iterations: use interpolation
   - On odd iterations: use binary search (guaranteed halving)
   
   Result: O(√n) worst case, O(log log n) average
   
   Better idea (Interpolation-Sequential):
   If interpolation probe is within k elements, do linear scan.
```

### Hybrid Implementation

```python
def hybrid_search(arr, target):
    lo, hi = 0, len(arr) - 1
    use_interpolation = True
    
    while lo <= hi and target >= arr[lo] and target <= arr[hi]:
        if arr[lo] == arr[hi]:
            return lo if arr[lo] == target else -1
        
        if use_interpolation:
            # Interpolation probe
            pos = lo + ((target - arr[lo]) * (hi - lo)) // (arr[hi] - arr[lo])
        else:
            # Binary search probe (guaranteed halving)
            pos = (lo + hi) // 2
        
        use_interpolation = not use_interpolation
        
        if arr[pos] == target:
            return pos
        elif arr[pos] < target:
            lo = pos + 1
        else:
            hi = pos - 1
    
    return -1
```

---

## Distribution Detection

```
   Before choosing interpolation search, check distribution:
   
   Quick heuristic to check uniformity:
   
   def is_roughly_uniform(arr, sample_size=10):
       """Check if array is roughly uniformly distributed."""
       n = len(arr)
       if n < 2:
           return True
       step = max(1, n // sample_size)
       expected_diff = (arr[-1] - arr[0]) / (n - 1)
       
       for i in range(0, n - step, step):
           actual_diff = (arr[i + step] - arr[i]) / step
           ratio = actual_diff / expected_diff if expected_diff else 1
           if ratio > 10 or ratio < 0.1:  # More than 10x off
               return False
       return True
```

---

## Real-World Uniform Data Examples

```
   ✅ Good candidates for interpolation search:
   
   1. Phone numbers in a directory
      → Roughly uniformly distributed
   
   2. Timestamps in log files
      → Events at roughly uniform intervals
   
   3. Auto-increment database IDs
      → Perfectly uniform (1, 2, 3, ...)
   
   4. Hash values
      → Designed to be uniformly distributed
   
   5. Sensor readings at fixed intervals
      → Temperature, pressure (usually uniform-ish)
   
   ❌ Bad candidates:
   
   1. Word lengths in a dictionary
      → Heavily clustered around 5-10
   
   2. Population of cities
      → Power law distribution
   
   3. File sizes
      → Log-normal distribution
   
   4. Social media followers
      → Power law (few have millions)
```

---

## Quadratic Interpolation Search

```
   Extension: Use QUADRATIC interpolation instead of linear.
   
   Linear interpolation assumes data lies on a straight line.
   Quadratic interpolation fits a PARABOLA through three points.
   
   Uses arr[lo], arr[mid], arr[hi] to fit a quadratic:
   
   f(x) = ax² + bx + c where:
   f(lo) = arr[lo], f(mid) = arr[mid], f(hi) = arr[hi]
   
   Then solve f(pos) = target for pos.
   
   Benefit: Better for slightly non-uniform data.
   Cost:    More computation per step.
   Verdict: Rarely worth it in practice.
```

---

## Summary

| Distribution | Expected Time | Practical? |
|-------------|--------------|------------|
| Perfectly uniform | O(log log n) | Excellent |
| Nearly uniform | O(log log n) | Good |
| Mildly skewed | O(log n) | OK (similar to BS) |
| Severely skewed | O(n) | Bad — use BS instead |
| Unknown | Use hybrid | Safe choice |

---

## Quick Revision Questions

1. **Why does uniform distribution give O(log log n)?**
2. **What happens to performance with exponentially spaced data?**
3. **How does n → √n reduction lead to log log n?**
4. **Give 3 examples of real-world uniform data.**
5. **What is the hybrid approach and what does it guarantee?**
6. **For n = 10⁹, how many steps does interpolation need on average?**

---

[← Previous: Concept](01-concept.md) | [Next: Comparison & Practice →](03-comparison-and-practice.md)
