# Chapter 3: String Operations

[← Previous: String Storage and Representation](02-string-storage.md) | [Back to README](../README.md) | [Next: Immutability and Its Impact →](04-immutability.md)

---

## Overview

This chapter covers the fundamental operations on strings: access, search, concatenation, substring extraction, insertion, deletion, and comparison. Understanding their complexities is essential for writing efficient string algorithms.

---

## 1. Character Access — O(1)

```
  s = "ALGORITHM"
       0 1 2 3 4 5 6 7 8

  s[0] = 'A'
  s[4] = 'R'
  s[8] = 'M'

  Direct index access: O(1)
  Same as array — jump to base + i × charSize
```

---

## 2. Length — O(1) or O(n)

```
  Length-prefixed (Java, Python, C++):
  len("HELLO") → reads stored field → O(1)

  Null-terminated (C):
  strlen("HELLO") → scans for '\0' → O(n)

  ┌───┬───┬───┬───┬───┬────┐
  │ H │ E │ L │ L │ O │ \0 │  scan until \0...
  └───┴───┴───┴───┴───┴────┘
  count: 1   2   3   4   5      O(n)
```

---

## 3. Concatenation — O(n + m)

```
  s1 = "Hello"  (length n = 5)
  s2 = " World" (length m = 6)
  result = s1 + s2 = "Hello World" (length 11)

  Steps:
  1. Allocate new space for n + m characters
  2. Copy s1: O(n)
  3. Copy s2: O(m)
  Total: O(n + m)

  Memory:
  s1:     [H][e][l][l][o]
  s2:     [ ][W][o][r][l][d]
  result: [H][e][l][l][o][ ][W][o][r][l][d]
              copied ←          copied ←
```

### Repeated Concatenation Trap

```
  Building "abcde" one character at a time:

  s = ""
  s = s + "a"     → copy 1 char      cost: 1
  s = s + "b"     → copy 2 chars     cost: 2
  s = s + "c"     → copy 3 chars     cost: 3
  s = s + "d"     → copy 4 chars     cost: 4
  s = s + "e"     → copy 5 chars     cost: 5

  Total: 1 + 2 + 3 + 4 + 5 = 15 = O(n²)

  Fix: Use StringBuilder or list.join()
```

---

## 4. Substring Extraction — O(k)

```
  s = "ALGORITHM"
       0 1 2 3 4 5 6 7 8

  substring(s, 2, 5) → "GORI"  (indices 2 to 5)

  Steps:
  1. Allocate space for k characters (k = end - start + 1)
  2. Copy characters from start to end

  Time: O(k) where k = substring length

  Some languages (old Java) returned a VIEW → O(1)
  but this caused memory leaks, so now they COPY → O(k)
```

---

## 5. Search / Find — O(n × m) Brute Force

```
  text = "HELLO WORLD"
  pattern = "WORLD"

  Brute force: try every starting position
  
  H E L L O   W O R L D
  W O R L D ✗
    W O R L D ✗
      W O R L D ✗
        W O R L D ✗
          W O R L D ✗
            W O R L D ✓ Found at index 6!

  Time: O((n - m + 1) × m) = O(n × m)
  Better algorithms: KMP O(n+m), Rabin-Karp O(n+m) avg
```

---

## 6. Comparison — O(min(n, m))

```
  Compare "apple" vs "application":

  a = a  ✓
  p = p  ✓
  p = p  ✓
  l = l  ✓
  e ≠ i  → 'e' < 'i' → "apple" < "application"

  Lexicographic (dictionary) order:
  Compare character by character until mismatch or end.

  Rules:
  1. Compare characters left to right
  2. First mismatch determines order
  3. If one string ends first, shorter < longer
     "app" < "apple"
```

### Pseudocode

```
FUNCTION compare(s1, s2):
    i = 0
    WHILE i < len(s1) AND i < len(s2):
        IF s1[i] < s2[i]: RETURN -1    // s1 < s2
        IF s1[i] > s2[i]: RETURN +1    // s1 > s2
        i = i + 1
    
    IF len(s1) < len(s2): RETURN -1
    IF len(s1) > len(s2): RETURN +1
    RETURN 0                            // equal
```

---

## 7. Insertion — O(n)

```
  s = "HELO"
  Insert 'L' at index 2:

  Before: [H][E][L][O]
                ↑ insert here

  1. Shift right from index 2:
          [H][E][ ][L][O]
  2. Place 'L':
          [H][E][L][L][O]

  Time: O(n) — need to shift elements right

  With immutable strings: create new string → O(n)
```

---

## 8. Deletion — O(n)

```
  s = "HELLLO"
  Delete character at index 3:

  Before: [H][E][L][L][L][O]
                    ↑ delete

  After:  [H][E][L][L][O]
  
  Shift left from index 4: O(n)
  With immutable strings: create new string → O(n)
```

---

## 9. Replace — O(n)

```
  s = "HELLO"
  Replace 'L' with 'R' (all occurrences):

  [H][E][L][L][O]
          ↓   ↓
  [H][E][R][R][O]

  Single character: O(n) scan
  Substring replace: O(n × m) naive, O(n) with smart algorithms
```

---

## 10. Reverse — O(n)

```
  s = "HELLO"

  Two-pointer swap (mutable string):
  [H][E][L][L][O]
   ↑           ↑    swap H↔O
  [O][E][L][L][H]
      ↑   ↑         swap E↔L
  [O][L][L][E][H]
         ↑           middle, stop
  
  Result: "OLLEH"
  
  Immutable: create new string, write backwards → O(n)
```

---

## Operations Summary

```
  ┌───────────────────┬───────────────┬─────────────────┐
  │ Operation         │ Mutable       │ Immutable       │
  ├───────────────────┼───────────────┼─────────────────┤
  │ Access s[i]       │ O(1)          │ O(1)            │
  │ Length            │ O(1)          │ O(1)            │
  │ Concatenate       │ O(m) append   │ O(n+m) new str  │
  │ Substring         │ O(k)          │ O(k)            │
  │ Search            │ O(n×m)        │ O(n×m)          │
  │ Compare           │ O(min(n,m))   │ O(min(n,m))     │
  │ Insert            │ O(n)          │ O(n) new str    │
  │ Delete            │ O(n)          │ O(n) new str    │
  │ Replace (all)     │ O(n)          │ O(n) new str    │
  │ Reverse           │ O(n)          │ O(n) new str    │
  └───────────────────┴───────────────┴─────────────────┘
```

---

## Summary Table

| Operation | Time Complexity | Notes |
|-----------|----------------|-------|
| Access | O(1) | Direct indexing |
| Length | O(1)* | O(n) for C strings |
| Concatenation | O(n + m) | Both strings copied |
| Substring | O(k) | k = extracted length |
| Search | O(n × m) | Brute force; O(n+m) with KMP |
| Compare | O(min(n, m)) | Lexicographic order |
| Insert/Delete | O(n) | Shift required |
| Reverse | O(n) | In-place with two pointers |

---

## Quick Revision Questions

1. **Why is repeated concatenation O(n²)?** How can you avoid this in Python? In Java?

2. **What is the time complexity of comparing two strings of length n?** When can it be O(1)?

3. **How does substring extraction work?** Why don't modern JVMs share the backing array?

4. **Explain lexicographic comparison.** How do you handle strings of different lengths?

5. **Why is string insertion O(n)?** How does immutability make this worse?

6. **What is the best algorithm for string search?** How does it improve over brute force?

---

[← Previous: String Storage and Representation](02-string-storage.md) | [Back to README](../README.md) | [Next: Immutability and Its Impact →](04-immutability.md)
