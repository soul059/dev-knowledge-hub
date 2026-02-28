# Chapter 3: Word Break II

## Overview
**Word Break II** returns all possible sentences formed by segmenting a string into valid dictionary words. Unlike Word Break I (which just checks feasibility), this problem requires enumerating all valid decompositions — a natural fit for backtracking with memoization.

---

## Problem Statement

```
Input:  s = "catsanddog"
        wordDict = ["cat","cats","and","sand","dog"]

Output: ["cats and dog", "cat sand dog"]

Breakdown:
  "catsanddog"
   ├── "cat" + "sanddog"
   │         ├── "sand" + "dog" ✓  → "cat sand dog"
   │         └── "s" ✗ (not in dict)
   └── "cats" + "anddog"
              ├── "and" + "dog" ✓  → "cats and dog"
              └── "a" ✗ (not in dict)
```

---

## Pseudocode

```
FUNCTION wordBreak(s, wordDict):
    wordSet = SET(wordDict)          ← O(1) lookups
    result = []
    backtrack(s, wordSet, 0, [], result)
    RETURN result

FUNCTION backtrack(s, wordSet, start, words, result):
    IF start == length(s):
        result.add(join(words, " "))     ← Complete sentence!
        RETURN
    
    FOR end = start+1 TO length(s):
        word = s[start..end-1]
        IF word IN wordSet:
            words.add(word)              ← CHOOSE
            backtrack(s, wordSet, end, words, result)  ← EXPLORE
            words.removeLast()           ← UNDO
```

---

## Trace: s = "catsanddog"

```
backtrack("catsanddog", 0, [])
│
├─ end=3: "cat" ∈ dict ✓
│  words=["cat"]
│  backtrack("catsanddog", 3, ["cat"])
│  │
│  ├─ end=4: "s" ∉ dict ✗
│  ├─ end=5: "sa" ∉ dict ✗
│  ├─ end=6: "san" ∉ dict ✗
│  ├─ end=7: "sand" ∈ dict ✓
│  │  words=["cat","sand"]
│  │  backtrack("catsanddog", 7, ["cat","sand"])
│  │  │
│  │  ├─ end=8: "d" ∉ dict ✗
│  │  ├─ end=9: "do" ∉ dict ✗
│  │  ├─ end=10: "dog" ∈ dict ✓
│  │  │  words=["cat","sand","dog"]
│  │  │  start==10==len → ADD "cat sand dog" ✓
│  │  │  removeLast → ["cat","sand"]
│  │  removeLast → ["cat"]
│  │
│  ├─ end=8..10: no more dict words
│  removeLast → []
│
├─ end=4: "cats" ∈ dict ✓
│  words=["cats"]
│  backtrack("catsanddog", 4, ["cats"])
│  │
│  ├─ end=5: "a" ∉ dict ✗
│  ├─ end=6: "an" ∉ dict ✗
│  ├─ end=7: "and" ∈ dict ✓
│  │  words=["cats","and"]
│  │  backtrack("catsanddog", 7, ["cats","and"])
│  │  │
│  │  ├─ end=10: "dog" ∈ dict ✓
│  │  │  words=["cats","and","dog"]
│  │  │  start==10==len → ADD "cats and dog" ✓
│  │  │  removeLast → ["cats","and"]
│  │  removeLast → ["cats"]
│  │
│  removeLast → []

Result: ["cat sand dog", "cats and dog"]
```

---

## Optimization: Memoization

```
Problem: Same suffix might be reached from different prefixes.

s = "aaaa", dict = ["a","aa","aaa","aaaa"]
  index 2 can be reached as: "a"+"a" or "aa"
  → Repeated work for suffix "aa"

Solution: Cache results for each starting index.

FUNCTION backtrackMemo(s, wordSet, start, memo):
    IF start IN memo:
        RETURN memo[start]
    
    IF start == length(s):
        RETURN [""]                     ← Base: empty suffix
    
    sentences = []
    FOR end = start+1 TO length(s):
        word = s[start..end-1]
        IF word IN wordSet:
            suffixSentences = backtrackMemo(s, wordSet, end, memo)
            FOR each suffix IN suffixSentences:
                IF suffix == "":
                    sentences.add(word)
                ELSE:
                    sentences.add(word + " " + suffix)
    
    memo[start] = sentences
    RETURN sentences
```

---

## Optimization: Pre-check Feasibility

```
Before generating all sentences, check if ANY segmentation exists.
Use Word Break I (DP, O(n²)) first.

If not feasible → return [] immediately.

This avoids exponential backtracking on strings like:
  s = "aaaa...aab"  (no 'b' in dict)
  → Would explore many dead-end paths before failing
  → DP check catches it in O(n²)

FUNCTION wordBreak(s, wordDict):
    IF NOT canBreak(s, wordDict):    ← Word Break I check
        RETURN []
    RETURN backtrackWithMemo(s, wordDict)
```

---

## Limiting Word Length

```
Further optimization: only try substrings up to 
the maximum word length in the dictionary.

maxLen = max(length(w) for w in wordDict)

FOR end = start+1 TO min(start+maxLen, length(s)):
    ...

This trims the inner loop significantly when
the dictionary has short words but s is long.
```

---

## Complexity

```
┌───────────────────────────────────────────────────────┐
│ Worst case: exponential number of valid sentences     │
│                                                       │
│ Example: s = "aaa...a" (n a's), dict = ["a","aa",...] │
│ Valid sentences = Fibonacci-like growth = O(2^n)       │
│                                                       │
│ Time:  O(2^n × n) in worst case                       │
│   (can't avoid if ALL decompositions are valid)       │
│                                                       │
│ With memoization: O(n² × k) where k = # of results   │
│   per suffix position                                 │
│                                                       │
│ Space: O(n × k) for memoization cache                 │
│   + O(n) recursion depth                              │
└───────────────────────────────────────────────────────┘
```

---

## Word Break I vs II Comparison

| Aspect | Word Break I | Word Break II |
|--------|-------------|---------------|
| Question | CAN it be segmented? | List ALL segmentations |
| Return | Boolean | List of sentences |
| Approach | DP (bottom-up) | Backtracking + memo |
| Time | O(n² × m) | O(2^n × n) worst case |
| Space | O(n) | O(2^n × n) for results |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Input | String + dictionary (set) |
| Output | All valid sentence segmentations |
| Choice | Where to split next (end position) |
| Constraint | Substring must be in dictionary |
| Pruning | Skip non-dictionary words, limit max length |
| Pre-check | Word Break I feasibility test |
| Memoization | Cache sentences per starting index |
| Time | O(2^n × n) worst case |

---

## Quick Revision Questions

1. **How does Word Break II** differ from I in approach?
2. **What does memoization** cache in this problem?
3. **Why is a pre-feasibility** check helpful?
4. **What's the worst-case** input for this problem?
5. **How does max word length** optimization help?
6. **Why can't we avoid** exponential time in the worst case?

---

[← Previous: Expression Add Operators](02-expression-add-operators.md) | [Next: N-Queens II →](04-n-queens-ii.md)

[← Back to README](../README.md)
