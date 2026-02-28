# Chapter 3: Permutation Problems

## 📋 Chapter Overview
Permutations = all orderings. Key variations: with/without duplicates, next permutation, and string permutations.

---

## 🔍 Problem 1: Permutations (No Duplicates)

```
  Input: [1, 2, 3]
  Output: [[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
  Count: 3! = 6
```

### Approach: Used Set

```
function permute(nums):
    result = []
    backtrack(nums, [], result, new boolean[len(nums)])
    return result

function backtrack(nums, path, result, used):
    if len(path) == len(nums):
        result.add(copy(path))
        return
    
    for i = 0 to len(nums)-1:
        if used[i]: continue
        used[i] = true
        path.add(nums[i])
        backtrack(nums, path, result, used)
        path.removeLast()
        used[i] = false
```

### Approach: Swap (In-Place)

```
function permuteSwap(nums, start, result):
    if start == len(nums):
        result.add(copy(nums))
        return
    
    for i = start to len(nums)-1:
        swap(nums, start, i)
        permuteSwap(nums, start + 1, result)
        swap(nums, start, i)          // un-swap
```

```
  [1, 2, 3]
  
  start=0: swap(0,0)=[1,2,3], swap(0,1)=[2,1,3], swap(0,2)=[3,2,1]
  
  For [1,2,3], start=1:
    swap(1,1)=[1,2,3], swap(1,2)=[1,3,2]
  
  For [2,1,3], start=1:
    swap(1,1)=[2,1,3], swap(1,2)=[2,3,1]
  ...
```

---

## 🔍 Problem 2: Permutations II (With Duplicates)

```
  Input: [1, 1, 2]
  Output: [[1,1,2], [1,2,1], [2,1,1]]
  NOT: [1,1,2] appearing twice
```

```
function permuteUnique(nums):
    sort(nums)                          // sort first
    result = []
    backtrack(nums, [], result, new boolean[len(nums)])
    return result

function backtrack(nums, path, result, used):
    if len(path) == len(nums):
        result.add(copy(path))
        return
    
    for i = 0 to len(nums)-1:
        if used[i]: continue
        // Skip duplicate: same value, previous not used (at this level)
        if i > 0 AND nums[i] == nums[i-1] AND !used[i-1]:
            continue
        used[i] = true
        path.add(nums[i])
        backtrack(nums, path, result, used)
        path.removeLast()
        used[i] = false
```

### Why `!used[i-1]`?

```
  nums = [1a, 1b, 2]  (subscripts for clarity)
  
  We allow: used[i-1] = true  → 1a is at an earlier position in path
  We skip:  used[i-1] = false → 1a was skipped at this level
  
  This ensures only ONE ordering of identical elements:
  1a before 1b (allowed), 1b before 1a (skipped)
```

---

## 🔍 Problem 3: Next Permutation

**Find the next lexicographically larger permutation.**

```
  1 2 3 → 1 3 2
  3 2 1 → 1 2 3 (wrap around)
  1 1 5 → 1 5 1
  1 3 5 4 2 → 1 4 2 3 5
```

### Algorithm

```
function nextPermutation(nums):
    n = len(nums)
    
    // Step 1: Find rightmost ascending pair
    i = n - 2
    while i >= 0 AND nums[i] >= nums[i+1]:
        i -= 1
    
    if i >= 0:
        // Step 2: Find rightmost element larger than nums[i]
        j = n - 1
        while nums[j] <= nums[i]:
            j -= 1
        // Step 3: Swap
        swap(nums, i, j)
    
    // Step 4: Reverse suffix after position i
    reverse(nums, i + 1, n - 1)
```

### Trace: [1, 3, 5, 4, 2]

```
  Step 1: Find i where nums[i] < nums[i+1] from right
    i=3: 4 >= 2 → continue
    i=2: 5 >= 4 → continue
    i=1: 3 < 5  → FOUND, i=1

  Step 2: Find j where nums[j] > nums[i]=3 from right
    j=4: 2 ≤ 3 → continue
    j=3: 4 > 3 → FOUND, j=3

  Step 3: Swap nums[1] and nums[3]
    [1, 4, 5, 3, 2]

  Step 4: Reverse suffix from i+1=2 to end
    [1, 4, 2, 3, 5]  ✓
```

---

## 🔍 Problem 4: Palindrome Permutation

**Can the letters form a palindrome? If yes, generate all palindrome permutations.**

### Check Feasibility

```
function canPermutePalindrome(s):
    freq = count frequency of each char
    oddCount = count chars with odd frequency
    return oddCount <= 1
```

### Generate Palindrome Permutations

```
function generatePalindromes(s):
    freq = count frequencies
    oddCount = count chars with odd freq
    if oddCount > 1: return []
    
    // Build half string
    half = []
    middle = ""
    for char, count in freq:
        if count % 2 == 1: middle = char
        for i = 0 to count/2 - 1:
            half.add(char)
    
    // Generate permutations of half (with duplicates)
    result = []
    permuteUnique(half, 0, result)
    
    // Build full palindrome: perm + middle + reverse(perm)
    return [p + middle + reverse(p) for p in result]
```

---

## 📊 Permutation vs Combination vs Subset

```
  {1, 2, 3}

  Subsets:       [], [1], [2], [3], [1,2], [1,3], [2,3], [1,2,3]
                 Order doesn't matter, any size → 2^n

  Combinations:  [1,2], [1,3], [2,3]  (k=2)
                 Order doesn't matter, fixed size → C(n,k)

  Permutations:  [1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], [3,2,1]
                 Order MATTERS, all elements → n!
```

| | Order Matters? | Size | Count | start param |
|--|---|---|---|---|
| Subsets | No | Any | 2^n | i+1 |
| Combinations | No | k | C(n,k) | i+1 |
| Permutations | Yes | n | n! | 0 + used[] |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Permutations | Loop from 0 with `used[]` set |
| With duplicates | Sort + skip `nums[i]==nums[i-1] && !used[i-1]` |
| Swap method | In-place, no extra space for used |
| Next permutation | Find ascending pair from right, swap, reverse |
| Palindrome perm | Generate half-permutations, mirror |

---

## ❓ Revision Questions

1. **Permutations vs Subsets: key structural difference?** → Permutations loop from index 0 with a used set; Subsets loop from `start` with `start = i+1`.
2. **Why `!used[i-1]` for duplicate permutations?** → Forces identical elements to appear in their original relative order, avoiding duplicates.
3. **Next permutation time complexity?** → O(n) — single pass to find i, single pass to find j, O(n) reverse.
4. **When can a string form a palindrome?** → At most one character has an odd frequency.
5. **n! grows how fast?** → Extremely: 10! = 3.6M, 12! = 479M, 15! = 1.3T. Permutation problems need n ≤ ~10-12.

---

[← Previous: Subsets and Combinations](02-subsets-and-combinations.md) | [Next: Constraint Satisfaction →](04-constraint-satisfaction.md)
