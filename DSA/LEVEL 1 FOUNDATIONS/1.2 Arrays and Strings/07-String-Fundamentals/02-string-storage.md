# Chapter 2: String Storage and Representation

[← Previous: What Is a String](01-what-is-a-string.md) | [Back to README](../README.md) | [Next: String Operations →](03-string-operations.md)

---

## Overview

How a string is stored in memory profoundly affects performance. This chapter covers null-terminated strings, length-prefixed strings, rope data structures, and interning — the techniques that underpin every string library.

---

## 1. Null-Terminated Strings (C-Style)

```
  s = "HELLO"

  Memory:
  Address:  100  101  102  103  104  105
           ┌────┬────┬────┬────┬────┬────┐
           │ H  │ E  │ L  │ L  │ O  │ \0 │
           │ 72 │ 69 │ 76 │ 76 │ 79 │  0 │
           └────┴────┴────┴────┴────┴────┘
                                      ↑
                              null terminator

  • The null character '\0' (value 0) marks the end
  • Need n+1 bytes for a string of length n
  • Functions scan until they find '\0'
```

### Operations

```
  strlen("HELLO"):
  Start at index 0, count until '\0'
  H → E → L → L → O → \0  (stop!)
  Length = 5
  Time: O(n)  ← must scan entire string!

  strcat(dest, src):
  1. Find end of dest: O(len(dest))
  2. Copy src there: O(len(src))
  Total: O(len(dest) + len(src))

  Repeated concatenation: O(n²) due to rescanning!
```

### Dangers

```
  Buffer overflow:
  char buf[5];
  strcpy(buf, "HELLO");  // Needs 6 bytes! (5 + '\0')
  
  ┌───┬───┬───┬───┬───┬───┐
  │ H │ E │ L │ L │ O │\0 │  ← writes beyond buf!
  └───┴───┴───┴───┴───┴───┘
                        ↑
                  outside allocated memory
```

---

## 2. Length-Prefixed Strings

```
  Modern approach: store length separately

  ┌────────┬───┬───┬───┬───┬───┐
  │ len=5  │ H │ E │ L │ L │ O │
  └────────┴───┴───┴───┴───┴───┘
     metadata      data

  Java String (simplified):
  ┌────────────────────────┐
  │ String object          │
  │ ┌─────────────┐        │
  │ │ length: 5   │        │
  │ │ value: ─────┼──→ ['H','E','L','L','O']
  │ │ hash: ...   │        │
  │ └─────────────┘        │
  └────────────────────────┘

  Python str:
  ┌─────────────────────────────┐
  │ PyObject header             │
  │ length: 5                   │
  │ hash: (cached)              │
  │ data: H E L L O             │
  └─────────────────────────────┘
```

### Advantages

| Feature | Null-terminated | Length-prefixed |
|---------|----------------|-----------------|
| Length query | O(n) | O(1) |
| Can contain '\0' | No | Yes |
| Buffer safety | Manual | Automatic |
| Memory overhead | +1 byte | +4-8 bytes |
| Substring check | Scan-based | Direct |

---

## 3. Memory Layout: Contiguous Characters

```
  Like arrays, string characters are stored contiguously:

  s = "CODE"

  Base address = 0x1000
  ┌──────────┬──────────┬──────────┬──────────┐
  │ 0x1000   │ 0x1001   │ 0x1002   │ 0x1003   │
  │   'C'    │   'O'    │   'D'    │   'E'    │
  │   67     │   79     │   68     │   69     │
  └──────────┴──────────┴──────────┴──────────┘

  Address of s[i] = base + i × char_size

  For ASCII: char_size = 1 byte
  For UTF-16: char_size = 2 bytes
  For UTF-32: char_size = 4 bytes
```

---

## 4. String Interning

```
  String interning: reuse identical string objects

  Without interning:
  s1 = "hello"    →  0x1000: "hello"
  s2 = "hello"    →  0x2000: "hello"  (separate copy!)

  With interning:
  s1 = "hello"    →  0x1000: "hello"
  s2 = "hello"    →  0x1000: "hello"  (same object!)
                         ↑
                    intern pool

  Benefits:
  • Memory savings for repeated strings
  • O(1) equality check (pointer comparison)
  • Used by Java (string pool), Python (small strings)
```

### Java Example

```
  String s1 = "hello";     // from string pool
  String s2 = "hello";     // same object from pool
  s1 == s2                 // TRUE (same reference)

  String s3 = new String("hello");  // new object
  s1 == s3                          // FALSE (different reference)
  s1.equals(s3)                     // TRUE (same content)
```

---

## 5. StringBuilder / StringBuffer

```
  Problem: immutable strings make repeated concatenation O(n²)

  "a" + "b" → creates "ab"
  "ab" + "c" → creates "abc"  (copies "ab" again!)
  "abc" + "d" → creates "abcd" (copies "abc" again!)

  Total copies: 1 + 2 + 3 + ... + n = O(n²)

  Solution: mutable buffer that grows like a dynamic array

  StringBuilder:
  ┌────────────────────────────────────────┐
  │ capacity: 16    length: 0              │
  │ [  ][  ][  ][  ][  ][  ][  ]...[  ]   │
  └────────────────────────────────────────┘
  
  append("Hello"):
  ┌────────────────────────────────────────┐
  │ capacity: 16    length: 5              │
  │ [H][E][L][L][O][  ][  ]...[  ]        │
  └────────────────────────────────────────┘

  append(" World"):
  ┌────────────────────────────────────────┐
  │ capacity: 16    length: 11             │
  │ [H][E][L][L][O][ ][W][O][R][L][D]     │
  └────────────────────────────────────────┘

  toString() → creates final immutable string
  
  Total cost: O(n) amortized!
```

---

## 6. Rope Data Structure

```
  For very large strings (text editors), ropes are used:

  A rope is a balanced binary tree of string fragments.

  String: "Hello World"

          ┌───────────┐
          │   11      │  (total length)
          └─────┬─────┘
          ┌─────┴─────┐
     ┌────┴───┐  ┌────┴───┐
     │ "Hello"│  │" World"│
     │  (5)   │  │  (6)   │
     └────────┘  └────────┘

  Operations:
  • Concatenation: O(1) — just create new root node
  • Split: O(log n) — split at any position
  • Insert: O(log n) — split + concatenate
  • Access char: O(log n) — traverse tree
  • Used by: text editors, IDEs, very large documents
```

---

## Comparison of Storage Methods

| Method | strlen | concat | index | insert | Use Case |
|--------|--------|--------|-------|--------|----------|
| Null-terminated | O(n) | O(n+m) | O(1) | O(n) | C, low-level |
| Length-prefixed | O(1) | O(n+m) | O(1) | O(n) | Java, Python |
| StringBuilder | O(1) | O(1)* | O(1) | O(n) | Building strings |
| Rope | O(1)† | O(1) | O(log n) | O(log n) | Text editors |

*amortized †stored at root

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Null-terminated | C-style, '\0' marker, O(n) length |
| Length-prefixed | Modern, O(1) length, safer |
| Contiguous storage | Characters at consecutive addresses |
| String interning | Reuse identical strings, O(1) equality |
| StringBuilder | Mutable buffer, O(n) total for n appends |
| Rope | Tree of fragments, O(log n) insert/split |

---

## Quick Revision Questions

1. **Why is finding the length of a C string O(n)?** How do modern languages avoid this?

2. **What is string interning?** Give an example where it helps.

3. **Why is repeated string concatenation O(n²) with immutable strings?** How does StringBuilder fix this?

4. **When would you use a rope instead of a regular string?** What are its trade-offs?

5. **Draw the memory layout** of "DSA" as both null-terminated and length-prefixed.

6. **Why can't a null-terminated string contain the character '\0'?**

---

[← Previous: What Is a String](01-what-is-a-string.md) | [Back to README](../README.md) | [Next: String Operations →](03-string-operations.md)
