# Chapter 5: String Compression

[← Previous: Substring Problems](04-substring-problems.md) | [Back to README](../README.md) | [Next: String Transformation →](06-string-transformation.md)

---

## Overview

String compression replaces consecutive repeated characters with the character followed by a count. This technique appears in interviews (LeetCode 443), file compression, and run-length encoding (RLE).

---

## Basic Compression (LeetCode 443)

```
  Input:  ['a','a','b','b','c','c','c']
  Output: ['a','2','b','2','c','3']
  Return: 6 (length of compressed result)

  Rules:
  - Groups of consecutive identical characters
  - If count = 1, omit the count
  - If count > 1, write count digits after the character
  - Must be done in-place with O(1) extra space
```

---

## Algorithm

```
FUNCTION compress(chars):
    write = 0       // position to write
    read = 0        // position to read
    n = length(chars)

    WHILE read < n:
        char = chars[read]
        count = 0

        // Count consecutive occurrences
        WHILE read < n AND chars[read] == char:
            read++
            count++

        // Write the character
        chars[write] = char
        write++

        // Write the count (if > 1)
        IF count > 1:
            countStr = toString(count)
            FOR digit in countStr:
                chars[write] = digit
                write++

    RETURN write
```

---

## Trace

```
  chars = ['a','a','b','b','c','c','c']
  read=0, write=0

  Char 'a': count consecutive 'a's
    read=0: 'a' → count=1, read=1
    read=1: 'a' → count=2, read=2
    read=2: 'b' ≠ 'a' → stop
    Write: chars[0]='a', chars[1]='2', write=2

  Char 'b': count consecutive 'b's
    read=2: 'b' → count=1, read=3
    read=3: 'b' → count=2, read=4
    read=4: 'c' ≠ 'b' → stop
    Write: chars[2]='b', chars[3]='2', write=4

  Char 'c': count consecutive 'c's
    read=4: 'c' → count=1, read=5
    read=5: 'c' → count=2, read=6
    read=6: 'c' → count=3, read=7
    read=7: past end → stop
    Write: chars[4]='c', chars[5]='3', write=6

  Result: ['a','2','b','2','c','3']
  Return: 6

  Array state:
  Before: [a][a][b][b][c][c][c]
  After:  [a][2][b][2][c][3][c]  ← last char is leftover
                                    but we return length 6
```

---

## Multi-Digit Counts

```
  Input: "aaaaaaaaaaaab"  (12 a's, 1 b)
  Output: "a12b"

  The count "12" takes TWO digit positions.

  chars[0] = 'a'
  chars[1] = '1'
  chars[2] = '2'
  chars[3] = 'b'
  Return: 4

  Important: convert count to string, write each digit!
  
  Count 12 → "12" → write '1' then '2'
  Count 100 → "100" → write '1', '0', '0'
```

---

## Run-Length Encoding (RLE)

```
  A more general compression scheme:

  Input:  "AAABBBCCDDDD"
  Output: "3A3B2C4D"

  Encoding:
  - 3 × 'A' → "3A"
  - 3 × 'B' → "3B"
  - 2 × 'C' → "2C"
  - 4 × 'D' → "4D"

  Decoding:
  "3A3B2C4D" → "AAABBBCCDDDD"
```

### When RLE Works Well

```
  GOOD for RLE:                  BAD for RLE:
  "AAAAABBBBB" → "5A5B"         "ABCDEFG" → "1A1B1C1D1E1F1G"
  10 chars → 4 chars             7 chars → 14 chars (WORSE!)
  Compression ratio: 40%         Expansion: 200%

  RLE works best with many consecutive repetitions:
  - Simple graphics (large solid regions)
  - Binary data
  - Fax transmission (originally designed for this)
```

---

## Decompression

```
FUNCTION decompress(compressed):
    result = []
    i = 0

    WHILE i < length(compressed):
        char = compressed[i]
        i++

        // Read count (could be multi-digit)
        count = 0
        WHILE i < length(compressed) AND isDigit(compressed[i]):
            count = count × 10 + (compressed[i] - '0')
            i++

        IF count == 0: count = 1    // no count means 1

        // Append 'count' copies of char
        FOR j = 1 TO count:
            result.append(char)

    RETURN result
```

---

## String Compression Variations

### 1. Count and Say (LeetCode 38)

```
  Start with "1"
  n=1: "1"              (one 1)
  n=2: "11"             (two 1s)
  n=3: "21"             (one 2, one 1)
  n=4: "1211"           (one 1, one 2, two 1s)
  n=5: "111221"         (three 1s, two 2s, one 1)
  n=6: "312211"

  Each term describes the previous term using RLE!
```

### 2. Encode and Decode Strings (LeetCode 271)

```
  Encode list of strings into a single string.

  Strategy: length prefix + delimiter
  
  ["hello", "world"] → "5#hello5#world"
  
  Decode: read number, skip '#', read that many chars.
```

### 3. String Compression III (Chunks of 9)

```
  Some variants limit count to single digit (max 9):
  
  "aaaaaaaaaaaab" (12 a's)
  → "9a3ab" (9 a's, then 3 a's, then b)
  
  This ensures each count is one digit.
```

---

## In-Place Writing: Why write ≤ read?

```
  In the LeetCode version (compress in-place):

  Write pointer is always ≤ read pointer because:
  - For groups of size 1: write 1 char (same as read 1)
  - For groups of size 2: write 2 chars ("a2") for reading 2
  - For groups of size 3-9: write 2 chars for reading 3-9 (SAVES space!)
  - For groups of size 10-99: write 3 chars for reading 10-99 (SAVES!)

  Write never overtakes read, so in-place is safe!

  ┌─────────────────────────────────────────────────┐
  │ Group size │ Chars read │ Chars written │ Net    │
  ├────────────┼────────────┼───────────────┼────────┤
  │ 1          │ 1          │ 1             │ even   │
  │ 2          │ 2          │ 2             │ even   │
  │ 3-9        │ 3-9        │ 2             │ saves  │
  │ 10-99      │ 10-99      │ 3             │ saves  │
  │ 100-999    │ 100-999    │ 4             │ saves  │
  └────────────┴────────────┴───────────────┴────────┘
```

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Compress (in-place) | O(n) | O(1) |
| Decompress | O(output length) | O(output) |
| RLE encode | O(n) | O(n) |
| RLE decode | O(output length) | O(output) |

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Basic idea | Replace runs with char + count |
| In-place | Write pointer ≤ read pointer, always safe |
| Count = 1 | Omit the count (LeetCode convention) |
| Multi-digit | Convert count to string, write each digit |
| RLE | General form: count before/after character |
| Works well when | Many consecutive repetitions |
| Works poorly when | No repetitions (output longer than input!) |

---

## Quick Revision Questions

1. **Trace the compression algorithm** on ['a','b','b','b','b','c','c']. What is the output?

2. **Why is the write pointer always ≤ the read pointer?** Prove it for different group sizes.

3. **When does RLE make the string longer?** Give an example.

4. **How do you handle multi-digit counts** like 12 or 100?

5. **What is Count and Say?** Generate the first 5 terms.

6. **Design an encode/decode scheme** for a list of strings that handles any character.

---

[← Previous: Substring Problems](04-substring-problems.md) | [Back to README](../README.md) | [Next: String Transformation →](06-string-transformation.md)
