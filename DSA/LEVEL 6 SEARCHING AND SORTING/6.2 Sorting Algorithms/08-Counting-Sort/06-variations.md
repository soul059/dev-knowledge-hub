# Unit 8: Counting Sort — Variations

[← Previous: Limitations & When to Use](05-limitations-and-when-to-use.md) | [Next: Unit 9 - Radix Sort →](../09-Radix-Sort/01-algorithm-description.md)

---

## Overview

Counting Sort has several useful variations for specific use cases — from simple frequency counting to Dutch National Flag problems and beyond.

---

## 1. Frequency Count Sort (Simplest Version)

```
  When you only need the sorted output (not stability):
  
  Array: [4, 2, 2, 8, 3, 3, 1]
  
  Count: [0, 1, 2, 2, 1, 0, 0, 0, 1]
          0  1  2  3  4  5  6  7  8
  
  Simply expand counts back:
  1 appears 1 time  → [1]
  2 appears 2 times → [1, 2, 2]
  3 appears 2 times → [1, 2, 2, 3, 3]
  4 appears 1 time  → [1, 2, 2, 3, 3, 4]
  8 appears 1 time  → [1, 2, 2, 3, 3, 4, 8]
  
  ✓ O(n + k) time
  ✓ O(k) space (no output array needed!)
  ✗ NOT stable (original element identity is lost)
```

```java
public static void frequencySort(int[] arr) {
    int max = Arrays.stream(arr).max().orElse(0);
    int min = Arrays.stream(arr).min().orElse(0);
    int[] count = new int[max - min + 1];
    
    for (int num : arr) count[num - min]++;
    
    int idx = 0;
    for (int i = 0; i < count.length; i++) {
        while (count[i]-- > 0) {
            arr[idx++] = i + min;
        }
    }
}
```

---

## 2. Counting Sort for Characters

```java
// Sort string characters
public static String sortString(String s) {
    int[] count = new int[26];  // lowercase only
    
    for (char c : s.toCharArray())
        count[c - 'a']++;
    
    StringBuilder result = new StringBuilder();
    for (int i = 0; i < 26; i++) {
        while (count[i]-- > 0)
            result.append((char)('a' + i));
    }
    return result.toString();
}

// "algorithm" → "aghilmort"
```

```
  Character frequency problems:
  
  ┌─────────────────────────────────────────────────────────┐
  │  • Check if two strings are anagrams                   │
  │  • Find first non-repeating character                  │
  │  • Count frequency of each character                   │
  │  • Sort string in alphabetical order                   │
  │  • Check if strings are permutations of each other     │
  └─────────────────────────────────────────────────────────┘
```

---

## 3. Counting Sort by Digit (Used in Radix Sort)

```java
// Sort array by a specific digit position
// digitPlace: 1 for ones, 10 for tens, 100 for hundreds...
public static void countingSortByDigit(int[] arr, int digitPlace) {
    int n = arr.length;
    int[] output = new int[n];
    int[] count = new int[10];  // digits 0-9
    
    // Count occurrences of each digit
    for (int num : arr) {
        int digit = (num / digitPlace) % 10;
        count[digit]++;
    }
    
    // Prefix sum
    for (int i = 1; i < 10; i++)
        count[i] += count[i - 1];
    
    // Build output (right-to-left for stability)
    for (int i = n - 1; i >= 0; i--) {
        int digit = (arr[i] / digitPlace) % 10;
        output[count[digit] - 1] = arr[i];
        count[digit]--;
    }
    
    System.arraycopy(output, 0, arr, 0, n);
}
```

```
  Example: Sort by ones digit
  
  Array: [329, 457, 657, 839, 436, 720, 355]
  
  Digits:  9    7    7    9    6    0    5
  Count:  [1, 0, 0, 0, 0, 1, 1, 2, 0, 2]
           0  1  2  3  4  5  6  7  8  9
  
  Prefix: [1, 1, 1, 1, 1, 2, 3, 5, 5, 7]
  
  Output: [720, 355, 436, 457, 657, 329, 839]
  
  Sorted by ones digit ✓
  Stability preserved for radix sort's next pass ✓
```

---

## 4. Count Inversions Using Counting

```
  An inversion is a pair (i, j) where i < j but A[i] > A[j].
  
  Counting approach for small range:
  
  Array: [3, 1, 2]
  
  Inversions: (3,1), (3,2) → 2 inversions
  
  Method: For each element, count how many SMALLER elements
  appear AFTER it using a count array (like a BIT/Fenwick tree).
```

---

## 5. In-Place Counting Sort (Cyclic Permutation)

```
  For arrays where values ARE the indices (0 to n-1):
  
  Array: [2, 0, 3, 1]    (permutation of 0-3)
  
  Goal: Place each value at its index position
  
  Cycle sort approach:
  i=0: arr[0]=2, put 2 at index 2: swap(0,2) → [3, 0, 2, 1]
       arr[0]=3, put 3 at index 3: swap(0,3) → [1, 0, 2, 3]
       arr[0]=1, put 1 at index 1: swap(0,1) → [0, 1, 2, 3]
       arr[0]=0 ✓
  i=1: arr[1]=1 ✓
  i=2: arr[2]=2 ✓
  i=3: arr[3]=3 ✓
  
  Result: [0, 1, 2, 3]  ← O(n), O(1) space!
```

```java
// Only works for 0 to n-1 values
public static void inPlaceCountingSort(int[] arr) {
    int i = 0;
    while (i < arr.length) {
        if (arr[i] == i) {
            i++;
        } else {
            // Swap arr[i] to its correct position
            int correct = arr[i];
            int temp = arr[correct];
            arr[correct] = arr[i];
            arr[i] = temp;
        }
    }
}
```

---

## 6. Sort Colors (Dutch National Flag Problem)

```
  Special case: When k = 3 (only 3 distinct values)
  
  Array: [2, 0, 2, 1, 1, 0]
  
  ┌──────────────────────────────────────────────────────────┐
  │  Method 1: Counting Sort                                │
  │  Count 0s, 1s, 2s → write back                         │
  │  count[0]=2, count[1]=2, count[2]=2                     │
  │  Result: [0, 0, 1, 1, 2, 2]                            │
  │                                                          │
  │  Method 2: Three-Way Partition (one pass, in-place)     │
  │  Three pointers: low, mid, high                         │
  │  - 0s go to [0..low-1]                                  │
  │  - 1s go to [low..mid-1]                                │
  │  - 2s go to [high+1..n-1]                               │
  └──────────────────────────────────────────────────────────┘
```

```java
// Dutch National Flag - O(n) time, O(1) space
public static void sortColors(int[] arr) {
    int low = 0, mid = 0, high = arr.length - 1;
    
    while (mid <= high) {
        if (arr[mid] == 0) {
            swap(arr, low++, mid++);
        } else if (arr[mid] == 1) {
            mid++;
        } else { // arr[mid] == 2
            swap(arr, mid, high--);
        }
    }
}
```

---

## Comparison of All Variations

```
  ┌────────────────────────────┬───────────┬──────────┬─────────┐
  │  Variation                │ Time      │ Space    │ Stable  │
  ├────────────────────────────┼───────────┼──────────┼─────────┤
  │  Standard (prefix sum)    │ O(n + k)  │ O(n + k) │ Yes     │
  │  Frequency count          │ O(n + k)  │ O(k)     │ No      │
  │  Character sort           │ O(n + 26) │ O(26)    │ No      │
  │  By digit (Radix helper)  │ O(n + 10) │ O(n + 10)│ Yes     │
  │  In-place (0 to n-1)      │ O(n)      │ O(1)     │ No      │
  │  Dutch National Flag      │ O(n)      │ O(1)     │ No      │
  └────────────────────────────┴───────────┴──────────┴─────────┘
```

---

## Summary Table

| Variation | Best For | Key Feature |
|---|---|---|
| Standard | General sorting with stability | Prefix sum + right-to-left |
| Frequency count | Simple integer sorting | No output array needed |
| Character sort | String problems | Fixed k=26 or k=256 |
| By digit | Radix Sort subroutine | Stable, k=10 |
| In-place | Permutation values (0 to n-1) | O(1) space |
| Dutch National Flag | 3-value problems | One pass, O(1) space |

---

## Quick Revision Questions

1. **When would you use frequency sort over standard counting sort?**
2. **How does counting sort by digit enable Radix Sort?**
3. **What constraint must hold for in-place counting sort to work?**
4. **How does Dutch National Flag differ from counting sort?**
5. **Which variation(s) are stable?**

---

[← Previous: Limitations & When to Use](05-limitations-and-when-to-use.md) | [Next: Unit 9 - Radix Sort →](../09-Radix-Sort/01-algorithm-description.md)
