# 3.2 Linked List Implementation

## Unit 3: Collision Handling - Chaining

---

## Chain Data Structures

While separate chaining typically uses **singly linked lists**, other data structures can serve as chains. Each has trade-offs.

```
╔═══════════════════════════════════════════════════════════════╗
║            CHAIN DATA STRUCTURE OPTIONS                      ║
║                                                              ║
║   1. Singly Linked List (most common)                       ║
║      [key|val|next] ──► [key|val|next] ──► null             ║
║                                                              ║
║   2. Doubly Linked List                                      ║
║      null ◄──[prev|key|val|next]──►[prev|key|val|next]──►null║
║                                                              ║
║   3. Dynamic Array / ArrayList                               ║
║      [entry0, entry1, entry2, ...]                          ║
║                                                              ║
║   4. Balanced BST (Java 8+ TreeMap)                         ║
║              [key3]                                          ║
║             /      \                                         ║
║          [key1]  [key5]                                      ║
╚═══════════════════════════════════════════════════════════════╝
```

---

## Singly Linked List Node

```
╔═══════════════════════════════════════════════════════╗
║                 NODE STRUCTURE                       ║
║                                                      ║
║   ┌─────────────────────────────────┐                ║
║   │  Node                           │                ║
║   │  ┌──────┬───────┬──────────┐   │                ║
║   │  │ key  │ value │ next ──►││   │                ║
║   │  └──────┴───────┴──────────┘   │                ║
║   └─────────────────────────────────┘                ║
║                                                      ║
║   Example chain at bucket 3:                         ║
║                                                      ║
║   bucket[3]                                          ║
║      │                                               ║
║      ▼                                               ║
║   ┌──────┬──────┬────┐    ┌──────┬──────┬────┐      ║
║   │"cat" │  5   │ ──►├───►│"dog" │  3   │null│      ║
║   └──────┴──────┴────┘    └──────┴──────┴────┘      ║
╚═══════════════════════════════════════════════════════╝
```

---

## Full Implementation Pseudocode

```
CLASS Node:
    key
    value
    next = null
    
    FUNCTION init(key, value):
        this.key = key
        this.value = value

CLASS ChainedHashTable:
    FUNCTION init(capacity = 16):
        this.capacity = capacity
        this.size = 0
        this.buckets = new Array(capacity, null)  // All null initially
    
    FUNCTION hash(key):
        h = key.hashCode()
        RETURN abs(h) MOD this.capacity
    
    // ─── INSERT ────────────────────────────────
    FUNCTION put(key, value):
        index = this.hash(key)
        current = this.buckets[index]
        
        // Check if key already exists → update
        WHILE current != null:
            IF current.key == key:
                current.value = value    // Update existing
                RETURN
            current = current.next
        
        // Key not found → prepend new node
        newNode = new Node(key, value)
        newNode.next = this.buckets[index]
        this.buckets[index] = newNode
        this.size = this.size + 1
        
        // Check if resize needed
        IF this.size / this.capacity > 0.75:
            this.resize()
    
    // ─── SEARCH ────────────────────────────────
    FUNCTION get(key):
        index = this.hash(key)
        current = this.buckets[index]
        
        WHILE current != null:
            IF current.key == key:
                RETURN current.value     // Found!
            current = current.next
        
        RETURN null                      // Not found
    
    // ─── DELETE ────────────────────────────────
    FUNCTION remove(key):
        index = this.hash(key)
        current = this.buckets[index]
        prev = null
        
        WHILE current != null:
            IF current.key == key:
                IF prev == null:
                    this.buckets[index] = current.next   // Remove head
                ELSE:
                    prev.next = current.next              // Bypass node
                this.size = this.size - 1
                RETURN current.value
            prev = current
            current = current.next
        
        RETURN null    // Key not found
    
    // ─── RESIZE ────────────────────────────────
    FUNCTION resize():
        oldBuckets = this.buckets
        this.capacity = this.capacity * 2
        this.buckets = new Array(this.capacity, null)
        this.size = 0
        
        FOR i = 0 TO length(oldBuckets) - 1:
            current = oldBuckets[i]
            WHILE current != null:
                this.put(current.key, current.value)
                current = current.next
```

---

## Trace: Delete Operation

```
Table state (h(k) = k mod 5):
  ┌───┐
  │ 2 │──► [17] ──► [12] ──► [7] ──► null
  └───┘

DELETE(12):
  index = h(12) = 2
  
  Step 1: current = [17], prev = null
          17 ≠ 12, move forward
          
  Step 2: current = [12], prev = [17]
          12 = 12, FOUND!
          prev.next = current.next
          [17].next = [7]
  
  Result:
  ┌───┐
  │ 2 │──► [17] ──► [7] ──► null
  └───┘
  
  Node [12] is removed, chain shortened by 1.
```

---

## Chain Structure Comparison

| Structure | Insert | Search | Delete | Space | Cache |
|-----------|:------:|:------:|:------:|:-----:|:-----:|
| Singly Linked List | O(1) | O(k) | O(k) | 1 ptr/node | Poor |
| Doubly Linked List | O(1) | O(k) | O(1)* | 2 ptr/node | Poor |
| Dynamic Array | O(1)† | O(k) | O(k) | Compact | **Good** |
| BST (balanced) | O(log k) | O(log k) | O(log k) | 2 ptr/node | Poor |

*k = chain length, †amortized, \*given node reference*

> **Java 8+** uses linked list for short chains (≤8) and converts to Red-Black tree for long chains, giving O(log k) worst case.

---

## Quick Revision Question

**Q: In the delete operation for separate chaining, why do we need to track the `prev` pointer? Can we avoid it?**

<details>
<summary>Click to reveal answer</summary>

We need `prev` to **reconnect the chain** after removing a node. When we find the target node, we set `prev.next = current.next` to bypass the deleted node. Without `prev`, we can't update the predecessor's `next` pointer.

**Alternatives to avoid tracking `prev`**:
1. **Doubly linked list**: Each node has a `prev` pointer, so deletion is O(1) once the node is found.
2. **Copy-and-delete**: Copy the next node's data into the current node and delete the next node (works only for non-tail nodes).
3. **Sentinel node**: Use a dummy head node so the first real node always has a predecessor.

</details>

---

| [← Previous: Separate Chaining](01-separate-chaining.md) | [Next: Load Factor →](03-load-factor.md) |
|:---|---:|
