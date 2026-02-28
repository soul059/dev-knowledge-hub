# Chapter 6: Decode Ways

## 📋 Overview
**Problem:** A message encoded with digits (A=1, B=2, ..., Z=26). Given a string of digits, count the number of ways to decode it.

Example: "226" → "BZ" (2,26), "VF" (22,6), "BBF" (2,2,6) → 3 ways

---

## 🧠 Problem Visualization

```
Mapping: A=1, B=2, C=3, ..., Z=26

"226" can be decoded as:
  ┌─── "2" "2" "6" ──► B B F
  │
  ├─── "22" "6"    ──► V F
  │
  └─── "2" "26"    ──► B Z

Answer: 3 ways

Key constraint:
  Single digit:  "1" to "9" are valid (NOT "0"!)
  Two digits:    "10" to "26" are valid
```

---

## 🔍 DP Development

### State
```
dp[i] = number of ways to decode s[0..i-1] (first i characters)
```

### Recurrence
```
At position i, consider:
  1. SINGLE digit s[i-1]:
     If s[i-1] != '0': dp[i] += dp[i-1]
     (valid single digit → extend all decodings of s[0..i-2])

  2. TWO digits s[i-2..i-1]:
     If "10" <= s[i-2..i-1] <= "26": dp[i] += dp[i-2]
     (valid two-digit → extend all decodings of s[0..i-3])
```

### Base Cases
```
dp[0] = 1  (empty string: one way to decode nothing)
dp[1] = 1 if s[0] != '0', else 0
```

### Pseudocode
```
function numDecodings(s):
    n = len(s)
    if s[0] == '0': return 0
    
    dp = array[n+1]
    dp[0] = 1
    dp[1] = 1
    
    for i = 2 to n:
        dp[i] = 0
        // Single digit
        if s[i-1] != '0':
            dp[i] += dp[i-1]
        // Two digits
        two_digit = int(s[i-2..i-1])
        if 10 <= two_digit <= 26:
            dp[i] += dp[i-2]
    
    return dp[n]
```

---

## 🧪 Step-by-Step Trace

```
s = "2264"

i │ s[i-1] │ single digit?  │ two digits?      │ dp[i]
──┼────────┼────────────────┼──────────────────┼──────
0 │   -    │       -        │        -         │  1 (base)
1 │  '2'   │ '2'≠'0' → yes │        -         │  1
2 │  '2'   │ '2'≠'0' →+dp[1]=1│ "22"→22, valid →+dp[0]=1│  2
3 │  '6'   │ '6'≠'0' →+dp[2]=2│ "26"→26, valid →+dp[1]=1│  3
4 │  '4'   │ '4'≠'0' →+dp[3]=3│ "64"→64, >26   → skip  │  3

Answer: 3

Verification for "226":
  "2|2|6" → B,B,F
  "22|6"  → V,F
  "2|26"  → B,Z
  = 3 ways ✓
```

---

## ⚠️ Edge Cases

```
┌─────────────────────────────────────────────────────────┐
│  EDGE CASE 1: Leading zero                              │
│    s = "0..."  → return 0 (no valid single digit)       │
│                                                         │
│  EDGE CASE 2: Zero in middle                            │
│    s = "10"  → "10"=J → 1 way                          │
│    s = "30"  → "30">26, '0' invalid alone → 0 ways     │
│    s = "100" → "10"=J + "0" invalid → 0 ways           │
│    s = "101" → "10"=J + "1"=A → 1 way                  │
│                                                         │
│  EDGE CASE 3: All same digits                           │
│    s = "1111" → like Fibonacci: 1,1,2,3,5 → 5 ways     │
│                                                         │
│  EDGE CASE 4: Consecutive zeros                         │
│    s = "00" → 0 ways (no valid decoding)                │
└─────────────────────────────────────────────────────────┘
```

### Trace for "10":
```
dp[0] = 1, dp[1] = 1 (s[0]='1')
i=2: s[1]='0' → single digit NO
     "10"=10 → valid → dp[2] = dp[0] = 1
Answer: 1 (J)
```

### Trace for "30":
```
dp[0] = 1, dp[1] = 1 (s[0]='3')
i=2: s[1]='0' → single digit NO
     "30"=30 → >26 → NO
     dp[2] = 0
Answer: 0 (impossible!)
```

---

## 💡 Space Optimized Solution

```
function numDecodings(s):
    if s[0] == '0': return 0
    prev2 = 1    // dp[i-2]
    prev1 = 1    // dp[i-1]
    
    for i = 2 to len(s):
        curr = 0
        if s[i-1] != '0':
            curr += prev1
        two_digit = int(s[i-2..i-1])
        if 10 <= two_digit <= 26:
            curr += prev2
        prev2 = prev1
        prev1 = curr
    
    return prev1

Time: O(n) | Space: O(1)
```

---

## 📐 Connection to Fibonacci

```
For a string with all digits 1-2 (like "1111"):
  Every single digit is valid (1 or 2)
  Every pair is valid (11, 12, 21, 22 — all ≤ 26)
  
  dp[i] = dp[i-1] + dp[i-2]  ← exactly Fibonacci!
  
  "1111": dp = [1, 1, 2, 3, 5] → 5 ways
  
  But zeros and digits > 2 break the Fibonacci pattern:
  "301": dp[0]=1, dp[1]=1
         i=2: '0' invalid, "30">26 → dp[2]=0 → stuck!
```

---

## 📊 Summary Table

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute Force Recursion | O(2ⁿ) | O(n) | Exponential |
| Memoization | O(n) | O(n) | Top-down |
| Tabulation | O(n) | O(n) | Bottom-up |
| **Space Optimized** | **O(n)** | **O(1)** | **Best** |

---

## ❓ Quick Revision Questions

1. **Why is dp[0] = 1 even though there are no digits to decode?**
2. **What makes this problem different from standard Fibonacci?**
3. **How do you handle the digit '0' in the input?**
4. **What is the output for s = "2101"?**
5. **Why do we check both single-digit and two-digit options at each position?**
6. **How would the problem change if the mapping went up to 99 (two-digit codes)?**

---

[← Previous: Coin Change 2](05-coin-change-ways.md) | [Next Unit: Fibonacci Pattern →](../04-Fibonacci-Pattern/01-fibonacci-number.md)

[← Back to README](../README.md)
