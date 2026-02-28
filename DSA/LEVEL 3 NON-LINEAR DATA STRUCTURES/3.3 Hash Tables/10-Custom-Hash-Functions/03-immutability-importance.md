# 10.3 Immutability and Hash Keys

## Unit 10: Custom Hash Functions

---

## Why Hash Keys Must Be Immutable

```
╔══════════════════════════════════════════════════════════════╗
║  CORE INVARIANT:                                            ║
║                                                              ║
║  Once an object is placed in a hash table, its hash code    ║
║  MUST NEVER CHANGE.                                         ║
║                                                              ║
║  If hash code changes → object is in the WRONG bucket      ║
║  → lookups, deletions, and contains-checks all FAIL        ║
║  → the object becomes a "ghost" — present but unfindable   ║
╚══════════════════════════════════════════════════════════════╝
```

---

## The Mutable Key Disaster — Step by Step

```java
class MutablePoint {
    int x, y;    // NOT final — can be changed!

    @Override
    public int hashCode() { return 31 * x + y; }

    @Override
    public boolean equals(Object o) {
        MutablePoint p = (MutablePoint) o;
        return x == p.x && y == p.y;
    }
}
```

### Trace of the Bug

```
Step 1: Create point and insert into HashMap
─────────────────────────────────────────────
  MutablePoint p = new MutablePoint(3, 4);
  p.hashCode() = 31 * 3 + 4 = 97

  map.put(p, "treasure");

  Bucket layout (table size = 16):
  bucket = 97 % 16 = 1
  ┌────────────────────────────────────┐
  │ [0]  │ [1]          │ [2] │ ...   │
  │ null │ (3,4)→"trea" │ null│       │
  └────────────────────────────────────┘

Step 2: Mutate the key object
─────────────────────────────
  p.x = 10;   // MUTATION!
  p.hashCode() = 31 * 10 + 4 = 314

  The object is STILL in bucket 1, but its hash is now 314.

  ┌────────────────────────────────────┐
  │ [0]  │ [1]              │ [2] │   │
  │ null │ (10,4)→"treasure"│ null│   │
  └────────────────────────────────────┘
           ↑ WRONG BUCKET!
           Object says bucket = 314 % 16 = 10
           But it's sitting in bucket 1

Step 3: Try to find the object
──────────────────────────────
  map.get(p);
  → Computes p.hashCode() = 314
  → Looks in bucket 314 % 16 = 10
  → Bucket 10 is empty!
  → Returns NULL   ← DATA LOST!

  map.containsKey(p) → false   ← LIES!
  map.size() → 1               ← It's still there!

  map.get(new MutablePoint(3, 4));
  → Computes hashCode = 97
  → Looks in bucket 1
  → Finds (10,4) — but equals((3,4), (10,4)) → false
  → Returns NULL   ← LOST FOREVER!

  ╔═══════════════════════════════════════╗
  ║  The entry is PERMANENTLY ORPHANED.   ║
  ║  Can't find by old key (3,4).         ║
  ║  Can't find by new key (10,4).        ║
  ║  It's a MEMORY LEAK.                  ║
  ╚═══════════════════════════════════════╝
```

---

## Making Keys Immutable

### Java: Use `final` Fields

```java
public final class ImmutablePoint {
    private final int x;
    private final int y;

    public ImmutablePoint(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() { return x; }
    public int getY() { return y; }

    @Override
    public int hashCode() { return 31 * x + y; }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ImmutablePoint)) return false;
        ImmutablePoint p = (ImmutablePoint) o;
        return x == p.x && y == p.y;
    }
}
```

### Java: `record` (Java 14+) — Automatic Immutability

```java
public record Point(int x, int y) {}
// Automatically generates:
// - final fields
// - constructor
// - hashCode() using all fields
// - equals() using all fields
// - toString()
```

### Python: `@dataclass(frozen=True)`

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Point:
    x: int
    y: int
    # __hash__ and __eq__ auto-generated
    # Fields cannot be modified after creation:
    # p = Point(3, 4)
    # p.x = 10   → FrozenInstanceError!
```

### Python: `NamedTuple`

```python
from typing import NamedTuple

class Point(NamedTuple):
    x: int
    y: int
    # Immutable and hashable by default
```

---

## What If You Must Use Mutable Objects?

```
╔══════════════════════════════════════════════════════════════╗
║  STRATEGY 1: Copy on Insert                                 ║
║    Store a defensive copy of the key                        ║
║    map.put(new Point(p.x, p.y), value);                    ║
║    ⚠ Caller's reference can still mutate their copy         ║
║      but the map's copy is safe                             ║
║                                                              ║
║  STRATEGY 2: Identity-Based Hashing                         ║
║    Use System.identityHashCode() — based on memory address  ║
║    Never changes, but two "equal" objects are distinct      ║
║    Java: IdentityHashMap does this                          ║
║                                                              ║
║  STRATEGY 3: Extract an Immutable Key                       ║
║    Use a stable, immutable field as the key                 ║
║    e.g., use student.getId() instead of the Student object  ║
║    Map<String, Student> — String is immutable ✓             ║
║                                                              ║
║  STRATEGY 4: Cache the Hash Code                            ║
║    Compute hash once, store in a field, never recompute     ║
║    ⚠ Still breaks equals() if fields change                 ║
║    ⚠ Only use if mutation is guaranteed not to happen       ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Hash Code Caching Pattern

```java
public class CachedHashKey {
    private final int cachedHash;
    private String name;  // Mutable but hash is frozen

    public CachedHashKey(String name) {
        this.name = name;
        this.cachedHash = name.hashCode();  // Computed ONCE
    }

    @Override
    public int hashCode() {
        return cachedHash;  // Always returns same value
    }

    // ⚠ WARNING: if name changes, equals() and hashCode()
    //    will be inconsistent — contract broken!
}
```

> Java's `String` class uses this pattern safely because `String` is immutable. It lazily caches its hash code on first call.

---

## Immutable Key Checklist

| Requirement | How to Ensure |
|------------|--------------|
| Fields don't change | Use `final` / `readonly` / `frozen` |
| Class can't be subclassed | Use `final class` / `sealed` |
| No mutable references leak | Return defensive copies from getters |
| Contained objects are immutable | Use immutable types for fields (String, Integer) |
| hashCode stays stable | Derived from immutable fields only |
| equals stays consistent | Same fields as hashCode |

---

## Built-In Immutable Types (Safe as Keys)

```
╔═══════════════════════════════════════════╗
║  SAFE (Immutable):        UNSAFE:         ║
║  ─────────────     ──────────────         ║
║  String             StringBuilder         ║
║  Integer, Long      int[] (array)         ║
║  Double, Float      ArrayList             ║
║  BigDecimal         HashMap               ║
║  LocalDate          Date (mutable!)       ║
║  Tuple (Python)     list (Python)         ║
║  frozenset          set (Python)          ║
║  record (Java 14+)  regular class         ║
╚═══════════════════════════════════════════╝
```

---

## Quick Revision Question

**Q: You insert an object `p` as a key in a HashMap, then mutate `p`'s fields. After mutation, `map.get(p)` returns `null` even though `map.size()` is 1. Explain step-by-step why this happens and how to prevent it.**

<details>
<summary>Click to reveal answer</summary>

**Step-by-step explanation:**

1. **Insert**: `map.put(p, val)` computes `p.hashCode()` → say `hash₁` → places entry in `bucket₁`
2. **Mutate**: Changing `p`'s fields changes what `p.hashCode()` returns → now produces `hash₂`
3. **Lookup**: `map.get(p)` computes `p.hashCode()` → gets `hash₂` → looks in `bucket₂`
4. **Miss**: `bucket₂` doesn't contain the entry (it's in `bucket₁`) → returns `null`
5. **Orphan**: The entry in `bucket₁` can never be found because:
   - The current key `p` hashes to `bucket₂`
   - A new object with the **old** field values would hash to `bucket₁` but `equals()` would fail (fields don't match anymore)

**Prevention strategies:**
1. **Make the key class immutable** (best solution) — use `final` fields, `record`, or `@dataclass(frozen=True)`
2. **Use a defensive copy**: `map.put(p.copy(), val)` — isolate the map's key from external mutation
3. **Use a natural immutable key**: `map.put(p.getId(), val)` — use an immutable identifier field instead of the whole object

</details>

---

| [← hashCode/equals Contract](02-hashcode-equals-contract.md) | [Next: Collision Minimization →](04-collision-minimization.md) |
|:---|---:|
