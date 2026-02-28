# 7.4 First Unique Character

## Unit 7: Hash Set Problems

---

## Problem Statement

> Given a string `s`, find the **first non-repeating character** and return its index. If none exists, return -1.

```
╔══════════════════════════════════════════════════════════════╗
║  Input:  s = "leetcode"                                     ║
║  Output: 0           (character 'l')                        ║
║                                                              ║
║  Input:  s = "loveleetcode"                                 ║
║  Output: 2           (character 'v')                        ║
║                                                              ║
║  Input:  s = "aabb"                                         ║
║  Output: -1          (no unique character)                  ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Approach 1: HashMap — Two Pass

```
FUNCTION firstUniqChar(s):
    freq = new HashMap()

    // Pass 1: Count frequencies
    FOR each char c in s:
        freq[c] = freq.getOrDefault(c, 0) + 1

    // Pass 2: Find first with count 1
    FOR i = 0 TO len(s) - 1:
        IF freq[s[i]] == 1:
            RETURN i

    RETURN -1
```

### Trace

```
s = "loveleetcode"

Pass 1 - Build frequency map:
  l:1 → o:1 → v:1 → e:1
  l:2 → e:2 → e:3 → t:1
  c:1 → o:2 → d:1 → e:4

  freq = {l:2, o:2, v:1, e:4, t:1, c:1, d:1}

Pass 2 - Scan left to right:
  i=0:  l → freq=2 → skip
  i=1:  o → freq=2 → skip
  i=2:  v → freq=1 → RETURN 2 ✓

Answer: index 2, character 'v'
```

**Time:** $O(n)$  |  **Space:** $O(k)$ where $k$ = alphabet size

---

## Approach 2: Array (Fixed Alphabet)

When the character set is known (e.g., lowercase English letters):

```
FUNCTION firstUniqChar(s):
    count = new int[26]    // All zeros

    FOR each char c in s:
        count[c - 'a']++

    FOR i = 0 TO len(s) - 1:
        IF count[s[i] - 'a'] == 1:
            RETURN i

    RETURN -1
```

This is faster in practice — array access is ~3x faster than HashMap lookup.

---

## Approach 3: Two HashSets — Seen + Unique

```
FUNCTION firstUniqChar(s):
    seen = new HashSet()
    unique = new LinkedHashSet()    // preserves insertion order

    FOR each char c in s:
        IF seen.contains(c):
            unique.remove(c)
        ELSE:
            seen.add(c)
            unique.add(c)

    IF unique.isEmpty():
        RETURN -1

    firstChar = unique.iterator().next()
    RETURN s.indexOf(firstChar)
```

### Trace

```
s = "loveleetcode"

c='l': seen={l}, unique={l}
c='o': seen={l,o}, unique={l,o}
c='v': seen={l,o,v}, unique={l,o,v}
c='e': seen={l,o,v,e}, unique={l,o,v,e}
c='l': seen already → remove from unique → unique={o,v,e}
c='e': seen already → remove from unique → unique={o,v}
c='e': seen already → not in unique → unique={o,v}
c='t': seen={l,o,v,e,t}, unique={o,v,t}
c='c': seen={l,o,v,e,t,c}, unique={o,v,t,c}
c='o': seen already → remove from unique → unique={v,t,c}
c='d': seen={l,o,v,e,t,c,d}, unique={v,t,c,d}
c='e': seen already → not in unique → unique={v,t,c,d}

First in unique: 'v' → index 2 ✓
```

---

## Variation: First Unique in a Stream

> Characters arrive one at a time. After each arrival, report the first non-repeating character.

```
╔══════════════════════════════════════════════════════════════╗
║  Stream: a → a b → a b c → a b c b → a b c b d           ║
║  First:  a     a     a       a         a                   ║
║  Unique? yes   yes   yes     yes       yes                 ║
║                                                              ║
║  Wait... let me re-trace:                                  ║
║  "a"        → unique={a}       → first='a'                 ║
║  "aa"       → unique={}        → first='#' (none)          ║
║  "aab"      → unique={b}       → first='b'                ║
║  "aabc"     → unique={b,c}     → first='b'                ║
║  "aabcb"    → unique={c}       → first='c'                ║
║  "aabcbd"   → unique={c,d}     → first='c'                ║
╚══════════════════════════════════════════════════════════════╝
```

Use **LinkedHashSet** (or Queue + HashMap) for $O(1)$ "first unique" queries:

```
CLASS FirstUnique:
    seen = new HashSet()
    unique = new LinkedHashSet()

    FUNCTION add(char c):
        IF seen.contains(c):
            unique.remove(c)
        ELSE:
            seen.add(c)
            unique.add(c)

    FUNCTION getFirst():
        IF unique.isEmpty(): RETURN '#'
        RETURN unique.iterator().next()
```

---

## Variation: First Unique Number

> Same problem but with integers instead of characters.

```
nums = [7, 2, 3, 2, 5, 7]

freq = {7:2, 2:2, 3:1, 5:1}

Scan: 7(2)→skip, 2(2)→skip, 3(1)→FOUND!
Answer: 3
```

The algorithm is identical — HashMap works for any hashable type, not just characters.

---

## Methods Compared

| Approach | Time | Space | Handles Stream? |
|----------|:---:|:---:|:---:|
| HashMap (Two-pass) | $O(n)$ | $O(k)$ | No |
| Array (fixed alphabet) | $O(n)$ | $O(1)$ | No |
| LinkedHashSet | $O(n)$ | $O(k)$ | Yes |
| Queue + HashMap | $O(n)$ amortized | $O(k)$ | Yes |

---

## Quick Revision Question

**Q: Can you find the first non-repeating character using only a single pass through the string? What data structure would you need?**

<details>
<summary>Click to reveal answer</summary>

Yes, using a **LinkedHashSet** (ordered set) combined with a regular **HashSet**:

1. **HashSet `seen`**: Tracks all characters encountered
2. **LinkedHashSet `unique`**: Maintains unique characters in insertion order

**Single-pass algorithm:**
```
FOR each char c in s:
    IF c in seen:
        unique.remove(c)    // O(1) in LinkedHashSet
    ELSE:
        seen.add(c)
        unique.add(c)       // adds to end

RETURN first element of unique (or -1 if empty)
```

The key insight is that **LinkedHashSet** preserves insertion order AND supports $O(1)$ removal. After the pass, the first element of `unique` is the answer — it was:
- The earliest character added (insertion order preserved)
- Never seen again (otherwise it would have been removed)

This is essentially the stream solution applied to a fixed string.

</details>

---

| [← Union of Arrays](03-union-of-arrays.md) | [Next: Contains Duplicate Variants →](05-contains-duplicate.md) |
|:---|---:|
