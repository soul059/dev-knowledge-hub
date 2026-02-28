# 2.6 String Hashing

## Unit 2: Hash Functions

---

## Why String Hashing is Special

Strings are the most common non-integer keys in hash tables. Unlike integers, strings need to be converted into numeric hash values. The quality of this conversion determines hash table performance.

```
╔═══════════════════════════════════════════════════════════╗
║              STRING HASHING CHALLENGE                    ║
║                                                          ║
║   Integer keys:  Already numbers → direct hashing        ║
║   String keys:   "hello" → ??? → number                ║
║                                                          ║
║   Must consider:                                         ║
║   • ALL characters (not just first/last)                ║
║   • Character ORDER ("abc" ≠ "bca")                     ║
║   • String LENGTH                                       ║
║   • Even distribution across table                      ║
╚═══════════════════════════════════════════════════════════╝
```

---

## Method 1: Simple Sum (Naive)

```
h(s) = (s[0] + s[1] + ... + s[n-1]) mod m

Problem: Anagrams collide!
h("cat") = (99+97+116) mod m = 312 mod m
h("act") = (97+99+116) mod m = 312 mod m   ← SAME!
h("tac") = (116+97+99) mod m = 312 mod m   ← SAME!

For ASCII strings, maximum sum ≈ 127 × length
Short strings cluster into small range of values.
```

---

## Method 2: Polynomial Rolling Hash

The standard approach — treats the string as a polynomial evaluated at a base $b$:

$$h(s) = \left(\sum_{i=0}^{n-1} s[i] \cdot b^{n-1-i}\right) \mod m$$

```
╔════════════════════════════════════════════════════════════╗
║              POLYNOMIAL ROLLING HASH                      ║
║                                                           ║
║   String: "cat"    base b = 31    mod m = 1000           ║
║                                                           ║
║   h("cat") = c×31² + a×31¹ + t×31⁰                      ║
║            = 99×961 + 97×31 + 116×1                      ║
║            = 95139 + 3007 + 116                          ║
║            = 98262                                       ║
║            = 98262 mod 1000 = 262                        ║
║                                                           ║
║   h("act") = a×31² + c×31¹ + t×31⁰                      ║
║            = 97×961 + 99×31 + 116×1                      ║
║            = 93217 + 3069 + 116                          ║
║            = 96402                                       ║
║            = 96402 mod 1000 = 402                        ║
║                                                           ║
║   "cat"=262 vs "act"=402  ← Different! Anagram resolved ║
╚════════════════════════════════════════════════════════════╝
```

### Why base 31?

| Base | Reason |
|------|--------|
| 31 | Prime, used by Java's `String.hashCode()` |
| 37 | Also prime, good alternative |
| 53 | Better for case-sensitive strings |
| 131 | Good for Unicode strings |

> **Rule**: Base should be a **prime** larger than the alphabet size.

---

## Step-by-Step Trace: Polynomial Hash

```
String: "hello", base = 31, mod = 997

Using Horner's method (efficient evaluation):
  hash = 0
  hash = hash × 31 + 'h' = 0 × 31 + 104  = 104
  hash = hash × 31 + 'e' = 104 × 31 + 101 = 3325
  hash = hash × 31 + 'l' = 3325 × 31 + 108 = 103183
  hash = hash × 31 + 'l' = 103183 × 31 + 108 = 3198781
  hash = hash × 31 + 'o' = 3198781 × 31 + 111 = 99162322

  h("hello") = 99162322 mod 997 = 819

  (In practice, take mod at each step to prevent overflow)

With mod at each step:
  hash = 0
  hash = (0 × 31 + 104) mod 997  = 104
  hash = (104 × 31 + 101) mod 997 = 3325 mod 997 = 334
  hash = (334 × 31 + 108) mod 997 = 10462 mod 997 = 494
  hash = (494 × 31 + 108) mod 997 = 15422 mod 997 = 470
  hash = (470 × 31 + 111) mod 997 = 14681 mod 997 = 726

  h("hello") = 726  (different due to intermediate mod, but still valid)
```

---

## Pseudocode

```
// Polynomial hash (Horner's method)
FUNCTION hash_string(s, table_size):
    base = 31
    hash_value = 0
    FOR i = 0 TO length(s) - 1:
        hash_value = (hash_value * base + ASCII(s[i])) MOD table_size
    RETURN hash_value

// Java's String.hashCode() implementation
FUNCTION java_string_hash(s):
    hash = 0
    FOR i = 0 TO length(s) - 1:
        hash = hash * 31 + s[i]    // Uses 32-bit integer overflow
    RETURN hash

// Double-hash for reduced collisions
FUNCTION hash_string_double(s, table_size):
    base1 = 31
    base2 = 37
    h1 = 0
    h2 = 0
    FOR i = 0 TO length(s) - 1:
        h1 = (h1 * base1 + ASCII(s[i])) MOD table_size
        h2 = (h2 * base2 + ASCII(s[i])) MOD table_size
    RETURN (h1, h2)    // Use both for collision verification
```

---

## Comparison of String Hash Methods

| Method | Speed | Quality | Handles Anagrams? |
|--------|:-----:|:-------:|:-----------------:|
| First character | O(1) | Terrible | No |
| Sum of chars | O(n) | Poor | **No** |
| Polynomial (base 31) | O(n) | **Good** | **Yes** |
| Polynomial (double) | O(n) | **Excellent** | **Yes** |
| Crypto (SHA-256) | O(n) | Excellent | Yes (overkill) |

---

## Language Implementations

```
// Java
"hello".hashCode()
// = 'h'×31⁴ + 'e'×31³ + 'l'×31² + 'l'×31¹ + 'o'×31⁰

// Python (simplified concept — actual CPython uses SipHash)
hash("hello")
// Returns consistent integer (randomized per session since 3.3)

// C++ std::hash<string>
std::hash<std::string>{}("hello")
// Implementation-defined, typically FNV-1a or similar
```

---

## Quick Revision Question

**Q: Given the string "abc" with base=31 and table_size=100, compute the polynomial hash step by step using Horner's method.**

<details>
<summary>Click to reveal answer</summary>

Characters: 'a'=97, 'b'=98, 'c'=99

**Using Horner's method** (multiply by base, add next char, mod):

1. `hash = 0`
2. `hash = (0 × 31 + 97) mod 100 = 97`
3. `hash = (97 × 31 + 98) mod 100 = 3105 mod 100 = 5`
4. `hash = (5 × 31 + 99) mod 100 = 254 mod 100 = 54`

**h("abc") = 54**

Verification without intermediate mod:
$97 \times 31^2 + 98 \times 31 + 99 = 97 \times 961 + 98 \times 31 + 99 = 93217 + 3038 + 99 = 96354$
$96354 \mod 100 = 54$ ✓

</details>

---

| [← Previous: Cryptographic Hash Functions](05-cryptographic-hash-functions.md) | [Unit 3: Separate Chaining →](../03-Collision-Handling-Chaining/01-separate-chaining.md) |
|:---|---:|
