# 8.3 Polynomial Hashing

## Unit 8: String Hashing Problems

---

## Definition

Polynomial hashing treats a string as the **coefficients of a polynomial** evaluated at a fixed base $d$, modulo a prime $q$:

$$H(s) = \left(\sum_{i=0}^{n-1} s[i] \cdot d^{n-1-i}\right) \mod q$$

```
╔══════════════════════════════════════════════════════════════╗
║  String "HELLO":                                            ║
║                                                              ║
║  H(s) = H·d⁴ + E·d³ + L·d² + L·d¹ + O·d⁰                ║
║                                                              ║
║  Using ASCII values (H=72, E=69, L=76, O=79), d=31:        ║
║  = 72·31⁴ + 69·31³ + 76·31² + 76·31 + 79                 ║
║  = 72·923521 + 69·29791 + 76·961 + 2356 + 79              ║
║  = 66493512 + 2055579 + 73036 + 2356 + 79                 ║
║  = 68624562                                                 ║
║                                                              ║
║  With q = 10⁹+7:  68624562 mod 10⁹+7 = 68624562           ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Why Polynomial?

```
╔══════════════════════════════════════════════════════════════╗
║  POSITION SENSITIVITY:                                      ║
║                                                              ║
║  Simple sum: hash("ABC") = A+B+C = hash("CBA")  COLLISION!║
║  XOR:        hash("AB")  = A^B   = hash("BA")   COLLISION!║
║  Polynomial: hash("ABC") = A·d²+B·d+C ≠ C·d²+B·d+A       ║
║              → Different for different orderings ✓          ║
║                                                              ║
║  The powers of d encode POSITION information.               ║
║  Character at index i is multiplied by d^(n-1-i),           ║
║  giving each position a unique weight.                      ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Efficient Computation: Horner's Method

Computing $d^{n-1}$ directly causes overflow. Instead, evaluate using **Horner's rule**:

$$H(s) = (\ldots((s[0] \cdot d + s[1]) \cdot d + s[2]) \cdot d + \ldots + s[n-1]) \mod q$$

```
FUNCTION polyHash(s, d, q):
    hash = 0
    FOR i = 0 TO len(s) - 1:
        hash = (hash * d + s[i]) % q
    RETURN hash
```

### Trace

```
s = "CAB", d = 31, q = 997

Step 0: hash = 0
Step 1: hash = (0 * 31 + 67) % 997 = 67      (C=67)
Step 2: hash = (67 * 31 + 65) % 997
       = (2077 + 65) % 997
       = 2142 % 997 = 148                     (A=65)
Step 3: hash = (148 * 31 + 66) % 997
       = (4588 + 66) % 997
       = 4654 % 997 = 663                     (B=66)

hash("CAB") = 663

Compare: hash("ABC") using same parameters:
  0 → 65 → (65*31+66)%997 = 2081 → (2081*31+67)%997 = 64578%997 = 793

663 ≠ 793 → different hashes, as expected ✓
```

---

## Choosing the Base

The base $d$ determines the "numerical system" for encoding:

```
╔══════════════════════════════════════════════════════════════╗
║  For LOWERCASE letters (a-z):                               ║
║    Map: 'a'→1, 'b'→2, ..., 'z'→26                         ║
║    Base: d ≥ 27 (must exceed max character value)           ║
║    Common: d = 31 (prime, good distribution)                ║
║                                                              ║
║  For GENERAL ASCII:                                         ║
║    Map: use ASCII value directly                            ║
║    Base: d = 256 or d = 131 (prime near 128)                ║
║                                                              ║
║  WHY d SHOULD BE PRIME:                                     ║
║    Reduces patterns in hash distribution                    ║
║    Minimizes systematic collisions                          ║
║    Popular choices: 31, 37, 53, 131, 257                    ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Substring Hash Extraction

With **prefix hashes**, extract any substring hash in $O(1)$:

```
// Precompute prefix hashes
prefixHash[0] = 0
prefixHash[i] = (prefixHash[i-1] * d + s[i-1]) % q

// Precompute powers
pow[0] = 1
pow[i] = (pow[i-1] * d) % q

// Hash of s[l..r] (0-indexed, inclusive)
hash(l, r) = (prefixHash[r+1] - prefixHash[l] * pow[r-l+1]) % q
```

### Trace

```
s = "ABCDE", d = 10, q = 97

Prefix hashes:
  pH[0] = 0
  pH[1] = (0*10 + 1) % 97 = 1       (A→1)
  pH[2] = (1*10 + 2) % 97 = 12      (AB→12)
  pH[3] = (12*10 + 3) % 97 = 123%97 = 26   (ABC)
  pH[4] = (26*10 + 4) % 97 = 264%97 = 70   (ABCD)
  pH[5] = (70*10 + 5) % 97 = 705%97 = 26   (ABCDE)

Powers: pow = [1, 10, 100%97=3, 30%97=30, 300%97=9]

hash("BCD") = s[1..3]:
  = (pH[4] - pH[1] * pow[3]) % 97
  = (70 - 1 * 30) % 97
  = 40 % 97 = 40

Verify directly: B·10² + C·10 + D = 2·100+3·10+4 = 234
  234 % 97 = 40 ✓
```

---

## Collision Analysis

The probability that two **random** strings of length $m$ collide under polynomial hashing:

$$\Pr[\text{collision}] \leq \frac{m-1}{q}$$

This is the **Schwartz-Zippel lemma** — two degree-$(m-1)$ polynomials agree on at most $m-1$ points out of $q$ possible.

```
╔══════════════════════════════════════════════════════════════╗
║  With q = 10⁹+7 and m = 1000:                              ║
║    Pr[collision] ≤ 999 / 10⁹ ≈ 10⁻⁶                      ║
║                                                              ║
║  For n string comparisons:                                  ║
║    Pr[any collision] ≤ n(n-1)/2 × (m-1)/q                  ║
║    (Birthday paradox applied)                               ║
║                                                              ║
║  With n=10⁵ strings:                                        ║
║    ≈ 5×10⁹ × 10⁻⁶ ≈ 5000    ← TOO HIGH!                  ║
║    Solution: Use double hashing or larger q                 ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Java's String hashCode

Java uses polynomial hashing with $d = 31$ and no modulus (uses 32-bit overflow):

```
// Java's actual implementation:
public int hashCode() {
    int h = 0;
    for (int i = 0; i < length(); i++) {
        h = 31 * h + charAt(i);
    }
    return h;   // 32-bit, wraps around on overflow
}

// Why 31?
// 1. It's prime (good distribution)
// 2. 31 * x = (x << 5) - x (CPU optimization)
// 3. Empirically low collision rate for English text
```

---

## Quick Revision Question

**Q: Using polynomial hashing with Horner's method, compute the hash of "DAD" with base $d=10$ and modulus $q=97$. Show each step.**

<details>
<summary>Click to reveal answer</summary>

Using ASCII-based mapping where D=4, A=1, D=4:

$$H = (((0 \times 10 + 4) \times 10 + 1) \times 10 + 4) \mod 97$$

**Step-by-step:**
1. `hash = 0`
2. `hash = (0 × 10 + 4) % 97 = 4`
3. `hash = (4 × 10 + 1) % 97 = 41`
4. `hash = (41 × 10 + 4) % 97 = 414 % 97 = 26`

**Answer: hash("DAD") = 26**

**Verification without Horner's:**
$H = 4 \times 10^2 + 1 \times 10 + 4 = 414$
$414 \mod 97 = 414 - 4 \times 97 = 414 - 388 = 26$ ✓

Horner's method is equivalent but avoids computing large powers of $d$ explicitly.

</details>

---

| [← Rolling Hash Technique](02-rolling-hash-technique.md) | [Next: Comparing Substrings →](04-comparing-substrings.md) |
|:---|---:|
