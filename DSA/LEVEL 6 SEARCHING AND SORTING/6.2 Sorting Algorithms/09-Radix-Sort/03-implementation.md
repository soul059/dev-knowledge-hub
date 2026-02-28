# Unit 9: Radix Sort — Implementation

[← Previous: LSD vs MSD](02-lsd-vs-msd.md) | [Next: Complexity Analysis →](04-complexity-analysis.md)

---

## Overview

This chapter provides complete implementations of LSD Radix Sort (most common) and MSD Radix Sort in Java, Python, and C++.

---

## LSD Radix Sort — Java

```java
public class RadixSort {
    
    public static void radixSort(int[] arr) {
        if (arr.length == 0) return;
        
        // Find maximum to determine number of digits
        int max = arr[0];
        for (int num : arr) {
            if (num > max) max = num;
        }
        
        // Sort by each digit position
        // exp: 1 → ones, 10 → tens, 100 → hundreds...
        for (int exp = 1; max / exp > 0; exp *= 10) {
            countingSortByDigit(arr, exp);
        }
    }
    
    private static void countingSortByDigit(int[] arr, int exp) {
        int n = arr.length;
        int[] output = new int[n];
        int[] count = new int[10]; // digits 0-9
        
        // Count occurrences of each digit
        for (int num : arr) {
            int digit = (num / exp) % 10;
            count[digit]++;
        }
        
        // Prefix sum
        for (int i = 1; i < 10; i++) {
            count[i] += count[i - 1];
        }
        
        // Build output (right-to-left for stability!)
        for (int i = n - 1; i >= 0; i--) {
            int digit = (arr[i] / exp) % 10;
            output[count[digit] - 1] = arr[i];
            count[digit]--;
        }
        
        // Copy back
        System.arraycopy(output, 0, arr, 0, n);
    }
    
    public static void main(String[] args) {
        int[] arr = {170, 45, 75, 90, 802, 24, 2, 66};
        radixSort(arr);
        System.out.println(java.util.Arrays.toString(arr));
        // Output: [2, 24, 45, 66, 75, 90, 170, 802]
    }
}
```

---

## LSD Radix Sort — Python

```python
def radix_sort(arr):
    if not arr:
        return
    
    max_val = max(arr)
    exp = 1
    
    while max_val // exp > 0:
        counting_sort_by_digit(arr, exp)
        exp *= 10

def counting_sort_by_digit(arr, exp):
    n = len(arr)
    output = [0] * n
    count = [0] * 10
    
    # Count digits
    for num in arr:
        digit = (num // exp) % 10
        count[digit] += 1
    
    # Prefix sum
    for i in range(1, 10):
        count[i] += count[i - 1]
    
    # Build output (right-to-left)
    for i in range(n - 1, -1, -1):
        digit = (arr[i] // exp) % 10
        output[count[digit] - 1] = arr[i]
        count[digit] -= 1
    
    arr[:] = output

# Test
arr = [170, 45, 75, 90, 802, 24, 2, 66]
radix_sort(arr)
print(arr)  # [2, 24, 45, 66, 75, 90, 170, 802]
```

---

## LSD Radix Sort — C++

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

void countingSortByDigit(vector<int>& arr, int exp) {
    int n = arr.size();
    vector<int> output(n);
    int count[10] = {0};
    
    for (int num : arr)
        count[(num / exp) % 10]++;
    
    for (int i = 1; i < 10; i++)
        count[i] += count[i - 1];
    
    for (int i = n - 1; i >= 0; i--) {
        int digit = (arr[i] / exp) % 10;
        output[count[digit] - 1] = arr[i];
        count[digit]--;
    }
    
    arr = output;
}

void radixSort(vector<int>& arr) {
    int maxVal = *max_element(arr.begin(), arr.end());
    
    for (int exp = 1; maxVal / exp > 0; exp *= 10)
        countingSortByDigit(arr, exp);
}

int main() {
    vector<int> arr = {170, 45, 75, 90, 802, 24, 2, 66};
    radixSort(arr);
    for (int x : arr) cout << x << " ";
    // Output: 2 24 45 66 75 90 170 802
    return 0;
}
```

---

## Handling Negative Numbers

```java
public static void radixSortWithNegatives(int[] arr) {
    // Separate negatives and positives
    List<Integer> negatives = new ArrayList<>();
    List<Integer> positives = new ArrayList<>();
    
    for (int num : arr) {
        if (num < 0) negatives.add(-num);  // Store as positive
        else positives.add(num);
    }
    
    // Sort both parts
    int[] negArr = negatives.stream().mapToInt(i -> i).toArray();
    int[] posArr = positives.stream().mapToInt(i -> i).toArray();
    
    if (negArr.length > 0) radixSort(negArr);
    if (posArr.length > 0) radixSort(posArr);
    
    // Merge: negatives (reversed) then positives
    int idx = 0;
    for (int i = negArr.length - 1; i >= 0; i--)
        arr[idx++] = -negArr[i];
    for (int num : posArr)
        arr[idx++] = num;
}

// Example: [-5, 3, -1, 8, -3] → [-5, -3, -1, 3, 8]
```

---

## Base-256 Radix Sort (Byte-Level)

```
  Instead of base-10 (digit by digit),
  use base-256 (byte by byte):
  
  32-bit integer = 4 bytes → only 4 passes!
  
  vs base-10: up to 10 passes for 32-bit ints
  
  ┌──────────────────────────────────────────────────────┐
  │  Base 10:  d = 10 digits, k = 10  → 10 passes      │
  │  Base 256: d = 4 bytes,  k = 256 → 4 passes        │
  │  Base 65536: d = 2,      k = 65536 → 2 passes      │
  │                                                      │
  │  Larger base = fewer passes but larger count array  │
  │  Sweet spot: base 256 for 32-bit integers           │
  └──────────────────────────────────────────────────────┘
```

```java
// Byte-level radix sort (4 passes for 32-bit int)
public static void radixSort256(int[] arr) {
    int n = arr.length;
    int[] output = new int[n];
    
    for (int shift = 0; shift < 32; shift += 8) {
        int[] count = new int[256];
        
        for (int num : arr)
            count[(num >> shift) & 0xFF]++;
        
        for (int i = 1; i < 256; i++)
            count[i] += count[i - 1];
        
        for (int i = n - 1; i >= 0; i--) {
            int bucket = (arr[i] >> shift) & 0xFF;
            output[count[bucket] - 1] = arr[i];
            count[bucket]--;
        }
        
        System.arraycopy(output, 0, arr, 0, n);
    }
}
```

---

## MSD Radix Sort Implementation

```java
public class MSDRadixSort {
    
    public static void msdRadixSort(int[] arr) {
        if (arr.length <= 1) return;
        
        int max = Arrays.stream(arr).max().orElse(0);
        int maxDigits = String.valueOf(max).length();
        int exp = (int) Math.pow(10, maxDigits - 1);
        
        msdSort(arr, 0, arr.length - 1, exp);
    }
    
    private static void msdSort(int[] arr, int lo, int hi, int exp) {
        if (lo >= hi || exp == 0) return;
        
        // Count sort by current digit
        int[] count = new int[12]; // 0-9 + boundaries
        int[] temp = new int[hi - lo + 1];
        
        // Count
        for (int i = lo; i <= hi; i++) {
            int digit = (arr[i] / exp) % 10;
            count[digit + 1]++;
        }
        
        // Prefix sum
        for (int i = 1; i < 12; i++)
            count[i] += count[i - 1];
        
        // Distribute
        for (int i = lo; i <= hi; i++) {
            int digit = (arr[i] / exp) % 10;
            temp[count[digit]++] = arr[i];
        }
        
        // Copy back
        System.arraycopy(temp, 0, arr, lo, hi - lo + 1);
        
        // Recursively sort each digit group
        // Reset count for boundary calculation
        int[] boundaries = new int[11];
        for (int i = lo; i <= hi; i++) {
            int digit = (arr[i] / exp) % 10;
            boundaries[digit + 1] = i + 1;
        }
        
        // Recurse on each bucket
        for (int d = 0; d < 10; d++) {
            int start = lo + (d > 0 ? count[d - 1] : 0);
            int end = lo + count[d] - 1;
            if (start < end) {
                msdSort(arr, start, end, exp / 10);
            }
        }
    }
}
```

---

## String Radix Sort (MSD)

```java
// MSD Radix Sort for strings
public static void msdStringSort(String[] arr, int lo, int hi, int d) {
    if (lo >= hi) return;
    
    int R = 256;  // ASCII
    int[] count = new int[R + 2];
    String[] aux = new String[hi - lo + 1];
    
    // Count
    for (int i = lo; i <= hi; i++) {
        int c = charAt(arr[i], d);
        count[c + 2]++;
    }
    
    // Prefix sum
    for (int r = 0; r < R + 1; r++)
        count[r + 1] += count[r];
    
    // Distribute
    for (int i = lo; i <= hi; i++) {
        int c = charAt(arr[i], d);
        aux[count[c + 1]++] = arr[i];
    }
    
    // Copy back
    for (int i = lo; i <= hi; i++)
        arr[i] = aux[i - lo];
    
    // Recurse on each character group
    for (int r = 0; r < R; r++) {
        msdStringSort(arr, lo + count[r], lo + count[r + 1] - 1, d + 1);
    }
}

private static int charAt(String s, int d) {
    return d < s.length() ? s.charAt(d) : -1;
}
```

---

## Summary Table

| Implementation | Base | Passes | Use Case |
|---|---|---|---|
| LSD base-10 | 10 | d (max digits) | Simple, general integers |
| LSD base-256 | 256 | 4 (for 32-bit) | Performance-critical |
| MSD base-10 | 10 | Up to d | Variable-length numbers |
| MSD strings | 256 | Up to max length | String sorting |
| With negatives | any | d + O(n) | Signed integers |

---

## Quick Revision Questions

1. **Why does the counting sort subroutine use right-to-left traversal?**
2. **How does base-256 reduce the number of passes?**
3. **How would you handle negative numbers in radix sort?**
4. **What's the trade-off of using a larger base?**
5. **Why is MSD more natural for strings than LSD?**

---

[← Previous: LSD vs MSD](02-lsd-vs-msd.md) | [Next: Complexity Analysis →](04-complexity-analysis.md)
