# Chapter 6.5: Common Bugs to Avoid

```
╔══════════════════════════════════════════════════════════╗
║          COMMON BUGS TO AVOID                            ║
║   The definitive catalog of interview coding bugs        ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Certain bugs appear in interview after interview. Knowing them in advance lets you **prevent them proactively** rather than debugging reactively. This chapter catalogues the most common bugs with examples and fixes.

---

## Bug #1: Off-By-One Errors

```
THE MOST COMMON BUG IN ALL OF PROGRAMMING

EXAMPLE 1: Loop bound
  ✗ for i in range(len(arr) + 1):  ← IndexError
  ✓ for i in range(len(arr)):

EXAMPLE 2: Substring length
  ✗ length = right - left          ← Off by 1
  ✓ length = right - left + 1      (inclusive both ends)

EXAMPLE 3: Binary search
  ✗ while left < right:            ← Misses single element
  ✓ while left <= right:           (standard binary search)

EXAMPLE 4: Two pointer convergence
  ✗ while left < right - 1:        ← Skips adjacent pair
  ✓ while left < right:

FIX STRATEGY:
  Test with arrays of size 0, 1, and 2.
  If any fails, it's likely off-by-one.
```

---

## Bug #2: Wrong Initialization

```
EXAMPLE 1: Maximum value
  ✗ max_val = 0                    ← Fails for [-3, -1, -5]
  ✓ max_val = float('-inf')        ← Works for all arrays
  ✓ max_val = nums[0]              ← Also works (check empty)

EXAMPLE 2: Minimum value
  ✗ min_val = 0                    ← Fails for [5, 10, 15]
  ✓ min_val = float('inf')

EXAMPLE 3: Count starting value
  ✗ count = 1                      ← Adds 1 to actual count
  ✓ count = 0                      ← Start from zero

EXAMPLE 4: Result container
  ✗ result = None
  ✓ result = []                    ← Ready to append

FIX STRATEGY:
  Ask: "What happens if the first element is negative?"
  Ask: "What happens if there are zero matching items?"
```

---

## Bug #3: Infinite Loops

```
EXAMPLE 1: While loop without progress
  ✗ while left <= right:
      mid = (left + right) // 2
      if nums[mid] < target:
          pass                      ← left never moves!
  ✓     left = mid + 1             ← Progress guaranteed

EXAMPLE 2: Linked list cycle not detected
  ✗ while node:
      node = node.next             ← Infinite if cycle exists
  ✓ visited = set()
    while node and node not in visited:
        visited.add(node)
        node = node.next

EXAMPLE 3: Recursion without base case
  ✗ def solve(n):
      return solve(n - 1) + 1     ← Never stops
  ✓ def solve(n):
      if n <= 0: return 0          ← Base case
      return solve(n - 1) + 1

FIX STRATEGY:
  Every loop must change the loop variable toward termination.
  Every recursion must have a base case that's reachable.
```

---

## Bug #4: Modifying Collection While Iterating

```
EXAMPLE:
  ✗ for item in my_list:
      if condition(item):
          my_list.remove(item)     ← Skips elements!
  
  ✓ my_list = [item for item in my_list
               if not condition(item)]
  
  OR:
  ✓ to_remove = [item for item in my_list if condition(item)]
    for item in to_remove:
        my_list.remove(item)

  ✗ for key in my_dict:
      if condition:
          del my_dict[key]         ← RuntimeError!
  
  ✓ keys_to_remove = [k for k in my_dict if condition(k)]
    for key in keys_to_remove:
        del my_dict[key]
```

---

## Bug #5: Integer Overflow

```
EXAMPLE: Binary search mid calculation
  ✗ mid = (left + right) / 2      ← Overflow if both large
  ✓ mid = left + (right - left) // 2

EXAMPLE: Large sum
  ✗ total = a * b                 ← Can overflow int range
  ✓ Use long/BigInteger in Java
  ✓ Python handles this automatically

EXAMPLE: Power calculation
  ✗ result = base ** exponent      ← Can be astronomical
  ✓ Use modular exponentiation:
    result = pow(base, exp, MOD)

WHERE IT MATTERS:
  Java: int max = 2,147,483,647
  C++:  int max = 2,147,483,647
  Python: No overflow (arbitrary precision)
```

---

## Bug #6: Null/None Pointer Access

```
EXAMPLE 1: Tree traversal
  ✗ def height(node):
      return 1 + max(height(node.left), height(node.right))
      ← Crashes when node is None!
  
  ✓ def height(node):
      if node is None:
          return 0
      return 1 + max(height(node.left), height(node.right))

EXAMPLE 2: Linked list
  ✗ while current.next:            ← Crashes if current=None
  ✓ while current and current.next:

EXAMPLE 3: Dictionary access
  ✗ value = my_dict[key]           ← KeyError
  ✓ value = my_dict.get(key, default_value)
```

---

## Bug #7: Wrong Return Position

```
EXAMPLE:
  ✗ def find_target(nums, target):
      for i, num in enumerate(nums):
          if num == target:
              return i
          return -1                ← Returns -1 after first element!
  
  ✓ def find_target(nums, target):
      for i, num in enumerate(nums):
          if num == target:
              return i
      return -1                    ← Returns -1 only after full loop

  The indentation difference is critical!
  return inside loop = return at wrong time
  return after loop = return only when not found
```

---

## Bug Prevention Checklist

```
BEFORE CODING:
  ☐ What should I initialize max/min to?
  ☐ What are my loop bounds?
  ☐ What's my base case for recursion?

WHILE CODING:
  ☐ Does every loop variable progress toward termination?
  ☐ Am I checking for null/None before accessing?
  ☐ Is my return statement at the right indentation?

AFTER CODING:
  ☐ Dry run with empty input
  ☐ Dry run with single element
  ☐ Dry run with normal input
  ☐ Check for off-by-one at boundaries
```

---

## Summary Table

| Bug Type | Frequency | Prevention |
|----------|-----------|------------|
| Off-by-one | Very common | Test with size 0, 1, 2 |
| Wrong init | Common | Use -inf/inf, not 0 |
| Infinite loop | Common | Verify loop variable changes |
| Modify while iterating | Moderate | Use list comprehension |
| Integer overflow | Moderate | Use safe mid calculation |
| Null pointer | Common | Guard clause before access |
| Wrong return position | Common | Check indentation carefully |

---

## Quick Revision Questions

1. **What's the most common bug in programming interviews?**
   - Off-by-one errors: wrong loop bounds, `<` vs `<=`, wrong substring length calculation

2. **How do you prevent wrong initialization bugs?**
   - Use `float('-inf')` for max tracking, `float('inf')` for min tracking, or initialize with the first element (after empty check)

3. **What causes infinite loops in binary search?**
   - Not updating `left` or `right` after comparison, or using wrong mid calculation that doesn't make progress

4. **How do you safely calculate mid in binary search?**
   - `mid = left + (right - left) // 2` instead of `(left + right) // 2` to prevent overflow

5. **What's the "wrong return position" bug?**
   - Placing `return -1` (or default) inside the loop instead of after it, causing the function to return after checking only the first element

6. **How do you handle null/None safely in tree problems?**
   - Always add `if node is None: return [base_value]` as the first line of any recursive tree function

---

[← Previous: Dry Run Technique](04-dry-run-technique.md) | [Next: Unit 7 — When Stuck on a Problem →](../07-Handling-Difficult-Situations/01-when-stuck-on-a-problem.md)

---
[← Back to README](../README.md)
