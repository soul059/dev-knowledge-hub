# Chapter 3: Hash Set Techniques

## 📋 Chapter Overview
Hash set for O(1) existence checks: longest consecutive sequence, intersection, happy number, and duplicate detection.

---

## 🧠 Hash Set vs Hash Map

```
  Hash Set: stores KEYS only        Hash Map: stores KEY-VALUE pairs
  
  set = {1, 5, 3, 9}                map = {1:"a", 5:"b", 3:"c"}
  
  Operation: "Is 5 present?" → O(1)  Operation: "Value for key 5?" → O(1)
  
  Use Set when: only need existence/membership
  Use Map when: need associated values
```

---

## 📝 Problem 1: Longest Consecutive Sequence (LeetCode 128)

**Problem:** Find length of the longest consecutive element sequence. Must be O(n).

```
function longestConsecutive(nums):
    numSet = set(nums)
    maxLen = 0
    
    for each num in numSet:
        // Only start counting from sequence START
        if (num - 1) not in numSet:
            length = 1
            current = num
            while (current + 1) in numSet:
                current++
                length++
            maxLen = max(maxLen, length)
    
    return maxLen
```

### Trace

```
  nums = [100, 4, 200, 1, 3, 2]
  numSet = {100, 4, 200, 1, 3, 2}
  
  100: 99 not in set → start. 101 not in set → length=1
  4:   3 in set → SKIP (not a sequence start)
  200: 199 not in set → start. 201 not in set → length=1
  1:   0 not in set → start. 2,3,4 in set → length=4
  3:   2 in set → SKIP
  2:   1 in set → SKIP
  
  Answer: 4 (sequence: 1,2,3,4)
```

**Key insight:** Only start counting from elements where `num-1` is NOT in the set — guarantees each sequence is counted once. O(n) total.

---

## 📝 Problem 2: Intersection of Two Arrays (LeetCode 349/350)

### Unique Intersection (LeetCode 349)

```
function intersection(nums1, nums2):
    set1 = set(nums1)
    result = set()
    
    for each num in nums2:
        if num in set1:
            result.add(num)
    
    return result
```

### Intersection with Duplicates (LeetCode 350)

```
function intersect(nums1, nums2):
    count = frequency map of nums1
    result = []
    
    for each num in nums2:
        if count[num] > 0:
            result.add(num)
            count[num]--
    
    return result
```

---

## 📝 Problem 3: Happy Number (LeetCode 202)

**Problem:** Starting from n, replace with sum of squares of digits. If reaches 1 → happy. If loops forever → not happy.

```
function isHappy(n):
    seen = set()
    
    while n ≠ 1:
        if n in seen: return false   // cycle detected
        seen.add(n)
        n = sumOfSquaresOfDigits(n)
    
    return true

function sumOfSquaresOfDigits(n):
    sum = 0
    while n > 0:
        d = n % 10
        sum += d * d
        n = n / 10
    return sum
```

### Trace

```
  n = 19
  19 → 1² + 9² = 82
  82 → 64 + 4 = 68
  68 → 36 + 64 = 100
  100 → 1 + 0 + 0 = 1 → Happy! ✓
  
  n = 2
  2 → 4 → 16 → 37 → 58 → 89 → 145 → 42 → 20 → 4 (cycle!)
  → Not happy
```

**Alternative:** Floyd's cycle detection (fast/slow pointers) — O(1) space.

---

## 📝 Problem 4: Valid Sudoku (LeetCode 36)

```
function isValidSudoku(board):
    rows = array of 9 empty sets
    cols = array of 9 empty sets
    boxes = array of 9 empty sets
    
    for i = 0 to 8:
        for j = 0 to 8:
            if board[i][j] == '.': continue
            num = board[i][j]
            boxIdx = (i / 3) * 3 + (j / 3)
            
            if num in rows[i] or num in cols[j] or num in boxes[boxIdx]:
                return false
            
            rows[i].add(num)
            cols[j].add(num)
            boxes[boxIdx].add(num)
    
    return true
```

### Box Index Formula

```
  Box layout (3×3 grid of boxes):
  ┌───┬───┬───┐
  │ 0 │ 1 │ 2 │    boxIdx = (row/3)*3 + (col/3)
  ├───┼───┼───┤
  │ 3 │ 4 │ 5 │    Example: row=4, col=7
  ├───┼───┼───┤    boxIdx = (4/3)*3 + (7/3) = 1*3 + 2 = 5
  │ 6 │ 7 │ 8 │
  └───┴───┴───┘
```

---

## 📝 Problem 5: Contains Duplicate (LeetCode 217)

```
function containsDuplicate(nums):
    seen = set()
    for each num in nums:
        if num in seen: return true
        seen.add(num)
    return false
```

**O(n)** time, O(n) space. Simplest hash set application.

---

## 📊 Hash Set Use Cases

| Use Case | Technique | Time |
|----------|-----------|------|
| Duplicate detection | Add to set; check before adding | O(n) |
| Cycle detection | Store visited states | O(n) |
| Membership testing | Convert to set; O(1) lookups | O(n) |
| Set operations | Union, intersection, difference | O(n) |
| Sequence counting | Skip non-starts using num-1 check | O(n) |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Hash set | O(1) membership check, no values |
| Longest consecutive | Start from sequence beginning (num-1 absent), count forward |
| Cycle detection | Store seen states; detect revisits |
| Sudoku validation | Three sets per dimension (row, col, box) |
| Set operations | Intersection, union — convert one list to set |

---

## ❓ Revision Questions

1. **Longest consecutive: why O(n)?** → Each element is visited at most twice (once in loop, once in while counting). Only sequence starts trigger counting.
2. **Happy number: guaranteed to terminate?** → Yes — the sequence either reaches 1 or enters a cycle (finite possible values for bounded digit sums).
3. **Hash set vs sorted array for membership?** → Set: O(1) average lookup. Sorted array: O(log n) via binary search. Set is faster if memory allows.
4. **Sudoku box index formula?** → `(row / 3) * 3 + (col / 3)` maps each cell to its 3×3 box (0-8).
5. **When to use set vs map?** → Set when only checking existence. Map when you need to associate a value (frequency, index, etc.).

---

[← Previous: Frequency Counting](02-frequency-counting.md) | [Next: Classic Problems →](04-classic-problems.md)
