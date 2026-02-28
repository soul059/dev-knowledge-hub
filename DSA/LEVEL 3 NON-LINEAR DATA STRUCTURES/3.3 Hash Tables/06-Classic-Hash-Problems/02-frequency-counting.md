# 6.2 Frequency Counting

## Unit 6: Classic Hash Problems

---

## The Pattern

**Frequency counting** uses a hash map to count how many times each element appears in a collection.

```
╔══════════════════════════════════════════════════════════════╗
║              FREQUENCY COUNTING PATTERN                     ║
║                                                              ║
║  Input:  [a, b, a, c, b, a, d, c, a]                       ║
║                                                              ║
║  Hash Map:                                                   ║
║  ┌─────┬───────┐                                            ║
║  │ Key │ Count │                                            ║
║  ├─────┼───────┤                                            ║
║  │  a  │   4   │                                            ║
║  │  b  │   2   │                                            ║
║  │  c  │   2   │                                            ║
║  │  d  │   1   │                                            ║
║  └─────┴───────┘                                            ║
║                                                              ║
║  Time: O(n)  |  Space: O(k) where k = unique elements      ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Basic Algorithm

```
FUNCTION countFrequencies(arr):
    freq = new HashMap()

    FOR each element in arr:
        IF freq.containsKey(element):
            freq.put(element, freq.get(element) + 1)
        ELSE:
            freq.put(element, 1)

    RETURN freq
```

Or more concisely with `getOrDefault`:

```
FUNCTION countFrequencies(arr):
    freq = new HashMap()
    FOR each element in arr:
        freq.put(element, freq.getOrDefault(element, 0) + 1)
    RETURN freq
```

---

## Step-by-Step Trace

```
Input: ["apple", "banana", "apple", "cherry", "banana", "apple"]

Step 1: "apple"   → freq = {apple: 1}
Step 2: "banana"  → freq = {apple: 1, banana: 1}
Step 3: "apple"   → freq = {apple: 2, banana: 1}
Step 4: "cherry"  → freq = {apple: 2, banana: 1, cherry: 1}
Step 5: "banana"  → freq = {apple: 2, banana: 2, cherry: 1}
Step 6: "apple"   → freq = {apple: 3, banana: 2, cherry: 1}

Result: apple=3, banana=2, cherry=1
```

---

## Classic Applications

### 1. Find Most Frequent Element (Mode)

```
FUNCTION findMode(arr):
    freq = countFrequencies(arr)
    maxCount = 0
    mode = null

    FOR each (key, count) in freq:
        IF count > maxCount:
            maxCount = count
            mode = key

    RETURN mode

Trace: [1, 3, 2, 1, 4, 1, 3, 1]
  freq = {1:4, 3:2, 2:1, 4:1}
  mode = 1 (appears 4 times)
```

### 2. Find First Non-Repeating Character

```
FUNCTION firstUnique(str):
    freq = new HashMap()

    // First pass: count
    FOR each char c in str:
        freq.put(c, freq.getOrDefault(c, 0) + 1)

    // Second pass: find first with count 1
    FOR each char c in str:
        IF freq.get(c) == 1:
            RETURN c

    RETURN null

Trace: "aabbcdde"
  freq = {a:2, b:2, c:1, d:2, e:1}
  Second pass: a(2)→skip, a(2)→skip, b(2)→skip, b(2)→skip,
               c(1) → RETURN 'c' ✓
```

### 3. Check if Two Strings are Anagrams

```
FUNCTION isAnagram(s1, s2):
    IF length(s1) != length(s2):
        RETURN false

    freq = new HashMap()

    // Count characters in s1
    FOR each char c in s1:
        freq.put(c, freq.getOrDefault(c, 0) + 1)

    // Subtract characters in s2
    FOR each char c in s2:
        IF not freq.containsKey(c):
            RETURN false
        freq.put(c, freq.get(c) - 1)
        IF freq.get(c) < 0:
            RETURN false

    RETURN true

Trace: s1="listen", s2="silent"
  After s1: {l:1, i:1, s:1, t:1, e:1, n:1}
  After s2: {l:0, i:0, s:0, t:0, e:0, n:0}  → all zero → true ✓
```

---

### 4. Top K Frequent Elements

```
FUNCTION topKFrequent(nums, k):
    // Step 1: Count frequencies
    freq = countFrequencies(nums)

    // Step 2: Bucket sort by frequency
    buckets = new Array(n + 1)    // index = frequency

    FOR each (num, count) in freq:
        IF buckets[count] is null:
            buckets[count] = []
        buckets[count].add(num)

    // Step 3: Collect top k from highest bucket
    result = []
    FOR i = n DOWN TO 0:
        IF buckets[i] is not null:
            FOR each num in buckets[i]:
                result.add(num)
                IF result.size() == k:
                    RETURN result

    RETURN result

Time: O(n)  |  Space: O(n)

Trace: nums = [1,1,1,2,2,3], k = 2
  freq = {1:3, 2:2, 3:1}
  buckets: [0]=null, [1]=[3], [2]=[2], [3]=[1]
  Collect from top: bucket[3]→1, bucket[2]→2
  RETURN [1, 2] ✓
```

---

## Frequency Counting on Various Data Types

```
╔═══════════════════════════════════════════════════════════════╗
║  Data Type        │ Key in HashMap        │ Example           ║
║  ─────────────────┼───────────────────────┼──────────────     ║
║  Characters       │ char                  │ 'a' → 3           ║
║  Words            │ String                │ "hello" → 2       ║
║  Numbers          │ int / Integer         │ 42 → 5            ║
║  Pairs            │ Tuple / String concat │ (1,2) → 1         ║
║  IP addresses     │ String                │ "1.2.3.4" → 100   ║
║  Subarrays        │ encoded String / hash │ [1,2,3] → 2       ║
╚═══════════════════════════════════════════════════════════════╝
```

---

## Complexity

| Problem | Time | Space |
|---------|:---:|:---:|
| Count frequencies | $O(n)$ | $O(k)$ |
| Find mode | $O(n)$ | $O(k)$ |
| First unique char | $O(n)$ | $O(k)$ |
| Check anagram | $O(n)$ | $O(k)$ |
| Top K frequent | $O(n)$ | $O(n)$ |

Where $k$ = number of unique elements, $k \leq n$.

---

## Quick Revision Question

**Q: Given the string "engineering", build the frequency map step by step and identify the most frequent character.**

<details>
<summary>Click to reveal answer</summary>

```
"engineering"

e → {e:1}
n → {e:1, n:1}
g → {e:1, n:1, g:1}
i → {e:1, n:1, g:1, i:1}
n → {e:1, n:2, g:1, i:1}
e → {e:2, n:2, g:1, i:1}
e → {e:3, n:2, g:1, i:1}
r → {e:3, n:2, g:1, i:1, r:1}
i → {e:3, n:2, g:1, i:2, r:1}
n → {e:3, n:3, g:1, i:2, r:1}
g → {e:3, n:3, g:2, i:2, r:1}
```

Final: `{e:3, n:3, g:2, i:2, r:1}`

Most frequent: **'e' and 'n'** (tied at 3 occurrences each). If we need a single answer, it would be whichever appears first in the string — 'e' (index 0).

</details>

---

| [← Previous: Two Sum](01-two-sum-problem.md) | [Next: Group Anagrams →](03-group-anagrams.md) |
|:---|---:|
