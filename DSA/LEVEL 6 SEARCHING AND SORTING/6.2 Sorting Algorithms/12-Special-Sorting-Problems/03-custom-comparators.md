# Unit 12: Special Sorting Problems — Custom Comparators

[← Previous: Sort by Frequency](02-sort-by-frequency.md) | [Next: Sorting Linked Lists →](04-sorting-linked-lists.md)

---

## Overview

Custom comparators let you define **any sorting order** beyond simple ascending/descending. This chapter covers how to write comparators in Java, Python, and C++, and tackles common sorting problems that require custom ordering.

---

## What Is a Comparator?

```
  A comparator is a function that defines the ORDER between two elements.
  
  compare(a, b) returns:
    Negative  → a comes BEFORE b
    Zero      → a and b are EQUAL in ordering
    Positive  → a comes AFTER b
  
  Example: Sort integers in descending order
  compare(a, b) = b - a
  
  If a=3, b=7: compare(3,7) = 7-3 = 4 > 0 → 7 before 3 ✓
```

---

## Java Comparators

### Using Comparator Interface

```java
import java.util.*;

// Method 1: Lambda
Arrays.sort(arr, (a, b) -> b - a);  // descending

// Method 2: Method reference
Arrays.sort(arr, Comparator.reverseOrder());

// Method 3: Multi-key comparator
List<int[]> students = ...;  // [name_hash, gpa, age]
students.sort(Comparator
    .comparingInt((int[] s) -> s[1])     // by GPA ascending
    .reversed()                           // actually descending GPA
    .thenComparingInt(s -> s[2]));        // then by age ascending
```

### Common Comparator Patterns

```java
// Sort strings by length, then alphabetically
Arrays.sort(words, Comparator
    .comparingInt(String::length)
    .thenComparing(Comparator.naturalOrder()));

// Sort by absolute value
Arrays.sort(arr, (a, b) -> Math.abs(a) - Math.abs(b));

// Sort even numbers before odd, then ascending
Arrays.sort(arr, (a, b) -> {
    if (a % 2 != b % 2) return (a % 2) - (b % 2); // even first
    return a - b;  // then ascending
});

// Sort by last digit
Arrays.sort(arr, (a, b) -> (a % 10) - (b % 10));
```

---

## Python Comparators

### Using `key` Function (Preferred)

```python
# Sort by length
words.sort(key=len)

# Sort by absolute value
nums.sort(key=abs)

# Sort by multiple keys (tuple comparison)
students.sort(key=lambda s: (-s.gpa, s.name))
# Higher GPA first, then alphabetical name

# Sort by last digit
nums.sort(key=lambda x: x % 10)
```

### Using `functools.cmp_to_key` (For Complex Comparisons)

```python
from functools import cmp_to_key

def compare(a, b):
    # Custom: sort by digit sum, then by value
    sum_a = sum(int(d) for d in str(abs(a)))
    sum_b = sum(int(d) for d in str(abs(b)))
    if sum_a != sum_b:
        return sum_a - sum_b
    return a - b

nums.sort(key=cmp_to_key(compare))
```

---

## C++ Comparators

```cpp
#include <algorithm>
#include <vector>
using namespace std;

// Lambda comparator
sort(arr.begin(), arr.end(), [](int a, int b) {
    return a > b;  // descending (return true if a should come first)
});

// Sort by absolute value
sort(arr.begin(), arr.end(), [](int a, int b) {
    return abs(a) < abs(b);
});

// Function object (functor)
struct CompareByLength {
    bool operator()(const string& a, const string& b) const {
        if (a.size() != b.size()) return a.size() < b.size();
        return a < b;  // alphabetical if same length
    }
};
sort(words.begin(), words.end(), CompareByLength());
```

> **C++ note**: The comparator must define a **strict weak ordering**:
> - `comp(a,a)` must be `false`
> - If `comp(a,b)` is true, `comp(b,a)` must be `false`
> - Transitivity must hold

---

## Classic Problem: Largest Number (LeetCode 179)

```
  Given: [3, 30, 34, 5, 9]
  Form the largest possible number: "9534330"
  
  Key insight: Compare concatenations
  "330" vs "303" → "330" > "303" → 3 before 30
  "534" vs "345" → "534" > "345" → 5 before 34
```

### Java Solution

```java
public String largestNumber(int[] nums) {
    String[] strs = new String[nums.length];
    for (int i = 0; i < nums.length; i++)
        strs[i] = String.valueOf(nums[i]);
    
    Arrays.sort(strs, (a, b) -> (b + a).compareTo(a + b));
    
    if (strs[0].equals("0")) return "0";
    
    StringBuilder sb = new StringBuilder();
    for (String s : strs) sb.append(s);
    return sb.toString();
}
```

### Python Solution

```python
from functools import cmp_to_key

def largestNumber(nums):
    strs = list(map(str, nums))
    strs.sort(key=cmp_to_key(lambda a, b: -1 if a+b > b+a else 1))
    result = ''.join(strs)
    return '0' if result[0] == '0' else result
```

---

## Classic Problem: Sort by Parity (Even Before Odd)

```
  Input:  [3, 1, 2, 4, 5, 8]
  Output: [2, 4, 8, 3, 1, 5]  (evens first, then odds)
```

### Two-Pointer In-Place (O(n) time, O(1) space)

```java
public static void sortByParity(int[] arr) {
    int left = 0, right = arr.length - 1;
    while (left < right) {
        if (arr[left] % 2 == 0) {
            left++;
        } else if (arr[right] % 2 == 1) {
            right--;
        } else {
            // left is odd, right is even → swap
            int temp = arr[left];
            arr[left] = arr[right];
            arr[right] = temp;
            left++;
            right--;
        }
    }
}
```

### Using Comparator (Stable)

```python
nums.sort(key=lambda x: x % 2)  # 0 (even) before 1 (odd)
```

---

## Classic Problem: Custom Alphabet Sort

```
  Given a custom alphabet order, sort strings accordingly.
  
  Custom order: "hlabcdefgijkmnopqrstuvwxyz"
  Words: ["hello", "leetcode"]
  
  In this order, 'h' comes before 'l', so "hello" < "leetcode"
```

```java
public boolean isAlienSorted(String[] words, String order) {
    int[] rank = new int[26];
    for (int i = 0; i < 26; i++) {
        rank[order.charAt(i) - 'a'] = i;
    }
    
    for (int i = 0; i < words.length - 1; i++) {
        if (!isOrdered(words[i], words[i + 1], rank))
            return false;
    }
    return true;
}

private boolean isOrdered(String w1, String w2, int[] rank) {
    int len = Math.min(w1.length(), w2.length());
    for (int i = 0; i < len; i++) {
        if (rank[w1.charAt(i) - 'a'] < rank[w2.charAt(i) - 'a'])
            return true;
        if (rank[w1.charAt(i) - 'a'] > rank[w2.charAt(i) - 'a'])
            return false;
    }
    return w1.length() <= w2.length();
}
```

---

## Comparator Pitfalls

```
  ┌──────────────────────────────────────────────────────────────┐
  │  PITFALL 1: Integer Overflow                                │
  │                                                              │
  │  compare(a, b) = a - b    ← DANGEROUS for large values!   │
  │  If a = Integer.MAX_VALUE, b = -1:                         │
  │  a - b = 2147483647 - (-1) = overflow! → negative number  │
  │                                                              │
  │  FIX: Use Integer.compare(a, b) instead                    │
  ├──────────────────────────────────────────────────────────────┤
  │  PITFALL 2: Inconsistent Comparator                        │
  │                                                              │
  │  compare(a, b) must be consistent:                         │
  │  If a < b and b < c, then a < c (transitivity)            │
  │  Violating this → unpredictable behavior, exceptions       │
  ├──────────────────────────────────────────────────────────────┤
  │  PITFALL 3: Forgetting to Handle Equal Elements            │
  │                                                              │
  │  compare(a, b) should return 0 when a equals b             │
  │  Returning non-zero for equal elements → unstable sort     │
  └──────────────────────────────────────────────────────────────┘
```

---

## Multi-Key Sorting Pattern

```java
// The general pattern for multi-key sorting:
// Sort by key1, then key2, then key3...

// Java
list.sort(Comparator
    .comparing(obj -> obj.key1)
    .thenComparing(obj -> obj.key2)
    .thenComparing(obj -> obj.key3));

// Python
list.sort(key=lambda obj: (obj.key1, obj.key2, obj.key3))

// C++
sort(vec.begin(), vec.end(), [](const auto& a, const auto& b) {
    if (a.key1 != b.key1) return a.key1 < b.key1;
    if (a.key2 != b.key2) return a.key2 < b.key2;
    return a.key3 < b.key3;
});
```

---

## Summary Table

| Problem | Comparator Logic | Time |
|---------|-----------------|------|
| Descending | `b - a` | O(n log n) |
| By absolute value | `|a| - |b|` | O(n log n) |
| Even before odd | `(a%2) - (b%2)` | O(n log n) or O(n) |
| By frequency | `freq[b] - freq[a]` | O(n log n) |
| Largest number | `(b+a).compareTo(a+b)` | O(n log n) |
| Custom alphabet | Rank-based comparison | O(n × L) |

---

## Quick Revision Questions

1. **Write a comparator to sort integers by digit count, then by value.**
2. **What's the integer overflow pitfall in `(a, b) -> a - b`?**
3. **Solve LeetCode 179 (Largest Number) — explain why string concatenation comparison works.**
4. **Write a multi-key sort: by grade descending, then by name alphabetically.**
5. **What is strict weak ordering and why does C++ require it?**

---

[← Previous: Sort by Frequency](02-sort-by-frequency.md) | [Next: Sorting Linked Lists →](04-sorting-linked-lists.md)
