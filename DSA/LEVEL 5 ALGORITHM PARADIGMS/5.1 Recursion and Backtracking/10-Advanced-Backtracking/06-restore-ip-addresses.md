# Chapter 6: Restore IP Addresses

## Overview
**Restore IP Addresses** breaks a digit string into 4 valid octets (0-255) to form all possible valid IPv4 addresses. This problem combines string partitioning backtracking with specific numeric constraints — a great exercise in multi-constraint pruning.

---

## Problem Statement

```
Input:  s = "25525511135"
Output: ["255.255.11.135", "255.255.111.35"]

IPv4 rules:
  • Exactly 4 segments (octets)
  • Each segment: 0 to 255
  • No leading zeros (except "0" itself)
  • Each segment: 1 to 3 digits

Invalid examples:
  "256.1.1.1"    → 256 > 255
  "01.1.1.1"     → leading zero
  "1.1.1.1.1"    → 5 segments
```

---

## Algorithm

```
Place 3 dots in the string to create 4 segments.
At each step, try taking 1, 2, or 3 characters for the
current segment. Validate each segment. Recurse for rest.

                    "25525511135"
                   /      |       \
              "2."    "25."     "255."
              /|\      /|\        /|\
           ...  ...  ...  ...  ...  ...

Continue until 4 segments formed, all chars used.
```

---

## Pseudocode

```
FUNCTION restoreIpAddresses(s):
    result = []
    backtrack(s, 0, [], result)
    RETURN result

FUNCTION backtrack(s, start, segments, result):
    // Pruning: if we have 4 segments, check if done
    IF length(segments) == 4:
        IF start == length(s):
            result.add(join(segments, "."))    ← Valid IP!
        RETURN                                 ← Either way, stop
    
    // Try 1, 2, or 3 digit segments
    FOR len = 1 TO 3:
        IF start + len > length(s):
            BREAK                              ← Not enough chars
        
        segment = s[start .. start+len-1]
        
        // Validate segment
        IF len > 1 AND segment[0] == '0':
            CONTINUE                           ← No leading zeros
        IF parseInt(segment) > 255:
            CONTINUE                           ← Must be ≤ 255
        
        // Pruning: enough chars left for remaining segments?
        remaining = length(s) - start - len
        segmentsLeft = 3 - length(segments)
        IF remaining < segmentsLeft OR remaining > segmentsLeft * 3:
            CONTINUE                           ← Can't form valid IP
        
        segments.add(segment)                  ← CHOOSE
        backtrack(s, start + len, segments, result)  ← EXPLORE
        segments.removeLast()                  ← UNDO
```

---

## Trace: s = "25525511135"

```
backtrack("25525511135", 0, [])
│
├─ len=1: seg="2" (valid)
│  backtrack(1, ["2"])
│  ├─ seg="5" → backtrack(2, ["2","5"])
│  │  ├─ seg="5" → ["2","5","5"] → remaining=8, need 1-3 → skip (8>3)
│  │  ├─ seg="25" → ["2","5","25"] → remaining=6, need 1-3 → skip
│  │  └─ seg="255" → ["2","5","255"] → remaining=5, need 1-3 → skip
│  ├─ seg="55" → backtrack(3, ["2","55"])
│  │  └─ ... remaining chars too many for 2 segments
│  └─ seg="552" → > 255 → skip
│
├─ len=2: seg="25" (valid)
│  backtrack(2, ["25"])
│  ├─ seg="5" → backtrack(3, ["25","5"])
│  │  └─ ... similar pruning
│  ├─ seg="52" → backtrack(4, ["25","52"])
│  │  └─ ... remaining too many
│  └─ seg="525" → > 255 → skip
│
├─ len=3: seg="255" (valid, =255)
│  backtrack(3, ["255"])
│  ├─ seg="2" → backtrack(4, ["255","2"])
│  │  └─ ... remaining=7, need 2 segs of 1-3 → 2≤7? 7≤6? skip
│  ├─ seg="25" → backtrack(5, ["255","25"])
│  │  └─ ... remaining=6, need 1-3 × 2 → skip (6>6? no, 6=6 ok... but 511135 splits needed)
│  ├─ seg="255" → backtrack(6, ["255","255"])
│  │  ├─ seg="1" → ["255","255","1"], remaining=4, need 1-3 → skip(4>3)
│  │  ├─ seg="11" → ["255","255","11"], remaining=3, need 1-3 → 
│  │  │  ├─ seg="135" → 135≤255 ✓
│  │  │  │  segments=4, start=11==len → ADD "255.255.11.135" ✓
│  │  ├─ seg="111" → ["255","255","111"], remaining=2, need 1-3 →
│  │  │  ├─ seg="35" → 35≤255 ✓
│  │  │  │  segments=4, start=11==len → ADD "255.255.111.35" ✓

Result: ["255.255.11.135", "255.255.111.35"]
```

---

## Pruning Effectiveness

```
Without pruning:
  At each of 4 levels, try 3 lengths = 3^4 = 81 paths
  Most fail at the end (used too few/many chars)

With remaining-chars pruning:
  remaining = chars left after current segment
  segmentsLeft = segments still needed

  Skip if: remaining < segmentsLeft     (too few chars)
           remaining > segmentsLeft * 3 (too many chars)

  This eliminates most branches early!

  For "25525511135" (11 chars):
  Without pruning: ~81 paths explored
  With pruning:    ~15 paths explored
```

---

## Three-Loop Alternative (No Recursion)

```
For exactly 4 segments, three nested loops work:

FUNCTION restoreIp(s):
    result = []
    n = length(s)
    FOR i = 1 TO min(3, n-3):           ← 1st dot position
      FOR j = i+1 TO min(i+3, n-2):     ← 2nd dot position
        FOR k = j+1 TO min(j+3, n-1):   ← 3rd dot position
          s1 = s[0..i-1]
          s2 = s[i..j-1]
          s3 = s[j..k-1]
          s4 = s[k..n-1]
          IF isValid(s1) AND isValid(s2) 
             AND isValid(s3) AND isValid(s4):
              result.add(s1+"."+s2+"."+s3+"."+s4)
    RETURN result

This is O(1) — at most 3×3×3 = 27 iterations!
Fixed depth (4 segments) makes enumeration bounded.
```

---

## Complexity

```
┌──────────────────────────────────────────────┐
│ Backtracking approach:                       │
│   Time:  O(3^4 × n) = O(27n) = O(n)         │
│   At most 3 choices × 4 levels = 81 paths    │
│   Each path: O(n) for string operations      │
│                                              │
│ Three-loop approach:                         │
│   Time:  O(27 × n) = O(n)                    │
│   Constant number of iterations              │
│                                              │
│ Space: O(1) extra (excluding output)         │
│   Maximum output: bounded constant           │
│   (at most ~20 valid IPs for any input)      │
└──────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Input | Digit string (length 4 to 12) |
| Output | All valid IPv4 addresses |
| Segments | Exactly 4 |
| Digit range | 1-3 digits per segment |
| Value range | 0-255 |
| No leading zeros | "01" invalid, "0" valid |
| Pruning | Remaining chars vs segments needed |
| Time | O(n) — bounded constant paths |

---

## Quick Revision Questions

1. **Why is this problem bounded** even though it's backtracking?
2. **What three validations** must each segment pass?
3. **How does the remaining-chars** pruning work?
4. **Why can a three-loop** approach solve this without recursion?
5. **What's the maximum length** of a valid IP input string?
6. **How do you handle** the segment "0" vs "00"?

---

[← Previous: Generate Parentheses](05-generate-parentheses.md) | [Next: Unit 11 - Pruning Strategies →](../11-Optimization-Techniques/01-pruning-strategies.md)

[← Back to README](../README.md)
