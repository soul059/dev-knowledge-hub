# Chapter 4: Immutability and Its Impact

[← Previous: String Operations](03-string-operations.md) | [Back to README](../README.md) | [Next: Character Encoding →](05-character-encoding.md)

---

## Overview

In languages like Java, Python, JavaScript, C#, and Go, strings are **immutable** — once created, their contents cannot be changed. This design choice has profound impacts on performance, safety, and how we write string algorithms.

---

## What Is Immutability?

```
  Mutable:   you CAN change the contents after creation
  Immutable: you CANNOT change the contents after creation

  Mutable (C):
  char s[] = "HELLO";
  s[0] = 'J';           // OK → "JELLO"

  Immutable (Python):
  s = "HELLO"
  s[0] = 'J'            // ERROR! TypeError

  Immutable (Java):
  String s = "HELLO";
  s.charAt(0) = 'J';    // Compile error
  // Must create new: s = "J" + s.substring(1);
```

---

## Why Immutability?

```
  ┌─────────────────────────────────────────────────┐
  │              BENEFITS OF IMMUTABILITY            │
  ├─────────────────────────────────────────────────┤
  │                                                 │
  │  1. THREAD SAFETY                               │
  │     No locks needed — can't be modified!        │
  │     Multiple threads can share safely.          │
  │                                                 │
  │  2. HASH CACHING                                │
  │     Hash computed once, cached forever.          │
  │     HashMap key lookups stay O(1).              │
  │                                                 │
  │  3. SECURITY                                    │
  │     Passwords, URLs, class names can't be       │
  │     tampered with after creation.               │
  │                                                 │
  │  4. STRING POOLING                              │
  │     Safe to share references (interning).       │
  │     Saves memory.                               │
  │                                                 │
  │  5. PREDICTABILITY                              │
  │     Function receives string → guaranteed       │
  │     unchanged. No defensive copying needed.     │
  │                                                 │
  └─────────────────────────────────────────────────┘
```

---

## The Cost of Immutability

```
  Every "modification" creates a NEW string object.

  s = "HELLO"
  s = s + "!"       ← creates brand new "HELLO!" → O(n)
  
  The original "HELLO" is untouched (eventually garbage collected).

  ┌─────────┐           ┌──────────┐
  │ "HELLO" │  ──add!→  │ "HELLO!" │  (new allocation)
  └─────────┘           └──────────┘
       ↑                      ↑
    old object           new object
    (orphaned)           (s now points here)
```

---

## The O(n²) Concatenation Problem

```
  Building a string character by character:

  result = ""
  FOR each char c in input:
      result = result + c     ← O(len(result)) each time!

  Step 1: "" + "a"        → copy 1 char
  Step 2: "a" + "b"       → copy 2 chars
  Step 3: "ab" + "c"      → copy 3 chars
  ...
  Step n: copy n chars

  Total: 1 + 2 + 3 + ... + n = n(n+1)/2 = O(n²)

  ┌──────────────────────────────────────────────┐
  │  For n = 100,000:                             │
  │  Immutable concat: ~5 billion char copies     │
  │  StringBuilder:    ~100,000 char copies       │
  │                                               │
  │  That's 50,000× slower!                       │
  └──────────────────────────────────────────────┘
```

---

## Solutions: Mutable Builders

### Java: StringBuilder

```
  StringBuilder sb = new StringBuilder();
  sb.append("Hello");
  sb.append(" ");
  sb.append("World");
  String result = sb.toString();  // "Hello World"

  Internal: dynamic char array, doubles when full
  Amortized O(1) per append → O(n) total

  ┌───────────────────────────────────────┐
  │ StringBuilder (capacity: 16)          │
  │ [H][e][l][l][o][ ][W][o][r][l][d]    │
  │  0  1  2  3  4  5  6  7  8  9  10    │
  │ length: 11                            │
  └───────────────────────────────────────┘
```

### Python: list + join

```
  parts = []
  parts.append("Hello")    # O(1) amortized
  parts.append(" ")
  parts.append("World")
  result = "".join(parts)   # O(n) single pass

  # Much better than: result += "Hello" + " " + "World"
```

### C++: std::string (mutable!)

```
  std::string s = "Hello";
  s += " World";    // modifies in place, O(m)
  // C++ strings ARE mutable, so this is fine
```

---

## Impact on Algorithm Design

### Example: Reverse a String

```
  IMMUTABLE approach (Python/Java):
  
  // BAD - O(n²):
  reversed = ""
  for i = n-1 downto 0:
      reversed = reversed + s[i]     // O(i) copy each time!

  // GOOD - O(n):
  chars = list(s)          // convert to mutable array
  reverse chars in-place   // two-pointer swap
  result = "".join(chars)  // join back to string

  // Or simply: reversed = s[::-1] in Python
```

### Example: Remove Duplicates

```
  // BAD approach with immutable strings:
  result = ""
  seen = set()
  for c in s:
      if c not in seen:
          result = result + c    // O(n) copy each time → O(n²)
          seen.add(c)

  // GOOD approach:
  result = []
  seen = set()
  for c in s:
      if c not in seen:
          result.append(c)       // O(1) amortized
          seen.add(c)
  return "".join(result)          // O(n) final join
```

---

## String Interning and Immutability

```
  Immutability ENABLES safe interning:

  s1 = "hello"  ──→  ┌───────────┐
                       │  "hello"  │  ← shared object
  s2 = "hello"  ──→  └───────────┘

  If strings were mutable:
  s1[0] = 'J'   → would change BOTH s1 and s2!
  That's why mutable strings can't be safely interned.
```

---

## Hash Caching

```
  Immutable string → hash never changes → compute once

  Java String:
  ┌──────────────────────────┐
  │ value: ['H','i']         │
  │ hash:  0 (not computed)  │  ← computed lazily
  └──────────────────────────┘

  First hashCode() call:
  hash = 31 * 'H' + 'i' = 31 * 72 + 105 = 2337

  ┌──────────────────────────┐
  │ value: ['H','i']         │
  │ hash:  2337 (cached)     │  ← reused forever
  └──────────────────────────┘

  This makes strings excellent HashMap keys!
```

---

## When to Use What

```
  ┌──────────────────────────────────────────────────┐
  │  DECISION TREE                                    │
  │                                                   │
  │  Building string from parts?                      │
  │    YES → StringBuilder / list.join()              │
  │    NO  → regular string operations are fine       │
  │                                                   │
  │  Modifying characters in place?                   │
  │    YES → convert to char array first              │
  │    NO  → use string directly                      │
  │                                                   │
  │  Many small reads/comparisons?                    │
  │    → Immutable strings are GREAT (cached hash!)   │
  │                                                   │
  │  Performance-critical character manipulation?     │
  │    → Use char array throughout, convert at end    │
  └──────────────────────────────────────────────────┘
```

---

## Language Comparison

| Language | String Type | Mutable? | Builder | Conversion |
|----------|------------|----------|---------|------------|
| C | char[] | Yes | N/A | N/A |
| C++ | std::string | Yes | N/A | N/A |
| Java | String | No | StringBuilder | toCharArray() |
| Python | str | No | list + join | list(s) |
| JavaScript | string | No | Array + join | s.split('') |
| C# | string | No | StringBuilder | ToCharArray() |
| Go | string | No | strings.Builder | []byte(s) |
| Rust | &str | No | String (owned) | .to_string() |

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Immutability | String contents cannot change after creation |
| Benefit | Thread safety, hash caching, security, interning |
| Cost | Every "edit" creates a new string O(n) |
| Concat trap | n concatenations = O(n²) with immutable strings |
| Solution | StringBuilder (Java), list+join (Python) |
| Algorithm tip | Convert to char array, manipulate, convert back |

---

## Quick Revision Questions

1. **Why are strings immutable in Java?** List at least three benefits.

2. **Show the O(n²) problem** with building "abcde" via concatenation. Calculate exact cost.

3. **How does StringBuilder solve the concatenation problem?** What is its amortized complexity?

4. **Why does immutability enable string interning?** What would go wrong with mutable interned strings?

5. **In Python, what is the idiomatic way** to build a string from parts? Why not `+=`?

6. **How does hash caching work** with immutable strings? Why can't mutable strings cache their hash?

---

[← Previous: String Operations](03-string-operations.md) | [Back to README](../README.md) | [Next: Character Encoding →](05-character-encoding.md)
