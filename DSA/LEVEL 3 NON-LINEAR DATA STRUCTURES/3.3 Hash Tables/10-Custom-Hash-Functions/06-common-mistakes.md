# 10.6 Common Hashing Mistakes

## Unit 10: Custom Hash Functions

---

## Overview

```
╔══════════════════════════════════════════════════════════════╗
║  THE 8 DEADLY SINS OF HASHING                               ║
║                                                              ║
║  1. Overriding equals() without hashCode()                  ║
║  2. Using mutable fields in hash computation                ║
║  3. Using bad combining strategies (XOR, sum)               ║
║  4. Ignoring null handling                                  ║
║  5. Constant hash function                                  ║
║  6. Not pre-sizing when count is known                      ║
║  7. Using wrong data structure                              ║
║  8. Trusting hash for security without crypto hash          ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Mistake 1: equals() Without hashCode()

```java
// ✗ WRONG — Violates the contract
class Student {
    String name;
    int age;

    @Override
    public boolean equals(Object o) {
        Student s = (Student) o;
        return name.equals(s.name) && age == s.age;
    }
    // hashCode() NOT overridden!
}
```

### What Goes Wrong

```
Student a = new Student("Alice", 20);
Student b = new Student("Alice", 20);

a.equals(b) → true
a.hashCode() → 735842 (memory address)
b.hashCode() → 291053 (different memory address)

HashSet<Student> set = new HashSet<>();
set.add(a);
set.contains(b) → FALSE!  ← BUG: equal objects not found!
```

### Fix

```java
// ✓ CORRECT
@Override
public int hashCode() {
    return Objects.hash(name, age);  // Must use same fields as equals
}
```

---

## Mistake 2: Mutable Keys

```python
# ✗ WRONG — Mutable list as dict key concept
class Config:
    def __init__(self, settings):
        self.settings = settings  # mutable list!

    def __hash__(self):
        return hash(tuple(self.settings))

    def __eq__(self, other):
        return self.settings == other.settings

c = Config([1, 2, 3])
cache = {c: "result"}
c.settings.append(4)     # MUTATION!
cache[c]                  # KeyError! Hash changed!
```

### Fix

```python
# ✓ CORRECT — Use immutable data
class Config:
    def __init__(self, settings):
        self.settings = tuple(settings)  # Frozen!

    def __hash__(self):
        return hash(self.settings)

    def __eq__(self, other):
        return self.settings == other.settings
```

---

## Mistake 3: Bad Combining Strategies

```
╔══════════════════════════════════════════════════════════════╗
║  ✗ XOR:  hash = x ^ y                                      ║
║    Point(3,5) = 6,  Point(5,3) = 6,  Point(6,0) = 6       ║
║    → Commutative → position-insensitive → many collisions  ║
║                                                              ║
║  ✗ SUM:  hash = x + y                                      ║
║    Point(1,4) = 5,  Point(2,3) = 5,  Point(4,1) = 5       ║
║    → Commutative → same problem as XOR                     ║
║                                                              ║
║  ✗ AND:  hash = x & y                                      ║
║    Lossy — many inputs produce zero bits                    ║
║                                                              ║
║  ✗ OR:   hash = x | y                                      ║
║    Lossy — many inputs produce one bits                     ║
║                                                              ║
║  ✓ POLYNOMIAL:  hash = 31 * x + y                          ║
║    Point(3,5) = 98,  Point(5,3) = 158,  Point(6,0) = 186  ║
║    → Order-sensitive → distinct values → few collisions    ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Mistake 4: Ignoring Null Fields

```java
// ✗ WRONG — NullPointerException!
@Override
public int hashCode() {
    return name.hashCode() + address.hashCode();
    //     ↑ Crashes if name is null!
}

// ✓ CORRECT — Null-safe
@Override
public int hashCode() {
    return Objects.hash(name, address);
    // Objects.hash handles nulls (returns 0 for null)
}

// ✓ CORRECT — Manual null handling
@Override
public int hashCode() {
    int h = 17;
    h = 31 * h + (name == null ? 0 : name.hashCode());
    h = 31 * h + (address == null ? 0 : address.hashCode());
    return h;
}
```

---

## Mistake 5: Constant Hash Function

```java
// ✗ CATASTROPHICALLY WRONG
@Override
public int hashCode() {
    return 42;  // "Technically" satisfies the contract
}
```

```
╔══════════════════════════════════════════════════════════════╗
║  All objects hash to 42 → all land in the SAME bucket      ║
║                                                              ║
║  ┌────────────────────────────────────────────┐             ║
║  │ Bucket 42: A → B → C → D → E → ... → Z   │             ║
║  └────────────────────────────────────────────┘             ║
║                                                              ║
║  Every operation degrades to O(n):                          ║
║    Search: linear scan through ALL entries                  ║
║    Insert: walk entire chain to check duplicates            ║
║    Delete: linear search                                    ║
║                                                              ║
║  HashMap becomes a LINKED LIST.                             ║
║  1 million entries → 1 million probes per lookup!           ║
║                                                              ║
║  Contract is satisfied (equal objects have equal hashes)    ║
║  but performance is destroyed.                              ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Mistake 6: Not Pre-Sizing

```python
# ✗ SLOW — 20+ resizes for 1M entries
data = {}
for i in range(1_000_000):
    data[i] = compute(i)

# ✓ FASTER — Pre-allocate (Python doesn't support this directly,
#   but Java/C++ do)
```

```java
// ✗ SLOW — Java
Map<Integer, String> map = new HashMap<>();  // default 16
for (int i = 0; i < 1_000_000; i++) {
    map.put(i, compute(i));  // Resizes ~17 times!
}

// ✓ FAST — Pre-sized
Map<Integer, String> map = new HashMap<>(1_400_000);  // > 1M/0.75
for (int i = 0; i < 1_000_000; i++) {
    map.put(i, compute(i));  // Zero resizes!
}
```

---

## Mistake 7: Wrong Data Structure Choice

```
╔══════════════════════════════════════════════════════════════╗
║  Using HashMap when you should use:                         ║
║                                                              ║
║  ✗ HashMap for sorted iteration                             ║
║    → Use TreeMap (sorted keys, O(log n) ops)                ║
║                                                              ║
║  ✗ HashMap for insertion-order iteration                    ║
║    → Use LinkedHashMap (Java) or OrderedDict (Python)       ║
║                                                              ║
║  ✗ HashMap for LRU cache                                    ║
║    → Use LinkedHashMap with removeEldestEntry               ║
║                                                              ║
║  ✗ HashMap for concurrent access                            ║
║    → Use ConcurrentHashMap (Java) or locks                  ║
║                                                              ║
║  ✗ HashMap for small fixed enum keys                        ║
║    → Use EnumMap (Java) — array-backed, faster              ║
║                                                              ║
║  ✗ HashMap for integer keys in small range                  ║
║    → Use simple array — O(1) with zero overhead             ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Mistake 8: Non-Crypto Hash for Security

```
╔══════════════════════════════════════════════════════════════╗
║  ✗ DANGEROUS:                                               ║
║  String passwordHash = String.valueOf(password.hashCode()); ║
║  // Java's hashCode is REVERSIBLE and predictable!          ║
║  // An attacker can find collisions trivially.              ║
║                                                              ║
║  ✓ CORRECT for password storage:                            ║
║  String hash = BCrypt.hashpw(password, BCrypt.gensalt());   ║
║  // Or: Argon2, scrypt, PBKDF2                              ║
║                                                              ║
║  ✓ CORRECT for integrity checking:                          ║
║  byte[] hash = MessageDigest.getInstance("SHA-256")         ║
║                             .digest(data);                  ║
║                                                              ║
║  RULE: hashCode() → for hash tables ONLY                    ║
║        SHA-256/BCrypt → for security applications           ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Mistake Reference Table

| # | Mistake | Consequence | Fix |
|---|---------|------------|-----|
| 1 | equals without hashCode | Lookup fails for equal objects | Override both, same fields |
| 2 | Mutable hash keys | Orphaned entries, memory leak | Use immutable keys |
| 3 | XOR / sum combining | Excessive collisions | Polynomial: `31*a + b` |
| 4 | Null field access | NullPointerException | `Objects.hash()` or null checks |
| 5 | Constant hashCode | O(n) everything | Use all identity fields |
| 6 | No pre-sizing | Many expensive resizes | `new HashMap<>(expectedSize / 0.75)` |
| 7 | Wrong structure | Missing features or overhead | Choose by access pattern |
| 8 | hashCode for security | Trivially reversible, vulnerable | Use bcrypt / SHA-256 |

---

## Debugging Hash Table Issues

```
╔══════════════════════════════════════════════════════════════╗
║  SYMPTOM → LIKELY CAUSE → FIX                               ║
║                                                              ║
║  map.get(key) returns null but key exists:                  ║
║    → equals/hashCode contract broken                        ║
║    → Key mutated after insertion                            ║
║    Fix: Check both methods, verify immutability             ║
║                                                              ║
║  map.put keeps adding duplicates:                           ║
║    → hashCode overridden but not equals                     ║
║    Fix: Override equals to compare by value                 ║
║                                                              ║
║  HashMap is slow (O(n) lookups):                            ║
║    → Bad hash function (too many collisions)                ║
║    → Load factor too high                                   ║
║    Fix: Check bucket distribution, improve hash function    ║
║                                                              ║
║  HashMap consuming too much memory:                         ║
║    → Load factor too low (too many empty buckets)           ║
║    → Never shrinks after mass deletion                      ║
║    Fix: Rebuild with right-sized constructor                ║
║                                                              ║
║  ConcurrentModificationException:                           ║
║    → Modifying map while iterating                          ║
║    Fix: Use iterator.remove() or ConcurrentHashMap         ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Final Best Practices Summary

```
╔══════════════════════════════════════════════════════════════╗
║              HASH TABLE GOLDEN RULES                        ║
║                                                              ║
║  1. Always override hashCode when you override equals       ║
║  2. Use the SAME fields in both methods                     ║
║  3. Make hash keys IMMUTABLE                                ║
║  4. Use polynomial combining: 31*h + field                  ║
║  5. Handle nulls explicitly                                 ║
║  6. Pre-size when you know the expected count               ║
║  7. Default load factor (0.75) is fine for most cases       ║
║  8. Use Objects.hash() / hash(tuple) — don't reinvent      ║
║  9. Test your hash function with realistic data             ║
║ 10. Never use hashCode() for security — use crypto hashes  ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Quick Revision Question

**Q: A developer writes `hashCode() { return 1; }` for a class used as HashMap keys. The program works correctly but is extremely slow. Explain: (a) why it's technically "correct", (b) what the time complexity becomes, and (c) what Java's HashMap does internally to partially mitigate this.**

<details>
<summary>Click to reveal answer</summary>

**(a) Why it's technically correct:**
The hashCode/equals contract only requires:
- If `a.equals(b)` then `a.hashCode() == b.hashCode()` ✓ (both return 1)
- `hashCode()` is consistent across calls ✓ (always returns 1)

Returning a constant satisfies both rules. The contract says nothing about distribution quality.

**(b) Time complexity:**
All entries land in **one bucket** → the hash table degrades to a single chain:
- **Insert**: O(n) — must scan entire chain to check for duplicates
- **Search**: O(n) — must scan chain linearly
- **Delete**: O(n) — must find entry first

The O(1) average-case guarantee assumes **uniform distribution**, which a constant hash function violates.

**(c) Java's mitigation — Treeification:**
Since Java 8, when a single bucket accumulates **≥ 8 entries**, the HashMap converts that bucket's linked list into a **Red-Black Tree**:
```
Before (≥ 8 entries): Linked List → O(n) search
After treeification:  Red-Black Tree → O(log n) search
```
So even with `hashCode() = 1`, Java degrades to O(log n) per operation instead of O(n). This is a safety net, not a substitute for a good hash function.

</details>

---

| [← Performance Tuning](05-performance-tuning.md) | [Back to Hash Tables README →](../README.md) |
|:---|---:|
