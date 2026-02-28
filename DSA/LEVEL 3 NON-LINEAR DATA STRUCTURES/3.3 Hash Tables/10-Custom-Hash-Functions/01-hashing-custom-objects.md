# 10.1 Hashing Custom Objects

## Unit 10: Custom Hash Functions

---

## The Problem

Built-in hash functions work for primitives and strings — but how do you use **custom objects** (e.g., `Point`, `Student`, `Edge`) as hash map keys?

```
╔══════════════════════════════════════════════════════════════╗
║  // This won't work as expected by default:                 ║
║                                                              ║
║  Point p1 = new Point(3, 4);                                ║
║  Point p2 = new Point(3, 4);                                ║
║                                                              ║
║  map.put(p1, "A");                                          ║
║  map.get(p2);       // Returns null! p1 ≠ p2 by reference  ║
║                                                              ║
║  WHY? Default hashCode() uses memory address.               ║
║  p1 and p2 are different objects → different hashes         ║
║  → different buckets → lookup fails                        ║
║                                                              ║
║  SOLUTION: Override hashCode() and equals()                 ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Java Example

```java
class Point {
    int x, y;

    Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public int hashCode() {
        return 31 * x + y;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof Point)) return false;
        Point other = (Point) obj;
        return this.x == other.x && this.y == other.y;
    }
}
```

---

## Python Example

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __hash__(self):
        return hash((self.x, self.y))

    def __eq__(self, other):
        return isinstance(other, Point) and \
               self.x == other.x and self.y == other.y

# Now works as dict key / set element:
seen = set()
seen.add(Point(3, 4))
Point(3, 4) in seen  # True ✓
```

---

## Combining Multiple Fields

The standard approach uses a **polynomial combination** of field hashes:

```
FUNCTION hashCode():
    result = 17                    // Start with a non-zero prime
    result = 31 * result + hash(field1)
    result = 31 * result + hash(field2)
    result = 31 * result + hash(field3)
    ...
    RETURN result
```

### Why 31?

```
╔══════════════════════════════════════════════════════════════╗
║  Properties of multiplier 31:                               ║
║                                                              ║
║  1. It's prime → fewer systematic collisions                ║
║  2. It's odd → won't zero out bits (even * anything stays  ║
║     even, losing the low bit forever)                       ║
║  3. 31 = 2⁵ - 1 → 31*x = (x << 5) - x                    ║
║     JVM optimizes this to a shift and subtract              ║
║  4. Empirically good distribution for string/object data    ║
║                                                              ║
║  Other common multipliers: 37, 41, 53, 131                  ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Trace: Hash Computation

```
class Student:
    name: "Alice"   (hash = 63476538)
    age:  20
    id:   "S001"    (hash = 2554747)

hashCode():
  result = 17
  result = 31 * 17 + 63476538 = 527 + 63476538 = 63477065
  result = 31 * 63477065 + 20 = 1967789015 + 20 = 1967789035
  result = 31 * 1967789035 + 2554747
         = (overflow → wraps in 32-bit) = some_int

Student("Alice", 20, "S001") will always hash to the same value ✓
```

---

## Multi-Field Hashing Strategies

| Strategy | Formula | Quality |
|----------|---------|:---:|
| Sum | `a + b + c` | Poor — order insensitive |
| XOR | `a ^ b ^ c` | Poor — cancellation |
| Polynomial | `31*(31*a + b) + c` | Good ✓ |
| Tuple hash | `hash((a, b, c))` | Good (Python) ✓ |
| `Objects.hash()` | `Objects.hash(a, b, c)` | Good (Java) ✓ |

```
╔══════════════════════════════════════════════════════════════╗
║  WHY XOR IS BAD:                                            ║
║                                                              ║
║  hash(Point(3, 5)) = 3 ^ 5 = 6                             ║
║  hash(Point(5, 3)) = 5 ^ 3 = 6   ← COLLISION!             ║
║  hash(Point(6, 0)) = 6 ^ 0 = 6   ← COLLISION!             ║
║                                                              ║
║  XOR is commutative: a^b = b^a → order doesn't matter     ║
║  XOR with 0 is identity: a^0 = a → zero fields ignored    ║
║                                                              ║
║  WHY SUM IS BAD:                                            ║
║  hash(Point(1, 4)) = 5                                     ║
║  hash(Point(2, 3)) = 5   ← COLLISION!                      ║
║  hash(Point(4, 1)) = 5   ← COLLISION!                      ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Handling Different Field Types

```
FUNCTION hashField(field):
    IF field is null:       RETURN 0
    IF field is boolean:    RETURN field ? 1 : 0
    IF field is byte/short/int/char:  RETURN (int) field
    IF field is long:       RETURN (int)(field ^ (field >>> 32))
    IF field is float:      RETURN Float.floatToIntBits(field)
    IF field is double:
        bits = Double.doubleToLongBits(field)
        RETURN (int)(bits ^ (bits >>> 32))
    IF field is array:      RETURN Arrays.hashCode(field)
    IF field is object:     RETURN field.hashCode()
```

---

## Language-Specific Approaches

```
╔══════════════════════════════════════════════════════════════╗
║  JAVA:                                                      ║
║    Override hashCode() and equals()                         ║
║    Use Objects.hash(field1, field2, ...)                    ║
║    IDE can auto-generate these methods                      ║
║                                                              ║
║  PYTHON:                                                    ║
║    Override __hash__() and __eq__()                          ║
║    Use hash(tuple_of_fields) — simplest approach            ║
║    @dataclass(frozen=True) auto-generates both!             ║
║                                                              ║
║  C++:                                                       ║
║    Specialize std::hash<T> in namespace std                 ║
║    Override operator== as well                               ║
║    Or provide hash functor to unordered_map                 ║
║                                                              ║
║  JAVASCRIPT:                                                ║
║    No built-in custom hashing for Map keys                  ║
║    Use serialized string as key: JSON.stringify(obj)        ║
║    Or use a custom toString() method                        ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Quick Revision Question

**Q: You have a `Pair` class with two integer fields `first` and `second`. Why would `hash = first + second` be a bad hash function? Give a concrete example showing excess collisions.**

<details>
<summary>Click to reveal answer</summary>

`hash = first + second` is **order-insensitive** and creates many collisions because different pairs can sum to the same value:

```
Pair(1, 5) → hash = 6
Pair(2, 4) → hash = 6
Pair(3, 3) → hash = 6
Pair(4, 2) → hash = 6
Pair(5, 1) → hash = 6
Pair(0, 6) → hash = 6
Pair(6, 0) → hash = 6
```

**Seven** distinct pairs all hash to 6! In general, for sum $= s$, there are $s + 1$ pairs of non-negative integers — all colliding.

**Better alternatives:**
```
// Polynomial (best general-purpose):
hash = 31 * first + second
Pair(1,5)→36, Pair(2,4)→66, Pair(3,3)→96, Pair(5,1)→156
// All different ✓

// Cantor pairing function:
hash = (first + second) * (first + second + 1) / 2 + second
// Bijective! Guaranteed unique for all non-negative pairs
```

</details>

---

| [← Unit 9: Locality-Sensitive Hashing](../09-Advanced-Hashing/06-locality-sensitive-hashing.md) | [Next: hashCode/equals Contract →](02-hashcode-equals-contract.md) |
|:---|---:|
