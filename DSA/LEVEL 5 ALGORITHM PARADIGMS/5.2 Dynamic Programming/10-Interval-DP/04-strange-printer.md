# Chapter 4: Strange Printer

## 📋 Overview
A printer can print a sequence of **the same character** at once from position `i` to `j`. Each turn uses a different character (overlapping is allowed — later prints overwrite earlier ones). Find the **minimum number of turns** to print a given string.

---

## 🧠 Core Concept

```
s = "aba"

Turn 1: print "aaa"  → a a a
Turn 2: print "b" at position 1 → a b a
Total: 2 turns

s = "aaabbb"

Turn 1: print "aaa___" → a a a
Turn 2: print "___bbb" → a a a b b b
Total: 2 turns (each segment of same chars → 1 turn)

s = "abab"

Turn 1: print "aaaa" → a a a a
Turn 2: print "b" at pos 1 → a b a a  
Turn 3: print "b" at pos 3 → a b a b
Total: 3 turns

Or: print "a" pos 0-2, "b" pos 1, "b" pos 3 = still 3
```

---

## 🔨 Interval DP

```
dp[i][j] = min turns to print s[i..j]

Base: dp[i][i] = 1 (single character → 1 turn)

Recurrence:
  Option 1: Print s[i] and then handle rest
    dp[i][j] = dp[i+1][j] + 1   (s[i] alone needs 1 more turn)

  Option 2: If s[k] == s[i] for some k in (i+1..j):
    We can print s[i] and s[k] in the SAME turn!
    When we print s[i..k], the character at positions i..k is s[i].
    Then we overwrite i+1..k-1 with proper characters.
    
    dp[i][j] = min(dp[i][k-1] + dp[k+1][j]) for k where s[k]==s[i]
    
    Merged: dp[i][j] = dp[i][k-1] + dp[k][j]
    where dp[i][k-1] handles i..k-1 and dp[k][j] handles k..j
    but s[i]=s[k] so they share the same initial print.

Simplified recurrence:
  dp[i][j] = dp[i][j-1] + 1    // print s[j] separately
  
  for k = i to j-1:
      if s[k] == s[j]:
          dp[i][j] = min(dp[i][j], dp[i][k] + dp[k+1][j-1])
          // s[k] and s[j] share same turn
          // handle i..k together, k+1..j-1 separately
```

---

## 💻 Pseudocode

```
function strangePrinter(s):
    // Remove consecutive duplicates (optimization)
    s = remove_consecutive_dupes(s)
    n = len(s)
    dp = 2D array [n][n]
    
    // Base
    for i = 0 to n-1:
        dp[i][i] = 1
    
    // Fill by length
    for length = 2 to n:
        for i = 0 to n-length:
            j = i + length - 1
            dp[i][j] = dp[i][j-1] + 1   // print s[j] alone
            
            for k = i to j-1:
                if s[k] == s[j]:
                    left = dp[i][k]
                    right = if k+1 <= j-1 then dp[k+1][j-1] else 0
                    dp[i][j] = min(dp[i][j], left + right)
    
    return dp[0][n-1]
```

---

## 🔬 Trace: s = "aba"

```
After removing consecutive dupes: "aba" (no change)
n = 3

Base: dp[0][0]=1, dp[1][1]=1, dp[2][2]=1

Length 2:
  dp[0][1] (s="ab"):
    dp[0][1] = dp[0][0] + 1 = 2   (print 'b' separately)
    k=0: s[0]='a' ≠ s[1]='b', skip
    dp[0][1] = 2

  dp[1][2] (s="ba"):
    dp[1][2] = dp[1][1] + 1 = 2
    k=1: s[1]='b' ≠ s[2]='a', skip
    dp[1][2] = 2

Length 3:
  dp[0][2] (s="aba"):
    dp[0][2] = dp[0][1] + 1 = 3   (print 'a' at pos 2 separately)
    k=0: s[0]='a' == s[2]='a' ✓
      left = dp[0][0] = 1
      right = dp[1][1] = 1 (k+1=1, j-1=1)
      dp[0][2] = min(3, 1+1) = 2
    k=1: s[1]='b' ≠ s[2]='a', skip
    dp[0][2] = 2

Answer: 2 ✓ (print "aaa" then overwrite pos 1 with "b")
```

---

## 🔑 Intuition: Why Matching Characters Help

```
s = "abcba"

s[0]='a' and s[4]='a' match.
When printing 'a' at position 0, we can EXTEND that print 
to also cover position 4.

Print "aaaaa" in turn 1 → covers positions 0 and 4 for free.
Then handle "bcb" in the middle.
  "bcb": s[0]='b' matches s[2]='b'
  Print "bbb" covers both, then handle 'c' = 1 turn.
  Total for "bcb": 2 turns

Total for "abcba": dp[0][0] + dp[1][3] = 1 + 2 = 3

Without optimization: each different char = 1 turn = 4 turns
With matching: 3 turns
```

---

## ⚡ Optimization: Remove Consecutive Duplicates

```
"aaabbbccc" can be treated as "abc" for printing purposes:
  Consecutive same characters cost no extra turns.
  They're all printed in one turn that already covers 
  the first occurrence.

This preprocessing can significantly reduce n.

function removeDupes(s):
    result = [s[0]]
    for i = 1 to len(s)-1:
        if s[i] != s[i-1]:
            result.append(s[i])
    return result
```

---

## 🌐 Related: Remove Boxes

```
A harder variant where boxes have colors and removing a 
group of same-colored boxes earns points.

dp[i][j][k] = max points from boxes[i..j] with k extra 
              boxes same as boxes[i] attached to the left.

This requires 3D state — significantly more complex!
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[i][j] = min turns to print s[i..j] |
| **Base** | dp[i][i] = 1 |
| **Recurrence** | dp[i][j] = min(dp[i][j-1]+1, dp[i][k]+dp[k+1][j-1]) where s[k]==s[j] |
| **Insight** | Matching chars can share a turn |
| **Optimization** | Remove consecutive duplicates first |
| **Complexity** | O(n³) time, O(n²) space |

---

## ❓ Quick Revision Questions

1. **Why can matching characters at positions i and j share a turn?**
2. **What is the "trivial" case dp[i][j] = dp[i][j-1] + 1?**
3. **How does removing consecutive duplicates help?**
4. **What is the base case and why?**
5. **How does this differ from standard interval DP problems like MCM?**
6. **What makes the "Remove Boxes" problem harder?**

---

[← Previous: Merge Stones](03-merge-stones.md) | [Next: Optimal BST →](05-optimal-bst.md)

[← Back to README](../README.md)
