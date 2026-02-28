# Unit 9: Radix Sort — String Sorting

[← Previous: Complexity Analysis](04-complexity-analysis.md) | [Next: Applications →](06-applications.md)

---

## Overview

Radix Sort is exceptionally powerful for sorting strings — especially when strings are of fixed length or share common prefixes. MSD Radix Sort naturally handles variable-length strings and produces **lexicographic ordering**.

---

## Fixed-Length String Sorting (LSD)

```
  Sort 3-character strings:
  ["cat", "bat", "cab", "bad", "bag"]
  
  Pass 1 (rightmost char, position 2):
    Sort by 3rd character:
    'd': bad          
    'b': cab
    'g': bag
    't': cat, bat
    
    After: [bad, cab, bag, cat, bat]
  
  Pass 2 (position 1):
    Sort by 2nd character:
    'a': bad, bag, bat, cat, cab  
    Wait — let me be more careful.
    
    Array after pass 1: [bad, cab, bag, cat, bat]
    2nd chars:           a    a    a    a    a
    All same! Stability preserves order.
    After: [bad, cab, bag, cat, bat]
  
  Pass 3 (position 0, leftmost):
    Sort by 1st character:
    'b': bad, bag, bat
    'c': cab, cat
    
    After: [bad, bag, bat, cab, cat]  ✓ Lexicographic!
```

---

## Variable-Length String Sorting (MSD)

```
  Strings: ["she", "sells", "sea", "shells", "shore"]
  
  MSD processes from LEFT to RIGHT:
  
  Pass 1 (char 0):
    's': [she, sells, sea, shells, shore]
    → All start with 's', recurse on all.
  
  Pass 2 (char 1):
    'e': [sells, sea]    → recurse
    'h': [she, shells, shore]  → recurse
  
  Recurse on 'e' group (char 2):
    'a': [sea]
    'l': [sells]
    After: [sea, sells]
  
  Recurse on 'h' group (char 2):
    'e': [she, shells]   → recurse
    'o': [shore]
  
  Recurse on 'he' group (char 3):
    '\0': [she]      (she has no char 3 → comes first)
    'l':  [shells]
    After: [she, shells]
  
  Final: [sea, sells, she, shells, shore]  ✓
```

---

## Handling End-of-String

```
  When strings have DIFFERENT lengths:
  
  ┌──────────────────────────────────────────────────────┐
  │  "app" vs "apple"                                   │
  │                                                      │
  │  At position 3:                                     │
  │  "app" has NO character → treated as '\0' (lowest)  │
  │  "apple" has 'l'                                    │
  │                                                      │
  │  '\0' < 'l' → "app" < "apple"  ✓                  │
  │                                                      │
  │  Convention: charAt(s, d) returns -1 if d ≥ length  │
  │  -1 gets its own bucket (bucket 0) → shortest first │
  └──────────────────────────────────────────────────────┘
```

---

## MSD String Sort Implementation

```java
public class MSDStringSort {
    private static final int R = 256;          // ASCII
    private static final int CUTOFF = 15;      // Switch to insertion
    private static String[] aux;
    
    public static void sort(String[] arr) {
        int n = arr.length;
        aux = new String[n];
        sort(arr, 0, n - 1, 0);
    }
    
    private static int charAt(String s, int d) {
        if (d < s.length()) return s.charAt(d);
        return -1;  // End of string
    }
    
    private static void sort(String[] arr, int lo, int hi, int d) {
        if (hi <= lo) return;
        
        // Use insertion sort for small subarrays
        if (hi - lo <= CUTOFF) {
            insertionSort(arr, lo, hi, d);
            return;
        }
        
        // Count frequencies
        int[] count = new int[R + 2];
        for (int i = lo; i <= hi; i++) {
            int c = charAt(arr[i], d);
            count[c + 2]++;
        }
        
        // Transform counts to indices
        for (int r = 0; r < R + 1; r++) {
            count[r + 1] += count[r];
        }
        
        // Distribute
        for (int i = lo; i <= hi; i++) {
            int c = charAt(arr[i], d);
            aux[count[c + 1]++] = arr[i];
        }
        
        // Copy back
        for (int i = lo; i <= hi; i++) {
            arr[i] = aux[i - lo];
        }
        
        // Recursively sort each character group
        for (int r = 0; r < R; r++) {
            sort(arr, lo + count[r], lo + count[r + 1] - 1, d + 1);
        }
    }
    
    private static void insertionSort(String[] arr, int lo, int hi, int d) {
        for (int i = lo + 1; i <= hi; i++) {
            String temp = arr[i];
            int j = i;
            while (j > lo && arr[j - 1].substring(d).compareTo(temp.substring(d)) > 0) {
                arr[j] = arr[j - 1];
                j--;
            }
            arr[j] = temp;
        }
    }
    
    public static void main(String[] args) {
        String[] arr = {"she", "sells", "sea", "shells", "shore", "by"};
        sort(arr);
        System.out.println(java.util.Arrays.toString(arr));
        // [by, sea, sells, she, shells, shore]
    }
}
```

---

## Three-Way String Quick Sort

```
  Combines Quick Sort partitioning with MSD character processing.
  Uses LESS space than MSD Radix Sort (no count array).
  
  Algorithm:
  1. Pick pivot character (first string's d-th character)
  2. Three-way partition:
     - Less than pivot char → left
     - Equal to pivot char → middle
     - Greater than pivot char → right
  3. Recurse: sort left (same d), middle (d+1), right (same d)
```

```java
public static void sort(String[] arr, int lo, int hi, int d) {
    if (lo >= hi) return;
    
    int lt = lo, gt = hi, i = lo + 1;
    int v = charAt(arr[lo], d);  // pivot character
    
    while (i <= gt) {
        int t = charAt(arr[i], d);
        if (t < v) swap(arr, lt++, i++);
        else if (t > v) swap(arr, i, gt--);
        else i++;
    }
    
    // arr[lo..lt-1] < v = arr[lt..gt] < arr[gt+1..hi]
    sort(arr, lo, lt - 1, d);
    if (v >= 0) sort(arr, lt, gt, d + 1);  // equal group, next char
    sort(arr, gt + 1, hi, d);
}
```

```
  Example: ["she", "sells", "sea", "shells", "shore"]
  
  d=0, pivot='s' (from "she"):
    All start with 's' → all go to middle
    Recurse on all with d=1
  
  d=1, pivot='h' (from "she"):
    < 'h': "sells", "sea" (both have 'e')
    = 'h': "she", "shells", "shore"
    Recurse on each group
  
  Much more efficient than standard MSD when 
  many strings share long common prefixes!
```

---

## When to Use Which String Sort

```
  ┌──────────────────────────────────────────────────────────┐
  │  Fixed-length strings:                                  │
  │    → LSD Radix Sort (simple, efficient)                │
  │                                                          │
  │  Variable-length, diverse prefixes:                     │
  │    → MSD Radix Sort                                    │
  │                                                          │
  │  Variable-length, many shared prefixes:                 │
  │    → Three-way String Quick Sort                       │
  │                                                          │
  │  Small arrays or mixed data:                            │
  │    → Standard comparison sort with compareTo()         │
  └──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Algorithm | Time | Space | Best For |
|---|---|---|---|
| LSD String Sort | O(W × n) | O(n + R) | Fixed-length strings |
| MSD String Sort | O(W × n) worst | O(n + W×R) | Variable-length, diverse |
| 3-Way String QS | O(W × n log n) worst | O(W + log n) | Shared prefixes |

W = string length, R = alphabet size (256 for ASCII)

---

## Quick Revision Questions

1. **How does MSD Radix Sort handle strings shorter than others?**
2. **Why use insertion sort for small sub-arrays in MSD?**
3. **What advantage does Three-Way String Quick Sort have over MSD?**
4. **When would LSD be better than MSD for strings?**
5. **What does the -1 return from charAt represent?**

---

[← Previous: Complexity Analysis](04-complexity-analysis.md) | [Next: Applications →](06-applications.md)
