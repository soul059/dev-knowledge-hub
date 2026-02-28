# Chapter 3: Linear Search Variations

[← Previous: Sentinel Search](02-sentinel-search.md) | [Next: When Linear Search is Best →](04-when-linear-search-is-best.md)

---

## Overview

Beyond the basic linear search, several **variations** exist that optimize for specific scenarios or solve slightly different problems. Understanding these variations strengthens your problem-solving toolkit.

---

## Variation 1: Search from Both Ends

Instead of scanning from left to right only, scan from **both ends** simultaneously toward the middle. This cuts the number of iterations roughly in half.

```
   Target = 17

   ┌────┬────┬────┬────┬────┬────┬────┬────┐
   │ 10 │ 23 │  5 │ 42 │ 17 │  8 │ 31 │ 56 │
   └────┴────┴────┴────┴────┴────┴────┴────┘
     ▲                                    ▲
    left                                right

   Iteration 1: arr[0]=10 ≠ 17, arr[7]=56 ≠ 17
   Iteration 2: arr[1]=23 ≠ 17, arr[6]=31 ≠ 17
   Iteration 3: arr[2]= 5 ≠ 17, arr[5]= 8 ≠ 17
   Iteration 4: arr[3]=42 ≠ 17, arr[4]=17 = 17 ✓ FOUND at index 4!
```

### Pseudocode

```
FUNCTION searchBothEnds(arr, n, target):
    left = 0
    right = n - 1
    
    WHILE left <= right:
        IF arr[left] == target:
            RETURN left
        IF arr[right] == target:
            RETURN right
        left = left + 1
        right = right - 1
    
    RETURN -1
```

### Implementation (Java)

```java
public static int searchBothEnds(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    
    while (left <= right) {
        if (arr[left] == target) return left;
        if (arr[right] == target) return right;
        left++;
        right--;
    }
    return -1;
}
```

| Aspect | Detail |
|--------|--------|
| Time Complexity | O(n) — but ~n/2 iterations |
| Space Complexity | O(1) |
| Advantage | On average, finds elements faster |
| Note | May return different index for duplicates |

---

## Variation 2: Find All Occurrences

Instead of returning the first match, find **every index** where the target appears.

```
   Target = 5

   Index:   0    1    2    3    4    5    6    7
          ┌────┬────┬────┬────┬────┬────┬────┬────┐
          │  5 │ 23 │  5 │ 42 │  5 │  8 │ 31 │  5 │
          └────┴────┴────┴────┴────┴────┴────┴────┘
            ↑         ↑         ↑                ↑
          Found     Found     Found            Found

   Result: [0, 2, 4, 7]
```

### Pseudocode

```
FUNCTION findAllOccurrences(arr, n, target):
    result = []
    FOR i = 0 TO n - 1:
        IF arr[i] == target:
            result.ADD(i)
    RETURN result
```

### Implementation (Python)

```python
def find_all_occurrences(arr, target):
    return [i for i, val in enumerate(arr) if val == target]
```

---

## Variation 3: Find Minimum / Maximum

A specialized linear search that finds the smallest or largest element.

```
   ┌────┬────┬────┬────┬────┬────┐
   │ 10 │  3 │  7 │  1 │  9 │  4 │
   └────┴────┴────┴────┴────┴────┘

   Finding Minimum:
   min = 10
   Step 1: 3 < 10 → min = 3
   Step 2: 7 > 3  → min = 3
   Step 3: 1 < 3  → min = 1
   Step 4: 9 > 1  → min = 1
   Step 5: 4 > 1  → min = 1

   Result: min = 1 at index 3
```

### Pseudocode

```
FUNCTION findMin(arr, n):
    minVal = arr[0]
    minIdx = 0
    FOR i = 1 TO n - 1:
        IF arr[i] < minVal:
            minVal = arr[i]
            minIdx = i
    RETURN minIdx
```

---

## Variation 4: Find Second Largest

Requires scanning with **two tracking variables**.

```
   ┌────┬────┬────┬────┬────┬────┐
   │ 10 │ 45 │  7 │ 38 │ 45 │  4 │
   └────┴────┴────┴────┴────┴────┘

   Tracking: first = -∞, second = -∞

   i=0: 10 > -∞  → first=10, second=-∞
   i=1: 45 > 10  → first=45, second=10
   i=2: 7  < 45  and 7 < 10  → no change
   i=3: 38 < 45  and 38 > 10 → second=38
   i=4: 45 == 45 → skip (equal to first)
   i=5: 4  < 45  and 4 < 38  → no change

   Result: second largest = 38
```

### Implementation (Java)

```java
public static int findSecondLargest(int[] arr) {
    int first = Integer.MIN_VALUE;
    int second = Integer.MIN_VALUE;
    
    for (int val : arr) {
        if (val > first) {
            second = first;
            first = val;
        } else if (val > second && val != first) {
            second = val;
        }
    }
    return second;
}
```

---

## Variation 5: Search in Strings

Linear search on **characters in a string** or **strings in a list**.

```
   Finding character 'o' in "Hello World"

   H  e  l  l  o     W  o  r  l  d
   0  1  2  3  4  5  6  7  8  9  10
                  ↑
               FOUND at index 4 (first occurrence)
```

### Python Implementation

```python
def search_char(text, char):
    for i, c in enumerate(text):
        if c == char:
            return i
    return -1
```

---

## Variation 6: Recursive Linear Search

Instead of using a loop, use **recursion** to scan elements.

```
   searchRecursive([10, 23, 5, 42], 0, 42)
       │
       ├── arr[0]=10 ≠ 42
       │   └── searchRecursive([10, 23, 5, 42], 1, 42)
       │       │
       │       ├── arr[1]=23 ≠ 42
       │       │   └── searchRecursive([10, 23, 5, 42], 2, 42)
       │       │       │
       │       │       ├── arr[2]=5 ≠ 42
       │       │       │   └── searchRecursive([10, 23, 5, 42], 3, 42)
       │       │       │       │
       │       │       │       ├── arr[3]=42 == 42         ← FOUND!
       │       │       │       └── RETURN 3    ──────────────→
       │       │       └── RETURN 3
       │       └── RETURN 3
       └── RETURN 3
```

### Pseudocode

```
FUNCTION searchRecursive(arr, i, target):
    IF i >= LENGTH(arr):
        RETURN -1
    IF arr[i] == target:
        RETURN i
    RETURN searchRecursive(arr, i + 1, target)
```

### Implementation (Java)

```java
public static int linearSearchRecursive(int[] arr, int i, int target) {
    if (i >= arr.length) return -1;
    if (arr[i] == target) return i;
    return linearSearchRecursive(arr, i + 1, target);
}
```

| Aspect | Detail |
|--------|--------|
| Time | O(n) |
| Space | **O(n)** — call stack depth |
| Use case | Educational / When recursion is required |
| Warning | Stack overflow on large arrays |

---

## Variation 7: Search with Condition (Predicate Search)

Find the first element satisfying an **arbitrary condition**.

```
   Find first even number:

   ┌────┬────┬────┬────┬────┐
   │  7 │  3 │  8 │  1 │  4 │
   └────┴────┴────┴────┴────┘
              ↑
           8 is even → FOUND at index 2
```

### Python Implementation

```python
def find_first(arr, predicate):
    for i, val in enumerate(arr):
        if predicate(val):
            return i
    return -1

# Usage
result = find_first([7, 3, 8, 1, 4], lambda x: x % 2 == 0)  # Returns 2
```

---

## All Variations at a Glance

```
                    Linear Search Variations
                    ════════════════════════
                            │
        ┌───────────┬───────┼───────┬───────────┐
        │           │       │       │           │
   ┌────▼────┐ ┌────▼───┐ ┌▼────┐ ┌▼─────┐ ┌───▼────┐
   │Both Ends│ │Find All│ │Min/ │ │Recur-│ │Predic- │
   │ Search  │ │Occur.  │ │Max  │ │sive  │ │ate     │
   └─────────┘ └────────┘ └─────┘ └──────┘ └────────┘
```

---

## Summary Table

| Variation | Returns | Time | Space | Key Use Case |
|-----------|---------|------|-------|---|
| Both Ends | First match from either end | O(n) | O(1) | Faster average case |
| Find All | List of all indices | O(n) | O(k) | Count/list all matches |
| Find Min/Max | Index of min/max | O(n) | O(1) | Finding extremes |
| Second Largest | Value of 2nd largest | O(n) | O(1) | Single-pass ranking |
| String Search | Index in string | O(n) | O(1) | Character lookup |
| Recursive | First match index | O(n) | O(n) | Educational / forced recursion |
| Predicate | First satisfying index | O(n) | O(1) | Custom conditions |

---

## Quick Revision Questions

1. **How does searching from both ends reduce the number of iterations?**
2. **What is the space complexity of "Find All Occurrences"? Why is it not O(1)?**
3. **Why is recursive linear search generally avoided in practice?**
4. **How can you find both the minimum and maximum in a single pass?**
5. **What is a predicate search and how does it generalize linear search?**
6. **If searching from both ends, what happens when `left == right`?**

---

[← Previous: Sentinel Search](02-sentinel-search.md) | [Next: When Linear Search is Best →](04-when-linear-search-is-best.md)
