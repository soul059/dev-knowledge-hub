# Chapter 6: Thinking Recursively

## Overview
Thinking recursively is a **mental skill** that takes practice. Instead of thinking about the entire problem at once, you focus on **one step** and trust that recursion handles the rest. This chapter teaches you a systematic approach to convert any problem into a recursive solution.

---

## The Recursive Thinking Framework

```
┌──────────────────────────────────────────────────────┐
│          5-STEP RECURSIVE THINKING                    │
│                                                      │
│  Step 1: DEFINE — What does f(input) return?         │
│  Step 2: BASE   — What's the simplest input?         │
│  Step 3: TRUST  — Assume f(smaller) works perfectly  │
│  Step 4: BUILD  — Use f(smaller) to compute f(input) │
│  Step 5: VERIFY — Trace with small examples          │
└──────────────────────────────────────────────────────┘
```

---

## Worked Example 1: Sum of Array

**Problem**: Find the sum of all elements in an array.

### Applying the Framework

```
Step 1: DEFINE
  sum(arr, i) = sum of elements from index i to end

Step 2: BASE  
  sum(arr, n) = 0     (when i equals array length, nothing left)

Step 3: TRUST
  Assume sum(arr, i+1) correctly returns sum from (i+1) to end

Step 4: BUILD
  sum(arr, i) = arr[i] + sum(arr, i+1)
  "Current element + sum of the rest"

Step 5: VERIFY for arr = [3, 1, 4, 2]
  sum(arr, 0) = 3 + sum(arr, 1)
               = 3 + (1 + sum(arr, 2))
               = 3 + (1 + (4 + sum(arr, 3)))
               = 3 + (1 + (4 + (2 + sum(arr, 4))))
               = 3 + (1 + (4 + (2 + 0)))
               = 3 + 1 + 4 + 2 = 10 ✓
```

---

## Worked Example 2: Reverse a String

**Problem**: Reverse a string "hello" → "olleh"

```
Step 1: DEFINE
  reverse(s) = the reversed version of string s

Step 2: BASE
  reverse("") = ""      (empty string is already reversed)
  reverse("x") = "x"    (single char is already reversed)

Step 3: TRUST
  Assume reverse(s[1:]) correctly reverses everything except first char

Step 4: BUILD
  reverse(s) = reverse(s[1:]) + s[0]
  "Reverse the rest, then put first char at end"

Step 5: VERIFY for s = "hello"
  reverse("hello") = reverse("ello") + "h"
                    = (reverse("llo") + "e") + "h"
                    = ((reverse("lo") + "l") + "e") + "h"
                    = (((reverse("o") + "l") + "l") + "e") + "h"
                    = ((("o" + "l") + "l") + "e") + "h"
                    = "olleh" ✓
```

```
Visualization:
  "hello"
  ├── first = "h"
  └── rest = "ello"
      ├── first = "e"
      └── rest = "llo"
          ├── first = "l"
          └── rest = "lo"
              ├── first = "l"
              └── rest = "o" ← base case
              
Unwinding:
  "o"           → "o"
  "o" + "l"     → "ol"
  "ol" + "l"    → "oll"
  "oll" + "e"   → "olle"
  "olle" + "h"  → "olleh"
```

---

## Worked Example 3: Check if Array is Sorted

```
Step 1: DEFINE
  isSorted(arr, i) = true if arr[i..end] is sorted ascending

Step 2: BASE
  isSorted(arr, n-1) = true   (single element is always sorted)
  isSorted(arr, n) = true     (no elements — trivially sorted)

Step 3: TRUST
  Assume isSorted(arr, i+1) correctly checks arr[i+1..end]

Step 4: BUILD
  isSorted(arr, i) = (arr[i] <= arr[i+1]) AND isSorted(arr, i+1)
  "Current pair is in order AND the rest is sorted"

Step 5: VERIFY for arr = [1, 3, 5, 2]
  isSorted(arr, 0):
    arr[0]<=arr[1]? 1<=3 ✓  AND isSorted(arr, 1)
    arr[1]<=arr[2]? 3<=5 ✓  AND isSorted(arr, 2)
    arr[2]<=arr[3]? 5<=2 ✗  → return false ✓
```

---

## Common Recursive Thinking Patterns

### Pattern 1: Process First, Recurse Rest
```
f(collection):
    process(first_element)
    f(rest_of_collection)

Example: Print all elements
  print(arr[0])
  printAll(arr[1:])
```

### Pattern 2: Recurse First, Process After
```
f(collection):
    result = f(rest_of_collection)
    process(first_element, result)

Example: Reverse print
  reversePrint(arr[1:])
  print(arr[0])
```

### Pattern 3: Include or Exclude (Choose)
```
f(collection, index):
    // Choice 1: Include current element
    include = f(collection, index+1) with arr[index]
    // Choice 2: Exclude current element
    exclude = f(collection, index+1) without arr[index]
    return combine(include, exclude)

Example: Generate subsets, knapsack
```

### Pattern 4: Try All Options
```
f(state):
    for each option in available_options:
        choose(option)
        f(next_state)
        unchoose(option)

Example: Permutations, N-Queens, Sudoku
```

```
┌──────────────────────────────────────────────┐
│        PATTERN SELECTION GUIDE               │
│                                              │
│  "Process each element"                      │
│     → Pattern 1 or 2                         │
│                                              │
│  "Include/exclude decisions"                 │
│     → Pattern 3 (subsets, combinations)      │
│                                              │
│  "Try all arrangements"                      │
│     → Pattern 4 (permutations, backtracking) │
│                                              │
│  "Split in half"                             │
│     → Divide and conquer                     │
└──────────────────────────────────────────────┘
```

---

## The "What's My Contribution?" Technique

For each level of recursion, ask: **"What is MY job at this level?"**

```
PROBLEM: Count occurrences of 'x' in a string

My job at each level:
┌─────────────────────────────────────────────┐
│ Check if MY character (first char) is 'x'   │
│ If yes: 1 + count done by recursion          │
│ If no:  0 + count done by recursion          │
│                                             │
│ I don't care HOW recursion counts the rest! │
└─────────────────────────────────────────────┘

function countX(str):
    if str is empty: return 0           // Base
    myContribution = (str[0] == 'x') ? 1 : 0   // My job
    restCount = countX(str[1:])         // Trust recursion
    return myContribution + restCount    // Combine
```

---

## Avoiding Common Thinking Traps

### Trap 1: Trying to Trace Everything

```
❌ DON'T: "Let me trace every single call for n=20..."
✅ DO:    "I'll verify my logic works for n=0, 1, 2, 3
          and trust it generalizes."
```

### Trap 2: Over-Complicating the Recursive Case

```
❌ DON'T: Try to handle multiple elements at once
✅ DO:    Handle ONE element + let recursion do the rest

// BAD: trying to be "smart"
function sum(arr, i):
    if i >= len-1: return arr[i]
    return arr[i] + arr[i+1] + sum(arr, i+2)  // Too complex!

// GOOD: simple and clean
function sum(arr, i):
    if i >= len: return 0
    return arr[i] + sum(arr, i+1)
```

### Trap 3: Not Identifying the Right Subproblem

```
❌ Wrong subproblem:
   "How do I sort the whole array recursively?"

✅ Right subproblem:
   "If the REST of the array is already sorted,
    how do I insert ONE element?"
   → This is Insertion Sort!
```

---

## Practice Problems with Thinking Framework

### Problem: Count digits in a number

```
Step 1: DEFINE   countDigits(n) = number of digits in n
Step 2: BASE     countDigits of single digit (n < 10) = 1
Step 3: TRUST    countDigits(n/10) counts remaining digits
Step 4: BUILD    countDigits(n) = 1 + countDigits(n/10)
Step 5: VERIFY   countDigits(1234) = 1 + countDigits(123)
                                    = 1 + 1 + countDigits(12)
                                    = 1 + 1 + 1 + countDigits(1)
                                    = 1 + 1 + 1 + 1 = 4 ✓
```

### Problem: Sum of digits

```
Step 1: DEFINE   digitSum(n) = sum of all digits
Step 2: BASE     digitSum of single digit (n < 10) = n
Step 3: TRUST    digitSum(n/10) sums remaining digits
Step 4: BUILD    digitSum(n) = (n % 10) + digitSum(n/10)
Step 5: VERIFY   digitSum(123) = 3 + digitSum(12)
                               = 3 + 2 + digitSum(1)
                               = 3 + 2 + 1 = 6 ✓
```

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Define clearly | Know what f(input) returns before coding |
| Identify base case | Simplest input with obvious answer |
| Leap of faith | Trust the recursive call works |
| My contribution | What does THIS level do? |
| Pattern matching | Recognize include/exclude, try-all, divide |
| Verify small | Trace n=0,1,2,3 — don't trace large n |
| One step at a time | Handle one element, recurse the rest |

---

## Quick Revision Questions

1. **What are the 5 steps** of the recursive thinking framework?
2. **Apply the framework** to find the length of a linked list.
3. **What does "my contribution"** mean in recursive thinking?
4. **Name the 4 common recursive patterns** discussed in this chapter.
5. **Why should you NOT try** to trace recursion for large inputs?
6. **How do you decide** between include/exclude pattern vs try-all-options pattern?

---

[← Previous: When to Use Recursion](05-when-to-use-recursion.md) | [Next: Unit 2 — Factorial →](../02-Basic-Recursion/01-factorial.md)

[← Back to README](../README.md)
