# Chapter 6.2 — Rolling Hash Technique

> **Unit 6: String Hashing** | [Course Home](../README.md)

---

## 📋 Chapter Overview

Rolling hash allows updating a substring hash in O(1) as we slide a window
across the string. This chapter covers both forward and backward rolling,
with applications to fixed-length and variable-length problems.

---

## 1. Rolling Hash Recap

```
  Window slides from left to right, one position at a time.
  
  Current window:   S[i..i+m-1]    hash = ht
  Next window:      S[i+1..i+m]

  ┌────────────────────────────────────────────────────┐
  │  Recurrence:                                       │
  │  ht_new = (ht - S[i] × p^(m-1)) × p + S[i+m]    │
  │           ─────────────  ───  ────────             │
  │           remove left   shift  add right           │
  │                                (all mod M)         │
  └────────────────────────────────────────────────────┘

  Note: This uses the "big-endian" polynomial form:
  hash = S[i]×p^(m-1) + S[i+1]×p^(m-2) + ... + S[i+m-1]×p⁰
```

---

## 2. Two Polynomial Forms

```
  Form 1: "Big-endian" (Rabin-Karp style)
  ────────────────────────────────────────
  hash = S[0]×p^(m-1) + S[1]×p^(m-2) + ... + S[m-1]×p⁰

  Rolling: ht = (ht - S[i]×p^(m-1)) × p + S[i+m]
  Advantage: Natural rolling forward.

  Form 2: "Little-endian" (prefix hash style)  
  ────────────────────────────────────────────
  hash = S[0]×p⁰ + S[1]×p¹ + ... + S[m-1]×p^(m-1)

  Rolling requires modular inverse of p.
  Advantage: Works well with prefix hash arrays.

  Both are equally valid. Pick one and be consistent!
```

---

## 3. Step-by-Step Rolling

```
  S = "abcde"   m = 3   p = 31   M = 1000

  Character values: a=1, b=2, c=3, d=4, e=5

  Using Form 1 (big-endian):
  h = S[0]×31² + S[1]×31 + S[2]

  Window "abc": h = 1×961 + 2×31 + 3 = 961+62+3 = 1026
                h mod 1000 = 26

  Roll to "bcd":
    Remove 'a'(1): ht = (26 - 1×961) = 26 - 961 = -935
                   ht mod 1000 = 65  (add 1000)
    Shift: ht = 65 × 31 = 2015 mod 1000 = 15
    Add 'd'(4): ht = 15 + 4 = 19

  Verify "bcd": h = 2×961 + 3×31 + 4 = 1922+93+4 = 2019
                h mod 1000 = 19 ✓ 

  Roll to "cde":
    ht = (19 - 2×961) × 31 + 5 mod 1000
    = (19 - 1922) × 31 + 5 mod 1000
    = (-1903) × 31 + 5 mod 1000
    = -58993 + 5 mod 1000
    = -58988 mod 1000 = 12

  Verify "cde": h = 3×961 + 4×31 + 5 = 2883+124+5 = 3012
                h mod 1000 = 12 ✓
```

---

## 4. Rolling with Prefix Hashes (Alternative)

```
  If you have prefix hashes h[0..n] and power array pw[0..n]:
  
  hash(S[l..r]) can be computed in O(1) without rolling:
    raw = (h[r+1] - h[l] + M) % M
    
  To compare hash(l₁,r₁) with hash(l₂,r₂):
    raw₁ × pw[l₂] == raw₂ × pw[l₁]  (mod M)

  This avoids explicit rolling entirely!
  Useful when you need RANDOM ACCESS to substring hashes.
```

---

## 5. Implementation — Rolling Hash Class

```python
class RollingHash:
    def __init__(self, s, window_size, p=31, mod=10**9 + 7):
        self.s = s
        self.m = window_size
        self.p = p
        self.mod = mod
        
        # Precompute p^(m-1)
        self.pm = pow(p, window_size - 1, mod)
        
        # Compute first window hash
        self.current_hash = 0
        for i in range(window_size):
            self.current_hash = (self.current_hash * p + (ord(s[i]) - ord('a') + 1)) % mod
        
        self.pos = 0  # current window starts at index 0
    
    def get_hash(self):
        return self.current_hash
    
    def roll(self):
        """Slide window one position to the right."""
        if self.pos + self.m >= len(self.s):
            return False
        
        old_char = ord(self.s[self.pos]) - ord('a') + 1
        new_char = ord(self.s[self.pos + self.m]) - ord('a') + 1
        
        self.current_hash = (
            (self.current_hash - old_char * self.pm) * self.p + new_char
        ) % self.mod
        
        self.pos += 1
        return True


# Example: find pattern
def rolling_search(text, pattern):
    m = len(pattern)
    rh_text = RollingHash(text, m)
    rh_pat = RollingHash(pattern, m)
    target = rh_pat.get_hash()
    
    matches = []
    pos = 0
    while True:
        if rh_text.get_hash() == target:
            if text[pos:pos + m] == pattern:  # verify
                matches.append(pos)
        if not rh_text.roll():
            break
        pos += 1
    return matches
```

---

## 6. Bidirectional Rolling

```
  Sometimes we need to roll BACKWARDS (right to left).

  Forward roll (standard):
    Remove left, add right:
    ht = (ht - S[i] × p^(m-1)) × p + S[i+m]

  Backward roll:
    Remove right, add left:
    ht = (ht - S[i+m-1]) × p⁻¹ + S[i-1] × p^(m-1)

  Requires modular inverse p⁻¹ = pow(p, M-2, M) (Fermat's theorem)

  ┌─────────────────────────────────────────────┐
  │  Bidirectional rolling is useful for:       │
  │  • Longest palindromic substring (binary    │
  │    search + hash expansion both ways)       │
  │  • Two-pointer hash problems                │
  └─────────────────────────────────────────────┘
```

---

## 7. Variable-Length Rolling

```
  For problems where window size changes dynamically:

  Append character c to the right:
    hash = hash × p + val(c)

  Remove character from the right:
    hash = (hash - val(c)) × p⁻¹

  Append character c to the left (prepend):
    hash = hash + val(c) × p^len

  Remove character from the left:
    hash = hash - val(c) × p^(len-1)

  Each operation: O(1)
  Requires maintaining current length.
```

---

## 📝 Summary Table

| Operation | Formula (big-endian) | Cost |
|-----------|---------------------|------|
| Initial hash | Horner's method | O(m) |
| Roll right | (ht - S[i]×p^(m-1)) × p + S[i+m] | O(1) |
| Roll left | (ht - S[i+m-1]) × p⁻¹ + S[i-1]×p^(m-1) | O(1) |
| Append right | ht × p + val(c) | O(1) |
| Prepend left | ht + val(c) × p^len | O(1) |
| Prefix hash query | h[r+1] - h[l] | O(1) |

---

## ❓ Quick Revision Questions

1. **What are the two polynomial forms and when do you use each?**
   <details><summary>Answer</summary>Big-endian: S[0]×p^(m-1)+...+S[m-1]×p⁰, natural for rolling forward. Little-endian: S[0]×p⁰+...+S[m-1]×p^(m-1), works well with prefix hash arrays for random-access substring hashing.</details>

2. **Why do we precompute p^(m-1)?**
   <details><summary>Answer</summary>To remove the leftmost character's contribution during rolling. Computing p^(m-1) inside the loop would make each roll O(m) instead of O(1).</details>

3. **How do you handle negative hash values during rolling?**
   <details><summary>Answer</summary>Add M before taking mod: ht = ((ht - S[i]×pm) % M + M) % M. This handles the case where subtraction produces a negative number.</details>

4. **What is p⁻¹ and when do you need it?**
   <details><summary>Answer</summary>The modular inverse of p, computed as pow(p, M-2, M) using Fermat's little theorem. Needed for backward rolling and removing characters from the right side of the hash.</details>

5. **What is the advantage of prefix hashes over explicit rolling?**
   <details><summary>Answer</summary>Prefix hashes allow O(1) random-access to ANY substring's hash, not just the next window. Useful when you need non-consecutive substring hashes.</details>

---

| [⬅️ Previous: Polynomial Hash](01-polynomial-hash.md) | [Next: Hash Collisions ➡️](03-hash-collisions.md) |
|:---|---:|
