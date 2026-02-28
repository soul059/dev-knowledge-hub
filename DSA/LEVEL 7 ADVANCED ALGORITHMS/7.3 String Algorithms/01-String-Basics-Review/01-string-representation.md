# Chapter 1.1 — String Representation

> **Unit 1: String Basics Review** | [Course Home](../README.md)

---

## 📋 Chapter Overview

Before diving into algorithms, we must understand how strings are **stored**,
**accessed**, and **measured** at the hardware level. This chapter covers the
internal representation of strings across languages and the cost model that
drives every complexity analysis later on.

---

## 1. What Is a String?

A **string** is a finite sequence of characters drawn from an **alphabet** Σ.

```
                     Alphabet (Σ)
                     ┌───────────┐
    "hello"  ──►     │ a-z, A-Z  │
    "42"     ──►     │ 0-9       │
    "a+b=c"  ──►     │ +, =, ... │
                     └───────────┘

    Formally:  s = s[0] s[1] s[2] ... s[n-1]
               where each s[i] ∈ Σ  and  n = |s|  (length)
```

### Key Definitions

| Term | Symbol | Meaning |
|------|--------|---------|
| Length | `|s|` or `n` | Number of characters |
| Empty string | `ε` or `""` | String with `|s| = 0` |
| Alphabet | Σ | Set of valid characters |
| Alphabet size | `|Σ|` | e.g., 26 for lowercase English |

---

## 2. Memory Layout

### 2.1 Character Array (C / C++)

```
  Index:     0     1     2     3     4     5
           ┌─────┬─────┬─────┬─────┬─────┬─────┐
  char[]:  │ 'h' │ 'e' │ 'l' │ 'l' │ 'o' │ '\0'│   ◄── null terminator
           └─────┴─────┴─────┴─────┴─────┴─────┘
  Address: 1000  1001  1002  1003  1004  1005

  ► Each cell = 1 byte (ASCII)
  ► Total memory = n + 1 bytes  (the +1 is for '\0')
```

💡 **Key Insight**: In C, the null terminator `'\0'` marks the end. Forgetting
it is one of the most common bugs.

### 2.2 Length-Prefixed (Pascal-style / Java / Python)

```
           ┌────────┬─────┬─────┬─────┬─────┬─────┐
           │ len=5  │ 'h' │ 'e' │ 'l' │ 'l' │ 'o' │
           └────────┴─────┴─────┴─────┴─────┴─────┘
                ▲
                │
           Length stored explicitly
           ► strlen() is O(1), not O(n)
```

### 2.3 Immutable vs Mutable

| Language | Type | Mutable? | Implication |
|----------|------|----------|-------------|
| C | `char[]` | Yes | Direct modification possible |
| C++ | `std::string` | Yes | Copy-on-write (some impls) |
| Java | `String` | **No** | Concatenation creates new object |
| Python | `str` | **No** | Concatenation creates new object |
| Java | `StringBuilder` | Yes | Use for repeated concatenation |

⚠️ **Common Pitfall**: In Java/Python, repeated `s += c` inside a loop is
**O(n²)** because each concatenation creates a new string.

```
  WRONG (O(n²)):                CORRECT (O(n)):
  ─────────────                 ──────────────
  s = ""                        parts = []
  for c in data:                for c in data:
      s += c      ◄── new          parts.append(c)
                   object       s = "".join(parts)
```

---

## 3. Character Encoding

### ASCII (7-bit / 128 characters)

```
  ┌──────────┬───────────┬──────────────┐
  │  Range   │   Chars   │   Example    │
  ├──────────┼───────────┼──────────────┤
  │  0 - 31  │  Control  │  \n, \t      │
  │ 32 - 47  │  Symbols  │  !, @, #     │
  │ 48 - 57  │  Digits   │  '0' - '9'  │
  │ 65 - 90  │  A - Z    │  uppercase   │
  │ 97 - 122 │  a - z    │  lowercase   │
  └──────────┴───────────┴──────────────┘

  💡 'a' - 'A' = 32   →  toggle case by XOR with 32
  💡 c - '0'          →  digit character to integer
  💡 c - 'a'          →  letter to 0-based index
```

### Unicode & UTF-8

```
  Bytes   Bit-pattern              Range
  ─────   ─────────────────────    ──────────────
    1     0xxxxxxx                 U+0000 – U+007F  (ASCII)
    2     110xxxxx 10xxxxxx        U+0080 – U+07FF
    3     1110xxxx 10xxxxxx ×2     U+0800 – U+FFFF
    4     11110xxx 10xxxxxx ×3     U+10000 – U+10FFFF
```

🔑 **For contest problems**, assume ASCII unless stated otherwise. Alphabet
size |Σ| = 26 (lowercase English) is the most common constraint.

---

## 4. Indexing & Access

| Operation | Array-Based | Linked-List |
|-----------|-------------|-------------|
| Access `s[i]` | **O(1)** | O(n) |
| Length | **O(1)** (stored) | O(n) or O(1) |
| Concatenate | **O(n+m)** | O(1) for append |

Strings in all major languages use **contiguous arrays** → random access is
O(1).

```
  s = "algorithm"

  s[0] = 'a'
  s[4] = 'r'
  s[8] = 'm'

       0   1   2   3   4   5   6   7   8
     ┌───┬───┬───┬───┬───┬───┬───┬───┬───┐
     │ a │ l │ g │ o │ r │ i │ t │ h │ m │
     └───┴───┴───┴───┴───┴───┴───┴───┴───┘
```

---

## 5. Common Representations in Algorithms

### 5.1 String as Integer Index (Hashing)

```
  Map each character to a number:
    'a' → 0,  'b' → 1,  ...,  'z' → 25

  Then a string maps to a polynomial:
    "abc" → 0 × 26² + 1 × 26¹ + 2 × 26⁰ = 28
```

### 5.2 Bitmask Representation (for sets of chars)

```
  "abc" → set {a, b, c}
        → bitmask: ...0000_0111  (bits 0, 1, 2 set)

  "az"  → set {a, z}
        → bitmask: ...0010_0000_0000_0000_0000_0000_0001
                       bit 25                     bit 0
```

💡 This is used in problems like "find all substrings that are anagrams" and
can represent character presence in **O(1)** per check.

---

## 6. Real-World Applications

| Application | Relevance |
|-------------|-----------|
| Text editors | Rope data structure for large mutable strings |
| Databases | VARCHAR, CHAR types – fixed vs variable length |
| Networking | Packet parsing, protocol headers (byte strings) |
| Compression | Strings represented as sequences of bits |

---

## 📝 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Definition | Finite sequence of chars from alphabet Σ |
| C strings | Null-terminated, `strlen` is O(n) |
| Java/Python strings | Immutable, length is O(1) |
| Concatenation trap | `s += c` in a loop → O(n²) |
| ASCII trick | `c - 'a'` gives 0-based index |
| Bitmask | 26 bits can represent a character set |
| Random access | O(1) for array-based strings |

---

## ❓ Quick Revision Questions

1. **Why is repeated string concatenation O(n²) in Java/Python?**
   <details><summary>Answer</summary>Each concatenation creates a new string object and copies all existing characters plus the new one.</details>

2. **How many bytes does the C string `"hello"` occupy in memory?**
   <details><summary>Answer</summary>6 bytes — 5 characters + 1 null terminator `'\0'`.</details>

3. **What is the time complexity of `s[i]` access for a typical string?**
   <details><summary>Answer</summary>O(1) — strings are stored as contiguous arrays.</details>

4. **How do you convert a character `'5'` to the integer 5?**
   <details><summary>Answer</summary>`'5' - '0'` = 53 - 48 = 5 (using ASCII values).</details>

5. **What is the advantage of a bitmask representation for a character set?**
   <details><summary>Answer</summary>O(1) set operations (union via OR, intersection via AND, membership via AND + check).</details>

6. **In UTF-8, how many bytes does the character 'A' take?**
   <details><summary>Answer</summary>1 byte — it falls in the ASCII range (U+0041).</details>

---

| [⬅️ Course Home](../README.md) | [Next: String Operations ➡️](02-string-operations.md) |
|:---|---:|
