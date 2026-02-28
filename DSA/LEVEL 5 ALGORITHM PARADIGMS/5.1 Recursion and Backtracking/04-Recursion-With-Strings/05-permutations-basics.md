# Chapter 5: Permutations Basics

## Overview
A permutation is a rearrangement of all elements. Unlike subsequences (include/exclude), permutations involve **choosing which element goes in each position**. This introduces the **swap-and-recurse** pattern.

---

## What is a Permutation?

```
String: "abc"
Permutations: "abc", "acb", "bac", "bca", "cab", "cba"
Total: n! = 3! = 6

[!] ALL characters are used, only ORDER changes
[!] Subsequences: 2^n choices, Permutations: n! arrangements
```

---

## Recursive Approach: Fix and Permute Rest

```
For each position, try every available character:

Position 0: fix 'a', permute "bc"  →  "abc", "acb"
            fix 'b', permute "ac"  →  "bac", "bca"
            fix 'c', permute "ab"  →  "cab", "cba"
```

### Pseudocode (Swap Method)
```
function permute(s, left, right):
    if left == right:
        print(s)                    // Base: all positions fixed
        return
    for i = left to right:
        swap(s[left], s[i])         // Try character i at position left
        permute(s, left + 1, right) // Permute the rest
        swap(s[left], s[i])         // UNDO the swap (backtrack!)
```

### Decision Tree for "abc"

```
                        "abc"
                     /    |    \
               fix a     fix b    fix c
              (swap 0,0) (swap 0,1) (swap 0,2)
              "abc"      "bac"     "cba"
             /    \     /    \    /    \
          fix b  fix c fix a fix c fix b fix a
          "abc" "acb" "bac" "bca" "cba" "cab"
           ↓     ↓     ↓     ↓     ↓     ↓
          LEAF  LEAF  LEAF  LEAF  LEAF  LEAF

All 6 permutations generated!
```

---

## Step-by-Step Trace

```
permute("abc", 0, 2):

1. i=0: swap(0,0)→"abc", permute("abc",1,2)
   1.1 i=1: swap(1,1)→"abc", permute("abc",2,2)→print "abc"
       swap(1,1)→"abc" (undo)
   1.2 i=2: swap(1,2)→"acb", permute("acb",2,2)→print "acb"
       swap(1,2)→"abc" (undo)
   swap(0,0)→"abc" (undo)

2. i=1: swap(0,1)→"bac", permute("bac",1,2)
   2.1 i=1: swap(1,1)→"bac"→print "bac"
   2.2 i=2: swap(1,2)→"bca"→print "bca"
   swap(0,1)→"abc" (undo)

3. i=2: swap(0,2)→"cba", permute("cba",1,2)
   3.1 i=1: swap(1,1)→"cba"→print "cba"
   3.2 i=2: swap(1,2)→"cab"→print "cab"
   swap(0,2)→"abc" (undo)

Output: abc, acb, bac, bca, cba, cab
```

---

## Complexity

```
Time:  O(n × n!) — n! permutations, each of length n
Space: O(n) — recursion depth (n levels)
Total permutations = n! (grows VERY fast)

n=3:  6
n=5:  120
n=8:  40,320
n=10: 3,628,800
n=12: 479,001,600
```

---

## Key: Backtracking (Undo Step)

```
swap(s[left], s[i])         ← MAKE choice
permute(s, left + 1, right) ← EXPLORE
swap(s[left], s[i])         ← UNDO choice (backtrack!)

This is the FIRST example of BACKTRACKING:
  Make a choice → Explore → Undo the choice → Try next
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Total permutations | n! |
| Pattern | Fix one position, permute rest |
| Technique | Swap + recurse + undo (backtrack) |
| Base case | All positions fixed (left == right) |
| Time | O(n × n!) |
| Space | O(n) |
| Key insight | First introduction to backtracking |

---

## Quick Revision Questions

1. **How many permutations** does a string of length 5 have?
2. **Why do we swap back** (undo) after the recursive call?
3. **What is the difference** between subsequences and permutations?
4. **Draw the decision tree** for permutations of "ab".
5. **Why is the time complexity** O(n × n!) and not just O(n!)?
6. **How does this problem** introduce the concept of backtracking?

---

[← Previous: Subsequences of String](04-subsequences-of-string.md) | [Next: String Compression →](06-string-compression.md)

[← Back to README](../README.md)
