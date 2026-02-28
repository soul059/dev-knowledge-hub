# 3.6 Implementation Details

## Unit 3: Collision Handling - Chaining

---

## Production-Ready Considerations

Building a chained hash table for production requires handling edge cases, choosing proper initial capacity, implementing dynamic resizing, and supporting iteration.

---

## Complete Implementation

```
CLASS Node:
    key, value, next
    FUNCTION init(key, value, next = null):
        this.key = key; this.value = value; this.next = next

CLASS ChainedHashMap:
    CONSTANT DEFAULT_CAPACITY = 16
    CONSTANT DEFAULT_LOAD_FACTOR = 0.75
    CONSTANT MINIMUM_CAPACITY = 4
    
    FUNCTION init(capacity = DEFAULT_CAPACITY, loadFactor = DEFAULT_LOAD_FACTOR):
        // Ensure capacity is power of 2
        this.capacity = next_power_of_2(capacity)
        this.maxLoadFactor = loadFactor
        this.size = 0
        this.buckets = new Array(this.capacity, null)
        this.threshold = floor(this.capacity * this.maxLoadFactor)
    
    // ─── HASH FUNCTION ──────────────────────────
    FUNCTION hash(key):
        IF key == null: RETURN 0
        h = key.hashCode()
        // Spread bits: XOR high bits into low bits
        h = h XOR (h >>> 16)
        RETURN h AND (this.capacity - 1)    // Faster than mod for power-of-2
    
    // ─── PUT ─────────────────────────────────────
    FUNCTION put(key, value):
        index = this.hash(key)
        current = this.buckets[index]
        
        WHILE current != null:
            IF current.key == key OR (key != null AND key.equals(current.key)):
                oldValue = current.value
                current.value = value
                RETURN oldValue    // Return previous value
            current = current.next
        
        // Insert new node at head
        this.buckets[index] = new Node(key, value, this.buckets[index])
        this.size++
        
        IF this.size > this.threshold:
            this.resize(this.capacity * 2)
        
        RETURN null    // No previous value
    
    // ─── GET ─────────────────────────────────────
    FUNCTION get(key):
        index = this.hash(key)
        current = this.buckets[index]
        
        WHILE current != null:
            IF current.key == key OR (key != null AND key.equals(current.key)):
                RETURN current.value
            current = current.next
        
        RETURN null
    
    // ─── CONTAINS KEY ────────────────────────────
    FUNCTION containsKey(key):
        RETURN this.get(key) != null
    
    // ─── REMOVE ──────────────────────────────────
    FUNCTION remove(key):
        index = this.hash(key)
        current = this.buckets[index]
        prev = null
        
        WHILE current != null:
            IF current.key == key OR (key != null AND key.equals(current.key)):
                IF prev == null:
                    this.buckets[index] = current.next
                ELSE:
                    prev.next = current.next
                this.size--
                RETURN current.value
            prev = current
            current = current.next
        
        RETURN null
    
    // ─── RESIZE ──────────────────────────────────
    FUNCTION resize(newCapacity):
        oldBuckets = this.buckets
        this.capacity = newCapacity
        this.buckets = new Array(newCapacity, null)
        this.threshold = floor(newCapacity * this.maxLoadFactor)
        this.size = 0
        
        FOR i = 0 TO length(oldBuckets) - 1:
            current = oldBuckets[i]
            WHILE current != null:
                nextNode = current.next
                this.put(current.key, current.value)
                current = nextNode
```

---

## Bit Manipulation Trick: Power-of-2 Modulo

```
╔═══════════════════════════════════════════════════════════════╗
║           WHY POWER-OF-2 CAPACITY?                           ║
║                                                              ║
║   h(k) mod m   ← requires expensive division                ║
║   h(k) & (m-1) ← bitwise AND, much faster!                 ║
║                                                              ║
║   Only works when m is a power of 2:                        ║
║                                                              ║
║   m = 16 = 10000₂                                          ║
║   m-1 = 15 = 01111₂                                        ║
║                                                              ║
║   h = 43 = 101011₂                                         ║
║   43 & 15 = 101011 & 001111 = 001011 = 11                  ║
║   43 % 16 = 11  ← Same result!                             ║
║                                                              ║
║   Combined with XOR bit spreading:                          ║
║   h = h ^ (h >>> 16)    // Mix high and low bits           ║
║   index = h & (capacity - 1)                               ║
╚═══════════════════════════════════════════════════════════════╝
```

---

## Rehashing Trace

```
Before resize: capacity=4, size=3, threshold=3

Buckets:
  [0]: ("apple",1) → null
  [1]: null
  [2]: ("banana",2) → ("cherry",3) → null
  [3]: null

Resize to capacity=8:
  Rehash "apple":  h("apple") & 7  = new index
  Rehash "banana": h("banana") & 7 = new index
  Rehash "cherry": h("cherry") & 7 = new index

After resize: capacity=8, size=3, threshold=6

Buckets (redistributed):
  [0]: null
  [1]: ("apple",1) → null
  [2]: null
  [3]: ("banana",2) → null
  [4]: null
  [5]: ("cherry",3) → null     ← was sharing bucket with "banana"
  [6]: null
  [7]: null

Collision resolved! Elements spread across more buckets.
```

---

## Iteration Support

```
// Iterate over all key-value pairs
FUNCTION forEach(callback):
    FOR i = 0 TO this.capacity - 1:
        current = this.buckets[i]
        WHILE current != null:
            callback(current.key, current.value)
            current = current.next

// Get all keys
FUNCTION keys():
    result = new List()
    this.forEach((key, value) => result.add(key))
    RETURN result

// Note: Iteration order is NOT guaranteed!
// Different from insertion order.
// Use LinkedHashMap for insertion-order iteration.
```

---

## Memory Layout Analysis

```
╔═══════════════════════════════════════════════════════════════╗
║              MEMORY OVERHEAD                                 ║
║                                                              ║
║   Bucket array: m × pointer_size                            ║
║   Each node:    key + value + next_pointer                  ║
║                                                              ║
║   Total memory = m × 8 bytes (pointers)                     ║
║                + n × (key_size + value_size + 8 bytes)      ║
║                                                              ║
║   Example: m=16, n=10, keys=4B ints, values=4B ints        ║
║   Array:  16 × 8  = 128 bytes                               ║
║   Nodes:  10 × 16 = 160 bytes                               ║
║   Total:  288 bytes                                          ║
║                                                              ║
║   Overhead per entry: ~13 bytes (pointer + array slot share)║
║   This is why open addressing can be more memory-efficient  ║
╚═══════════════════════════════════════════════════════════════╝
```

---

## Quick Revision Question

**Q: Why does Java's HashMap use `h ^ (h >>> 16)` before computing the bucket index? What problem does this solve?**

<details>
<summary>Click to reveal answer</summary>

When the capacity is a power of 2, the bucket index is computed as `h & (capacity - 1)`, which only uses the **lower bits** of the hash. If many keys differ only in their upper bits, they'll all map to the same bucket.

The operation `h ^ (h >>> 16)` **XORs the upper 16 bits into the lower 16 bits**, effectively mixing information from all parts of the hash code into the bits that actually determine the bucket index. This provides better distribution especially for hash codes where the lower bits have poor entropy.

Example: Two keys with hash codes `0x0000ABCD` and `0x1234ABCD` would map to the same bucket without bit spreading (same lower bits), but after XOR they get different lower bits.

</details>

---

| [← Previous: Worst Case Scenario](05-worst-case-scenario.md) | [Unit 4: Open Addressing Concept →](../04-Collision-Handling-Open-Addressing/01-open-addressing-concept.md) |
|:---|---:|
