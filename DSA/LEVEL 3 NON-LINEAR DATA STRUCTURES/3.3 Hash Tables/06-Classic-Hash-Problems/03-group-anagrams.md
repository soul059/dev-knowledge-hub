# 6.3 Group Anagrams

## Unit 6: Classic Hash Problems

---

## Problem Statement

> Given an array of strings, group the anagrams together. An anagram is a word formed by rearranging the letters of another word.

```
Input:  ["eat", "tea", "tan", "ate", "nat", "bat"]
Output: [["eat","tea","ate"], ["tan","nat"], ["bat"]]
```

---

## Core Insight: Canonical Form

```
╔══════════════════════════════════════════════════════════════╗
║  Two strings are anagrams ↔ they have the SAME sorted form ║
║                                                              ║
║  "eat" → sort → "aet"                                      ║
║  "tea" → sort → "aet"   ← same!                           ║
║  "ate" → sort → "aet"   ← same!                           ║
║                                                              ║
║  "tan" → sort → "ant"                                      ║
║  "nat" → sort → "ant"   ← same!                           ║
║                                                              ║
║  "bat" → sort → "abt"                                      ║
║                                                              ║
║  Use sorted string as HASH MAP KEY                          ║
║  → all anagrams map to the same key                        ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Algorithm 1: Sort-Based Key

```
FUNCTION groupAnagrams(strs):
    groups = new HashMap()    // sorted_string → list of originals

    FOR each str in strs:
        key = sort(str)       // "eat" → "aet"
        IF not groups.containsKey(key):
            groups.put(key, [])
        groups.get(key).add(str)

    RETURN groups.values()

Time: O(n × k log k)    where k = max string length
Space: O(n × k)
```

### Trace

```
Input: ["eat", "tea", "tan", "ate", "nat", "bat"]

Step 1: "eat" → sort = "aet"
  groups = {"aet": ["eat"]}

Step 2: "tea" → sort = "aet"
  groups = {"aet": ["eat", "tea"]}

Step 3: "tan" → sort = "ant"
  groups = {"aet": ["eat", "tea"], "ant": ["tan"]}

Step 4: "ate" → sort = "aet"
  groups = {"aet": ["eat", "tea", "ate"], "ant": ["tan"]}

Step 5: "nat" → sort = "ant"
  groups = {"aet": ["eat", "tea", "ate"], "ant": ["tan", "nat"]}

Step 6: "bat" → sort = "abt"
  groups = {"aet": ["eat","tea","ate"], "ant": ["tan","nat"], "abt": ["bat"]}

Output: [["eat","tea","ate"], ["tan","nat"], ["bat"]]
```

---

## Algorithm 2: Frequency-Based Key (Faster)

Instead of sorting, use character counts as the key:

```
FUNCTION groupAnagrams(strs):
    groups = new HashMap()

    FOR each str in strs:
        // Build frequency key
        count = new Array(26, fill=0)
        FOR each char c in str:
            count[c - 'a']++
        
        key = arrayToString(count)   // "1#0#0#...#1#0#1#..." 
        
        IF not groups.containsKey(key):
            groups.put(key, [])
        groups.get(key).add(str)

    RETURN groups.values()

Time: O(n × k)     ← faster! No sorting needed
Space: O(n × k)
```

### Key Encoding Example

```
"eat":  e=1, a=1, t=1
  count = [1,0,0,0,1,0,...,0,1,0,...,0]
  key   = "1#0#0#0#1#0#...#0#1#0#...#0"

"tea":  t=1, e=1, a=1
  count = [1,0,0,0,1,0,...,0,1,0,...,0]
  key   = "1#0#0#0#1#0#...#0#1#0#...#0"  ← SAME KEY!

"tan":  t=1, a=1, n=1
  count = [1,0,0,0,0,...,1,...,0,1,0,...,0]
  key   = "1#0#0#0#0#...#1#...#0#1#0#...#0"  ← different
```

---

## Why Use # Separator?

```
╔══════════════════════════════════════════════════════════════╗
║  Without separator:                                         ║
║  count = [1, 12, 0, ...] → "1120..."                      ║
║  count = [11, 2, 0, ...] → "1120..."  ← COLLISION!        ║
║                                                              ║
║  With separator:                                            ║
║  count = [1, 12, 0, ...]  → "1#12#0#..."                  ║
║  count = [11, 2, 0, ...]  → "11#2#0#..."  ← different ✓  ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Comparison of Approaches

| Approach | Time | Space | Pros | Cons |
|----------|:---:|:---:|------|------|
| Sort-based | $O(nk\log k)$ | $O(nk)$ | Simple, intuitive | Sorting overhead |
| Frequency-based | $O(nk)$ | $O(nk)$ | Faster | Longer key strings |
| Prime product | $O(nk)$ | $O(nk)$ | Compact key | Overflow risk |

### Prime Product Method (Alternative)

```
Assign each letter a unique prime:
  a=2, b=3, c=5, d=7, e=11, ...

Product of primes = unique per anagram group:
  "eat" → 11 × 2 × 71 = 1562
  "tea" → 71 × 11 × 2 = 1562  ← same!
  "ate" → 2 × 71 × 11 = 1562  ← same!

Caveat: Products can overflow for long strings!
```

---

## Related Problem: Valid Anagram

```
FUNCTION isAnagram(s, t):
    IF len(s) != len(t):
        RETURN false

    count = new HashMap()
    
    FOR i = 0 TO len(s) - 1:
        count[s[i]] = count.getOrDefault(s[i], 0) + 1
        count[t[i]] = count.getOrDefault(t[i], 0) - 1

    FOR each val in count.values():
        IF val != 0:
            RETURN false
    RETURN true
```

---

## Complexity

| Metric | Sort-based | Frequency-based |
|--------|:---:|:---:|
| Time | $O(n \cdot k \log k)$ | $O(n \cdot k)$ |
| Space | $O(n \cdot k)$ | $O(n \cdot k)$ |
| Hash key size | $k$ characters | $26$ numbers |
| Better when | $k$ is small | $k$ is large |

Where $n$ = number of strings, $k$ = max string length.

---

## Quick Revision Question

**Q: Why is using a sorted string as a hash map key guaranteed to group all anagrams correctly? Could two non-anagram strings produce the same sorted key?**

<details>
<summary>Click to reveal answer</summary>

Sorting is guaranteed to group anagrams correctly because **anagrams are defined as strings with exactly the same characters in different orders**. Sorting normalizes the order, so any two anagrams will produce identical sorted strings.

Two non-anagram strings **cannot** produce the same sorted key. If two strings have the same sorted form, they contain exactly the same character multiset (same characters with same frequencies), which by definition makes them anagrams. The sorted form is a **canonical representation** — a unique, order-independent fingerprint of the character content.

The frequency-based approach works for the same reason: two strings are anagrams if and only if they have identical character frequency distributions.

</details>

---

| [← Previous: Frequency Counting](02-frequency-counting.md) | [Next: Prefix Sum with HashMap →](04-prefix-sum-hashmap.md) |
|:---|---:|
