# Chapter 1: What Is a String

[← Previous: Kadane's Variations](../06-Kadanes-Algorithm/06-kadanes-variations.md) | [Back to README](../README.md) | [Next: String Storage and Representation →](02-string-storage.md)

---

## Overview

A **string** is a sequence of characters. It is one of the most fundamental data types in computing — every program that deals with text, user input, file paths, URLs, or DNA sequences uses strings.

---

## Definition

```
  A string is a finite, ordered sequence of characters
  drawn from some alphabet (character set).

  s = "HELLO"

  ┌───┬───┬───┬───┬───┐
  │ H │ E │ L │ L │ O │   length = 5
  └───┴───┴───┴───┴───┘
    0   1   2   3   4      (zero-indexed)
```

### Formal Definition

$$s = c_0 c_1 c_2 \ldots c_{n-1} \quad \text{where each } c_i \in \Sigma$$

where $\Sigma$ is an alphabet (e.g., ASCII, Unicode) and $n$ is the length.

---

## Strings as Arrays of Characters

```
  At the lowest level, a string IS an array of characters.

  String "CAT":
  ┌─────┬─────┬─────┐
  │ 'C' │ 'A' │ 'T' │   char array
  │ 67  │ 65  │ 84  │   ASCII values
  └─────┴─────┴─────┘
    [0]   [1]   [2]

  Everything we learned about arrays applies to strings!
  - Index access: O(1)
  - Sequential storage in memory
  - Length tracking
```

---

## Key Terminology

```
  String: "ABCDEF"

  ┌───┬───┬───┬───┬───┬───┐
  │ A │ B │ C │ D │ E │ F │
  └───┴───┴───┴───┴───┴───┘

  Substring:   contiguous portion
               "BCD" is a substring (indices 1..3)
               "ACE" is NOT a substring (not contiguous)

  Subsequence: ordered but not necessarily contiguous
               "ACE" IS a subsequence
               "ECA" is NOT a subsequence (wrong order)

  Prefix:      substring starting at index 0
               "ABC" is a prefix

  Suffix:      substring ending at last index
               "DEF" is a suffix

  Empty string: "" (length 0) — often denoted ε
```

### Visual

```
  "ABCDEF"

  Prefixes:          Suffixes:
  ""                 ""
  "A"                "F"
  "AB"               "EF"
  "ABC"              "DEF"
  "ABCD"             "CDEF"
  "ABCDE"            "BCDEF"
  "ABCDEF"           "ABCDEF"

  Substrings of "ABC":
  "", "A", "B", "C", "AB", "BC", "ABC"
  Count: n(n+1)/2 + 1 = 3(4)/2 + 1 = 7
```

---

## String vs Character

```
  Character:   'A'    — single symbol, constant size
  String:      "A"    — sequence of length 1
  
  'A' ≠ "A"  in many languages!

  'A' → a single byte (or code unit)
  "A" → may include metadata (length, null terminator, etc.)

  In C:    'A' is an int (value 65)
           "A" is a char array {'A', '\0'}

  In Java: 'A' is char (primitive)
           "A" is String (object)
```

---

## Common String Operations

| Operation | Description | Typical Complexity |
|-----------|-------------|-------------------|
| Access `s[i]` | Get character at index | O(1) |
| Length | Number of characters | O(1)* |
| Concatenation | Join two strings | O(n + m) |
| Substring | Extract portion | O(k) where k = length |
| Search | Find pattern in text | O(n × m) brute force |
| Compare | Lexicographic comparison | O(min(n, m)) |
| Replace | Substitute characters | O(n) |
| Reverse | Reverse the string | O(n) |

*O(1) with stored length; O(n) with null-terminated strings in C.

---

## Null-Terminated vs Length-Prefixed

```
  C-style (null-terminated):
  ┌───┬───┬───┬────┐
  │ H │ I │ ! │ \0 │   \0 marks the end
  └───┴───┴───┴────┘
  - Finding length requires scanning: O(n)
  - Can't contain \0 in the string
  - Buffer overflow risk

  Length-prefixed (Pascal, most modern languages):
  ┌───┬───┬───┬───┐
  │ 3 │ H │ I │ ! │   length stored separately
  └───┴───┴───┴───┘
  - Length is O(1)
  - Can contain any character
  - Safer
```

---

## Strings in Different Languages

```
  C:       char str[] = "hello";      // null-terminated array
  C++:     std::string s = "hello";   // dynamic, length-stored
  Java:    String s = "hello";        // immutable object
  Python:  s = "hello"                // immutable sequence
  JS:      let s = "hello";           // immutable primitive

  Key difference: MUTABILITY
  ┌────────────┬──────────┬────────────────────────┐
  │ Language   │ Mutable? │ Modification approach   │
  ├────────────┼──────────┼────────────────────────┤
  │ C          │ Yes      │ Direct char access      │
  │ C++        │ Yes      │ Direct or methods       │
  │ Java       │ No       │ Create new String       │
  │ Python     │ No       │ Create new string       │
  │ JavaScript │ No       │ Create new string       │
  └────────────┴──────────┴────────────────────────┘
```

---

## Real-World Applications

```
  ┌──────────────────────────────────────┐
  │  Where strings appear:               │
  │                                      │
  │  • Text processing / editors         │
  │  • Search engines (indexing, query)  │
  │  • DNA/RNA sequences (bioinformatics)│
  │  • URLs, file paths                  │
  │  • Network protocols                 │
  │  • Compilers (parsing source code)   │
  │  • Databases (text fields)           │
  │  • Cryptography (hashing messages)   │
  └──────────────────────────────────────┘
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| String | Ordered sequence of characters |
| Relation to arrays | String = array of characters |
| Substring | Contiguous portion of string |
| Subsequence | Ordered but not necessarily contiguous |
| Prefix | Starts at index 0 |
| Suffix | Ends at last index |
| Mutability | Varies by language (Java/Python: immutable) |

---

## Quick Revision Questions

1. **How is a string related to an array?** What operations carry over?

2. **What is the difference between a substring and a subsequence?** Give an example.

3. **How many substrings does a string of length n have?** Include the empty string.

4. **What is the difference between null-terminated and length-prefixed strings?** Which is faster for finding length?

5. **Why are strings immutable in Java and Python?** What are the trade-offs?

6. **Is "ACE" a substring or subsequence of "ABCDEF"?** Explain.

---

[← Previous: Kadane's Variations](../06-Kadanes-Algorithm/06-kadanes-variations.md) | [Back to README](../README.md) | [Next: String Storage and Representation →](02-string-storage.md)
