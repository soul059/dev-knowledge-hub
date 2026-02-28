# 8.1 Rabin-Karp Algorithm

## Unit 8: String Hashing Problems

---

## The Problem

> Given a text of length $n$ and a pattern of length $m$, find all occurrences of the pattern in the text.

```
╔══════════════════════════════════════════════════════════════╗
║  Text:    "ABCABCDABCABC"                                   ║
║  Pattern: "ABCABC"                                          ║
║                                                              ║
║  Naive: Compare m characters at each of n-m+1 positions    ║
║         → O(nm) worst case                                  ║
║                                                              ║
║  Rabin-Karp: Hash the pattern, then SLIDE a hash window    ║
║              across the text. Only compare characters       ║
║              when hashes match.                              ║
║         → O(n+m) average case                               ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Core Idea

1. Compute `hash(pattern)`
2. For each window of length $m$ in text, compute `hash(window)`
3. If hashes match → verify character by character (avoid false positives)
4. Use **rolling hash** to update window hash in $O(1)$

```
Text:   [A B C D E F G H]
         ─────
         hash₁  compare with hash(pattern)
           ─────
           hash₂ = update hash₁ in O(1)
             ─────
             hash₃ = update hash₂ in O(1)
```

---

## Algorithm

```
FUNCTION rabinKarp(text, pattern):
    n = len(text), m = len(pattern)
    IF m > n: RETURN []

    d = 256          // alphabet size
    q = 101          // a prime modulus
    h = d^(m-1) mod q   // hash multiplier for leading digit
    p = 0            // pattern hash
    t = 0            // text window hash
    results = []

    // Compute initial hashes
    FOR i = 0 TO m - 1:
        p = (d * p + pattern[i]) mod q
        t = (d * t + text[i]) mod q

    // Slide window
    FOR i = 0 TO n - m:
        IF p == t:
            // Hash match → verify characters
            IF text[i..i+m-1] == pattern:
                results.add(i)

        // Compute hash for next window
        IF i < n - m:
            t = (d * (t - text[i] * h) + text[i + m]) mod q
            IF t < 0: t += q

    RETURN results
```

---

## Rolling Hash Update

The key formula for shifting the window by one position:

$$t_{i+1} = (d \cdot (t_i - \text{text}[i] \cdot h) + \text{text}[i+m]) \mod q$$

```
╔══════════════════════════════════════════════════════════════╗
║  Window shift from position i to i+1:                       ║
║                                                              ║
║  Old window: text[i], text[i+1], ..., text[i+m-1]          ║
║  New window: text[i+1], text[i+2], ..., text[i+m]          ║
║                                                              ║
║  Steps:                                                     ║
║  1. Remove leading character: t - text[i] * d^(m-1)        ║
║  2. Shift left (multiply by d): result * d                  ║
║  3. Add new trailing character: + text[i+m]                 ║
║  4. Apply modulus: mod q                                    ║
║                                                              ║
║  Old: [A][ B  C  D ]     hash = A·d³ + B·d² + C·d + D     ║
║  Remove A: B·d² + C·d + D                                  ║
║  Shift:    B·d³ + C·d² + D·d                               ║
║  Add E:    B·d³ + C·d² + D·d + E                           ║
║  New: [ B  C  D ][E]                                        ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Complete Trace

```
Text:    "AABCAB"    Pattern: "ABC"
d = 256, q = 101, m = 3

Step 0: Compute h = d^(m-1) mod q = 256² mod 101 = 65536 mod 101 = 82

Step 1: Initial hashes (i = 0 to 2)
  Pattern "ABC":
    p = 0
    p = (256*0 + 65) mod 101 = 65        (A=65)
    p = (256*65 + 66) mod 101 = 16706 mod 101 = 43
    p = (256*43 + 67) mod 101 = 11075 mod 101 = 64

  Text "AAB" (first window):
    t = 0
    t = (256*0 + 65) mod 101 = 65
    t = (256*65 + 65) mod 101 = 16705 mod 101 = 42
    t = (256*42 + 66) mod 101 = 10818 mod 101 = 12

Step 2: Slide window

  i=0: t=12, p=64, t≠p → no match
       Update: t = (256*(12 - 65*82) + 67) mod 101
             = (256*(12 - 5330) + 67) mod 101
             = (256*(-5318) + 67) mod 101
             = (-1361341) mod 101 = 43
       (Alternatively: "ABC" window hash)

  i=1: t=43, p=64, t≠p → no match
       Update for next window "BCA":
       t = (256*(43 - 65*82) + 65) mod 101 = ... = 64

  i=2: t=64, p=64, t==p → VERIFY!
       text[2..4] = "BCA"? No, text[2..4] = "BCA" 
       Wait — let me recalculate...

Actually let me use a simpler example with small values:

Text: "2 3 5 9 7"  Pattern: "5 9"   d=10, q=13, m=2
h = 10^(2-1) mod 13 = 10

Pattern hash: p = (10*0 + 5) mod 13 = 5
              p = (10*5 + 9) mod 13 = 59 mod 13 = 7

Text initial (window "2 3"):
  t = (10*0 + 2) mod 13 = 2
  t = (10*2 + 3) mod 13 = 23 mod 13 = 10

i=0: t=10, p=7, no match
     t = (10*(10 - 2*10) + 5) mod 13
     = (10*(-10) + 5) mod 13
     = -95 mod 13 = 5       (window "3 5")

i=1: t=5, p=7, no match
     t = (10*(5 - 3*10) + 9) mod 13
     = (10*(-25) + 9) mod 13
     = -241 mod 13 = 7      (window "5 9")

i=2: t=7, p=7, MATCH! Verify: "5 9" == "5 9" ✓ → found at index 2

i=3: t = (10*(7 - 5*10) + 7) mod 13
     = (10*(-43) + 7) mod 13
     = -423 mod 13 = 5      (window "9 7")
     t=5, p=7, no match

Result: [2] ✓
```

---

## Complexity Analysis

| Case | Time | When |
|------|:---:|------|
| Best / Average | $O(n + m)$ | Few hash collisions |
| Worst case | $O(nm)$ | All hashes match (e.g., "AAAA" in "AAAAAA") |

**Space:** $O(1)$ extra (beyond input/output)

**Why average $O(n + m)$:** With a good modulus $q$, the probability of a spurious hit (hash match but string mismatch) is $\frac{1}{q}$. Expected number of false positives: $\frac{n-m+1}{q}$, each costing $O(m)$ to verify. Total: $O(n + m + \frac{nm}{q})$. With $q \gg m$, this approaches $O(n + m)$.

---

## Advantages and Limitations

```
╔══════════════════════╦═══════════════════════════════════════╗
║  ADVANTAGES          ║  LIMITATIONS                         ║
╠══════════════════════╬═══════════════════════════════════════╣
║  Simple to implement ║  Worst case still O(nm)              ║
║  O(n+m) average      ║  Spurious hits waste time            ║
║  Multi-pattern search║  KMP guarantees O(n+m) worst case   ║
║  Easy to extend to   ║  Hash collisions must be handled    ║
║   2D pattern search  ║  Modular arithmetic edge cases      ║
╚══════════════════════╩═══════════════════════════════════════╝
```

---

## Quick Revision Question

**Q: Why do we need to verify characters when the hash values match? Can't we just trust the hash?**

<details>
<summary>Click to reveal answer</summary>

No — because of **hash collisions** (spurious hits). Different strings can produce the same hash value.

Since we use `mod q` to keep hash values small:
- Hash values are in range $[0, q-1]$
- There are far more possible strings of length $m$ than $q$ values
- By the pigeonhole principle, collisions are inevitable

**Example:** With $q = 101$, the strings "ABC" and "XYZ" might both hash to 64. Without verification, we'd falsely report a match.

**Verification cost:** $O(m)$ per match, but true matches are what we're looking for anyway. Spurious hits add overhead:
- Expected spurious hits $\approx \frac{n}{q}$
- Choosing a large prime $q$ minimizes this
- In practice, $q = 10^9 + 7$ makes spurious hits extremely rare

The character verification step is what makes Rabin-Karp a **Las Vegas algorithm** — it always gives the correct answer (unlike a Monte Carlo version that would skip verification).

</details>

---

| [← Unit 7: Cycle Detection](../07-Hash-Set-Problems/06-cycle-detection.md) | [Next: Rolling Hash Technique →](02-rolling-hash-technique.md) |
|:---|---:|
