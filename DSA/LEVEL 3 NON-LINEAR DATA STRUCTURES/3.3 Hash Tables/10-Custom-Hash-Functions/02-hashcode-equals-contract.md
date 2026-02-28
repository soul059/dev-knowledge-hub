# 10.2 The hashCode/equals Contract

## Unit 10: Custom Hash Functions

---

## The Golden Rule

```
╔══════════════════════════════════════════════════════════════╗
║                THE hashCode/equals CONTRACT                 ║
║                                                              ║
║  Rule 1 (Consistency):                                      ║
║    If a.equals(b) is TRUE                                   ║
║    then a.hashCode() MUST equal b.hashCode()                ║
║                                                              ║
║  Rule 2 (One-Way):                                          ║
║    If a.hashCode() == b.hashCode()                          ║
║    a.equals(b) may or may not be true (collisions exist)    ║
║                                                              ║
║  Rule 3 (Stability):                                        ║
║    Multiple calls to hashCode() on an unchanged object      ║
║    must return the same value within one execution           ║
║                                                              ║
║  Rule 4 (equals Symmetry):                                  ║
║    If a.equals(b) then b.equals(a)                          ║
║                                                              ║
║  Rule 5 (equals Transitivity):                              ║
║    If a.equals(b) and b.equals(c) then a.equals(c)          ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Visual Summary

```
      equals() is TRUE          hashCode() is EQUAL
     ──────────────────  ──→  ──────────────────────
     a.equals(b) = true  IMPLIES  a.hashCode() == b.hashCode()

                 ⚠ BUT the reverse is NOT guaranteed:

      hashCode() is EQUAL          equals() is ???
     ──────────────────────  ──→  ─────────────────
     a.hashCode() == b.hashCode()  MAY → a.equals(b) = false
                                               (collision)
```

---

## What Happens When the Contract is Broken

### Case 1: Override `equals()` but NOT `hashCode()`

```
class Student {
    String name;

    @Override
    public boolean equals(Object o) {
        return this.name.equals(((Student)o).name);
    }

    // hashCode() NOT overridden — uses default (memory address)
}
```

```
╔══════════════════════════════════════════════════════════════╗
║  Student s1 = new Student("Alice");   // hashCode → 1a2b3c ║
║  Student s2 = new Student("Alice");   // hashCode → 4d5e6f ║
║                                                              ║
║  s1.equals(s2) → TRUE   ✓  (same name)                     ║
║  s1.hashCode() ≠ s2.hashCode()   ✗  CONTRACT VIOLATED!     ║
║                                                              ║
║  ┌──────────┐                                               ║
║  │ Bucket 3 │ → [s1:"Alice"]     ← s1 lands here           ║
║  │ Bucket 7 │ → [s2:"Alice"]     ← s2 lands here           ║
║  └──────────┘                                               ║
║                                                              ║
║  map.put(s1, "GPA: 3.8");                                   ║
║  map.get(s2);   → NULL!  s2 checks bucket 7, but data is   ║
║                   in bucket 3. Lookup fails!                ║
║                                                              ║
║  Result: "Equal" objects can't find each other in the map   ║
╚══════════════════════════════════════════════════════════════╝
```

### Case 2: Override `hashCode()` but NOT `equals()`

```
class Student {
    String name;

    @Override
    public int hashCode() {
        return name.hashCode();
    }

    // equals() NOT overridden — uses default (reference equality)
}
```

```
╔══════════════════════════════════════════════════════════════╗
║  Student s1 = new Student("Alice");                         ║
║  Student s2 = new Student("Alice");                         ║
║                                                              ║
║  s1.hashCode() == s2.hashCode()   ✓  (same name hash)      ║
║  s1.equals(s2) → FALSE  (different objects in memory)       ║
║                                                              ║
║  ┌──────────┐                                               ║
║  │ Bucket 3 │ → [s1:"Alice"] → [s2:"Alice"]                ║
║  └──────────┘                                               ║
║                                                              ║
║  map.put(s1, "GPA: 3.8");                                   ║
║  map.put(s2, "GPA: 3.9");   // Doesn't overwrite s1!       ║
║  map.size() → 2   (two separate entries for "same" student) ║
║                                                              ║
║  Result: Duplicates accumulate — map acts like a multimap   ║
╚══════════════════════════════════════════════════════════════╝
```

### Case 3: Both Overridden Correctly ✓

```
╔══════════════════════════════════════════════════════════════╗
║  s1.hashCode() == s2.hashCode()   ✓                        ║
║  s1.equals(s2) → TRUE             ✓                        ║
║                                                              ║
║  ┌──────────┐                                               ║
║  │ Bucket 3 │ → [s1:"Alice"]                                ║
║  └──────────┘                                               ║
║                                                              ║
║  map.put(s1, "GPA: 3.8");                                   ║
║  map.put(s2, "GPA: 3.9");  // Overwrites! Same key.        ║
║  map.size() → 1   ✓                                         ║
║  map.get(new Student("Alice")) → "GPA: 3.9"   ✓            ║
║                                                              ║
║  Result: Hash map works correctly.                          ║
╚══════════════════════════════════════════════════════════════╝
```

---

## HashMap Lookup Internals

```
GET(key):
    Step 1: bucket = key.hashCode() % table.length
                    ─────────────
                    Narrows to ONE bucket

    Step 2: For each entry e in table[bucket]:
                IF key.hashCode() == e.key.hashCode()    // Fast check
                   AND key.equals(e.key)                  // Slow check
                THEN return e.value
                   ──────────────
                   Confirms EXACT match

    Step 3: Return null (not found)
```

The two-step process:
1. **hashCode()** → finds the **bucket** (fast, narrows search space)
2. **equals()** → confirms the **exact key** (within that bucket)

Both are needed — neither alone suffices.

---

## Complete Correct Implementation

```java
public class Employee {
    private final String id;
    private final String name;
    private final int department;

    // Constructor...

    @Override
    public boolean equals(Object obj) {
        // 1. Reference check (fast path)
        if (this == obj) return true;

        // 2. Null and type check
        if (obj == null || getClass() != obj.getClass()) return false;

        // 3. Field comparison
        Employee other = (Employee) obj;
        return this.department == other.department
            && Objects.equals(this.id, other.id)
            && Objects.equals(this.name, other.name);
    }

    @Override
    public int hashCode() {
        // Use the SAME fields as equals()
        return Objects.hash(id, name, department);
    }
}
```

---

## The Fields Rule

```
╔══════════════════════════════════════════════════════════════╗
║              WHICH FIELDS TO INCLUDE?                       ║
║                                                              ║
║  hashCode() fields  ⊆  equals() fields                     ║
║                                                              ║
║  Fields in equals() → MUST be in hashCode()                 ║
║  Fields NOT in equals() → MUST NOT be in hashCode()         ║
║                                                              ║
║  If equals uses: {id, name}                                 ║
║  hashCode uses:  {id, name}        ✓  CORRECT               ║
║  hashCode uses:  {id}              ✓  OK (fewer is fine*)    ║
║  hashCode uses:  {id, name, age}   ✗  WRONG (age not in     ║
║                                        equals → breaks rule) ║
║                                                              ║
║  * Fewer fields is legal but increases collisions.          ║
║    Using ALL equals-fields gives the best distribution.     ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Contract Violation Summary

| Scenario | Contract? | HashMap Behavior |
|----------|:---------:|-----------------|
| Override both correctly | ✓ Valid | Works correctly |
| Override `equals()` only | ✗ Broken | Equal objects in different buckets — lookup fails |
| Override `hashCode()` only | ⚠ Legal | Same bucket, but duplicates accumulate |
| Override neither | ✓ Valid | Reference equality — each object is unique |
| Different fields in each | ✗ Broken | Unpredictable: may find or miss entries |

---

## Python's Equivalent Contract

```python
# Python automatically makes objects unhashable
# if you override __eq__ without __hash__

class Broken:
    def __eq__(self, other):
        return True

s = set()
s.add(Broken())  # TypeError: unhashable type: 'Broken'
                  # Python PREVENTS the broken state!

# Fix: override both
class Fixed:
    def __init__(self, val):
        self.val = val

    def __eq__(self, other):
        return isinstance(other, Fixed) and self.val == other.val

    def __hash__(self):
        return hash(self.val)
```

> Python 3 is **stricter** than Java: if you define `__eq__`, it automatically sets `__hash__ = None`, making the class unhashable until you explicitly define `__hash__`.

---

## Quick Revision Question

**Q: A class overrides `equals()` based on fields `{name, age}` but implements `hashCode()` using only `{name}`. Does this violate the contract? What is the practical consequence?**

<details>
<summary>Click to reveal answer</summary>

**No, this does NOT violate the contract.** The contract requires:

> If `a.equals(b)` then `a.hashCode() == b.hashCode()`

If two objects are equal, they have the same `name` AND `age`. Since `hashCode()` uses `name`, and both have the same `name`, their hash codes will be equal. ✓

**However, the practical consequence is more collisions:**
```
Student("Alice", 20) → hashCode = hash("Alice") = 63476538
Student("Alice", 21) → hashCode = hash("Alice") = 63476538  ← COLLISION
Student("Alice", 22) → hashCode = hash("Alice") = 63476538  ← COLLISION
```

All students named "Alice" (regardless of age) collide into the same bucket. The hash map degrades toward O(n) for lookups among same-name students.

**Best practice:** Include ALL fields used in `equals()` in `hashCode()` for optimal distribution.

</details>

---

| [← Hashing Custom Objects](01-hashing-custom-objects.md) | [Next: Immutability Importance →](03-immutability-importance.md) |
|:---|---:|
