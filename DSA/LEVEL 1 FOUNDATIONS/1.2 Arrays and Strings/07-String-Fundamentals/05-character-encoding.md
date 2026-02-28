# Chapter 5: Character Encoding

[← Previous: Immutability and Its Impact](04-immutability.md) | [Back to README](../README.md) | [Next: String Comparison →](06-string-comparison.md)

---

## Overview

Characters are stored as numbers. **Character encoding** is the mapping between characters and their numeric representations. Understanding encoding is critical for handling international text, avoiding bugs, and writing correct string algorithms.

---

## ASCII

```
  ASCII = American Standard Code for Information Interchange
  
  Uses 7 bits → 128 characters (0-127)

  ┌───────────────────────────────────────────────┐
  │ Range     │ Characters                        │
  ├───────────┼───────────────────────────────────┤
  │  0 - 31   │ Control chars (newline, tab, etc.)│
  │    32      │ Space                             │
  │ 33 - 47   │ Punctuation ( ! " # $ % & ' etc.) │
  │ 48 - 57   │ Digits (0-9)                      │
  │ 58 - 64   │ More punctuation (: ; < = > ? @)  │
  │ 65 - 90   │ Uppercase A-Z                     │
  │ 91 - 96   │ Brackets and more ([ \ ] ^ _ `)   │
  │ 97 - 122  │ Lowercase a-z                     │
  │ 123 - 127 │ Braces and DEL ({ | } ~ DEL)      │
  └───────────┴───────────────────────────────────┘
```

### Key ASCII Values to Know

```
  '0' = 48        'A' = 65        'a' = 97
  '1' = 49        'B' = 66        'b' = 98
  ...             ...             ...
  '9' = 57        'Z' = 90        'z' = 122

  Useful relationships:
  • digit value = char - '0'     ('5' - '0' = 53 - 48 = 5)
  • uppercase = char - 32        ('a' - 32 = 97 - 32 = 65 = 'A')
  • lowercase = char + 32        ('A' + 32 = 65 + 32 = 97 = 'a')
  • isLetter?   'A' ≤ c ≤ 'Z' or 'a' ≤ c ≤ 'z'
  • isDigit?    '0' ≤ c ≤ '9'
```

### ASCII Table (Printable)

```
  32  SP   48  0   64  @   80  P   96   `   112  p
  33  !    49  1   65  A   81  Q   97   a   113  q
  34  "    50  2   66  B   82  R   98   b   114  r
  35  #    51  3   67  C   83  S   99   c   115  s
  36  $    52  4   68  D   84  T   100  d   116  t
  37  %    53  5   69  E   85  U   101  e   117  u
  38  &    54  6   70  F   86  V   102  f   118  v
  39  '    55  7   71  G   87  W   103  g   119  w
  40  (    56  8   72  H   88  X   104  h   120  x
  41  )    57  9   73  I   89  Y   105  i   121  y
  42  *    58  :   74  J   90  Z   106  j   122  z
  43  +    59  ;   75  K   91  [   107  k   123  {
  44  ,    60  <   76  L   92  \   108  l   124  |
  45  -    61  =   77  M   93  ]   109  m   125  }
  46  .    62  >   78  N   94  ^   110  n   126  ~
  47  /    63  ?   79  O   95  _   111  o   127  DEL
```

---

## Extended ASCII

```
  Standard ASCII: 7 bits (0-127)
  Extended ASCII: 8 bits (0-255) — additional 128 characters

  Problem: many DIFFERENT extended ASCII standards!
  • Latin-1 (ISO 8859-1): Western European characters
  • Windows-1252: Microsoft's extension
  • Others for Greek, Cyrillic, Arabic, etc.

  A file encoded in Latin-1 looks garbled in Windows-1252!
  This led to the need for Unicode.
```

---

## Unicode

```
  Unicode: one encoding to cover ALL human languages

  Goal: assign a unique number (code point) to every character
  Currently: ~150,000 characters, 161 scripts

  Code points are written as U+XXXX:
  U+0041 = A
  U+0042 = B
  U+00E9 = é
  U+4E16 = 世
  U+1F600 = 😀

  Unicode is NOT an encoding — it's a character set.
  HOW these code points are stored is determined by:
  UTF-8, UTF-16, or UTF-32.
```

---

## UTF-8

```
  Variable-length encoding: 1 to 4 bytes per character

  ┌───────────────────┬────────┬───────────────────────┐
  │ Code Point Range  │ Bytes  │ Byte Pattern           │
  ├───────────────────┼────────┼───────────────────────┤
  │ U+0000 - U+007F  │   1    │ 0xxxxxxx              │
  │ U+0080 - U+07FF  │   2    │ 110xxxxx 10xxxxxx     │
  │ U+0800 - U+FFFF  │   3    │ 1110xxxx 10xx 10xx    │
  │ U+10000 - U+10FFFF│  4    │ 11110xxx 10xx 10xx 10x│
  └───────────────────┴────────┴───────────────────────┘

  Examples:
  'A' (U+0041) → 0x41             → 1 byte
  'é' (U+00E9) → 0xC3 0xA9       → 2 bytes
  '世' (U+4E16) → 0xE4 0xB8 0x96  → 3 bytes
  '😀' (U+1F600) → 0xF0 0x9F 0x98 0x80 → 4 bytes
```

### Key Properties

```
  • Backward compatible with ASCII
    (all ASCII chars are 1 byte, same values)
  • Most common encoding on the web (>98%)
  • Variable length → s[i] is NOT O(1) for non-ASCII!

  ┌──────────────────────────────────────────────┐
  │  "Café"                                       │
  │  C    a    f    é                              │
  │  0x43 0x61 0x66 0xC3 0xA9                     │
  │  1B   1B   1B   2 bytes                       │
  │  Total: 5 bytes for 4 characters!             │
  │                                               │
  │  s[3] in bytes ≠ 'é' in characters            │
  │  Byte index 3 = 0xC3 (first byte of é)        │
  │  Character index 3 = 'é' (need to scan!)      │
  └──────────────────────────────────────────────┘
```

---

## UTF-16

```
  Uses 2 or 4 bytes per character

  BMP (Basic Multilingual Plane): U+0000 - U+FFFF → 2 bytes
  Beyond BMP: U+10000+ → 4 bytes (surrogate pairs)

  Used by: Java (internal), JavaScript, Windows API

  'A'  → 0x0041 (2 bytes)
  '世'  → 0x4E16 (2 bytes)
  '😀' → 0xD83D 0xDE00 (4 bytes — surrogate pair!)

  Problem: character at index i might be a surrogate!
  Java: "😀".length() = 2, not 1!
```

---

## UTF-32

```
  Fixed 4 bytes per character

  'A'  → 0x00000041
  '世'  → 0x00004E16
  '😀' → 0x0001F600

  Advantage: s[i] is always O(1) — true random access
  Disadvantage: 4× memory for ASCII text

  Used by: Python 3 (internal, selectively)
```

---

## Comparison

| Encoding | Bytes/char | Index access | ASCII compatible | Used by |
|----------|-----------|-------------|-----------------|---------|
| ASCII | 1 | O(1) | Yes | C, legacy |
| UTF-8 | 1-4 | O(n) | Yes | Web, Linux, files |
| UTF-16 | 2-4 | O(n)* | No | Java, JS, Windows |
| UTF-32 | 4 | O(1) | No | Python (internal) |

*O(1) only within BMP

---

## DSA Implications

```
  For coding interviews and competitive programming:
  • Usually assume ASCII (128 chars)
  • Frequency arrays of size 26 (lowercase) or 128 (ASCII)
  • Character math: c - 'a' gives index 0-25

  For production code:
  • Be aware of encoding
  • "length" might mean bytes or characters
  • Reversing a UTF-8 string byte-by-byte BREAKS it!

  ┌──────────────────────────────────────────────────┐
  │  Common DSA pattern:                              │
  │                                                   │
  │  freq = array of size 26  (for lowercase a-z)    │
  │  freq[c - 'a']++                                 │
  │                                                   │
  │  This works because 'a'-'z' are contiguous in    │
  │  ASCII: 97, 98, 99, ..., 122                     │
  │  So 'a'-'a'=0, 'b'-'a'=1, ..., 'z'-'a'=25     │
  └──────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| ASCII | 7-bit, 128 chars, basis for everything |
| Unicode | Universal character set, 150K+ chars |
| UTF-8 | Variable (1-4 bytes), web standard, ASCII-compatible |
| UTF-16 | Variable (2-4 bytes), Java/JS internal |
| UTF-32 | Fixed 4 bytes, true O(1) access |
| DSA tip | Usually assume ASCII; use c - 'a' for indexing |

---

## Quick Revision Questions

1. **What is the ASCII value of 'A'? Of '0'?** How do you convert a digit character to its integer value?

2. **Why was Unicode created?** What problem did it solve?

3. **What is the difference between UTF-8 and UTF-32?** When would you prefer each?

4. **Why is `s[i]` not always O(1) with UTF-8?** Give an example.

5. **In a DSA context, how do you use character encoding** to create a frequency array for lowercase letters?

6. **Java says `"😀".length() = 2`. Why?** What encoding does Java use internally?

---

[← Previous: Immutability and Its Impact](04-immutability.md) | [Back to README](../README.md) | [Next: String Comparison →](06-string-comparison.md)
