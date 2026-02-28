# Chapter 6: String Transformation

[← Previous: String Compression](05-string-compression.md) | [Back to README](../README.md) | [Next: Matrix Basics →](../10-Matrix-2D-Array/01-matrix-basics.md)

---

## Overview

String transformation problems require converting strings between formats, parsing values, or restructuring character arrangements. These are very common in interviews because they test handling of edge cases, overflow, and character manipulation.

---

## 1. String to Integer (atoi) — LeetCode 8

```
  Rules:
  1. Skip leading whitespace
  2. Check for '+' or '-' sign
  3. Read digits until non-digit or end
  4. Clamp to [INT_MIN, INT_MAX] on overflow

  Examples:
  "   -42"       → -42
  "4193 word"    → 4193
  "words 987"    → 0 (no leading digits)
  "  +0 123"     → 0 (stops at space)
  "-91283472332" → -2147483648 (clamped to INT_MIN)
```

### Algorithm

```
FUNCTION myAtoi(s):
    i = 0
    n = length(s)
    sign = 1
    result = 0

    // Step 1: Skip whitespace
    WHILE i < n AND s[i] == ' ':
        i++

    // Step 2: Check sign
    IF i < n AND (s[i] == '+' OR s[i] == '-'):
        sign = (s[i] == '-') ? -1 : 1
        i++

    // Step 3: Read digits
    WHILE i < n AND isDigit(s[i]):
        digit = s[i] - '0'

        // Check for overflow BEFORE adding
        IF result > (INT_MAX - digit) / 10:
            RETURN (sign == 1) ? INT_MAX : INT_MIN

        result = result × 10 + digit
        i++

    RETURN sign × result
```

### Trace

```
  s = "   -42abc"
  
  Step 1: Skip spaces → i=3
  Step 2: s[3]='-' → sign=-1, i=4
  Step 3: s[4]='4' → result = 0×10+4 = 4, i=5
          s[5]='2' → result = 4×10+2 = 42, i=6
          s[6]='a' → not digit → stop
  Return: -1 × 42 = -42
```

---

## 2. Integer to String

```
FUNCTION intToStr(num):
    IF num == 0: RETURN "0"

    negative = (num < 0)
    IF negative: num = -num

    result = []
    WHILE num > 0:
        result.append(chr(num % 10 + '0'))
        num = num / 10

    IF negative: result.append('-')
    RETURN reverse(result)
```

### Trace

```
  num = -123

  negative = true, num = 123
  123 % 10 = 3 → result = ['3'], num = 12
   12 % 10 = 2 → result = ['3','2'], num = 1
    1 % 10 = 1 → result = ['3','2','1'], num = 0
  Append '-' → ['3','2','1','-']
  Reverse → "-123"
```

---

## 3. String Rotation (LeetCode 796)

```
  Check if s2 is a rotation of s1.

  "abcde" rotated by 2 → "cdeab"

  Key Insight: s2 is a rotation of s1 
               ⟺ s2 is a substring of s1 + s1

  ┌─────────────────────────────┐
  │ s1 = "abcde"                │
  │ s1 + s1 = "abcdeabcde"     │
  │                ─────        │
  │            "cdeab" found!   │
  │ So "cdeab" IS a rotation.   │
  └─────────────────────────────┘
```

### Algorithm

```
FUNCTION isRotation(s1, s2):
    IF length(s1) ≠ length(s2): RETURN false
    IF length(s1) == 0: RETURN true
    RETURN (s1 + s1).contains(s2)
```

### Why It Works

```
  Any rotation splits s1 into two parts:
  s1 = A + B
  rotation = B + A

  s1 + s1 = A + B + A + B
                 ─────
                 B + A  ← always a substring!

  Example: s1 = "ab|cde"  (A="ab", B="cde")
           rotation = "cde|ab"
           s1+s1 = "ab|cde|ab|cde"
                       ─────────
                       "cdeab" found here
```

---

## 4. Zigzag Conversion (LeetCode 6)

```
  "PAYPALISHIRING", numRows=3

  P   A   H   N       Row 0: index 0, 4, 8, 12
  A P L S I I G       Row 1: index 1, 3, 5, 7, 9, 11, 13
  Y   I   R           Row 2: index 2, 6, 10

  Reading row by row: "PAHNAPLSIIGYIR"

  Pattern with numRows=4:
  P     I     N       Row 0: step = 6
  A   L S   I G       Row 1: alternating steps 4, 2
  Y A   H R           Row 2: alternating steps 2, 4
  P     I             Row 3: step = 6

  Cycle length = 2 × (numRows - 1)
```

### Algorithm

```
FUNCTION zigzag(s, numRows):
    IF numRows == 1: RETURN s

    rows = array of numRows empty strings
    curRow = 0
    goingDown = false

    FOR each char c in s:
        rows[curRow] += c
        IF curRow == 0 OR curRow == numRows - 1:
            goingDown = NOT goingDown
        curRow += goingDown ? 1 : -1

    RETURN concatenate all rows
```

### Trace

```
  s="PAYPALISHIRING", numRows=3, cycle=4

  P → row 0 ↓    rows = ["P", "", ""]
  A → row 1 ↓    rows = ["P", "A", ""]
  Y → row 2 ↑    rows = ["P", "A", "Y"]
  P → row 1 ↓    rows = ["P", "AP", "Y"]
  A → row 0 ↓    rows = ["PA", "AP", "Y"]
  L → row 1 ↓    rows = ["PA", "APL", "Y"]
  I → row 2 ↑    rows = ["PA", "APL", "YI"]
  S → row 1 ↓    rows = ["PA", "APLS", "YI"]
  H → row 0 ↓    rows = ["PAH", "APLS", "YI"]
  I → row 1 ↓    rows = ["PAH", "APLSI", "YI"]
  R → row 2 ↑    rows = ["PAH", "APLSI", "YIR"]
  I → row 1 ↓    rows = ["PAH", "APLSII", "YIR"]
  N → row 0 ↓    rows = ["PAHN", "APLSII", "YIR"]
  G → row 1      rows = ["PAHN", "APLSIIG", "YIR"]

  Result: "PAHN" + "APLSIIG" + "YIR" = "PAHNAPLSIIGYIR"
```

---

## 5. Add Strings (LeetCode 415)

```
  Add two non-negative integers represented as strings.
  Cannot convert to int (numbers can be very large).

  "456" + "77" = "533"

  Process from right to left with carry:

      4  5  6
    +    7  7
    --------
      5  3  3

  Trace:
    6 + 7 = 13 → write 3, carry 1
    5 + 7 + 1 = 13 → write 3, carry 1
    4 + 0 + 1 = 5 → write 5, carry 0
    Result: "533"
```

### Algorithm

```
FUNCTION addStrings(num1, num2):
    i = length(num1) - 1
    j = length(num2) - 1
    carry = 0
    result = []

    WHILE i ≥ 0 OR j ≥ 0 OR carry > 0:
        x = (i ≥ 0) ? num1[i] - '0' : 0
        y = (j ≥ 0) ? num2[j] - '0' : 0

        sum = x + y + carry
        result.append(chr(sum % 10 + '0'))
        carry = sum / 10

        i--
        j--

    RETURN reverse(result)
```

---

## 6. Multiply Strings (LeetCode 43)

```
  Multiply two numbers represented as strings.

  "123" × "45":

        1 2 3
      ×   4 5
      -------
        6 1 5   (123 × 5)
      4 9 2 0   (123 × 4, shifted left)
      -------
      5 5 3 5

  Key insight: digit at position i × digit at position j
               → contributes to position i + j and i + j + 1
```

### Algorithm

```
FUNCTION multiply(num1, num2):
    m = length(num1), n = length(num2)
    result = array of (m + n) zeros

    FOR i = m-1 DOWNTO 0:
        FOR j = n-1 DOWNTO 0:
            mul = (num1[i] - '0') × (num2[j] - '0')
            p1 = i + j        // tens position
            p2 = i + j + 1    // ones position
            sum = mul + result[p2]

            result[p2] = sum % 10
            result[p1] += sum / 10

    // Convert to string, skip leading zeros
    str = ""
    FOR digit in result:
        IF NOT (str is empty AND digit == 0):
            str += chr(digit + '0')

    RETURN (str is empty) ? "0" : str
```

### Trace

```
  "12" × "34"
  result = [0, 0, 0, 0]

  i=1 (digit '2'):
    j=1 (digit '4'): 2×4=8, p1=2, p2=3
      result[3] += 8 → result = [0,0,0,8]
    j=0 (digit '3'): 2×3=6, p1=1, p2=2
      result[2] += 6 → result = [0,0,6,8]

  i=0 (digit '1'):
    j=1 (digit '4'): 1×4=4, p1=1, p2=2
      sum = 4+6=10, result[2]=0, result[1]+=1
      result = [0,1,0,8]
    j=0 (digit '3'): 1×3=3, p1=0, p2=1
      sum = 3+1=4, result[1]=4, result[0]+=0
      result = [0,4,0,8]

  Skip leading zeros → "0408" → "408"
  12 × 34 = 408 ✓
```

---

## 7. Common Type Conversions

```
  ┌──────────────────┬──────────────────────────────┐
  │ Conversion       │ Key Considerations           │
  ├──────────────────┼──────────────────────────────┤
  │ String → Int     │ Overflow, signs, whitespace   │
  │ Int → String     │ Negative, zero edge case      │
  │ String → Float   │ Decimal point, precision      │
  │ Char → Int       │ char - '0' for digits         │
  │ Int → Char       │ chr(digit + '0') for digits   │
  │ Upper → Lower    │ char + 32 (ASCII) or OR 0x20  │
  │ Lower → Upper    │ char - 32 (ASCII) or AND ~0x20│
  │ Base conversion  │ Repeated divide & remainder   │
  └──────────────────┴──────────────────────────────┘

  ASCII tricks:
  'A' = 65, 'a' = 97 → difference = 32
  
  'A' | 0x20 → 'a'   (set bit 5)
  'a' & ~0x20 → 'A'  (clear bit 5)
  
  isLetter: (c | 0x20) >= 'a' AND (c | 0x20) <= 'z'
```

---

## Complexity

| Problem | Time | Space |
|---------|------|-------|
| atoi | O(n) | O(1) |
| int to string | O(d) digits | O(d) |
| String rotation check | O(n) | O(n) |
| Zigzag conversion | O(n) | O(n) |
| Add strings | O(max(m,n)) | O(max(m,n)) |
| Multiply strings | O(m × n) | O(m + n) |

---

## Summary Table

| Concept | Key Insight |
|---------|-------------|
| atoi | Skip space → sign → digits → overflow check |
| int to string | Extract digits with % 10, reverse |
| String rotation | s2 ∈ (s1 + s1) |
| Zigzag | Bounce row index between 0 and numRows-1 |
| Add strings | Right-to-left with carry, like grade school |
| Multiply strings | Position i × j → result[i+j] and result[i+j+1] |
| Case conversion | ASCII bit manipulation (bit 5) |

---

## Quick Revision Questions

1. **Implement atoi** handling spaces, signs, and overflow. What happens with "  -2147483649"?

2. **Prove that** checking `s2 in (s1+s1)` correctly identifies all rotations.

3. **Trace zigzag conversion** of "ABCDEFGHIJ" with numRows=4.

4. **Add the strings** "999" and "1". Trace each step.

5. **Multiply the strings** "99" and "99". Show the result array at each step.

6. **How does bit manipulation** convert 'G' to 'g'? What is 'G' in binary?

---

[← Previous: String Compression](05-string-compression.md) | [Back to README](../README.md) | [Next: Matrix Basics →](../10-Matrix-2D-Array/01-matrix-basics.md)
