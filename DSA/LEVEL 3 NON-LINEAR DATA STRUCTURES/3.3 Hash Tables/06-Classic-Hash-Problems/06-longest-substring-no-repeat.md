# 6.6 Longest Substring Without Repeating Characters

## Unit 6: Classic Hash Problems

---

## Problem Statement

> Given a string `s`, find the length of the **longest substring** without repeating characters.

```
Input:  "abcabcbb"
Output: 3  ("abc")

Input:  "bbbbb"
Output: 1  ("b")

Input:  "pwwkew"
Output: 3  ("wke")
```

---

## Approach: Sliding Window + HashSet

```
╔══════════════════════════════════════════════════════════════╗
║            SLIDING WINDOW + HASH SET                        ║
║                                                              ║
║  Maintain a window [left, right] of unique characters:      ║
║                                                              ║
║  String:  a  b  c  a  b  c  b  b                           ║
║           ↑        ↑                                         ║
║          left    right                                       ║
║          └────────┘                                          ║
║          Window "abc" — all unique ✓                        ║
║                                                              ║
║  When duplicate found:                                       ║
║  Shrink window from left until duplicate is removed         ║
║                                                              ║
║  Use HashSet to track which characters are in the window    ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Algorithm 1: HashSet (Shrink One by One)

```
FUNCTION lengthOfLongestSubstring_v1(s):
    charSet = new HashSet()
    left = 0
    maxLen = 0

    FOR right = 0 TO len(s) - 1:
        WHILE s[right] IN charSet:
            charSet.remove(s[left])
            left++

        charSet.add(s[right])
        maxLen = max(maxLen, right - left + 1)

    RETURN maxLen

Time: O(n)  — each char added/removed at most once
Space: O(min(n, alphabet_size))
```

### Trace

```
s = "abcabcbb"

right=0: char='a', set={}, no dup → add 'a', set={a}, window="a", maxLen=1
right=1: char='b', set={a}, no dup → add 'b', set={a,b}, window="ab", maxLen=2
right=2: char='c', set={a,b}, no dup → add 'c', set={a,b,c}, window="abc", maxLen=3
right=3: char='a', set={a,b,c}, 'a' IS in set!
          remove s[0]='a', left=1, set={b,c}
          now 'a' not in set → add 'a', set={b,c,a}, window="bca", maxLen=3
right=4: char='b', set={b,c,a}, 'b' IS in set!
          remove s[1]='b', left=2, set={c,a}
          now 'b' not in set → add 'b', set={c,a,b}, window="cab", maxLen=3
right=5: char='c', set={c,a,b}, 'c' IS in set!
          remove s[2]='c', left=3, set={a,b}
          now 'c' not in set → add 'c', set={a,b,c}, window="abc", maxLen=3
right=6: char='b', set={a,b,c}, 'b' IS in set!
          remove s[3]='a', left=4, set={b,c}
          'b' still in set!
          remove s[4]='b', left=5, set={c}
          now 'b' not in set → add 'b', set={c,b}, window="cb", maxLen=3
right=7: char='b', set={c,b}, 'b' IS in set!
          remove s[5]='c', left=6, set={b}
          'b' still in set!
          remove s[6]='b', left=7, set={}
          now 'b' not in set → add 'b', set={b}, window="b", maxLen=3

Answer: 3
```

---

## Algorithm 2: HashMap (Jump Left Pointer)

More efficient — instead of shrinking one by one, **jump** left directly past the duplicate:

```
FUNCTION lengthOfLongestSubstring_v2(s):
    charIndex = new HashMap()   // char → last seen index
    left = 0
    maxLen = 0

    FOR right = 0 TO len(s) - 1:
        IF charIndex.containsKey(s[right]):
            // Jump left past the previous occurrence
            left = max(left, charIndex.get(s[right]) + 1)

        charIndex.put(s[right], right)    // Update latest index
        maxLen = max(maxLen, right - left + 1)

    RETURN maxLen

Time: O(n)  — single pass, no inner loop
Space: O(min(n, alphabet_size))
```

### Trace

```
s = "abcabcbb"

right=0: 'a' not in map → map={a:0}, left=0, len=1, max=1
right=1: 'b' not in map → map={a:0,b:1}, left=0, len=2, max=2
right=2: 'c' not in map → map={a:0,b:1,c:2}, left=0, len=3, max=3
right=3: 'a' in map at 0 → left=max(0,0+1)=1
          map={a:3,b:1,c:2}, len=3, max=3
right=4: 'b' in map at 1 → left=max(1,1+1)=2
          map={a:3,b:4,c:2}, len=3, max=3
right=5: 'c' in map at 2 → left=max(2,2+1)=3
          map={a:3,b:4,c:5}, len=3, max=3
right=6: 'b' in map at 4 → left=max(3,4+1)=5
          map={a:3,b:6,c:5}, len=2, max=3
right=7: 'b' in map at 6 → left=max(5,6+1)=7
          map={a:3,b:7,c:5}, len=1, max=3

Answer: 3
```

---

## Why max(left, ...)?

```
╔══════════════════════════════════════════════════════════════╗
║  The max() is crucial to avoid moving LEFT backward!        ║
║                                                              ║
║  Example: s = "abba"                                        ║
║                                                              ║
║  right=0: 'a', map={a:0}, left=0                           ║
║  right=1: 'b', map={a:0,b:1}, left=0                       ║
║  right=2: 'b', found at 1 → left=max(0, 1+1)=2            ║
║           map={a:0,b:2}, left=2                             ║
║  right=3: 'a', found at 0 → left=max(2, 0+1)=2  ← NOT 1! ║
║           Without max: left would go backward to 1          ║
║           including 'b' which has a duplicate!              ║
║                                                              ║
║  max() ensures left only moves FORWARD                      ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Algorithm 3: Array as Map (Fixed Alphabet)

For ASCII characters, use an array of size 128 instead of a HashMap:

```
FUNCTION lengthOfLongestSubstring_v3(s):
    lastSeen = new Array(128, fill=-1)   // ASCII
    left = 0
    maxLen = 0

    FOR right = 0 TO len(s) - 1:
        charCode = s[right].charCodeAt()
        
        IF lastSeen[charCode] >= left:
            left = lastSeen[charCode] + 1
        
        lastSeen[charCode] = right
        maxLen = max(maxLen, right - left + 1)

    RETURN maxLen

Time: O(n)  |  Space: O(1)   ← constant 128-element array
```

---

## Comparison of Approaches

| Approach | Time | Space | Inner Loop? |
|----------|:---:|:---:|:---:|
| HashSet (shrink) | $O(n)$ | $O(k)$ | Yes (while loop) |
| HashMap (jump) | $O(n)$ | $O(k)$ | No |
| Array (ASCII) | $O(n)$ | $O(1)$ | No |

Where $k = \min(n, |\Sigma|)$ and $|\Sigma|$ = alphabet size (26 for lowercase, 128 for ASCII).

---

## Related Problems

| Problem | Hash Structure | Key Idea |
|---------|:---:|---------|
| Longest substring with at most K distinct chars | HashMap (char→count) | Track count per char, shrink when distinct > K |
| Longest substring with at most K replacements | HashMap (char→count) | Window - maxFreq ≤ K |
| Minimum window substring | HashMap (char→count) | Two frequency maps |
| Longest repeating character replacement | HashMap (char→count) | len - maxCount ≤ k |

---

## Quick Revision Question

**Q: In the HashMap approach (Algorithm 2), trace through `s = "tmmzuxt"` and find the length of the longest substring without repeating characters.**

<details>
<summary>Click to reveal answer</summary>

```
s = "tmmzuxt"

right=0: 't', not in map → map={t:0}, left=0, len=1, max=1
right=1: 'm', not in map → map={t:0,m:1}, left=0, len=2, max=2
right=2: 'm', found at 1 → left=max(0,2)=2, map={t:0,m:2}, len=1, max=2
right=3: 'z', not in map → map={t:0,m:2,z:3}, left=2, len=2, max=2
right=4: 'u', not in map → map={t:0,m:2,z:3,u:4}, left=2, len=3, max=3
right=5: 'x', not in map → map={t:0,m:2,z:3,u:4,x:5}, left=2, len=4, max=4
right=6: 't', found at 0 → left=max(2,1)=2 (stays at 2!)
          map={t:6,m:2,z:3,u:4,x:5}, len=5, max=5
```

**Answer: 5** — the substring is "mzuxt" (indices 2-6).

Note how at right=6, the previous 't' was at index 0 which is before `left=2`, so `left` doesn't move — the old 't' is already outside the window.

</details>

---

| [← Previous: Subarray Sum Equals K](05-subarray-sum-k.md) | [Next: Unit 7 — Duplicate Detection →](../07-Hash-Set-Problems/01-duplicate-detection.md) |
|:---|---:|
