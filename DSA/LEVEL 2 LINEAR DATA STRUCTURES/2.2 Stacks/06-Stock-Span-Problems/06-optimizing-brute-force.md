# Chapter 6: Optimizing Brute Force with Stack

[← Previous: Pattern Recognition](05-pattern-recognition.md) | [Next: Min Stack →](../07-Advanced-Stack-Problems/01-min-stack.md) | [↑ Back to Unit 6](../README.md#unit-6-stock-span-problems) | [🏠 Home](../README.md)

---

## Overview

This chapter demonstrates the systematic process of converting **O(n²) brute force** solutions into **O(n) stack-based** solutions. We walk through three problems showing the transformation step by step.

---

## The Optimization Pattern

```
┌──────────────────────────────────────────────────────────┐
│  BRUTE FORCE → STACK TRANSFORMATION                     │
│                                                          │
│  Step 1: Identify the brute force pattern               │
│    → Nested loop where inner loop scans left/right      │
│                                                          │
│  Step 2: Identify what the inner loop searches for      │
│    → First element satisfying a condition (>, <, ≥, ≤)  │
│                                                          │
│  Step 3: Replace inner loop with stack                  │
│    → Elements waiting for their answer sit on stack     │
│    → When answer arrives, pop and record                │
│                                                          │
│  Step 4: Verify amortized O(n)                          │
│    → Each element pushed once, popped at most once      │
└──────────────────────────────────────────────────────────┘
```

---

## Example 1: Remove K Digits

**Problem**: Remove k digits from number string to make the smallest possible number.

```
Input: num = "1432219", k = 3
Output: "1219"

Remove 4, 3, 2 → "1219" (smallest possible)
```

### Brute Force: O(n × k) or O(n^k)

```
Try all combinations of removing k digits:
  C(n, k) possibilities → exponential or at minimum O(n×k)
```

### Optimized with Increasing Stack: O(n)

```
┌──────────────────────────────────────────────────────────┐
│  Key Insight: To make the number smallest, remove        │
│  digits that are LARGER than the next digit.             │
│  This is a monotonically INCREASING stack problem!       │
│                                                          │
│  If current digit < stack top → pop (remove the larger   │
│  digit to make number smaller)                           │
└──────────────────────────────────────────────────────────┘
```

```
FUNCTION removeKdigits(num, k):
    stack ← empty
    
    FOR each digit d in num:
        WHILE k > 0 AND stack NOT empty AND d < stack.top():
            stack.pop()
            k ← k - 1
        stack.push(d)
    
    // If k > 0, remove from end (stack is increasing, largest at top)
    WHILE k > 0:
        stack.pop()
        k ← k - 1
    
    // Build result, remove leading zeros
    result ← stack contents from bottom to top
    Remove leading zeros
    RETURN result (or "0" if empty)
```

### Trace: num = "1432219", k = 3

```
d='1': Push '1'                      Stack: [1]       k=3
d='4': 4>1 → Push '4'               Stack: [1,4]     k=3
d='3': 3<4 → Pop '4', k=2           Stack: [1]
       3>1 → Push '3'               Stack: [1,3]     k=2
d='2': 2<3 → Pop '3', k=1           Stack: [1]
       2>1 → Push '2'               Stack: [1,2]     k=1
d='2': 2≤2 → Push '2'              Stack: [1,2,2]    k=1
d='1': 1<2 → Pop '2', k=0           Stack: [1,2]
       k=0 → Push '1'               Stack: [1,2,1]   k=0
d='9': Push '9'                      Stack: [1,2,1,9] k=0

Result: "1219" ✓
```

---

## Example 2: Sum of Subarray Minimums

**Problem**: Find sum of min(subarray) for all contiguous subarrays.

```
arr = [3, 1, 2, 4]

Subarrays and their mins:
  [3]=3, [1]=1, [2]=2, [4]=4
  [3,1]=1, [1,2]=1, [2,4]=2
  [3,1,2]=1, [1,2,4]=1
  [3,1,2,4]=1

Sum = 3+1+2+4+1+1+2+1+1+1 = 17
```

### Brute Force: O(n³) or O(n²)

```
FOR each subarray (i, j):     // O(n²) pairs
    Find minimum               // O(n) or O(1) with tracking
    Add to sum

Total: O(n²) at best
```

### Optimized with Stack: O(n)

```
Insight: For each element arr[i], count HOW MANY subarrays
have arr[i] as their minimum.

left[i] = # of subarrays ending at i where arr[i] is min
        = i - PSE_index(i)

right[i]= # of subarrays starting at i where arr[i] is min
        = NSE_index(i) - i

Total subarrays where arr[i] is min = left[i] × right[i]
Contribution = arr[i] × left[i] × right[i]
```

```
FUNCTION sumSubarrayMins(arr):
    n ← length(arr)
    
    // Find PSE indices (use strict < for left, ≤ for right to avoid duplicates)
    pse ← array of -1s
    nse ← array of ns
    stack ← empty
    
    // PSE
    FOR i = 0 TO n-1:
        WHILE stack NOT empty AND arr[stack.top()] >= arr[i]:
            stack.pop()
        pse[i] ← IF stack empty THEN -1 ELSE stack.top()
        stack.push(i)
    
    // NSE (strictly less for right boundary)
    stack ← empty
    FOR i = n-1 DOWNTO 0:
        WHILE stack NOT empty AND arr[stack.top()] > arr[i]:
            stack.pop()
        nse[i] ← IF stack empty THEN n ELSE stack.top()
        stack.push(i)
    
    // Calculate sum
    sum ← 0
    FOR i = 0 TO n-1:
        left ← i - pse[i]
        right ← nse[i] - i
        sum ← sum + arr[i] × left × right
    
    RETURN sum
```

### Trace: arr = [3, 1, 2, 4]

```
PSE: [-1, -1, 1, 2]     (previous smaller indices)
NSE: [1, 4, 4, 4]       (next smaller or equal indices)

i=0: left=0-(-1)=1, right=1-0=1, contribution=3×1×1=3
i=1: left=1-(-1)=2, right=4-1=3, contribution=1×2×3=6
i=2: left=2-1=1,    right=4-2=2, contribution=2×1×2=4
i=3: left=3-2=1,    right=4-3=1, contribution=4×1×1=4

Sum = 3+6+4+4 = 17 ✓
```

---

## Example 3: 132 Pattern

**Problem**: Find if there exist i < j < k such that arr[i] < arr[k] < arr[j] (a "132 pattern").

```
arr = [3, 1, 4, 2]
Has 132 pattern: 1, 4, 2 where 1 < 2 < 4 ✓

arr = [1, 2, 3, 4]
No 132 pattern ✗
```

### Brute Force: O(n³) or O(n²)

```
Check all triples (i,j,k) → O(n³)
Optimize by tracking min prefix → O(n²)
```

### Stack Solution: O(n)

```
FUNCTION find132pattern(arr):
    n ← length(arr)
    stack ← empty stack
    third ← -∞    // The "2" in "132" (largest valid third element)
    
    // Scan RIGHT to LEFT
    FOR i = n-1 DOWNTO 0:
        // Check if current element can be "1" in "132"
        IF arr[i] < third:
            RETURN true    // Found: arr[i] < third < some element in stack
        
        // Maintain decreasing stack, popped elements become candidates for "2"
        WHILE stack NOT empty AND arr[i] > stack.top():
            third ← stack.pop()    // This is the current best "2"
        
        stack.push(arr[i])
    
    RETURN false
```

### Trace: arr = [3, 1, 4, 2]

```
i=3: arr[3]=2, 2>-∞? no. Push 2.       Stack: [2], third=-∞
i=2: arr[2]=4, 4>-∞? no. 
     4>2 → Pop 2, third=2.             Stack: [], third=2
     Push 4.                            Stack: [4], third=2
i=1: arr[1]=1, 1<2? YES!               → Return TRUE ✓

The pattern: arr[1]=1 (the "1"), some popped element formed "3"=4,
             and third=2 is the "2". Pattern: 1 < 2 < 4 ✓
```

---

## Transformation Summary

```
┌──────────────────────┬────────────────┬──────────────────┐
│ Problem              │ Brute Force    │ Stack            │
├──────────────────────┼────────────────┼──────────────────┤
│ Remove K Digits      │ O(n×k) or O(nk)│ O(n)            │
│ Sum Subarray Mins    │ O(n²)          │ O(n)             │
│ 132 Pattern          │ O(n²)          │ O(n)             │
│ Next Greater Element │ O(n²)          │ O(n)             │
│ Stock Span           │ O(n²)          │ O(n)             │
│ Largest Rect Histo   │ O(n²)          │ O(n)             │
│ Trapping Rain Water  │ O(n²)          │ O(n)             │
└──────────────────────┴────────────────┴──────────────────┘
```

---

## The General Template

```
// Elements "waiting" for their answer
stack ← empty

FOR i = 0 TO n-1:   // (or n-1 to 0 for reverse scan)
    
    // Pop elements that found their answer
    WHILE stack NOT empty AND condition(arr[i], stack.top()):
        popped ← stack.pop()
        // Record answer for popped element
    
    // Current element starts waiting
    stack.push(i)  // or push (value, metadata)

// Handle elements still waiting (no answer found)
```

---

## Complexity Analysis

| Aspect | All Stack Problems |
|--------|-------------------|
| **Time** | O(n) — amortized |
| **Space** | O(n) — stack size |
| **Key property** | Each element: ≤1 push + ≤1 pop = O(1) amortized |

---

## Quick Revision Questions

1. **What brute force pattern suggests a stack optimization?**
   > Nested loops where the inner loop scans left/right to find the first element satisfying a comparison.

2. **In Remove K Digits, why do we use an increasing stack?**
   > To ensure digits are in ascending order; removing a larger digit before a smaller one creates a smaller number.

3. **How does the 132 Pattern problem use the stack differently?**
   > It scans right-to-left with a decreasing stack, using popped values as the "2" candidate and checking if current element (the "1") is smaller.

4. **What is the general principle behind all stack optimizations?**
   > Instead of scanning for each element individually (O(n) per element), the stack remembers "unanswered" elements. When the answer arrives, it resolves multiple elements at once, amortizing to O(1) per element.

5. **How do you handle duplicate elements in Sum of Subarray Minimums?**
   > Use strict `<` for one direction and `<=` for the other to avoid counting the same subarray minimum twice.

---

[← Previous: Pattern Recognition](05-pattern-recognition.md) | [Next: Min Stack →](../07-Advanced-Stack-Problems/01-min-stack.md) | [↑ Back to Unit 6](../README.md#unit-6-stock-span-problems) | [🏠 Home](../README.md)
