# Chapter 2: Subsets and Combinations

## 📋 Chapter Overview
Subsets and combinations are the bread-and-butter of backtracking. Master the variations: with/without duplicates, exact size, and target sum.

---

## 🔍 Problem 1: Subsets (No Duplicates)

```
  Input: [1, 2, 3]
  Output: [[], [1], [2], [3], [1,2], [1,3], [2,3], [1,2,3]]
```

```
function subsets(nums):
    result = []
    backtrack(nums, 0, [], result)
    return result

function backtrack(nums, start, path, result):
    result.add(copy(path))
    for i = start to len(nums)-1:
        path.add(nums[i])
        backtrack(nums, i + 1, path, result)
        path.removeLast()
```

---

## 🔍 Problem 2: Subsets II (With Duplicates)

```
  Input: [1, 2, 2]
  Output: [[], [1], [1,2], [1,2,2], [2], [2,2]]
  NOT: [2] appearing twice
```

### Key: Sort + Skip Adjacent Duplicates

```
function subsetsWithDup(nums):
    sort(nums)                          // sort first!
    result = []
    backtrack(nums, 0, [], result)
    return result

function backtrack(nums, start, path, result):
    result.add(copy(path))
    for i = start to len(nums)-1:
        if i > start AND nums[i] == nums[i-1]:
            continue                    // skip duplicate at same level
        path.add(nums[i])
        backtrack(nums, i + 1, path, result)
        path.removeLast()
```

### Why `i > start` (not `i > 0`)?

```
  nums = [1, 2, 2]

  At start=0: i=1 (2), i=2 (2) ← skip i=2 because nums[2]==nums[1] and 2>start(0)
  At start=1: i=1 (2), i=2 (2) ← DON'T skip i=2 because we need [2,2]
                                   But wait: 2 > start(1)? Yes → skip

  Actually, let me re-check:
  At start=1: i=1, path=[1,2] → recurse with start=2, i=2, path=[1,2,2] ✓
  At start=0: i=1 first 2, i=2 second 2 → skip because i(2)>start(0) and nums[2]==nums[1]

  Result: we get [2] once (from i=1) but NOT again (i=2 skipped) ✓
  We get [2,2] once (from i=1→i=2 inside recursion) ✓
```

---

## 🔍 Problem 3: Combinations (Choose k from n)

```
  Input: n=4, k=2
  Output: [[1,2], [1,3], [1,4], [2,3], [2,4], [3,4]]
```

```
function combine(n, k):
    result = []
    backtrack(n, k, 1, [], result)
    return result

function backtrack(n, k, start, path, result):
    if len(path) == k:
        result.add(copy(path))
        return
    
    for i = start to n:
        path.add(i)
        backtrack(n, k, i + 1, path, result)
        path.removeLast()
```

### Pruning Optimization

```
  If remaining candidates < spots to fill, prune early.
  
  Need: k - len(path) more elements
  Available: n - i + 1 elements
  Prune when: n - i + 1 < k - len(path)
  → Loop until: i <= n - (k - len(path)) + 1
```

```
function backtrack(n, k, start, path, result):
    if len(path) == k:
        result.add(copy(path))
        return
    
    remaining = k - len(path)
    for i = start to n - remaining + 1:    // pruned upper bound
        path.add(i)
        backtrack(n, k, i + 1, path, result)
        path.removeLast()
```

---

## 🔍 Problem 4: Combination Sum (Reuse Allowed)

```
  Input: candidates=[2,3,6,7], target=7
  Output: [[2,2,3], [7]]
```

```
function combinationSum(candidates, target):
    result = []
    backtrack(candidates, target, 0, [], result)
    return result

function backtrack(candidates, target, start, path, result):
    if target == 0:
        result.add(copy(path))
        return
    if target < 0: return
    
    for i = start to len(candidates)-1:
        path.add(candidates[i])
        backtrack(candidates, target - candidates[i], i, path, result)  // i, not i+1 (reuse)
        path.removeLast()
```

### Trace: candidates=[2,3,6,7], target=7

```
  start=0, target=7, path=[]
  │
  ├── i=0: add 2, target=5, path=[2]
  │   ├── i=0: add 2, target=3, path=[2,2]
  │   │   ├── i=0: add 2, target=1, path=[2,2,2]
  │   │   │   └── i=0: add 2, target=-1 → PRUNE
  │   │   └── i=1: add 3, target=0, path=[2,2,3] → FOUND ✓
  │   ├── i=1: add 3, target=2, path=[2,3]
  │   │   └── i=1: add 3, target=-1 → PRUNE
  │   └── ...
  │
  └── i=3: add 7, target=0, path=[7] → FOUND ✓
  
  Result: [[2,2,3], [7]]
```

---

## 🔍 Problem 5: Combination Sum II (No Reuse, With Duplicates)

```
  Input: candidates=[10,1,2,7,6,1,5], target=8
  Output: [[1,1,6], [1,2,5], [1,7], [2,6]]
```

```
function combinationSum2(candidates, target):
    sort(candidates)                    // sort for duplicate handling
    result = []
    backtrack(candidates, target, 0, [], result)
    return result

function backtrack(candidates, target, start, path, result):
    if target == 0:
        result.add(copy(path))
        return
    
    for i = start to len(candidates)-1:
        if candidates[i] > target: break           // prune (sorted)
        if i > start AND candidates[i] == candidates[i-1]:
            continue                               // skip duplicates
        
        path.add(candidates[i])
        backtrack(candidates, target - candidates[i], i + 1, path, result)
        path.removeLast()
```

---

## 📊 Variation Comparison

| Problem | start next | Duplicates | Sort? | Key Difference |
|---------|-----------|------------|-------|----------------|
| Subsets | i + 1 | No | No | Every path is result |
| Subsets II | i + 1 | Yes | Yes | Skip `nums[i]==nums[i-1]` |
| Combinations | i + 1 | No | No | Stop at len == k |
| CombSum (reuse) | i | No | Optional | start = i (reuse) |
| CombSum II | i + 1 | Yes | Yes | Sort + skip dupes |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Subsets | No base case filter — add every path |
| Duplicates | Sort + `if i > start && nums[i] == nums[i-1]: skip` |
| Reuse allowed | `start = i` instead of `i + 1` |
| Pruning | `break` when candidate > remaining target |
| Combinations | Add pruned upper bound for loop |

---

## ❓ Revision Questions

1. **How to handle duplicates in subsets/combinations?** → Sort the array, skip when `nums[i] == nums[i-1]` and `i > start`.
2. **Reuse allowed: what changes?** → Recurse with `start = i` instead of `i + 1`.
3. **Subsets vs Combinations: structural difference?** → Subsets add every path to result; Combinations only add paths of size k.
4. **Why sort for Combination Sum II?** → To group duplicates together and enable the skip condition.
5. **How to prune Combination Sum?** → If `candidates[i] > remaining target`, break (since array is sorted).

---

[← Previous: Backtracking Fundamentals](01-backtracking-fundamentals.md) | [Next: Permutation Problems →](03-permutation-problems.md)
