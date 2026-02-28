# Chapter 1: Backtracking Fundamentals

## 📋 Chapter Overview
Backtracking systematically explores **all possible solutions** by building candidates incrementally and **abandoning** (backtracking) as soon as a candidate is invalid. Think of it as DFS on a **decision tree**.

---

## 🧠 Core Concept

### The Decision Tree

```
  Problem: Generate all subsets of {1, 2, 3}

  At each element, decide: INCLUDE or EXCLUDE

                    []
                 /      \
          [1]              []
         /   \            /  \
     [1,2]   [1]      [2]    []
     /  \    / \      / \    / \
  [1,2,3][1,2][1,3][1][2,3][2][3][]

  Leaves = all 2³ = 8 subsets
```

### Backtracking = DFS + Undo

```
  1. CHOOSE:    Make a choice (add element)
  2. EXPLORE:   Recurse with the choice
  3. UNCHOOSE:  Undo the choice (backtrack)

  path = []
  path.add(1)        // choose
  explore(...)       // recurse
  path.remove(1)     // un-choose (backtrack)
```

---

## 📝 Universal Template

```
function backtrack(candidates, start, path, result):
    if isComplete(path):             // base case
        result.add(copy(path))
        return
    
    for i = start to len(candidates)-1:
        if isValid(candidates[i]):   // pruning
            path.add(candidates[i])           // choose
            backtrack(candidates, i+1, path, result)  // explore
            path.removeLast()                 // un-choose
```

### Template Parts

| Part | Purpose | Example |
|------|---------|---------|
| `isComplete` | Stop condition | Path length == n |
| `isValid` | Prune invalid branches | No duplicate, sum ≤ target |
| `choose` | Add to current path | path.add(x) |
| `explore` | Recurse deeper | backtrack(i+1, ...) |
| `un-choose` | Restore state | path.removeLast() |
| `start` | Avoid revisiting | i+1 for subsets, i for reuse |

---

## 🔍 Example: Generate All Subsets

```
function subsets(nums):
    result = []
    backtrack(nums, 0, [], result)
    return result

function backtrack(nums, start, path, result):
    result.add(copy(path))          // every path is a valid subset
    
    for i = start to len(nums)-1:
        path.add(nums[i])          // choose
        backtrack(nums, i+1, path, result)  // explore
        path.removeLast()          // un-choose
```

### Trace: nums = [1, 2, 3]

```
  backtrack(start=0, path=[])
  │  result: [[]]
  │
  ├── i=0: path=[1]
  │   │  result: [[], [1]]
  │   ├── i=1: path=[1,2]
  │   │   │  result: [[], [1], [1,2]]
  │   │   └── i=2: path=[1,2,3]
  │   │       result: [[], [1], [1,2], [1,2,3]]
  │   │       backtrack → path=[1,2]
  │   │   backtrack → path=[1]
  │   └── i=2: path=[1,3]
  │       result: [[], [1], [1,2], [1,2,3], [1,3]]
  │       backtrack → path=[1]
  │   backtrack → path=[]
  │
  ├── i=1: path=[2]
  │   result: [..., [2]]
  │   └── i=2: path=[2,3]
  │       result: [..., [2,3]]
  │       backtrack → path=[2]
  │   backtrack → path=[]
  │
  └── i=2: path=[3]
      result: [..., [3]]
      backtrack → path=[]

  Final: [[], [1], [1,2], [1,2,3], [1,3], [2], [2,3], [3]]
```

---

## 🔍 Example: Permutations

```
function permutations(nums):
    result = []
    backtrack(nums, [], result, new Set())
    return result

function backtrack(nums, path, result, used):
    if len(path) == len(nums):
        result.add(copy(path))
        return
    
    for i = 0 to len(nums)-1:
        if i in used: continue         // skip used elements
        used.add(i)
        path.add(nums[i])
        backtrack(nums, path, result, used)
        path.removeLast()
        used.remove(i)
```

### Decision Tree for [1,2,3]

```
              []
          /   |   \
       [1]   [2]   [3]
       / \   / \   / \
    [1,2][1,3] [2,1][2,3] [3,1][3,2]
      |    |     |    |     |    |
  [1,2,3][1,3,2][2,1,3][2,3,1][3,1,2][3,2,1]
  
  6 permutations = 3! ✓
```

---

## ⚠️ Common Mistakes

### 1. Forgetting to Copy Path

```
  result.add(path)          ❌  Adds reference — all entries point to same list
  result.add(copy(path))    ✅  Adds snapshot
```

### 2. Forgetting to Backtrack

```
  path.add(x)
  backtrack(...)
  // forgot path.removeLast()  ❌  Path keeps growing!
```

### 3. Wrong Start Index

```
  Subsets: start = i + 1      // don't revisit earlier elements
  Permutations: start = 0     // any element can go anywhere (use 'used' set)
  Combinations with reuse: start = i  // can reuse same element
```

---

## 📊 Complexity

| Problem | Time | Space |
|---------|------|-------|
| Subsets | O(2^n) | O(n) recursion depth |
| Permutations | O(n!) | O(n) recursion depth |
| Combinations(n,k) | O(C(n,k)) | O(k) recursion depth |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Pattern | Choose → Explore → Un-choose |
| Data structure | Recursion stack (implicit DFS) |
| When to use | Need ALL solutions or ANY valid solution |
| Base case | Path is complete / target reached |
| Pruning | Skip invalid choices early |
| Copy path | Always copy before adding to result |
| Start index | Controls subset vs permutation behavior |

---

## ❓ Revision Questions

1. **What are the 3 steps of backtracking?** → Choose (add), Explore (recurse), Un-choose (remove).
2. **Why copy the path before adding to result?** → The path list is reused; without copying, all result entries reference the same (eventually empty) list.
3. **Subsets vs Permutations: key code difference?** → Subsets use `start = i+1`; Permutations loop from 0 with a `used` set.
4. **Time complexity for generating all subsets?** → O(2^n) — each element has 2 choices (include/exclude).
5. **What is pruning?** → Skipping branches of the decision tree that can't lead to valid solutions, reducing work.

---

[← Back to README](../README.md) | [Next: Subsets and Combinations →](02-subsets-and-combinations.md)
