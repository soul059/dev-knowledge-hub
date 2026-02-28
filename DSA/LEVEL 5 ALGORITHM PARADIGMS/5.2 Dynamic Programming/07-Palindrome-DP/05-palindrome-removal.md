# Chapter 5: Palindrome Removal

## 📋 Overview
**Palindrome Removal** (LeetCode #1246) asks for the minimum number of moves to remove all characters from an array, where each move removes a palindromic subarray. This is an **interval DP** problem that combines palindrome detection with optimal partitioning.

---

## 🧠 Core Concept

```
arr = [1, 3, 4, 1, 5]

Move 1: Remove [1,3,4,1] — wait, is it palindrome? 
  [1,3,4,1] → 1≠1 wait, 1==1 but 3≠4 → NOT palindrome

Move 1: Remove [5] → arr = [1,3,4,1]
Move 2: Remove [4] → arr = [1,3,1]  
Move 3: Remove [1,3,1] → palindrome! arr = []
Total: 3 moves

Better:
Move 1: Remove [1,3,1] (indices 0,1,3 — NO! must be subarray = contiguous)
  [1,3,4,1] → can only remove contiguous sequences

arr = [1, 3, 4, 1, 5]
Move 1: Remove [4] (index 2) → [1,3,1,5]
Move 2: Remove [1,3,1] → [5]
Move 3: Remove [5] → []
Total: 3 moves

Actually minimum might be different. Let's think with DP.
```

---

## 🔨 Interval DP Formulation

```
State: dp[i][j] = minimum moves to remove all elements in arr[i..j]

Base cases:
  dp[i][i] = 1                          // single element: 1 move
  dp[i][i-1] = 0                        // empty range: 0 moves

Recurrence:
  // Option 1: Remove arr[i] alone, then handle rest
  dp[i][j] = dp[i+1][j] + 1

  // Option 2: If arr[i] == arr[k] for some k in [i+1..j],
  // pair them up and potentially remove together
  for k = i+1 to j:
      if arr[i] == arr[k]:
          dp[i][j] = min(dp[i][j], dp[i+1][k-1] + dp[k][j])
          // dp[i+1][k-1]: remove middle elements
          // dp[k][j]: handle arr[k..j] where arr[k] is already 
          //           "merged" with arr[i]

  Wait, this needs more thought. When arr[i]==arr[k],
  we can think of it as: arr[i] can be removed together with arr[k]
  in the same palindrome removal. So:
  dp[i][j] = min(dp[i][j], dp[i+1][k-1] + dp[k][j])
  Note: dp[k][j] benefits because arr[k] "extends" the palindrome
        being built from the right partition.
  
  Special: if arr[i]==arr[i+1], dp[i][j] = min(dp[i][j], dp[i+2][j]+1)
           (remove pair in one move if adjacent and equal)
```

---

## 🔨 Cleaner Formulation

```
function minimumMoves(arr):
    n = len(arr)
    dp = 2D array n × n, filled with 0
    
    // Base: single elements
    for i = 0 to n-1: dp[i][i] = 1
    
    // Fill by increasing length
    for length = 2 to n:
        for i = 0 to n-length:
            j = i + length - 1
            
            // Worst case: remove each element individually
            dp[i][j] = dp[i+1][j] + 1
            
            // Try pairing arr[i] with some arr[k] where arr[i]==arr[k]
            for k = i+1 to j:
                if arr[i] == arr[k]:
                    // Elements between i and k: dp[i+1][k-1]
                    // Elements k to j (k absorbed into i's palindrome): dp[k+1][j] + ?
                    // Actually, merge the palindrome containing i with segment k..j
                    left = dp[i+1][k-1] if k > i+1 else 0
                    dp[i][j] = min(dp[i][j], left + dp[k][j])
            
            // Also: if arr[i]==arr[j], they form outer pair
            if arr[i] == arr[j]:
                dp[i][j] = min(dp[i][j], max(1, dp[i+1][j-1]))
    
    return dp[0][n-1]
```

---

## 🔬 Trace: arr = [1, 2, 1]

```
n=3

Base: dp[0][0]=1, dp[1][1]=1, dp[2][2]=1

Length 2:
  dp[0][1]: arr[0]=1≠arr[1]=2
    dp[0][1] = dp[1][1]+1 = 2  (remove individually)
  
  dp[1][2]: arr[1]=2≠arr[2]=1
    dp[1][2] = dp[2][2]+1 = 2

Length 3:
  dp[0][2]: start with dp[1][2]+1 = 3
    k=1: arr[0]=1≠arr[1]=2 → skip
    k=2: arr[0]=1==arr[2]=1 → 
      left = dp[1][1] = 1  (middle element)
      dp[0][2] = min(3, 1 + dp[2][2]) = min(3, 1+1) = 2
    
    Also: arr[0]==arr[2], dp[0][2] = min(2, max(1, dp[1][1])) = min(2,1) = 1

Answer: dp[0][2] = 1 (remove [1,2,1] as palindrome in one move) ✓
```

---

## 🔬 Trace: arr = [1, 3, 4, 1, 5]

```
Key computations:

dp[0][0]=1, dp[1][1]=1, dp[2][2]=1, dp[3][3]=1, dp[4][4]=1

dp[0][1]=2, dp[1][2]=2, dp[2][3]=2, dp[3][4]=2

dp[0][2]: 1≠4 anywhere → try splits + default
  = dp[1][2]+1 = 3
dp[1][3]: 3≠1 → dp[2][3]+1 = 3
dp[2][4]: 4≠5 → dp[3][4]+1 = 3

dp[0][3]: arr[0]=1==arr[3]=1
  Default: dp[1][3]+1 = 4
  k=3: arr[0]==arr[3] → left=dp[1][2]=2, dp[3][3]=1 → 2+1=3
  arr[0]==arr[3]: max(1, dp[1][2]) = max(1,2) = 2
  dp[0][3] = 2

dp[1][4]: all different → dp[2][4]+1 = 4, try splits...
  dp[1][4] = 4? Let's see: best is individual = 4

dp[0][4]:
  Default: dp[1][4]+1 = 5
  k=3: arr[0]=1==arr[3]=1 → dp[1][2]+dp[3][4] = 2+2 = 4
  Try splits at each point:
  dp[0][4] = min(5, 4, ...) 
  Split: dp[0][0]+dp[1][4]=1+4=5, dp[0][1]+dp[2][4]=2+3=5,
         dp[0][2]+dp[3][4]=3+2=5, dp[0][3]+dp[4][4]=2+1=3

  dp[0][4] = 3

Answer: 3
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[i][j] = min moves to clear arr[i..j] |
| **Key Transition** | Pair equal elements to share palindrome removal |
| **Equal Ends** | dp[i][j] = max(1, dp[i+1][j-1]) when arr[i]==arr[j] |
| **Split** | dp[i][j] = min(dp[i][k] + dp[k+1][j]) |
| **Complexity** | O(n³) time, O(n²) space |
| **Pattern** | Interval DP with palindrome constraints |

---

## ❓ Quick Revision Questions

1. **Why is this an interval DP problem?**
2. **What is the significance of pairing equal elements?**
3. **What are the base cases?**
4. **When arr[i]==arr[j], why is the answer max(1, dp[i+1][j-1])?**
5. **What is the time complexity and why?**
6. **Trace the algorithm for arr = [1, 1].**

---

[← Previous: Count Palindromic Subsequences](04-count-palindromic-subsequences.md) | [Next Unit: Knapsack Patterns →](../08-Knapsack-Patterns/01-01-knapsack.md)

[← Back to README](../README.md)
