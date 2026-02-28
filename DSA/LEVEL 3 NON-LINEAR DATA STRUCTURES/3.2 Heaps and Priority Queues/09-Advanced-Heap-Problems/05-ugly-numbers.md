# 9.5 Ugly Numbers & Super Ugly Numbers

[← Previous: Employee Free Time](04-employee-free-time.md) | [Next: Problem-Solving Framework →](06-problem-solving-framework.md)

---

## Ugly Numbers

### Problem

**Ugly numbers** are positive integers whose only prime factors are 2, 3, or 5. Find the nth ugly number.

Sequence: 1, 2, 3, 4, 5, 6, 8, 9, 10, 12, 15, ...

### Approach: Min-Heap + Set

```
The next ugly number is always (some previous ugly number) × 2, 3, or 5.

NTH_UGLY_NUMBER(n):
    min_heap = [1]
    seen = {1}
    
    FOR i = 1 TO n:
        ugly = min_heap.extract_min()
        
        IF i == n:
            RETURN ugly
        
        FOR factor in [2, 3, 5]:
            next_ugly = ugly * factor
            IF next_ugly NOT in seen:
                seen.add(next_ugly)
                min_heap.insert(next_ugly)
    
    RETURN ugly
```

### Trace (n=10)

```
Heap: [1], seen={1}

Extract 1: generate 2,3,5
  Heap: [2,3,5], seen={1,2,3,5}

Extract 2: generate 4,6,10
  Heap: [3,4,5,6,10]

Extract 3: generate 6,9,15
  Heap: [4,5,6,9,10,15] (6 already seen)

Extract 4: generate 8,12,20
  Heap: [5,6,8,9,10,12,15,20]

Extract 5: generate 10,15,25
  Heap: [6,8,9,10,12,15,20,25]

Extract 6 → Extract 8 → Extract 9 → Extract 10

10th ugly = 12 ✅
```

---

## Optimized: Three Pointer Approach (No Heap)

```
NTH_UGLY_DP(n):
    ugly = [1] * n
    i2 = i3 = i5 = 0
    
    FOR i = 1 TO n-1:
        next2 = ugly[i2] * 2
        next3 = ugly[i3] * 3
        next5 = ugly[i5] * 5
        
        ugly[i] = min(next2, next3, next5)
        
        IF ugly[i] == next2: i2 += 1
        IF ugly[i] == next3: i3 += 1
        IF ugly[i] == next5: i5 += 1
    
    RETURN ugly[n-1]
```

### Complexity Comparison

| Approach | Time | Space |
|----------|------|-------|
| Heap | O(n log n) | O(n) |
| Three pointers | O(n) | O(n) |

---

## Super Ugly Numbers

### Problem

Like ugly numbers, but with a **custom set of prime factors**.

Given primes = [2, 7, 13, 19], find the nth super ugly number.

### Generalized Multi-Pointer Approach

```
SUPER_UGLY(n, primes):
    ugly = [1] * n
    pointers = [0] * len(primes)   // One pointer per prime
    
    FOR i = 1 TO n-1:
        candidates = [ugly[pointers[j]] * primes[j] for j in range(len(primes))]
        ugly[i] = min(candidates)
        
        FOR j in range(len(primes)):
            IF ugly[pointers[j]] * primes[j] == ugly[i]:
                pointers[j] += 1
    
    RETURN ugly[n-1]
```

### With a Heap (When primes list is large)

```
SUPER_UGLY_HEAP(n, primes):
    min_heap = [(p, p, 0) for p in primes]   // (value, prime, ugly_index)
    ugly = [1] * n
    
    FOR i = 1 TO n-1:
        ugly[i] = min_heap.peek()[0]
        
        WHILE min_heap.peek()[0] == ugly[i]:
            (val, prime, idx) = min_heap.extract_min()
            min_heap.insert((prime * ugly[idx + 1], prime, idx + 1))
    
    RETURN ugly[n-1]
```

### When to Use Which

| Approach | Time | Space | Best When |
|----------|------|-------|-----------|
| Multi-pointer | O(n × k) | O(n + k) | k is small |
| Heap | O(n × log k) | O(n + k) | k is large |

---

## Quick Check

1. **In the heap approach, why do we need a "seen" set?**
   <details><summary>Answer</summary>
   Multiple paths can generate the same ugly number. For example: 2×3 = 6 and 3×2 = 6. The seen set prevents duplicate entries in the heap, which would produce incorrect results.
   </details>

2. **What's the key difference between heap and three-pointer approaches?**
   <details><summary>Answer</summary>
   The heap approach generates candidates eagerly (pushing many future values), using O(n log n) time. The three-pointer approach generates exactly one candidate per prime per step and picks the minimum, achieving O(n) time. The pointer approach is more efficient but harder to generalize.
   </details>

---

[← Previous: Employee Free Time](04-employee-free-time.md) | [Next: Problem-Solving Framework →](06-problem-solving-framework.md)
