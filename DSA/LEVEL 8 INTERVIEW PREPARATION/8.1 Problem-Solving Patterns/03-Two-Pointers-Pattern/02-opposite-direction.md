# Chapter 2: Opposite Direction (Two Pointers)

## 📋 Chapter Overview
Two pointers starting at **opposite ends** and moving toward each other. The most classic two-pointer variant — used for pair finding, palindromes, and container problems.

---

## 🎯 Core Concept

```
  OPPOSITE DIRECTION TWO POINTERS
  
  Array:  [ 1   2   3   4   5   6   7   8 ]
           L──►                       ◄──R
  
  L starts at index 0 (left end)
  R starts at index n-1 (right end)
  They move toward each other based on a comparison.
  
  ┌──────────────────────────────────────────────┐
  │  while L < R:                                │
  │      compute something with arr[L], arr[R]   │
  │      if condition: move L right (L++)        │
  │      else: move R left (R--)                 │
  └──────────────────────────────────────────────┘
```

---

## 📐 The Template

```
PSEUDOCODE — Opposite Direction Two Pointers:

function oppositePointers(arr):
    left = 0
    right = len(arr) - 1
    result = initial_value
    
    while left < right:
        current = compute(arr[left], arr[right])
        
        if current matches target:
            recordResult()
            left += 1        // or right -= 1, or both
        elif current < target:
            left += 1         // need larger value
        else:
            right -= 1        // need smaller value
    
    return result
```

---

## 🧪 Trace 1: Two Sum II (Sorted Array)

**Problem:** Find two numbers in sorted array that add up to target.

**Input:** `arr = [2, 7, 11, 15]`, `target = 9`

```
  ┌──────┬──────┬──────────┬─────────┬───────────┐
  │ left │ right│ sum      │ compare │ action    │
  ├──────┼──────┼──────────┼─────────┼───────────┤
  │  0   │  3   │ 2+15=17  │ 17 > 9  │ right--   │
  │  0   │  2   │ 2+11=13  │ 13 > 9  │ right--   │
  │  0   │  1   │ 2+7=9    │ 9 == 9  │ FOUND!    │
  └──────┴──────┴──────────┴─────────┴───────────┘
  
  Answer: indices [0, 1] → values [2, 7]
```

### Why This Works on Sorted Arrays:

```
  Sorted: [ 2   7   11   15 ]
             L              R
  
  sum = 2 + 15 = 17 > 9
  
  We know: 2 + 15 > target
  So:  7 + 15 > target     (7 > 2, so even bigger)
       11 + 15 > target    (11 > 2, bigger still)
  
  → ELIMINATE entire column with R = 15
  → Move R left!
  
  This eliminates possibilities in O(1), giving O(n) total.
```

---

## 🧪 Trace 2: Container With Most Water

**Problem:** Find two lines forming a container that holds the most water.

**Input:** `heights = [1, 8, 6, 2, 5, 4, 8, 3, 7]`

```
  Water = min(h[L], h[R]) × (R - L)
  
  ┌──────┬──────┬────────────────────┬───────┬─────────────┐
  │  L   │  R   │ water              │  max  │ action      │
  ├──────┼──────┼────────────────────┼───────┼─────────────┤
  │  0   │  8   │ min(1,7)×8 = 8     │  8    │ h[L]<h[R] L++│
  │  1   │  8   │ min(8,7)×7 = 49    │  49   │ h[L]>h[R] R--│
  │  1   │  7   │ min(8,3)×6 = 18    │  49   │ h[L]>h[R] R--│
  │  1   │  6   │ min(8,8)×5 = 40    │  49   │ h[L]=h[R] L++│
  │  2   │  6   │ min(6,8)×4 = 24    │  49   │ h[L]<h[R] L++│
  │  3   │  6   │ min(2,8)×3 = 6     │  49   │ h[L]<h[R] L++│
  │  4   │  6   │ min(5,8)×2 = 10    │  49   │ h[L]<h[R] L++│
  │  5   │  6   │ min(4,8)×1 = 4     │  49   │ L meets R    │
  └──────┴──────┴────────────────────┴───────┴─────────────┘
  
  Answer: 49
```

### Why Move the Shorter Line:

```
  ┌──────────────────────────────────────────────┐
  │  Water = min(h[L], h[R]) × width            │
  │                                              │
  │  If h[L] < h[R]:                             │
  │    Moving R left → width decreases           │
  │    min(h[L], h[R']) ≤ h[L]  (can't increase) │
  │    Both factors ≤ → water can only decrease   │
  │    → NO POINT moving R!                      │
  │                                              │
  │  So: Move the SHORTER side to potentially    │
  │  find a taller line that increases water.    │
  └──────────────────────────────────────────────┘
```

---

## 🧪 Trace 3: Valid Palindrome

**Problem:** Check if a string is a palindrome (ignoring non-alphanumeric and case).

**Input:** `"A man, a plan, a canal: Panama"`

```
  Clean: "amanaplanacanalpanama"
  
  ┌──────┬──────┬──────┬──────┬─────────┐
  │  L   │  R   │ s[L] │ s[R] │ match?  │
  ├──────┼──────┼──────┼──────┼─────────┤
  │  0   │  20  │  'a' │  'a' │  YES    │
  │  1   │  19  │  'm' │  'm' │  YES    │
  │  2   │  18  │  'a' │  'a' │  YES    │
  │  ...  │ ...  │ ...  │ ...  │  YES    │
  │  10  │  10  │  'c' │  'c' │ L≥R END │
  └──────┴──────┴──────┴──────┴─────────┘
  
  Answer: true (is palindrome)
```

---

## 🧪 Trace 4: Three Sum

**Problem:** Find all unique triplets that sum to zero.

**Input:** `[-1, 0, 1, 2, -1, -4]`

```
  Step 1: Sort → [-4, -1, -1, 0, 1, 2]
  
  Step 2: Fix one element, use two pointers for remaining
  
  i=0: fix -4, find pair summing to 4 in [-1,-1,0,1,2]
    L=1, R=5: -1+2=1  < 4  → L++
    L=2, R=5: -1+2=1  < 4  → L++
    L=3, R=5: 0+2=2   < 4  → L++
    L=4, R=5: 1+2=3   < 4  → L++
    L=5, R=5: done
  
  i=1: fix -1, find pair summing to 1 in [-1,0,1,2]
    L=2, R=5: -1+2=1  == 1 → FOUND! [-1,-1,2] ✓
    L=3, R=4: 0+1=1   == 1 → FOUND! [-1,0,1] ✓
    L=4, R=4: done
  
  i=2: fix -1, SKIP (same as i=1, duplicate!)
  
  i=3: fix 0, find pair summing to 0 in [1,2]
    L=4, R=5: 1+2=3   > 0  → R--
    L=4, R=4: done
  
  Answer: [[-1,-1,2], [-1,0,1]]
```

### Pseudocode:

```
function threeSum(arr):
    sort(arr)
    result = []
    
    for i = 0 to n - 3:
        if i > 0 AND arr[i] == arr[i-1]: continue   // skip duplicates
        
        target = -arr[i]
        left = i + 1
        right = n - 1
        
        while left < right:
            sum = arr[left] + arr[right]
            if sum == target:
                result.add([arr[i], arr[left], arr[right]])
                // Skip duplicates
                while left < right AND arr[left] == arr[left+1]: left++
                while left < right AND arr[right] == arr[right-1]: right--
                left += 1
                right -= 1
            elif sum < target:
                left += 1
            else:
                right -= 1
    
    return result
```

**Time:** O(n²)  **Space:** O(1) excluding output

---

## 📊 Opposite Direction Patterns

| Problem | Move L when | Move R when |
|---------|-------------|-------------|
| Two Sum (sorted) | sum < target | sum > target |
| Container Water | h[L] < h[R] | h[L] ≥ h[R] |
| Palindrome | s[L] == s[R] (both) | s[L] == s[R] (both) |
| Three Sum | sum < target | sum > target |
| Trapping Rain Water | h[L] < h[R] | h[L] ≥ h[R] |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Structure | L at start, R at end, move toward each other |
| Sorted requirement | Most problems need sorted array (or inherent order) |
| Move logic | Move the pointer that can improve the result |
| Three Sum | Fix one, two-pointer on rest → O(n²) |
| Container Water | Move the shorter line (taller can't help) |
| Time | O(n) for pair, O(n²) for triplet |

---

## ❓ Quick Revision Questions

1. **Why must the array be sorted for Two Sum II?**
   > Because the movement logic (move L for larger, move R for smaller) only works when elements are ordered.

2. **In container with most water, why move the shorter side?**
   > Moving the taller side can only decrease or maintain water (width shrinks, min stays same). Moving shorter side has a chance of finding a taller line.

3. **How do you handle duplicates in Three Sum?**
   > Skip duplicate values of the fixed element (`arr[i] == arr[i-1]`) and duplicate pairs (`arr[left] == arr[left+1]`, `arr[right] == arr[right-1]`).

4. **What is the time complexity of Three Sum?**
   > O(n²) — O(n log n) sort + O(n) × O(n) two-pointer for each fixed element.

5. **When do both pointers move inward simultaneously?**
   > In palindrome checking — when characters match, both L and R move inward.

6. **Can opposite direction work on unsorted arrays?**
   > Rarely. Container with most water works on unsorted (uses indices as positions), but most problems require sorting first.

---

[← Previous: Same Direction](01-same-direction.md) | [Next: Fast and Slow →](03-fast-and-slow.md)

[← Back to README](../README.md)
