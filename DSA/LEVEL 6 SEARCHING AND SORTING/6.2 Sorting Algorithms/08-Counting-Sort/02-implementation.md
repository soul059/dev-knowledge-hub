# Unit 8: Counting Sort — Implementation

[← Previous: Algorithm Description](01-algorithm-description.md) | [Next: Stability Deep Dive →](03-stability-deep-dive.md)

---

## Overview

This chapter provides complete implementations of Counting Sort in Java, Python, and C++, along with variations for handling negative numbers, characters, and objects.

---

## Java Implementation (Standard)

```java
public class CountingSort {
    
    public static void countingSort(int[] arr) {
        if (arr.length == 0) return;
        
        // Find range
        int max = arr[0], min = arr[0];
        for (int num : arr) {
            if (num > max) max = num;
            if (num < min) min = num;
        }
        
        int range = max - min + 1;
        int[] count = new int[range];
        int[] output = new int[arr.length];
        
        // Step 1: Count occurrences (offset by min)
        for (int num : arr) {
            count[num - min]++;
        }
        
        // Step 2: Prefix sum
        for (int i = 1; i < range; i++) {
            count[i] += count[i - 1];
        }
        
        // Step 3: Build output (right-to-left for stability)
        for (int i = arr.length - 1; i >= 0; i--) {
            output[count[arr[i] - min] - 1] = arr[i];
            count[arr[i] - min]--;
        }
        
        // Step 4: Copy back
        System.arraycopy(output, 0, arr, 0, arr.length);
    }
    
    public static void main(String[] args) {
        int[] arr = {4, 2, 2, 8, 3, 3, 1};
        countingSort(arr);
        System.out.println(java.util.Arrays.toString(arr));
        // Output: [1, 2, 2, 3, 3, 4, 8]
    }
}
```

---

## Python Implementation

```python
def counting_sort(arr):
    if not arr:
        return arr
    
    min_val = min(arr)
    max_val = max(arr)
    range_val = max_val - min_val + 1
    
    count = [0] * range_val
    output = [0] * len(arr)
    
    # Count occurrences
    for num in arr:
        count[num - min_val] += 1
    
    # Prefix sum
    for i in range(1, range_val):
        count[i] += count[i - 1]
    
    # Build output (right-to-left for stability)
    for i in range(len(arr) - 1, -1, -1):
        output[count[arr[i] - min_val] - 1] = arr[i]
        count[arr[i] - min_val] -= 1
    
    # Copy back
    arr[:] = output

# Test
arr = [4, 2, 2, 8, 3, 3, 1]
counting_sort(arr)
print(arr)  # [1, 2, 2, 3, 3, 4, 8]
```

---

## C++ Implementation

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

void countingSort(vector<int>& arr) {
    if (arr.empty()) return;
    
    int minVal = *min_element(arr.begin(), arr.end());
    int maxVal = *max_element(arr.begin(), arr.end());
    int range = maxVal - minVal + 1;
    
    vector<int> count(range, 0);
    vector<int> output(arr.size());
    
    for (int num : arr)
        count[num - minVal]++;
    
    for (int i = 1; i < range; i++)
        count[i] += count[i - 1];
    
    for (int i = arr.size() - 1; i >= 0; i--) {
        output[count[arr[i] - minVal] - 1] = arr[i];
        count[arr[i] - minVal]--;
    }
    
    arr = output;
}

int main() {
    vector<int> arr = {4, 2, 2, 8, 3, 3, 1};
    countingSort(arr);
    for (int x : arr) cout << x << " ";
    // Output: 1 2 2 3 3 4 8
    return 0;
}
```

---

## Simplified Version (Non-Stable)

```java
// Simpler but NOT stable — fine for plain integers
public static void countingSortSimple(int[] arr) {
    int max = Arrays.stream(arr).max().orElse(0);
    int min = Arrays.stream(arr).min().orElse(0);
    int range = max - min + 1;
    
    int[] count = new int[range];
    
    // Count
    for (int num : arr)
        count[num - min]++;
    
    // Write back directly
    int idx = 0;
    for (int i = 0; i < range; i++) {
        while (count[i] > 0) {
            arr[idx++] = i + min;
            count[i]--;
        }
    }
}
```

```
  Simple version trace:
  
  Array: [4, 2, 2, 8, 3]
  Count: [0, 1, 2, 2, 1, 0, 0, 0, 1]   (offset by min=2)
          2  3  4  5  6  7  8  9  10     ← actual values
  Actually: min=2, count indices 0-6 map to values 2-8
  
  count[0]=1 → write 2         arr=[2,...]
  count[1]=0 → skip
  count[2]=2 → write 4,4       Wait — that's wrong!
  
  Let me redo: arr=[4,2,2,8,3], min=2, max=8, range=7
  count[0]=2 (val 2), count[1]=1 (val 3), count[2]=1 (val 4),
  count[3-5]=0, count[6]=1 (val 8)
  
  Write: 2,2,3,4,8 ✓
  
  Simple but loses stability information.
```

---

## Handling Negative Numbers

```
  Array: [3, -1, -5, 0, 2, -5, 3]
  
  min = -5, max = 3, range = 3 - (-5) + 1 = 9
  
  Offset: index = value - min
  -5 → 0, -4 → 1, -3 → 2, -2 → 3, -1 → 4, 0 → 5, 1 → 6, 2 → 7, 3 → 8
  
  Count: [2, 0, 0, 0, 1, 1, 0, 1, 2]
          -5 -4 -3 -2 -1  0  1  2  3   ← actual values
  
  Reverse mapping: value = index + min
  
  Sorted: [-5, -5, -1, 0, 2, 3, 3]  ✓
```

---

## Sorting Characters

```java
// Sort a string using counting sort
public static String countingSortChars(String s) {
    int[] count = new int[256];  // ASCII range
    
    for (char c : s.toCharArray())
        count[c]++;
    
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < 256; i++) {
        while (count[i] > 0) {
            sb.append((char) i);
            count[i]--;
        }
    }
    return sb.toString();
}

// Example: "programming" → "aggimmnoprr"
```

---

## Sorting Objects by Key (Stable)

```java
// Sort students by grade (0-100) while preserving name order
public static void sortByGrade(Student[] students) {
    int k = 100;  // max grade
    int[] count = new int[k + 1];
    Student[] output = new Student[students.length];
    
    // Count grades
    for (Student s : students)
        count[s.grade]++;
    
    // Prefix sum
    for (int i = 1; i <= k; i++)
        count[i] += count[i - 1];
    
    // Build output (right-to-left for stability)
    for (int i = students.length - 1; i >= 0; i--) {
        int grade = students[i].grade;
        output[count[grade] - 1] = students[i];
        count[grade]--;
    }
    
    System.arraycopy(output, 0, students, 0, students.length);
}
```

```
  Example:
  Input:  [(Alice,85), (Bob,72), (Carol,85), (Dave,72)]
  
  Count[72]=2, Count[85]=2
  Prefix: Count[72]=2, Count[85]=4
  
  Process right-to-left:
  Dave(72)  → pos 1   Count[72]=1
  Carol(85) → pos 3   Count[85]=3
  Bob(72)   → pos 0   Count[72]=0
  Alice(85) → pos 2   Count[85]=2
  
  Output: [Bob(72), Dave(72), Alice(85), Carol(85)]
  
  Stability preserved: Bob before Dave (both 72) ✓
                       Alice before Carol (both 85) ✓
```

---

## Summary Table

| Variant | Use Case | Stable? | Space |
|---------|----------|---------|-------|
| Standard (prefix sum) | General integers | Yes | O(n + k) |
| Simple (no prefix sum) | Plain integers only | No | O(k) |
| With offset (min subtraction) | Negative numbers | Yes | O(n + k) |
| Character sort | Strings/chars | Yes/No | O(256) |
| Object sort by key | Multi-field data | Yes | O(n + k) |

---

## Quick Revision Questions

1. **Why do we subtract `min` from each element?**
2. **What's the difference between the stable and simple versions?**
3. **How would you sort lowercase letters using counting sort?**
4. **Why is `System.arraycopy` used at the end?**
5. **What happens if the range k is much larger than n?**

---

[← Previous: Algorithm Description](01-algorithm-description.md) | [Next: Stability Deep Dive →](03-stability-deep-dive.md)
